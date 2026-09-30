# WatchGuard Network Basics — Certificates

## Chapter Overview

A **digital certificate** is an electronic document used to bind an identity to a public key.

Certificates are a major part of **PKI (Public Key Infrastructure)** and are used in technologies such as:

- HTTPS / TLS
- VPNs
- Device authentication
- User authentication
- Secure email
- Firewall management
- TLS/SSL inspection

The most important idea:

> **A certificate helps prove that a particular public key belongs to a particular identity.**

---

# 1. What Is a Digital Certificate?

A digital certificate is an electronic document that contains information about a public key and the identity associated with that key.

A certificate typically contains information such as:

- Subject / owner identity
- Subject public key
- Issuer (the CA that issued the certificate)
- Validity period
- Certificate serial number
- Signature algorithm
- Digital signature from the issuer
- Key usage / extended key usage
- Subject Alternative Name (SAN)
- Other certificate extensions

### Simplified structure

```text
Digital Certificate
|
+-- Subject / Identity
+-- Public Key
+-- Issuer / CA
+-- Valid From
+-- Valid Until
+-- Key Usage
+-- Extensions
+-- CA Digital Signature
```

---

# 2. Why Are Certificates Needed?

Suppose you connect to:

```text
https://www.example.com
```

Your browser receives a certificate containing a public key and identity information.

The browser needs to determine:

> "Can I trust that this public key really belongs to example.com?"

The certificate provides the identity binding, while the **Certificate Authority (CA)** provides the trust mechanism.

Without certificate-based authentication, an attacker could potentially present their own public key while pretending to be the legitimate server.

---

# 3. Certificate Authority (CA)

A **Certificate Authority (CA)** is a trusted entity that issues and signs digital certificates.

The basic idea is:

```text
CA
|
| Signs
v
Server Certificate
|
v
Client verifies CA signature
|
v
Client can establish trust in the certificate
```

The CA is trusted because its certificate is already trusted by the operating system, browser, device, or organization's trust store.

---

# 4. Public Key Infrastructure (PKI)

**PKI = Public Key Infrastructure**

PKI is the broader system used to create, issue, manage, validate, renew, and revoke digital certificates and their associated keys.

It can involve:

- Certificate Authorities
- Registration/validation processes
- Certificate repositories
- Revocation mechanisms
- Trust stores
- Public/private keys
- Digital certificates

### Simple PKI model

```text
                 PKI
                  |
       +----------+----------+
       |          |          |
      CA       Certificates  Keys
       |                     |
       +----------+----------+
                  |
            Trust / Identity
```

---

# 5. Certificate Signing Request (CSR)

A **CSR = Certificate Signing Request**.

A CSR is created by the organization, server, device, or user requesting a certificate.

The CSR contains information that the CA needs to create/sign the certificate.

### Important security point

The private key should be generated and retained by the requester.

The **private key is not sent to the CA as part of the CSR**.

Conceptually:

```text
Company / Server
|
+-- Generates private key
|
+-- Generates public key
|
+-- Creates CSR containing public-key information
|   and identity/request information
|
v
Certificate Authority
|
+-- Validates request
|
+-- Issues/signs certificate
|
v
Signed Certificate
```

---

# 6. Typical CSR Information

The lecture highlights fields such as:

### Common Name (CN)

Historically used for the primary name of the certificate.

Examples:

```text
www.example.com
```

or

```text
*.example.com
```

### Organization (O)

The organization/company name.

### Locality (L)

City/locality.

### Country (C)

Two-letter country code.

Example:

```text
IN
US
GB
```

### Important modern note

For TLS server identity validation, the **Subject Alternative Name (SAN)** extension is the important field used for DNS names/IP addresses. Do not rely on CN alone.

---

# 7. Key Algorithms and Key Length

The lecture shows examples such as:

- RSA
- DSA
- ECC

The key length depends on the algorithm.

### RSA

The slide gives examples including:

- 1024 bits
- 2048 bits
- 3072 bits
- 4096 bits

For modern systems, **RSA 2048 bits or stronger** is generally used rather than legacy 1024-bit RSA.

### DSA

DSA = Digital Signature Algorithm.

The lecture lists several key sizes, but DSA is a legacy algorithm and is not the normal choice for new TLS deployments.

### ECDSA

ECDSA = Elliptic Curve Digital Signature Algorithm.

Examples from the lecture:

- P-256
- P-384

ECC provides strong security with comparatively smaller key sizes than RSA.

---

# 8. Certificate Key Usage

A certificate can indicate what its associated public key is intended to be used for.

Examples include:

- Digital signatures
- Key agreement
- Key encipherment
- Server authentication
- Client authentication

This is important because a certificate should not automatically be treated as valid for every cryptographic purpose.

---

# 9. HTTPS Certificate Example

The lecture demonstrates HTTPS using WatchGuard's website as an example.

A simplified model:

```text
                HTTPS / TLS
Client  <-------------------------->  Server
   |                                     |
   |          Server Certificate         |
   |<------------------------------------|
   |                                     |
   |       Establish session keys        |
   |<----------------------------------->|
   |                                     |
   |     Symmetrically encrypted data    |
   |<===================================>|
```

### Two cryptographic stages

## Stage 1 — Authentication / key establishment

Certificates and public-key/key-agreement mechanisms are involved.

## Stage 2 — Data protection

A symmetric session key is used to efficiently protect application data.

This connects directly to the previous **Encryption** chapter.

---

# 10. Important: Certificate vs Encryption

A certificate itself is **not the same thing as encryption**.

A certificate mainly provides:

> **Identity + public key + trusted binding**

For example:

```text
Certificate
     |
     +-- "This public key belongs to example.com"
     |
     +-- CA signature helps establish trust
```

The public/private key pair and TLS cryptographic mechanisms are then used as part of authentication and key establishment.

---

# 11. Chain of Trust

This is one of the most important concepts in this chapter.

A certificate is trusted because it can be validated through a **chain of trusted certificates**.

A common chain looks like:

```text
Root CA
   |
   | signs
   v
Intermediate CA
   |
   | signs
   v
End-Entity / Server Certificate
```

### Example

```text
Trusted Root CA
       |
       v
Intermediate CA
       |
       v
www.example.com Certificate
```

The client ultimately trusts the server certificate because it can build a valid chain back to a trusted root CA.

---

# 12. Root CA Certificate

A **Root CA certificate** sits at the top of the trust hierarchy.

The root certificate is normally:

- Self-signed
- Installed in a trusted root store
- Used to establish trust for subordinate/intermediate CAs

### Simplified structure

```text
Root CA Certificate
|
+-- Root CA name
+-- Root CA public key
+-- Root CA signature
```

### Why is it self-signed?

A root CA is at the top of its trust hierarchy.

There is no higher CA in that hierarchy to sign it.

Therefore, it signs its own certificate.

```text
Root CA
  |
  +---- signs its own certificate
             |
             v
       Self-signed Root
```

---

# 13. Self-Signed Certificate

A **self-signed certificate** is a certificate where the certificate is signed using the private key corresponding to the public key contained in the certificate.

In simple terms:

```text
Certificate public key
        +
Certificate owner's private key
        |
        v
Certificate signature
```

The issuer and subject may be the same in a typical self-signed certificate.

### Most important point

> **Self-signed does NOT automatically mean "insecure."**

It means that the certificate's signature is not issued by a CA that is already trusted by the client.

Whether it is trusted depends on how the certificate is distributed and configured.

---

# 14. Self-Signed Certificate Use Cases

Self-signed certificates are commonly useful in:

### 1. Internal testing

Example:

```text
Lab HTTPS Server
       |
       v
Self-signed certificate
```

Useful for testing TLS without purchasing a publicly trusted certificate.

### 2. Development environments

Developers can use self-signed certificates for local applications and test environments.

### 3. Private/internal systems

An organization can deliberately use an internal PKI or a locally trusted root CA.

Important distinction:

- A self-signed **server certificate** may be manually trusted on clients.
- An organization's self-signed **root CA certificate** can be installed in enterprise trust stores and used to sign internal certificates.

### 4. Appliances and labs

Firewalls, network devices, management interfaces, and lab systems may use self-signed certificates when public CA validation is not required.

---

# 15. Self-Signed Certificate vs CA-Signed Certificate

| Feature | Self-Signed | CA-Signed |
|---|---|---|
| Issuer | Same entity / certificate owner | CA |
| Public trust | Not automatically trusted | Can be trusted through CA chain |
| Cost | Usually no public CA cost | May involve CA service/cost |
| Best for | Labs, testing, controlled environments | Public websites/services |
| Browser trust | Usually warning unless explicitly trusted | Normally trusted if chain is valid |
| Chain | Usually no external CA chain | Root → Intermediate → End entity |

---

# 16. Self-Signed Certificate in the Chain of Trust

This is the part you specifically asked to understand.

### Root CA is normally self-signed

```text
                Root CA
                  |
          Self-signed certificate
                  |
                  v
          Trusted Root Store
                  |
                  v
            Trust anchor
```

The client does not normally need another CA to validate the root CA.

Instead, the root CA certificate is already configured as a **trust anchor**.

Then the chain continues downward:

```text
Root CA
   |
   | signs
   v
Intermediate CA
   |
   | signs
   v
End-Entity Certificate
```

### How validation works

Suppose your browser receives:

```text
www.example.com certificate
```

The browser can verify:

```text
End-Entity Certificate
        |
        | signed by
        v
Intermediate CA
        |
        | signed by
        v
Trusted Root CA
```

If the chain is valid and other checks pass, the browser can trust the certificate.

---

# 17. Why Is the Root CA Trusted?

This is a key PKI concept.

The root CA is a **trust anchor**.

Operating systems and browsers contain trusted root CA certificates.

For example:

```text
Operating System / Browser
|
+-- Trusted Root CA 1
+-- Trusted Root CA 2
+-- Trusted Root CA 3
+-- ...
```

An organization can also add its own internal root CA to managed devices.

Therefore:

> Trust ultimately comes from a trusted root certificate already present in the client's trust store or explicitly configured as trusted.

---

# 18. Internal PKI Example

Imagine a company wants to secure internal servers.

It creates:

```text
Company Root CA
       |
       v
Company Intermediate CA
       |
       +------> Internal Web Server
       |
       +------> Firewall
       |
       +------> VPN Gateway
```

The company installs the root CA certificate on employee devices.

Now those devices can validate certificates issued by the company's internal PKI.

---

# 19. WatchGuard Relevance — Certificates

Certificates are extremely important for firewall/security engineers.

You'll encounter them in:

- HTTPS/TLS inspection
- SSL inspection
- VPN authentication
- Web server certificates
- Firewall management
- Device authentication
- Client authentication
- PKI
- Internal CA deployments

### HTTPS inspection example

A firewall can act as a TLS intermediary:

```text
Client
   |
   | TLS session
   v
WatchGuard
   |
   | TLS session
   v
Internet Server
```

For inspection, the firewall may generate/sign a certificate for the destination hostname.

The client must trust the CA used by the firewall.

---

# 20. Why Does HTTPS Inspection Need a Trusted CA?

Suppose:

```text
Client -> WatchGuard -> Internet Server
```

The WatchGuard firewall intercepts the TLS connection.

It may present a certificate for:

```text
www.example.com
```

to the client.

If that certificate was generated by an internal inspection CA, the client must trust that CA.

Otherwise:

```text
Client
   |
   v
Certificate received
   |
   v
CA not trusted
   |
   v
Certificate warning
```

Therefore, enterprise TLS inspection often requires deploying the organization's inspection CA certificate to managed clients.

---

# 21. Certificate Validation

When a client receives a certificate, it can perform multiple checks.

Common checks include:

### 1. Certificate chain

Does the certificate chain lead to a trusted root?

### 2. Signature

Are the signatures valid?

### 3. Validity period

Is the certificate currently valid?

```text
Not Before < Current Time < Not After
```

### 4. Hostname

Does the requested hostname match the certificate's SAN?

Example:

```text
Requested:
www.example.com

Certificate SAN:
DNS:www.example.com
```

### 5. Key usage

Is the certificate allowed to be used for the intended purpose?

### 6. Revocation status

Depending on the environment, the client may check whether the certificate has been revoked using mechanisms such as CRL or OCSP.

---

# 22. Complete Certificate Validation Model

```text
                  Certificate
                       |
          +------------+-------------+
          |            |             |
       Identity      Validity      Signature
          |            |             |
          v            v             v
         SAN       Date/Time       CA chain
          |            |             |
          +------------+-------------+
                       |
                       v
                Trusted Root?
                       |
                       v
                 Trust decision
```

---

# 23. Certificate Chain Example

```text
                 ROOT CA
              Self-Signed
                   |
                   | Signs
                   v
            INTERMEDIATE CA
                   |
                   | Signs
                   v
         END-ENTITY CERTIFICATE
             example.com
```

### What each certificate does

**Root CA**

- Trust anchor
- Normally self-signed
- Kept in trusted root store

**Intermediate CA**

- Issued/signed by root or another CA
- Used to sign end-entity certificates
- Helps protect the root CA by keeping it offline/less exposed in many PKI designs

**End-entity certificate**

- Belongs to a server, user, device, etc.
- Contains the subject's public key
- Used for authentication and/or other permitted purposes

---

# 24. Key Concepts to Remember

### Certificate

> Binds an identity to a public key through a trusted signature.

### CA

> Trusted authority that issues/signs certificates.

### CSR

> Request containing certificate/public-key and identity information submitted to a CA.

### Root CA

> Top-level trust anchor, normally self-signed.

### Intermediate CA

> CA below the root that can issue/sign certificates for end entities.

### End-entity certificate

> Certificate used by the actual server, user, or device.

### Self-signed certificate

> Certificate signed by its own corresponding private key rather than by a separate CA.

### PKI

> The infrastructure and processes used to manage certificates, keys, trust, issuance, validation, and revocation.

---

# 25. Interview Questions and Answers

## Q1. What is a digital certificate?

**Answer:**

A digital certificate is an electronic document that binds an identity to a public key and is digitally signed by a trusted issuer such as a Certificate Authority.

---

## Q2. What information does a certificate contain?

**Answer:**

It can contain the subject identity, public key, issuer, validity period, serial number, signature information, SAN entries, key usage, and other extensions.

---

## Q3. What is a Certificate Authority?

**Answer:**

A CA is a trusted authority that validates certificate requests and signs/Issues digital certificates.

---

## Q4. What is PKI?

**Answer:**

PKI, or Public Key Infrastructure, is the framework of technologies, policies, processes, certificates, keys, CAs, trust stores, and validation/revocation mechanisms used to manage public-key certificates.

---

## Q5. What is a CSR?

**Answer:**

A CSR, or Certificate Signing Request, is a request generated by the certificate requester and submitted to a CA. It contains the public-key-related information and identity/request details needed for certificate issuance. The private key should remain with the requester.

---

## Q6. What is the difference between a CSR and a certificate?

**Answer:**

A CSR is a request submitted to a CA to obtain a certificate. A certificate is the CA-signed document that binds an identity to a public key.

---

## Q7. What is a self-signed certificate?

**Answer:**

A self-signed certificate is signed using the private key corresponding to the public key in the certificate, rather than being signed by an external CA.

---

## Q8. Is a self-signed certificate insecure?

**Answer:**

Not automatically. It is simply not trusted through a public CA chain by default. It can be appropriate for labs, development, internal systems, and controlled environments where the certificate is explicitly trusted.

---

## Q9. What is the most important use case of a self-signed certificate?

**Answer:**

Controlled environments such as testing, development, labs, or private systems where public CA trust is unnecessary. Self-signed root CA certificates can also serve as trust anchors in private PKI.

---

## Q10. Why is a Root CA certificate self-signed?

**Answer:**

The Root CA is at the top of its trust hierarchy and has no higher CA above it to sign its certificate. It therefore signs its own certificate and becomes a trust anchor when installed in a trusted root store.

---

## Q11. Explain the chain of trust.

**Answer:**

A certificate chain typically goes from a trusted Root CA to an Intermediate CA and then to an End-Entity certificate. The client verifies each signature and ultimately verifies that the chain terminates at a trusted root.

```text
Trusted Root CA
      |
      v
Intermediate CA
      |
      v
Server Certificate
```

---

## Q12. What is a trust anchor?

**Answer:**

A trust anchor is a certificate or public key that a system already trusts, commonly a Root CA certificate stored in the operating system or browser trust store.

---

## Q13. Why are intermediate CAs used?

**Answer:**

Intermediate CAs allow the root CA to remain more protected while subordinate CAs issue certificates to end entities. They also provide organizational and administrative separation within PKI.

---

## Q14. What is the difference between a Root CA and an Intermediate CA?

**Answer:**

A Root CA is the top-level trust anchor and is normally self-signed. An Intermediate CA is signed by a trusted CA above it and is used to issue/sign certificates further down the chain.

---

## Q15. What is an end-entity certificate?

**Answer:**

An end-entity certificate is a certificate issued to the actual server, user, device, or service that needs to authenticate using the certificate.

---

## Q16. What is the purpose of the public key in a certificate?

**Answer:**

The certificate binds the public key to an identity. Other parties can use the public key for cryptographic operations allowed by the certificate and verify that it belongs to the identified entity.

---

## Q17. What is the purpose of the CA's digital signature?

**Answer:**

The CA's signature allows a client to verify that the certificate was issued/signed by the claimed CA and that the certificate data has not been altered since signing.

---

## Q18. What is the difference between certificate and private key?

**Answer:**

A certificate is normally public and contains a public key plus identity and issuer information. The private key is secret and must be protected by the certificate owner.

---

## Q19. What is SAN?

**Answer:**

SAN stands for Subject Alternative Name. It identifies the DNS names, IP addresses, or other identities for which a certificate is valid. Modern TLS hostname validation relies on SAN rather than relying solely on the Common Name.

---

## Q20. What happens if the certificate hostname does not match?

**Answer:**

The TLS client can reject the certificate or display a certificate/identity warning because it cannot verify that the certificate belongs to the requested hostname.

---

## Q21. What happens if the CA is not trusted?

**Answer:**

The client generally cannot build a trusted chain to a configured trust anchor, so it may reject the certificate or display a trust warning.

---

## Q22. What happens when a certificate expires?

**Answer:**

The certificate fails its validity-period check and should no longer be accepted as valid for authentication.

---

## Q23. What is a certificate chain?

**Answer:**

A certificate chain is a sequence of certificates linking an end-entity certificate through one or more intermediate CAs to a trusted root CA.

---

## Q24. How does HTTPS use certificates?

**Answer:**

During TLS connection establishment, the server presents a certificate. The client validates the certificate's identity, validity, signatures, and trust chain. Once authentication and key establishment are completed, symmetric cryptography protects the application data.

---

## Q25. Why does a firewall need certificates for HTTPS inspection?

**Answer:**

When a firewall performs TLS inspection, it can act as an intermediary and establish separate TLS sessions with the client and destination server. The client must trust the CA used by the firewall to generate/sign inspection certificates; otherwise, certificate warnings occur.

---

# 26. Scenario-Based Interview Questions

## Scenario 1 — Internal web server

**Question:**

You have an internal web server used only by employees. You do not want to purchase a public certificate. What options do you have?

**Answer:**

You can use an organization's internal PKI and issue a certificate from an internal CA. The organization's root CA certificate can be installed in the managed client trust stores.

For a small lab/testing environment, a self-signed certificate can also be used and explicitly trusted by the test clients.

---

## Scenario 2 — Browser certificate warning

**Question:**

A user visits an HTTPS site and receives "Certificate not trusted." What should you check?

**Answer:**

Check:

1. Certificate chain.
2. Trusted root/intermediate CA.
3. Certificate validity period.
4. Hostname/SAN.
5. Certificate signature.
6. Revocation status where applicable.
7. Whether a TLS inspection device is presenting an enterprise certificate that the client does not trust.

---

## Scenario 3 — WatchGuard HTTPS inspection

**Question:**

You enable HTTPS inspection on WatchGuard and users suddenly receive certificate warnings. What is a likely cause?

**Answer:**

The client devices may not trust the CA certificate used by the firewall for TLS inspection. Deploying the appropriate inspection CA certificate to managed client trust stores can resolve the trust problem, assuming the inspection configuration and certificates are otherwise correct.

---

## Scenario 4 — Root CA compromise

**Question:**

Why is protecting a Root CA private key extremely important?

**Answer:**

The root CA is a trust anchor. If its private key is compromised, an attacker could potentially create fraudulent certificates that chain to that trusted root, depending on the PKI design and controls.

---

## Scenario 5 — Self-signed certificate in production

**Question:**

Can you use a self-signed certificate on a production server?

**Answer:**

Technically yes, but whether it is appropriate depends on the environment and trust requirements. It may be acceptable for a tightly controlled internal service where clients explicitly trust the certificate, but public services generally use certificates chaining to publicly trusted CAs.

---

# 27. Quick Revision Sheet

```text
CERTIFICATES
|
+-- Certificate
|   +-- Identity
|   +-- Public key
|   +-- Issuer
|   +-- Validity
|   +-- Signature
|   +-- SAN / extensions
|
+-- PKI
|   +-- CA
|   +-- Certificates
|   +-- Trust stores
|   +-- Validation
|   +-- Revocation
|
+-- CSR
|   +-- Certificate request
|   +-- Public-key information
|   +-- Identity/request information
|   +-- Private key stays with requester
|
+-- Chain of Trust
|   +-- Root CA
|   |    +-- Normally self-signed
|   |    +-- Trust anchor
|   |
|   +-- Intermediate CA
|   |    +-- Signed by CA above it
|   |
|   +-- End-Entity Certificate
|        +-- Server/user/device
|
+-- Self-Signed Certificate
    +-- Signed by its own corresponding private key
    +-- Not automatically publicly trusted
    +-- Useful for labs/testing/private environments
    +-- Can be explicitly trusted
```

---

# 28. Key Takeaways

1. A digital certificate binds an identity to a public key.
2. Certificates are a fundamental part of PKI.
3. A CA signs certificates and provides the trust mechanism.
4. A CSR is a request for certificate issuance.
5. The private key should remain with the certificate requester and should not be sent to the CA as part of normal CSR processing.
6. A certificate contains information such as identity, public key, issuer, validity period, signature, SAN, and key-usage extensions.
7. The Root CA is normally self-signed.
8. A Root CA is a trust anchor when its certificate is installed in a trusted root store.
9. Intermediate CAs create the middle layer between roots and end-entity certificates.
10. The chain of trust normally follows:
    `Root CA -> Intermediate CA -> End-Entity Certificate`
11. A self-signed certificate is not automatically insecure; it simply lacks automatic trust from a public CA chain.
12. Self-signed certificates are useful for labs, development, testing, and controlled internal systems.
13. An internal Root CA can be deliberately installed as a trusted CA in an organization's managed devices.
14. TLS clients validate certificate chains, signatures, validity periods, hostname/SAN, key usage, and potentially revocation.
15. HTTPS uses certificates for server authentication and cryptographic key establishment.
16. WatchGuard HTTPS/TLS inspection relies heavily on certificates and trusted CA configuration.
17. If clients do not trust the inspection CA, TLS inspection can produce certificate warnings.
18. Root CA private keys require strong protection because the root is a trust anchor.

---

# Final Mental Model

> **A certificate says "this public key belongs to this identity," a CA provides the trust behind that statement, and the chain of trust lets a client validate that statement back to a trusted Root CA.**

For WatchGuard specifically:

> **Certificates are essential for HTTPS/TLS, VPNs, authentication, and TLS inspection. Understanding the Root CA → Intermediate CA → End-Entity chain is fundamental to troubleshooting certificate and SSL/TLS inspection problems.**

**WatchGuard Network Basics — Certificates Chapter: COMPLETE**
