# WatchGuard Firebox — Multi-WAN Modes, SD-WAN Precedence & Failback

## Module: Networking on the Firebox

---

# 1. What is Multi-WAN?

**Multi-WAN** allows a Firebox to use multiple external/WAN interfaces.

Example:

```text
                    FIREBOX
                   /       \
                  /         \
             WAN 1           WAN 2
             ISP-A           ISP-B
                \             /
                 \           /
                  INTERNET
```

The Firebox can use the WANs for:

- Failover
- Load distribution
- Route-based path selection
- Bandwidth overflow
- SD-WAN path selection

### Important scope

WatchGuard documents that Multi-WAN affects **outbound, non-VPN traffic**. It does not control incoming connections, and Multi-WAN settings do not apply to BOVPN traffic.

Multi-WAN requires **Mixed Routing Mode**; it does not operate in Drop-In or Bridge mode.



---

# 2. Multi-WAN Methods

WatchGuard provides four main Multi-WAN methods:

1. **Failover**
2. **Routing Table**
3. **Round Robin**
4. **Interface Overflow**

In addition, **SD-WAN** can make routing decisions for traffic when an SD-WAN action is applied to a policy.

Current Fireware documentation states that **Failover is the default Multi-WAN option in Fireware v12.5.4 and later**.



---

# 3. Method 1 — Failover

## What is Failover?

Failover uses one WAN as the **primary** connection and the other WANs as backup connections.

Example:

```text
             FIREBOX
             /     \
            /       \
        WAN 1       WAN 2
       PRIMARY      BACKUP
          |
          v
       Internet
```

If WAN 1 becomes inactive:

```text
WAN 1
  ↓
Failure detected
  ↓
WAN 2
  ↓
Internet
```

The Firebox monitors the primary interface and, if it becomes inactive, sends new traffic through the next configured interface.

The order of interfaces determines the failover order.



---

## Failover order

Example:

```text
1. WAN 1
2. WAN 2
3. WAN 3
```

If WAN 1 fails:

```text
WAN 1 ❌
   ↓
WAN 2
```

If WAN 1 and WAN 2 fail:

```text
WAN 1 ❌
WAN 2 ❌
WAN 3 ✅
   ↓
Traffic uses WAN 3
```

The first interface in the Multi-WAN list is the primary interface.

You can change the order by moving interfaces up or down.



---

# 4. Failover and Link Monitor

Failover becomes much more useful when combined with **Link Monitor**.

A physical interface can remain connected while Internet connectivity beyond the ISP gateway is broken.

```text
Firebox
   |
WAN 1 = physically UP
   |
ISP
   X
Internet unavailable
```

Link Monitor can detect the loss of connectivity and mark the interface inactive.

Then Multi-WAN can fail over.

```text
Link Monitor
     ↓
WAN 1 inactive
     ↓
Multi-WAN Failover
     ↓
WAN 2
```

WatchGuard recommends defining Link Monitor targets for participating WAN interfaces.



---

# 5. Method 2 — Routing Table / ECMP

The **Routing Table** method uses the Firebox routing table to determine the outgoing path.

If a specific route exists, the Firebox uses that route.

If no specified route determines the path, the Firebox uses **ECMP — Equal-Cost Multi-Path**.

```text
                FIREBOX
              /    |    \
           WAN 1  WAN 2  WAN 3
              \    |    /
                ECMP
```

The active external interfaces can be placed into an ECMP group when they have equal-cost routes.



---

# 6. What is ECMP?

**ECMP = Equal-Cost Multi-Path**

It allows multiple equal-cost routes to the same destination.

Example:

```text
Destination: Internet

Route 1 → WAN 1 → Cost 5
Route 2 → WAN 2 → Cost 5
Route 3 → WAN 3 → Cost 5
```

Because the paths have equal cost, ECMP can distribute connections across them.

WatchGuard's Routing Table Multi-WAN method uses the ECMP algorithm described in RFC 2992.

The algorithm uses source and destination IP information and does **not** use current interface traffic load as its decision factor.



---

# 7. Routing Table vs Failover

| Feature | Failover | Routing Table / ECMP |
|---|---|---|
| Primary WAN | Yes | No preferred WAN in the same sense |
| Backup WAN | Yes | Multiple active paths |
| Main purpose | Availability | Route/load distribution |
| Decision | Interface order + availability | Route table + ECMP |
| Uses multiple WANs simultaneously | Normally no | Yes |
| Traffic load considered | No | No current byte load |
| Best mental model | Active/standby | Equal-cost paths |

### Easy memory

```text
Failover = "Use WAN 1, then WAN 2 if WAN 1 fails."

ECMP = "WAN 1 and WAN 2 are equal-cost paths; distribute connections."
```

---

# 8. SD-WAN Precedence

This is one of the **most important concepts** in Multi-WAN.

If an outbound policy has an **SD-WAN routing action**, that SD-WAN routing decision takes precedence over the general Multi-WAN configuration for traffic matched by that policy.

In other words:

```text
Policy
  ↓
SD-WAN Action
  ↓
Path decision
```

If the policy does not specify SD-WAN routing:

```text
Policy
  ↓
Multi-WAN configuration
  ↓
Path decision
```

WatchGuard explicitly documents that SD-WAN routing settings in a policy override the Multi-WAN settings for connections to which that policy applies.



---

# 9. SD-WAN Can Use More Than Availability

Traditional Multi-WAN can primarily decide based on interface availability and routing behavior.

SD-WAN can also evaluate network-quality measurements such as:

- **Loss**
- **Latency**
- **Jitter**

Example:

```text
WAN 1
Loss: 2%
Latency: 40 ms
Jitter: 8 ms

WAN 2
Loss: 8%
Latency: 220 ms
Jitter: 70 ms
```

An SD-WAN action can use configured thresholds to determine whether a path is qualified.



---

# 10. SD-WAN Failover

With an SD-WAN **Failover** action:

```text
Primary WAN
     ↓
Healthy?
     |
    YES
     ↓
Use primary

     NO
     ↓
Fail over
     ↓
Secondary WAN
```

The first interface in the SD-WAN action list is the primary interface.

The primary is preferred when it is active and its configured performance metrics are within acceptable limits.



---

# 11. Method 3 — Round Robin

**Round Robin** distributes outbound traffic across multiple external interfaces.

WatchGuard's current documentation describes Round Robin as a load-balancing method.

It can use interface **weights** to influence how traffic is distributed.

Example:

```text
WAN 1 → Weight 3
WAN 2 → Weight 2
```

Conceptually, traffic is distributed in a 3:2 relationship.



---

# 12. Round Robin is not simply "packet 1 to WAN 1, packet 2 to WAN 2"

This is an important clarification.

WatchGuard's Multi-WAN Round Robin behavior is based on connections/traffic distribution rather than blindly alternating every packet.

The algorithm is applied after:

1. Routes
2. Sticky connections
3. SD-WAN routing

have failed to determine the path.



---

# 13. Round Robin weights

Weights are **relative**.

Example:

```text
WAN 1 = Weight 3
WAN 2 = Weight 2
```

Conceptually:

```text
WAN 1 → 60%
WAN 2 → 40%
```

The exact distribution depends on traffic/connection behavior; the weights are not a guarantee of exact bandwidth percentages.

### Example

Suppose:

```text
ISP 1 = 300 Mbps
ISP 2 = 100 Mbps
```

You might choose a relative weight such as:

```text
WAN 1 = 3
WAN 2 = 1
```

This expresses the desired ratio rather than directly setting Mbps.

---

# 14. Method 4 — Interface Overflow

Interface Overflow is used when you want to restrict how much traffic each WAN interface carries.

Example:

```text
WAN 1
Threshold = 100 Mbps

WAN 2
Threshold = 200 Mbps
```

The Firebox starts with WAN 1.

When WAN 1 reaches its configured threshold:

```text
WAN 1
  ↓
Threshold reached
  ↓
New connections
  ↓
WAN 2
```



---

# 15. Interface Overflow order

Example:

```text
1. WAN 1 — 100 Mbps
2. WAN 2 — 200 Mbps
3. WAN 3 — 500 Mbps
```

Traffic starts on:

```text
WAN 1
```

When its threshold is reached:

```text
WAN 1 → Threshold reached
             ↓
           WAN 2
```

When WAN 2 also reaches its threshold:

```text
WAN 1 → Full
WAN 2 → Full
             ↓
           WAN 3
```

The order matters.



---

# 16. How Interface Overflow measures bandwidth

WatchGuard states that the Firebox examines:

- TX — transmitted traffic
- RX — received traffic

It uses the **higher** value when determining whether the threshold has been reached.

Example:

```text
TX = 80 Mbps
RX = 120 Mbps

Measured value = 120 Mbps
```

This is important when your ISP connection is asymmetric.



---

# 17. What happens when every Interface Overflow WAN reaches its limit?

If all WAN interfaces reach their configured bandwidth thresholds, WatchGuard states that the Firebox uses **ECMP** to find the best path.

```text
WAN 1 → Full
WAN 2 → Full
WAN 3 → Full
       ↓
      ECMP
       ↓
Best equal-cost path
```



---

# 18. Multi-WAN Methods — Comparison

| Method | Main purpose | Key decision |
|---|---|---|
| **Failover** | High availability | Primary/backup order |
| **Routing Table / ECMP** | Route-based distribution | Routing table + equal-cost paths |
| **Round Robin** | Load distribution | Connection/traffic distribution + weights |
| **Interface Overflow** | Bandwidth thresholding | Move new connections when threshold is reached |
| **SD-WAN Failover** | Application/path quality | Availability + loss/latency/jitter metrics |
| **SD-WAN Round Robin** | Quality-aware distribution | Participation based on configured metrics |

---

# 19. Failback

**Failback** is what happens after the original/preferred WAN becomes healthy again.

Example:

```text
WAN 1 = Primary
WAN 2 = Backup

WAN 1 fails
    ↓
Traffic moves to WAN 2
    ↓
WAN 1 recovers
    ↓
FAILBACK decision
```

WatchGuard documents three failback options for applicable SD-WAN Failover configurations:

1. **Immediate**
2. **Gradual**
3. **No Failback**



---

# 20. Immediate Failback

With **Immediate Failback**:

```text
WAN 1 recovers
     ↓
Active + new connections
     ↓
WAN 1
```

Active connections are moved back to the original/failback interface.

WatchGuard documents Immediate as the default failback setting for SD-WAN Failover actions.



### Simple meaning

**"WAN 1 is back — move everything back now."**

### Potential consideration

Existing connections can be terminated/re-established when they are moved immediately.

---

# 21. Gradual Failback

With **Gradual Failback**:

```text
WAN 1 recovers
     |
     +---- Existing connections → WAN 2
     |
     +---- New connections ------> WAN 1
```

Existing connections continue on the failover interface.

New connections use the original interface again.



### Simple meaning

**"Let existing sessions finish on WAN 2; send new sessions through WAN 1."**

This reduces disruption to active sessions.

---

# 22. No Failback

With **No Failback**:

```text
WAN 1 recovers
     ↓
Existing connections → WAN 2
New connections      → WAN 2
```

The Firebox continues to use the failover interface.

You can later manually initiate gradual or immediate failback.



### Simple meaning

**"WAN 1 is back, but don't automatically move traffic back."**

This can be useful when you want to confirm that the original WAN problem is truly resolved before returning traffic.

---

# 23. Failback Comparison

| Option | Existing connections | New connections | Automatic? |
|---|---|---|---|
| **Immediate** | Move to original WAN | Original WAN | Yes |
| **Gradual** | Stay on backup WAN | Original WAN | Yes, gradually |
| **No Failback** | Stay on backup WAN | Stay on backup WAN | No |

### Memory trick

```text
Immediate = Everything moves now

Gradual = Old stays, new moves

No Failback = Nothing moves automatically
```

---

# 24. Important distinction: Multi-WAN Failback vs SD-WAN Failback

Do not assume every Multi-WAN method has the same failback controls.

WatchGuard documents that the failback option applies to **Failover** configurations.

For the Global Multi-WAN action:

- **Failover** → failback is applicable.
- **Routing Table** → failback option is N/A.
- **Round Robin** → failback option is N/A.
- **Interface Overflow** → failback option is N/A.



This is an important exam/interview point.

---

# 25. SD-WAN Precedence — Full Mental Model

For a connection, think about path selection in this order:

```text
Incoming traffic?
      |
      +-- YES → Multi-WAN does not control it
      |
      +-- NO
          |
       VPN/BOVPN?
          |
          +-- Special VPN routing behavior
          |
          +-- Normal outbound traffic
                  |
             Policy has SD-WAN?
                  |
             +----YES----+
             |           |
          SD-WAN      SD-WAN action
          decision
             |
             NO
             ↓
        Multi-WAN method
             |
      +------+------+------+------+
      |      |      |      |
   Failover ECMP Round  Interface
             Robin   Overflow
```

For normal outbound traffic covered by a policy, an SD-WAN routing action in that policy overrides the general Multi-WAN configuration.



---

# 26. Example — Two ISPs + SD-WAN

Suppose:

```text
WAN 1 = Fiber
WAN 2 = Broadband
```

General Multi-WAN:

```text
Mode = Failover
WAN 1 = Primary
WAN 2 = Backup
```

Normal Internet traffic:

```text
Users → Internet
        ↓
      WAN 1
```

Now create an SD-WAN action for VoIP:

```text
VoIP Policy
   ↓
SD-WAN
   ↓
WAN 1 + WAN 2
   ↓
Measure:
Loss
Latency
Jitter
```

If WAN 1 becomes unsuitable according to the configured SD-WAN criteria:

```text
VoIP
 ↓
SD-WAN
 ↓
WAN 2
```

Meanwhile, other traffic can continue following the general Multi-WAN configuration.

This is why SD-WAN is more granular than simply configuring a global WAN failover.

---

# 27. Example — Interface Overflow

Assume:

```text
WAN 1 = 100 Mbps
WAN 2 = 300 Mbps
WAN 3 = 500 Mbps
```

Configure:

```text
WAN 1 threshold = 80 Mbps
WAN 2 threshold = 250 Mbps
WAN 3 threshold = 400 Mbps
```

Traffic:

```text
Start
 ↓
WAN 1
 ↓
WAN 1 reaches 80 Mbps
 ↓
WAN 2
 ↓
WAN 2 reaches 250 Mbps
 ↓
WAN 3
```

If all are full:

```text
WAN 1 = Full
WAN 2 = Full
WAN 3 = Full
       ↓
      ECMP
```



---

# 28. Configuration Locations

### Fireware Web UI

Multi-WAN is configured under:

```text
Network
  ↓
Multi-WAN
```

The method can be selected from the Multi-WAN Mode drop-down.

For Interface Overflow, you configure the threshold for each participating interface and arrange the interface order.



### SD-WAN

SD-WAN actions can be configured under the SD-WAN configuration area and then assigned to policies.

For a policy:

```text
Firewall
  ↓
Firewall Policies
  ↓
Select/Edit Policy
  ↓
SD-WAN
  ↓
Select SD-WAN Action
```



---

# 29. Multi-WAN + Link Monitor

These features work closely together.

```text
WAN 1
  ↓
Link Monitor
  ↓
Healthy?
  |
  +-- YES → Available
  |
  +-- NO → Inactive
             ↓
        Multi-WAN decision
```

WatchGuard recommends configuring Link Monitor targets for Multi-WAN interfaces.

For practical deployments, using two reliable monitoring targets per interface is recommended.



---

# 30. Important Multi-WAN Notes

### Multi-WAN does not control inbound traffic

Multi-WAN settings apply to outbound connections, not incoming connections.

### Multi-WAN does not replace VPN failover

VPN/BOVPN failover is configured separately.

### Multi-WAN requires Mixed Routing Mode

It does not operate in Drop-In or Bridge mode.

### At least two interfaces

You need at least two external interfaces participating in Multi-WAN.

### SD-WAN can override Multi-WAN

A policy-level SD-WAN routing action takes precedence for the traffic that matches that policy.



---

# 31. Key Takeaways

The supplied WatchGuard lesson emphasizes these points:

- Multi-WAN affects **outbound, non-VPN traffic only**.
- Verify **interface order** and load-balancing parameters.
- VPN failover options are configured separately from Multi-WAN.

The most important operational concept is:

```text
Multi-WAN
   ↓
General outbound path selection

SD-WAN
   ↓
More granular policy-based path selection
   ↓
Can use loss / latency / jitter
```

---

# 32. Interview Questions

### Q1. What are the four Multi-WAN methods?

Failover, Routing Table, Round Robin, and Interface Overflow.

### Q2. What is the default Multi-WAN method in current Fireware?

Failover in Fireware v12.5.4 and later.

### Q3. What is ECMP?

Equal-Cost Multi-Path. It allows traffic to use multiple equal-cost routes.

### Q4. Does ECMP consider current WAN bandwidth?

No. WatchGuard states that the ECMP algorithm used by the Routing Table method does not consider current traffic load.

### Q5. What is Round Robin?

A load-distribution method that distributes traffic across external interfaces using connection/traffic distribution and configurable relative weights.

### Q6. What is Interface Overflow?

A method that starts with one WAN and moves new connections to the next configured WAN when the current interface reaches its bandwidth threshold.

### Q7. What happens if all Interface Overflow WANs reach their thresholds?

The Firebox uses ECMP to find the best path.

### Q8. What takes precedence: policy SD-WAN or global Multi-WAN?

For connections covered by the policy, the policy's SD-WAN routing settings override the Multi-WAN configuration.

### Q9. What are the three failback options?

Immediate, Gradual, and No Failback.

### Q10. What does Immediate Failback do?

Active and new connections use the original/failback interface.

### Q11. What does Gradual Failback do?

Existing connections stay on the failover interface; new connections use the original interface.

### Q12. What does No Failback do?

Existing and new connections continue to use the failover interface until a manual failback is initiated.

### Q13. Does Multi-WAN control inbound traffic?

No.

### Q14. Does Multi-WAN control VPN traffic?

Multi-WAN does not apply to VPN/BOVPN traffic; VPN failover is configured separately.

---

# 33. Quick Revision Sheet

```text
                    MULTI-WAN
                       |
       +---------------+----------------+
       |               |                |
    Failover      Routing Table     Round Robin
       |               |                |
 Primary/Backup       ECMP          Load distribution
       |               |                |
 Interface order    Equal cost       Weights
```

```text
             Interface Overflow
                    |
             Bandwidth threshold
                    |
          Threshold reached?
              /          \
            NO            YES
            |              |
       Keep WAN 1      New connections
                           ↓
                        WAN 2
```

```text
                  SD-WAN
                    |
        Policy-level routing action
                    |
                    ↓
           Overrides general
             Multi-WAN choice
                    |
           +--------+--------+
           |        |        |
         Loss    Latency   Jitter
```

---

# 34. Failback Cheat Sheet

```text
WAN 1 = Primary
WAN 2 = Backup

WAN 1 FAILS
     ↓
WAN 2 becomes active
     ↓
WAN 1 RECOVERS
     ↓
+-------------------------------+
| IMMEDIATE                     |
| Active + new → WAN 1          |
+-------------------------------+

+-------------------------------+
| GRADUAL                       |
| Existing → WAN 2              |
| New → WAN 1                   |
+-------------------------------+

+-------------------------------+
| NO FAILBACK                   |
| Existing → WAN 2              |
| New → WAN 2                   |
| Manual failback required      |
+-------------------------------+
```

---

# 35. Final Mental Model

### Failover

**"Keep one WAN primary and use another when it fails."**

### Routing Table / ECMP

**"Use equal-cost routes and let ECMP distribute connections."**

### Round Robin

**"Distribute traffic across WANs according to configured weights."**

### Interface Overflow

**"Use WAN 1 until its threshold is reached, then use the next WAN."**

### SD-WAN

**"For selected traffic, choose a path based on availability and/or network-quality metrics."**

### Failback

**"When the preferred WAN returns, decide whether traffic moves back immediately, gradually, or not automatically."**

---

## Official WatchGuard References

- About Multi-WAN  
  https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/Fireware/multiwan/multiwan_about_c.html

- About Multi-WAN Methods  
  https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/Fireware/multiwan/multi_wan_options_c.html

- Routing Table Multi-WAN / ECMP  
  https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/multiwan/routing_table_configure_c.html

- Failover Multi-WAN  
  https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/multiwan/failover_configure_c.html

- Interface Overflow  
  https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/multiwan/int_overflow_config_c.html

- SD-WAN Routing  
  https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/sd-wan/sd_wan_routing_configure.html

- SD-WAN Methods  
  https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/Fireware/sd-wan/sd_wan_routing_methods.html

- SD-WAN Status & Manual Failback  
  https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/system_status/stats_sdwan_web.html
