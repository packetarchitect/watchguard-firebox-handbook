# Locally-Managed Networking --- Network Modes & Optional Interface

## Learning Objectives

Understand: - Mixed Routing, Drop-In, and Bridge Mode - Trusted vs
Optional interfaces - What an Optional Interface is - Why Optional is
commonly used as a DMZ - How network mode affects routing, NAT, and
available features - How to build a simple Optional/DMZ lab

## 1. What is a Firebox Network Mode?

A network mode defines how the Firebox connects its interfaces to
networks and how traffic moves through the device.

``` text
                 FIREBOX NETWORK MODE
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
    Mixed Routing      Drop-In        Bridge
```

WatchGuard documents three primary modes: **Mixed Routing, Drop-In, and
Bridge**.

Official reference:
https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/net_setup_about_c.html

------------------------------------------------------------------------

# 2. Mixed Routing Mode

**Mixed Routing Mode**, also called Routed Mode, is the **default** and
most flexible Firebox network mode.

Each interface normally has its own subnet.

``` text
                    INTERNET
                       |
                 203.0.113.1/24
                       |
                 +-----------+
                 |  Firebox  |
                 +-----------+
                  |         |
                  |         |
       Trusted    |         |    Optional
     10.10.10.0/24          192.168.10.0/24
                  |         |
                Users      Servers
```

Typical interfaces include:

-   External
-   Trusted
-   Optional
-   Custom
-   VLAN
-   Bridge
-   Link Aggregation

In Mixed Routing Mode, interfaces normally use different subnets, except
for specific VLAN/bridged cases.

WatchGuard identifies Mixed Routing as the mode with the greatest
network flexibility.

Official reference:
https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/Fireware/networksetup/net_config_mixedroutingmode_c.html

### Why use it?

It allows strong network segmentation:

``` text
             INTERNET
                 |
              External
                 |
              FIREBOX
             /                   /                Trusted       Optional
       Employees        DMZ
```

This is the design you will encounter most often.

------------------------------------------------------------------------

# 3. Drop-In Mode

Drop-In Mode is designed for an existing network where you want to place
the Firebox into the network without changing host addressing.

The Firebox uses the same primary IP address on its interfaces.

``` text
              Router
          198.51.100.1/24
                 |
            +----------+
            | Firebox  |
            | .2/24    |
            +----------+
              /                   /                  LAN         Server
```

Important characteristics:

-   One logical network is used.
-   Static IP is required on the external interface.
-   Hosts can retain existing IP addresses and gateways.
-   NAT is not needed for typical traffic to public servers.
-   Dynamic routing such as OSPF, BGP, and RIP is not supported.
-   VLAN-tagged traffic cannot be routed.
-   Link aggregation is not supported.

Official reference:
https://www.watchguard.com/help/docs/help-center/en-us/Content/en-us/Fireware/networksetup/net_config_dropin_about_c.html

------------------------------------------------------------------------

# 4. Bridge Mode

Bridge Mode places the Firebox transparently between an existing network
and its gateway.

``` text
Existing Network
       |
       v
   +---------+
   | Firebox |
   | Bridge  |
   +---------+
       |
       v
    Gateway
       |
       v
   Internet
```

The Firebox filters/manages traffic while the traffic appears to the
gateway to come from the original device.

Bridge Mode is useful when you want security inspection without making
the Firebox the normal Layer-3 gateway.

Important limitations include:

-   No normal routing
-   No NAT
-   No VLAN routing
-   The Firebox cannot act as a VPN endpoint in Bridge Mode

Official reference:
https://www.watchguard.com/help/docs/help-center/en-us/Content/en-us/Fireware/networksetup/net_config_bridgemode_c.html

------------------------------------------------------------------------

# 5. Network Mode Comparison

  ------------------------------------------------------------------------
  Feature           Mixed Routing     Drop-In            Bridge
  ----------------- ----------------- ------------------ -----------------
  Default           Yes               No                 No

  Separate subnets  Yes               No                 No normal routed
                                                         interfaces

  Routing           Yes               Limited design     No normal routing

  NAT               Yes               Not normally       No
                                      required           

  VLAN routing      Yes               No                 No

  Dynamic routing   Available         Limited            No

  Transparency      No                Existing-network   Yes
                                      friendly           

  Typical use       Most deployments  Existing network   Transparent
                                                         inspection
  ------------------------------------------------------------------------

Always check the Fireware version and required features before selecting
a mode.

------------------------------------------------------------------------

# 6. What Is an Optional Interface?

This is the key concept.

An **Optional Interface** is an internal Firebox interface used for a
**mixed-trust or DMZ network that is separate from the Trusted
network**.

WatchGuard specifically describes Optional Interfaces as suitable for
DMZ environments and public servers.

Official reference:
https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/net_setup_about_c.html

The easiest mental model is:

> **Optional Interface = a separate internal security zone, commonly
> used as a DMZ.**

------------------------------------------------------------------------

# 7. Optional Interface ≠ Internet

This is one of the most important beginner concepts.

``` text
External
   ↓
Outside / WAN / Internet

Trusted
   ↓
Main internal LAN

Optional
   ↓
Internal mixed-trust / DMZ network
```

The **External** interface normally faces the WAN/Internet.

The **Optional** interface is an **internal** interface.

WatchGuard classifies Trusted and Optional interfaces as internal
interfaces.

Official reference:
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/other/QSW_fb_internal_interfaces_wsm.html

------------------------------------------------------------------------

# 8. Why Use an Optional Interface?

Suppose you have:

-   Employee PCs
-   A public web server
-   A mail server
-   An FTP/SFTP server

Do not automatically put the public-facing systems in the same network
as employee PCs.

Instead:

``` text
             INTERNET
                 |
              FIREBOX
              /                  /              TRUSTED       OPTIONAL
          |             |
      Employees       Servers
                    / DMZ
```

Now you can apply different firewall policies.

Example:

``` text
Internet → Web Server
        ALLOW HTTPS

Internet → Trusted
        DENY

Trusted → Optional
        ALLOW only required administration

Optional → Trusted
        DENY unless explicitly required
```

The exact rules depend on the organization's requirements.

------------------------------------------------------------------------

# 9. Optional Interface = DMZ

A DMZ is a network where systems with a different security exposure are
placed separately from the main internal LAN.

Example:

``` text
Internet
    |
    v
External
    |
 FIREBOX
    |
    +------------------+
    |                  |
    v                  v
Trusted            Optional / DMZ
10.10.10.0/24      192.168.10.0/24
    |                  |
Employees          Web Server
                   Mail Server
                   Public Services
```

If a public web server is compromised, segmentation can reduce the
attacker's direct access to the Trusted network.

**Important:** An Optional interface does not automatically make a
server secure. Firewall policies, NAT, host security, application
security, and patching still matter.

------------------------------------------------------------------------

# 10. Trusted vs Optional

  Characteristic                     Trusted                      Optional
  ---------------------------------- ---------------------------- ------------------------------
  Purpose                            Main internal LAN            Separate/mixed-trust network
  Common use                         Employees/internal systems   DMZ/public servers
  Built-in alias                     Any-Trusted                  Any-Optional
  Typical trust                      Higher                       Different/lower/mixed
  Internal interface                 Yes                          Yes
  Separate subnet in Mixed Routing   Yes                          Yes

WatchGuard states that Trusted interfaces are members of
**Any-Trusted**, while Optional interfaces are members of
**Any-Optional**.

------------------------------------------------------------------------

# 11. The Any-Optional Alias

When an interface is configured as Optional, it becomes part of:

**Any-Optional**

This can be used in firewall policies.

Example:

``` text
Source:
Any-Trusted

Destination:
Any-Optional
```

or:

``` text
Source:
Any-Optional

Destination:
Any-External
```

The alias identifies the Optional security zone; it does **not**
automatically grant Internet access.

------------------------------------------------------------------------

# 12. Optional Interface Example

A practical network:

``` text
Internet
203.0.113.0/24
       |
       |
External
203.0.113.2/24
       |
   +---------+
   | Firebox |
   +---------+
      |   |
      |   |
      v   v
Trusted   Optional
10.10.10.0/24   192.168.10.0/24
      |              |
   Users          Web Server
10.10.10.10     192.168.10.10
```

Possible policy design:

``` text
Internet → DMZ Web Server
HTTPS       ALLOW

Internet → Trusted
            DENY

Trusted → Internet
            ALLOW as required

Trusted → DMZ
            ALLOW only required admin/services

DMZ → Trusted
            DENY unless explicitly required
```

------------------------------------------------------------------------

# 13. Multiple Optional Interfaces

You can have more than one Optional interface.

For example:

``` text
                 FIREBOX
                    |
        +-----------+-----------+
        |           |           |
     Trusted      Optional    Optional
      Users        DMZ          IoT
        |           |           |
      PCs         Servers      Devices
```

Separate interfaces can represent separate networks and can be
controlled with policies.

In Mixed Routing Mode, interfaces of the same type are still separate
networks; communication between them requires appropriate policy
configuration.

------------------------------------------------------------------------

# 14. Optional vs Custom Interface

A **Custom Interface** is useful when you want a security zone that is
not automatically part of Any-Trusted or Any-Optional.

``` text
Trusted
   ↓
Any-Trusted

Optional
   ↓
Any-Optional

Custom
   ↓
Custom security zone
```

This gives you more granular zone design.

Official reference:
https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/interface_custom_c.html

------------------------------------------------------------------------

# 15. How to Configure an Optional Interface

From Fireware Web UI:

1.  Go to **Network → Interfaces**.
2.  Select the physical interface.
3.  Select **Configure/Edit**.
4.  Set **Interface Type = Optional**.
5.  Give it a meaningful alias, such as `DMZ`.
6.  Assign an IPv4 address using slash notation.
7.  Configure DHCP if required.
8.  Save.

Example:

``` text
Interface: eth2
Type:      Optional
Alias:     DMZ
IP:        192.168.10.254/24
```

WatchGuard's current documentation uses this workflow for Trusted and
Optional interfaces.

Official reference:
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interface_trusted_opt_c.html

------------------------------------------------------------------------

# 16. Lab --- Build an Optional/DMZ Network

### Trusted network

``` text
10.10.10.0/24
Firebox: 10.10.10.254
```

### Optional/DMZ network

``` text
192.168.10.0/24
Firebox: 192.168.10.254
```

### Test server

``` text
Web Server:
192.168.10.10

Gateway:
192.168.10.254
```

Topology:

``` text
             FIREBOX
             /                 /            Trusted         Optional
        |               |
     Test PC        Test Server
10.10.10.10       192.168.10.10
```

------------------------------------------------------------------------

# 17. Lab --- Prove the Security Separation

### Test 1: Trusted → Internet

Confirm normal outbound policy behavior.

### Test 2: Trusted → DMZ

Create a controlled rule, for example:

``` text
Trusted → DMZ
HTTPS
ALLOW
```

### Test 3: DMZ → Trusted

Test this direction separately.

Do not assume that allowing Trusted → DMZ automatically permits DMZ →
Trusted.

### Test 4: Internet → DMZ

If publishing a web server, configure the appropriate NAT/policy.

### Test 5: Internet → Trusted

Do not expose internal employee systems unnecessarily.

------------------------------------------------------------------------

# 18. Common Beginner Mistakes

### Mistake 1 --- "Optional means optional cable"

No. It is a **security/interface zone**.

### Mistake 2 --- "Optional means Internet"

No. External normally represents the WAN/Internet side.

### Mistake 3 --- "Optional automatically secures the server"

No. Policies and the server's own security controls still matter.

### Mistake 4 --- "Trusted and Optional can use the same subnet in Mixed Routing"

Normally no. Separate interfaces generally require different subnets.

### Mistake 5 --- "Any-Optional means Internet access"

No. Any-Optional is an alias. Policies determine what is allowed.

------------------------------------------------------------------------

# 19. When Should You Use Each Mode?

``` text
Need normal routing, NAT, VLANs and broad Firebox features?
        |
       YES
        ↓
Mixed Routing
```

``` text
Need to insert Firebox into an existing IP network
without changing host addressing?
        |
       YES
        ↓
Drop-In
```

``` text
Need transparent inspection between an existing network
and its gateway?
        |
       YES
        ↓
Bridge
```

For most new deployments, WatchGuard identifies Mixed Routing as the
default and most flexible mode.

------------------------------------------------------------------------

# 20. Interview Questions

### Q1. What are the three Firebox network modes?

-   Mixed Routing
-   Drop-In
-   Bridge

### Q2. Which is the default?

Mixed Routing Mode.

### Q3. What is an Optional Interface?

An internal Firebox interface used for a separate mixed-trust or DMZ
network.

### Q4. Is Optional the same as External?

No.

External normally connects toward the WAN/Internet. Optional is an
internal zone.

### Q5. What is the common use of Optional?

DMZ/public-facing servers.

### Q6. What alias represents Optional interfaces?

**Any-Optional**

### Q7. Why place public servers in Optional?

To separate them from the main Trusted network and apply different
firewall policies.

### Q8. Which mode provides the greatest flexibility?

Mixed Routing Mode.

### Q9. What is Drop-In Mode?

A mode where the Firebox uses the same primary IP across interfaces and
can be inserted into an existing network while hosts retain their
addressing.

### Q10. What is Bridge Mode?

A transparent mode that filters/manages traffic between an existing
network and its gateway without operating as the normal Layer-3 gateway.

------------------------------------------------------------------------

# 21. Quick Revision

``` text
NETWORK MODES
│
├── Mixed Routing
│   ├── Default
│   ├── Separate subnets
│   ├── Routing + NAT
│   └── Most flexible
│
├── Drop-In
│   ├── Same primary IP across interfaces
│   ├── Existing network
│   └── More limitations
│
└── Bridge
    ├── Transparent
    ├── Existing network + gateway
    └── Limited routing/NAT/VLAN functions
```

``` text
INTERFACE ZONES
│
├── External
│   └── WAN / outside
│
├── Trusted
│   └── Main internal LAN
│
└── Optional
    └── Mixed-trust / DMZ
```

------------------------------------------------------------------------

# 22. Key Takeaways

1.  **Mixed Routing is the default Firebox network mode.**
2.  Mixed Routing provides the greatest network flexibility.
3.  In Mixed Routing, interfaces normally use different subnets.
4.  **Drop-In Mode** is designed for inserting the Firebox into an
    existing network while retaining host addressing.
5.  **Bridge Mode** provides transparent traffic inspection between an
    existing network and its gateway.
6.  **Optional Interface = separate internal mixed-trust/DMZ zone.**
7.  Optional is **not** the same as External.
8.  Public-facing servers are a common use for an Optional/DMZ network.
9.  Optional interfaces are members of **Any-Optional**.
10. An Optional interface does not automatically secure a server;
    policies still control traffic.
11. The key purpose of Optional is **segmentation and different policy
    treatment**.

------------------------------------------------------------------------

# 23. Uploaded Slide Placement

Use the supplied screenshots at these locations:

-   **Network Modes overview** → after Section 1
-   **Mixed Routing Mode diagram** → after Section 2
-   **Drop-In Mode diagram** → after Section 3
-   **Bridge Mode diagram** → after Section 4
-   **Key Takeaways slide** → after Section 22

The slides are especially useful for visually comparing the three modes.

------------------------------------------------------------------------

# 24. Official WatchGuard References

-   About Network Modes and Interfaces\
    https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/net_setup_about_c.html

-   Mixed Routing Mode\
    https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/Fireware/networksetup/net_config_mixedroutingmode_c.html

-   Drop-In Mode\
    https://www.watchguard.com/help/docs/help-center/en-us/Content/en-us/Fireware/networksetup/net_config_dropin_about_c.html

-   Bridge Mode\
    https://www.watchguard.com/help/docs/help-center/en-us/Content/en-us/Fireware/networksetup/net_config_bridgemode_c.html

-   Common Interface Settings\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interfaces_config_c.html

-   Configure a Trusted or Optional Interface\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/networksetup/interface_trusted_opt_c.html

-   About Internal Interfaces\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/other/QSW_fb_internal_interfaces_wsm.html

-   Configure a Custom Interface\
    https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/interface_custom_c.html

------------------------------------------------------------------------

# Final Mental Model

``` text
                 FIREBOX
                    |
       +------------+------------+
       |            |            |
       v            v            v
   External       Trusted      Optional
   WAN/Internet   Main LAN     DMZ / Mixed Trust
                                  |
                                  v
                           Public Servers
```

And:

``` text
Mixed Routing
= Normal routed firewall

Drop-In
= Insert firewall into existing IP network

Bridge
= Transparent firewall inspection
```

> **Optional Interface is not "the optional cable." It is a separate
> security zone, commonly used as a DMZ, that lets you apply different
> firewall policies to systems that should not live directly inside the
> Trusted network.**
