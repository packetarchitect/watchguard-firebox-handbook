# Networking on the Firebox --- Firebox Interfaces

> **WatchGuard Firebox / Fireware Training Handbook**
>
> This chapter follows the supplied WatchGuard training slides and
> expands them with practical explanations, current WatchGuard
> documentation, examples, and lab exercises.

------------------------------------------------------------------------

## 1. Learning Objectives

By the end of this chapter, you should understand:

-   What a **Firebox interface** is.
-   The difference between a **physical interface** and a logical
    interface.
-   The four primary security-zone types:
    -   External
    -   Trusted
    -   Optional
    -   Custom
-   What **Bridge** interfaces do.
-   What **VLAN** interfaces do.
-   What **Link Aggregation** does.
-   What interface aliases such as `Any-External`, `Any-Trusted`, and
    `Any-Optional` mean.
-   How VLAN tagging and trunks relate to Firebox interfaces.
-   How Multi-WAN can use External interfaces.
-   How to choose the correct interface type for a real deployment.

------------------------------------------------------------------------

# 2. What Is a Firebox Interface?

A Firebox interface is a network connection or logical network object
through which the Firebox sends and receives traffic.

An interface can be:

``` text
Physical
   |
   +-- Ethernet port

Logical
   |
   +-- VLAN
   +-- Bridge
   +-- Link Aggregation
```

WatchGuard currently allows interfaces to be configured as **External,
Trusted, Optional, Custom, Bridge, VLAN, Link Aggregation, or
Disabled**, depending on the interface and Fireware configuration.

Official reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interfaces_config_c.html

------------------------------------------------------------------------

# 3. Physical Interface vs Logical Interface

This distinction is extremely important.

## Physical interface

A physical interface corresponds to an actual network port on the
Firebox.

Example:

``` text
Firebox
+--------------------------------+
|                                |
|  eth0  eth1  eth2  eth3  eth4 |
|   |     |     |     |     |   |
+---|-----|-----|-----|-----|---+
    |     |     |     |     |
   WAN   LAN   LAN   LAN   LAN
```

The physical Ethernet port is the hardware connection.

## Logical interface

A logical interface is created in software from one or more physical
interfaces.

Examples:

-   VLAN
-   Bridge
-   Link Aggregation

``` text
Physical ports
      |
      +---------> VLAN interface
      |
      +---------> Bridge interface
      |
      +---------> Link Aggregation
```

This is why the term **interface** does not always mean "one physical
Ethernet port."

------------------------------------------------------------------------

# 4. The Four Main Firebox Security Zones

The supplied lesson identifies four primary interface/security-zone
types:

``` text
                 FIREBOX
                    |
      +-------------+-------------+
      |             |             |
      v             v             v
   External       Trusted       Optional
                                    |
                                    |
                                  Custom
```

More accurately, these are four **security-zone/interface types**:

1.  **External**
2.  **Trusted**
3.  **Optional**
4.  **Custom**

The security zone affects how Firebox policies and aliases treat
traffic.

------------------------------------------------------------------------

# 5. External Interface

The **External** interface normally connects the Firebox to a network
outside the organization.

Typical example:

``` text
              INTERNET
                  |
            ISP Router/ONT
                  |
                  |
            External
          203.0.113.2/24
                  |
              FIREBOX
```

WatchGuard documents that an External interface normally has a **default
route (`0.0.0.0/0`)**.

It is also a member of the built-in:

``` text
Any-External
```

### Why is the default route important?

The default route tells the Firebox:

> "If I do not have a more specific route for this destination, send the
> traffic toward this external interface."

Example:

``` text
Destination:
8.8.8.8

No specific route exists
        |
        v
0.0.0.0/0
        |
        v
External interface
        |
        v
Internet
```

WatchGuard reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interfaces_config_c.html

------------------------------------------------------------------------

# 6. Multi-WAN and External Interfaces

A Firebox can have multiple External interfaces.

Example:

``` text
              INTERNET
              /      \
             /        \
        ISP-1          ISP-2
          |              |
      External 0      External 1
          \              /
           \            /
             FIREBOX
```

This is the basis for a **Multi-WAN** design.

Typical reasons include:

-   ISP redundancy
-   Load balancing
-   Better availability
-   Multiple Internet circuits
-   Failover

The supplied slide shows the concept visually with multiple external
connections.

### Important

Multiple external interfaces do not automatically mean that traffic will
use both ISPs in the exact way you expect. Multi-WAN behavior depends on
Firebox configuration and the selected routing/load-balancing/failover
behavior.

------------------------------------------------------------------------

# 7. Trusted Interface

A **Trusted Interface** connects the Firebox to a private internal
network.

Typical examples:

-   Employee LAN
-   Server LAN
-   Internal management network
-   Corporate wired network

Example:

``` text
             FIREBOX
                |
             Trusted
          10.10.10.254/24
                |
             Switch
          /     |      \
        PC      PC     Server
```

Trusted interfaces are members of:

``` text
Any-Trusted
```

This alias is extremely important when building policies.

For example:

``` text
Source:
Any-Trusted

Destination:
Any-External
```

means:

> Traffic originating from interfaces in the Trusted zone is being
> matched as the source.

WatchGuard reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interface_trusted_opt_c.html

------------------------------------------------------------------------

# 8. Optional Interface

An **Optional Interface** connects the Firebox to a separate internal
network with a different/mixed trust level.

The most common example is a **DMZ**.

``` text
                  FIREBOX
                 /       \
                /         \
          Trusted         Optional
             |               |
             |               |
        Employee LAN        DMZ
                           Servers
```

Optional interfaces are members of:

``` text
Any-Optional
```

Typical uses:

-   Public web servers
-   Mail servers
-   FTP/SFTP servers
-   Systems that need separation from the main LAN
-   Mixed-trust networks

### Remember

``` text
External  = outside/WAN

Trusted   = main internal network

Optional  = internal mixed-trust/DMZ network
```

**Optional does not mean Internet.**

WatchGuard reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interface_trusted_opt_c.html

------------------------------------------------------------------------

# 9. Custom Interface

A **Custom Interface** creates a separate security zone that is not
automatically part of the predefined Trusted, Optional, or External
aliases.

This is useful when you need a security zone with more specific policy
control.

Example:

``` text
                  FIREBOX
                 /   |    \
                /    |     \
          Trusted Optional Custom
             |       |        |
           Users     DMZ    Guest Wi-Fi
```

A Custom interface is **not** automatically included in:

``` text
Any-Trusted
Any-Optional
Any-External
```

Therefore, traffic to or from the Custom zone requires explicit
policies.

This is one of the most important differences between Custom and the
predefined zones.

WatchGuard reference:

https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/interface_custom_c.html

------------------------------------------------------------------------

# 10. Why Use a Custom Interface?

Imagine an organization with:

``` text
Trusted
   |
Corporate users

Optional
   |
Public servers

Custom
   |
Guest Wi-Fi
```

You may want guest users to:

-   Access the Internet
-   Reach DNS/DHCP
-   Be isolated from employees
-   Be isolated from servers

A Custom security zone makes this separation explicit.

The firewall policy can then be written specifically for that zone.

------------------------------------------------------------------------

# 11. Interface Aliases

WatchGuard uses interface aliases extensively in policies.

The most important built-in aliases are:

  Interface type   Built-in alias
  ---------------- ------------------------------
  External         `Any-External`
  Trusted          `Any-Trusted`
  Optional         `Any-Optional`
  Custom           No built-in Any-Custom alias

The purpose of an alias is to allow a policy to refer to a group of
interfaces instead of one physical interface.

Example:

``` text
Any-Trusted
```

could represent multiple Trusted interfaces.

``` text
Any-Optional
```

could represent multiple Optional/DMZ interfaces.

------------------------------------------------------------------------

# 12. Why Aliases Matter

Imagine you have:

``` text
Trusted interface 0
Trusted interface 1
Trusted VLAN 10
Trusted VLAN 20
```

Instead of writing separate policies for every interface, a policy can
use:

``` text
Any-Trusted
```

Conceptually:

``` text
                Any-Trusted
                     |
       +-------------+-------------+
       |             |             |
    Trusted-0     Trusted-1      VLANs
```

This makes policies easier to manage.

------------------------------------------------------------------------

# 13. Security Zone vs Physical Port

A common beginner mistake is to think:

> "Port 1 is Trusted."

That is not necessarily true.

The physical port is hardware.

The interface type is configuration.

For example:

``` text
Physical Port 1
       |
       +---- configured as Trusted
```

But another deployment could configure a different port as:

``` text
Physical Port 1
       |
       +---- configured as Optional
```

So always distinguish:

``` text
PHYSICAL PORT
      ↓
INTERFACE CONFIGURATION
      ↓
SECURITY ZONE
      ↓
FIREWALL POLICY
```

------------------------------------------------------------------------

# 14. Bridge Interface

A **Bridge** logically combines multiple interfaces into a single
network.

Example:

``` text
             FIREBOX BRIDGE
                  |
        +---------+---------+
        |                   |
        v                   v
     Port 1               Port 2
        |                   |
     Network 1           Network 1
```

Both physical interfaces belong to the same logical network.

The bridge has:

-   One interface name
-   One IP address
-   Multiple member interfaces

WatchGuard describes a LAN bridge as a Layer-2 style interface that
combines multiple interfaces into one logical network.

Official reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/net_config_bridge_about_c.html

------------------------------------------------------------------------

# 15. Bridge = Think Layer 2

The easiest way to remember Bridge is:

``` text
Bridge
  =
Layer 2
  =
Same network
```

Example:

``` text
10.10.10.0/24

       FIREBOX
       /     \
   Port 1   Port 2
      |        |
     PC       PC
```

Both sides belong to the same network.

They do not need separate IP subnets simply because they use different
physical ports.

------------------------------------------------------------------------

# 16. Intra-Bridge Traffic

There is an important Fireware version detail.

In Fireware **v12.7 and higher**, you can configure firewall policies to
apply to traffic that passes between bridge member interfaces.

This is called:

**Intra-bridge traffic**

Without such configuration, bridge traffic can operate as Layer-2
forwarding.

WatchGuard reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/net_config_bridge_about_c.html

------------------------------------------------------------------------

# 17. VLAN Interface

A **VLAN interface** allows the Firebox to work with IEEE 802.1Q VLAN
tagging.

Example:

``` text
                    FIREBOX
                       |
                  VLAN trunk
                       |
                    Switch
                /      |      \
             VLAN 10 VLAN 20 VLAN 30
                |       |       |
              Users   Voice   Guest
```

Instead of requiring one physical Firebox port per network, multiple
logical networks can travel over a trunk.

------------------------------------------------------------------------

# 18. Why VLANs Are Useful

Suppose you have:

``` text
VLAN 10 = Users
VLAN 20 = Voice
VLAN 30 = Guest
VLAN 40 = Servers
```

You can carry them through one physical connection:

``` text
Firebox
   |
   | 802.1Q trunk
   |
Switch
   |
   +-- VLAN 10
   +-- VLAN 20
   +-- VLAN 30
   +-- VLAN 40
```

This saves physical ports and allows logical segmentation.

WatchGuard notes that VLANs can be assigned to Trusted, Optional, or
External security zones, depending on the Firebox configuration.

Official reference:

https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/vlans_about_c.html

------------------------------------------------------------------------

# 19. Tagged vs Untagged VLAN Traffic

This is a very important networking concept.

### Tagged

The Ethernet frame carries a VLAN identifier.

Example:

``` text
Frame
+----------------------------+
| Ethernet | VLAN 20 | Data |
+----------------------------+
```

The switch can identify:

> "This frame belongs to VLAN 20."

### Untagged

The frame does not contain a VLAN tag.

``` text
Frame
+---------------------+
| Ethernet | Data     |
+---------------------+
```

The receiving port determines which network the traffic belongs to.

------------------------------------------------------------------------

# 20. Trunk Port

A trunk port normally carries traffic for multiple VLANs.

Example:

``` text
Firebox
   |
   | VLAN 10 tagged
   | VLAN 20 tagged
   | VLAN 30 tagged
   |
   v
Switch
```

Think:

> **Trunk = multiple VLANs over one link.**

This is why the supplied VLAN slide shows multiple VLANs travelling
through a trunk interface.

WatchGuard documentation confirms that Firebox VLAN interfaces can be
configured for tagged or untagged traffic on selected interfaces.

Official reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/vlan_define_new_c.html

------------------------------------------------------------------------

# 21. Access Port vs Trunk Port

Although the exact terminology can vary by switch vendor, the common
concept is:

### Access-style port

Normally carries one VLAN untagged.

``` text
PC
 |
 | untagged
 |
Switch
 |
VLAN 10
```

### Trunk

Carries multiple VLANs, usually tagged.

``` text
Switch
   |
   | VLAN 10
   | VLAN 20
   | VLAN 30
   |
Firebox
```

This is the same concept you have worked with on managed switches.

------------------------------------------------------------------------

# 22. VLAN Security Zones

A VLAN is not itself automatically "Trusted" or "Optional."

You assign the VLAN to a security zone.

Example:

``` text
VLAN 10
Users
    ↓
Trusted

VLAN 20
Guest
    ↓
Custom

VLAN 30
Servers
    ↓
Optional
```

The security zone determines which policy aliases and security behavior
apply.

WatchGuard documents that VLANs can be assigned to Trusted, Optional,
Custom, or External security zones.

------------------------------------------------------------------------

# 23. VLAN Gateway

When the Firebox is routing a VLAN, the Firebox interface acts as the
gateway for that VLAN.

Example:

``` text
VLAN 10
10.10.10.0/24

Firebox:
10.10.10.1

PC:
10.10.10.50

Default Gateway:
10.10.10.1
```

Traffic leaving VLAN 10 goes to:

``` text
PC
 |
 | Default Gateway
 v
Firebox VLAN interface
 |
 v
Other network
```

WatchGuard's VLAN configuration documentation specifies the VLAN IP
address as the gateway address for computers in that VLAN.

------------------------------------------------------------------------

# 24. Link Aggregation

**Link Aggregation (LA)** groups multiple physical interfaces and makes
them operate as one logical interface.

Example:

``` text
             FIREBOX
            /       \
         Port 2     Port 3
            \       /
             \     /
             Link Aggregation
                    |
                    |
                 Switch
```

Instead of seeing two independent links, the Firebox uses them as one
logical interface.

WatchGuard supports link aggregation according to IEEE 802.1AX/802.3ad.

Official reference:

https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/networksetup/link_aggregation_about_c.html

------------------------------------------------------------------------

# 25. Why Use Link Aggregation?

There are two major reasons.

## 1. More cumulative throughput

Example:

``` text
1 Gbps + 1 Gbps
       =
2 Gbps cumulative capacity
```

But be careful:

> This does **not** normally mean that one individual TCP connection
> magically becomes 2 Gbps.

Traffic distribution depends on the aggregation mode and hashing.

## 2. Redundancy

``` text
       FIREBOX
       /     \
      /       \
   Link 1    Link 2
      \       /
       \     /
        Switch
```

If one physical link fails, another member can continue carrying
traffic, depending on the configured mode.

------------------------------------------------------------------------

# 26. Link Aggregation Modes

WatchGuard documents three modes:

### Dynamic --- 802.3ad

Uses **LACP**.

The connected switch must support and be configured for compatible LACP.

### Static

Uses a static aggregation configuration.

The connected network device must also be configured appropriately.

### Active-backup

Only one member is active at a time.

If the active interface fails, another member can become active.

This mode does not require the switch to support link aggregation.

WatchGuard reference:

https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/networksetup/link_aggregation_about_c.html

------------------------------------------------------------------------

# 27. Important Link Aggregation Limitation

WatchGuard currently documents that Link Aggregation is supported only
when the Firebox is configured in **Mixed Routing Mode**.

Also, not every Firebox model supports it.

Therefore, always check the model and Fireware version before planning a
LAG.

Official reference:

https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/networksetup/link_aggregation_about_c.html

------------------------------------------------------------------------

# 28. Interface Types --- Big Picture

Now combine everything:

``` text
                    FIREBOX INTERFACES
                           |
        +------------------+------------------+
        |                  |                  |
     SECURITY           LOGICAL            AGGREGATED
      ZONES             NETWORKS             LINKS
        |                  |                  |
  +-----+-----+        +---+---+          Link
  |     |     |        |       |        Aggregation
External Trusted     VLAN    Bridge
Optional Custom
```

This is a much better mental model than memorizing a flat list.

------------------------------------------------------------------------

# 29. Practical Network Example

Imagine this company:

``` text
                    INTERNET
                        |
                     ISP 1
                        |
                   External
                        |
                   +---------+
                   | FIREBOX |
                   +---------+
                    |   |   |
                    |   |   |
                    |   |   +------ Optional / DMZ
                    |   |             |
                    |   |          Web Server
                    |   |
                    |   +---------- Custom / Guest
                    |                 |
                    |              Guest Wi-Fi
                    |
                    +-------------- Trusted
                                      |
                                    Users
```

Now add VLANs:

``` text
Trusted
   |
   +-- VLAN 10 Users
   +-- VLAN 20 Voice
   +-- VLAN 30 Management

Optional
   |
   +-- VLAN 40 Servers

Custom
   |
   +-- VLAN 50 Guest
```

This is a realistic enterprise-style design.

------------------------------------------------------------------------

# 30. Example Interface Table

  Interface   Type       Network           Typical Purpose
  ----------- ---------- ----------------- --------------------------
  eth0        External   ISP network       Internet
  eth1        Trusted    10.10.10.0/24     Users
  eth2        Optional   192.168.10.0/24   DMZ
  eth3        Custom     172.16.50.0/24    Guest
  VLAN 10     Trusted    10.10.20.0/24     Voice
  VLAN 20     Custom     10.10.30.0/24     IoT
  bond0       Trusted    10.10.40.0/24     High-availability uplink

The exact interface names and available features depend on the Firebox
model.

------------------------------------------------------------------------

# 31. How Policies See Interfaces

This is where interfaces become security controls.

Example:

``` text
Source:
Any-Trusted

Destination:
Any-External

Service:
HTTP/HTTPS
```

Means:

``` text
Trusted
   |
   v
Firebox Policy
   |
   v
External
```

Another example:

``` text
Source:
Any-External

Destination:
DMZ-Web-Server

Service:
HTTPS
```

This could be used to publish a web server.

The interface/security zone therefore becomes an important part of the
policy decision.

------------------------------------------------------------------------

# 32. Interface Configuration Workflow

From Fireware Web UI, the general workflow is:

``` text
Network
   |
Interfaces
   |
Select interface
   |
Edit / Configure
   |
Choose Interface Type
   |
Configure IP/network settings
   |
Configure DHCP/VLAN/other settings
   |
Save
```

WatchGuard's current documentation confirms this workflow for common
interface configuration.

Official reference:

https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interfaces_config_c.html

------------------------------------------------------------------------

# 33. Example --- Configure a Trusted Interface

Suppose:

``` text
Physical interface:
eth1

Type:
Trusted

Alias:
LAN

IP:
10.10.10.254/24
```

Then:

``` text
Firebox
   |
eth1 / LAN
   |
10.10.10.254/24
   |
Switch
   |
PC
10.10.10.50/24
Gateway:
10.10.10.254
```

The PC now uses the Firebox as its Layer-3 gateway.

------------------------------------------------------------------------

# 34. Example --- Configure an Optional Interface

``` text
Physical interface:
eth2

Type:
Optional

Alias:
DMZ

IP:
192.168.10.254/24
```

Server:

``` text
IP:
192.168.10.10

Gateway:
192.168.10.254
```

Topology:

``` text
Trusted LAN
10.10.10.0/24
      |
      |
   FIREBOX
      |
      |
DMZ / Optional
192.168.10.0/24
      |
      |
192.168.10.10
Web Server
```

------------------------------------------------------------------------

# 35. Example --- VLAN Trunk

Suppose the switch has:

``` text
VLAN 10 = Users
VLAN 20 = Voice
VLAN 30 = Guest
```

The Firebox/switch link is a trunk:

``` text
             Firebox
                |
       VLAN 10/20/30 tagged
                |
             Switch
          /      |      \
      VLAN 10  VLAN 20  VLAN 30
       Users    Voice    Guest
```

Each VLAN can have a different security zone and policy.

------------------------------------------------------------------------

# 36. Example --- Link Aggregation

Suppose you have two 1-Gbps links:

``` text
Firebox eth2 -------- Switch
Firebox eth3 -------- Switch
```

Configure:

``` text
eth2 + eth3
    ↓
  bond0
```

Then:

``` text
bond0
  |
  +---- Logical interface
```

The switch must be configured consistently when using Dynamic/LACP or
Static aggregation.

------------------------------------------------------------------------

# 37. Common Mistakes

## Mistake 1 --- Treating External as just "another LAN"

External is normally the outside/WAN side and normally provides the
default route.

------------------------------------------------------------------------

## Mistake 2 --- Thinking Optional means Internet

Optional is an internal mixed-trust/DMZ zone.

------------------------------------------------------------------------

## Mistake 3 --- Assuming Custom is automatically trusted

It is not.

Custom is a separate security zone and requires explicit policies.

------------------------------------------------------------------------

## Mistake 4 --- Confusing VLAN with VLAN port

A VLAN is a logical Layer-2 segmentation mechanism.

A physical port can carry one or multiple VLANs depending on
configuration.

------------------------------------------------------------------------

## Mistake 5 --- Calling every VLAN link an access port

A trunk normally carries multiple VLANs, commonly tagged.

------------------------------------------------------------------------

## Mistake 6 --- Assuming two LAG links equal one giant link

Aggregation increases **cumulative capacity** and can provide
redundancy, but traffic distribution depends on the aggregation/hash
mechanism.

------------------------------------------------------------------------

## Mistake 7 --- Forgetting switch configuration

For VLANs:

``` text
Firebox VLAN configuration
          +
Switch VLAN configuration
          =
Working VLAN design
```

For LACP:

``` text
Firebox LACP
          +
Switch LACP
          =
Working dynamic aggregation
```

------------------------------------------------------------------------

# 38. Hands-On Lab

## Lab Objective

Build a Firebox with:

-   One Trusted LAN
-   One Optional/DMZ network
-   One VLAN
-   Optional: one Link Aggregation group

### Step 1 --- Trusted

``` text
Interface:
eth1

Type:
Trusted

IP:
10.10.10.254/24
```

PC:

``` text
10.10.10.10/24
Gateway:
10.10.10.254
```

------------------------------------------------------------------------

### Step 2 --- Optional

``` text
Interface:
eth2

Type:
Optional

IP:
192.168.10.254/24
```

Server:

``` text
192.168.10.10/24
Gateway:
192.168.10.254
```

------------------------------------------------------------------------

### Step 3 --- Test routing

From the Trusted PC:

``` text
ping 192.168.10.10
```

Whether this succeeds depends on the configured Firebox policies and
server firewall.

------------------------------------------------------------------------

### Step 4 --- Create a controlled policy

For example:

``` text
Source:
Any-Trusted

Destination:
DMZ Server

Service:
HTTPS
```

Test the connection.

------------------------------------------------------------------------

### Step 5 --- VLAN

Create:

``` text
VLAN ID:
20

Name:
VOICE

Security Zone:
Trusted

Gateway:
10.10.20.254/24
```

Configure the connected switch port as a compatible trunk/tagged
connection.

------------------------------------------------------------------------

# 39. Troubleshooting Checklist

When an interface does not work:

### 1. Check physical link

``` text
Firebox port
    ↓
Cable
    ↓
Switch port
```

### 2. Check interface type

Is it:

``` text
External?
Trusted?
Optional?
Custom?
VLAN?
Bridge?
Disabled?
```

### 3. Check IP/subnet

Example:

``` text
Firebox:
10.10.10.254/24

PC:
10.10.10.10/24
```

### 4. Check default gateway

The host should normally use the Firebox interface IP as its gateway
when the Firebox is the router for that network.

### 5. Check VLAN tagging

``` text
Firebox tagged
        ↕
Switch tagged
```

Must match the intended VLAN design.

### 6. Check firewall policy

A correct IP configuration does not automatically mean the Firebox will
allow the traffic.

### 7. Check aliases

Verify whether the policy uses:

``` text
Any-Trusted
Any-Optional
Any-External
```

or a specific/custom interface.

------------------------------------------------------------------------

# 40. Interview Questions

### Q1. What is a Firebox interface?

A physical or logical network connection through which the Firebox sends
and receives traffic.

### Q2. What are the four primary security-zone types?

-   External
-   Trusted
-   Optional
-   Custom

### Q3. What is External used for?

Normally the WAN/outside network and default route.

### Q4. What is Trusted used for?

Main internal/private networks.

### Q5. What is Optional used for?

Mixed-trust networks, commonly DMZs.

### Q6. What is Custom?

A separate security zone that is not automatically included in
Any-Trusted, Any-Optional, or Any-External.

### Q7. What is Any-Trusted?

A built-in alias representing Trusted interfaces.

### Q8. What is Any-Optional?

A built-in alias representing Optional interfaces.

### Q9. What is a Bridge interface?

A logical interface that combines multiple interfaces into one network.

### Q10. What is a VLAN interface?

A logical interface used to provide network segmentation using VLAN
tagging/untagging.

### Q11. What is Link Aggregation?

A logical interface made from multiple physical interfaces to provide
cumulative throughput and/or redundancy.

### Q12. What is LACP?

Link Aggregation Control Protocol, used for dynamic link aggregation.

### Q13. Does LAG automatically double the speed of one connection?

No. It increases cumulative capacity across traffic flows;
individual-flow behavior depends on the hashing/aggregation mechanism.

### Q14. What is a trunk?

A link designed to carry traffic for multiple VLANs, commonly using VLAN
tags.

------------------------------------------------------------------------

# 41. Quick Revision

``` text
FIREBOX INTERFACES
│
├── External
│   ├── Outside/WAN
│   ├── Default route
│   └── Any-External
│
├── Trusted
│   ├── Internal LAN
│   └── Any-Trusted
│
├── Optional
│   ├── DMZ / mixed trust
│   └── Any-Optional
│
├── Custom
│   ├── Separate security zone
│   └── Explicit policies required
│
├── Bridge
│   ├── Layer 2 style
│   └── Multiple interfaces / one network
│
├── VLAN
│   ├── Logical segmentation
│   └── 802.1Q tagging
│
└── Link Aggregation
    ├── Multiple physical links
    ├── One logical interface
    ├── Throughput
    └── Redundancy
```

------------------------------------------------------------------------

# 42. Key Takeaways

1.  A Firebox interface can be **physical or logical**.
2.  The four main security-zone types are **External, Trusted, Optional,
    and Custom**.
3.  **External** normally connects to the outside/WAN and has the
    default route.
4.  **Trusted** normally connects to the main internal LAN.
5.  **Optional** is commonly used for a DMZ or mixed-trust network.
6.  **Custom** creates a separate security zone and is not automatically
    part of the built-in Trusted/Optional/External aliases.
7.  `Any-External`, `Any-Trusted`, and `Any-Optional` are important
    policy aliases.
8.  A **Bridge** combines multiple interfaces into one logical network.
9.  A **VLAN** provides logical Layer-2 segmentation and can be assigned
    to a security zone.
10. A **trunk** commonly carries multiple tagged VLANs.
11. **Link Aggregation** combines physical interfaces into one logical
    interface for cumulative throughput and/or redundancy.
12. **LACP** is used for dynamic link aggregation.
13. Network interfaces are not just "ports"; they are part of the
    Firebox's security architecture.
14. Always think in this order:

``` text
Physical/Logical Interface
          ↓
Security Zone
          ↓
IP Network
          ↓
Routing
          ↓
Firewall Policy
          ↓
Allowed / Denied Traffic
```

------------------------------------------------------------------------

# 43. Supplied Slide Placement

Use the supplied screenshots in the following positions:

### Slide 1 --- Firebox Interfaces

Place after Section 4.

Shows the four primary types:

-   External
-   Trusted
-   Optional
-   Custom

### Slide 2 --- External

Place after Section 5.

Use it to explain:

-   External connectivity
-   Default route
-   Any-External
-   Multi-WAN concept

### Slide 3 --- Trusted

Place after Section 7.

Use it to show:

-   Private LAN
-   Workstations
-   Internal resources
-   Any-Trusted

### Slide 4 --- Optional

Place after Section 8.

Use it to explain:

-   DMZ
-   Public servers
-   Any-Optional

### Slide 5 --- Custom

Place after Section 9.

Use it to explain:

-   Separate security zone
-   Guest network
-   Explicit policy control

### Slide 6 --- Bridge

Place after Section 14.

Use it to explain:

-   Multiple physical interfaces
-   One logical network
-   Layer-2 behavior
-   Intra-bridge policy

### Slide 7 --- VLAN

Place after Section 17.

Use it to explain:

-   Tagged traffic
-   Untagged traffic
-   Trunk
-   VLAN 10 / VLAN 20
-   Security zones

### Slide 8 --- Link Aggregation

Place after Section 24.

Use it to explain:

-   Multiple physical interfaces
-   One logical interface
-   Throughput
-   Redundancy

### Slide 9 --- Key Takeaways: Zones / VLAN

Place near Section 41.

### Slide 10 --- Key Takeaways: Link Aggregation / Aliases

Place before Section 42.

------------------------------------------------------------------------

# 44. Official WatchGuard References

-   Common Interface Settings\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interfaces_config_c.html

-   Configure a Trusted or Optional Interface\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interface_trusted_opt_c.html

-   Configure a Custom Interface\
    https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/interface_custom_c.html

-   About LAN Bridges\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/net_config_bridge_about_c.html

-   About VLANs\
    https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/vlans_about_c.html

-   Define a New VLAN\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/vlan_define_new_c.html

-   About Link Aggregation\
    https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/networksetup/link_aggregation_about_c.html

-   Configure Link Aggregation\
    https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/networksetup/link_aggregation_config_c.html

------------------------------------------------------------------------

# Final Mental Model

``` text
                         FIREBOX
                            |
        +-------------------+-------------------+
        |                   |                   |
     SECURITY             LOGICAL             LINK
       ZONES             NETWORKS          AGGREGATION
        |                   |                   |
   +----+----+         +----+----+             |
   |    |    |         |         |             |
External Trusted    VLAN      Bridge         LAG
        |
     Optional
        |
      Custom
```

The most important relationship to remember is:

``` text
INTERFACE
    ↓
SECURITY ZONE
    ↓
IP NETWORK
    ↓
POLICY
    ↓
TRAFFIC DECISION
```

> **A Firebox interface is not just a physical port. It is the point
> where the Firebox's physical connectivity, logical network design,
> security zone, IP addressing, routing, and firewall policies come
> together.**
