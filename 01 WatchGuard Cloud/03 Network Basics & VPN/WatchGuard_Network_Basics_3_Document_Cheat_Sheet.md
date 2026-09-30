# 🔥 WatchGuard Network Basics — 3-Document Cheat Sheet

**Based on:**
1. Network Basics — VPN
2. WatchGuard Network Basics — Encryption
3. WatchGuard Network Basics — Certificates

> **Goal:** Fast revision before labs, interviews, or troubleshooting.

---

# 1. VPN — CORE CONCEPTS

## What is a VPN?

**VPN = Virtual Private Network**

A VPN creates a logical, protected connection across an untrusted network such as the Internet.

### VPN provides

- **Confidentiality** → unauthorized users cannot read traffic
- **Integrity** → detects unauthorized modification
- **Authentication** → verifies VPN peers
- **Secure tunneling** → carries private-network traffic across an untrusted network

## Main VPN Types

| Type | Connects |
|---|---|
| **Site-to-Site VPN** | Network ↔ Network |
| **Remote-Access VPN** | User/device ↔ Corporate network |

### Site-to-Site example

```text
HQ 10.10.10.0/24
      │
   Firewall
      ║
   IPsec VPN
      ║
   Firewall
      │
Branch 10.20.20.0/24
```

### Remote Access

```text
Laptop
   │
Internet
   │
VPN Tunnel
   │
Corporate Firewall
   │
Corporate Network
```

---

# 2. IPsec — THE MOST IMPORTANT FLOW

## IPsec has two major stages

```text
                 IPsec VPN
                    │
          ┌─────────┴─────────┐
          │                   │
       Phase 1             Phase 2
          │                   │
       IKE SA             IPsec SA
          │                   │
   Secure negotiation     Actual traffic
```

## Phase 1

### Purpose

**Establish a secure control/negotiation channel and authenticate the peers.**

Negotiates:

- Authentication
- Encryption
- Integrity/hash
- DH group
- IKE lifetime
- Peer authentication

Authentication can use:

- **PSK**
- **Digital certificates**

### Memory

> **Phase 1 = "Who are you, and how will we securely negotiate?"**

---

# 3. IKEv1 vs IKEv2

| | IKEv1 | IKEv2 |
|---|---|---|
| Generation | Older | Modern |
| Main Mode | ✅ | ❌ |
| Aggressive Mode | ✅ | ❌ |
| Exchange | More messages | More streamlined |
| NAT traversal | Supported | Supported |
| Modern deployments | Mostly legacy/compatibility | Common choice |

## IKEv1 Main Mode

**6 messages**

```text
1 → SA proposals
2 ← SA selection
3 → DH information
4 ← DH information
5 → Authentication/Identity
6 ← Authentication/Identity
```

**Key point:** Better identity protection than Aggressive Mode.

## IKEv1 Aggressive Mode

**3 messages**

```text
1 →
2 ←
3 →
```

Can be useful when the remote peer has a **dynamic/non-static public IP**.

But:

> Dynamic IP ≠ Aggressive Mode is always required.

Aggressive Mode provides weaker identity protection than Main Mode.

## IKEv2

Simplified:

```text
IKE_SA_INIT
     ↓
IKE_AUTH
     ↓
IKE SA established
```

**Important:** IKEv2 does **not** have Main Mode or Aggressive Mode.

---

# 4. IPsec PHASE 2

For IKEv1, Phase 2 is commonly called **Quick Mode**.

### Purpose

Negotiate protection for the **actual IP traffic**.

Can determine:

- ESP or AH
- Encryption
- Integrity/authentication
- Traffic selectors
- IPsec lifetime
- PFS/DH settings, if enabled

### Memory

> **Phase 2 = "What traffic should be protected and how?"**

```text
Phase 1
  ↓
IKE SA
  ↓
Phase 2 / Quick Mode
  ↓
IPsec SA
  ↓
Encrypted data
```

---

# 5. ESP vs AH

| | ESP | AH |
|---|---|---|
| Encryption | ✅ | ❌ |
| Integrity | ✅ | ✅ |
| Authentication | ✅ | ✅ |
| Anti-replay | ✅ | — |
| NAT compatibility | Better | Problematic |

### Remember

> **ESP = commonly used IPsec protection.**

> **AH = authentication/integrity, no encryption.**

---

# 6. VPN TOPOLOGIES

## Hub-and-Spoke

```text
Branch A ─┐
Branch B ─┼── HQ
Branch C ─┤
Branch D ─┘
```

**Pros:** Centralized, easier to manage.

**Potential drawback:** Branch-to-branch traffic may pass through HQ.

## Full Mesh

Every site can have direct tunnels with other sites.

**Pros:** Direct communication.

**Cons:** Number of tunnels increases rapidly as sites increase.

## Partial Mesh / Hybrid

Combination of centralized and direct VPN tunnels.

---

# 7. REMOTE-ACCESS VPN

Common technologies listed in the source:

- IPsec
- IKEv2
- SSL/TLS VPN
- L2TP

## Split Tunnel

```text
Laptop
 ├── Corporate traffic → VPN → HQ
 └── Internet traffic → Local ISP
```

**Advantages**
- Lower VPN bandwidth use
- Internet traffic does not necessarily traverse corporate infrastructure
- Can improve performance

**Security consideration:** Internet traffic bypassing corporate security controls can increase risk depending on design.

## Full Tunnel / Default Route VPN

```text
Laptop
   ↓
VPN
   ↓
Corporate Firewall
   ├── Corporate resources
   └── Internet
```

Conceptually:

```text
0.0.0.0/0 → VPN
```

### Split vs Full

| | Split | Full |
|---|---|---|
| Corporate traffic | VPN | VPN |
| Internet | Local | VPN |
| VPN bandwidth | Lower | Higher |
| Centralized inspection | Limited | Greater |
| Performance | Often better | Can be slower |

---

# 8. ENCRYPTION — CORE CONCEPTS

## Basic Flow

```text
Plaintext
   ↓
Encryption + Key
   ↓
Ciphertext
   ↓
Decryption + Required Key
   ↓
Plaintext
```

### Key Terms

| Term | Meaning |
|---|---|
| Plaintext | Original readable data |
| Ciphertext | Encrypted/unreadable data |
| Encryption | Plaintext → Ciphertext |
| Decryption | Ciphertext → Plaintext |
| Key | Secret/mathematical value used by crypto |
| Cipher | Algorithm used for encryption/decryption |

---

# 9. SYMMETRIC vs ASYMMETRIC

## Symmetric

**Same secret key** for encryption and decryption.

Examples:

- **AES**
- **ChaCha20**

### Advantages

- Fast
- Efficient
- Excellent for bulk data
- Used for VPN traffic and application data

### Main problem

**Key distribution and management.**

## Asymmetric

Uses:

- **Public key**
- **Private key**

Examples:

- RSA
- ECC

### Public key

Can be shared.

### Private key

Must remain secret.

### Main role

- Authentication
- Public-key operations
- Key establishment mechanisms

---

# 10. HYBRID CRYPTOGRAPHY

Modern secure protocols commonly combine both.

```text
Asymmetric / Key Agreement
          ↓
    Establish session key
          ↓
      Symmetric crypto
          ↓
      Actual data
```

### Memory

> **Asymmetric helps establish trust/keys. Symmetric protects the bulk data efficiently.**

---

# 11. DIFFIE-HELLMAN (DH)

**DH = key-agreement mechanism**

It allows two parties to establish shared secret key material over an insecure channel **without directly transmitting the final secret**.

```text
Alice                         Bob
  │                            │
Private A                    Private B
  │                            │
Public A                     Public B
  │                            │
  └────── Exchange ────────────┘
             ↓
       Same shared secret
```

### Critical points

- DH is **not** a bulk-data encryption algorithm.
- DH by itself does **not authenticate** the parties.
- Real protocols combine DH/key agreement with authentication.
- Certificates/signatures can provide that authentication.

### DH Mental Model

> **DH = Agree on key material.**

---

# 12. PERFECT FORWARD SECRECY (PFS)

PFS protects previously established sessions if a long-term private key is compromised later.

With ephemeral key exchange:

```text
Session 1 → Fresh temporary keys → Session Key 1
Session 2 → New temporary keys   → Session Key 2
Session 3 → New temporary keys   → Session Key 3
```

### Memory

> **PFS = compromise of a long-term key does not automatically expose old sessions when fresh ephemeral session keys were used.**

---

# 13. TLS

**TLS = Transport Layer Security**

Used to secure network communications.

```text
HTTP + TLS = HTTPS
```

TLS provides:

- Confidentiality
- Integrity protection
- Authentication mechanisms

## TLS high-level flow

```text
TLS Handshake
     ↓
Cryptographic negotiation
     ↓
Authentication
     ↓
Key establishment
     ↓
Session keys
     ↓
Encrypted application data
```

---

# 14. TLS 1.2 vs TLS 1.3

| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Design | Older | Modern |
| Handshake | More complex | More streamlined |
| Legacy options | More historical options | Many removed |
| Forward secrecy | Depends on key exchange | Modern key exchange provides it |
| Performance | Good | Improved |
| Application data | Symmetric | Symmetric |

### Important

> TLS 1.3 does **not** use asymmetric encryption for all traffic.

Instead:

```text
Authentication + Key Agreement
          ↓
       Session Keys
          ↓
Symmetric encryption
          ↓
Application data
```

---

# 15. CERTIFICATES — CORE CONCEPT

A **digital certificate** binds an **identity to a public key**.

### Certificate commonly contains

- Subject / identity
- Public key
- Issuer
- Validity period
- Serial number
- Signature
- Key usage
- SAN
- Extensions

### Memory

> **Certificate = Identity + Public Key + Trusted Binding**

---

# 16. CA & PKI

## CA

**CA = Certificate Authority**

A CA is a trusted entity that issues/signs certificates.

```text
CA
 ↓ signs
Server Certificate
 ↓
Client verifies CA signature
 ↓
Trust
```

## PKI

**PKI = Public Key Infrastructure**

Includes:

- Certificate Authorities
- Certificates
- Keys
- Trust stores
- Validation
- Revocation
- Certificate management processes

### Memory

> **CA issues/signs certificates. PKI is the larger system around certificates and trust.**

---

# 17. CSR

**CSR = Certificate Signing Request**

Created by the server/device/user requesting a certificate.

Typical flow:

```text
Requester
   ↓
Generate private key
   ↓
Generate public key
   ↓
Create CSR
   ↓
CA validates request
   ↓
CA signs/issues certificate
```

### Critical rule

> **The private key stays with the requester. It is not sent to the CA as part of the normal CSR process.**

---

# 18. SAN vs CN

### CN

Historically used for the primary certificate name.

### SAN

**SAN = Subject Alternative Name**

Modern TLS hostname validation relies on SAN for DNS names/IP addresses rather than relying only on CN.

Example:

```text
Requested hostname:
www.example.com

Certificate SAN:
DNS:www.example.com
```

---

# 19. CERTIFICATE CHAIN OF TRUST

The classic chain:

```text
ROOT CA
   ↓ signs
INTERMEDIATE CA
   ↓ signs
END-ENTITY / SERVER CERTIFICATE
```

## Root CA

- Top-level trust anchor
- Normally self-signed
- Installed in trusted root store

## Intermediate CA

- Signed by a CA above it
- Issues/signs certificates further down
- Helps protect the root CA

## End-Entity Certificate

Belongs to:

- Server
- User
- Device
- Service

---

# 20. WHY IS THE ROOT CA SELF-SIGNED?

Because it is at the top of its trust hierarchy.

There is no higher CA above it.

```text
Root CA
  ↓
Self-signs own certificate
  ↓
Trusted as a trust anchor
```

### Critical distinction

> **Self-signed does NOT automatically mean insecure.**

Trust depends on how the certificate is distributed/configured.

---

# 21. SELF-SIGNED vs CA-SIGNED

| | Self-Signed | CA-Signed |
|---|---|---|
| Issuer | Same entity/certificate owner | CA |
| Public trust | Not automatically trusted | Can be trusted via CA chain |
| Common use | Labs/testing/private systems | Public services |
| Browser | Usually warning unless trusted | Normally trusted if chain valid |
| Chain | Usually no external CA chain | Root → Intermediate → End entity |

---

# 22. CERTIFICATE VALIDATION CHECKLIST

When troubleshooting a certificate:

### 1. Chain

Does it lead to a trusted root?

### 2. Signature

Are signatures valid?

### 3. Validity

```text
Not Before < Current Time < Not After
```

### 4. Hostname / SAN

Does the requested hostname match SAN?

### 5. Key Usage

Is the certificate permitted for the intended purpose?

### 6. Revocation

Where applicable, check CRL/OCSP.

---

# 23. WATCHGUARD HTTPS/TLS INSPECTION

WatchGuard/firewall relevance:

```text
Client
   │
   │ TLS
   ▼
WatchGuard
   │
   │ TLS
   ▼
Internet Server
```

The firewall may act as a TLS intermediary.

Conceptually:

```text
Client ←── TLS ──→ WatchGuard ←── TLS ──→ Server
```

The client must trust the CA used by the firewall to generate/sign inspection certificates.

Otherwise:

```text
Inspection Certificate
        ↓
CA not trusted
        ↓
Certificate Warning
```

### Troubleshooting checklist

If HTTPS inspection causes certificate warnings, check:

- Inspection CA installed on clients?
- Client trusts the CA?
- Certificate chain?
- Hostname/SAN?
- Certificate validity?
- Inspection configuration?
- Revocation/trust issues?

---

# 24. VPN + ENCRYPTION + CERTIFICATES — HOW THEY CONNECT

This is the **big picture** across all three chapters:

```text
                    SECURE NETWORKING
                           │
          ┌────────────────┼────────────────┐
          │                │                │
         VPN           Encryption       Certificates
          │                │                │
       Tunneling       Protect data     Prove identity
          │                │                │
       IPsec           AES/ChaCha       CA / PKI
          │                │                │
    IKE Phase 1       Symmetric data   Trust chain
          │                │                │
    IKE Phase 2       DH/key agreement  Root → Int → End
          │                │                │
         ESP            TLS/HTTPS       HTTPS inspection
```

---

# 25. ⭐ INTERVIEW ONE-LINERS

### VPN

**What is a VPN?**

> A protected logical connection across an untrusted network providing tunneling and security services such as confidentiality, integrity and authentication.

### Phase 1

> Establishes the authenticated IKE security association and secure negotiation channel.

### Phase 2

> Negotiates the IPsec security association used to protect actual traffic.

### Main Mode

> IKEv1, 6 messages, better identity protection.

### Aggressive Mode

> IKEv1, 3 messages, can be useful with dynamic peer IPs, but weaker identity protection.

### IKEv2

> Modern IKE version; does not use Main or Aggressive Mode.

### ESP

> Provides IPsec protection including encryption and integrity/authentication depending on configuration.

### AH

> Provides authentication/integrity but not encryption.

### Symmetric

> Same secret key; fast; ideal for bulk data.

### Asymmetric

> Public/private key pair; used for authentication and public-key operations/key establishment.

### DH

> Key-agreement mechanism; does not directly encrypt bulk data.

### PFS

> Uses fresh ephemeral session key material so later compromise of a long-term key does not automatically expose previous sessions.

### TLS

> Protocol for securing network communications such as HTTPS.

### Certificate

> Binds an identity to a public key.

### CA

> Trusted authority that issues/signs certificates.

### PKI

> Infrastructure and processes for certificate/key trust, issuance, validation and management.

### CSR

> Request submitted to a CA to obtain a certificate; the private key stays with the requester.

### Root CA

> Top-level trust anchor, normally self-signed.

### Intermediate CA

> CA below the root that issues/signs certificates further down the chain.

### Self-Signed

> Signed by its own corresponding private key; not automatically publicly trusted.

---

# 26. 🚨 FAST TROUBLESHOOTING CHEAT SHEET

## VPN DOWN

Think:

```text
Phase 1?
   ↓
Authentication
Encryption
Integrity
DH
IKE settings
Peer
   ↓
Phase 2?
   ↓
ESP/AH
Encryption
Integrity
Traffic selectors
PFS
Lifetime
   ↓
Routes / policies
```

## Certificate Warning

Think:

```text
Certificate
   ↓
Expired?
   ↓
Hostname/SAN?
   ↓
Trusted CA?
   ↓
Chain complete?
   ↓
Signature valid?
   ↓
Key usage?
   ↓
Revocation?
   ↓
TLS inspection CA trusted?
```

## TLS / HTTPS Problem

Think:

```text
Client
  ↓
Certificate presented
  ↓
Can client trust CA?
  ↓
Does SAN match?
  ↓
Is certificate valid?
  ↓
Is chain complete?
  ↓
TLS handshake
  ↓
Session keys
  ↓
Encrypted traffic
```

---

# 27. 🧠 FINAL MEMORY MAP

```text
                         NETWORK SECURITY
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
       VPN                 CRYPTOGRAPHY          CERTIFICATES
        │                      │                      │
     IPsec                 Symmetric              Identity
        │                 / Asymmetric                │
   ┌────┴────┐               │                    Public Key
 Phase 1   Phase 2            │                      │
   │          │          ┌────┴────┐                 │
  IKE SA    IPsec SA     AES       RSA/ECC            │
   │          │             │         │              │
   │         ESP            │        Trust            │
   │          │             │         │              │
   └──────────┼─────────────┴─────────┼──────────────┘
              │                       │
              └────────── TLS / HTTPS ┘
                         │
                  Secure Application
```

---

# 28. 🔥 10 THINGS TO MEMORIZE

1. **VPN = protected connection over an untrusted network.**
2. **Phase 1 = IKE SA / secure negotiation.**
3. **Phase 2 = IPsec SA / actual traffic protection.**
4. **Main + Aggressive = IKEv1 only.**
5. **IKEv2 has no Main/Aggressive Mode.**
6. **ESP can encrypt; AH does not encrypt.**
7. **Symmetric = same key + fast bulk encryption.**
8. **DH = key agreement, not bulk encryption.**
9. **Certificate = identity bound to public key.**
10. **Root CA → Intermediate CA → End-Entity = chain of trust.**

---

# 29. ONE-PAGE ULTRA-FAST REVISION

```text
VPN
├─ Site-to-Site = Network ↔ Network
├─ Remote Access = User ↔ Network
├─ IPsec Phase 1 = IKE SA
├─ IPsec Phase 2 = IPsec SA
├─ IKEv1 Main = 6 messages
├─ IKEv1 Aggressive = 3 messages
├─ IKEv2 = no Main/Aggressive
├─ ESP = encryption + integrity/authentication
└─ AH = integrity/authentication, no encryption

ENCRYPTION
├─ Symmetric = same secret key
├─ AES / ChaCha20
├─ Asymmetric = public + private
├─ RSA / ECC
├─ DH = key agreement
├─ DH alone = no authentication
├─ PFS = fresh ephemeral session keys
└─ TLS = secure communications

CERTIFICATES
├─ Certificate = identity + public key
├─ CA = issues/signs
├─ PKI = complete trust/certificate infrastructure
├─ CSR = certificate request
├─ Private key stays with requester
├─ Root CA = trust anchor / normally self-signed
├─ Intermediate CA = signs lower certificates
├─ End-Entity = server/user/device certificate
├─ SAN = modern hostname identity field
└─ HTTPS inspection = client must trust inspection CA
```

---

# 🎯 Final Mental Model

> **VPN creates the protected path.**
>
> **Cryptography provides the mechanisms to protect and establish secure keys.**
>
> **Certificates provide identity and trust.**
>
> **TLS combines authentication/key establishment with efficient symmetric encryption for secure application traffic.**

**WatchGuard Network Basics — 3-Document Cheat Sheet**
