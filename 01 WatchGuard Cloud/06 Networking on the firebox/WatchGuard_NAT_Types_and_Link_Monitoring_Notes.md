# WatchGuard Firebox — NAT Types & Link Monitoring

## 1. NAT Overview

WatchGuard Fireware supports three main NAT types:

1. Dynamic NAT
2. Static NAT (SNAT / Port Forwarding)
3. 1-to-1 NAT

### Mental model

- **Dynamic NAT:** LAN → Internet, changes the source IP.
- **Static NAT:** Internet → Server, forwards a public IP/port to a private server.
- **1-to-1 NAT:** maps one public IP to one private IP (or a range/subnet).

---

# 2. Dynamic NAT

Dynamic NAT is also called **IP masquerading**. It changes the source IP of an outgoing connection to a public IP on the Firebox.

Example:

```text
PC 192.168.10.50
       |
       | source translated
       v
Firebox 203.0.113.10
       |
       v
Internet
```

The Internet sees the Firebox public IP rather than the private client IP.

### Common use

- Internet access for internal hosts
- Many private hosts sharing one public IP
- Hiding internal private addresses from external networks

WatchGuard documents Dynamic NAT as enabled by default for traffic from RFC1918 private addresses to the external network.

---

# 3. Static NAT (SNAT / Port Forwarding)

Static NAT is commonly called **port forwarding**.

It maps a public IP/port to an internal IP/port.

Example:

```text
Internet
203.0.113.10:443
       |
       | Static NAT
       v
192.168.20.10:443
Web Server
```

Use it when publishing a specific service such as HTTPS.

Example policy concept:

```text
Source:      Any-External
Destination: Web_Server
Service:     HTTPS
Action:      Allow
```

Static NAT can be configured for connections to External or Optional interfaces, with documented restrictions for Trusted/Custom interfaces and some VPN scenarios.

---

# 4. 1-to-1 NAT

1-to-1 NAT creates a mapping between IP addresses on different networks.

Example:

```text
203.0.113.20  <---->  192.168.20.20
```

It can map:

- One IP
- An IP range
- A subnet

It is useful when you have multiple public IPs or a server needs a dedicated public IP.

### Important rule

**1-to-1 NAT has precedence over Dynamic NAT.**

WatchGuard recommends considering SNAT + DNAT/Static NAT instead of 1-to-1 NAT in many smaller/public-IP-limited deployments.

---

# 5. NAT Comparison

| Type | Direction / purpose | Mapping | Typical use |
|---|---|---|---|
| Dynamic NAT | Outbound | Private source → Public source | Internet access |
| Static NAT / SNAT | Inbound | Public IP:Port → Private IP:Port | Publish a service |
| 1-to-1 NAT | Broad IP mapping | Public IP ↔ Private IP | Dedicated public IP |

### Memory trick

```text
Dynamic NAT = Outbound masquerading
Static NAT  = Port forwarding
1-to-1 NAT  = IP-to-IP mapping
```

---

# 6. Link Monitor

Link Monitor checks whether a Firebox interface has usable connectivity by sending probes to remote targets.

A physical interface can be **UP** while Internet connectivity beyond the ISP gateway is broken:

```text
Firebox
   |
   | Physical link UP
   |
ISP Gateway
   X
Internet
```

Link Monitor tests beyond the local physical link.

If probes fail according to the configured thresholds, the Firebox can consider the interface **inactive**.

Link Monitor status is used by **Multi-WAN** and **SD-WAN**.

---

# 7. Link Monitor Target Types

WatchGuard supports:

### Ping

```text
Firebox → ICMP → Target
```

Tests basic IP reachability.

### TCP

```text
Firebox → TCP/443 → Target
```

Useful when you want to test reachability to a specific TCP service/port.

### DNS

The Firebox queries a DNS server to resolve a specified domain.

Useful for testing DNS resolution as part of connectivity.

Current Fireware documentation allows up to **three targets per interface**.

---

# 8. Choosing Link Monitor Targets

For External interfaces, the default target is normally the **default gateway**.

WatchGuard recommends using a target farther upstream for meaningful operational monitoring.

Why?

```text
Firebox → ISP Gateway → Internet
          |
          | gateway still responds
          X Internet broken
```

The gateway can be reachable while the Internet path beyond it is unavailable.

WatchGuard recommends at least **two Link Monitor targets for each external interface**.

Choose reliable targets with good uptime.

---

# 9. Internal Interface Link Monitoring

Fireware 12.4+ supports Link Monitor for:

- Trusted
- Optional
- Custom

Internal interfaces must have a next-hop IP address or custom target configured.

The next-hop address helps the Firebox route Link Monitor and SD-WAN traffic through the correct interface.

---

# 10. Link Monitor Settings

Current default values documented by WatchGuard:

| Setting | Default |
|---|---:|
| Probe interval | 5 seconds |
| Deactivate after | 3 consecutive failures |
| Reactivate after | 3 consecutive successes |
| Targets | Up to 3 per interface |

Example:

```text
Probe 1 → Failure
Probe 2 → Failure
Probe 3 → Failure
             ↓
       Interface inactive
```

Recovery:

```text
Success 1
Success 2
Success 3
   ↓
Interface active
```

The exact failover behavior also depends on Multi-WAN/SD-WAN configuration.

---

# 11. Require All Targets

WatchGuard provides:

**Require a successful probe to all targets to define the interface as active**

If enabled, all required targets must successfully respond for the interface to be considered active.

Example:

```text
Target 1 → Success
Target 2 → Success
Target 3 → Success
             ↓
          ACTIVE
```

---

# 12. Link Monitor + Multi-WAN

Example:

```text
                 FIREBOX
                /                    WAN 1     WAN 2
              ISP-A     ISP-B
                |         |
           Link Monitor  Link Monitor
                \         /
                 \       /
                  Internet
```

If WAN 1 fails:

```text
WAN 1
  ↓
Link Monitor probes fail
  ↓
WAN 1 = Inactive
  ↓
Multi-WAN Failover
  ↓
WAN 2
```

For Multi-WAN Failover, the first interface in the configured list is the primary interface.

---

# 13. Link Monitor + SD-WAN

Link Monitor can also support SD-WAN decisions.

A Link Monitor target can be selected to measure:

- Packet loss
- Latency
- Jitter

If configured thresholds are exceeded, SD-WAN can choose another path according to its configured action.

This is more advanced than simply checking whether an interface is physically connected.

---

# 14. Physical Link vs Link Monitor

This is a very important interview concept.

### Physical failure

```text
Cable disconnected
      ↓
Interface DOWN
```

### Link Monitor failure

```text
Cable connected
      ↓
Interface physically UP
      ↓
Remote connectivity fails
      ↓
Probes fail
      ↓
Interface considered INACTIVE
```

**Physical link status ≠ end-to-end connectivity.**

---

# 15. Practical Dual-WAN Lab

```text
                 INTERNET
                /        \
             ISP 1       ISP 2
               |           |
             WAN 1       WAN 2
                \         /
                 \       /
                  FIREBOX
                     |
                    LAN
                     |
                   Switch
```

Example:

```text
WAN 1:
Target 1 = reliable upstream host
Target 2 = second reliable host

Probe interval = 5 sec
Deactivate after = 3 failures
Reactivate after = 3 successes

Multi-WAN:
Primary = WAN 1
Backup  = WAN 2
```

Test:

1. Disconnect ISP 1 or otherwise simulate WAN failure.
2. Confirm Link Monitor probes fail.
3. Confirm WAN 1 becomes inactive.
4. Confirm traffic uses WAN 2.
5. Restore ISP 1.
6. Confirm successful probes.
7. Confirm WAN 1 becomes active again according to the configured Multi-WAN behavior.

---

# 16. Troubleshooting Link Monitor

If an interface is physically UP but Link Monitor says inactive, check:

- Target is reachable.
- Target is not blocking the selected probe type.
- Correct routing exists.
- DNS works when using a DNS target.
- TCP port is reachable when using TCP.
- Correct next hop is configured for internal interfaces.
- Probe interval.
- Failure threshold.
- Multiple-target settings.

If failover does not occur, check:

```text
[ ] Link Monitor configured
[ ] Correct interfaces selected
[ ] Targets work during normal operation
[ ] Targets fail when WAN is unavailable
[ ] Multi-WAN mode configured
[ ] Primary/backup order correct
[ ] Failure threshold appropriate
[ ] Routing/policy configuration correct
```

---

# 17. NAT + Link Monitor Together

They solve different problems.

### NAT

> How should source/destination IP addresses be translated?

### Link Monitor

> Is this network path actually usable?

### Multi-WAN

> Which available path should carry traffic?

Combined:

```text
LAN
 |
 | Dynamic NAT
 v
WAN 1
 |
 | Link Monitor
 v
Internet

WAN 1 fails
    ↓
Link Monitor
    ↓
WAN 1 inactive
    ↓
Multi-WAN
    ↓
WAN 2
    ↓
Dynamic NAT
    ↓
Internet
```

---

# 18. Interview Questions

### Q1. What are the three NAT types in WatchGuard?

Dynamic NAT, Static NAT (SNAT/port forwarding), and 1-to-1 NAT.

### Q2. What is Dynamic NAT?

It changes the source IP of outgoing connections to a public IP.

### Q3. What is Static NAT?

It forwards a public IP/port to an internal IP/port.

### Q4. What is 1-to-1 NAT?

It maps IP addresses between networks, potentially for a host, range, or subnet.

### Q5. Which takes precedence: Dynamic NAT or 1-to-1 NAT?

1-to-1 NAT.

### Q6. What is Link Monitor?

A Firebox feature that probes remote targets through an interface to determine whether the interface has usable connectivity.

### Q7. What probe types are supported?

Ping, TCP, and DNS.

### Q8. How many targets can be configured?

Up to three per interface in current Fireware versions.

### Q9. What is the default probe interval?

5 seconds.

### Q10. What is the default deactivate threshold?

3 consecutive failures.

### Q11. What is the default reactivate threshold?

3 consecutive successful probes.

### Q12. Why is Link Monitor important for Multi-WAN?

It lets the Firebox determine when an interface is inactive so Multi-WAN can fail over or remove that interface from path selection according to the configured mode.

---

# 19. Quick Revision

```text
NAT
 |
 +-- Dynamic NAT
 |     +-- Outbound
 |     +-- Source translation
 |     +-- Internet access
 |
 +-- Static NAT / SNAT
 |     +-- Port forwarding
 |     +-- Public IP:Port → Private IP:Port
 |
 +-- 1-to-1 NAT
       +-- Public IP ↔ Private IP
       +-- Broad IP mapping


LINK MONITOR
 |
 +-- Probe target
 |     +-- Ping
 |     +-- TCP
 |     +-- DNS
 |
 +-- Measure connectivity
 |
 +-- Interface Active / Inactive
 |
 +-- Used by Multi-WAN / SD-WAN
```

## Final mental model

**NAT:** "How should this traffic's addresses be translated?"

**Link Monitor:** "Can this interface actually reach the configured target?"

**Multi-WAN:** "Which available path should be used?"

---

# Official WatchGuard References

- About Network Address Translation (NAT):  
  https://www.watchguard.com/help/docs/help-center/en-us/Content/en-us/Fireware/nat/network_addr_translation_about_c.html

- About Dynamic NAT:  
  https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/nat/nat_dynamic_use_c.html

- Configure Static NAT (SNAT):  
  https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/nat/nat_static_config_about_c.html

- About 1-to-1 NAT:  
  https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/fireware/nat/one_to_one_nat_c.html

- Configure Link Monitor:  
  https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/link%20monitor/link_monitor_configure.html

- About Link Monitor:  
  https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/link%20monitor/link_monitor_about.html

- Configure Failover Multi-WAN:  
  https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/Fireware/multiwan/failover_configure_c.html
