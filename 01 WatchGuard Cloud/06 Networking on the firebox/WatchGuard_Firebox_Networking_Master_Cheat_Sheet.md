# 🔥 WatchGuard Firebox — Networking Master Cheat Sheet

> **Source basis:** Uploaded WatchGuard training notes covering Network Modes, Firebox Interfaces, VLANs, NAT, Link Monitor, Multi-WAN, SD-WAN, and Failback.
>
> **Purpose:** Fast revision + interview preparation + practical lab reference.

---

# 1. 🧠 BIG PICTURE

Think about Firebox networking in this order:

```text
NETWORK MODE
     ↓
INTERFACE
     ↓
SECURITY ZONE
     ↓
IP NETWORK / VLAN
     ↓
ROUTING
     ↓
NAT
     ↓
LINK MONITOR
     ↓
MULTI-WAN / SD-WAN
     ↓
FIREWALL POLICY
     ↓
TRAFFIC DECISION
```

### Core mental models

```text
Network Mode
= How the Firebox sits in the network

Interface
= Where traffic enters/leaves

Security Zone
= How Fireware classifies the network

VLAN
= Logical Layer-2 segmentation

NAT
= How addresses are translated

Link Monitor
= Is this path actually usable?

Multi-WAN
= Which WAN path should be used?

SD-WAN
= Which path is best for selected traffic?

Policy
= What traffic is allowed or denied?
```

---

# 2. 🌐 FIREBOX NETWORK MODES

WatchGuard's supplied notes identify three primary network modes:

1. **Mixed Routing**
2. **Drop-In**
3. **Bridge**

## Quick comparison

| Feature | Mixed Routing | Drop-In | Bridge |
|---|---|---|---|
| Default | ✅ | ❌ | ❌ |
| Separate subnets | ✅ | ❌ | ❌ normal routed interfaces |
| Routing | ✅ | Limited design | ❌ normal routing |
| NAT | ✅ | Not normally required | ❌ |
| VLAN routing | ✅ | ❌ | ❌ |
| Dynamic routing | Available | Limited | ❌ |
| Transparency | ❌ | Existing-network friendly | ✅ |
| Typical use | Most deployments | Existing network | Transparent inspection |

## Mixed Routing

**Mental model:**

> Normal routed firewall.

Typical topology:

```text
                 INTERNET
                     |
                 External
                     |
                  FIREBOX
                 /       \
          Trusted       Optional
        10.10.10.0/24  192.168.10.0/24
           Users          DMZ
```

Key points:

- Default and most flexible mode.
- Interfaces normally use different subnets.
- Supports normal routing and NAT.
- Common enterprise design.
- Supports interfaces such as External, Trusted, Optional, Custom, VLAN, Bridge and Link Aggregation according to configuration.

## Drop-In Mode

**Mental model:**

> Insert the Firebox into an existing IP network without changing host addressing.

Key points:

- Uses the same primary IP across interfaces.
- One logical network.
- Static IP required on the External interface.
- Hosts can retain existing IPs and gateways.
- NAT is not normally required for typical traffic to public servers.
- Dynamic routing such as OSPF, BGP and RIP is not supported.
- VLAN-tagged traffic cannot be routed.
- Link aggregation is not supported.

## Bridge Mode

**Mental model:**

> Transparent firewall inspection.

```text
Existing Network
       |
    FIREBOX
    Bridge
       |
    Gateway
       |
   Internet
```

Key points:

- Firebox is placed transparently between an existing network and gateway.
- No normal routing.
- No NAT.
- No VLAN routing.
- Firebox cannot act as a VPN endpoint in Bridge Mode.
- Useful when security inspection is needed without making Firebox the normal Layer-3 gateway.

---

# 3. 🔌 FIREBOX INTERFACES

A Firebox interface can be **physical or logical**.

### Physical

```text
Actual Firebox Ethernet port
```

### Logical

```text
VLAN
Bridge
Link Aggregation
```

Current supplied notes identify interface configurations including:

- External
- Trusted
- Optional
- Custom
- Bridge
- VLAN
- Link Aggregation
- Disabled

---

# 4. 🛡️ SECURITY ZONES

## External

**Think:** WAN / outside / Internet.

Typical use:

```text
Internet
   ↓
ISP
   ↓
External
   ↓
Firebox
```

Key points:

- Normally connects to the outside network.
- Normally has the default route `0.0.0.0/0`.
- Member of `Any-External`.
- Multiple External interfaces can participate in Multi-WAN.

## Trusted

**Think:** Main internal LAN.

Typical uses:

- Employee LAN
- Internal servers
- Management network
- Corporate wired network

Member of:

```text
Any-Trusted
```

## Optional

**Think:** Internal mixed-trust / DMZ.

Typical uses:

- Public web servers
- Mail servers
- FTP/SFTP servers
- Systems requiring separation from the main LAN

Member of:

```text
Any-Optional
```

### Critical point

```text
External = outside/WAN

Trusted = main internal LAN

Optional = internal mixed-trust / DMZ
```

**Optional does NOT mean Internet.**

## Custom

**Think:** Separate security zone requiring explicit policy control.

Custom is not automatically included in:

```text
Any-Trusted
Any-Optional
Any-External
```

Useful for designs such as:

```text
Trusted  → Corporate users
Optional → DMZ
Custom   → Guest Wi-Fi
```

---

# 5. 🏷️ INTERFACE ALIASES

| Interface type | Built-in alias |
|---|---|
| External | `Any-External` |
| Trusted | `Any-Trusted` |
| Optional | `Any-Optional` |
| Custom | No built-in Any-Custom alias |

### Why aliases matter

Instead of creating separate policies for every Trusted interface:

```text
Trusted-0
Trusted-1
Trusted VLAN 10
Trusted VLAN 20
```

you can use:

```text
Any-Trusted
```

---

# 6. 🌉 BRIDGE INTERFACE

A Bridge logically combines multiple interfaces into one network.

```text
          FIREBOX BRIDGE
             /       \
          Port 1    Port 2
             \       /
             Same network
```

Think:

```text
Bridge = Layer 2
       = Same network
```

The supplied notes describe:

- One logical interface
- One IP address
- Multiple member interfaces

### Intra-bridge traffic

For Fireware **12.7 and higher**, policies can be configured for traffic passing between bridge member interfaces.

---

# 7. 🏷️ VLAN

A VLAN logically separates devices into different Layer-2 broadcast domains.

Core mental model:

```text
Switch VLAN
     ↓
802.1Q tag
     ↓
Firebox VLAN interface
     ↓
Gateway IP
     ↓
Security Zone
     ↓
Firewall Policy
```

## Example VLAN design

| VLAN | Purpose | Example subnet |
|---|---|---|
| 10 | Users | `192.168.10.0/24` |
| 20 | Servers | `192.168.20.0/24` |
| 30 | Guest | `192.168.30.0/24` |
| 40 | Voice | `192.168.40.0/24` |
| 50 | Management | `192.168.50.0/24` |

## VLAN configuration fields

### VLAN Name

Meaningful name, with no spaces according to the supplied notes.

Example:

```text
Users_VLAN10
```

### VLAN ID

Layer-2 identifier.

Example:

```text
VLAN ID = 10
```

The Firebox VLAN ID must match the connected switch/network design.

### Security Zone

A VLAN can be assigned to:

- Trusted
- Optional
- Custom
- External

### Gateway IP

Layer-3 Firebox address.

Example:

```text
192.168.10.1/24
```

Clients in that VLAN use the Firebox address as their default gateway when the Firebox is routing the VLAN.

### Remember

```text
VLAN ID     = Layer-2 identifier
Gateway IP  = Layer-3 Firebox address
```

---

# 8. 🔗 TAGGED VS UNTAGGED

## Tagged

Ethernet frame contains an IEEE 802.1Q VLAN tag.

```text
Frame
+----------------------------+
| Ethernet | VLAN 20 | Data |
+----------------------------+
```

Typical use:

```text
Firebox
   |
802.1Q trunk
   |
Switch
```

## Untagged

Frame arrives without an 802.1Q tag for that VLAN.

Typical access/native VLAN concept.

### Memory rule

```text
Trunk  → Tagged VLANs
Access → Usually untagged
```

The Firebox and switch must be configured consistently.

---

# 9. 🚦 TRUNK VS ACCESS

## Trunk

Carries multiple VLANs.

```text
Firebox
   |
   | VLAN 10 tagged
   | VLAN 20 tagged
   | VLAN 30 tagged
   |
Switch
```

## Access-style port

Normally carries one VLAN untagged.

```text
PC
 |
 | untagged
 |
Switch
 |
VLAN 10
```

---

# 10. 🖥️ VLAN CONFIGURATION WORKFLOW

General Fireware Web UI workflow:

```text
Network
  ↓
Interfaces
  ↓
Select physical interface
  ↓
Edit
  ↓
Interface Type = VLAN
  ↓
Save
```

Then:

```text
Network
  ↓
VLAN
  ↓
Add
  ↓
Name
Description
VLAN ID
Security Zone
Gateway IP
Interface traffic setting
  ↓
Save
```

Choose:

```text
Tagged
Untagged
No Traffic
```

Then configure:

```text
DHCP / DHCP Relay
       ↓
Switch VLAN
       ↓
Switch trunk/access
       ↓
Firewall policies
       ↓
Testing
```

---

# 11. 🧩 VLAN + FIREWALL POLICY

Creating a VLAN does **not** automatically mean all traffic between VLANs is allowed.

Example:

```text
VLAN 10 Users
      |
     HTTPS
      ↓
VLAN 20 Servers
```

Example policy:

```text
Source:      VLAN10_Users
Destination: VLAN20_Servers
Service:     HTTPS
Action:      Allow
```

### Same-VLAN traffic

Hosts in the same Layer-2 VLAN normally communicate directly through the switch.

A firewall policy does not automatically force same-VLAN traffic through the Firebox.

---

# 12. 🔄 LINK AGGREGATION

Link Aggregation groups multiple physical interfaces into one logical interface.

```text
Firebox
 /    \
Port2 Port3
 \    /
  LAG
   |
Switch
```

Main benefits:

1. Cumulative throughput
2. Redundancy

Example:

```text
1 Gbps + 1 Gbps
      =
2 Gbps cumulative capacity
```

### Important

This does **not** normally mean one individual TCP connection automatically becomes 2 Gbps.

Traffic distribution depends on the aggregation mode and hashing.

## Modes

### Dynamic — 802.3ad

- Uses LACP.
- Connected switch must support/configure compatible LACP.

### Static

- Static aggregation.
- Connected device must be configured appropriately.

### Active-backup

- One member active at a time.
- Another member can become active if the first fails.
- Does not require switch link-aggregation support.

### Important limitation

Supplied notes state Link Aggregation is supported only in **Mixed Routing Mode**, and not every Firebox model supports it.

---

# 13. 🧭 NAT

WatchGuard Fireware supports three main NAT types in the supplied notes:

1. **Dynamic NAT**
2. **Static NAT / SNAT / Port Forwarding**
3. **1-to-1 NAT**

## One-line memory trick

```text
Dynamic NAT = Outbound masquerading
Static NAT  = Port forwarding
1-to-1 NAT  = IP-to-IP mapping
```

---

# 14. 🔁 DYNAMIC NAT

Dynamic NAT is also called **IP masquerading**.

It changes the source IP of an outgoing connection to a public IP on the Firebox.

```text
PC
192.168.10.50
    |
    | source translated
    ↓
Firebox
203.0.113.10
    |
    ↓
Internet
```

Common uses:

- Internet access for internal hosts
- Many private hosts sharing one public IP
- Hiding internal private addresses from external networks

---

# 15. 🌐 STATIC NAT / PORT FORWARDING

Static NAT commonly means **port forwarding**.

It maps:

```text
Public IP:Port
      ↓
Private IP:Port
```

Example:

```text
Internet
203.0.113.10:443
      |
 Static NAT
      ↓
192.168.20.10:443
Web Server
```

Example policy concept:

```text
Source:      Any-External
Destination: Web_Server
Service:     HTTPS
Action:      Allow
```

---

# 16. 🔀 1-to-1 NAT

Maps IP addresses between networks.

```text
203.0.113.20
      ↕
192.168.20.20
```

Can map:

- One IP
- Range
- Subnet

Useful when multiple public IPs are available or a server needs a dedicated public IP.

### Important

```text
1-to-1 NAT takes precedence over Dynamic NAT.
```

---

# 17. 🧮 NAT COMPARISON

| NAT | Direction / purpose | Mapping | Typical use |
|---|---|---|---|
| Dynamic | Outbound | Private source → Public source | Internet access |
| Static / SNAT | Inbound | Public IP:Port → Private IP:Port | Publish service |
| 1-to-1 | Broad mapping | Public IP ↔ Private IP | Dedicated public IP |

### NAT mental question

> **How should this traffic's addresses be translated?**

---

# 18. 📡 LINK MONITOR

Link Monitor checks whether an interface has usable connectivity by sending probes to remote targets.

### Critical concept

```text
Physical link UP
        ≠
End-to-end Internet connectivity
```

Example:

```text
Firebox
   |
ISP Gateway
   X
Internet
```

The interface can remain physically UP while upstream connectivity is broken.

If configured probes fail according to thresholds, the Firebox can consider the interface **inactive**.

Link Monitor is used by:

- Multi-WAN
- SD-WAN

---

# 19. 🎯 LINK MONITOR TARGET TYPES

## Ping

```text
Firebox → ICMP → Target
```

Tests basic IP reachability.

## TCP

```text
Firebox → TCP/443 → Target
```

Tests reachability to a specific TCP service/port.

## DNS

Firebox queries a DNS server to resolve a specified domain.

### Targets

Current supplied notes state:

```text
Up to 3 targets per interface
```

---

# 20. 📍 LINK MONITOR TARGET DESIGN

For External interfaces, the default target is normally the default gateway.

But a gateway can respond even when Internet connectivity beyond it is broken.

Better operational model:

```text
Firebox
   ↓
ISP Gateway
   ↓
Reliable upstream target
   ↓
Internet
```

Supplied notes recommend:

```text
At least 2 Link Monitor targets
for each external interface
```

Choose reliable targets.

---

# 21. ⏱️ LINK MONITOR DEFAULTS

| Setting | Default |
|---|---:|
| Probe interval | 5 seconds |
| Deactivate after | 3 consecutive failures |
| Reactivate after | 3 consecutive successes |
| Targets | Up to 3/interface |

Example:

```text
Failure 1
   ↓
Failure 2
   ↓
Failure 3
   ↓
Interface inactive
```

Recovery:

```text
Success 1
   ↓
Success 2
   ↓
Success 3
   ↓
Interface active
```

Exact failover behavior depends on Multi-WAN/SD-WAN configuration.

---

# 22. 🔗 LINK MONITOR + MULTI-WAN

```text
WAN 1
  ↓
Link Monitor
  ↓
Probes fail
  ↓
WAN 1 inactive
  ↓
Multi-WAN Failover
  ↓
WAN 2
```

For Failover Multi-WAN:

```text
First interface in configured list
=
Primary interface
```

---

# 23. 📈 LINK MONITOR + SD-WAN

Link Monitor can support SD-WAN decisions using:

- Packet loss
- Latency
- Jitter

Example:

```text
WAN 1
Loss    = 2%
Latency = 40 ms
Jitter  = 8 ms

WAN 2
Loss    = 8%
Latency = 220 ms
Jitter  = 70 ms
```

Configured SD-WAN thresholds can determine whether a path qualifies.

---

# 24. 🌍 MULTI-WAN

Multi-WAN allows a Firebox to use multiple external/WAN interfaces.

Uses include:

- Failover
- Load distribution
- Route-based path selection
- Bandwidth overflow
- SD-WAN path selection

### Scope

Supplied notes state Multi-WAN affects:

```text
Outbound, non-VPN traffic
```

It does **not** control:

- Incoming connections
- BOVPN traffic

Multi-WAN requires:

```text
Mixed Routing Mode
```

---

# 25. 🛠️ FOUR MAIN MULTI-WAN METHODS

1. **Failover**
2. **Routing Table / ECMP**
3. **Round Robin**
4. **Interface Overflow**

SD-WAN can also make routing decisions when an SD-WAN action is applied to a policy.

Supplied notes state:

```text
Failover = default Multi-WAN option
in Fireware v12.5.4 and later
```

---

# 26. 🔴 FAILOVER

Failover = active/standby.

```text
WAN 1 = Primary
WAN 2 = Backup
```

If WAN 1 becomes inactive:

```text
WAN 1 ❌
   ↓
WAN 2 ✅
```

The order determines failover order.

Example:

```text
1. WAN 1
2. WAN 2
3. WAN 3
```

If WAN 1 and WAN 2 fail:

```text
WAN 3
  ↓
Traffic
```

---

# 27. ⚖️ ROUTING TABLE / ECMP

Routing Table method uses the Firebox routing table.

If a specific route exists:

```text
Use that route
```

If no specified route determines the path:

```text
ECMP
```

## ECMP

**Equal-Cost Multi-Path**

Example:

```text
Internet

WAN 1 → Cost 5
WAN 2 → Cost 5
WAN 3 → Cost 5
```

Multiple equal-cost routes can distribute connections.

### Important

Supplied notes state the ECMP algorithm:

- Uses source/destination IP information.
- Does **not** use current interface traffic load as its decision factor.

---

# 28. ⚖️ FAILOVER VS ECMP

| Feature | Failover | Routing Table / ECMP |
|---|---|---|
| Primary WAN | Yes | No preferred WAN in same sense |
| Backup WAN | Yes | Multiple active paths |
| Main purpose | Availability | Route/load distribution |
| Decision | Interface order + availability | Route table + ECMP |
| Multiple WANs simultaneously | Normally no | Yes |
| Current byte load considered | No | No |
| Mental model | Active/standby | Equal-cost paths |

### Memory

```text
Failover
= WAN 1, then WAN 2 if WAN 1 fails

ECMP
= Multiple equal-cost paths
```

---

# 29. 🔄 ROUND ROBIN

Round Robin distributes outbound traffic across external interfaces.

It can use **relative weights**.

Example:

```text
WAN 1 = Weight 3
WAN 2 = Weight 2
```

Conceptually:

```text
WAN 1 ≈ 60%
WAN 2 ≈ 40%
```

The exact distribution depends on traffic/connection behavior.

### Important

Round Robin is not simply:

```text
Packet 1 → WAN 1
Packet 2 → WAN 2
```

Supplied notes describe it as connection/traffic distribution rather than blind packet alternation.

It is applied after:

1. Routes
2. Sticky connections
3. SD-WAN routing

have failed to determine the path.

---

# 30. 📊 INTERFACE OVERFLOW

Interface Overflow moves **new connections** to the next WAN when a configured bandwidth threshold is reached.

Example:

```text
WAN 1 = 100 Mbps
Threshold = 80 Mbps

WAN 2 = 300 Mbps
Threshold = 250 Mbps
```

Flow:

```text
Start
 ↓
WAN 1
 ↓
Threshold reached
 ↓
New connections → WAN 2
```

## Measurement

WatchGuard notes state that Firebox examines:

- TX
- RX

and uses the **higher** value.

Example:

```text
TX = 80 Mbps
RX = 120 Mbps

Measured = 120 Mbps
```

If every Interface Overflow WAN reaches its threshold, the supplied notes state the Firebox uses ECMP.

---

# 31. 📊 MULTI-WAN METHODS COMPARISON

| Method | Main purpose | Key decision |
|---|---|---|
| Failover | High availability | Primary/backup order |
| Routing Table / ECMP | Route-based distribution | Routing table + equal-cost paths |
| Round Robin | Load distribution | Connection distribution + weights |
| Interface Overflow | Bandwidth thresholding | Threshold reached |
| SD-WAN Failover | Quality-aware availability | Availability + loss/latency/jitter |
| SD-WAN Round Robin | Quality-aware distribution | Configured metrics + distribution |

---

# 32. ⭐ SD-WAN PRECEDENCE

This is one of the most important concepts.

If an outbound policy has an **SD-WAN routing action**, that policy-level SD-WAN decision takes precedence over the general Multi-WAN configuration for traffic matched by that policy.

### Without policy SD-WAN

```text
Policy
  ↓
Multi-WAN
  ↓
Path decision
```

### With policy SD-WAN

```text
Policy
  ↓
SD-WAN Action
  ↓
Path decision
```

### Memory

> **Policy SD-WAN overrides general Multi-WAN for matching traffic.**

---

# 33. 🎯 SD-WAN QUALITY METRICS

SD-WAN can evaluate:

```text
Loss
Latency
Jitter
```

This makes SD-WAN more granular than simple WAN availability.

Example:

```text
Normal Internet
    ↓
General Multi-WAN

VoIP Policy
    ↓
SD-WAN
    ↓
WAN 1 + WAN 2
    ↓
Evaluate loss/latency/jitter
```

Other traffic can continue following general Multi-WAN behavior.

---

# 34. 🔁 SD-WAN FAILOVER

Example:

```text
Primary WAN
     ↓
Healthy?
  /     \
YES      NO
 |        |
Use      Fail over
primary    ↓
        Secondary
```

The first interface in the SD-WAN action list is the primary.

The primary is preferred when active and its configured performance metrics are within acceptable limits.

---

# 35. 🔙 FAILBACK

Failback happens after the original/preferred WAN becomes healthy again.

Three options in the supplied notes:

1. **Immediate**
2. **Gradual**
3. **No Failback**

---

# 36. ⚡ IMMEDIATE FAILBACK

```text
WAN 1 recovers
      ↓
Active + new connections
      ↓
WAN 1
```

Meaning:

> **Everything moves back now.**

Supplied notes state Immediate is the default failback setting for SD-WAN Failover actions.

Potential consideration:

- Existing connections can be terminated/re-established when moved immediately.

---

# 37. 🐢 GRADUAL FAILBACK

```text
WAN 1 recovers

Existing connections → WAN 2
New connections      → WAN 1
```

Meaning:

> **Old sessions stay on backup; new sessions use primary.**

This reduces disruption to active sessions.

---

# 38. 🚫 NO FAILBACK

```text
WAN 1 recovers

Existing → WAN 2
New      → WAN 2
```

The Firebox continues using the failover interface until manual failback is initiated.

Meaning:

> **Primary is back, but don't automatically move traffic back.**

---

# 39. 🔙 FAILBACK COMPARISON

| Failback | Existing connections | New connections | Automatic |
|---|---|---|---|
| Immediate | Move to original WAN | Original WAN | Yes |
| Gradual | Stay on backup | Original WAN | Yes |
| No Failback | Stay on backup | Stay on backup | No |

### Memory trick

```text
Immediate = Everything moves now

Gradual = Old stays, new moves

No Failback = Nothing moves automatically
```

### Important distinction

Supplied notes state the failback option applies to **Failover** configurations.

For Global Multi-WAN:

```text
Failover          → Failback applicable
Routing Table     → N/A
Round Robin       → N/A
Interface Overflow→ N/A
```

---

# 40. 🧠 COMPLETE PATH-SELECTION MENTAL MODEL

For normal outbound traffic:

```text
Incoming?
   |
   +-- YES → Multi-WAN does not control it
   |
   NO
   ↓
VPN/BOVPN?
   |
   +-- Special VPN routing behavior
   |
   Normal outbound traffic
   ↓
Policy has SD-WAN?
   |
   +-- YES → SD-WAN decision
   |
   NO
   ↓
General Multi-WAN
   ↓
Failover / ECMP / Round Robin / Overflow
```

---

# 41. 🧪 DUAL-WAN LAB

Topology:

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
Deactivate     = 3 failures
Reactivate     = 3 successes

Multi-WAN:
Primary = WAN 1
Backup  = WAN 2
```

### Test

1. Disconnect ISP 1 or simulate WAN failure.
2. Confirm Link Monitor probes fail.
3. Confirm WAN 1 becomes inactive.
4. Confirm traffic uses WAN 2.
5. Restore ISP 1.
6. Confirm successful probes.
7. Observe configured failback behavior.

---

# 42. 🧪 VLAN LAB

Topology:

```text
                         INTERNET
                            |
                         FIREBOX
                            |
                     VLAN Trunk / Port 3
                            |
                      Managed Switch
                    _____|_____|_____
                   /       |       \
               VLAN 10   VLAN 20   VLAN 30
                Users     Servers    Guest
```

Example:

| VLAN | Purpose | Zone | Gateway |
|---|---|---|---|
| 10 | Users | Trusted | `192.168.10.1/24` |
| 20 | Servers | Trusted | `192.168.20.1/24` |
| 30 | Guest | Optional | `192.168.30.1/24` |

Example policy design:

```text
Users  → Internet   ALLOW
Users  → Servers    ALLOW required services
Guest  → Internet   ALLOW
Guest  → Users      DENY
Guest  → Servers    DENY
```

Use actual organizational requirements when designing policies.

---

# 43. 🧪 OPTIONAL / DMZ LAB

Trusted:

```text
10.10.10.0/24
Firebox = 10.10.10.254
```

Optional/DMZ:

```text
192.168.10.0/24
Firebox = 192.168.10.254
```

Server:

```text
192.168.10.10
Gateway = 192.168.10.254
```

Test:

```text
Trusted → Internet
Trusted → DMZ
DMZ → Trusted
Internet → DMZ
Internet → Trusted
```

For Internet → DMZ publishing, configure the appropriate NAT and policy.

---

# 44. 🔍 TROUBLESHOOTING — VLAN

## Client gets no DHCP

Check:

```text
[ ] VLAN exists
[ ] VLAN ID correct
[ ] Gateway IP correct
[ ] DHCP / relay configured
[ ] Switch access port correct
[ ] Trunk carries VLAN
[ ] Firebox interface configured as VLAN
[ ] Tagged/untagged settings match
```

## Client has IP but cannot reach gateway

Trace:

```text
Client
 ↓
Access Port
 ↓
Switch VLAN
 ↓
Trunk
 ↓
Firebox VLAN
 ↓
Gateway
```

Check:

```text
[ ] VLAN ID
[ ] Tagging
[ ] Trunk allowed VLANs
[ ] Subnet
[ ] Gateway
[ ] Interface assignment
```

## One VLAN works, another does not

Compare:

```text
VLAN ID
Security Zone
Gateway IP
Switch VLAN
Trunk allowed VLANs
Tagged/Untagged
DHCP
Firewall Policy
```

---

# 45. 🔍 TROUBLESHOOTING — LINK MONITOR

If interface is physically UP but Link Monitor says inactive:

```text
[ ] Target reachable
[ ] Target not blocking probe
[ ] Correct routing
[ ] DNS works for DNS target
[ ] TCP port reachable for TCP target
[ ] Correct next hop for internal interfaces
[ ] Probe interval
[ ] Failure threshold
[ ] Multiple-target settings
```

If failover does not occur:

```text
[ ] Link Monitor configured
[ ] Correct WAN interfaces selected
[ ] Targets work normally
[ ] Targets fail during WAN outage
[ ] Multi-WAN mode configured
[ ] Primary/backup order correct
[ ] Failure threshold appropriate
[ ] Routing/policy configuration correct
```

---

# 46. 🔍 TROUBLESHOOTING — FIREBOX INTERFACE

When an interface does not work:

### 1. Physical link

```text
Firebox port
    ↓
Cable
    ↓
Switch port
```

### 2. Interface type

Check:

```text
External?
Trusted?
Optional?
Custom?
VLAN?
Bridge?
Disabled?
```

### 3. IP/subnet

Example:

```text
Firebox = 10.10.10.254/24
PC      = 10.10.10.10/24
```

### 4. Default gateway

Host should normally use the Firebox interface IP when Firebox is the router.

### 5. VLAN tagging

```text
Firebox tagged
      ↕
Switch tagged
```

### 6. Firewall policy

Correct IP configuration does not automatically mean Firebox will allow traffic.

### 7. Aliases

Check:

```text
Any-Trusted
Any-Optional
Any-External
```

or a specific/custom interface.

---

# 47. ⚠️ IMPORTANT VLAN NOTES

From the supplied notes:

- VLANs cannot be used when Firebox is in **Drop-In Mode**.
- Maximum VLAN count depends on the Firebox feature key.
- WatchGuard recommends not creating more than 10 VLANs operating on external interfaces because too many can affect performance.
- VLANs use IEEE 802.1Q tagging.
- WatchGuard recommends using a VLAN ID other than **1** for networks passing traffic to the Firebox and not using VLAN 1 for user, server, or management networks.
- Current documentation has model/Fireware-version-specific reserved VLAN IDs, so check the exact model/version before using VLAN IDs near the top of the range.

---

# 48. 🧩 NAT + LINK MONITOR + MULTI-WAN

These solve different problems.

### NAT

> **How should addresses be translated?**

### Link Monitor

> **Is this network path actually usable?**

### Multi-WAN

> **Which available path should carry traffic?**

Combined:

```text
LAN
 |
Dynamic NAT
 |
WAN 1
 |
Link Monitor
 |
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

# 49. 🎯 COMMON BEGINNER MISTAKES

### Mistake 1
**"Optional means Internet."**

❌ No.

```text
External = WAN/Internet side
Optional = internal mixed-trust/DMZ
```

### Mistake 2
**"Any-Optional means Internet access."**

❌ No. It is an alias.

Policies determine access.

### Mistake 3
**"A VLAN is a physical port."**

❌ No.

VLAN is logical Layer-2 segmentation.

### Mistake 4
**"A trunk carries one VLAN."**

❌ Normally a trunk carries multiple VLANs.

### Mistake 5
**"LAG doubles one connection's speed."**

❌ Not necessarily.

It provides cumulative capacity across traffic flows and/or redundancy.

### Mistake 6
**"Physical link UP means Internet is working."**

❌ No.

Use Link Monitor for usable end-to-end connectivity.

### Mistake 7
**"Multi-WAN controls inbound traffic."**

❌ No.

Supplied notes state it applies to outbound non-VPN traffic.

### Mistake 8
**"General Multi-WAN always wins."**

❌ Not for traffic matched by a policy-level SD-WAN routing action.

---

# 50. 🎤 INTERVIEW Q&A — NETWORK MODES & INTERFACES

### Q1. What are the three Firebox network modes?

**A:** Mixed Routing, Drop-In and Bridge.

### Q2. Which is the default?

**A:** Mixed Routing.

### Q3. What is Mixed Routing?

**A:** Normal routed firewall mode with separate subnets and broad networking flexibility.

### Q4. What is Drop-In Mode?

**A:** A mode for inserting Firebox into an existing IP network while hosts can retain their addressing.

### Q5. What is Bridge Mode?

**A:** Transparent mode for filtering/managing traffic between an existing network and its gateway.

### Q6. What is an Optional Interface?

**A:** An internal interface for a separate mixed-trust/DMZ network.

### Q7. Is Optional the same as External?

**A:** No. External is normally WAN/outside; Optional is internal.

### Q8. What alias represents Optional interfaces?

**A:** `Any-Optional`.

### Q9. What is a Custom Interface?

**A:** A separate security zone requiring explicit policy control and not automatically included in the predefined aliases.

### Q10. What is a Bridge interface?

**A:** A logical interface combining multiple interfaces into one network, with Layer-2-style behavior.

### Q11. What is a VLAN interface?

**A:** A logical interface using VLAN tagging/untagging for network segmentation.

### Q12. What is Link Aggregation?

**A:** Multiple physical interfaces operating as one logical interface for cumulative throughput and/or redundancy.

### Q13. What is LACP?

**A:** Link Aggregation Control Protocol used for dynamic link aggregation.

### Q14. Does LAG automatically double one connection's speed?

**A:** No. Individual-flow behavior depends on the aggregation/hash mechanism.

---

# 51. 🎤 INTERVIEW Q&A — VLAN

### Q1. What VLAN standard does WatchGuard use?

**A:** IEEE 802.1Q.

### Q2. Difference between VLAN ID and gateway IP?

**A:**

```text
VLAN ID    = Layer-2 identifier
Gateway IP = Firebox Layer-3 address
```

### Q3. What is tagged traffic?

**A:** Ethernet traffic carrying an IEEE 802.1Q VLAN tag.

### Q4. What is untagged traffic?

**A:** Traffic arriving without an 802.1Q tag for that VLAN.

### Q5. What is a trunk?

**A:** A link carrying multiple VLANs, normally using VLAN tags.

### Q6. Can one Firebox interface carry multiple VLANs?

**A:** Yes, a Firebox interface can carry multiple tagged VLANs and function as a VLAN trunk.

### Q7. Can Firebox provide DHCP for a VLAN?

**A:** Yes, for supported internal VLAN security zones; DHCP Relay can also be used.

### Q8. Can VLANs be used in Drop-In Mode?

**A:** No, according to the supplied notes.

### Q9. Does creating a VLAN automatically allow inter-VLAN traffic?

**A:** No. Fireware policies control traffic between networks.

---

# 52. 🎤 INTERVIEW Q&A — NAT

### Q1. What are the three NAT types?

**A:** Dynamic NAT, Static NAT/SNAT/Port Forwarding, and 1-to-1 NAT.

### Q2. What is Dynamic NAT?

**A:** Changes the source IP of outgoing connections to a public IP.

### Q3. What is Static NAT?

**A:** Forwards a public IP/port to an internal IP/port.

### Q4. What is 1-to-1 NAT?

**A:** Maps IP addresses between networks, potentially for a host, range or subnet.

### Q5. Which has precedence: Dynamic NAT or 1-to-1 NAT?

**A:** 1-to-1 NAT.

---

# 53. 🎤 INTERVIEW Q&A — LINK MONITOR

### Q1. What is Link Monitor?

**A:** A Firebox feature that probes remote targets through an interface to determine whether the interface has usable connectivity.

### Q2. Probe types?

**A:** Ping, TCP and DNS.

### Q3. How many targets?

**A:** Up to three per interface in the supplied current Fireware notes.

### Q4. Default probe interval?

**A:** 5 seconds.

### Q5. Default deactivate threshold?

**A:** 3 consecutive failures.

### Q6. Default reactivate threshold?

**A:** 3 consecutive successful probes.

### Q7. Why is Link Monitor important for Multi-WAN?

**A:** It helps the Firebox determine when an interface is inactive so Multi-WAN can fail over or remove it from path selection according to configuration.

---

# 54. 🎤 INTERVIEW Q&A — MULTI-WAN / SD-WAN

### Q1. What are the four Multi-WAN methods?

**A:** Failover, Routing Table, Round Robin and Interface Overflow.

### Q2. What is the current default Multi-WAN method in the supplied notes?

**A:** Failover in Fireware v12.5.4 and later.

### Q3. What is ECMP?

**A:** Equal-Cost Multi-Path; it allows multiple equal-cost routes to a destination.

### Q4. Does ECMP consider current WAN bandwidth/load?

**A:** The supplied notes state no; the algorithm does not use current interface traffic load as its decision factor.

### Q5. What is Round Robin?

**A:** A load-distribution method that distributes traffic across external interfaces using connection/traffic distribution and relative weights.

### Q6. What is Interface Overflow?

**A:** Starts with one WAN and moves new connections to the next configured WAN when the current interface reaches its threshold.

### Q7. What happens when all Interface Overflow WANs reach their thresholds?

**A:** The supplied notes state the Firebox uses ECMP.

### Q8. What takes precedence: policy SD-WAN or global Multi-WAN?

**A:** Policy SD-WAN routing settings override general Multi-WAN for matching connections.

### Q9. What are the SD-WAN quality metrics?

**A:** Loss, latency and jitter.

### Q10. What are the three failback options?

**A:** Immediate, Gradual and No Failback.

### Q11. What does Immediate Failback do?

**A:** Active and new connections move to the original/failback interface.

### Q12. What does Gradual Failback do?

**A:** Existing connections stay on the failover interface; new connections use the original interface.

### Q13. What does No Failback do?

**A:** Existing and new connections remain on the failover interface until manual failback.

### Q14. Does Multi-WAN control inbound traffic?

**A:** No.

### Q15. Does Multi-WAN control VPN/BOVPN traffic?

**A:** The supplied notes state Multi-WAN does not apply to VPN/BOVPN traffic; VPN failover is configured separately.

---

# 55. 🧠 ONE-MINUTE REVISION

```text
===============================
FIREBOX NETWORK MODES
===============================

Mixed Routing
→ Default
→ Separate subnets
→ Routing + NAT
→ Most flexible

Drop-In
→ Existing network
→ Retain host addressing
→ More limitations

Bridge
→ Transparent
→ Layer-2 style inspection
→ No normal routing/NAT
```

```text
===============================
INTERFACES
===============================

External
→ WAN / outside
→ Any-External

Trusted
→ Main LAN
→ Any-Trusted

Optional
→ DMZ / mixed trust
→ Any-Optional

Custom
→ Separate security zone
→ Explicit policies

Bridge
→ Layer 2
→ Multiple interfaces / one network

VLAN
→ Logical segmentation
→ 802.1Q

LAG
→ Multiple physical links
→ One logical interface
→ Throughput + redundancy
```

```text
===============================
NAT
===============================

Dynamic
→ Outbound
→ Source translation

Static
→ Port forwarding
→ Public IP:Port → Private IP:Port

1-to-1
→ Public IP ↔ Private IP
→ Takes precedence over Dynamic NAT
```

```text
===============================
LINK MONITOR
===============================

Ping / TCP / DNS
→ Tests usable connectivity
→ Up to 3 targets/interface
→ 5 sec interval
→ 3 failures = inactive
→ 3 successes = active
→ Supports Multi-WAN / SD-WAN
```

```text
===============================
MULTI-WAN
===============================

Failover
→ Primary / Backup

Routing Table
→ ECMP
→ Equal-cost paths

Round Robin
→ Distribution
→ Relative weights

Interface Overflow
→ Bandwidth threshold
→ Move NEW connections

SD-WAN
→ Policy-level path decision
→ Loss / Latency / Jitter
```

```text
===============================
FAILBACK
===============================

Immediate
→ Everything moves now

Gradual
→ Existing stays
→ New moves

No Failback
→ Nothing moves automatically
```

---

# 56. 🚀 FINAL MASTER MENTAL MODEL

```text
                    FIREBOX
                       |
        +--------------+--------------+
        |              |              |
      NETWORK       INTERFACES      WANs
       MODE             |              |
        |          +----+----+         |
   +----+----+     |    |    |         |
   |    |    |   Trusted Optional   Multi-WAN
 Mixed Drop Bridge External Custom       |
 Routing In     |      |                 |
                VLAN   LAG            Link Monitor
                 |      |                 |
                 +------+-----------------+
                        |
                       NAT
                        |
                    SD-WAN
                        |
                     POLICY
                        |
                 ALLOW / DENY
```

## The five questions to ask when troubleshooting

```text
1. WHAT NETWORK MODE IS THE FIREBOX USING?

2. WHICH INTERFACE / SECURITY ZONE IS INVOLVED?

3. WHAT IS THE IP/VLAN/ROUTING PATH?

4. IS NAT OR LINK MONITOR AFFECTING THE FLOW?

5. WHICH MULTI-WAN / SD-WAN / FIREWALL POLICY
   DECIDES THE FINAL PATH?
```

## Golden interview lines

> **Mixed Routing = normal routed firewall.**

> **Optional = internal mixed-trust/DMZ, not Internet.**

> **VLAN ID is Layer 2; gateway IP is Layer 3.**

> **Trunk = multiple VLANs, normally tagged.**

> **LAG increases cumulative capacity/redundancy; it does not automatically double one flow.**

> **Dynamic NAT = outbound source translation.**

> **Static NAT = port forwarding.**

> **1-to-1 NAT = IP-to-IP mapping and takes precedence over Dynamic NAT.**

> **Physical link UP does not prove end-to-end connectivity.**

> **Link Monitor determines whether a path is usable.**

> **Multi-WAN chooses among WAN paths for applicable outbound traffic.**

> **Policy-level SD-WAN routing overrides general Multi-WAN for matching traffic.**

> **Immediate failback = everything moves back.**

> **Gradual failback = existing stays, new moves.**

> **No Failback = stay on backup until manual failback.**

---

# 57. 📚 SOURCE TOPICS COVERED

This cheat sheet consolidates the uploaded notes on:

- Firebox Network Modes
- Mixed Routing
- Drop-In Mode
- Bridge Mode
- External / Trusted / Optional / Custom interfaces
- Interface aliases
- VLANs
- 802.1Q
- Tagged / Untagged traffic
- Trunks
- DHCP / DHCP Relay
- Link Aggregation
- LACP
- Dynamic / Static / 1-to-1 NAT
- Link Monitor
- Ping / TCP / DNS probes
- Multi-WAN
- Failover
- Routing Table / ECMP
- Round Robin
- Interface Overflow
- SD-WAN
- Loss / Latency / Jitter
- SD-WAN precedence
- Immediate / Gradual / No Failback
- Practical labs
- Troubleshooting
- Interview questions

---

## Official WatchGuard references listed in the supplied notes

- Network Modes / Interfaces
- Mixed Routing Mode
- Drop-In Mode
- Bridge Mode
- Common Interface Settings
- Trusted / Optional Interfaces
- Custom Interface
- VLANs / Define VLAN
- Link Aggregation
- NAT / Dynamic NAT / Static NAT / 1-to-1 NAT
- Link Monitor
- Multi-WAN
- Routing Table / ECMP
- Failover Multi-WAN
- Interface Overflow
- SD-WAN Routing / Methods / Status & Manual Failback

> **Study note:** For version- or model-specific behavior, use the current WatchGuard documentation for the exact Fireware version and Firebox model in your lab.
