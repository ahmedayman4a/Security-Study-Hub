# Lecture 06 — Secure Random Number Generation
## CS464: Computer System Security — Complete Study Guide

---

## Part 1: Why Random Numbers Are Critical in Security

There is a line that appears in almost every cryptographic protocol: "A chooses a random number…" or "generate a fresh random key." The entire security of these protocols often depends on that random number being genuinely unpredictable.

Consider what happens if an attacker can predict your "random" numbers:
- If the session key for AES is predictable, the attacker can try all likely keys trivially.
- If the nonces in Needham-Schroeder or Kerberos are predictable, the replay attack protection collapses.
- If the primes $p$ and $q$ in RSA are generated from a predictable source, an attacker can reproduce them and compute the private key. (This is exactly what the "Mining your Ps and Qs" study exploited — recall from Lecture 03.)
- If the ephemeral Diffie-Hellman private values are predictable, perfect forward secrecy is lost.
- If the random salt in a password hash is predictable, targeted pre-computation attacks become feasible.

Cryptography is as strong as its randomness. A brilliant algorithm running on a bad random number generator is broken.

### Where Random Numbers Are Used
- **Session keys** (symmetric keys for AES/3DES in each communication session)
- **Nonces** (freshness tokens in handshake protocols)
- **RSA prime generation** (generating $p$ and $q$)
- **Diffie-Hellman private keys** ($X_A$, $X_B$)
- **ElGamal signing nonce** ($K$ — and we saw what happens when it's reused)
- **Password salts** (random values added to passwords before hashing, preventing rainbow table attacks)
- **IVs** for CBC, CTR, and other block cipher modes

---

## Part 2: What Makes a Random Number "True"?

The word "random" is used loosely. In security, we need a precise definition.

### Two Essential Properties of True Random Numbers

**1. Uniform Distribution:**  
All numbers in the designated range must occur equally often. If you are generating random bits, 0s and 1s must occur with equal probability. If generating numbers in $[0, m-1]$, each value must have probability $1/m$. A biased generator makes some outputs more predictable than others.

**2. Independence (Unpredictability):**  
If you know all the numbers generated so far — indeed if you know the entire history of outputs — you should not be able to predict the next output (or any future output) better than random guessing. Independence is the cryptographic property: a generator that is deterministic once you know its state fails independence.

### PRNG vs. TRNG

**Pseudorandom Number Generator (PRNG):** An algorithm that starts from a **seed** (an initial value) and generates a sequence of numbers that *appear* random — they pass statistical tests for randomness. But the sequence is entirely **deterministic** given the seed. Two runs with the same seed produce identical sequences. PRNGs are fast and convenient but cryptographically dangerous if the seed is predictable or small.

**True Random Number Generator (TRNG):** Derives randomness from **physical phenomena** — events in the real world that are genuinely unpredictable. Unlike PRNGs, TRNGs work without seeds. The output is not reproducible because it comes from real-world randomness. TRNGs are slower but provide genuine unpredictability.

**Cryptographically Secure PRNG (CSPRNG):** A PRNG that meets additional cryptographic requirements — specifically, that even an attacker who observes many outputs cannot predict future outputs or determine past outputs. CSPRNGs are the practical compromise: they use a PRNG structure (for speed) but are seeded from a TRNG-quality entropy source (for security).

---

## Part 3: PRNG — Linear Congruential Generator (LCG)

### The Algorithm

The LCG (Linear Congruential Generator) is the most widely used basic PRNG algorithm. Starting from a seed $X_0$, it generates successive values using:

$$X_{n+1} = (a \times X_n + c) \bmod m$$

Parameters:
- $m$ — the **modulus** ($m > 0$)
- $a$ — the **multiplier** ($0 < a < m$)
- $c$ — the **increment** ($0 \leq c < m$)
- $X_0$ — the **seed** ($0 < X_0 < m$)

The output sequence $X_0, X_1, X_2, \ldots$ appears statistically random for well-chosen parameters.

### Worked Example

With $a = 7$, $c = 0$, $m = 32$, seed $X_0 = 1$:

| $n$ | $X_n$ | Computation |
|-----|--------|------------|
| 0 | 1 | seed |
| 1 | 7 | $(7 \times 1 + 0) \bmod 32$ |
| 2 | 17 | $(7 \times 7 + 0) \bmod 32 = 49 \bmod 32$ |
| 3 | 23 | $(7 \times 17) \bmod 32 = 119 \bmod 32$ |
| 4 | 1 | $(7 \times 23) \bmod 32 = 161 \bmod 32 = 1$ |

The sequence cycles with **period 4**: $\{7, 17, 23, 1, 7, 17, 23, 1, \ldots\}$. The parameters were chosen poorly — the period is far shorter than $m$.

With better parameters — specifically when $m$ is prime, $c = 0$, and $a$ is a primitive root modulo $m$ — the recursion is guaranteed to produce a sequence of period $m - 1$ (every non-zero value in $\{1, 2, \ldots, m-1\}$ appears exactly once before cycling).

The widely studied and good-quality LCG uses $m = 2^{31} - 1$ (a Mersenne prime) and $a = 7^5 = 16807$.

### Why LCG Is Not Cryptographically Secure

The LCG is **not a CSPRNG**. Given three consecutive outputs $(X_n, X_{n+1}, X_{n+2})$, an attacker can solve for $a$, $c$, and $m$ using linear algebra (since the recurrence is linear). Once the parameters are known — and they usually are, since LCG implementations publish them — the attacker only needs the **current value** $X_n$ to predict all future values.

A partial fix: restart the LCG with a new seed after generating $N$ numbers. This limits the predictable window but is still fragile. The fundamental problem is that the generator is entirely deterministic and linear.

### Practice: LCG Computation

**Problem:** With $a = 5$, $c = 3$, $m = 13$, seed $X_0 = 7$. Generate the next 4 values.  
**Answer:**
- $X_1 = (5 \times 7 + 3) \bmod 13 = 38 \bmod 13 = 12$
- $X_2 = (5 \times 12 + 3) \bmod 13 = 63 \bmod 13 = 11$
- $X_3 = (5 \times 11 + 3) \bmod 13 = 58 \bmod 13 = 6$
- $X_4 = (5 \times 6 + 3) \bmod 13 = 33 \bmod 13 = 7$

At $X_4 = 7$, the sequence has cycled back to the seed! Period = 4.

---

## Part 4: CSPRNG — ANSI X9.17/X9.31

### Overview

The **ANSI X9.17/X9.31 algorithm** is a CSPRNG standard used in many secure systems — including those for financial transactions (the X9.17 designation comes from the ANSI financial standards track) and email exchange. Unlike the LCG, this algorithm is designed to be cryptographically unpredictable even to an attacker who knows the algorithm.

The algorithm uses **two-key 3DES (EDE mode)** as its underlying cryptographic primitive. The EDE boxes in the algorithm diagram refer to: Encrypt with $K_1$, Decrypt with $K_2$, Encrypt with $K_1$.

### Inputs and State

The algorithm is driven by:
- **Two 56-bit 3DES keys** $K_1$ and $K_2$ (these form the long-term secret state).
- **$DT_j$**: a 64-bit representation of the current date and time — different for each output number.
- **$V_j$**: a 64-bit state value from the previous iteration (initialized with seed $V_0$).

### The Formulas

At each iteration $j$, the algorithm produces a random number $R_j$ and the next state $V_{j+1}$:

$$R_j = \text{EDE}([K_1, K_2],\ V_j \oplus \text{EDE}([K_1, K_2],\ DT_j))$$

$$V_{j+1} = \text{EDE}([K_1, K_2],\ R_j \oplus \text{EDE}([K_1, K_2],\ DT_j))$$

Between two consecutive random numbers, there are **nine DES encryptions** total (three EDE operations of three DES operations each). This computational intensity makes it virtually impossible to predict the next random number even if all previous outputs are known.

### Limitation

The 9 DES operations per output number make this algorithm **much slower** than simple PRNGs. It is unsuitable for applications that need high-throughput random numbers (e.g., encrypting a video stream, where millions of random bytes per second are needed).

---

## Part 5: CSPRNG — Blum Blum Shub (BBS)

### Motivation

The BBS (Blum Blum Shub) generator produces a pseudorandom **bit stream** with a security reduction to a hard mathematical problem — specifically, the difficulty of factoring large integers. Unlike X9.17 (which relies on 3DES security), BBS can be proven secure under the assumption that factoring is hard.

### Setup

1. Choose two large primes $p$ and $q$, both satisfying $p \equiv q \equiv 3 \pmod{4}$ (i.e., $p \bmod 4 = 3$ and $q \bmod 4 = 3$).
2. Compute $n = p \times q$ (same structure as RSA modulus).
3. Choose a seed $s$ that is **coprime to $n$** (i.e., $\gcd(s, n) = 1$, meaning $p$ and $q$ are not factors of $s$).

### Generation

Generate the state sequence:

$$X_0 = s^2 \bmod n$$
$$X_{i+1} = X_i^2 \bmod n$$

At each step $i$, the output bit $B_i$ is the **least significant bit** (LSB) of $X_i$:

$$B_i = X_i \bmod 2$$

The sequence $B_0, B_1, B_2, \ldots$ is the pseudorandom bit stream.

### Worked Example

Let $p = 11$, $q = 19$ (verify: $11 \bmod 4 = 3$ ✓, $19 \bmod 4 = 3$ ✓), $n = 11 \times 19 = 209$.  
Seed $s = 3$: $\gcd(3, 209) = 1$ ✓ (3 is not a factor of 209).

$X_0 = 3^2 \bmod 209 = 9$, $B_0 = 9 \bmod 2 = 1$  
$X_1 = 9^2 \bmod 209 = 81$, $B_1 = 81 \bmod 2 = 1$  
$X_2 = 81^2 \bmod 209 = 6561 \bmod 209 = 6561 - 31 \times 209 = 6561 - 6479 = 82$, $B_2 = 0$  
$X_3 = 82^2 \bmod 209 = 6724 \bmod 209 = 6724 - 32 \times 209 = 6724 - 6688 = 36$, $B_3 = 0$

Bit stream: $1, 1, 0, 0, \ldots$

### Security and Practicality

BBS is **provably secure** under the quadratic residuosity assumption (related to factoring). An attacker who can predict BBS outputs can factor $n$ — and vice versa. This is a strong theoretical guarantee.

However, BBS is **slow**: each bit requires a modular squaring of a large number. It is not suitable for high-throughput applications but is valuable when a provably secure bit source is needed (e.g., generating cryptographic keys).

---

## Part 6: True Random Number Generators (TRNG) — Entropy Sources

### The Fundamental Difference

A PRNG must be initialized with a seed — remove the seed and you have nothing. A **TRNG works without seeds** because it derives randomness from physical phenomena that are genuinely unpredictable by nature.

The key insight: **only analog phenomena can be trusted to produce truly random numbers.** Digital systems are deterministic (given the same state, they always produce the same output). Physical phenomena — thermal noise in resistors, radioactive decay, quantum tunneling, turbulent airflow — are not deterministic; they are fundamentally random at the quantum level.

### Shannon Entropy

If we treat the bit stream from an entropy source as a sequence of words (bytes, words, etc.), and if each word $i$ occurs with probability $p_i$, the **Shannon entropy** quantifies how much information (randomness) is in each word:

$$H = -\sum_i p_i \log_2 p_i \quad \text{(bits per word)}$$

For a perfectly uniform distribution (maximum randomness), $H$ equals $\log_2 N$ where $N$ is the number of possible word values. For example, for a byte ($N = 256$) with uniform distribution, $H = \log_2 256 = 8$ bits — maximum entropy per byte.

An entropy source with low $H$ (e.g., only a few common byte values) is a poor randomness source. High entropy sources approach the theoretical maximum.

### Limitation of Entropy Sources

Entropy sources are inherently **slow** — they cannot produce random bits at the rate needed by high-performance cryptographic applications. A typical software entropy source on Linux generates only a few hundred bits of entropy per second. This is far too slow for, say, AES key stream generation.

The solution: use entropy sources to **seed CSPRNGs**. A small amount of high-quality entropy seeds a fast, cryptographically secure PRNG that can then generate large amounts of pseudorandom bits quickly.

---

## Part 7: Hardware-Based Entropy Sources

Hardware-based entropy sources are built directly into modern processors, enabling high-speed, high-quality random number generation.

### Intel Bull Mountain DRNG

Intel's **Digital Random Number Generator (DRNG)**, code-named "Bull Mountain," is implemented on-chip in modern Intel processors. It uses thermal noise in electronic components as the entropy source and can produce a random bit stream at approximately **3 GHz** — billions of bits per second.

Intel provides a machine-code instruction, **`RDRAND`**, for 64-bit processors. When executed, `RDRAND` fetches a fresh 64-bit random number directly from the DRNG hardware. This can be called from software in any language via inline assembly or compiler intrinsics.

The DRNG architecture:
1. An analog thermal noise source generates truly random bits.
2. A conditioning component (using AES in CBC-MAC mode) distills and whitens the raw noise.
3. A CSPRNG (seeded from the conditioned entropy) provides high-throughput output.
4. The `RDRAND` instruction fetches from the CSPRNG output.

This provides a practical combination: a hardware TRNG as the entropy source and a hardware CSPRNG for throughput.

---

## Part 8: Software-Based Entropy Sources

When hardware entropy is unavailable or untrusted, software can gather entropy from the inherent randomness in human and system interactions.

### Sources of Software Entropy

In every computer, several phenomena have timing or content that is unpredictable at fine-grained resolution:
- **Keystroke timings**: the exact nanosecond at which each key is pressed.
- **Mouse movement timings**: interrupt timing from mouse button clicks and movements.
- **Disk I/O event times**: when disk operations complete.
- **Network packet arrival times**: interrupts from network interface cards.
- **Log file contents**: various system log entries with unpredictable content.
- **System command outputs**: `ps aux`, `netstat`, memory statistics — all contain entropy.

None of these are individually very random, but combining many sources over time accumulates significant entropy.

### Two Categories: Kernel Space vs. User Space

**Kernel space entropy** (available to root/system processes):
- The Linux/Unix kernel gathers entropy from hardware interrupts, disk I/O, and other sources.
- Accessible via the special file **`/dev/random`**.
  - Very high quality — blocks until sufficient entropy is available.
  - Can typically only generate a few hundred bits per second.
  - May **block** if the entropy pool is exhausted — useful for generating long-term keys where quality matters more than speed.
- **`/dev/urandom`** (unbounded random): uses the entropy from `/dev/random` to seed an internal CSPRNG, which then provides unlimited pseudorandom bytes without blocking. High quality, fast, but theoretically the CSPRNG output is pseudorandom (not true-random) once the seed is set.

**User space entropy** (accessible without root privileges):
- **EGD (Entropy Gathering Daemon)**: a Perl-based daemon that gathers entropy from system events and makes it available to user-space programs via a Unix domain socket.
- **PRNGD (Pseudo Random Number Generator Daemon)**: similar to EGD.

### `/dev/random` vs `/dev/urandom` — When to Use Which

| Feature | `/dev/random` | `/dev/urandom` |
|---------|---------------|----------------|
| Blocking | Yes (waits for entropy) | No |
| Quality | True random (entropy-bound) | CSPRNG output |
| Speed | Slow (hundreds of bits/sec) | Fast |
| Use case | Long-term key generation | Session keys, IVs, nonces |

For RSA key generation, use `/dev/random`. For AES session keys and IVs, `/dev/urandom` is sufficient and much faster.

---

## Part 9: The Complete Picture

### Putting It Together

The typical architecture in a real system:

```
Physical noise (thermal, quantum)
        ↓
  Hardware TRNG / /dev/random
        ↓ (seed - small, high quality)
       CSPRNG (BBS, X9.31, or AES-CTR-DRBG)
        ↓ (large, fast pseudorandom output)
  Cryptographic protocol
  (session keys, nonces, IVs, primes, ...)
```

The design principle: **true randomness is precious and slow; use it to seed a fast CSPRNG**. The CSPRNG stretches a small amount of true randomness into a large stream of cryptographically secure pseudorandom bits.

### Why This Matters: Real-World Failures

The RSA low-entropy vulnerability from Lecture 03 is the canonical example: at device boot time, the entropy pool is nearly empty (the system has been running for milliseconds, barely any human interactions or disk events have occurred). Devices that generate RSA keys at boot time using `/dev/random` or equivalent may end up with very similar seeds — and thus very similar primes — because there simply is not enough entropy yet. This was exploited to find common factors between thousands of RSA moduli, breaking 0.5% of all TLS servers.

The fix: wait for the entropy pool to fill before generating long-term keys, or use a hardware TRNG (like `RDRAND`) that is not affected by this boot-time issue.

---

## Part 10: Knowledge Checks and Practice

### Knowledge Check 1

**Q:** What is the period of an LCG, and why does it matter?  
**A:** The period is the length of the repeating cycle before the sequence repeats. If the period is short (e.g., 4 as in the example), an attacker can predict all future outputs by observing a few. For cryptographic use, the period must be exponentially long — ideally $m - 1$ for prime modulus LCGs.

**Q:** What two properties distinguish a CSPRNG from a plain PRNG?  
**A:** (1) Next-bit unpredictability: an attacker who sees all past outputs cannot predict the next bit better than 50%. (2) State compromise extension resistance: even if the current internal state is somehow revealed, the attacker cannot reconstruct past outputs.

**Q:** What is the role of `ipad` and `opad` in HMAC? (Cross-reference with Lecture 05.)  
**A:** They are fixed constants used to derive two different key variants from the same key $K^+$: one for the inner hash (with `ipad`) and one for the outer hash (with `opad`). Having two different key variants prevents the inner and outer hashes from using the same keyed state, which is what defeats length extension attacks.

### Knowledge Check 2

**Q:** Why must the primes in BBS satisfy $p \equiv q \equiv 3 \pmod 4$?  
**A:** This condition ensures that $-1$ is a quadratic non-residue modulo $n$. This property is essential for the mathematical security proof of BBS — it ensures that the quadratic residuosity problem (on which BBS security is based) is hard. Without this condition, the mathematical security reduction breaks down.

**Q:** Explain the difference between `/dev/random` and `/dev/urandom` in terms of what they output.  
**A:** `/dev/random` outputs bits from the kernel's entropy pool — genuine, harvested randomness. It blocks when the pool is empty. `/dev/urandom` uses the entropy pool to seed an internal CSPRNG and outputs pseudorandom bytes generated by that CSPRNG. It never blocks. Both are considered secure for most purposes; `/dev/random` is preferred for generating long-term keys where the highest quality is needed.

---

## Part 11: Summary — PRNG/CSPRNG/TRNG Comparison

| Generator | Type | Key Property | Speed | Crypto Secure? | Examples |
|-----------|------|-------------|-------|---------------|---------|
| LCG | PRNG | Deterministic, linear recurrence | Very fast | No | Many standard libraries |
| ANSI X9.31 | CSPRNG | 3DES-based, unpredictable | Slow (9 DES per output) | Yes | Financial systems |
| BBS | CSPRNG | Factoring-based, provable security | Slow (modular squaring) | Yes (provable) | Key generation |
| `/dev/random` | TRNG | Harvested physical entropy | Very slow | Yes | Long-term key gen |
| `/dev/urandom` | Hybrid | TRNG seed + CSPRNG | Fast | Yes (practical) | Session keys, IVs |
| Intel RDRAND | TRNG + CSPRNG | Hardware TRNG + conditioner | 3 GHz | Yes | Modern systems |

---

## Part 12: Final Exam Preparation — Lecture 06

### Exam Coverage Note

The 2025 midterm (Q1–Q60) does not include questions specifically covering Lecture 06 (Secure Random Number Generation). Exam questions Q49–Q60 cover Lecture 05 (Hashing). This lecture's material has not appeared in the provided exam questions.

However, the concepts here are foundational to understanding why the security failures in other lectures occur (RSA low entropy, DH ephemeral key security, Kerberos nonces). Be prepared to connect these ideas in essay questions or in explaining vulnerabilities.

### Comprehensive Review Questions

**1.** Explain why an LCG with publicly known parameters $m$, $a$, $c$ is not cryptographically secure.  
**Answer:** Given any current value $X_n$, the next value $X_{n+1} = (a \times X_n + c) \bmod m$ is completely predictable. All future values follow deterministically. Even if the parameters are private, observing three consecutive outputs allows solving for $m$, $a$, $c$ algebraically. A CSPRNG must ensure that observing any number of outputs gives no information about future or past outputs.

**2.** What property must a CSPRNG have that a regular PRNG does not?  
**Answer:** Next-bit unpredictability: given all previous outputs, it must be computationally infeasible to predict the next bit better than random guessing (50%). A regular PRNG may pass statistical tests but still be predictable from its outputs (like the LCG). Additionally, CSPRNGs should be resistant to state compromise — if an attacker somehow learns the current internal state, they should not be able to recover past outputs.

**3.** In the BBS generator, why does it output only the LSB of each $X_i$?  
**Answer:** Outputting more bits per iteration would make it easier for an attacker to solve for the seed $s$ and predict future values. The LSB is the "hardest" bit to predict from the factoring problem (it corresponds to the quadratic residuosity parity). Outputting more bits reduces the work factor for an attacker. For maximum security, only the LSB (or a small number of bits) is output per iteration.

**4.** Why is the boot-time period dangerous for random number generation?  
**Answer:** At boot time, the entropy pool is nearly empty. The system has only been running for a short time — very few keystrokes, mouse events, disk operations, or network packets have occurred. If a long-term key (like an RSA private key) is generated immediately at boot, the seed used for prime generation has very low entropy. Different devices in the same batch that boot in similar states may generate the same or factorably related primes, completely breaking RSA security.

---

*End of Lecture 06 Study Guide*  
*Next: Lecture 07 — (check syllabus)*
