# Network Basics — VPN

## 1. What is a VPN?

**VPN = Virtual Private Network**

A VPN creates a logical, protected connection across an untrusted network such as the Internet.

A VPN can provide:

- **Confidentiality** — prevents unauthorized parties from reading traffic.
- **Integrity** — detects unauthorized modification.
- **Authentication** — verifies the VPN peers.
- **Secure tunneling** — carries private-network traffic across an untrusted network.

### Simple example

```text
HQ Network                         Branch Network
10.10.10.0/24                     10.20.20.0/24
     │                                  │
     ▼                                  ▼
┌──────────┐                        ┌──────────┐
│ Firewall │========================│ Firewall │
└──────────┘       Internet         └──────────┘
                  🔐 VPN Tunnel
```

---

## 2. Main Types of VPN

### 1. Site-to-Site VPN

Connects one network/site to another network/site.

Typical uses:

- Head Office ↔ Branch Office
- Data Center ↔ Branch
- Office ↔ Cloud VPC/VNet
- Partner network ↔ Enterprise network

```text
                    INTERNET
                       │
             🔐 IPsec VPN Tunnel
                       │
       ┌───────────────┴───────────────┐
       │                               │
   HQ Firewall                    Branch Firewall
       │                               │
   10.10.10.0/24                  10.20.20.0/24
```

### 2. Remote-Access VPN

Connects an individual user/device to a corporate network.

```text
Laptop
   │
   │ Internet
   ▼
🔐 VPN Tunnel
   │
   ▼
Corporate Firewall
   │
   ▼
Corporate Network
```

Typical uses:

- Employee working from home
- Engineer accessing corporate resources
- Mobile users
- Remote administrators

---

# 3. Site-to-Site IPsec VPN

For a traditional site-to-site VPN, **IPsec** is commonly used.

IPsec operates at the **Network Layer (Layer 3)** and provides security for IP traffic.

The two VPN gateways negotiate how the tunnel will be created and protected.

```text
HQ Firewall                     Branch Firewall
     │                                │
     │────── IKE Phase 1 ────────────►│
     │◄───── IKE Phase 1 ─────────────│
     │                                │
     └────── IKE SA established ──────┘
```

There are two important stages:

```text
             IPsec VPN
                 │
        ┌────────┴────────┐
        │                 │
     Phase 1           Phase 2
        │                 │
   IKE SA /            IPsec SA /
   IKE security        CHILD SA
   association
```

---

# 4. IPsec VPN Phase 1

## Purpose of Phase 1

Phase 1 establishes a secure management/control channel between the two VPN peers.

Its purpose is:

> **Let's securely authenticate each other and agree on how we will protect our further negotiations.**

The resulting **IKE Security Association (IKE SA)** protects the Phase 2 negotiation.

Phase 1 negotiates things such as:

- Authentication method
- Encryption algorithm
- Integrity/hash algorithm
- Diffie-Hellman (DH) group
- IKE lifetime
- Peer authentication

Authentication can commonly use:

- Pre-shared key (**PSK**)
- Digital certificates

---

# 5. IKEv1 vs IKEv2

| Feature | IKEv1 | IKEv2 |
|---|---|---|
| Older protocol | Yes | No |
| Main Mode | Yes | No |
| Aggressive Mode | Yes | No |
| Initial exchange | More messages | Fewer messages |
| Mobility support | Limited | Better |
| NAT traversal | Supported | Supported |
| Modern deployments | Legacy | Common choice |

### Important

**Main Mode and Aggressive Mode belong to IKEv1.**

**IKEv2 does NOT have Main Mode or Aggressive Mode.**

---

# 6. IKEv1 Main Mode

IKEv1 Main Mode uses a **6-message exchange**.

Its main advantage is better protection of peer identities compared with Aggressive Mode.

```text
Initiator                         Responder
    │                                │
    │── SA proposals ───────────────►│
    │◄─ SA selection ────────────────│
    │── DH information ─────────────►│
    │◄─ DH information ──────────────│
    │── Authentication/Identity ────►│
    │◄─ Authentication/Identity ─────│
    │                                │
    └────── IKE SA established ──────┘
```

Main Mode provides:

- Negotiation of security parameters
- Diffie-Hellman key exchange
- Peer authentication
- Better identity protection

---

# 7. IKEv1 Aggressive Mode

IKEv1 Aggressive Mode uses **3 messages** instead of Main Mode's 6.

```text
Initiator                         Responder
    │                                │
    │──── Message 1 ────────────────►│
    │◄─── Message 2 ────────────────│
    │──── Message 3 ────────────────►│
    │                                │
    └────── IKE SA established ──────┘
```

## Why use Aggressive Mode?

One common scenario is when the remote peer has a:

**Dynamic / non-static public IP address.**

For example:

```text
HQ Firewall
Public IP:
203.0.113.10
      │
      │ Internet
      │
      ▼
Branch Firewall
Public IP:
Dynamic
```

The branch public IP might change over time:

```text
Today:     198.51.100.20
Tomorrow:  198.51.100.75
Later:     198.51.100.120
```

A configuration that depends on a fixed peer IP may therefore be unsuitable.

Aggressive Mode can allow the peer to be identified using information exchanged during IKE negotiation rather than relying solely on a preconfigured static peer IP.

### Interview answer

> **Aggressive Mode is an IKEv1 exchange that completes Phase 1 in three messages. It can be useful when the remote peer has a dynamic IP address because peer identification can be based on the exchanged identity rather than requiring a fixed IP. However, Aggressive Mode exposes identity information earlier and provides weaker identity protection than Main Mode, so it should not be preferred simply because it is faster.**

### Important correction

Do **not** memorize:

> Dynamic IP = Aggressive Mode always.

Instead remember:

> **Dynamic IP can be a reason to use Aggressive Mode, but it isn't a mandatory requirement.**

Aggressive Mode is also a legacy IKEv1 mechanism and has weaker identity protection than Main Mode.

---

# 8. IKEv2

IKEv2 simplifies the negotiation process compared with IKEv1.

A simplified view:

```text
Initiator                         Responder
    │                                │
    │──── IKE_SA_INIT ──────────────►│
    │◄─── IKE_SA_INIT ───────────────│
    │──── IKE_AUTH ─────────────────►│
    │◄─── IKE_AUTH ──────────────────│
    │                                │
    └──────── IKE SA established ────┘
```

IKEv2 supports modern VPN deployments and handles dynamic IP peers without requiring an IKEv1-style Aggressive Mode.

---

# 9. Phase 2 — Quick Mode

Once Phase 1 is complete, the peers negotiate the actual **IPsec data protection**.

For IKEv1 this is called **Quick Mode**.

```text
             Phase 1
          IKE SA established
                 │
                 ▼
            Quick Mode
                 │
                 ▼
            Phase 2
                 │
                 ▼
          IPsec SA established
                 │
                 ▼
          🔐 Encrypted Data
```

Phase 2 determines things such as:

- ESP or AH
- Encryption algorithm
- Integrity/authentication algorithm
- Traffic selectors
- IPsec lifetime
- PFS/DH settings, if enabled

---

# 10. ESP vs AH

## ESP — Encapsulating Security Payload

ESP is the commonly used IPsec protocol.

It can provide:

- Encryption
- Integrity
- Authentication
- Anti-replay protection

```text
Original packet
      │
      ▼
┌──────────────────┐
│ IP Header        │
│ ESP Header       │
│ Encrypted Data   │
│ ESP Auth Data    │
└──────────────────┘
```

## AH — Authentication Header

AH provides:

- Integrity
- Authentication

but **does not provide encryption**.

AH is also problematic with NAT because it protects parts of the IP header.

Therefore:

> **ESP is generally used instead of AH for modern IPsec VPN deployments.**

---

# 11. What is the Actual VPN Tunnel?

```text
HQ LAN
10.10.10.0/24
     │
     ▼
┌───────────┐
│ HQ FW     │
└─────┬─────┘
      │
      │ ESP
      ▼
══════════════════════════
     INTERNET
   🔐 IPsec Tunnel
══════════════════════════
      │
      ▼
┌───────────┐
│ Branch FW │
└─────┬─────┘
      │
      ▼
Branch LAN
10.20.20.0/24
```

A packet such as:

```text
10.10.10.10 → 10.20.20.20
```

is encapsulated/protected by IPsec as it crosses the Internet.

The receiving firewall removes the IPsec protection and forwards the original traffic to the destination network.

---

# 12. VPN Tunnel Topologies

## A. Hub-and-Spoke

A central site acts as the hub.

```text
             Branch 1
                │
                │
                ▼
Branch 2 ───► HQ ◄─── Branch 3
                │
                │
                ▼
             Branch 4
```

Advantages:

- Centralized management
- Easier to deploy
- Suitable for many branches

Potential drawback:

- Traffic between branches may have to pass through the hub.

---

## B. Full Mesh

Every site can have a direct VPN tunnel with other sites.

```text
       Branch A
        /    \
       /      \
      /        \
   HQ ───────── Branch B
      \        /
       \      /
        \    /
       Branch C
```

Advantages:

- Direct site-to-site communication
- Lower dependency on a central hub

Disadvantage:

- Number of tunnels increases rapidly as sites increase.

---

## C. Partial Mesh / Hybrid

A combination of centralized and direct tunnels.

```text
             Branch A
                │
                │
                ▼
                HQ
              /    \
             /      \
            ▼        ▼
       Branch B ─── Branch C
```

Some sites communicate through the hub while selected sites have direct tunnels.

---

# 13. Remote-Access VPN

Remote-access VPN allows an individual device to connect to the corporate network.

```text
               Internet
                  │
          🔐 VPN Connection
                  │
                  ▼
           ┌────────────┐
           │ VPN Gateway│
           └──────┬─────┘
                  │
                  ▼
          Corporate Network
```

Common technologies include:

- **IPsec**
- **IKEv2**
- **SSL/TLS VPN**
- **L2TP**

---

# 14. Split Tunneling

With **split tunneling**, only selected corporate traffic uses the VPN.

```text
Laptop
   │
   ├──── Corporate traffic ────► 🔐 VPN ───► HQ
   │
   └──── Internet traffic ─────► Internet
```

For example:

```text
10.0.0.0/8
    │
    └──► VPN

Internet
    │
    └──► Local ISP
```

### Advantages

- Reduces VPN bandwidth consumption
- Internet traffic doesn't necessarily traverse corporate infrastructure
- Can improve performance

### Security consideration

Internet traffic bypassing corporate security controls can increase risk depending on the organization's security architecture.

---

# 15. Default-Route VPN / Full Tunnel

In a default-route VPN, essentially **all client traffic** is sent through the VPN.

```text
Laptop
   │
   ▼
🔐 VPN Tunnel
   │
   ▼
Corporate Firewall
   │
   ├──► Corporate resources
   │
   └──► Internet
```

Conceptually:

```text
0.0.0.0/0
    │
    ▼
VPN Tunnel
```

The corporate security infrastructure can then inspect and enforce policies on the client's Internet traffic.

---

# 16. Split Tunnel vs Full Tunnel

| Feature | Split Tunnel | Full Tunnel |
|---|---|---|
| Corporate traffic | VPN | VPN |
| Internet traffic | Local Internet | VPN |
| VPN bandwidth usage | Lower | Higher |
| Centralized inspection | Limited | Greater |
| Performance | Often better | Can be slower |
| Security control | Depends on design | More centralized |

---

# 17. VPN Authentication

VPN peers need to authenticate each other.

## Pre-Shared Key — PSK

Both devices know the same shared secret.

```text
Firewall A
    │
    │ Shared secret
    │
Firewall B
```

Example:

```text
PSK = StrongSecret123!
```

**Never use weak/default shared secrets in production.**

## Certificate Authentication

Certificates can authenticate VPN peers.

```text
Certificate
     │
     ▼
Identity
     │
     ▼
Authentication
     │
     ▼
VPN established
```

For larger environments, certificate-based authentication can be easier to manage securely than many manually distributed PSKs.

---

# 18. Diffie-Hellman — Why Is It Used?

DH allows two peers to establish shared keying material over an untrusted network without directly sending the final secret key.

```text
HQ Firewall                 Branch Firewall
     │                           │
     │──── DH exchange ─────────►│
     │◄─── DH exchange ──────────│
     │                           │
     └── Shared secret material ─┘
```

Important:

> **DH is a key-agreement mechanism, not an encryption algorithm.**

---

# 19. Complete IPsec VPN Flow

```text
              IPsec VPN
                  │
                  ▼
        ┌──────────────────┐
        │ Phase 1 / IKE    │
        │                  │
        │ Authenticate     │
        │ Negotiate crypto │
        │ DH exchange      │
        └────────┬─────────┘
                 │
                 ▼
            IKE SA
                 │
                 ▼
        ┌──────────────────┐
        │ Phase 2 /        │
        │ Quick Mode       │
        │                  │
        │ ESP/AH            │
        │ Encryption       │
        │ Integrity        │
        │ Traffic selectors│
        └────────┬─────────┘
                 │
                 ▼
            IPsec SA
                 │
                 ▼
          🔐 Encrypted Data
```

---

# 20. The Most Important Concept

Do not think:

> **Phase 1 encrypts my application traffic.**

Think:

### Phase 1

**Creates a secure control channel and establishes the IKE SA.**

### Phase 2

**Negotiates the security parameters/SAs used to protect the actual IP traffic.**

```text
Phase 1
   ↓
"Who are you and how should we securely negotiate?"

Phase 2
   ↓
"What traffic should be protected and how?"

Data
   ↓
"Now send the actual protected traffic."
```

---

# 🎯 Interview Questions

## Q1. What is a VPN?

A VPN creates a secure logical connection across an untrusted network, providing tunneling and security services such as confidentiality, integrity and authentication.

## Q2. What are the two major types of VPN?

**Site-to-Site VPN** and **Remote-Access VPN**.

## Q3. What is the purpose of IPsec Phase 1?

Phase 1 establishes an authenticated and protected IKE security association used to securely negotiate further IPsec parameters.

## Q4. What is the purpose of Phase 2?

Phase 2 negotiates the IPsec SAs that protect the actual data traffic, including parameters such as ESP/AH, encryption, integrity and traffic selectors.

## Q5. What is the difference between Main Mode and Aggressive Mode?

Both are **IKEv1 Phase 1 exchanges**.

- Main Mode → 6 messages, better identity protection.
- Aggressive Mode → 3 messages, faster, but weaker identity protection.

## Q6. Why might Aggressive Mode be used?

A common use case is when the remote peer has a **dynamic/non-static IP address**, because peer identification can be based on exchanged identity information rather than relying solely on a fixed peer IP.

However, it is not automatically required whenever an IP is dynamic.

## Q7. Does IKEv2 have Aggressive Mode?

**No.**

Main Mode and Aggressive Mode are concepts from **IKEv1**.

## Q8. What is ESP?

**Encapsulating Security Payload.**

ESP provides IPsec protection including encryption and integrity/authentication, depending on the configured algorithms.

## Q9. What is AH?

**Authentication Header.**

AH provides authentication/integrity but **does not encrypt the payload**.

## Q10. What is split tunneling?

Split tunneling sends selected traffic through the VPN while other traffic uses the user's normal Internet connection.

## Q11. What is a full-tunnel VPN?

A full-tunnel VPN routes essentially all client traffic through the VPN gateway.

## Q12. What is Hub-and-Spoke VPN?

A centralized topology where multiple branch VPN tunnels terminate at a central hub.

---

# 🔥 Quick Revision

```text
VPN
│
├── Site-to-Site
│     │
│     └── Usually IPsec
│
└── Remote Access
      │
      ├── IPsec
      ├── IKEv2
      ├── SSL/TLS VPN
      └── L2TP
```

```text
IPsec
 │
 ├── Phase 1
 │    └── IKE SA
 │         ├── Authentication
 │         ├── Crypto negotiation
 │         └── DH
 │
 └── Phase 2
      └── IPsec SA
           ├── ESP/AH
           ├── Encryption
           ├── Integrity
           └── Traffic selectors
```

```text
IKEv1
 ├── Main Mode → 6 messages
 └── Aggressive Mode → 3 messages
                         │
                         └── Can be useful with
                             dynamic peer IPs
                             but weaker identity
                             protection

IKEv2
 └── No Main/Aggressive Mode
```

## ⭐ One-line memory trick

> **Phase 1 = Build the secure negotiation channel.**  
> **Phase 2 = Build the security for the actual traffic.**  
> **Aggressive Mode = Faster IKEv1 Phase 1, useful in some dynamic-peer designs, but with weaker identity protection.**

---

# 📌 Interview-Focused Notes

### If asked: "Why does a VPN need IKE?"

IKE (Internet Key Exchange) negotiates security parameters, authenticates the VPN peers and establishes keying material/security associations used by IPsec.

### If asked: "What happens if Phase 1 fails?"

Phase 2 cannot successfully establish because the peers do not have the required secure IKE relationship for negotiating the IPsec SA.

### If asked: "What happens if Phase 1 is UP but Phase 2 is DOWN?"

The VPN gateways may have successfully authenticated and established the IKE SA, but the IPsec/traffic-protection parameters have not successfully negotiated. Actual protected traffic will therefore not pass through that IPsec tunnel.

### If asked: "What is a dynamic peer?"

A dynamic peer is a VPN endpoint whose reachable public IP address is not fixed and may change, commonly because the endpoint receives its address dynamically from an ISP.

### If asked: "Is Aggressive Mode more secure because it is faster?"

No. Faster negotiation does not mean stronger security. Aggressive Mode has weaker identity protection than Main Mode.

### If asked: "Which is preferred for modern deployments, IKEv1 or IKEv2?"

**IKEv2 is generally preferred for modern deployments** when supported, while IKEv1 is mainly encountered in legacy or compatibility scenarios.
