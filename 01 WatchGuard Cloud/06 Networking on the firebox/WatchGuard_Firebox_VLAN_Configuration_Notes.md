# WatchGuard Firebox — VLAN Configuration Notes

## 1. VLAN on a Firebox

A VLAN (Virtual Local Area Network) logically separates devices into different Layer-2 broadcast domains without requiring separate physical cabling.

On a WatchGuard Firebox, a VLAN can be assigned to a security zone such as **Trusted, Optional, Custom, or External**.

**Mental model:**

`Switch VLAN → 802.1Q tag → Firebox VLAN interface → Gateway IP → Firewall policies`

---

## 2. Why use VLANs?

Typical examples:

| VLAN | Purpose | Example subnet |
|---|---|---|
| VLAN 10 | Users | 192.168.10.0/24 |
| VLAN 20 | Servers | 192.168.20.0/24 |
| VLAN 30 | Guest Wi-Fi | 192.168.30.0/24 |
| VLAN 40 | Voice | 192.168.40.0/24 |
| VLAN 50 | Management | 192.168.50.0/24 |

VLANs let you segment networks and then use Fireware policies to control traffic between them.

---

## 3. Main VLAN configuration fields

### VLAN Name

Use a meaningful name. WatchGuard requires the name to contain no spaces.

Example:

```text
Users_VLAN10
```

### Description

Optional documentation field.

Example:

```text
Corporate user network
```

### VLAN ID

The numerical identifier for the VLAN.

Example:

```text
VLAN ID: 10
```

The Firebox VLAN ID must match the VLAN ID used by the connected switch/network device.

### Security Zone

Select the security zone:

- **Trusted** — normally used for internal trusted networks.
- **Optional** — commonly used for less-trusted internal/DMZ-style networks.
- **Custom** — separate security zone with explicit policy control.
- **External** — for external/WAN-side VLAN use cases.

A VLAN's security zone affects how Fireware policies and aliases treat the VLAN.

### IP Address

Enter the Firebox gateway address for the VLAN.

Example:

```text
192.168.10.1/24
```

Clients in that VLAN use this IP as their default gateway.

Remember:

```text
VLAN ID = Layer-2 identifier
Gateway IP = Layer-3 Firebox address
```

---

## 4. Configure a VLAN in Fireware Web UI

### Step 1 — Configure the physical interface

Go to:

```text
Network
  ↓
Interfaces
  ↓
Select the connected physical interface
  ↓
Edit
  ↓
Interface Type = VLAN
  ↓
Save
```

At least one interface must be configured as a VLAN interface before creating the VLAN.

### Step 2 — Create the VLAN

Go to:

```text
Network
  ↓
VLAN
  ↓
Add
```

Enter:

1. Name
2. Description (optional)
3. VLAN ID
4. Security Zone
5. IP Address / gateway
6. Interface traffic setting

Then save.

---

## 5. Tagged / Untagged / No Traffic

For each selected interface, WatchGuard lets you choose:

### Tagged traffic

The interface sends and receives VLAN-tagged frames using IEEE 802.1Q.

Typical use:

```text
Firebox
   |
   | 802.1Q trunk
   |
Switch
```

### Untagged traffic

The interface sends and receives traffic without a VLAN tag for that VLAN.

Typical use:

```text
Access/native VLAN
```

### No traffic

The interface does not carry that VLAN.

---

## 6. Tagged vs untagged

Easy memory rule:

```text
Trunk → Tagged VLANs
Access → Usually untagged traffic
```

The switch and Firebox must be configured consistently.

A Firebox interface can carry multiple tagged VLANs, allowing one physical connection to function as a VLAN trunk.

---

## 7. Example: multiple VLANs on one Firebox port

```text
                 FIREBOX
                    |
              Port 3 / Trunk
                    |
              Managed Switch
            ____/____|____\
           /       |       \
       VLAN 10   VLAN 20   VLAN 30
        Users     Servers    Guest
```

Example Firebox configuration:

```text
VLAN 10
Name: Users
Zone: Trusted
Gateway: 192.168.10.1/24
Traffic: Tagged

VLAN 20
Name: Servers
Zone: Trusted
Gateway: 192.168.20.1/24
Traffic: Tagged

VLAN 30
Name: Guest
Zone: Optional
Gateway: 192.168.30.1/24
Traffic: Tagged
```

---

## 8. DHCP on a VLAN

For supported internal VLAN security zones, the Firebox can provide DHCP.

Example:

```text
VLAN 10
Gateway: 192.168.10.1
DHCP pool:
192.168.10.100 - 192.168.10.200
```

The Firebox can also use **DHCP Relay** when the DHCP server is elsewhere.

---

## 9. Firewall policies between VLANs

Creating a VLAN does **not** mean all traffic between VLANs is automatically allowed.

Example:

```text
VLAN 10 Users
      |
      | HTTPS
      v
VLAN 20 Servers
```

You can create a policy that permits only required services.

Example:

```text
Source:      VLAN10_Users
Destination: VLAN20_Servers
Service:     HTTPS
```

This is where VLAN segmentation and Fireware security policies work together.

---

## 10. Intra-VLAN traffic

Intra-VLAN traffic is traffic where the source and destination are in the same VLAN.

Example:

```text
PC1 ───── VLAN 10 ───── PC2
```

Hosts on the same Layer-2 VLAN normally communicate directly through the switch. For Firebox policies to inspect/control same-VLAN traffic, the traffic must actually pass through the Firebox.

Therefore, a firewall policy does not automatically force a same-VLAN conversation through the Firebox.

---

## 11. Switch-side configuration

The switch configuration must match the Firebox.

For a trunk:

```text
Firebox Port 3
     |
     | Tagged VLAN 10
     | Tagged VLAN 20
     | Tagged VLAN 30
     |
Switch Trunk Port
```

For an access port:

```text
Switch Port 10
VLAN = 10
Untagged
```

A client on the access port normally sends ordinary Ethernet frames; the switch associates those frames with VLAN 10.

---

## 12. Practical lab

### Topology

```text
                         INTERNET
                            |
                         FIREBOX
                            |
                    VLAN Trunk / Port 3
                            |
                     MANAGED SWITCH
                  _________|_________
                 /         |         \
             VLAN 10    VLAN 20    VLAN 30
              Users      Servers      Guest
```

### Addressing

| VLAN | Purpose | Zone | Gateway |
|---|---|---|---|
| 10 | Users | Trusted | 192.168.10.1/24 |
| 20 | Servers | Trusted | 192.168.20.1/24 |
| 30 | Guest | Optional | 192.168.30.1/24 |

Example policy design:

```text
Users  → Internet       ALLOW
Users  → Servers        ALLOW required services
Guest  → Internet       ALLOW
Guest  → Users          DENY
Guest  → Servers        DENY
```

Use the organization's actual security requirements when designing policies.

---

## 13. Troubleshooting checklist

### Client gets no DHCP address

Check:

- VLAN exists on the Firebox.
- VLAN ID is correct.
- Gateway IP is correct.
- DHCP server/relay is configured.
- Switch access port has the correct VLAN.
- Trunk carries the VLAN.
- Firebox interface is configured as VLAN.
- Tagged/untagged settings match.

### Client has an IP but cannot reach gateway

Check the complete path:

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

Common causes:

- VLAN ID mismatch
- Wrong tagging
- VLAN missing from trunk
- Wrong subnet
- Incorrect gateway
- Incorrect interface assignment

### One VLAN works but another does not

Compare:

```text
VLAN ID
Security Zone
Gateway IP
Switch VLAN
Trunk allowed VLANs
Tagged/Untagged setting
DHCP
Firewall policy
```

---

## 14. Important WatchGuard restrictions / notes

- VLANs cannot be used when the Firebox is configured in **Drop-In Mode**.
- Maximum VLAN count depends on the Firebox feature key.
- WatchGuard recommends not creating more than 10 VLANs operating on external interfaces because too many can affect performance.
- VLANs use IEEE 802.1Q tagging.
- WatchGuard recommends using a VLAN ID other than **1** for networks passing traffic to the Firebox and not using VLAN 1 for user, server, or management networks.
- Current WatchGuard documentation contains model/Fireware-version-specific reserved VLAN IDs. For certain T-series models, the reserved high VLAN ID changed between Fireware 2026.2 and 2026.2.1+, so check the current documentation for your exact model/version before using VLAN IDs near the top of the range.

---

## 15. Configuration checklist

```text
[ ] Network > Interfaces
[ ] Set connected physical interface to VLAN
[ ] Save

[ ] Network > VLAN
[ ] Add VLAN
[ ] Enter Name
[ ] Enter VLAN ID
[ ] Select Security Zone
[ ] Enter Gateway IP
[ ] Select interface
[ ] Choose Tagged / Untagged / No Traffic
[ ] Save

[ ] Configure DHCP or DHCP Relay
[ ] Configure firewall policies
[ ] Configure switch VLAN
[ ] Configure switch trunk/access ports
[ ] Test gateway
[ ] Test DHCP
[ ] Test Internet
[ ] Test inter-VLAN policies
```

---

## 16. Interview questions

**Q: What VLAN standard does WatchGuard use?**  
A: IEEE 802.1Q.

**Q: What is the difference between VLAN ID and gateway IP?**  
A: VLAN ID identifies the Layer-2 VLAN; the gateway IP is the Firebox Layer-3 address used by clients as their default gateway.

**Q: What is tagged traffic?**  
A: Ethernet traffic carrying an IEEE 802.1Q VLAN tag.

**Q: What is untagged traffic?**  
A: Traffic arriving without an 802.1Q tag for that VLAN.

**Q: What is a trunk?**  
A: A link that can carry multiple VLANs, normally using VLAN tags.

**Q: Can one Firebox interface carry multiple VLANs?**  
A: Yes. A Firebox interface can manage multiple tagged VLANs and function as a VLAN trunk.

**Q: Can the Firebox provide DHCP for a VLAN?**  
A: Yes, for supported internal VLAN security zones; DHCP Relay can also be used.

**Q: Can VLANs be used in Drop-In Mode?**  
A: No.

**Q: Does creating a VLAN automatically allow inter-VLAN traffic?**  
A: No. Fireware policies control traffic between networks.

---

## 17. Quick revision

```text
VLAN
 |
 +-- VLAN ID
 |     +-- Identifies Layer-2 VLAN
 |
 +-- Security Zone
 |     +-- Trusted
 |     +-- Optional
 |     +-- Custom
 |     +-- External
 |
 +-- Gateway IP
 |     +-- Firebox Layer-3 address
 |
 +-- Interface Traffic
 |     +-- Tagged
 |     +-- Untagged
 |     +-- No traffic
 |
 +-- DHCP
 |     +-- Firebox DHCP or Relay
 |
 +-- Firewall Policies
       +-- Control traffic between networks
```

### Most important mental model

**VLAN ID → Switch VLAN → 802.1Q Tag → Firebox VLAN Interface → Gateway IP → Security Zone → Firewall Policy → Allowed/Denied Traffic**

---

## Official WatchGuard references

- About Virtual Local Area Networks (VLANs)  
  https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/Fireware/networksetup/vlans_about_c.html

- Define a New VLAN  
  https://www.watchguard.com/help/docs/help-center/en-us/Content/en-us/Fireware/networksetup/vlan_define_new_c.html

- Network Interface Settings  
  https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/networksetup/network_interface-settings.html

- Configure One VLAN Bridged Across Two Interfaces  
  https://www.watchguard.com/help/docs/help-center/en-us/Content/en-us/Fireware/networksetup/vlan_example_1vlan_2switches_c.html
