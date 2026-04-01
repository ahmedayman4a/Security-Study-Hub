# Lecture 03 — Public-Key Cryptography & RSA
## CS464: Computer System Security — Complete Study Guide

---

## Part 1: The Problem With Symmetric Cryptography

Every symmetric cipher we have studied — DES, 3DES, AES — shares a fundamental assumption: both parties already have the same secret key. But how do two parties who have never communicated before establish that shared key? If they want to use AES to talk securely, they first need to get the same 128-bit key — but sharing it over an insecure channel is precisely the problem they are trying to solve.

This is the **key distribution problem**, and it plagued cryptography for centuries. Symmetric cryptography provides powerful encryption once a key is established, but it offers no clean answer to how two strangers on the internet can begin communicating securely.

**Public-key cryptography** (also called **asymmetric cryptography**) solves this by using a mathematically related *pair* of keys: one public, one private. What one key encrypts, only the other can decrypt — and the two keys cannot practically be derived from each other. This asymmetry enables completely new protocols that symmetric cryptography cannot provide.

---

## Part 2: Public-Key Cryptography — The Big Picture

### Key Pairs

In public-key cryptography, every party has a **key pair**:
- A **public key** (denoted $PU$): published openly to the world. Anyone can have it.
- A **private key** (denoted $PR$): kept secret. Only the owner knows it.

The two keys are mathematically linked but the link is computationally one-way: knowing the public key does not let you compute the private key (assuming the underlying hard mathematical problem is indeed hard).

You might wonder: why does publishing the public key not compromise security? Because the public key is used for *locking* (encryption or signature verification), and only the private key can *unlock* (decrypt or sign). An analogy: imagine a padlock that anyone can snap shut (public key), but only you have the key to open (private key).

### Public-Key Services

Public-key cryptography provides three fundamental services that symmetric cryptography cannot easily provide:

**1. Confidentiality (from A to B):**  
B publishes $PU_B$. A encrypts the message with B's public key. Only B — who holds $PR_B$ — can decrypt it.

$$C = E(PU_B,\ M) \qquad M = D(PR_B,\ C)$$

**2. Digital Signature / Authentication / Non-repudiation (from A to B):**  
A signs the message with its own *private* key. Anyone with A's public key can verify the signature — but only A could have created it (since only A has $PR_A$). This proves the message came from A (authentication) and that A cannot later deny sending it (non-repudiation).

$$C = E(PR_A,\ M) \qquad M = D(PU_A,\ C)$$

**3. Confidentiality + Digital Signature:**  
A signs first (with $PR_A$), then B encrypts the signed message (with $PU_B$). B decrypts first, then verifies the signature.

$$C = E(PU_B,\ E(PR_A,\ M)) \qquad M = D(PU_A,\ D(PR_B,\ C))$$

### Does Public-Key Cryptography Replace Symmetric Cryptography?

No — and this is a common misconception. Public-key cryptography is **computationally expensive**: encrypting a large file with RSA is orders of magnitude slower than AES. In practice, public-key cryptography is used for two narrow but critical tasks:
1. **Authentication**: proving who you are (digital signatures).
2. **Key exchange**: securely establishing a symmetric session key (which then encrypts the actual data with AES or similar).

Real protocols (TLS/HTTPS) do exactly this: RSA or Diffie-Hellman establishes a shared session key, and AES with that key encrypts the actual traffic.

---

### Knowledge Check 1

**Q:** What fundamental problem does public-key cryptography solve that symmetric cryptography cannot?  
**A:** The key distribution problem — how two parties who have never communicated establish a shared secret key over an insecure channel.

**Q:** If A encrypts with $PR_A$ (the private key), what is this used for?  
**A:** Digital signature / authentication. It proves the message came from A, since only A has $PR_A$. Anyone with $PU_A$ can decrypt and verify.

**Q:** Why is public-key cryptography not used for bulk data encryption?  
**A:** It is computationally expensive — many orders of magnitude slower than symmetric ciphers. It is used only for authentication and key exchange.

---

## Part 3: RSA — Mathematical Foundation

### Why RSA Works: Euler's Theorem

RSA is built on a result from number theory. **Euler's theorem** states:

For every positive integer $n$ and every integer $a$ that is **coprime** to $n$ (meaning $\gcd(a, n) = 1$):

$$a^{\phi(n)} \equiv 1 \pmod{n}$$

where $\phi(n)$ is **Euler's totient function** — the count of integers from 1 to $n$ that are coprime to $n$.

A useful consequence: if $k = k_1 \cdot \phi(n) + k_2$, then:

$$a^k \equiv a^{k_1 \cdot \phi(n)} \cdot a^{k_2} \equiv (a^{\phi(n)})^{k_1} \cdot a^{k_2} \equiv 1^{k_1} \cdot a^{k_2} \equiv a^{k_2} \pmod{n}$$

In plain English: an exponent that is a multiple of $\phi(n)$ contributes nothing — it cancels out.

### The RSA Insight: Multiplicative Inverses Mod $\phi(n)$

Now suppose we find two integers $e$ and $d$ such that:

$$e \cdot d \equiv 1 \pmod{\phi(n)}$$

This means $e$ and $d$ are **multiplicative inverses modulo $\phi(n)$**, so $e \cdot d = 1 + k \cdot \phi(n)$ for some integer $k$.

Apply this to a message $M$ (coprime to $n$):

$$M^{e \cdot d} = M^{1 + k \cdot \phi(n)} = M \cdot (M^{\phi(n)})^k \equiv M \cdot 1^k \equiv M \pmod{n}$$

So encrypting with $e$ (raising to the power $e$ mod $n$) and then decrypting with $d$ (raising to the power $d$ mod $n$) recovers the original message exactly. This is the mathematical core of RSA.

### The Role of $n = p \cdot q$

When $n$ is the product of two distinct primes $p$ and $q$, the totient has a simple closed form:

$$\phi(n) = \phi(p) \cdot \phi(q) = (p-1)(q-1)$$

This is important because: Rivest, Shamir, and Adleman proved that the property $M^{e \cdot d} \equiv M \pmod{n}$ holds for **all** $M$ in the range $0 \leq M < n$ when $n = pq$ — not just for $M$ coprime to $n$. This ensures RSA works for any message.

The security relies on this: multiplying $p \times q$ to get $n$ is easy. Reversing it — factoring $n$ back into $p$ and $q$ — is computationally infeasible for large primes.

---

## Part 4: RSA Key Generation

### The Complete Procedure

RSA key generation follows these steps in order:

**Step 1 — Choose two large distinct primes $p$ and $q$.**  
These must be generated randomly and kept secret forever. If $n$ must be $B$ bits, each prime is roughly $B/2$ bits.

**Step 2 — Compute the modulus $n$.**

$$n = p \times q$$

**Step 3 — Compute the totient $\phi(n)$.**

$$\phi(n) = (p - 1)(q - 1)$$

**Step 4 — Choose the public exponent $e$.**  
Select $e$ such that:

$$1 < e < \phi(n) \quad \text{and} \quad \gcd(e,\ \phi(n)) = 1$$

Typical values: **3**, 17, and **65537** ($= 2^{16} + 1$). The value 65537 is strongly preferred in practice — it is prime, large enough to avoid cube-root attacks (unlike 3), and has only two bits set in its binary representation ($10000000000000001_2$) which makes exponentiation very fast.

$e = 3$ is considered cryptographically weak: for a short message $M$, $M^3$ might be smaller than $n$, in which case $C = M^3$ (not $M^3 \bmod n$) and the attacker simply takes the cube root.

**Step 5 — Compute the private exponent $d$.**

$$d = e^{-1} \bmod \phi(n)$$

$d$ is the modular multiplicative inverse of $e$ modulo $\phi(n)$, computed using the **Extended Euclidean Algorithm**. It satisfies $e \cdot d \equiv 1 \pmod{\phi(n)}$.

**Step 6 — Publish the public key and secure the private key.**

$$\text{Public key} = [e,\ n] \qquad \text{Private key} = [d,\ n]$$

Note: $\phi(n)$ is also kept secret — knowing $\phi(n)$ is essentially equivalent to knowing $p$ and $q$.

### How Primes Are Generated in Practice

To generate a large prime $p$ of $B/2$ bits:
1. Use a random number generator to produce a random $B/2$-bit number.
2. Set the lowest bit to 1 (ensuring the number is odd — all primes greater than 2 are odd).
3. Set the two highest bits to 1 (ensuring the number has the right bit-length and that $p \times q$ will have the full $B$ bits).
4. Apply the **Miller-Rabin primality test** — a probabilistic algorithm that is extremely accurate with repeated applications.
5. If not prime, increment by 2 and test again. By the prime number theorem, primes are dense enough that a suitable prime is found quickly.

### Choosing the Public Exponent $e$

The condition $\gcd(e, \phi(n)) = 1$ is equivalent to requiring:
- $\gcd(e, p-1) = 1$ — equivalently $p \bmod e \neq 1$
- $\gcd(e, q-1) = 1$ — equivalently $q \bmod e \neq 1$

If a specific value of $e$ is preferred (say, $e = 65537$), one simply checks both conditions when generating $p$ and $q$ and rejects any prime that fails them.

---

## Part 5: RSA Encryption and Decryption

With the key pair established:

### Encryption
$$C = M^e \bmod n$$

### Decryption
$$M = C^d \bmod n$$

These are both **modular exponentiation** operations. The mathematical correctness follows from Euler's theorem as derived in Part 3: $C^d = (M^e)^d = M^{ed} \equiv M \pmod{n}$.

### Why Decryption is Slower Than Encryption

$e$ is chosen to be small (e.g., 65537) with few set bits — exponentiation with $e$ is fast. $d$, however, is computed as a modular inverse of $e$ and is roughly the same size as $n$ (hundreds or thousands of bits). Computing $C^d \bmod n$ is significantly slower than computing $M^e \bmod n$.

---

## Part 6: Worked Example — RSA from Scratch

Let us work through RSA step by step with a small toy example from the lecture (16-bit modulus, 8-bit primes).

**Given:** $p = 197$, $q = 211$

**Step 1: Compute $n$:**
$$n = 197 \times 211 = 41{,}567$$

**Step 2: Compute $\phi(n)$:**
$$\phi(n) = (197 - 1)(211 - 1) = 196 \times 210 = 41{,}160$$

**Step 3: Choose $e$:**  
We need $\gcd(e, 41160) = 1$. Try:
- $e = 3$: $\gcd(3, 210) = 3 \neq 1$. No.
- $e = 5$: $\gcd(5, 210) = 5 \neq 1$. No.
- $e = 17$: $\gcd(17, 196) = 1$ ✓ and $\gcd(17, 210) = 1$ ✓. **Use $e = 17$.**

**Step 4: Compute $d$:**  
We need $17 \cdot d \equiv 1 \pmod{41160}$.  
Using Extended Euclidean Algorithm: $d = 26{,}633$.  
Verify: $17 \times 26633 = 452761 = 11 \times 41160 + 1$. ✓

**Public key:** $[e = 17,\ n = 41567]$  
**Private key:** $[d = 26633,\ n = 41567]$

### Practice Encryption/Decryption (Small Numbers)

To practice, use the standard exam-scale example: $p = 3$, $q = 11$.

$$n = 33,\quad \phi(n) = 2 \times 10 = 20$$

Choose $e = 7$: $\gcd(7, 20) = 1$ ✓.  
Compute $d$: $7d \equiv 1 \pmod{20}$ → $d = 3$ (since $7 \times 3 = 21 \equiv 1 \pmod{20}$). ✓

Encrypt $M = 2$:
$$C = 2^7 \bmod 33 = 128 \bmod 33 = 128 - 3 \times 33 = 128 - 99 = 29$$

Decrypt $C = 29$:
$$M = 29^3 \bmod 33 = 24389 \bmod 33$$
$24389 \div 33 = 739.06\ldots$, so $739 \times 33 = 24387$, $24389 - 24387 = 2$. ✓

$M = 2$ — the original message is recovered.

---

### Practice Problems — RSA Key Generation

**Problem 1:** Given $p = 5$, $q = 11$. Find $n$, $\phi(n)$, choose a valid $e$, and compute $d$.  
**Answer:**
- $n = 55$
- $\phi(n) = 4 \times 10 = 40$
- $e = 3$: $\gcd(3, 40) = 1$ ✓
- $d$: $3d \equiv 1 \pmod{40}$ → $d = 27$ (since $3 \times 27 = 81 = 2 \times 40 + 1$) ✓

**Problem 2:** Given $p = 7$, $q = 13$. Find $n$, $\phi(n)$, verify $e = 5$ works, compute $d$. Then encrypt $M = 3$.  
**Answer:**
- $n = 91$, $\phi(n) = 6 \times 12 = 72$
- $e = 5$: $\gcd(5, 72) = 1$ ✓
- $d$: $5d \equiv 1 \pmod{72}$ → $d = 29$ (since $5 \times 29 = 145 = 2 \times 72 + 1$) ✓
- Encrypt: $C = 3^5 \bmod 91 = 243 \bmod 91 = 243 - 2 \times 91 = 61$

---

## Part 7: Fast Modular Exponentiation

### Why We Need a Special Algorithm

RSA requires computing $M^e \bmod n$ where $e$ is a large integer (even with $e = 65537$, that's 17 bits — meaning up to $2^{65537}$ operations naively). Computing $M^e$ directly and then reducing mod $n$ is impossible — $M^e$ would have an astronomically large number of digits.

The key insight is that we can **reduce modulo $n$ at every multiplication step**, keeping the numbers manageable throughout. Combined with **repeated squaring**, this makes modular exponentiation efficient.

### The Algorithm (Square-and-Multiply)

Express the exponent $B$ in binary: $B = (b_k b_{k-1} \ldots b_1 b_0)_2$

Then:
$$A^B = A^{\sum_{b_i \neq 0} 2^i} = \prod_{b_i \neq 0} A^{2^i}$$

And since $(AB \bmod n) = ((A \bmod n)(B \bmod n)) \bmod n$:
$$A^B \bmod n = \prod_{b_i \neq 0} \left[A^{2^i} \bmod n\right] \bmod n$$

**Algorithm:**
```
result = 1
base = A mod n
for each bit b of B from least significant to most significant:
    if b == 1:
        result = (result × base) mod n
    base = (base × base) mod n
return result
```

This requires at most $2 \log_2 B$ multiplications (one squaring per bit, one multiplication per set bit), all with numbers at most $n^2$ in size.

### Worked Example: $7^{11} \bmod 13$

$11 = (1011)_2 = 2^0 + 2^1 + 2^3$

| Step | Bit | Base $= 7^{2^i} \bmod 13$ | Multiply into result? |
|------|-----|---|----|
| $i=0$ | 1 | $7^1 = 7$ | result = $1 \times 7 = 7$ |
| $i=1$ | 1 | $7^2 = 49 \bmod 13 = 10$ | result = $7 \times 10 = 70 \bmod 13 = 5$ |
| $i=2$ | 0 | $10^2 = 100 \bmod 13 = 9$ | skip |
| $i=3$ | 1 | $9^2 = 81 \bmod 13 = 3$ | result = $5 \times 3 = 15 \bmod 13 = 2$ |

$7^{11} \bmod 13 = 2$ (verify: $7^{11} = 1977326743$; $1977326743 \bmod 13 = 2$ ✓)

---

## Part 8: RSA-CRT — Speeding Up Decryption

### The Problem

RSA decryption requires $M = C^d \bmod n$. Since $d$ is roughly the same size as $n$ (often 2048 or more bits), and modular exponentiation costs $O(\log d)$ multiplications each involving numbers up to $n$, decryption can be slow.

The **Chinese Remainder Theorem (CRT)** provides a ~4x speedup by splitting the computation into two smaller modular exponentiations using $p$ and $q$ separately (which are half the size of $n$).

### The CRT Decryption Formula

During decryption, $p$ and $q$ are known (they are the private key's secret factors). Compute:

$$V_p = C^d \bmod p \qquad V_q = C^d \bmod q$$

Then combine using:
$$X_p = q \cdot (q^{-1} \bmod p) \qquad X_q = p \cdot (p^{-1} \bmod q)$$

$$M = C^d \bmod n = (V_p \cdot X_p + V_q \cdot X_q) \bmod n$$

**Further speedup via Fermat's Little Theorem:** Since $p$ is prime, $a^{p-1} \equiv 1 \pmod{p}$ for $a$ not divisible by $p$. So $C^d \bmod p$ can be reduced: write $d = u \cdot (p-1) + v$, then $C^d \equiv C^v \pmod{p}$. This makes the exponents even smaller.

Combined, CRT + FLT reduces the cost to roughly **one-quarter** of naive $C^d \bmod n$ decryption.

**Important:** A step that is NOT part of RSA-CRT decryption is **factoring $n$ dynamically** — $p$ and $q$ are already known to the key holder. They are part of the private key. An attacker who doesn't know $p$ and $q$ cannot use CRT at all.

---

### Exam Practice — RSA Key Generation and Operations
*(2025 Midterm Q30–Q38)*

**Q30:** What is the foundation of RSA's security?  
a) The difficulty of factoring large integers  
b) The hardness of solving discrete logarithms  
c) The complexity of hash collisions  
d) The secrecy of its S-boxes

**Answer: (a).** RSA security relies on the difficulty of factoring the modulus $n = pq$ back into its prime factors $p$ and $q$. Given $n$, computing $\phi(n) = (p-1)(q-1)$ and thus $d$ requires knowing $p$ and $q$. (b) is the basis of Diffie-Hellman. (d) is a property of block ciphers.

---

**Q31:** What is the correct order for RSA key generation?  
a) Select $e$ → Choose $n$ → Factor $n$ → Compute $d$  
b) Generate $d$ → Find $e$ → Randomize $n$ → Pick $p, q$  
c) Compute $n$ → Solve for $p, q$ → Derive $e$ → Choose $d$  
d) Choose primes $p, q$ → Compute $n = pq$ → Select $e$ → Calculate $d$

**Answer: (d).** The correct order: choose primes $p$ and $q$ → compute $n = pq$ and $\phi(n) = (p-1)(q-1)$ → select $e$ coprime to $\phi(n)$ → compute $d = e^{-1} \bmod \phi(n)$.

---

**Q32:** In RSA, what condition must the public exponent $e$ satisfy?  
a) $e > n$  
b) $e$ must be prime  
c) $1 < e < \phi(n)$ and $\gcd(e, \phi(n)) = 1$  
d) $e \equiv 1 \bmod p$

**Answer: (c).** $e$ must be in the range $(1, \phi(n))$ and coprime to $\phi(n)$ — this guarantees that $d = e^{-1} \bmod \phi(n)$ exists (a multiplicative inverse exists only when $\gcd(e, \phi(n)) = 1$). $e$ need not be prime (though typical choices like 65537 happen to be prime).

---

**Q33:** How is the private exponent $d$ computed?  
a) $d \equiv e^{-1} \bmod \phi(n)$  
b) $d = n$  
c) $d = e \bmod p$  
d) $d$ is randomly selected

**Answer: (a).** $d$ is the modular multiplicative inverse of $e$ modulo $\phi(n)$, computed using the Extended Euclidean Algorithm.

---

**Q34:** Which RSA operation is computationally expensive?  
a) Decryption (using $d$)  
b) Encryption (using $e$)  
c) Generating $p$ and $q$  
d) Calculating $\gcd$

**Answer: (a).** Decryption requires computing $C^d \bmod n$. Since $d$ is as large as $n$ (typically 2048+ bits), this is slow. Encryption with a small $e$ (like 65537) is much faster. CRT can speed up decryption ~4x.

---

**Q35:** Why is RSA rarely used for bulk encryption?  
a) Lack of authentication  
b) Slow performance  
c) Fixed block size  
d) Inability to encrypt large files

**Answer: (b).** RSA's modular exponentiation is orders of magnitude slower than symmetric encryption. In practice, RSA is used only for authentication and key exchange; the actual data is encrypted with a fast symmetric cipher (AES) using a session key established via RSA.

---

**Q36:** Which vulnerability arises from reusing the same $n$ with different $e$?  
a) Nonce reuse  
b) Timing attack  
c) Padding oracle attack  
d) Common modulus attack

**Answer: (d).** If two parties use the same modulus $n$ with different public exponents $e_1$ and $e_2$ (and $\gcd(e_1, e_2) = 1$), and the same message $M$ is encrypted with both keys, an attacker can recover $M$ using the **common modulus attack**: compute $e_1 \cdot a + e_2 \cdot b = 1$ (Extended Euclidean), then $C_1^a \cdot C_2^b \equiv M^{e_1 a + e_2 b} = M^1 = M \pmod{n}$.

---

**Q37:** What is the primary purpose of using CRT with RSA?  
a) To increase the key size  
b) To enable quantum resistance  
c) To replace modular exponentiation  
d) To speed up decryption/signature generation by ~4x

**Answer: (d).** CRT splits the decryption $C^d \bmod n$ into two smaller computations $C^d \bmod p$ and $C^d \bmod q$, then combines via CRT. Since $p$ and $q$ are ~half the size of $n$, exponentiation with them is much faster. Combined with Fermat's Little Theorem, the speedup is approximately 4x.

---

**Q38:** Which step is NOT part of RSA-CRT decryption?  
a) Compute $m_p = C^{d_p} \bmod p$ and $m_q = C^{d_q} \bmod q$  
b) Factor $n$ into $p$ and $q$ dynamically  
c) Recombine results via CRT  
d) Verify the result modulo $n$

**Answer: (b).** The private key holder already knows $p$ and $q$ — they were used to generate the key in the first place. "Factor $n$ dynamically" describes what an attacker would need to do to break RSA. The legitimate decryptor simply uses the pre-known factors directly.

---

## Part 9: RSA Security — Threats and Vulnerabilities

### 1. The Mathematical Foundation — Factoring

RSA's security depends entirely on the practical infeasibility of factoring $n$ into $p$ and $q$. Multiplying two large primes is easy; reversing this — factoring the product — is believed to be computationally hard for numbers of sufficient size.

Various factoring algorithms exist (trial division, Fermat's method, quadratic sieve, number field sieve, Pollard's rho), but none can practically factor numbers of the sizes used in modern RSA:
- RSA-768 (768 bits) was factored in 2009.
- RSA-1024 (1024 bits) has not yet been publicly factored.
- RSA-2048 (2048 bits, NIST recommended minimum) carries a $200,000 prize from RSA Labs — still unclaimed.

### 2. Chosen Ciphertext / Small Exponent Attack

If $e$ is small (especially $e = 3$) and the message $M$ is short:

$$C = M^3 \bmod n$$

If $M$ is short enough that $M^3 < n$ (i.e., the modular reduction never happens), then $C = M^3$ exactly. An attacker simply computes $\sqrt[3]{C}$ to recover $M$ — no factoring needed.

**Defense:** Message padding. Before encryption, $M$ is padded with random bytes (per the OAEP scheme) to fill the full block size. The padded message is always at least as large as $n$, ensuring $M^3 \bmod n$ actually reduces.

### 3. Low Entropy — Shared Prime Factor Attack

This is a serious real-world vulnerability. If two different RSA public keys $n_1$ and $n_2$ happen to share a common prime factor (i.e., $\gcd(n_1, n_2) = p$ for some prime $p$), an attacker can factor both moduli immediately:

- $\gcd(n_1, n_2) = p$ (computed in milliseconds using Euclidean algorithm)
- $q_1 = n_1 / p$, $q_2 = n_2 / p$

Both private keys are broken with essentially no work.

How can this happen? Primes are generated using random number generators. At device boot time — when network devices first generate RSA keys — the system entropy (randomness) pool is often almost empty. Devices with nearly identical state generate nearly identical random numbers, and some pairs accidentally share a prime factor.

A 2012 study ("Mining your Ps and Qs") harvested over 5 million TLS/SSL certificates and 4 million SSH host keys, and was able to compute the private keys for **0.50% of TLS servers** and **0.03% of SSH servers** using nothing but GCD computations. This is a reminder that cryptographic security depends on true randomness as much as mathematical hardness.

### 4. The RSA Factoring Challenge

RSA Labs used to sponsor a factoring challenge to establish the practical difficulty of factoring:
- RSA-576 (174 digits): factored in 2003
- RSA-640 (193 digits): factored in 2005
- RSA-768 (232 digits): factored in 2009
- RSA-1024, RSA-2048: not yet publicly factored

This provides guidance on minimum key sizes. The current recommendation is 2048-bit keys as a minimum, with 3072 or 4096 bits for long-term security.

### 5. Perfect Forward Secrecy — A Limitation of RSA Key Exchange

When RSA is used to exchange a session key — A encrypts the session key $K_S$ with B's public key — an eavesdropper can record all the encrypted traffic. If the eavesdropper later obtains B's private key (through a compromise, legal order, or eventual factoring), they can decrypt all the session keys and therefore all the recorded conversations retroactively.

This lack of **perfect forward secrecy (PFS)** is a serious concern for long-term confidentiality. The Diffie-Hellman key exchange (studied in Lecture 04) addresses this by generating ephemeral keys that are discarded after each session, so past sessions cannot be decrypted even if long-term keys are later compromised.

---

### Generated Practice — RSA Security

**Practice 1:** An attacker observes that two servers have public keys with moduli $n_1 = 221$ and $n_2 = 247$. Compute $\gcd(n_1, n_2)$ and explain what it reveals.  
**Answer:** $\gcd(221, 247) = 13$ (since $221 = 13 \times 17$ and $247 = 13 \times 19$). This means both RSA keys share the prime factor $p = 13$. The attacker immediately recovers $q_1 = 221/13 = 17$ and $q_2 = 247/13 = 19$, factoring both moduli and breaking both private keys.

**Practice 2:** Why does increasing $e$ from 3 to 65537 improve security against small-exponent attacks?  
**Answer:** With $e = 3$ and a short message $M$, the value $M^3$ might be smaller than $n$, meaning $C = M^3$ exactly (no reduction modulo $n$). The attacker simply takes the cube root. With $e = 65537$, $M^{65537}$ grows so rapidly that it will always exceed $n$ for any realistic message, so modular reduction always occurs. The attack fails. Additionally, OAEP padding ensures the message is always large.

**Practice 3:** Explain why RSA private key decryption is slower than public key encryption, and how CRT addresses this.  
**Answer:** The public exponent $e$ is small (e.g., 65537 = $2^{16} + 1$, only 17 bits with 2 set bits). Computing $M^e \bmod n$ requires at most $\sim 2 \times 17 = 34$ multiplications. The private exponent $d$ is roughly as large as $n$ (e.g., 2048 bits with ~1024 set bits). Computing $C^d \bmod n$ requires $\sim 3072$ multiplications with 2048-bit numbers. CRT reduces this by replacing one large exponentiation with two exponentiations modulo $p$ and $q$ (each half the size), which is approximately 4 times faster by the square law: halving the size of the modulus reduces each multiplication to $1/4$ the cost.

---

### Knowledge Check 9

**Q:** What are the seven steps of RSA key generation in order?  
**A:** (1) Choose primes $p$, $q$. (2) Compute $n = pq$. (3) Compute $\phi(n) = (p-1)(q-1)$. (4) Choose $e$ with $1 < e < \phi(n)$ and $\gcd(e, \phi(n)) = 1$. (5) Compute $d = e^{-1} \bmod \phi(n)$. (6) Public key = $[e, n]$. (7) Private key = $[d, n]$.

**Q:** What makes the private key "private"? What must be kept secret?  
**A:** $d$, $p$, $q$, and $\phi(n)$ must all be kept secret. Knowing $\phi(n)$ is equivalent to knowing $p$ and $q$ (since $\phi(n) = (p-1)(q-1)$ can be solved for $p$ and $q$ given $n$ and $\phi(n)$). Knowing any of them allows computing $d$ and breaking the cipher.

**Q:** What is the "fast" part of fast modular exponentiation?  
**A:** Instead of computing $A^B$ (which would have an astronomically large number of digits) and then reducing mod $n$, the reduction mod $n$ is applied at every multiplication step. Repeated squaring reduces the number of multiplications from $B$ to $O(\log B)$. Numbers never grow larger than $n^2$.

---

## Part 10: Operational Details

### Key Representation

RSA keys are large integers that must be stored and transmitted. In practice:
- Keys are often encoded in **Base64** for printable ASCII representation.
- For **SSH**: key generation uses `ssh-keygen` (RFC 4253). The private key is Base64-encoded and stored in a file like `~/.ssh/id_rsa`.
- For **SSL/TLS**: `openssl genrsa` generates keys. The key data is first encoded in **ASN.1** (Abstract Syntax Notation One) format, DER-encoded into a byte stream, and then Base64-encoded for storage/transmission.

### Extracting RSA Parameters

Python tools (`pyasn1`, `cryptography` library) can extract the modulus $n$ and public exponent $e$ from an RSA public key file. This is useful for security analysis — for example, scanning many public keys to find ones sharing common factors.

---

## Part 11: Final Exam Preparation — Lecture 03

### Relevant 2025 Midterm Questions — Lecture 03 Index

| Question | Topic | Location in Guide |
|----------|-------|------------------|
| Q30 | RSA security — factoring | Part 9 |
| Q31 | RSA key generation order | Part 4 |
| Q32 | Public exponent $e$ condition | Part 4 |
| Q33 | Private exponent $d$ computation | Part 4 |
| Q34 | Computationally expensive RSA operation | Part 5 / Part 8 |
| Q35 | Why RSA not for bulk encryption | Part 2 |
| Q36 | Common modulus attack | Part 9 |
| Q37 | RSA-CRT speedup | Part 8 |
| Q38 | NOT part of RSA-CRT (factor $n$ dynamically) | Part 8 |

### Comprehensive Review Questions

**1.** A server uses RSA with $p = 61$, $q = 53$. Compute $n$, $\phi(n)$. If $e = 17$, verify it is valid and compute $d$.  
**Answer:**
- $n = 61 \times 53 = 3233$
- $\phi(n) = 60 \times 52 = 3120$
- $\gcd(17, 3120) = 1$ ✓ (17 is prime and does not divide 3120)
- $d \equiv 17^{-1} \pmod{3120}$: Extended Euclid gives $d = 2753$ (verify: $17 \times 2753 = 46801 = 15 \times 3120 + 1$ ✓)

**2.** Why does RSA require $n$ to be a product of exactly two primes, not three or more?  
**Answer:** The key property is $\phi(n) = (p-1)(q-1)$ when $n = pq$. For three primes $n = pqr$, $\phi(n) = (p-1)(q-1)(r-1)$. RSA still works mathematically, but using three or more factors makes $n$ easier to factor (smaller factors are easier to find), reducing security. The two-prime choice optimizes the balance between mathematical tractability and security.

**3.** Describe precisely why the common-modulus attack works algebraically.  
**Answer:** Given two ciphertexts of the same message $M$ under the same modulus $n$ but different exponents $e_1$ and $e_2$ with $\gcd(e_1, e_2) = 1$: use Extended Euclidean to find $a, b$ such that $e_1 a + e_2 b = 1$. Then $C_1^a \cdot C_2^b = (M^{e_1})^a \cdot (M^{e_2})^b = M^{e_1 a + e_2 b} = M^1 = M \pmod{n}$. The message is recovered without factoring.

---

*End of Lecture 03 Study Guide*  
*Next: Lecture 04 — More Public-Key Cryptography (DH, ElGamal, PKI, Certificates)*
