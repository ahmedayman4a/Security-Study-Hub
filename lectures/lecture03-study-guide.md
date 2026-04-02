# Lecture 03 — Public-Key Cryptography & RSA
## CS464: Computer System Security — Complete Study Guide

---

## Part 1: The Problem With Symmetric Cryptography

Every symmetric cipher we have studied — DES, 3DES, AES — shares a fundamental assumption that is easy to miss: both parties already share the secret key *before* any secure communication begins. Think carefully about what this means. Symmetric encryption solves the problem of secrecy *after* trust is established. It does not solve the problem of establishing that trust in the first place.

Imagine Alice wants to send Bob a confidential message over the internet. She decides to use AES. She encrypts her message and sends it. Bob receives the ciphertext. The problem: Bob cannot decrypt it without the key. Alice would need to tell Bob the key — but if an eavesdropper is watching the channel, she cannot send the key securely without already having a secure channel. That is a circular dependency. Every method for sharing the key before communication begins assumes some prior secure contact or trusted courier — which either does not scale or does not exist for strangers on the internet.

This is the **key distribution problem**, and it plagued cryptography for centuries. The question it poses is not "how do we encrypt?" but "how do two strangers establish a shared secret when they have no prior relationship and no trusted channel?"

**Public-key cryptography** (also called **asymmetric cryptography**) dissolves this problem by changing the setup entirely. Instead of sharing a secret unlocker, Bob publishes a *lock* that anyone can snap shut — but only Bob can open. Structurally, the shift is this: symmetric cryptography requires both parties to own the same secret before communication. Public-key cryptography makes one operation safe to expose. Anyone can encrypt for Bob using his public key; only Bob can decrypt using his private key. You no longer need to establish a shared secret before communicating. The asymmetry of the key pair is what breaks the circular dependency.

---

## Part 2: Public-Key Cryptography — The Big Picture

### Key Pairs

In public-key cryptography, every party has a **key pair**:
- A **public key** (denoted $PU$): published openly to the world. Anyone can have it.
- A **private key** (denoted $PR$): kept secret. Only the owner knows it.

The two keys are mathematically linked, but the link is computationally one-way: knowing the public key does not let you compute the private key (assuming the underlying hard mathematical problem is genuinely hard). One direction is easy — generate, publish; the other direction is computationally infeasible — recover.

You might wonder: if the public key is just... public, why does that not compromise security? Because the public key can only *lock* — it encrypts or verifies. Only the private key can *unlock* — it decrypts or signs. The padlock analogy is apt: anyone can snap the padlock shut, but only the keyholder can open it. Publishing the closed padlock reveals nothing about the key.

### Public-Key Services — Designed Around Who Should See What

Understanding the three services of public-key cryptography is easier if you think about them as design problems: who is supposed to be able to perform this operation, and who must be excluded? The protocol structure follows from answering those questions.

**1. Confidentiality (from A to B):**

The goal is that only Bob can read Alice's message. Who must be able to perform the encryption? Anyone — Alice, or any sender. Who must be excluded from decryption? Everyone except Bob. The only operation unique to Bob is his private key. Therefore: encrypt with Bob's public key, decrypt with Bob's private key.

$$C = E(PU_B,\ M) \qquad M = D(PR_B,\ C)$$

Anyone can encrypt for Bob. Only Bob can decrypt. The design follows directly from the goal.

**2. Digital Signature / Authentication / Non-repudiation (from A to B):**

The goal is that anyone can verify the message came from Alice, but only Alice could have created the signature. Who must be able to perform the signing? Only Alice. Who must be able to verify? Anyone. The only operation unique to Alice is her private key. Therefore: sign with Alice's private key, verify with Alice's public key.

$$C = E(PR_A,\ M) \qquad M = D(PU_A,\ C)$$

Only Alice can sign. Anyone can verify. If you are tempted to think of this as "encrypting with the private key," resist that framing — it is thinking about the mechanism rather than the goal. The goal is that one person can produce a value and everyone else can check it, and the private key is the only piece of information exclusive to Alice.

**3. Confidentiality + Digital Signature:**

The goal is both secrecy and authenticity. Alice wants Bob to receive a message that only Bob can read *and* that Bob can verify came from Alice. Here the order of operations is not arbitrary — it must be chosen based on who needs to see what.

Alice signs first (with $PR_A$), then encrypts the signed message (with $PU_B$). Bob decrypts first (with $PR_B$), then verifies the signature (with $PU_A$).

$$C = E(PU_B,\ E(PR_A,\ M)) \qquad M = D(PU_A,\ D(PR_B,\ C))$$

You might ask why this order rather than the reverse. If Alice encrypted first and then signed the ciphertext, Bob could verify that someone signed the ciphertext — but not that Alice signed the underlying message. Anyone with access to the ciphertext could attach a signature. Signing the *plaintext first* means Alice is committing to the actual message content, not to an arbitrary encrypted blob.

### Does Public-Key Cryptography Replace Symmetric Cryptography?

It does not — and this is one of the most important design principles in applied cryptography. Public-key cryptography is **computationally expensive**: a single RSA decryption with a 2048-bit key requires thousands of large-number multiplications. Encrypting a large file directly with RSA would be orders of magnitude slower than AES, and would also require breaking the file into blocks and managing padding — an architectural mess.

The correct model — used in every real protocol including TLS/HTTPS — is **hybrid encryption**: use public-key cryptography for the narrow task it excels at (key exchange and authentication), then use fast symmetric encryption for the actual data. RSA or Diffie-Hellman establishes a shared session key; AES with that session key encrypts the traffic. You get the trust-establishment power of public-key cryptography without paying its performance penalty on every byte of data.

The reason hybrid is inevitable — not just convenient — is this: the mathematical hardness that makes public-key crypto secure is also what makes it slow. You cannot have RSA-like security with AES-like speed, because the former requires working in a number-theoretic structure that makes large-number operations unavoidable. The two approaches occupy different points on the security-speed tradeoff, and hybrid design uses each where it fits.

---

### Knowledge Check 1

**Q:** What fundamental problem does public-key cryptography solve that symmetric cryptography cannot?  
**A:** The key distribution problem — how two parties who have never communicated establish a shared secret key over an insecure channel. Symmetric cryptography requires that shared secret *before* communication. Public-key crypto dissolves the requirement by making one direction of the key operation safe to publish.

**Q:** If A encrypts with $PR_A$ (the private key), what is this used for?  
**A:** Digital signature / authentication. It proves the message came from A, since only A has $PR_A$. Anyone with $PU_A$ can decrypt and verify. The design follows from who must be excluded from creating signatures (everyone except A) vs. who must be able to verify (everyone).

**Q:** Why is public-key cryptography not used for bulk data encryption?  
**A:** The mathematical hardness that makes it secure also makes it slow — orders of magnitude slower than symmetric ciphers. Hybrid design uses public-key crypto for key exchange and authentication, then symmetric crypto for bulk data. This is not a workaround; it is the intended architecture.

---

## Part 3: RSA — Mathematical Foundation

### Designing the Right Kind of Arithmetic

RSA needs to do something that sounds simple but is surprisingly hard to arrange: two operations that undo each other, where one operation is easy for the owner to compute but the inverse is hard for everyone else to find without knowing a secret. Ordinary arithmetic does not give you this for free. If you multiply by 3, dividing by 3 undoes it — but finding the divisor is just as easy as finding the multiplier. There is no asymmetry.

Modular exponentiation turns out to have exactly the right properties — but only if the exponents are chosen carefully. Euler's theorem tells us how.

### Euler's Theorem — The Exponent Cycle

**Euler's theorem** states: for every positive integer $n$ and every integer $a$ coprime to $n$ (meaning $\gcd(a, n) = 1$):

$$a^{\phi(n)} \equiv 1 \pmod{n}$$

where $\phi(n)$ is **Euler's totient function** — the count of integers from 1 to $n$ that are coprime to $n$.

This means that in the modular world defined by $n$, exponents *wrap around* in cycles of length $\phi(n)$. Raising $a$ to a multiple of $\phi(n)$ always gives 1. The exponent has a periodic structure — a cycle — and the period is exactly $\phi(n)$.

A useful consequence: if $k = k_1 \cdot \phi(n) + k_2$ (that is, if $k$ leaves remainder $k_2$ when divided by $\phi(n)$), then:

$$a^k \equiv a^{k_1 \cdot \phi(n)} \cdot a^{k_2} \equiv (a^{\phi(n)})^{k_1} \cdot a^{k_2} \equiv 1^{k_1} \cdot a^{k_2} \equiv a^{k_2} \pmod{n}$$

In plain terms: an exponent that is a multiple of $\phi(n)$ contributes nothing — it cancels. Only the remainder matters. This is the cycle in action.

### The RSA Insight: Inverses in the Exponent Cycle

Now here is the central design idea. We want two operations — encrypt with exponent $e$, decrypt with exponent $d$ — such that applying both returns the original message. For that we need $M^{ed} \equiv M \pmod{n}$. By the cycle idea above, this happens whenever $ed \equiv 1 \pmod{\phi(n)}$, because then $ed = 1 + k\phi(n)$ for some integer $k$, and:

$$M^{ed} = M^{1 + k\phi(n)} = M \cdot (M^{\phi(n)})^k \equiv M \cdot 1^k \equiv M \pmod{n}$$

So the requirement is: choose $e$ and $d$ to be **multiplicative inverses modulo $\phi(n)$**. This means that encryption (raise to power $e$) is undone by decryption (raise to power $d$) — in the exponent world, they compose to give "do nothing."

This is the entire mathematical core of RSA. It is not magic; it is the consequence of using a number system where exponents cycle, and manufacturing two exponents that are inverses in that cycle. Every design decision in key generation is about making this construction possible while keeping $d$ secret from an attacker who sees only $n$ and $e$.

### Why RSA Correctness and RSA Security Are Different Questions

It is worth separating two things: why RSA works for the key owner, and why RSA is secure against attackers. These are answered by different arguments.

**Correctness:** Euler's theorem guarantees $M^{ed} \equiv M \pmod{n}$ whenever $ed \equiv 1 \pmod{\phi(n)}$. This is why encryption followed by decryption recovers the original message. The owner can do this because they know $d$.

**Security:** An attacker who sees $n$ and $e$ (both public) and wants to compute $d$ would need to find $e^{-1} \bmod \phi(n)$. To compute $\phi(n)$, they need $p$ and $q$ (since $\phi(n) = (p-1)(q-1)$). To find $p$ and $q$, they must factor $n$. Factoring large numbers is believed to be computationally infeasible. So the security rests on factoring hardness, not on Euler's theorem.

Students sometimes conflate these: Euler's theorem is about why it works; factoring hardness is about why it is secure. Both must hold. If factoring became easy tomorrow, RSA would break even though Euler's theorem remains true.

### The Role of $n = p \cdot q$

When $n$ is the product of two distinct primes $p$ and $q$, the totient has a beautifully simple closed form:

$$\phi(n) = \phi(p) \cdot \phi(q) = (p-1)(q-1)$$

This choice of $n$ is not arbitrary — it is the practical bridge between correctness and security, serving two opposite needs simultaneously. For the key *owner* who knows $p$ and $q$: computing $\phi(n)$ is instant, finding $d = e^{-1} \bmod \phi(n)$ takes milliseconds, and decryption works correctly by Euler's theorem. For the *attacker* who sees only $n$: computing $\phi(n)$ requires knowing $p$ and $q$, which requires factoring $n$ — infeasible for large enough primes. The two-prime structure simultaneously makes key generation easy and decryption-exponent recovery hard. That is why no simpler choice of $n$ works.

---

## Part 4: RSA Key Generation

### Key Generation as a Chain of Necessities

The key generation steps look like a checklist, but each step exists because the next step requires it. Missing or weakening any step does not just reduce security — it makes the whole scheme fail. Let us build the chain.

**Step 1 — Choose two large distinct primes $p$ and $q$.**

We need two secret primes because they let us compute $\phi(n) = (p-1)(q-1)$, and $\phi(n)$ lets us find the decryption exponent $d$. An attacker who knows $n$ cannot compute $\phi(n)$ without knowing $p$ and $q$ — that is the security claim.

Why *distinct*? If $p = q$, then $n = p^2$, and $\phi(p^2) = p^2 - p = p(p-1)$. An attacker who suspects $p = q$ simply computes $\sqrt{n}$ and recovers $p$ immediately. The distinctness requirement is not bureaucratic hygiene — it eliminates a trivial factorization shortcut.

Why *random*? The security claim "factoring is hard" assumes the primes were chosen unpredictably. If the random number generator produces predictable outputs — due to low entropy at boot time, for instance — two devices might generate primes with overlapping factors. Then $\gcd(n_1, n_2)$ immediately factors both moduli, as the 2012 "Mining your Ps and Qs" study demonstrated on real-world internet servers. Randomness is not hygiene; it is the precondition for the hardness assumption to apply.

These must be generated randomly and kept secret forever. If $n$ must be $B$ bits, each prime is roughly $B/2$ bits.

**Step 2 — Compute the modulus $n$.**

$$n = p \times q$$

**Step 3 — Compute the totient $\phi(n)$.**

$$\phi(n) = (p - 1)(q - 1)$$

$\phi(n)$ must be kept secret. Knowing $\phi(n)$ is essentially equivalent to knowing $p$ and $q$: given $n$ and $\phi(n)$, one can solve for $p$ and $q$ via the relation $p + q = n - \phi(n) + 1$.

**Step 4 — Choose the public exponent $e$.**

Select $e$ such that:

$$1 < e < \phi(n) \quad \text{and} \quad \gcd(e,\ \phi(n)) = 1$$

The condition $\gcd(e, \phi(n)) = 1$ is the crucial one. A multiplicative inverse $d$ satisfying $ed \equiv 1 \pmod{\phi(n)}$ exists if and only if $\gcd(e, \phi(n)) = 1$. If you chose an $e$ that shares a factor with $\phi(n)$, there is no valid $d$ — decryption would be mathematically impossible. The coprimality condition does not just improve security; it is what makes decryption exist at all.

Typical values: **3**, 17, and **65537** ($= 2^{16} + 1$). The value 65537 is strongly preferred in practice — it is prime, large enough to avoid cube-root attacks (unlike 3), and has only two bits set in its binary representation ($10000000000000001_2$), which makes exponentiation with $e$ very fast.

$e = 3$ is considered weak: for a short message $M$, $M^3$ might be smaller than $n$, in which case $C = M^3$ exactly (no modular reduction), and the attacker simply computes $\sqrt[3]{C}$ to recover $M$ — no factoring needed.

**Step 5 — Compute the private exponent $d$.**

$$d = e^{-1} \bmod \phi(n)$$

$d$ is the modular multiplicative inverse of $e$ modulo $\phi(n)$, computed using the **Extended Euclidean Algorithm**. It satisfies $e \cdot d \equiv 1 \pmod{\phi(n)}$. This is the only step that requires knowing $\phi(n)$ — and it is why $\phi(n)$ must remain secret.

**Step 6 — Publish the public key; destroy and secure the private key.**

$$\text{Public key} = [e,\ n] \qquad \text{Private key} = [d,\ n]$$

The chain is complete: $p$ and $q$ are the root secret → they yield $\phi(n)$ → which yields $d$ → which enables decryption. Knowing $n$ alone (which the attacker has) breaks the chain at the very first link.

### How Primes Are Generated in Practice

To generate a large prime $p$ of $B/2$ bits, the procedure is:

1. Use a cryptographically secure random number generator to produce a random $B/2$-bit number. The quality of randomness here is not optional — it is the entire basis of the security claim.
2. Set the lowest bit to 1, ensuring the number is odd (all primes above 2 are odd, so even candidates are guaranteed composites).
3. Set the two highest bits to 1, ensuring the number genuinely has $B/2$ bits and that $p \times q$ will have the full $B$ bits needed.
4. Apply the **Miller-Rabin primality test** — a probabilistic test that, with repeated applications, confirms primality with overwhelming probability.
5. If the test fails, increment by 2 and try again. By the prime number theorem, primes are dense enough that a suitable candidate is found quickly.

The reason the high bits are forced is subtle: if the highest bit were 0, the prime would be effectively $B/2 - 1$ bits, and the product $n = pq$ might have fewer than $B$ bits — weakening security without any warning.

### Choosing the Public Exponent $e$

The condition $\gcd(e, \phi(n)) = 1$ is equivalent to requiring:
- $\gcd(e, p-1) = 1$ — equivalently $p \bmod e \neq 1$
- $\gcd(e, q-1) = 1$ — equivalently $q \bmod e \neq 1$

If a specific value of $e$ is preferred (say, $e = 65537$), one checks these conditions when generating $p$ and $q$ and rejects any prime that fails them. The prime generation step is interleaved with the $e$ selection.

---

## Part 5: RSA Encryption and Decryption

### Messages as Numbers in a Modular World

RSA treats the message as an integer in the range $0$ to $n-1$ — a number living in the modular universe defined by $n$. Encryption is not "hiding" in the everyday intuitive sense; it is raising that number to the public exponent inside a number system where only someone with the matching inverse exponent can reliably reverse the operation. The whole setup in Part 4 — choosing $e$, computing $d$ so that $ed \equiv 1 \pmod{\phi(n)}$ — was specifically to manufacture an exponent that composes back to "do nothing" for the key owner.

Why exponentiation modulo $n$ rather than some simpler invertible arithmetic — say, multiplication by $e$? Because simple linear operations can be inverted without secret knowledge. $C = M \cdot e \bmod n$ can be reversed as $M = C \cdot e^{-1} \bmod n$, and $e^{-1}$ can be computed by anyone who knows $e$ and $n$. Exponentiation in a modular world makes the inverse exponent $d$ computationally hidden behind the factoring problem. The choice of exponentiation is not cosmetic — it is the source of the asymmetry.

### Encryption

$$C = M^e \bmod n$$

### Decryption

$$M = C^d \bmod n$$

These are both **modular exponentiation** operations. The mathematical correctness follows from the construction in Part 3: $C^d = (M^e)^d = M^{ed} \equiv M \pmod{n}$ because $ed \equiv 1 \pmod{\phi(n)}$ implies $M^{ed} = M^{1+k\phi(n)} \equiv M \pmod{n}$.

### Why Decryption is Slower Than Encryption

$e$ is chosen to be small (e.g., 65537) with few set bits — exponentiation with $e$ requires at most $2 \log_2 e$ multiplications, roughly 34 for $e = 65537$. $d$, computed as a modular inverse of $e$, is roughly the same size as $n$ (hundreds or thousands of bits) with roughly half its bits set. Computing $C^d \bmod n$ requires on the order of $3n$-bit multiplications. This is the direct cost of having a large, secret inverse exponent — and it is unavoidable for the private-key operation.

---

## Part 6: Worked Example — RSA from Scratch

Let us work through RSA with a small toy example (16-bit modulus, 8-bit primes), narrating *why* each decision is made rather than just what the result is.

**Given:** $p = 197$, $q = 211$

**Step 1: Compute $n$:**
$$n = 197 \times 211 = 41{,}567$$

**Step 2: Compute $\phi(n)$:**
$$\phi(n) = (197 - 1)(211 - 1) = 196 \times 210 = 41{,}160$$

**Step 3: Choose $e$:**

We need $\gcd(e, 41160) = 1$ — that is, $e$ must have no common factor with $\phi(n)$, so that a modular inverse exists and decryption is possible.

- $e = 3$: $\gcd(3, 210) = 3 \neq 1$. Rejected — 3 divides $q - 1 = 210$, so no inverse $d$ exists modulo $\phi(n)$. Choosing this $e$ would make decryption impossible.
- $e = 5$: $\gcd(5, 210) = 5 \neq 1$. Rejected for the same reason — 5 also divides $q - 1$.
- $e = 17$: $\gcd(17, 196) = 1$ ✓ and $\gcd(17, 210) = 1$ ✓. Accepted — 17 shares no factor with $p-1$ or $q-1$, so a valid inverse $d$ exists. **Use $e = 17$.**

The rejections of $e = 3$ and $e = 5$ are not bad luck — they are examples of a general pattern. Whenever $e$ divides $p - 1$ or $q - 1$, the encryption function is "degenerate" in that modular world, and the inverse exponent does not exist.

**Step 4: Compute $d$:**

We need $17 \cdot d \equiv 1 \pmod{41160}$ — the inverse of 17 in the exponent cycle of length 41160.

The Extended Euclidean Algorithm traces back through the GCD computation of $\gcd(17, 41160)$ to express 1 as a linear combination of 17 and 41160. The algorithm does not merely name $d$ — it constructs it. The result: $d = 26{,}633$.

Verify: $17 \times 26633 = 452761 = 11 \times 41160 + 1$. ✓ (That $+1$ is exactly $17d \equiv 1 \pmod{\phi(n)}$.)

**Public key:** $[e = 17,\ n = 41567]$  
**Private key:** $[d = 26633,\ n = 41567]$

### Practice Encryption/Decryption (Small Numbers)

For exam-scale practice: $p = 3$, $q = 11$.

$$n = 33,\quad \phi(n) = 2 \times 10 = 20$$

Choose $e = 7$: $\gcd(7, 20) = 1$ ✓.  
Compute $d$: $7d \equiv 1 \pmod{20}$ → $d = 3$ (since $7 \times 3 = 21 \equiv 1 \pmod{20}$). ✓

Encrypt $M = 2$:
$$C = 2^7 \bmod 33 = 128 \bmod 33 = 128 - 3 \times 33 = 29$$

Decrypt $C = 29$:
$$M = 29^3 \bmod 33 = 24389 \bmod 33$$
$739 \times 33 = 24387$, so $24389 - 24387 = 2$. ✓

$M = 2$ — the original message is recovered.

---

### Practice Problems — RSA Key Generation

**Problem 1:** Given $p = 5$, $q = 11$. Which of $e = 3$, $e = 5$, $e = 10$ are valid choices and which fail? For the valid ones, compute $d$.

**Answer:**
- $n = 55$, $\phi(n) = 4 \times 10 = 40$
- $e = 3$: $\gcd(3, 40) = 1$ ✓ → valid. $d = 27$ (since $3 \times 27 = 81 = 2 \times 40 + 1$) ✓
- $e = 5$: $\gcd(5, 40) = 5 \neq 1$ → **invalid.** 5 divides $\phi(n)$, so no inverse $d$ exists. Choosing this $e$ makes decryption impossible.
- $e = 10$: $\gcd(10, 40) = 10 \neq 1$ → **invalid.** Same reason: 10 divides $\phi(n)$.

**Problem 2:** Given $p = 7$, $q = 13$. Why is $e = 5$ valid and $e = 6$ not? Predict what property decryption loses if you use the bad exponent.

**Answer:**
- $n = 91$, $\phi(n) = 6 \times 12 = 72$
- $e = 5$: $\gcd(5, 72) = 1$ ✓ → valid. $d = 29$ (since $5 \times 29 = 145 = 2 \times 72 + 1$) ✓. Encrypt: $C = 3^5 \bmod 91 = 243 \bmod 91 = 61$.
- $e = 6$: $\gcd(6, 72) = 6 \neq 1$ → **invalid.** 6 and 72 share factors 2 and 3. No integer $d$ satisfies $6d \equiv 1 \pmod{72}$. If you tried to use $e = 6$, you could encrypt but could never decrypt — there is no valid private key to compute.

---

## Part 7: Fast Modular Exponentiation

### Why We Need a Special Algorithm

RSA requires computing $M^e \bmod n$ where $e$ can be tens of thousands. Even $e = 65537$ means raising $M$ to a 65537-power. Computing $M^e$ naively — multiply by $M$ one step at a time — would require 65537 multiplications, each producing a number with thousands of digits that grows unboundedly. Worse, performing the modular reduction only at the end would mean intermediate numbers are astronomically large — billions of digits before reduction. That is computationally infeasible.

Two ideas together make this efficient. First: reduce modulo $n$ at every single multiplication step. Since $(ab) \bmod n = ((a \bmod n)(b \bmod n)) \bmod n$, you can reduce at any point without affecting the final result. Numbers never grow larger than $n^2$ in size. Second: decompose the exponent in binary and use repeated squaring to compute only $\log_2 e$ intermediate powers, then multiply together only those that correspond to set bits.

### The Algorithm (Square-and-Multiply)

The invariant that makes this algorithm obvious rather than magical: at every stage, `base` means "the current power of $A$ I might need next" — it starts at $A^1$ and doubles (squares) at every bit. And `result` means "the product of all the powers I have selected so far." We accumulate into `result` only when the current bit is 1, because that bit says this power is part of the binary decomposition of $B$.

Express the exponent $B$ in binary: $B = (b_k b_{k-1} \ldots b_1 b_0)_2$, so $A^B = \prod_{b_i = 1} A^{2^i}$.

```
result = 1
base = A mod n
for each bit b of B from least significant to most significant:
    if b == 1:
        result = (result × base) mod n
    base = (base × base) mod n       ← square to advance to next power
return result
```

We square `base` because moving one bit left in binary doubles the exponent — $A^{2^{i+1}} = (A^{2^i})^2$. We multiply into `result` only when the current bit is 1 because that bit says this particular power contributes to the decomposition. Once you see those two roles clearly, the algorithm is not a recipe to memorize — it is a consequence of binary notation applied to exponentiation.

Why must we reduce modulo $n$ at every step rather than postponing until the end? If you postpone, the intermediate values grow without bound — after $k$ squarings, `base` could have $2^k$ times as many digits as $n$. Even with $e = 65537$, that would produce a number with roughly $10^{20000}$ digits before reduction. The reduction at each step is not an optimization; it is what makes the computation feasible at all.

### Worked Example: $7^{11} \bmod 13$

$11 = (1011)_2 = 2^0 + 2^1 + 2^3$ — so bits 0, 1, and 3 are set.

| Step | Bit | Base $= 7^{2^i} \bmod 13$ | Multiply into result? |
|------|-----|---|----|
| $i=0$ | 1 | $7^1 = 7$ | result = $1 \times 7 = 7$ |
| $i=1$ | 1 | $7^2 = 49 \bmod 13 = 10$ | result = $7 \times 10 = 70 \bmod 13 = 5$ |
| $i=2$ | 0 | $10^2 = 100 \bmod 13 = 9$ | skip — bit is 0 |
| $i=3$ | 1 | $9^2 = 81 \bmod 13 = 3$ | result = $5 \times 3 = 15 \bmod 13 = 2$ |

$7^{11} \bmod 13 = 2$. At each row: `base` is the next power we might need; `result` accumulates exactly the powers at set-bit positions. The skipped row ($i=2$, bit 0) still squares `base` — we may not need $7^4$ now, but we need $7^8 = (7^4)^2$ in the next step.

---

## Part 8: RSA-CRT — Speeding Up Decryption

### The Costly Computation and Its Split

RSA decryption requires $M = C^d \bmod n$. The exponent $d$ is roughly the same size as $n$ — for 2048-bit RSA, $d$ is a 2048-bit number. Modular exponentiation with a $k$-bit modulus and $k$-bit exponent costs roughly $O(k^3)$ in bit operations, because each of the $O(k)$ multiplications involves numbers with $O(k)$ bits, and each such multiplication costs $O(k^2)$. At 2048 bits, this is measurable.

The **Chinese Remainder Theorem (CRT)** provides a ~4x speedup by splitting the decryption into two smaller exponentiations, each in a world half the size of $n$. The key owner knows $p$ and $q$, so they can work in the worlds mod $p$ and mod $q$ separately, then combine the results. An attacker who does not know $p$ and $q$ cannot do this — CRT decryption is only possible for the key owner.

### Why CRT Works — Satisfying Two Constraints at Once

The goal is to find $M = C^d \bmod n$. Instead of computing this directly, compute two partial results:

$$V_p = C^d \bmod p \qquad V_q = C^d \bmod q$$

These are much cheaper because $p$ and $q$ are each roughly half the size of $n$. Halving the modulus size reduces the cost of each multiplication by a factor of 4, and since there are also half as many multiplications (the exponent in the reduced world is shorter), the total cost drops by roughly a factor of 4 per partial computation.

Now the question is: how do we reconstruct $M = C^d \bmod n$ from $V_p$ and $V_q$? We need a single number $M$ in the range $[0, n)$ that simultaneously satisfies both:

$$M \equiv V_p \pmod{p} \qquad M \equiv V_q \pmod{q}$$

The CRT combination formula is engineered to satisfy exactly this. Define:

$$X_p = q \cdot (q^{-1} \bmod p) \qquad X_q = p \cdot (p^{-1} \bmod q)$$

These coefficients have a special property: $X_p \equiv 1 \pmod{p}$ and $X_p \equiv 0 \pmod{q}$ — it contributes exactly 1 in the $p$-world and exactly 0 in the $q$-world. Similarly, $X_q \equiv 0 \pmod{p}$ and $X_q \equiv 1 \pmod{q}$. They act like "selectors" for each modular world.

Then:

$$M = (V_p \cdot X_p + V_q \cdot X_q) \bmod n$$

Check: modulo $p$, the $V_q \cdot X_q$ term vanishes (since $X_q \equiv 0 \pmod p$), and $V_p \cdot X_p \equiv V_p \cdot 1 = V_p \pmod p$. Modulo $q$, the $V_p \cdot X_p$ term vanishes, and the result is $V_q$. So $M$ is exactly $V_p$ in the $p$-world and exactly $V_q$ in the $q$-world — the two partial results are "glued together" into a single global answer.

Once you see CRT as manufacturing a number that is exactly right in two separate modular worlds simultaneously, the formula stops looking arbitrary.

**Further speedup via Fermat's Little Theorem:** Since $p$ is prime, $a^{p-1} \equiv 1 \pmod{p}$ for $a$ not divisible by $p$ — the same exponent-cycle idea from Part 3, now applied in the smaller world mod $p$ instead of mod $n$. So $C^d \bmod p = C^{d \bmod (p-1)} \bmod p$. The exponent shrinks from roughly $\log_2 n$ bits to roughly $\log_2 p$ bits, halving the work again. This is the same underlying principle as RSA correctness — exponents cycle — now used inside the CRT sub-computations.

Combined, CRT + Fermat reduction brings decryption cost to roughly **one-quarter** of naive $C^d \bmod n$.

**Important:** The step that is NOT part of RSA-CRT decryption is dynamically factoring $n$ to find $p$ and $q$. The legitimate key holder already knows $p$ and $q$ — they were used to generate the key in the first place. Only an attacker who does not have the private key would need to factor $n$. CRT is a speed optimization for someone who already has the secret factors, not a shortcut for someone who does not.

---

### Exam Practice — RSA Key Generation and Operations
*(2025 Midterm Q30–Q38)*

**Q30:** What is the foundation of RSA's security?  
a) The difficulty of factoring large integers  
b) The hardness of solving discrete logarithms  
c) The complexity of hash collisions  
d) The secrecy of its S-boxes

**Answer: (a).** RSA security relies on the difficulty of factoring $n = pq$ into its prime factors. An attacker who could factor $n$ would recover $p$ and $q$, compute $\phi(n) = (p-1)(q-1)$, and immediately derive $d = e^{-1} \bmod \phi(n)$. (b) is the basis of Diffie-Hellman — a different hard problem. (d) is a symmetric cipher concept. This is the security question; the correctness question is answered separately by Euler's theorem.

---

**Q31:** What is the correct order for RSA key generation?  
a) Select $e$ → Choose $n$ → Factor $n$ → Compute $d$  
b) Generate $d$ → Find $e$ → Randomize $n$ → Pick $p, q$  
c) Compute $n$ → Solve for $p, q$ → Derive $e$ → Choose $d$  
d) Choose primes $p, q$ → Compute $n = pq$ → Select $e$ → Calculate $d$

**Answer: (d).** The order is a chain: primes $p, q$ are needed to compute $n$ and $\phi(n)$; $\phi(n)$ is needed to check $e$'s validity and compute $d$. You cannot go backward through this chain — $e$ must come before $d$ because $d$ is defined as $e$'s inverse.

---

**Q32:** In RSA, what condition must the public exponent $e$ satisfy?  
a) $e > n$  
b) $e$ must be prime  
c) $1 < e < \phi(n)$ and $\gcd(e, \phi(n)) = 1$  
d) $e \equiv 1 \bmod p$

**Answer: (c).** The coprimality condition $\gcd(e, \phi(n)) = 1$ is not a preference — it is a mathematical requirement. Without it, no inverse $d$ exists modulo $\phi(n)$, so decryption is literally impossible. $e$ need not be prime (though choices like 65537 happen to be prime).

---

**Q33:** How is the private exponent $d$ computed?  
a) $d \equiv e^{-1} \bmod \phi(n)$  
b) $d = n$  
c) $d = e \bmod p$  
d) $d$ is randomly selected

**Answer: (a).** $d$ is the modular multiplicative inverse of $e$ modulo $\phi(n)$, constructed by the Extended Euclidean Algorithm. Computing this requires knowing $\phi(n)$, which requires knowing $p$ and $q$. That is why $d$ cannot be found by an attacker who knows only $n$ and $e$.

---

**Q34:** Which RSA operation is computationally expensive?  
a) Decryption (using $d$)  
b) Encryption (using $e$)  
c) Generating $p$ and $q$  
d) Calculating $\gcd$

**Answer: (a).** Decryption requires $C^d \bmod n$. Since $d$ is as large as $n$ (typically 2048+ bits), this is expensive. Encryption with small $e$ (like 65537, only 17 bits) is fast. CRT can speed up decryption approximately 4x by splitting the work across the smaller moduli $p$ and $q$.

---

**Q35:** Why is RSA rarely used for bulk encryption?  
a) Lack of authentication  
b) Slow performance  
c) Fixed block size  
d) Inability to encrypt large files

**Answer: (b).** RSA's modular exponentiation is orders of magnitude slower than symmetric encryption. The mathematical hardness that makes it secure is also what makes it slow — you cannot separate the two. In practice, RSA handles only key exchange and authentication; AES encrypts the actual data with a session key established via RSA.

---

**Q36:** Which vulnerability arises from reusing the same $n$ with different $e$?  
a) Nonce reuse  
b) Timing attack  
c) Padding oracle attack  
d) Common modulus attack

**Answer: (d).** If two parties use the same $n$ but different public exponents $e_1$ and $e_2$ with $\gcd(e_1, e_2) = 1$, and the same message $M$ is encrypted with both: use Extended Euclidean to find $a, b$ with $e_1 a + e_2 b = 1$, then $C_1^a \cdot C_2^b \equiv M^{e_1 a + e_2 b} = M^1 = M \pmod{n}$. The message is recovered without factoring — RSA's algebra becomes a weapon when structural shortcuts are shared.

---

**Q37:** What is the primary purpose of using CRT with RSA?  
a) To increase the key size  
b) To enable quantum resistance  
c) To replace modular exponentiation  
d) To speed up decryption/signature generation by ~4x

**Answer: (d).** CRT splits $C^d \bmod n$ into $C^d \bmod p$ and $C^d \bmod q$, each operating on half-size moduli. Halving the modulus reduces each multiplication to $1/4$ the cost, and with Fermat's reduction the exponents are also halved. The total speedup is approximately 4x. This is the same exponent-cycling principle from the RSA mathematical foundation, now applied in the sub-rings mod $p$ and mod $q$.

---

**Q38:** Which step is NOT part of RSA-CRT decryption?  
a) Compute $m_p = C^{d_p} \bmod p$ and $m_q = C^{d_q} \bmod q$  
b) Factor $n$ into $p$ and $q$ dynamically  
c) Recombine results via CRT  
d) Verify the result modulo $n$

**Answer: (b).** The private key holder already knows $p$ and $q$ — they used them to generate the key. Dynamically factoring $n$ is what an *attacker* would need to do, not the legitimate decryptor. CRT is a speed optimization for someone who already has the secret factors.

---

## Part 9: RSA Security — Threats and Vulnerabilities

### A Map of RSA Failure Modes

RSA fails for different reasons depending on what assumption is violated. Grouping the attacks by failure type makes them much easier to understand and remember than treating them as an unrelated list.

**Category 1 — Mathematical hardness fails:** The factoring problem becomes tractable (e.g., quantum computers, or simply using too-short keys). This directly yields $p$ and $q$, and from them $\phi(n)$ and $d$.

**Category 2 — Algebraic misuse:** Raw, structured inputs to RSA expose mathematical relationships that leak the message without requiring factoring. This is textbook RSA's core design flaw.

**Category 3 — Randomness failure:** The supposedly-hard factoring problem collapses to a trivial computation when primes were generated with insufficient entropy.

**Category 4 — Protocol design limitation:** RSA is used correctly as a cipher, but its role in the protocol creates a long-term confidentiality problem.

### 1. The Mathematical Foundation — Factoring

RSA's security depends on the practical infeasibility of factoring $n$ into $p$ and $q$. Multiplying two large primes is easy; factoring the product is believed computationally hard for numbers of sufficient size. Various algorithms exist (trial division, Fermat's method, quadratic sieve, number field sieve, Pollard's rho), but none can practically factor numbers of the sizes used in modern RSA:

| RSA Challenge | Bits | Status |
|---|---|---|
| RSA-576 | 576 | Factored 2003 |
| RSA-640 | 640 | Factored 2005 |
| RSA-768 | 768 | Factored 2009 |
| RSA-1024 | 1024 | Not yet publicly factored |
| RSA-2048 | 2048 | \$200K prize — unclaimed |

Current recommendation: **2048-bit minimum**; 3072–4096 bits for long-term security.

### 2. Chosen Ciphertext / Small Exponent Attack (Category 2)

If $e$ is small (especially $e = 3$) and the message $M$ is short enough that $M^3 < n$ (the modular reduction never activates), then $C = M^3$ exactly — it is just an ordinary cube. An attacker computes $\sqrt[3]{C}$ to recover $M$ with no cryptanalysis at all.

The deeper issue is not the size of $e$ but the predictability of the input to the RSA function. Raw RSA is a mathematical bijection over integers mod $n$. When the input has predictable algebraic structure — because the message is short, repeated, or guessable — that structure survives into the ciphertext and can be exploited. The common thread across several RSA attacks is this: **textbook RSA is dangerous whenever the message representation is structured or repeated**, because RSA's algebra preserves and sometimes amplifies that structure.

**Defense:** Message padding (OAEP — Optimal Asymmetric Encryption Padding). Before encryption, $M$ is padded with random bytes to fill the full block size, destroying predictable structure and ensuring the input to RSA is always close to $n$ in size. Padding addresses the underlying problem — algebraic predictability of inputs — more fundamentally than simply changing $e$. Larger $e$ makes the direct cube-root attack harder; OAEP eliminates the exploitable structure regardless of $e$.

### 3. Common Modulus Attack (Category 2)

If two parties share the same modulus $n$ but use different public exponents $e_1$ and $e_2$ (with $\gcd(e_1, e_2) = 1$), and the same message $M$ is encrypted under both:

$$C_1 = M^{e_1} \bmod n \qquad C_2 = M^{e_2} \bmod n$$

Using the Extended Euclidean Algorithm, find $a$ and $b$ such that $e_1 a + e_2 b = 1$. Then:

$$C_1^a \cdot C_2^b \equiv M^{e_1 a + e_2 b} = M^1 = M \pmod{n}$$

The message is recovered without factoring — RSA's multiplicative structure becomes a weapon when structural shortcuts (shared moduli) are reused. This illustrates the broader principle: any algebraic shortcut that makes RSA key generation easier also tends to make RSA's algebra exploitable.

### 4. Low Entropy — Shared Prime Factor Attack (Category 3)

If two different RSA public keys $n_1$ and $n_2$ happen to share a common prime factor $p$, an attacker computes $\gcd(n_1, n_2) = p$ in milliseconds using the Euclidean algorithm, then factors both moduli immediately.

How does this happen? At device boot time — when network devices first generate RSA keys — the system entropy pool is often nearly empty. Devices with nearly identical state generate nearly identical random numbers, and some pairs accidentally generate the same prime. The "hard" factoring problem collapses to a trivial GCD because the assumption of random, independent prime generation was violated.

A 2012 study ("Mining your Ps and Qs") harvested over 5 million TLS/SSL certificates and 4 million SSH host keys, and was able to compute private keys for **0.50% of TLS servers** and **0.03% of SSH servers** using nothing but GCD computations. This illustrates a profound lesson: "hard factoring" assumes the modulus was generated sanely. Randomness failure retroactively destroys the entire security argument.

This also connects to why key extraction and analysis (Part 10) matters: scanning many public keys for shared factors requires being able to read the modulus $n$ from each key, which is exactly what parameter extraction tools do.

### 5. Perfect Forward Secrecy — A Protocol Design Limitation (Category 4)

When RSA is used to exchange a session key — Alice encrypts session key $K_S$ with Bob's public key — an eavesdropper who records all traffic can decrypt it later if they ever obtain Bob's private key (through a data breach, legal order, hardware seizure, or eventual advances in factoring).

This lack of **perfect forward secrecy (PFS)** means that breaking one long-term key retroactively breaks all past sessions that key was used to establish. The long-term private key is a single point of failure for all historical traffic.

The Diffie-Hellman key exchange (studied in Lecture 04) addresses this by generating **ephemeral keys** — short-lived values created fresh for each session and discarded immediately after. Even if the long-term authentication keys are later compromised, the per-session ephemeral values are gone and past sessions cannot be decrypted.

The connection to keys from Lecture 02 is direct: the same principle that motivated session keys in Needham-Schroeder and Kerberos — isolate ephemeral communication secrets from persistent trust roots — applies here. Long-term RSA keys are the trust roots; session keys should be ephemeral.

---

### Generated Practice — RSA Security

**Practice 1:** Why is a GCD scan across many public moduli such a devastating attack surface? Explain what hidden assumption in RSA key generation is violated when two moduli share a prime.

**Answer:** $\gcd(n_1, n_2)$ runs in time proportional to the key size — milliseconds per pair, seconds for millions of pairs. RSA's security claim is "factoring $n$ is hard." That claim assumes $p$ and $q$ were chosen randomly and independently. If poor entropy causes two devices to generate the same prime, GCD hands you the factorization instantly. The assumption was never met, so the security guarantee never applied. The entire hardness argument is conditional on proper randomness, not a property of the RSA math itself.

**Practice 2:** Why is "choose a larger $e$" only a partial fix for small-exponent attacks? What does padding address more fundamentally?

**Answer:** Larger $e$ means $M^e$ outgrows $n$ for most message sizes, so the modular reduction activates. But the underlying problem is not the size of the exponent — it is that raw RSA is algebraically transparent. If the message is guessable (e.g., one of a small set of values), an attacker can encrypt all candidates and compare to $C$. If the message has low entropy in any way, the algebraic structure leaks. OAEP padding randomizes the message before encryption, destroying the algebraic relationship between the plaintext and the resulting ciphertext. Padding addresses the root cause; increasing $e$ addresses one symptom.

**Practice 3:** Why does splitting one decryption into two smaller modular worlds make the work much cheaper? Explain in terms of operand size and the CRT reconstruction.

**Answer:** Each modular multiplication with a $k$-bit modulus costs $O(k^2)$ bit operations. Using modulus $p \approx n^{1/2}$ costs $O((k/2)^2) = O(k^2/4)$ per multiplication — 4x cheaper. The exponents also reduce (via Fermat) from $d$ to $d \bmod (p-1)$, roughly halving their length. Two partial exponentiations mod $p$ and mod $q$, each at 4x lower cost, give the same total answer for roughly one-quarter the work. CRT recombination is $O(k^2)$ at most — cheap compared to the exponentiations it replaces.

---

### Knowledge Check 9

**Q:** What are the key generation steps in order, and what does each step require from the previous one?  
**A:** (1) Choose primes $p$, $q$ — required to compute everything else. (2) Compute $n = pq$ — the public modulus. (3) Compute $\phi(n) = (p-1)(q-1)$ — required to check $e$ and compute $d$. (4) Choose $e$ with $\gcd(e, \phi(n)) = 1$ — required so that the inverse $d$ exists. (5) Compute $d = e^{-1} \bmod \phi(n)$ — the private exponent. (6) Public key $= [e, n]$. (7) Private key $= [d, n]$. Each step unlocks the next; the chain cannot be reordered.

**Q:** What must be kept secret, and what is the consequence of each leak?  
**A:** $p$ and $q$ — leaking either immediately gives the attacker $\phi(n)$ and thus $d$. $\phi(n)$ — leaking it immediately gives $d$. $d$ — the private decryption exponent; leaking it means the key is fully compromised. $n$ and $e$ are public by design.

**Q:** What is the "fast" part of fast modular exponentiation, and what breaks without it?  
**A:** Two ideas: (1) reduce modulo $n$ at every step — without this, intermediate values grow to billions of digits before reduction, making computation infeasible; (2) use binary decomposition with repeated squaring — without this, computing $M^e$ requires $e$ multiplications, absurd for $e = 65537$. Together, the algorithm requires only $2\log_2 e$ multiplications with numbers bounded in size by $n^2$.

---

## Part 10: Operational Details

### Key Representation — Why Each Layer Exists

An RSA key is mathematically just a tuple of large integers: $(n, e)$ for the public key, $(d, n)$ for the private key. But software needs a standard way to serialize those integers into a form that can be stored in files, transmitted over networks, and parsed by different implementations without ambiguity. Each encoding layer solves a specific part of that problem.

**ASN.1** (Abstract Syntax Notation One) describes the *structure* of the key data — which field is the modulus, which is the public exponent, which is the prime $p$, and so on. It is a schema that gives meaning to the integers.

**DER** (Distinguished Encoding Rules) turns that structured description into an unambiguous byte sequence. Given an ASN.1 structure, DER produces exactly one canonical byte sequence — no ambiguity, no optional encodings. This matters for security: if the serialization were ambiguous, two implementations might parse the same bytes differently, creating exploitable discrepancies.

**Base64** takes the DER byte sequence and encodes it as printable ASCII text. Raw bytes cannot be pasted into emails, configuration files, or web forms without corruption. Base64 makes the key copy-paste safe and human-readable in a minimal sense.

The result is what you see in practice — a `-----BEGIN RSA PUBLIC KEY-----` block in a `.pem` file. Peel off Base64, parse DER, interpret the ASN.1 structure, and you have the integers.

### Extracting RSA Parameters

Python tools (`pyasn1`, `cryptography` library) can extract $n$ and $e$ from an RSA public key file. This is useful not as an abstract exercise but because it is the first step of the shared-prime-factor attack described in Part 9: collect many public keys, extract the modulus $n$ from each, compute pairwise GCDs, and identify any pairs sharing a factor. Without being able to read $n$ from a key, the attack cannot begin. Keys are not opaque magic objects — they are structured data, and their structure is precisely what makes bulk analysis possible.

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

### "What Breaks If..." — Understanding Checks

**1. What breaks if you choose $e$ with $\gcd(e, \phi(n)) \neq 1$?**  
No inverse $d$ exists satisfying $ed \equiv 1 \pmod{\phi(n)}$. Decryption is not merely inconvenient — it is mathematically impossible. There is no exponent $d$ that can undo the encryption. The entire scheme fails to be a cipher.

**2. What breaks if $p = q$?**  
$n = p^2$, and $\sqrt{n} = p$ reveals both prime factors immediately. An attacker computes $p = \sqrt{n}$ in microseconds, then computes $\phi(p^2) = p^2 - p$, then finds $d$. The "hard factoring" problem collapses to a single square root. The distinct prime requirement prevents this trivial factorization.

**3. What breaks if you postpone modular reduction until after computing the full power?**  
Intermediate values grow without bound. After $k$ squarings with no reduction, `base` has $2^k$ times as many digits as $n$. For $e = 65537$, this means numbers with billions of digits before the final reduction — computationally infeasible to store or multiply. Reducing at every step is not an optimization; it is what makes the computation possible.

**4. What breaks if two users share the same modulus $n$ but use different exponents?**  
The common modulus attack applies: if the same message is encrypted under both $(n, e_1)$ and $(n, e_2)$ with $\gcd(e_1, e_2) = 1$, the message is recoverable with only public information. RSA's multiplicative algebra becomes exploitable whenever structural shortcuts are shared. Each key pair must have its own modulus.

**5. What breaks if RSA encrypts raw short messages without padding?**  
If $M^e < n$, the modular reduction never activates: $C = M^e$ exactly, and the attacker takes the $e$-th root. For any guessable message (low entropy), the attacker encrypts their guesses and compares to $C$. Padding with OAEP randomizes the message representation before encryption, ensuring both that $M^e$ always exceeds $n$ and that the ciphertext reveals no algebraic relationship to the plaintext.

**6. What breaks if RSA is used for key transport with a long-term key that is later compromised?**  
Every past session whose key was RSA-encrypted under that long-term key can now be decrypted retroactively. An eavesdropper who recorded encrypted traffic years ago simply decrypts all the session key wrappings and recovers the session keys. Ephemeral Diffie-Hellman prevents this: even if the long-term authentication keys are later compromised, the per-session DH values were discarded and cannot be recovered.

**7. What breaks if prime generation uses poor entropy and two devices share one prime factor?**  
$\gcd(n_1, n_2) = p$ in milliseconds. Both private keys are immediately broken. The hardness claim "factoring is infeasible" assumed the primes were chosen with unpredictable randomness. If that assumption fails, there is no mathematical barrier — the RSA security guarantee is conditional on the generation process, not a property of the integers themselves.

**8. What breaks if you know $n$ and $e$ but not $\phi(n)$?**  
Without $\phi(n)$, you cannot compute $d = e^{-1} \bmod \phi(n)$. Computing $\phi(n) = (p-1)(q-1)$ requires knowing $p$ and $q$, which requires factoring $n$. So $\phi(n)$ is the real hidden lever: the key owner knows it (they generated $p$ and $q$); the attacker does not (they have only $n$). Every barrier between the attacker and $d$ flows through $\phi(n)$ being unknowable without factoring.

### Comprehensive Review Questions

**1.** A server uses RSA with $p = 61$, $q = 53$. Compute $n$, $\phi(n)$. If $e = 17$, verify it is valid and compute $d$.  
**Answer:**
- $n = 61 \times 53 = 3233$
- $\phi(n) = 60 \times 52 = 3120$
- $\gcd(17, 3120) = 1$ ✓ (17 is prime and does not divide 3120)
- $d \equiv 17^{-1} \pmod{3120}$: Extended Euclid gives $d = 2753$ (verify: $17 \times 2753 = 46801 = 15 \times 3120 + 1$ ✓)

**2.** Why does RSA require $n$ to be a product of exactly two primes, not three or more?  
**Answer:** RSA still works mathematically with three or more primes. The problem is security: smaller prime factors are easier to find. With $n = pqr$ where each prime is $B/3$ bits instead of $B/2$ bits, factoring algorithms are significantly more effective. The two-prime choice maximizes the difficulty of factoring while keeping $\phi(n) = (p-1)(q-1)$ simple to compute.

**3.** Describe precisely why the common-modulus attack works algebraically.  
**Answer:** Given $C_1 = M^{e_1} \bmod n$ and $C_2 = M^{e_2} \bmod n$ with $\gcd(e_1, e_2) = 1$: Extended Euclidean gives $a, b$ with $e_1 a + e_2 b = 1$. Then $C_1^a \cdot C_2^b = (M^{e_1})^a \cdot (M^{e_2})^b = M^{e_1 a + e_2 b} = M^1 = M \pmod{n}$. The shared modulus means both ciphertexts live in the same algebraic structure, and the Bézout identity for the coprime exponents gives direct access to $M$.

---

*End of Lecture 03 Study Guide*  
*Next: Lecture 04 — More Public-Key Cryptography (DH, ElGamal, PKI, Certificates)*
