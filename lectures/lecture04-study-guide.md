# Lecture 04 — More Public-Key Cryptography
## CS464: Computer System Security — Complete Study Guide
### Certificates, Diffie-Hellman, ElGamal, and PKI

---

## Part 1: Overview and Motivation

Lecture 03 established RSA — a powerful asymmetric cipher capable of both encryption and digital signatures. But simply having public-key cryptography does not automatically solve all the problems of secure communication. Three fundamental questions remain:

1. **Key exchange**: RSA is too slow for bulk encryption. How do two parties establish a shared session key efficiently?
2. **Authentication of public keys**: If A publishes a public key, how does B know it really belongs to A and not an impostor?
3. **Forward secrecy**: If an attacker records your RSA-encrypted traffic today and later obtains your private key, can they decrypt everything? (Yes — unless you use the right protocol.)

This lecture answers all three questions by studying:
- The **Direct Key Exchange Protocol** and why it is broken without authentication.
- **Certificates and PKI**: the infrastructure that authenticates public keys.
- **Diffie-Hellman Key Exchange**: a way to establish a shared secret without either party transmitting it.
- **ElGamal Algorithm**: encryption and digital signatures built on the same math as DH.
- **Perfect Forward Secrecy**: why ephemeral DH (DHE) is better than static RSA for key exchange.

---

## Part 2: RSA for Key Exchange — The Direct Protocol and Its Flaw

### The Intended Protocol

RSA is far too slow for encrypting large messages, but it can be used to exchange a short symmetric **session key**. The idea is:

1. A generates a fresh RSA key pair $[PU_A, PR_A]$ and sends $PU_A$ and $ID_A$ (A's identifier, e.g., IP address) to B in plaintext.
2. B generates a random symmetric session key $K_S$ and sends $E(PU_A, K_S)$ — the session key encrypted with A's public key — back to A.
3. A decrypts with $PR_A$ to recover $K_S$.
4. Both A and B now share $K_S$, which they use for fast symmetric encryption.

This looks clean. But it has a fatal flaw.

### The Man-in-the-Middle Attack

An adversary $V$ who can intercept messages between A and B can completely subvert this protocol:

| Step | Who sends | What | Reality |
|------|-----------|------|---------|
| A → B | A | $PU_A$, $ID_A$ | V intercepts. B never sees it. |
| V → B | **V** | **$PU_V$**, $ID_A$ | B thinks this is A's public key. |
| B → A | B | $E(PU_V, K_S)$ | V intercepts. V decrypts with $PR_V$, gets $K_S$. |
| V → A | **V** | $E(PU_A, K_S)$ | V re-encrypts with the real $PU_A$. A decrypts normally. |

Now both A and B have $K_S$ and believe the exchange was secure — but V also has $K_S$ and can read (and modify) everything. Neither A nor B can detect this.

The fundamental problem: **B has no way to verify that $PU_A$ actually belongs to A**. V successfully impersonated A by substituting their own public key.

### The Root Cause and the Fix

The direct protocol fails because public key distribution is unauthenticated. The fix is **digital certificates**: a trusted third party (Certificate Authority) digitally signs A's public key, cryptographically binding it to A's identity. This is the Certificate Authority model.

---

## Part 3: Certificates and Public Key Infrastructure (PKI)

### What Is a Certificate?

A **digital certificate** is a data structure that binds a public key to an identity. Specifically, a certificate contains:
- The **owner's identity** (name, domain name, email, etc.)
- The **owner's public key**
- The **issuing CA's digital signature** over all the above fields

The signature is created by hashing the certificate's fields with a hash function like SHA-256 and then encrypting the hash with the CA's private key. Anyone who trusts the CA can verify the certificate using the CA's public key.

You might wonder: who verifies the CA? This is answered by the **CA hierarchy**.

### The CA Hierarchy

Certificate Authorities are organized in a strict **tree hierarchy**:
- At the top: **Root CAs** (VeriSign/DigiCert, Comodo, etc.). These are the ultimate trust anchors.
- Below: **Intermediate-level CAs** — verified and certified by root CAs.
- **Registration Authorities (RA)** — resellers/agents that handle administrative tasks but delegate signing to a CA.

Your computer and browser come **pre-loaded with the public keys of trusted root CAs** (check browser settings → security → certificates). This is the foundational trust assumption: you trust the OS/browser vendor to have loaded legitimate root CA keys.

### How Certificate Validation Works

When your browser connects to `https://example.com` and receives a certificate, it:

1. Extracts the CA that signed the certificate.
2. If it is an **intermediate CA**, it checks if it has a higher-level certificate for that intermediate CA from a CA it already trusts.
3. Follows the **chain of trust** recursively upward.
4. Eventually reaches a **root CA certificate** that is pre-loaded in the system.
5. Verifies the entire chain. If every signature in the chain is valid and the root CA is trusted, the certificate is accepted.

The webserver may send the full certificate bundle (its own certificate plus all intermediate CA certificates) to make this chain validation easier for the client.

### Using Certificates for Authentication

A's certificate is: $CA = E(PR_{CA}, [T,\ ID_A,\ PU_A])$

Where $T$ is a timestamp/validity period and the entire structure is signed with the CA's private key. When A sends this certificate to B:
- B uses the CA's public key (already known) to decrypt and verify the signature.
- If valid, B knows that $PU_A$ genuinely belongs to A (as vouched for by the CA).
- B can then safely encrypt with $PU_A$ knowing the key belongs to A.

In practice, most e-commerce uses **one-way authentication**: the server presents its certificate to the client, but the client doesn't present a certificate. Two-way authentication is used in high-security corporate environments.

### Types of Certificates

| Type | Validation | Speed | Cost | Trust Level |
|------|-----------|-------|------|------------|
| DV (Domain Validation) | Domain ownership only | Minutes | Lowest | Basic |
| OV (Organization Validation) | Organization identity verified | Days | Medium | Better |
| EV (Extended Validation) | Thorough legal/organizational check | Days | Highest | Strongest |

EV certificates typically show a green padlock or company name in browsers.

### PKI and X.509

**Public Key Infrastructure (PKI)** is the complete set of standards, policies, hardware, and software for creating, distributing, using, and revoking digital certificates. **X.509** is the most widely used standard specifying the certificate format.

An X.509 certificate's signature field contains the CA's digital signature, computed as:
1. Take a **message digest** (hash with SHA-1 or SHA-256) of all the other certificate fields.
2. Encrypt the digest with the **CA's private key**.

Anyone can verify: compute the same hash of the fields, decrypt the signature with the CA's public key, compare — if they match, the certificate is authentic.

### Certificate Revocation

What if a certificate needs to be invalidated before it expires (e.g., the private key was compromised)?
- **Certificate Revocation List (CRL)**: a list published by the CA of certificate serial numbers that have been revoked. Clients should check this list before trusting a certificate.
- If a **Root CA** is compromised, the only fix is a software update to all devices — affecting billions of endpoints.
- If an **Intermediate CA** is compromised, its certificates are added to the CRL.

### Certificate Forgery

Certificates can be forged if the underlying cryptographic assumption breaks:
- In **2008**, researchers exploited the weak collision resistance of **MD5** to create a forged certificate. (MD5 was still used in some CAs at the time for certificate signatures.)
- In **2011**, a rogue certificate for an intermediate-level CA was created and used for DNS cache poisoning and phishing attacks.

Creating a "fake" root CA with OpenSSL is technically easy. The hard part is getting parties to add it to their trusted root store — but social engineering attacks have achieved this.

---

### Exam Practice — Certificates and PKI (2025 Midterm Q39–Q43)

**Q39:** Which mechanism prevents a man-in-the-middle attack during key exchange?  
a) Using a large modulus $n$  
b) Shared secret key agreement  
c) Encryption combined with digital certificates  
d) Symmetric key distribution

**Answer: (c).** Certificates (verified by a CA) authenticate the public keys being exchanged, preventing the MITM from substituting their own public key. The attack succeeds precisely because the direct protocol uses no certificates. Encryption alone doesn't help if you're encrypting to the wrong key.

---

**Q40:** What is the primary purpose of a Certificate Revocation List (CRL)?  
a) Listing active certificates  
b) Registering new certificates  
c) Identifying certificates that are no longer valid  
d) Rotating encryption keys

**Answer: (c).** A CRL lists certificate serial numbers that have been revoked before their expiration date — due to key compromise, change of ownership, etc. Clients check the CRL before trusting a certificate.

---

**Q41:** What distinguishes a root certificate from other certificates?  
a) It is self-signed and serves as the ultimate trust anchor  
b) It is issued by an RA  
c) It has a shorter validity period  
d) It uses symmetric encryption

**Answer: (a).** A root CA certificate is **self-signed** — it is signed by the root CA's own private key (there is no higher authority to sign it). It is pre-loaded in operating systems and browsers as a trust anchor. Everything else in the PKI chain ultimately traces back to this certificate.

---

**Q42:** What does a digital certificate contain?  
a) The user's private key  
b) Only the CA's signature  
c) A symmetric encryption key  
d) The owner's public key, identity, and the CA's digital signature

**Answer: (d).** A certificate binds a **public key** to an **identity** (name, domain), and this binding is authenticated by the **CA's digital signature** over both pieces of information. The private key is never in a certificate — it stays with the owner.

---

**Q43:** What is the role of a Registration Authority (RA) in PKI?  
a) It generates encryption keys  
b) It verifies the identity of certificate applicants before the CA issues a certificate  
c) It revokes certificates  
d) It stores private keys

**Answer: (b).** An RA acts as a front-end for a CA — it handles the administrative process of verifying the identity of applicants (checking documents, domain ownership, etc.) but delegates the actual certificate signing to the CA. Think of it as the "reseller" or "office" that handles paperwork.

---

### Knowledge Check 2

**Q:** Describe the chain of trust for a website certificate. What happens at each step?  
**A:** The web server presents its certificate (signed by an intermediate CA). The browser extracts the intermediate CA and looks up its certificate (signed by a root CA or another intermediate CA). This continues recursively until a pre-loaded root CA is reached. If every signature in the chain is valid and the root CA is trusted, the certificate is accepted.

**Q:** Why is revoking a root CA certificate essentially impossible at scale?  
**A:** Root CA public keys are pre-loaded in all operating systems, browsers, and devices. Revoking a root CA requires pushing a software update to every device — billions of endpoints worldwide. This is why root CAs are extremely carefully guarded and their private keys are stored in hardware security modules (HSMs) in secure facilities.

---

## Part 4: Diffie-Hellman Key Exchange

### The Motivation: Perfect Forward Secrecy

In the RSA key exchange scenario, if an attacker records your encrypted traffic and later obtains the server's long-term private key $PR_B$, they can retroactively decrypt every session key ever exchanged — breaking the confidentiality of all past sessions.

**Perfect Forward Secrecy (PFS)** is a property of a key exchange protocol that ensures: even if the long-term private keys are compromised in the future, previously established session keys remain secure.

The key insight: if the session key was never transmitted (even encrypted), there is nothing to decrypt retroactively. DH achieves this by having both parties **independently compute** the same shared secret, which is never sent across the network.

### Mathematical Background: The Multiplicative Group $\mathbb{Z}_p^*$

Before explaining the DH protocol, we need some number theory.

For a prime $p$, the set $\{1, 2, 3, \ldots, p-1\}$ forms a **multiplicative group modulo $p$**, denoted $\mathbb{Z}_p^*$. The group operation is multiplication modulo $p$.

**Key definitions:**
- The **order of the group** $\mathbb{Z}_p^*$ is $p - 1$ (the number of elements).
- The **order of an element** $a$ is the smallest positive integer $t$ such that $a^t \equiv 1 \pmod{p}$.
- A **generator** (or **primitive root**) $g$ of a subgroup is an element such that $\{g^0, g^1, g^2, \ldots\} \bmod p$ generates all elements of that subgroup. The subgroup is called **cyclic**.
- For the DH protocol, we choose $g$ such that it generates a subgroup of **large prime order $n$** — specifically, $n$ should be a large prime factor of $p - 1$.

### Example: $p = 17$, $g = 2$

The subgroup generated by $g = 2$ in $\mathbb{Z}_{17}^*$:

| $i$ | $2^i \bmod 17$ |
|-----|--------------|
| 0 | 1 |
| 1 | 2 |
| 2 | 4 |
| 3 | 8 |
| 4 | 16 |
| 5 | 15 |
| 6 | 13 |
| 7 | 9 |
| 8 | $2^8 = 256 \bmod 17 = 1$ (cycle repeats) |

The cyclic subgroup is $\{1, 2, 4, 8, 16, 15, 13, 9\}$ with **order $n = 8$** (a prime factor of $p - 1 = 16$). The triple $(p = 17, g = 2, n = 8)$ is made public.

### The DH Key Exchange Protocol

**Setup (public):** All parties know $p$ (large prime), $g$ (generator), $n$ (order of the subgroup generated by $g$).

**Protocol:**

1. **A** selects a random private key $X_A$ where $1 \leq X_A < n$.
2. **A** computes public key: $Y_A = g^{X_A} \bmod p$ and sends it to B.
3. **B** selects a random private key $X_B$ where $1 \leq X_B < n$.
4. **B** computes public key: $Y_B = g^{X_B} \bmod p$ and sends it to A.
5. **A** computes the shared secret: $K = Y_B^{X_A} \bmod p$
6. **B** computes the shared secret: $K = Y_A^{X_B} \bmod p$

### Why Both Parties Get the Same Key (Proof)

$K$ as computed by A:
$$K_A = Y_B^{X_A} \bmod p = (g^{X_B} \bmod p)^{X_A} \bmod p = g^{X_B \cdot X_A} \bmod p$$

$K$ as computed by B:
$$K_B = Y_A^{X_B} \bmod p = (g^{X_A} \bmod p)^{X_B} \bmod p = g^{X_A \cdot X_B} \bmod p$$

Since $X_A \cdot X_B = X_B \cdot X_A$, both arrive at the same value $K = g^{X_A X_B} \bmod p$. The key was never transmitted — it was independently computed by both parties.

### Worked Example: DH with $p = 17$, $g = 2$, $n = 8$

**A** chooses $X_A = 5$:
$$Y_A = 2^5 \bmod 17 = 32 \bmod 17 = 15$$

**B** chooses $X_B = 7$:
$$Y_B = 2^7 \bmod 17 = 128 \bmod 17 = 9$$

(Note: $128 = 7 \times 17 + 9$, so $128 \bmod 17 = 9$.)

They exchange: A sends 15 to B; B sends 9 to A.

**A** computes:
$$K_A = 9^5 \bmod 17 = 59049 \bmod 17$$
$59049 = 3473 \times 17 + 8$, so $K_A = 8$.

**B** computes:
$$K_B = 15^7 \bmod 17 = 170859375 \bmod 17$$
$170859375 = 10050551 \times 17 + 8$, so $K_B = 8$.

**Shared secret: $K = 8$.**

An eavesdropper sees $\{p = 17, g = 2, n = 8, Y_A = 15, Y_B = 9\}$ but does not see $X_A = 5$ or $X_B = 7$. To recover the secret, they would need to solve: "find $X_A$ such that $2^{X_A} \equiv 15 \pmod{17}$" — this is the **discrete logarithm problem**.

---

### Practice Problems — Diffie-Hellman

**Problem 1:** Using $p = 23$, $g = 5$. A chooses $X_A = 6$, B chooses $X_B = 15$.  
a) Compute $Y_A$ and $Y_B$.  
b) Compute the shared secret $K$ from A's perspective.  
c) Verify from B's perspective.  
d) What does an eavesdropper see?

**Answer:**
a) $Y_A = 5^6 \bmod 23 = 15625 \bmod 23 = 8$. $Y_B = 5^{15} \bmod 23 = 30517578125 \bmod 23 = 19$.  
b) $K_A = 19^6 \bmod 23 = 47045881 \bmod 23 = 2$.  
c) $K_B = 8^{15} \bmod 23 = 35184372088832 \bmod 23 = 2$. ✓  
d) The eavesdropper sees $\{p = 23, g = 5, Y_A = 8, Y_B = 19\}$. To find the secret, they would need to solve $5^{X_A} \equiv 8 \pmod{23}$ for $X_A$ — the discrete logarithm.

---

## Part 5: DH Security — Attacks and the Discrete Logarithm

### The Computational DH Assumption

The security of Diffie-Hellman rests on two related assumptions:

1. **Discrete logarithm is hard**: Given $Y_A = g^{X_A} \bmod p$, computing $X_A$ from $(g, Y_A, p)$ is computationally infeasible for large $p$.

    Formally: $X_A = \log_g Y_A \bmod p$ — this is the **discrete logarithm problem**. Unlike exponentiation (easy), its inverse is believed to be hard.

2. **Computational DH assumption**: Given $(g^a \bmod p)$ and $(g^b \bmod p)$, computing $g^{ab} \bmod p$ is also hard — even if individual discrete logs are not computationally hard.

### DH is Vulnerable to Man-in-the-Middle

Unlike RSA, DH does not inherently provide authentication. An adversary can mount a MITM attack:

1. The adversary intercepts $Y_A$ (sent from A) and replaces it with their own $Y_A'$.
2. The adversary intercepts $Y_B$ (sent from B) and replaces it with their own $Y_B'$.
3. The adversary separately establishes shared secrets $K_1$ with A (using $Y_A$ and $Y_B'$) and $K_2$ with B (using $Y_A'$ and $Y_B$).
4. A thinks $K_1$ is the shared secret with B. B thinks $K_2$ is the shared secret with A. The adversary knows both.
5. The adversary relays messages between A and B, decrypting and re-encrypting as needed.

**Defense: Authenticated DH.** Before DH exchange, each party must verify the other's identity using certificates (public keys certified by a CA). Messages during the DH exchange are digitally signed. This is what **TLS** does.

### DHE Security and the Logjam Vulnerability

**DHE** (Diffie-Hellman Ephemeral) means fresh $X_A$ and $X_B$ values are generated for every session. Old sessions cannot be decrypted even if long-term keys are compromised — this is PFS.

A major weakness in deployed DHE: many servers use the same DH parameters $(p, g, n)$ from RFC 2412 (OAKLEY Key Determination Protocol):
- Group 1: 768-bit prime
- Group 2: 1024-bit prime  
- Group 5: 1536-bit prime

The **Logjam** vulnerability (2015) exploited this: since so many servers used the same 768-bit or 1024-bit primes, an attacker could pre-compute a massive table for one specific prime and then quickly solve discrete logs for all servers using that prime. Solving discrete logs for 768-bit primes is within reach for researchers; 1024-bit primes are within reach for nation-state-level attackers.

---

### Knowledge Check 3

**Q:** Why does DH provide Perfect Forward Secrecy, but static RSA key exchange does not?  
**A:** With DHE, fresh private values $X_A$ and $X_B$ are generated for every session and discarded after use. No record of them exists. Even if the long-term private key is later compromised, there is nothing to decrypt — the session key was never transmitted and the ephemeral DH values are gone. With RSA, the session key $K_S$ was transmitted as $E(PU_B, K_S)$. If $PR_B$ is later obtained, the attacker can decrypt $K_S$ from the recorded ciphertext.

**Q:** An eavesdropper sees $g = 5$, $p = 23$, $Y_A = 8$, $Y_B = 19$ from a DH exchange. What would they need to compute the shared secret?  
**A:** They need to solve the discrete logarithm — find $X_A$ such that $5^{X_A} \equiv 8 \pmod{23}$, then compute $K = Y_B^{X_A} \bmod 23$. For large primes, this is computationally infeasible.

---

## Part 6: ElGamal — Encryption Based on DH

### The ElGamal Encryption Protocol

ElGamal turns the Diffie-Hellman key exchange mechanism into an encryption scheme. Unlike DH (which is interactive — both parties need to be online), ElGamal allows B to encrypt a message to A using A's fixed public key, without A being online.

**Setup:**
- A has a long-term private key $X_A$ and public key $Y_A = g^{X_A} \bmod p$.
- The tuple $(p, g, n, Y_A)$ is A's public key.

**Encryption (by B):**
1. B chooses a fresh random number $X_B$ (ephemeral) and computes $Y_B = g^{X_B} \bmod p$.
2. B computes the shared secret $K = Y_A^{X_B} \bmod p$.
3. B encrypts the message: $C = M \times K \bmod p$.
4. B sends $(Y_B, C)$ to A.

**Decryption (by A):**
1. A computes $K = Y_B^{X_A} \bmod p$ (same shared secret, since $Y_A^{X_B} = g^{X_A X_B} = Y_B^{X_A}$).
2. A recovers the message: $M = C \times K^{-1} \bmod p$.

Note that each encryption uses a fresh $X_B$, so the scheme provides semantic security (encrypting the same message twice gives different ciphertexts).

---

## Part 7: ElGamal Digital Signature Standard (DSS)

### What Is a Digital Signature?

A digital signature scheme provides:
- **Authentication**: the signature could only have been created by the party holding the private key.
- **Integrity**: any modification to the signed message invalidates the signature.
- **Non-repudiation**: the signer cannot later deny having signed the message.

The process:
1. Compute a **hash** of the message (e.g., using SHA-256).
2. **Encrypt the hash with the signer's private key** — this is the signature.
3. Send the message and signature together.

Verification:
1. Use the signer's **public key** to decrypt the signature and recover the hash.
2. Independently compute the hash of the received message.
3. If both hashes match, the signature is valid.

### ElGamal DSS — Key Setup

The ElGamal Digital Signature Standard is a NIST standard that uses the same mathematical structure as DH but for signatures.

**Key Generation:**
1. Select a large prime $p$ and random integers $g < p$ and $X < p$. ($X$ is the private key.)
2. Compute $Y = g^X \bmod p$. ($Y$ is the public key.)
3. Public key: $(p, g, Y)$. Private key: $X$.

### Signing a Message

To sign a message $M$ (as an integer):
1. Generate a **fresh random number $K$** such that $0 < K < p-1$ and $\gcd(K, p-1) = 1$. This $K$ must be unique per signature — reusing it leaks the private key.
2. Compute: $\text{sig}_1 = g^K \bmod p$
3. Compute: $\text{sig}_2 = K^{-1} \times (M - X \times \text{sig}_1) \bmod (p-1)$

The signature is the pair $(\text{sig}_1, \text{sig}_2)$.

### Verifying a Signature

To verify that $(M, \text{sig}_1, \text{sig}_2)$ is a valid signature from the holder of private key $X$:
$$Y^{\text{sig}_1} \times \text{sig}_1^{\text{sig}_2} \equiv g^M \pmod{p}$$

If this congruence holds, the signature is valid.

**Why this works (proof sketch):** If the signature equations hold: $\text{sig}_2 = K^{-1}(M - X \cdot \text{sig}_1) \bmod (p-1)$, then $K \cdot \text{sig}_2 = M - X \cdot \text{sig}_1 \bmod (p-1)$, so $M = X \cdot \text{sig}_1 + K \cdot \text{sig}_2 \bmod (p-1)$. By Fermat's little theorem, $g^M \equiv g^{X \cdot \text{sig}_1 + K \cdot \text{sig}_2} = (g^X)^{\text{sig}_1} \cdot (g^K)^{\text{sig}_2} = Y^{\text{sig}_1} \cdot \text{sig}_1^{\text{sig}_2} \pmod{p}$.

### Critical Security Note: Never Reuse $K$

The random $K$ used in signing must be unique for every signature. If the same $K$ is used to sign two different messages $M_1$ and $M_2$, an attacker can solve for $X$ (the private key) from the two signature equations. This is not a theoretical concern — the Sony PlayStation 3 was hacked in 2010 precisely because their firmware signing implementation reused the same $K$ for every signature.

---

### Exam Practice — DH and ElGamal (2025 Midterm Q44–Q48)

**Q44:** Which of the following best describes the primary use of the Diffie-Hellman algorithm?  
a) Key exchange  
b) Non-repudiation  
c) Digital signatures  
d) Password storage

**Answer: (a).** DH is fundamentally a **key exchange** (or key agreement) protocol — it allows two parties to establish a shared secret over an insecure channel. It does not provide non-repudiation because it lacks authentication (DH is vulnerable to MITM), so (b) is incorrect. Signing uses ElGamal DSS or similar, not plain DH, so (c) is incorrect.

> **"Questions of Doubt" Note:** This question was disputed. The answer is definitively **(a) key exchange**. DH does not provide non-repudiation — it does not authenticate who you are talking to, which is why it is vulnerable to MITM attacks. Digital signatures require a mechanism that proves identity, which vanilla DH does not.

---

**Q45:** What property makes DH secure against passive eavesdroppers?  
a) The hardness of computing discrete logarithms  
b) The difficulty of factoring large integers  
c) The use of symmetric encryption  
d) The randomness of the key size

**Answer: (a).** An eavesdropper sees $Y_A = g^{X_A} \bmod p$ and $Y_B = g^{X_B} \bmod p$. To compute the shared secret $K = g^{X_A X_B} \bmod p$, they need either $X_A$ or $X_B$ — which requires solving the discrete logarithm $X_A = \log_g Y_A \bmod p$. This is computationally infeasible for large $p$. (b) is the basis of RSA, not DH.

---

**Q46:** What advantage does DHE (Diffie-Hellman Ephemeral) provide over static RSA key exchange?  
a) Faster computation  
b) Larger key sizes  
c) Perfect Forward Secrecy  
d) Better hash functions

**Answer: (c).** DHE generates fresh private values $X_A$ and $X_B$ for every session and discards them afterwards. Even if the server's long-term private key is later compromised, there is no way to recover past session keys — they were never transmitted and the ephemeral values no longer exist. Static RSA key exchange transmits $K_S$ encrypted with a long-term key, making all past sessions vulnerable if that key is later compromised.

---

**Q47:** What type of cryptographic algorithm is the ElGamal DSS?  
a) Symmetric encryption  
b) Hash function  
c) Stream cipher  
d) Asymmetric encryption/signature scheme

**Answer: (d).** ElGamal DSS is an **asymmetric** (public-key) algorithm. It uses a public key $(p, g, Y)$ for verification and a private key $X$ for signing. The security is based on the discrete logarithm problem, same as DH.

---

**Q48:** What is the purpose of the ElGamal signature scheme?  
a) To encrypt large files  
b) To verify the authenticity and integrity of a message  
c) To generate symmetric keys  
d) To replace AES

**Answer: (b).** A digital signature proves that the message came from the claimed sender (authenticity) and has not been altered (integrity). Anyone with the signer's public key can verify the signature, but only the holder of the private key can create it. ElGamal is not used for bulk file encryption (that's symmetric ciphers like AES).

---

## Part 8: The Discrete Logarithm Problem

### The Problem Formally

Given integers $g$, $k$, and prime $p$, find $s$ such that:
$$g^s \equiv k \pmod{p}$$

This is the **discrete logarithm** of $k$ base $g$ modulo $p$. The brute-force approach — trying all values of $s$ — requires $O(p)$ steps. If $p$ requires $n$ bits, the complexity is $O(2^n)$, which grows exponentially.

### Known Algorithms

Brute force is not the only option. More sophisticated algorithms exist:
- **Baby-step giant-step**: reduces complexity to $O(\sqrt{p})$ — still exponential in bit size but much better than brute force.
- **Pollard's rho algorithm**: similar asymptotic complexity to baby-step/giant-step.
- **Index calculus methods**: subexponential in the size of $p$ (analogous to number field sieve for factoring).

For the discrete log problem in $\mathbb{Z}_p^*$, the best algorithms are subexponential but not polynomial. Solving the discrete log for 768-bit primes is now feasible for researchers; 1024-bit primes are within reach for nation-state attackers (as demonstrated by the Logjam attack). The recommendation is to use at least 2048-bit primes for DH.

### Connection to DH and ElGamal Security

If the discrete logarithm could be computed efficiently, an eavesdropper seeing $Y_A = g^{X_A} \bmod p$ could compute $X_A$ and break both DH and ElGamal. The security of both protocols is directly tied to the hardness of this problem.

---

## Part 9: Summary and Connections

### Where We've Been

| Concept | Lecture | Key Idea |
|---------|---------|----------|
| AES | 01 | Fast symmetric cipher; key distribution problem |
| Block cipher modes | 02 | How to apply a block cipher to long messages |
| RSA | 03 | Asymmetric cipher based on integer factoring |
| Certificates / PKI | 04 | How to authenticate public keys |
| Diffie-Hellman | 04 | Key exchange without transmitting the key |
| ElGamal | 04 | Encryption and signatures based on DH math |

### The Big Picture of a Real Secure Connection (TLS)

A modern TLS 1.3 handshake uses all of these concepts:
1. **DHE**: Both parties exchange ephemeral DH public values. Neither the session key nor the DH private values are ever transmitted or stored.
2. **Certificates**: The server sends an X.509 certificate signed by a trusted CA. The client verifies the certificate chain.
3. **Authentication**: The server's certificate binds its DH public value to its verified identity, preventing MITM.
4. **Session key**: The DH shared secret is used to derive AES keys for encrypting the actual data.
5. **PFS**: Because DHE is used, all past sessions are secure even if the server's long-term key is later compromised.

---

## Part 10: Final Exam Preparation — Lecture 04

### Relevant 2025 Midterm Questions — Lecture 04 Index

| Question | Topic | Location in Guide |
|----------|-------|------------------|
| Q39 | MITM prevention — certificates | Part 3 |
| Q40 | CRL purpose | Part 3 |
| Q41 | Root certificate (self-signed, trust anchor) | Part 3 |
| Q42 | Digital certificate contents | Part 3 |
| Q43 | RA role in PKI | Part 3 |
| Q44 | DH primary use — key exchange | Part 7 |
| Q45 | DH security — discrete logarithm | Part 5 |
| Q46 | DHE advantage — PFS | Part 5 |
| Q47 | ElGamal type — asymmetric | Part 7 |
| Q48 | ElGamal purpose — authenticity/integrity | Part 7 |

### Summary of "Questions of Doubt" Clarifications — Lecture 04

| Question | Clarification |
|----------|--------------|
| Q44 | DH is for **key exchange** (a), not non-repudiation. DH is vulnerable to MITM and does not authenticate, so it cannot provide non-repudiation. |

### Comprehensive Review Questions

**1.** Explain why the Direct Key Exchange Protocol (without certificates) is vulnerable to MITM. Be specific about which step V exploits and why A and B cannot detect the attack.  
**Answer:** A sends $PU_A$ in plaintext. V intercepts and substitutes $PU_V$. B encrypts the session key with $PU_V$ instead of $PU_A$. V decrypts, re-encrypts with $PU_A$, and forwards. A decrypts normally. Neither A nor B detects anything wrong because the protocol has no mechanism to verify that the public key actually belongs to the claimed party. The fix is certificates — the public key is bundled with an identity and signed by a CA, so substituting $PU_V$ would fail signature verification.

**2.** Walk through the DH exchange with $p = 11$, $g = 2$, $X_A = 3$, $X_B = 7$. Show all computations and verify both parties arrive at the same key.  
**Answer:**
- $Y_A = 2^3 \bmod 11 = 8$
- $Y_B = 2^7 \bmod 11 = 128 \bmod 11 = 7$ (since $128 = 11 \times 11 + 7$)
- $K_A = Y_B^{X_A} \bmod 11 = 7^3 \bmod 11 = 343 \bmod 11 = 2$ (since $343 = 31 \times 11 + 2$)
- $K_B = Y_A^{X_B} \bmod 11 = 8^7 \bmod 11 = 2097152 \bmod 11 = 2$ (since $2097152 = 190650 \times 11 + 2$)
- Shared secret: $K = 2$ ✓

**3.** A server uses the same DH parameters for years without regenerating. What attack does this enable, and why does DHE prevent it?  
**Answer:** Static DH means if the server's long-term private key $X_A$ is ever compromised, all session keys $K = Y_B^{X_A} \bmod p$ from all past sessions can be recomputed (since $Y_B$ values were exchanged in public). An attacker who stored all past traffic can now decrypt it retroactively. DHE prevents this because a fresh $X_A$ is generated per session and immediately discarded. Even if a past $X_A$ could somehow be obtained, it only compromises that one session. Future and past sessions with different ephemeral $X_A$ values remain secure.

**4.** What is the difference between ElGamal encryption and ElGamal DSS? When would you use each?  
**Answer:** ElGamal **encryption** uses A's long-term public key $Y_A$ to allow B to encrypt a message to A. B generates a fresh $X_B$, computes shared secret $K = Y_A^{X_B} \bmod p$, and encrypts as $C = M \times K$. Used when you want **confidentiality** — only A can decrypt. ElGamal **DSS** allows A to use their private key $X$ to sign a message. Anyone with the public key $(p, g, Y)$ can verify the signature. Used when you want **authenticity and integrity** — proving the message came from A and hasn't been tampered with.

---

*End of Lecture 04 Study Guide*  
*Next: Lecture 05 — (check your syllabus for upcoming topics)*
