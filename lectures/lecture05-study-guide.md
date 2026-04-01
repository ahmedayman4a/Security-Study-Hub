# Lecture 05 — Cryptographically Secure Hashing
## CS464: Computer System Security — Complete Study Guide

---

## Part 1: What Is a Hash Function?

### The Core Idea

Imagine you have a long document — a contract, a firmware image, a password. You want a way to take that variable-length document and produce a short, fixed-length "fingerprint" that uniquely identifies it. That fingerprint is a **hash value** (also called a **hashcode**, **message digest**, or **digest**).

A **hash function** $h$ takes a variable-length input message $M$ and produces a fixed-length output:

$$h : \{0,1\}^* \to \{0,1\}^n$$

For example, SHA-256 accepts messages of essentially any length (up to $2^{64} - 1$ bits for SHA-256) and always outputs exactly 256 bits. SHA-512 accepts up to $2^{128}$ bits and always outputs 512 bits.

The key property: **any change to the input — even flipping a single bit — produces a completely different hash**. This is sometimes called the avalanche effect in hashing.

### Why Hash Functions Matter

Hash functions are used everywhere in security:
- **File integrity verification**: "Does this downloaded file match what the server sent? Compare hashes."
- **Password storage**: Instead of storing passwords in plaintext, store their hashes. When the user logs in, hash the entered password and compare.
- **Digital signatures**: Signing a large document directly with RSA is slow. Instead, hash the document (fast) and sign the hash (small). The signature is still binding because changing the document changes its hash.
- **Message authentication**: Prove that a message has not been tampered with in transit.
- **Blockchain/cryptocurrency**: Chain blocks together using their hashes; proof-of-work mining involves finding inputs that produce hashes with specific prefixes.

---

## Part 2: Properties of Cryptographically Secure Hash Functions

A hash function that is merely fast and deterministic is not automatically *secure*. For cryptographic use, three specific properties are required:

### Property 1: Pre-Image Resistance (One-Way Property)

Given a hash value $h$, it should be **computationally infeasible** to find any message $M$ such that $\text{hash}(M) = h$.

In plain terms: you cannot reverse the hash. You cannot work backwards from the fingerprint to the document. This is what makes it safe to store password hashes — even if an attacker gets the hash database, they cannot easily recover the original passwords.

You might wonder: can't you just try all possible inputs? In principle yes, but with a 256-bit hash, there are $2^{256}$ possible hash values, and finding a pre-image by brute force requires on average $2^{255}$ tries — computationally infeasible.

### Property 2: Weak Collision Resistance (Second Pre-Image Resistance)

Given a specific message $M_1$ and its hash $h(M_1)$, it should be computationally infeasible to find a **different** message $M_2 \neq M_1$ such that:
$$h(M_2) = h(M_1)$$

This protects against an attacker substituting one specific document with a different document that has the same hash. If this property fails, an attacker could replace a signed contract (which has a digital signature based on its hash) with a forged version that has the same hash — and the signature would still verify.

### Property 3: Strong Collision Resistance

It should be computationally infeasible to find **any two different messages** $M_1 \neq M_2$ such that:
$$h(M_1) = h(M_2)$$

The attacker gets to choose both $M_1$ and $M_2$ freely. This is a strictly harder requirement than weak collision resistance, because the attacker is not constrained to start from a specific message.

You might wonder: are these three properties related? Yes:
- **Strong collision resistance implies weak collision resistance.** If you can find $M_2 \neq M_1$ given $M_1$ (weak collision), you have certainly found a collision — any collision satisfies the strong definition too. So if finding any collision is hard, finding one from a fixed starting point must also be hard. Formally: strong collision resistance ⊇ weak collision resistance.
- **Pre-image resistance is independent** of both collision properties — a hash function can be collision resistant but not pre-image resistant, or vice versa.

### The Hash Space: Why $n$ bits Matters

If the hash output is $n$ bits, there are only $2^n$ distinct possible hash values. Since inputs are infinite but outputs are finite, **collisions must exist** — by the pigeonhole principle. The security question is not whether collisions exist, but whether finding them takes infeasible effort.

---

### Knowledge Check 1

**Q:** Explain the difference between pre-image resistance and weak collision resistance in concrete terms.  
**A:** Pre-image resistance: given a hash $h$, cannot find any $M$ with $\text{hash}(M) = h$. Weak collision resistance: given a specific $M_1$ (and its hash), cannot find a different $M_2$ with the same hash. The key difference is direction: pre-image starts from a hash and goes to a message; weak collision starts from a message and finds another with the same hash.

**Q:** Does strong collision resistance imply weak collision resistance? Why?  
**A:** Yes. If strong collision resistance holds (cannot find any two messages with the same hash), then in particular you cannot find a second message that collides with a given specific message. Weak is a special case of strong — being unable to find any collision is a stronger requirement than being unable to find a collision from a fixed starting point.

---

## Part 3: Message Authentication — Six Schemes

Before going deeper into hash construction, let us understand how hashes are actually used for message authentication. There are six standard schemes, each providing different security guarantees.

Let $M$ be the message, $K$ a shared symmetric key, $PR_A$ the sender A's private key, $h(M)$ the hash of $M$, and $S$ a shared secret string.

### Scheme 1: Authentication + Confidentiality (Symmetric)
$$A \to B: C = E(K,\ [M\ ||\ h(M)])$$
A concatenates the message with its hash, then encrypts everything with the shared key $K$. B decrypts, splits off the hash, recomputes $h(M)$ from the received $M$, and checks they match.
- **Provides:** Authentication (hash proves integrity) + Confidentiality (encrypted).

### Scheme 2: Authentication Only (MAC with Symmetric Key)
$$A \to B: C = M\ ||\ E(K,\ h(M))$$
The message is sent in cleartext; only the hash is encrypted with the shared key $K$. Only B (who has $K$) can verify the hash. This is a **Message Authentication Code (MAC)** — also called a cryptographic checksum.
- **Provides:** Authentication only. An eavesdropper can read $M$ but cannot forge the encrypted hash.

### Scheme 3: Digital Signature (Authentication with Private Key)
$$A \to B: C = M\ ||\ E(PR_A,\ h(M))$$
A encrypts the hash with its own private key. Anyone with A's public key can verify. Only A could have created this.
- **Provides:** Authentication, integrity, and **non-repudiation**. (A cannot deny sending it.)

### Scheme 4: Digital Signature + Symmetric Confidentiality
$$A \to B: C = E(K,\ [M\ ||\ E(PR_A,\ h(M))])$$
A first creates a digital signature on the hash, then encrypts the whole bundle (message + signature) with the symmetric key $K$.
- **Provides:** Authentication + non-repudiation + confidentiality.

### Scheme 5: Authentication via Shared Secret (No Encryption)
$$A \to B: C = M\ ||\ h(M\ ||\ S)$$
$S$ is a shared secret known only to A and B. The hash is computed over the concatenation of the message and the secret. The message travels in cleartext.
- **Provides:** Authentication without any encryption algorithm. B verifies by computing $h(M || S)$ and comparing.
- **Note:** This is the simplest approach. An attacker who sees $M$ and $h(M||S)$ cannot modify $M$ without knowing $S$.

### Scheme 6: Shared Secret + Confidentiality
$$A \to B: C = E(K,\ M\ ||\ h(M\ ||\ S))$$
Combines Scheme 5 with symmetric encryption of the whole message.
- **Provides:** Authentication + confidentiality.

### Sending a Hash Without Encryption Is Useless
A critical point: if you send $M || h(M)$ with no encryption or shared secret, it provides **zero security**. An attacker can modify $M$ to $M'$, recompute $h(M')$, and send $M' || h(M')$. B has no way to detect the forgery. The hash must be somehow protected — either encrypted with a key, or computed over a shared secret.

---

## Part 4: Simple Hash Functions and Their Weaknesses

Before studying real cryptographic hash functions, it is instructive to look at naive designs and why they fail. This builds intuition for what makes the secure constructions work.

### The XOR Hash

The simplest possible hash: partition the message into $n$-bit blocks $X_1, X_2, \ldots, X_m$ and XOR them all together:

$$\Delta(M) = X_1 \oplus X_2 \oplus \cdots \oplus X_m$$

This is trivially broken. If an attacker wants to replace the message with a different message $Y_1, Y_2, \ldots, Y_{m-1}, Y_m$ that has the same hash, they simply choose any $Y_1, \ldots, Y_{m-1}$ freely and then set:

$$Y_m = Y_1 \oplus Y_2 \oplus \cdots \oplus Y_{m-1} \oplus \Delta(M)$$

This ensures $Y_1 \oplus \cdots \oplus Y_m = \Delta(M)$. The hash is unchanged, but the entire message content is different.

### The Rotated-XOR (ROXR) Hash

A slightly better attempt: after processing each block, rotate the partial hash one bit circularly before XORing the next block. This introduces some position-sensitivity (the same block in different positions produces different contributions). But it still fails: an attacker can append a block of carefully chosen "gibberish" to a forged message $M_2$ that forces the final hash to equal the original $h(M_1)$.

### The Fix: Include Message Length

Both attacks above exploit the fact that the attacker can freely choose blocks. A partial fix: **include the message length in what gets hashed**. This prevents appending gibberish blocks to equalize hashes, because changing the length changes the hash. All production hash functions (SHA-1, SHA-256, etc.) incorporate the message length in the final padded block.

---

## Part 5: SHA-1 Padding

Before a message can be processed by SHA-1, it must be **padded** to a multiple of 512 bits. The padding scheme is:

**Given:** message $M$ of $L$ bits.

**Step 1:** Append a single `1` bit at the end of the message.

**Step 2:** Append $K$ zero bits, where $K$ is the smallest non-negative integer satisfying:

$$(L + 1 + K) \bmod 512 = 448$$

This ensures that after padding, 64 bits remain free at the end of the block.

**Step 3:** Append the 64-bit binary representation of the original message length $L$.

The result is a message whose total length is a multiple of 512 bits, with the last 64 bits being the original length.

**Why 448?** Because $448 + 64 = 512$. The last 64 bits are reserved for the length field. The padding (1 bit + zeros) fills the gap between the message end and position 448.

**Why include the length?** Length inclusion prevents the length extension attack: an attacker who appends gibberish to a message would change $L$, which changes the padded input, which (almost certainly) changes the hash.

### Worked Example

Suppose $L = 1000$ bits. We need $L + 1 + K \equiv 448 \pmod{512}$:
$1000 + 1 + K \equiv 448 \pmod{512}$
$K \equiv 448 - 1001 \equiv 448 - 489 \equiv -41 \equiv 471 \pmod{512}$

So $K = 471$ zero bits. Total padded length = $1000 + 1 + 471 + 64 = 1536 = 3 \times 512$ bits ✓.

---

## Part 6: Birthday Attack — The Mathematics of Collision Finding

### Weak Collision (Pre-image Attack Effort)

How many messages must an attacker try to find one that matches a given hash $h$? If the hash has $n$ bits, there are $N = 2^n$ possible hash values (assumed equiprobable). For each random message $x$:
- Probability of matching $h$: $1/N$
- Probability of NOT matching: $1 - 1/N$
- Probability that a pool of $k$ messages contains NO match: $(1 - 1/N)^k$
- Probability of at least one match: $1 - (1-1/N)^k$

Setting this equal to 0.5 and solving:

$(1 - 1/N)^k = 0.5$, so $k \approx 0.5N = 2^{n-1}$

To break weak collision resistance (find a pre-image), an attacker needs approximately $2^{n-1}$ tries — effectively $2^n/2$.

### Strong Collision — The Birthday Paradox

The birthday paradox is a well-known probability puzzle: in a group of 23 randomly chosen people, the probability that two share the same birthday is already greater than 50%. This seems counterintuitive because we are not asking "does anyone share MY birthday" — we are asking "does any pair share a birthday."

For hash functions: a **strong collision attack** asks "do any two messages in a pool have the same hash?" — equivalent to the birthday question.

Probability that a pool of $k$ randomly generated messages has at least one collision:

$$P(k) = 1 - \frac{N!}{(N-k)! \cdot N^k}$$

The denominator $M_2 = N^k$ counts all pools of $k$ messages with repetitions allowed. The numerator portion $M_1 = N \cdot (N-1) \cdots (N-k+1) = N!/(N-k)!$ counts pools with no repeated hashes.

Using the approximation $(1-x) \approx e^{-x}$ for small $x$:

$$P(k) \approx 1 - e^{-k(k-1)/(2N)}$$

Setting $P(k) = 0.5$ and solving for large $k$:

$$k \approx \sqrt{N} = \sqrt{2^n} = 2^{n/2}$$

This is the **birthday bound**: to find a strong collision (any two matching messages), an attacker needs approximately $2^{n/2}$ tries — dramatically fewer than the $2^{n-1}$ needed for a pre-image attack.

### Birthday Attack on Digital Signatures

The birthday attack can be weaponized against digital signatures. Scenario:

1. Mr. Creepy wants to trick Mr. BigShot into signing a fraudulent contract.
2. Mr. Creepy creates $k \approx 2^{n/2}$ variations of a legitimate contract $\{c_1, c_2, \ldots, c_k\}$ (changing whitespace, synonyms, punctuation — semantically identical but bit-different).
3. Mr. Creepy also creates $k$ variations of a fraudulent contract $\{f_1, f_2, \ldots, f_k\}$.
4. With probability ≈ 0.5, there exists a pair $(c_i, f_j)$ such that $h(c_i) = h(f_j)$.
5. Mr. Creepy gets BigShot to sign $c_i$ (the legitimate-looking version).
6. Mr. Creepy replaces the signed contract with $f_j$ (the fraudulent version). The signature still verifies because the hashes match!

This attack requires $k \approx 2^{n/2}$ pairs, which is why SHA-1 (160-bit hash, $k \approx 2^{80}$) is considered borderline, and SHA-256 ($k \approx 2^{128}$) is the current minimum recommendation.

### Key Insight: Weak vs. Strong Collision Security Levels

| Attack Type | What attacker finds | Required effort |
|-------------|--------------------|----|
| Pre-image (weak collision) | Any $M$ with hash $= h$ | $\approx 2^n$ |
| Second pre-image (weak collision) | $M_2 \neq M_1$ with $h(M_1) = h(M_2)$ | $\approx 2^n$ |
| Birthday (strong collision) | Any pair $M_1 \neq M_2$ with same hash | $\approx 2^{n/2}$ |

The birthday attack halves the effective security level. A 256-bit hash provides only 128-bit collision resistance against birthday attacks. This is why hash outputs need to be roughly twice as long as the equivalent symmetric key security level.

---

### Knowledge Check 2

**Q:** What is the birthday bound for a hash function with $n = 128$ bits? What does it imply?  
**A:** The birthday bound is $2^{128/2} = 2^{64}$ tries to find a collision with 50% probability. This means a 128-bit hash has only 64-bit effective security against birthday/strong-collision attacks. This is why MD5 (128-bit) is considered broken — $2^{64}$ operations is within reach of modern hardware.

**Q:** Why does including the message length in the hash input help security?  
**A:** It prevents the "appending gibberish" attack. An attacker who appends blocks to a message to manipulate the hash must also match the original message length — which they cannot do if they've added extra data. Every production hash function includes the length as part of the padded input.

---

## Part 7: Merkle-Damgård Construction

### The Structure

Real cryptographic hash functions use a structure proposed by Ralph Merkle in 1979 (independently also by Ivan Damgård). This is the basis for MD5, SHA-1, SHA-2 (SHA-256, SHA-512), and Whirlpool.

**How it works:**
1. Pad the input message and partition it into $L$ blocks of $b$ bits each. The final block includes the message length.
2. Initialize a fixed **Initialization Vector (IV)** — a fixed starting value that is part of the hash function's specification.
3. Process blocks one by one through a **compression function** $f$:
   - $H_0 = IV$
   - $H_i = f(H_{i-1}, B_i)$ for $i = 1, 2, \ldots, L$
4. The final $H_L$ is the hash output.

The compression function $f$ takes the $b$-bit current block and the $n$-bit previous hash state, and produces a new $n$-bit hash state. Since $b > n$, $f$ is indeed a **compression** function (more bits in than out).

### Why This Works

The Merkle-Damgård construction has the property that if the compression function $f$ is collision resistant, then the overall hash function is collision resistant. This is a formal security reduction — it lets cryptographers focus on building a secure compression function rather than reasoning about the entire hash function at once.

### Initialization Vector

The IV is a fixed set of constants (published as part of the standard) used to initialize the hash state $H_0$. It is **not a secret** — unlike an IV in CBC mode, the hash IV is the same for every computation of the same hash function. Its role is to start the hash computation from a defined, non-trivial state.

---

## Part 8: The SHA Family

### SHA-1

SHA-1 (Secure Hash Algorithm 1) produces a **160-bit** hash. It was developed by the NSA and standardized by NIST.

Timeline:
- **2005:** Theoretically broken — researchers demonstrated a method requiring $2^{69}$ operations (less than brute force $2^{80}$) to find a collision.
- **2010:** NIST withdrew its approval for government applications.
- **2017:** SHAttered — Google researchers produced the first real-world SHA-1 collision using GPU clusters, publishing two different PDF files with identical SHA-1 hashes.

Despite this, SHA-1 continues to be widely used in SSL/TLS (legacy), PGP, SSH, S/MIME, and IPsec. This is a significant real-world security concern.

### SHA-2

SHA-256, SHA-384, and SHA-512 are collectively called **SHA-2**. As of 2024, SHA-2 has no known practical attacks and remains the NIST-recommended standard.

### SHA-512 Internal Structure

SHA-512 processes messages in 1024-bit blocks:

**Step 1 — Pad the message:**
- Block size $b = 1024$ bits.
- Last 128 bits of the final block encode the message length (SHA-512 reserves 128 bits, not 64 like SHA-1).

**Step 2 — Generate the message schedule:**
- Each 1024-bit block contains $16 \times 64$-bit words.
- The message schedule expands these 16 words into **80 words** using a defined mixing function.

**Step 3 — Round-based processing:**
- Each block is processed through **80 rounds** of a compression function using the 80-word schedule.

**Step 4 — Update hash registers:**
- Eight **64-bit registers** ($a, b, c, d, e, f, g, h$) hold the running hash state. After all $L$ blocks are processed, these 8 registers concatenated form the 512-bit final hash.

### SHA Family Comparison

| Algorithm | Output size | Block size | Rounds | Security Level |
|-----------|------------|------------|--------|---------------|
| SHA-1 | 160 bits | 512 bits | 80 | Broken (2017) |
| SHA-256 | 256 bits | 512 bits | 64 | 128-bit against birthday |
| SHA-384 | 384 bits | 1024 bits | 80 | 192-bit against birthday |
| SHA-512 | 512 bits | 1024 bits | 80 | 256-bit against birthday |

---

## Part 9: Message Authentication Codes (MAC) and HMAC

### Why a Simple Hash with Secrecy Still Fails

You might think: just append a secret key to the message and hash — $\text{MAC}(K, M) = h(K || M)$. Unfortunately, the Merkle-Damgård construction has a length extension vulnerability. Given $h(K || M)$, an attacker can compute $h(K || M || \text{padding} || M')$ for any $M'$ without knowing $K$. This is because the Merkle-Damgård structure means the hash state after processing $K || M$ is the IV for the next block.

### Simple XOR-based MAC Is Broken

Consider the XOR-based MAC: for a message $M = X_1 || X_2 || \cdots || X_m$ (64-bit blocks):
$$\text{MAC}(K, M) = E(K, X_1 \oplus X_2 \oplus \cdots \oplus X_m)$$

An attacker who observes $\{M, \text{MAC}(K, M)\}$ can replace blocks $X_1, \ldots, X_{m-1}$ with any desired $Y_1, \ldots, Y_{m-1}$, then set:
$$Y_m = Y_1 \oplus Y_2 \oplus \cdots \oplus Y_{m-1} \oplus \Delta(M)$$

This makes the XOR equal $\Delta(M)$, so $E(K, \Delta(M_{\text{forged}})) = E(K, \Delta(M)) = \text{MAC}(K, M)$. The recipient accepts the forged message with the original MAC.

### HMAC — The Standard Solution

**HMAC** (Hash-based MAC) is the standardized, provably secure way to construct a MAC from a hash function. It is used in IPsec, SSL/TLS, and many other protocols.

The HMAC construction specifically defends against length extension attacks by hashing the key twice — once as an "inner" padding and once as an "outer" padding:

$$\text{HMAC}_K(M) = h\!\left( (K^+ \oplus \text{opad})\ ||\ h\!\left( (K^+ \oplus \text{ipad})\ ||\ M \right) \right)$$

Where:
- $K^+$ is the key $K$ zero-padded to length $b$ (the hash block size).
- $\text{ipad}$ = the byte `0x36` ($00110110_2$) repeated $b/8$ times.
- $\text{opad}$ = the byte `0x5C` ($01011100_2$) repeated $b/8$ times.

The inner hash $h((K^+ \oplus \text{ipad}) || M)$ produces an $n$-bit intermediate hash. The outer hash wraps this with the key again, preventing length extension.

The MAC output size equals the underlying hash size ($n$ bits).

### HMAC vs. Digital Signature

| Feature | HMAC | Digital Signature |
|---------|------|------------------|
| Key type | Symmetric (shared secret) | Asymmetric (private key) |
| Non-repudiation | No — both parties have the same key | Yes — only the signer has the private key |
| Verification | Anyone with the shared key | Anyone with the signer's public key |
| Speed | Fast | Slower (modular exponentiation) |
| Certificate required | No | Usually yes |

HMAC provides **authentication and integrity** but not non-repudiation. Digital signatures provide all three plus non-repudiation.

---

## Part 10: Cryptocurrency and Hashing

Hash functions underpin cryptocurrency systems in multiple ways:

1. **Proof-of-work ("mining")**: To add a block to a blockchain, miners must find a nonce $n$ such that $h(\text{block header} || n)$ has a certain number of leading zeros. This is computationally hard (brute force) but trivially verifiable. It is the computational basis of Bitcoin.

2. **Ownership authentication**: Each cryptocurrency transaction is signed by the owner's private key. The signature is verified using a hash of the transaction data.

3. **Ownership transfer**: Transferring a coin involves creating a new transaction signed by the current owner, creating a hash chain of ownership.

4. **Double-spending prevention**: The blockchain's hash-linked structure means modifying any past transaction would invalidate all subsequent blocks, making fraud computationally infeasible on a large network.

---

## Part 11: Exam Practice — Q49–Q60

### Questions of Doubt Clarifications

**Q52 is marked "None Correct":** The question asked for the definition of "weak collision resistance" but all four options described something else:
- (a) describes **strong** collision resistance
- (b) is wrong (easy to modify ≠ any property)
- (c) describes **pre-image resistance**
- (d) is wrong (no key required)

The correct definition of weak collision resistance (second pre-image resistance) is: *given a specific message $M_1$ and its hash $h(M_1)$, it is computationally infeasible to find $M_2 \neq M_1$ such that $h(M_2) = h(M_1)$.*

**Q54:** Strong collision resistance does imply weak collision resistance. Answer: **(c)**.

---

**Q49:** What is the primary purpose of a cryptographic hash function?  
a) Generate a fixed-size output from variable-size input  
b) Encrypt data for secure transmission  
c) Provide compression for large files  
d) Create digital signatures

**Answer: (a).** A hash function maps any-length input to a fixed-length digest. (b) is wrong — hashing is one-way and keyless (unlike encryption). (c) is wrong — the "compression" is in the mathematical sense (large input to small output), not file-size compression like zip. (d) is wrong — hashing is a component of digital signatures but is not itself a signature scheme.

---

**Q50:** What does "collision resistance" mean in hash functions?  
a) It should be hard to find two different inputs with the same hash  
b) It should be easy to reverse the hash output  
c) The hash output should always be unique  
d) The hash function should use a secret key

**Answer: (a).** Collision resistance (strong collision resistance) means it is computationally infeasible to find any pair $M_1 \neq M_2$ with $h(M_1) = h(M_2)$. (b) is the opposite — reversibility would be a catastrophic weakness. (c) is impossible by the pigeonhole principle — with infinite inputs and finite outputs, collisions must exist; the goal is to make them hard to find. (d) is wrong — hash functions are public and keyless.

---

**Q51:** Which attack involves finding any two inputs that produce the same hash?  
a) Pre-image attack  
b) Birthday attack  
c) Brute-force attack  
d) Man-in-the-middle attack

**Answer: (b).** A birthday attack exploits the birthday paradox to find any two messages with the same hash (strong collision), requiring only $\approx 2^{n/2}$ tries. (a) pre-image attack finds a message with a specific given hash (much harder, $\approx 2^n$ tries). (c) brute force is the general strategy, but "birthday attack" specifically names the technique of looking for any collision among a pool.

---

**Q52:** What is "weak collision resistance" in hash functions?  
a) It should be hard to find two different inputs with the same hash  
b) It should be easy to modify an input without changing its hash  
c) Given a hash output, it should be hard to find any input that produces it  
d) The hash function should use a secret key

**Answer: None of the above (exam marked as disputed).**  
- (a) describes **strong** collision resistance.
- (c) describes **pre-image resistance**.
- Correct definition of weak collision resistance (second pre-image resistance): *Given a specific message $M_1$, it is hard to find $M_2 \neq M_1$ such that $h(M_1) = h(M_2)$.* This was not among the options.

---

**Q53:** What is the difference between weak (second pre-image) and strong collision resistance?  
a) Weak deals with fixed inputs, while strong deals with any two inputs  
b) Weak is about reversing the hash; strong is about finding any collision  
c) Weak requires a secret key; strong does not  
d) There is no difference

**Answer: (a).** Weak (second pre-image) resistance: given a **fixed** $M_1$, cannot find another $M_2$ with the same hash. Strong collision resistance: cannot find **any** two messages (free choice) with the same hash. The attacker's freedom is the key difference — in weak, $M_1$ is given; in strong, both are chosen freely.

---

**Q54:** If a hash function is strong collision-resistant, does it imply weak collision resistance?  
a) No, they are independent  
b) Only if also pre-image resistant  
c) Yes, because strong is a stronger property  
d) Only for keyed hash functions

**Answer: (c).** Strong collision resistance says: cannot find any two messages with the same hash. Weak (second pre-image) resistance says: given $M_1$, cannot find $M_2$ with same hash. Strong implies weak — if you cannot find any colliding pair, you certainly cannot find one that starts from a specific $M_1$. The strong property is strictly harder to violate, so it subsumes the weaker one.

> **Questions of Doubt:** This is confirmed correct by the course materials — the answer is (c).

---

**Q55:** What is the most common structural design used in traditional hash functions like SHA-1 and SHA-2?  
a) Sponge construction  
b) Merkle-Damgård construction  
c) Feistel network  
d) Davies-Meyer construction

**Answer: (b).** SHA-1, SHA-2 (SHA-256, SHA-512), MD5, and Whirlpool all use the Merkle-Damgård construction — processing blocks iteratively through a compression function, starting from an IV. (a) Sponge construction is used by SHA-3/Keccak. (c) Feistel networks are used in DES. (d) Davies-Meyer is a specific method of building a compression function from a block cipher, which can be used within Merkle-Damgård.

---

**Q56:** Which component of a hash function processes one block at a time and updates the internal state?  
a) Initialization Vector (IV)  
b) Permutation function  
c) Key schedule  
d) Compression function

**Answer: (d).** The **compression function** $f$ takes the current $b$-bit message block and the $n$-bit previous state, and produces a new $n$-bit state. It is called a compression function because $b > n$ (more bits in than out). The IV is the initial state, not the processor. There is no key schedule in a hash function.

---

**Q57:** What is the role of the "initialization vector (IV)" in a hash function?  
a) Acts as a secret key  
b) Provides the initial fixed value for hash computation  
c) Ensures collision resistance  
d) Shortens the output hash

**Answer: (b).** The IV is a fixed constant (specified in the standard, same for every use of the hash function) that initializes the internal hash state $H_0$ before the first block is processed. It is not secret — unlike IVs in block cipher modes (CBC, CTR), the hash IV is public and standardized. It starts the computation from a defined, non-trivial point.

---

**Q58:** Which of the following components does HMAC require?  
a) A public key and a block cipher  
b) An IV  
c) A secret key and a hash function  
d) A digital certificate

**Answer: (c).** HMAC requires exactly two ingredients: a **secret key** $K$ (symmetric, shared between parties) and an **underlying hash function** (e.g., SHA-256). No public key, no block cipher (directly), no certificate. This is why HMAC is much simpler and faster than digital signatures.

---

**Q59:** How does HMAC differ from a digital signature?  
a) HMAC uses symmetric keys; signatures use asymmetric keys  
b) HMAC is faster but provides non-repudiation  
c) Signatures are shorter than HMAC outputs  
d) HMAC requires a certificate authority

**Answer: (a).** HMAC uses a **symmetric (shared) secret key** — both sender and receiver hold the same key. Digital signatures use **asymmetric keys** — the private key signs, the public key verifies. This means HMAC does NOT provide non-repudiation (both parties could have created the MAC), while signatures DO (only the holder of the private key can sign). (b) is wrong about non-repudiation. (c) is wrong — HMAC output is as large as the hash digest (e.g., 256 bits for HMAC-SHA-256), not shorter than signatures.

---

**Q60:** What is the formula for HMAC?  
a) $\text{HMAC}(K, m) = H(K || m)$  
b) $\text{HMAC}(K, m) = H(K \oplus \text{opad}\ ||\ H(K \oplus \text{ipad}\ ||\ m))$  
c) $\text{HMAC}(K, m) = H(m || K)$  
d) $\text{HMAC}(K, m) = \text{Encrypt}(K, H(m))$

**Answer: (b).** The correct HMAC formula uses two-level hashing with XOR pads:
$$\text{HMAC}_K(M) = h\!\left( (K^+ \oplus \text{opad})\ ||\ h\!\left( (K^+ \oplus \text{ipad})\ ||\ M \right) \right)$$
Options (a) and (c) are naive constructions vulnerable to length extension attacks. (d) is MAC using block cipher encryption of the hash — a different construction (not HMAC). The double-hashing structure in (b) is what makes HMAC secure against length extension.

---

## Part 12: Generated Practice — Hashing and MAC

**Practice 1:** A hash function produces 80-bit outputs. How many messages must an attacker hash (approximately) to find a pre-image? How many to find any collision (birthday attack)?  
**Answer:** Pre-image: $\approx 2^{80}$ tries (must match a specific hash). Birthday: $\approx 2^{80/2} = 2^{40}$ tries (find any matching pair). The birthday bound makes the effective collision security 40 bits — quite weak.

**Practice 2:** Explain why $\text{MAC}(K, M) = H(K || M)$ is insecure due to length extension, using the Merkle-Damgård structure.  
**Answer:** After the hash function processes the blocks of $K || M$ and produces output $H(K || M)$, the internal hash state is that output. An attacker can continue feeding new blocks to the Merkle-Damgård compression function, starting from state $H(K || M)$, without knowing $K$. This lets them compute $H(K || M || \text{padding} || M')$ for any extension $M'$ — a valid MAC for the extended message — even without knowing $K$. HMAC's outer hash wraps the result with the key again, breaking this chain.

**Practice 3:** In the birthday attack on digital signatures, why does the attacker need two sets of variations (legitimate and fraudulent), rather than just one set?  
**Answer:** The attacker needs a $(c_i, f_j)$ pair where both $c_i$ and $f_j$ look legitimate to the victim — $c_i$ must get signed, and $f_j$ must be the substitution. If only one set, the attacker would need a self-collision (a message that collides with another version of itself), which is harder to arrange semantically. With two sets of size $k$, the probability of finding a cross-set collision is $1 - (1 - 1/N)^{k^2} \approx k^2/N$, which equals 0.5 when $k \approx \sqrt{N} = 2^{n/2}$.

---

## Part 13: Final Exam Preparation — Lecture 05

### Relevant 2025 Midterm Questions — Lecture 05 Index

| Question | Topic | Key Concept |
|----------|-------|-------------|
| Q49 | Hash function purpose | Fixed-size output from variable input |
| Q50 | Collision resistance | Hard to find any two inputs with same hash |
| Q51 | Birthday attack | Finding any pair with same hash |
| Q52 | Weak collision resistance | **NONE CORRECT** — see clarification above |
| Q53 | Weak vs. strong collision | Weak = fixed input; strong = any two inputs |
| Q54 | Strong implies weak? | Yes — (c) |
| Q55 | Hash function structure | Merkle-Damgård construction |
| Q56 | Compression function | Processes one block, updates state |
| Q57 | IV role in hash | Initial fixed value (not secret) |
| Q58 | HMAC requirements | Secret key + hash function |
| Q59 | HMAC vs. signature | HMAC: symmetric keys; signature: asymmetric |
| Q60 | HMAC formula | Double-hash with ipad/opad |

---

*End of Lecture 05 Study Guide*  
*Next: Lecture 06 — Secure Random Number Generation (LCG, CSPRNG, BBS, Entropy Sources)*
