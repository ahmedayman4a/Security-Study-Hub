# CSE 464: Computer System Security
## Lecture 01 — Complete Study Guide
### Classical Ciphers, Block Ciphers & AES

> **Course:** CSE 464, Alexandria University — Prof. Dr. Sahar M. Ghanem  
> **Sources:** Lecture 01 slides (based on Prof. Kak's Lectures 2, 3 & 8) · Assignment 1 (Ch. 2–3) · Assignment 2 (Ch. 8–9) · 2025 Midterm Exam + Questions of Doubt

---

## How to Use This Guide

This guide is written for someone studying the material from scratch, not revising bullet points. Each section explains *why* something exists before explaining *how* it works. Exam questions, assignment questions, and practice problems are placed immediately after the concept they test — not collected at the end.

Read the explanation first, in full. Then attempt the practice questions from memory. Only then check the answers.

---

## Part 1: Core Vocabulary — The Language of Cryptography

### Why Vocabulary Matters

Before you can understand any cryptographic algorithm, you need to be precise about the words you use. Cryptography has a well-defined vocabulary, and confusing two terms in an exam answer — say, mixing up "key" and "algorithm," or "encryption" and "encipherment" — can cost you marks and, more importantly, will confuse your own thinking. So let us build the vocabulary carefully, starting from first principles.

### What Is the Problem We Are Trying to Solve?

Imagine Alice wants to send a private message to Bob over the internet. The internet is a public network — anyone monitoring the connection can potentially read the data passing through it. Alice and Bob need a way to transform the message so that an eavesdropper who intercepts it cannot make sense of it, while Bob himself can still recover the original message. This transformation is **encryption**.

The original message — the thing Alice wants to say — is called the **plaintext**. It doesn't have to be text in the English sense; it could be a video file, a bank transaction record, a photograph. "Plain" simply means *unprotected* and *readable*.

The result of encrypting the plaintext is called the **ciphertext**. An important detail here: in a block cipher (which we will study in detail shortly), the ciphertext is typically the *same size* as the plaintext. You might expect encryption to compress data or expand it dramatically, but in most modern ciphers the block size is preserved. The difference is that the ciphertext looks like complete nonsense to anyone who doesn't have the secret.

### The Algorithm and the Key

**Encryption** is the process of transforming plaintext into ciphertext. The **encryption algorithm** is the specific, deterministic sequence of computational steps that performs this transformation. You might think of the algorithm as a recipe — a procedure that anyone can in principle follow.

But here is something critically important: two people following the same recipe can produce entirely different outputs if they use different *ingredients*. In cryptography, that ingredient is the **secret key**. The key is a parameter (or set of parameters) that controls how the encryption algorithm behaves for a specific communication. Given the same plaintext, two different keys will produce two completely different ciphertexts. Without the key, even knowing the algorithm does not help you recover the plaintext.

**Decryption** is the reverse process — recovering the original plaintext from the ciphertext. The **decryption algorithm** takes the ciphertext and the key as inputs, and produces the plaintext as output. In symmetric cryptography (more on this shortly), the same key that was used for encryption is also used for decryption.

### Block Ciphers vs. Stream Ciphers

You will encounter two broad categories of symmetric encryption:

A **block cipher** divides the plaintext into fixed-size chunks called *blocks* and processes one complete block at a time, producing one complete ciphertext block of the same size. AES, for instance, works on 128-bit blocks — exactly 16 bytes at a time. If your plaintext is 1000 bytes, AES processes it in roughly 63 blocks (with some padding for the last one). The key insight about block ciphers is that the entire block is transformed as a unit; every bit of the output depends on every bit of the input within that block.

A **stream cipher**, by contrast, encrypts data continuously — typically one byte, or even one bit, at a time — as it flows through the cipher. A stream cipher generates a long sequence of pseudo-random bits called a **keystream** (derived from the key), and then XORs this keystream with the plaintext to produce ciphertext. RC4, which you will encounter in Assignment 2, is a stream cipher. Stream ciphers are stateful: the state of the cipher changes with every byte processed, so you can't easily process bytes out of order.

You might wonder: which is better, block or stream? It depends on context. Stream ciphers are naturally suited to real-time communications where data arrives byte by byte and you can't wait for a full block. Block ciphers are better for files and stored data. Modern protocols often use block ciphers in a special *mode* (like CTR mode, which you will study) that converts them into stream-like behavior.

### Cryptanalysis, Cryptography, and Cryptology

**Cryptography** is the broad field of designing encryption and decryption schemes. It is the "defense" side of the story.

**Cryptanalysis** is the art of *breaking* cryptographic systems — recovering the plaintext or the key without authorized access. The word literally means "analysis of hidden writing." You might wonder: is cryptanalysis only for attackers? Absolutely not. Security researchers perform cryptanalysis on new ciphers to find weaknesses *before* those ciphers are deployed. A cipher that has never been seriously attacked is a cipher that hasn't been seriously trusted. The process of designing a cipher, attacking it, fixing it, and attacking it again is how the field makes progress.

**Cryptology** is the parent discipline encompassing both cryptography and cryptanalysis.

### The Key Space

The **key space** is the total number of distinct keys that a cryptographic system can use. If a cipher uses a 56-bit key, every key is a 56-bit binary string, and there are 2⁵⁶ ≈ 7.2 × 10¹⁶ possible keys. If a cipher uses a 128-bit key, there are 2¹²⁸ ≈ 3.4 × 10³⁸ possible keys.

The key space size determines how resistant a cipher is to **brute-force attack** — trying every possible key. With 2⁵⁶ keys and a machine that can check one million keys per microsecond, you would need on average about 1,140 years to try half the key space. That seems secure until you realize that a parallel machine with a million cores reduces that to about one year — and eventually hardware improves. This is why 56-bit keys (DES) are now considered broken, and why AES uses at minimum 128-bit keys.

It is crucial to understand, though, that a large key space is *necessary but not sufficient* for security. As you will see with the monoalphabetic cipher, a system can have a key space of 26! ≈ 4 × 10²⁶ — vastly larger than DES's 2⁵⁶ — and yet be broken in minutes. Key space protects against brute-force; it does not protect against statistical or algebraic attacks.

---

### Quick Reference Table

| Term | Definition | Key Insight |
|------|-----------|-------------|
| Plaintext | Original, unencrypted message | Can be any digital data |
| Ciphertext | The encrypted output | Same size as plaintext in block ciphers |
| Encryption Algorithm | The computational procedure | Should be **public** |
| Secret Key | The parameter controlling the algorithm | Should be **secret** |
| Block Cipher | Processes fixed-size blocks | AES: 128-bit blocks |
| Stream Cipher | Encrypts byte-by-byte | RC4, using a keystream |
| Cryptanalysis | The art of breaking ciphers | Used by defenders too |
| Key Space | Total number of possible keys | 2ⁿ for an n-bit key |

---

### Knowledge Check 1

**Q:** A student says "I'm not going to tell you the encryption algorithm I use — that's what makes it secure." What is wrong with this?  
**A:** This is the "security through obscurity" fallacy. The algorithm should be public (Kerckhoffs's Principle). Security must rest entirely on the secrecy of the *key*, not the algorithm. If the algorithm is ever leaked or reverse-engineered, the entire system breaks. Publicly known algorithms are independently scrutinized by the global cryptographic community — which is actually how we gain *confidence* in them.

**Q:** A block cipher processes 128-bit blocks. If you encrypt a 500-byte file, how many complete blocks are produced?  
**A:** 500 bytes = 4000 bits. 4000 / 128 = 31.25 blocks, so 32 blocks are needed (the last block uses padding to fill the final incomplete block). The output will be 32 × 128 bits = 512 bytes.

---

### Exam Practice — Vocabulary
*(2025 Midterm Exam, Question 2)*

**Q2:** What is the key difference between stream ciphers and block ciphers?  
a) Stream ciphers encrypt data one bit or byte at a time, while block ciphers encrypt data in fixed-size blocks.  
b) Stream ciphers are generally faster than block ciphers.  
c) Block ciphers use a key stream generated from the key and an IV, while stream ciphers directly encrypt each bit with the key.  
d) Stream ciphers are more resistant to known-plaintext attacks than block ciphers.

**Answer: (a)**

*Why (a)?* This is the definitional distinction: block ciphers accumulate a full fixed-size block then process it; stream ciphers process data continuously. *Why not (b)?* Speed is context-dependent and not a defining structural difference. *Why not (c)?* This reverses the definitions — it is stream ciphers that generate and use a keystream, not block ciphers (though block ciphers *can be placed in stream-cipher-like modes*, which is a separate concept). *Why not (d)?* Resistance to known-plaintext attacks is not a defining or universal characteristic that separates the two categories.

---

## Part 2: Symmetric vs. Asymmetric Cryptography — and Why the Algorithm Must Be Public

### The Counterintuitive Principle

One of the first genuine "aha" moments in learning cryptography is this: **the encryption algorithm should be published openly and available to everyone — including your adversaries.** If you have never studied security before, this seems absurd. Surely the more secret your system, the more secure it is?

Let's think through why the opposite is true. Suppose you design an encryption algorithm and keep it completely secret. Who can test whether it is actually secure? Only you. But security is not something you can verify by yourself, because security is defined by an adversary's inability to break it — and adversaries are creative, persistent, and have different skills than you. History is full of examples of "secret" cryptographic systems that turned out to have catastrophic weaknesses their designers never imagined, weaknesses that were discovered only after they were deployed. The GSM mobile phone standard used a secret encryption algorithm that was reverse-engineered and found to be weak. Various proprietary video game encryption schemes, ATM PIN encryption systems, and military codes have all failed because they were never publicly scrutinized.

By contrast, when you publish your algorithm, every cryptographer in the world can attack it. Those attacks either expose weaknesses (which you then fix) or fail (which builds confidence). AES was selected through an open international competition in which teams from around the world submitted algorithms and attacked each other's submissions for years. The winner survived all those attacks. That public process is what makes us trust AES far more than any secret algorithm.

This principle — that **security must rest on the secrecy of the key, not the algorithm** — is known as **Kerckhoffs's Principle**, named after 19th-century cryptographer Auguste Kerckhoffs. It is the bedrock assumption of all modern cryptographic design. Every algorithm you study in this course (AES, RSA, SHA-256) assumes the algorithm is known to everyone; the only secret is the key.

### Symmetric Key Cryptography

In **symmetric key cryptography**, Alice and Bob share the same secret key. Alice uses it to encrypt; Bob uses the same key to decrypt. The algorithms are typically fast — AES can encrypt at multiple gigabytes per second on modern hardware. The challenge is purely logistical: how do Alice and Bob agree on a shared key in the first place? If they have no existing secure channel, they have a **key distribution problem**: how do you send a key securely over an insecure channel?

This was the defining unsolved problem of cryptography for thousands of years. Armies needed couriers to physically deliver codebooks. Banks needed secure telephone lines to set up keys. Every symmetric system assumed you had *already* somehow solved key distribution before you needed to use the cipher.

### Asymmetric (Public Key) Cryptography

In **asymmetric (public key) cryptography**, invented in the 1970s, each party has two mathematically linked keys: a **public key** (which can be published openly to anyone, posted on a website, put in a certificate) and a **private key** (kept absolutely secret, never shared with anyone). The mathematical relationship between them is the key: data encrypted with the public key can only be decrypted with the corresponding private key. But given only the public key, it is computationally infeasible to derive the private key.

This beautifully solves the key distribution problem: Bob publishes his public key openly. Alice encrypts her message with Bob's public key. Only Bob, with his private key, can decrypt it. An eavesdropper who intercepts both the ciphertext and the public key cannot decrypt, because they don't have the private key. Alice and Bob never needed to share a secret in advance.

The trade-off is performance. RSA (a widely used asymmetric cipher) is roughly 1000 times slower than AES. For this reason, real-world systems typically use asymmetric cryptography only to securely exchange a symmetric key, then switch to that symmetric key for bulk data encryption. This hybrid approach is exactly what TLS (the protocol securing HTTPS websites) does.

---

### Exam Practice — Kerckhoffs's Principle
*(2025 Midterm Exam, Question 3)*

**Q3:** What is the generally accepted consensus regarding the security of cryptographic algorithms?  
a) The security of a cryptographic system should primarily rely on the secrecy of the algorithm itself.  
b) A strong cryptographic system relies on the secrecy of the key, not the algorithm.  
c) Keeping the algorithm secret is an effective defense against all types of attacks.  
d) Open-source algorithms are inherently less secure than proprietary algorithms.

**Answer: (b)**

*Why (b)?* This is a direct statement of Kerckhoffs's Principle. *Why not (a) and (c)?* "Security through obscurity" is universally rejected by the cryptographic community because it depends on the algorithm never being reverse-engineered — an assumption that consistently fails in practice. *Why not (d)?* Exactly the opposite is true. Open-source algorithms like AES and SHA-256 are trusted precisely because they have been subjected to public scrutiny by thousands of experts worldwide for decades.

---

## Part 3: Cryptanalysis — The Art and Science of Breaking Ciphers

### Why You Need to Understand Attacks

You might be wondering why a course on *security* spends time teaching you how to *break* things. The answer is that every design decision in a modern cipher — every step of AES, every parameter of RSA — was made in direct response to a known attack. If you don't understand the attacks, the design choices look arbitrary. Once you understand the attacks, the design becomes obviously motivated.

Think of it like building a lock: you cannot design a good lock unless you understand how people pick locks. The history of cryptography is an arms race between cipher designers and cryptanalysts, with each side driving the other to greater sophistication.

### Brute-Force Attack

The simplest possible attack on any cipher is the **brute-force attack**: systematically try every possible key until you find the one that produces intelligible plaintext when used to decrypt the ciphertext. This requires no cleverness — just computation.

The feasibility of a brute-force attack depends entirely on the size of the key space. With a 56-bit key (DES), there are 2⁵⁶ ≈ 7.2 × 10¹⁶ possible keys. A machine checking one million keys per microsecond (10¹² per second) would take on average 0.5 × 2⁵⁶ / 10¹² ≈ 3.6 × 10⁴ seconds... wait, let me be more careful:

Average number of keys to try = 2⁵⁵ ≈ 3.6 × 10¹⁶  
At 10⁹ keys/second: 3.6 × 10¹⁶ / 10⁹ = 3.6 × 10⁷ seconds ≈ **416 days ≈ 13 months**

That sounds secure. But with a parallel machine of one million processors: 13 months / 10⁶ = about 37 seconds. With cloud computing, renting 10 million virtual machines for an hour costs a few thousand dollars. DES was cracked in 1998 by a dedicated machine for $250,000. Today it could be done much more cheaply.

This is why key size matters so much. With AES's 128-bit key, the key space is 2¹²⁸ ≈ 3.4 × 10³⁸. At 10¹⁸ keys per second (the combined speed of all computers on Earth, generously estimated), it would take 3.4 × 10²⁰ seconds ≈ 10¹³ years — thousands of times longer than the age of the universe. Brute-force is simply not a viable attack against AES.

An important insight for the exam: **brute-force has minimal memory requirements but potentially enormous time requirements.** You don't need to store anything; you just try key after key.

### Codebook Attack

A **codebook attack** takes a completely different approach. Instead of trying keys, the attacker tries to collect as many **known plaintext–ciphertext pairs** as possible and build a lookup table (the "codebook") that maps plaintexts directly to ciphertexts — or vice versa. Once you have a complete enough codebook, you can decrypt future messages simply by looking them up in the table, without ever knowing or finding the key.

You might wonder: how would an attacker get plaintext–ciphertext pairs? More often than you might think. A standard greeting at the start of every email ("Dear all,"), a fixed file header, a timestamp in a known format — these are all known plaintexts that appear in encrypted messages. If an attacker can identify which ciphertext block corresponds to which plaintext block, they get a pair. Wartime cryptanalysis was full of such "cribs" (known plaintext fragments).

The limitation of a codebook attack is memory. A complete codebook for a 64-bit block cipher would require 2⁶⁴ entries, each 64 bits long: that's 64 × 2⁶⁴ bits ≈ 10²¹ bits of storage, which is completely infeasible. For practical block sizes like 128 bits, it's even more absurd.

**In contrast to brute-force:** a codebook attack needs enormous memory but can produce instantaneous results once built. Brute-force needs minimal memory but enormous time. This is the **time–memory tradeoff** — a fundamental concept in cryptanalysis.

### Algebraic Attack

An **algebraic attack** is more sophisticated. The attacker attempts to express the mathematical relationship between the plaintext, the ciphertext, and the key as a system of equations, then solves that system to find the key. This works if the cipher has exploitable algebraic structure — particularly if some operation in the cipher is *linear*.

The Hill cipher, which you will study shortly, is a textbook example of a cipher that is trivially broken by algebraic attack. Because the Hill cipher is entirely based on linear matrix multiplication, an attacker with enough known plaintext–ciphertext pairs can write: K × P = C (mod 26), where P is a matrix of known plaintexts and C is the corresponding known ciphertexts. Solving for K is straightforward linear algebra — no exhaustive search required at all.

This is why AES is designed with deliberate, strong **non-linearity**. The SubBytes step (the AES S-box) is specifically constructed using the mathematics of finite fields to ensure that there is no simple algebraic relationship that an attacker can exploit. Every design decision in AES has a counterpart attack it is designed to prevent.

### The Time–Memory Tradeoff in Practice

Real attackers rarely operate at either extreme of the brute-force/codebook spectrum. They precompute *partial* codebooks — computing and storing as many pairs as their memory budget allows, then using those stored values to speed up their search. **Rainbow tables** (used to attack password hashes) are a famous real-world application of this tradeoff: instead of storing all possible passwords and their hashes (too much memory), or computing the hash for every guess during the attack (too much time), rainbow tables precompute chains of hashes that compress the storage while retaining most of the time savings.

Understanding this tradeoff helps you appreciate why salted password hashing exists, why AES uses a 128-bit key rather than a 64-bit key, and why many security decisions are ultimately about making the attacker's time-memory tradeoff as painful as possible.

---

### Worked Calculation — Key Space and Brute Force Time

**Given (from Assignment 1):** The EncryptForFun.py script, with BLOCKSIZE=16, has an effective key size of only 16 bits.

Key space = 2¹⁶ = 65,536 possible keys  
Average keys to try = 32,768  
At 10⁶ keys/second: 32,768 / 10⁶ ≈ **0.033 seconds**

This is trivially crackable in under a second on any modern computer. This is exactly why the programming part of Assignment 1 asks you to mount a brute-force attack: the 16-bit key space is small enough to search completely in your own Python script.

---

### Knowledge Check 3

**Q:** Why is a large key space necessary but not sufficient for security?  
**A:** Key space size determines resistance to brute-force attack only. A cipher can have a key space of 26! ≈ 4 × 10²⁶ (monoalphabetic cipher) and still be broken in minutes by frequency analysis, because the attack doesn't try all keys — it exploits statistical patterns. True security requires a large key space *and* resistance to all non-brute-force attacks (statistical, algebraic, etc.).

**Q:** What does the time–memory tradeoff mean, and why does it matter?  
**A:** It means that an attacker can trade off computation time against storage space. Precomputing a partial codebook costs memory upfront but speeds up future attacks. This matters because it means brute-force isn't the only threat: a well-resourced attacker might accept high memory costs in exchange for faster attack times. System designers must consider both dimensions.

---

### Exam Practice — Cryptanalysis
*(2025 Midterm Exam, Questions 1, 4, 9)*

**Q1:** Which of the following BEST describes the primary goal of cryptanalysis?  
a) To securely encrypt messages, preventing unauthorized access.  
b) To develop new cryptographic algorithms and protocols.  
c) To analyze and break cryptographic systems, revealing the underlying plaintext or key.  
d) To manage and distribute cryptographic keys securely.

**Answer: (c).** Cryptanalysis is specifically the discipline of breaking ciphers to recover plaintext or keys. The other options describe cryptographic design (b), encryption (a), and key management (d).

---

**Q4:** In a codebook attack, what does the attacker need to succeed?  
a) Only ciphertexts without any plaintext  
b) The encryption key  
c) Access to the encryption algorithm's source code  
d) A collection of known plaintext-ciphertext pairs

**Answer: (d).** A codebook attack is built entirely on accumulating known plaintext–ciphertext pairs. The attacker doesn't need the key (knowing the key would make it trivially simple but not a codebook attack) and doesn't need the source code (the algorithm is assumed public anyway per Kerckhoffs).

---

**Q9:** How does doubling the key length of a block cipher affect brute-force effort?  
a) Effort increases exponentially  
b) Effort remains the same  
c) Effort doubles  
d) Effort squares

**Answer: (a) — with the nuance clarified by "Questions of Doubt":**  
*From the clarification:* "The effort squares in a single key length doubling and increases exponentially in the long run." When you double the key length once (say, from n bits to 2n bits), the key space goes from 2ⁿ to 2²ⁿ, so the effort squares — that supports (d) for one doubling. But if you add bits incrementally (n → n+1 → n+2 → ...), the effort doubles with each added bit, which is exponential growth. The long-run behaviour is exponential, making (a) the more accurate general statement. The exam accepts (a) as the intended answer.

---

**Additional Practice:** Which of the following is NOT a cryptanalysis attack?  
a) Trying all 2¹²⁸ possible keys  
b) Collecting known plaintext–ciphertext pairs  
c) Expressing the cipher as a linear system and solving for the key  
d) Increasing the key length from 128 to 256 bits

**Answer: (d).** Increasing key length is a defensive design decision, not an attack strategy. Options (a), (b), (c) describe brute-force, codebook/known-plaintext, and algebraic attacks respectively.

---

## Part 4: Classical Encryption Techniques

### The Two Primitive Operations

Every encryption scheme ever devised — from ancient Egyptian hieroglyphs to AES — ultimately relies on only two fundamental operations: **substitution** and **transposition** (also called permutation).

**Substitution** means replacing each element of the plaintext with a different element. 'A' becomes 'Q'. Byte 0x6A becomes byte 0xF3. The element changes; its position in the message stays the same.

**Transposition** means rearranging the order of elements without changing what those elements are. 'HELLO' might become 'OELLH'. The letters are the same; their positions have changed.

Taken alone, either operation is easily broken. Substitution preserves frequency patterns (every 'E' is still replaced by the same letter, so the substitute appears just as often as 'E' does in English). Transposition preserves the actual letter frequencies completely (you're just moving the same letters around). But when you combine multiple rounds of substitution and transposition, their effects amplify each other in a way that becomes extremely hard to reverse without the key. This is exactly what AES does — ten rounds of mixed substitution and permutation steps.

Understanding the classical ciphers is not just historical trivia. Each one illustrates a different cryptographic idea, and each one fails in a way that directly motivated the next design. By studying them in sequence, you will see the design space of cryptography gradually fill in.

---

### 4.1 The Caesar Cipher

#### Building Intuition from Scratch

Julius Caesar, according to Suetonius, used a simple substitution cipher to protect military communications: he replaced each letter of the alphabet with the letter three positions further along. A became D, B became E, C became F, and so on. When you reached the end of the alphabet, you wrapped around: X became A, Y became B, Z became C.

Before we even write a formula, make sure the *physical intuition* is clear. Imagine writing out the alphabet in a circle. You have a "pointer" starting at some letter. To encrypt a letter, you move the pointer clockwise by 3 steps and write down where you land. To decrypt, you move the pointer counter-clockwise by 3 steps. The number 3 is the key.

Now here is the generalization: instead of always shifting by 3, let the shift amount *k* be any number from 0 to 25. This is the **general Caesar cipher** (sometimes called a shift cipher or rotation cipher). The value *k* is the secret key.

#### The Mathematics

To make this precise, we assign each letter an integer. A = 0, B = 1, C = 2, ..., Z = 25. Encryption and decryption then become:

**Encryption:** c = E(k, p) = (p + k) mod 26  
**Decryption:** p = D(k, c) = (c − k) mod 26

The **mod 26** handles the wraparound. When you go past Z (position 25), you wrap back to A (position 0). For example, if p = 24 (which is 'Y') and k = 3, then c = (24 + 3) mod 26 = 27 mod 26 = 1, which is 'B'. So Y with a shift of 3 becomes B. This is exactly the wrap-around behavior you expect.

For decryption, you might worry about negative numbers. If c = 1 (which is 'B') and k = 3, then p = (1 − 3) mod 26 = −2 mod 26. In modular arithmetic, −2 mod 26 = 24 (since 26 − 2 = 24). And 24 is 'Y'. Correct! Python handles negative modular arithmetic correctly; in other languages you may need to add 26 before taking mod.

#### Why the Caesar Cipher Fails Completely

The Caesar cipher has only **26 possible keys** (shifts of 0 through 25). This means even a human, working by hand, can try all possible decryptions in a few minutes. You don't need any sophisticated analysis — you simply try k = 0, k = 1, ..., k = 25 and look for the one that produces readable English. For k = 0, the ciphertext is the plaintext (no encryption). Clearly, with 26 possibilities, brute-force is trivial.

This is what makes the Caesar cipher historically interesting but practically useless: its key space is minuscule. The insight that matters here is not "the Caesar cipher is bad" but rather "key space alone does not determine security quality, but *insufficient* key space immediately makes a cipher useless."

---

#### Worked Example — Encrypting "are you ready" (from lecture slide)

Key k = 3.

| Letter | Numerical value | + 3 | mod 26 | Ciphertext letter |
|--------|----------------|-----|--------|------------------|
| a | 0 | 3 | 3 | **D** |
| r | 17 | 20 | 20 | **U** |
| e | 4 | 7 | 7 | **H** |
| (space) | — | — | — | (space) |
| y | 24 | 27 | 1 | **B** |
| o | 14 | 17 | 17 | **R** |
| u | 20 | 23 | 23 | **X** |
| (space) | — | — | — | (space) |
| r | 17 | 20 | 20 | **U** |
| e | 4 | 7 | 7 | **H** |
| a | 0 | 3 | 3 | **D** |
| d | 3 | 6 | 6 | **G** |
| y | 24 | 27 | 1 | **B** |

Result: **`DUH BRX UHDGB`** — exactly matches the lecture slide.

#### Worked Example — Decrypting "KHOOR" with k = 3

| Letter | Numerical value | − 3 | mod 26 | Plaintext |
|--------|----------------|-----|--------|----------|
| K | 10 | 7 | 7 | h |
| H | 7 | 4 | 4 | e |
| O | 14 | 11 | 11 | l |
| O | 14 | 11 | 11 | l |
| R | 17 | 14 | 14 | o |

Result: **`hello`**

---

#### Practice Problems — Caesar Cipher

**Easy 1.** Encrypt `HELLO` using a Caesar cipher with k = 13. What pattern do you notice?  
**Answer:** H(7+13=20→U), E(4+13=17→R), L(11+13=24→Y), L(11+13=24→Y), O(14+13=27 mod 26=1→B) → **URYYB**. The pattern: with k = 13, the cipher is its own inverse. Encrypting "URYYB" with k = 13 again gives back "HELLO". This special case is called **ROT-13** and is used in internet forums to hide spoilers — applying ROT-13 twice restores the original text.

**Easy 2.** Decrypt `MJQQT` with k = 5.  
**Answer:** M(12−5=7→H), J(9−5=4→E), Q(16−5=11→L), Q(16−5=11→L), T(19−5=14→O) → **HELLO**

**Medium 3.** You intercept the ciphertext `WKH TXLFN EURZQ IRA`. You know it was encrypted with a Caesar cipher but do not know the key. Describe your attack strategy and find the plaintext.  
**Answer:** Since there are only 26 possible keys, try each one. The first word `WKH` is 3 letters — try k = 3: W(22−3=19→T), K(10−3=7→H), H(7−3=4→E) → "THE". "THE" is a common English word — this is likely correct. Decrypting the rest with k = 3: TXLFN → QUICK, EURZQ → BROWN, IRA → FOX. **Plaintext: `THE QUICK BROWN FOX`**. Key = 3.

**Hard 4.** Prove mathematically that applying a Caesar cipher with key k₁ followed by a Caesar cipher with key k₂ is equivalent to a single Caesar cipher with key (k₁ + k₂) mod 26.  
**Answer:** First cipher: c = (p + k₁) mod 26. Second cipher: c' = (c + k₂) mod 26 = ((p + k₁) + k₂) mod 26 = (p + k₁ + k₂) mod 26 = (p + (k₁ + k₂) mod 26) mod 26. This is a single Caesar cipher with key (k₁ + k₂) mod 26. This shows that applying Caesar cipher twice offers no additional security — the combined effect is still a Caesar cipher with a single key.

---

### Exam Practice — Caesar Cipher *(From 2025 Midterm, related question)*

**MCQ:** If you encrypt a message with Caesar cipher k = 5, then encrypt the result again with Caesar cipher k = 8, what is the equivalent single key?  
a) 5   b) 8   c) 13   d) 40

**Answer: (c) 13.** As proven above, applying two Caesar ciphers with keys k₁ and k₂ is equivalent to a single Caesar cipher with key (k₁ + k₂) mod 26 = (5 + 8) mod 26 = 13.

---

### 4.2 The Monoalphabetic Substitution Cipher and Statistical Attacks

#### Making the Key Space Much Larger

The Caesar cipher's fatal flaw is its tiny key space: only 26 possible shifts. A natural response is to make the substitution completely arbitrary rather than constrained to a simple shift. Instead of saying "always shift by k," you define any permutation of the alphabet: A maps to some letter, B maps to some (different) letter, C maps to some other letter, and so on. The only rule is that the mapping must be one-to-one (otherwise decryption would be ambiguous).

This is the **monoalphabetic substitution cipher**. The key is the complete substitution table — a specific permutation of the 26 letters. For example:

Plain:  A B C D E F G H I J K L M N O P Q R S T U V W X Y Z  
Cipher: Q W E R T Y U I O P A S D F G H J K L Z X C V B N M

The key space is now the number of distinct permutations of 26 letters: **26!** = 403,291,461,126,605,635,584,000,000 ≈ **4 × 10²⁶**. This is astronomically larger than DES's 2⁵⁶ ≈ 7.2 × 10¹⁶. Even at one billion permutations tested per second, trying half of them would take more than the age of the universe. Brute-force is completely hopeless.

And yet. The monoalphabetic cipher can be broken by a human, by hand, with pencil and paper, in a matter of minutes.

#### The Fatal Flaw: Frequency Preservation

Here is the core insight that breaks the monoalphabetic cipher: **the cipher changes which symbol represents a letter, but it preserves how often that letter appears.**

In any sufficiently long piece of English text, the letter 'E' appears approximately 12.7% of the time. 'T' appears about 9.1% of the time. 'A' appears about 8.2%. The frequencies follow a characteristic distribution for English text (and a different but equally characteristic distribution for French, German, Spanish, etc.).

Now suppose 'E' is mapped to 'K' in the substitution table. Every 'E' in the plaintext becomes a 'K' in the ciphertext. Since 'E' appeared 12.7% of the time in the plaintext, 'K' now appears 12.7% of the time in the ciphertext. The frequency is preserved, just attached to a different symbol.

An attacker who intercepts a monoalphabetic ciphertext can do the following:
1. Count how often each letter appears in the ciphertext.
2. The most frequent ciphertext letter almost certainly represents 'E' in the plaintext.
3. The second most frequent probably represents 'T'.
4. And so on, working down the frequency ranking.
5. After guessing a few high-frequency letters, common words start to emerge (e.g., if you can guess which letters are T, H, and E, you look for the pattern _ H E and  T H _).
6. Each correctly identified letter constrains what the remaining letters can be, and the substitution table fills in rapidly.

This attack is called **frequency analysis** and was first described by Arab scholar Al-Kindi in the 9th century — over a thousand years ago. It works reliably on any monoalphabetic cipher, regardless of how complex the substitution table is, as long as the plaintext is natural language.

#### Digrams and Trigrams

Single-letter frequency analysis is just the beginning. In English, certain *pairs* of letters (called **digrams**) appear much more often than others. The most common English digrams include: TH, HE, IN, ER, AN, RE, ON, EN, AT, ND. Similarly, *triples* of letters (**trigrams**) like THE, AND, ING, ION, ENT, TIO are extremely common.

A monoalphabetic cipher preserves the frequencies of digrams and trigrams just as it preserves single-letter frequencies. So even if you can't immediately identify individual letters, looking for the most common ciphertext trigram and guessing it is "THE" gives you three letter identifications at once.

The combination of single-letter, digram, and trigram frequency analysis, plus pattern matching for common words, can break a monoalphabetic cipher from as few as 25 characters of ciphertext with high reliability.

The lesson is devastating: **a large key space does not guarantee security.** The cipher must also destroy the statistical structure of the plaintext — something a monoalphabetic cipher completely fails to do.

---

#### Practice — Frequency Analysis Reasoning

**Scenario:** You intercept the ciphertext `ZMX BMLZ YWTTYT ZMEBU WU ZMX XULDEYM DEBUKIKIX`. The message was encrypted with a monoalphabetic substitution cipher.

Count frequencies: Z=5, M=4, X=4, T=3, L=3, ...

The most frequent letter is Z. In English, 'E' is most frequent. Guess Z → E. M appears 4 times — guess M → T (second most common in English). X appears 4 times — guess X → H (third or fourth most common in English).

With Z→E, M→T, X→H: "EHT BTLY YWTTYH ETHU WU EHT THYDHYT DEBUKIKHE"... still garbled. Let's try X → A instead: "ZAT BLZT YATTYT ZMABU WU ZAT AULDEYA DEBUKIKAE". "ZAT" with Z→E → "EAT"? That's a word! Let's pursue Z→T instead (T is very frequent): "TMX BMLT YWTTYT TMEBU WU TMX XULDEYM DEBUKIKXE"...

*(This exercise shows the iterative, hypothesis-testing nature of frequency analysis. The full solution requires a few iterations.)*

---

### Knowledge Check 4.2

**Q:** True or False: The monoalphabetic cipher is unbreakable because its key space of 26! is too large to brute-force.  
**A:** False. While the key space is indeed too large for brute-force, the cipher is easily broken by frequency analysis. "Unbreakable by brute-force" and "unbreakable" are not the same thing. The cipher fails because it preserves the statistical structure (frequency distribution) of the plaintext, allowing an attacker to deduce the substitution table by analysing ciphertext letter frequencies.

---

### 4.3 The Playfair Cipher

#### The Motivation: One Step Beyond Monoalphabetic

The monoalphabetic cipher fails because individual letter frequencies are preserved. A natural response is to encrypt not individual letters but *pairs* of letters. If you encrypt the pair "TH" as a unit, then the frequency of "TH" in the plaintext no longer directly reveals the frequency of any single letter in the ciphertext. The attacker must now analyse *digram* frequencies in the ciphertext — a harder problem, because there are 26² = 676 possible digrams rather than just 26 letters.

This is the core idea behind the **Playfair cipher**, invented by Charles Wheatstone in 1854 but popularized by Baron Playfair (hence the name). It encrypts plaintext two letters at a time, using a 5×5 matrix as the key. The British Army used it in World War I and the Allied forces used it in World War II.

#### Step 1: Building the Key Matrix

Choose an encryption key — a word or phrase with no repeated letters. For example, the lecture slide uses **`smythework`**.

Now create a 5×5 matrix (25 cells for 25 distinct letters — I and J share one cell, since the Latin alphabet has 26 letters but we need exactly 25):

1. Write the key's letters left-to-right, top-to-bottom, skipping any duplicates.
2. Continue filling the matrix with remaining letters of the alphabet (in alphabetical order), skipping any already used.
3. I and J occupy the same cell (treat J as I during encryption).

For key `smythework` (letters: S, M, Y, T, H, E, W, O, R, K — all distinct, no duplicates):

```
 Col:  0   1   2   3   4
Row 0: S   M   Y   T   H
Row 1: E   W   O   R   K
Row 2: A   B   C   D   F
Row 3: G  I/J  L   N   P
Row 4: Q   U   V   X   Z
```

The remaining letters after the key (A, B, C, D, F, G, I/J, L, N, P, Q, U, V, X, Z — 15 letters/cells) fill rows 2–4. This matrix is the key to the entire cipher.

#### Step 2: Preprocessing the Plaintext

Before encrypting, the plaintext must be split into consecutive **pairs of letters**. But first, two preprocessing rules apply:

- **Repeated letters in a pair:** If a pair would consist of the same letter twice (like "LL"), insert a filler letter — typically 'X' — between them. So "LLAMA" becomes pairs (L, X), (L, A), (M, A). (If the filler 'X' causes a new repeated pair, use 'Q' or another letter instead.)
- **Odd-length message:** If the total number of letters (after inserting fillers) is odd, add a filler 'X' at the end.

#### Step 3: The Three Encryption Rules

For each pair (P1, P2), locate both letters in the 5×5 matrix and apply one of three rules:

**Rule 1 — Same Row:** If P1 and P2 are in the same row, replace each with the letter immediately to its *right* in that row, wrapping around from the last column back to the first.

**Rule 2 — Same Column:** If P1 and P2 are in the same column (but different rows), replace each with the letter immediately *below* it in that column, wrapping around from the last row back to the first.

**Rule 3 — Rectangle:** If P1 and P2 are neither in the same row nor the same column, they define the corners of a rectangle in the matrix. Replace each letter with the letter in the same row but the *column of the other letter*. In other words: P1 moves horizontally to the column where P2 lives, and P2 moves horizontally to the column where P1 lives.

Let's verify all three with the lecture slide examples:

**Example: `BF` → `CA` (Same Row)**  
B is at row 2, col 1. F is at row 2, col 4. They share row 2 → Rule 1.  
B shifts right: col 1 + 1 = col 2 → **C** (row 2, col 2).  
F shifts right: col 4 + 1 = col 5, wraps to col 0 → **A** (row 2, col 0). ✓

**Example: `OL` → `CV` (Same Column)**  
O is at row 1, col 2. L is at row 3, col 2. They share col 2 → Rule 2.  
O shifts down: row 1 + 1 = row 2, col 2 → **C** (row 2, col 2).  
L shifts down: row 3 + 1 = row 4, col 2 → **V** (row 4, col 2). ✓

**Example: `GF` → `PA` (Rectangle)**  
G is at row 3, col 0. F is at row 2, col 4. Different row and column → Rule 3 (rectangle).  
G takes the column of F (col 4), stays in its own row (row 3) → row 3, col 4 = **P**.  
F takes the column of G (col 0), stays in its own row (row 2) → row 2, col 0 = **A**. ✓

#### Why Playfair Is Better — and Why It Still Falls

By encrypting digrams rather than individual letters, the Playfair cipher substantially disrupts single-letter frequency analysis. The statistical fingerprint of individual letters is no longer directly visible in the ciphertext. To break Playfair, an attacker must analyse *digram* frequencies in the ciphertext — there are 676 possible digrams, far more than 26 letters, so more ciphertext is needed before the frequencies are reliable.

However, the cipher still falls to a skilled cryptanalyst. The reason: the Playfair cipher preserves *some* digram structure. The digram frequencies are altered but not destroyed — statistically, certain digrams are still more common than others in both the plaintext and ciphertext. With enough ciphertext (a few hundred characters is usually sufficient), an experienced analyst can apply digram frequency analysis to recover the key. The cipher was used in practice until analysts figured this out.

It is worth noting: Playfair was a genuine improvement over its predecessors. The message "this cipher is hard to break" sent to an unskilled codebreaker during wartime provided real operational security. Security has always been relative to the attacker's capabilities — a perfectly valid design consideration.

---

#### Worked Example — Encrypting "hello" with Playfair

Plaintext: `hello`  
Split into pairs: h-e, l-l, o-?  
`ll` is a repeated pair → insert filler X: h-e, l-x, l-o

**Pair HE:** H at (0, 4), E at (1, 0). Different row and column → Rectangle Rule.  
H stays in row 0, takes column of E (col 0): row 0, col 0 = **S**.  
E stays in row 1, takes column of H (col 4): row 1, col 4 = **K**.  
`HE` → **SK**

**Pair LX:** L at (3, 2), X at (4, 3). Different row and column → Rectangle Rule.  
L stays in row 3, takes column of X (col 3): row 3, col 3 = **N**.  
X stays in row 4, takes column of L (col 2): row 4, col 2 = **V**.  
`LX` → **NV**

**Pair LO:** L at (3, 2), O at (1, 2). Same column (col 2) → Shift Down.  
L (row 3) → row 4, col 2 = **V**.  
O (row 1) → row 2, col 2 = **C**.  
`LO` → **VC**

**Ciphertext: `SKNVVC`**

---

#### Practice Problems — Playfair

**Problem 1:** Encrypt the plaintext `MEET` using the `smythework` matrix.  
**Solution:**  
Pairs: M-E, E-T.  
ME: M at (0,1), E at (1,0). Rectangle. M takes col of E (0) in row 0 → S. E takes col of M (1) in row 1 → W. → **SW**  
ET: E at (1,0), T at (0,3). Rectangle. E takes col of T (3) in row 1 → R. T takes col of E (0) in row 0 → S. → **RS**  
**Ciphertext: SWRS**

**Problem 2 (Decryption):** Given ciphertext pair `CV` (using `smythework` matrix), find the plaintext.  
**Solution:** C at (2,2), V at (4,2). Same column → Same rule applies, but in reverse: shift *up* instead of down.  
C (row 2) → row above = row 1, col 2 = **O**.  
V (row 4) → row above = row 3, col 2 = **L**.  
**Plaintext: OL** ✓

**Problem 3 (Hard):** Explain why the pair `IE` would be treated the same as `JE` in the Playfair cipher, and what implication this has for messages containing the letter J.  
**Answer:** I and J share the same cell in the matrix (row 3, col 1). When encrypting, J is always treated as I. So `JE` and `IE` produce the same ciphertext. During decryption, you might recover `IE` where the original was `JE` — context must be used to determine which was intended. This is a known limitation of Playfair: the cipher is approximate for messages containing J.

---

### 4.4 The Hill Cipher

#### The Algebraic Leap

The Hill cipher, introduced by Lester Hill in 1929, takes a fundamentally different approach from Caesar and Playfair. Rather than applying clever but ad-hoc rules to letter pairs, Hill applies **formal linear algebra** — specifically, matrix multiplication over modular arithmetic.

To understand the motivation: Playfair encrypts pairs of letters. What if we encrypted *triples*? Or groups of ten? The more letters we mix together at once, the more each output letter depends on many input letters simultaneously, and the harder it becomes to extract frequency information from the ciphertext. The Hill cipher generalizes this idea to arbitrary group sizes using matrix multiplication.

#### How It Works

Assign integers 0–25 to letters A–Z as before. Choose a key matrix **K** — for example, a 3×3 matrix of integers. To encrypt three plaintext letters (represented as integers p₁, p₂, p₃), you compute:

```
[c₁]   [k₁₁  k₁₂  k₁₃] [p₁]
[c₂] = [k₂₁  k₂₂  k₂₃] [p₂]  (mod 26)
[c₃]   [k₃₁  k₃₂  k₃₃] [p₃]
```

Every ciphertext letter depends on all three plaintext letters. This is a significant improvement over substitution ciphers where each ciphertext letter depends on only one plaintext letter.

For decryption, you compute **K⁻¹** (the modular inverse of the matrix) and apply:

```
[p₁]        [c₁]
[p₂] = K⁻¹ [c₂]  (mod 26)
[p₃]        [c₃]
```

For the inverse to exist, the determinant of K must be coprime to 26 (i.e., gcd(det(K), 26) = 1). Otherwise decryption is impossible.

#### Extreme Strength Against Ciphertext-Only Attack

With a 3×3 matrix key, there are roughly 26⁹ ≈ 5.4 × 10¹² possible keys — far too many for brute-force. More importantly, the cipher mixes three letters simultaneously, making frequency analysis of individual ciphertext letters completely useless: each ciphertext letter is influenced by three plaintext letters at once.

#### Catastrophic Weakness Against Known-Plaintext Attack

Despite its strength against ciphertext-only analysis, the Hill cipher has a devastating flaw: it is **completely linear**. This makes it trivially breakable if the attacker knows even a small number of plaintext–ciphertext pairs.

Here is why: if you know that plaintext triplets P₁, P₂, P₃ encrypt to ciphertext triplets C₁, C₂, C₃, you can write the matrix equation **K × [P₁|P₂|P₃] = [C₁|C₂|C₃] (mod 26)**, and then solve for K by computing **K = [C₁|C₂|C₃] × [P₁|P₂|P₃]⁻¹ (mod 26)**. Just three known plaintext triplets are sufficient to completely recover the key matrix.

You might wonder: why is linearity so dangerous? Because linear equations are solvable by well-understood methods (Gaussian elimination). Once the attacker sets up the linear system, solving it is purely mechanical — there's no search involved at all.

This is why **non-linearity** is considered a fundamental requirement for modern ciphers. AES's SubBytes step is specifically designed to provide a strong non-linear component. The SubBytes transformation is computed using multiplicative inverses in a finite field — a deliberately non-linear operation that prevents the cipher from being expressed as any simple system of linear equations. Every time you study the AES S-box, remember: its complexity exists specifically to defend against the algebraic attack that destroys the Hill cipher.

---

### Knowledge Check 4.4

**Q:** You have a 3×3 Hill cipher key matrix K. How many known plaintext–ciphertext triplets do you need (at minimum) to uniquely recover K?  
**A:** Three triplets, because you are solving a 3×3 matrix equation K × P = C where P is a 3×3 matrix of plaintexts and C is a 3×3 matrix of ciphertexts. With three triplets, the matrix P is 3×3 and (if invertible mod 26) the equation has a unique solution for K.

**Q:** What property of the Hill cipher makes it vulnerable to known-plaintext attacks? Why does AES not have this vulnerability?  
**A:** The Hill cipher is entirely linear — all operations are matrix multiplication over integers mod 26. Given known pairs, the attacker can solve a linear system for the key. AES is not vulnerable because it includes SubBytes, a strongly non-linear operation constructed from GF(2⁸) multiplicative inverses, which ensures that no linear (or even low-degree algebraic) relationship exists between plaintexts, keys, and ciphertexts.

---

### 4.5 The Vigenère Cipher — The First Polyalphabetic Cipher

#### A Critical Concept: Monoalphabetic vs. Polyalphabetic

Step back and think about what all the ciphers we've studied so far have in common: Caesar, monoalphabetic, Playfair, Hill. In every case, there is a fixed, consistent relationship between plaintext symbols and ciphertext symbols. If 'E' maps to 'K' in one position, it maps to 'K' in every position. The substitution rule is the same throughout the entire message.

This is called a **monoalphabetic** cipher: "mono" = one, "alphabetic" = substitution alphabet. One substitution alphabet is used for the entire message.

The fundamental consequence of being monoalphabetic is that **frequency analysis always works**. No matter how complex the substitution table, no matter how many letters are mixed together at a time, if the same substitution rule is used everywhere, the frequencies of symbols are preserved from plaintext to ciphertext.

The Vigenère cipher breaks out of this constraint by using a **polyalphabetic** substitution: the substitution rule changes at every character position, controlled by a repeating key. Two identical plaintext letters in different positions will (usually) produce different ciphertext letters.

#### How the Vigenère Cipher Works

Choose a keyword — for example, `abracadabra`. Write it out, repeating it as needed to match the length of the plaintext. Each letter of the keyword defines a Caesar cipher for the corresponding position in the plaintext:

- Key letter 'a' means shift 0 (no shift, i.e., encrypt with Caesar k=0).
- Key letter 'b' means shift 1 (Caesar k=1).
- Key letter 'r' means shift 17 (Caesar k=17).
- And so on.

Formally:  
**Encryption:** cᵢ = (pᵢ + kᵢ) mod 26  
**Decryption:** pᵢ = (cᵢ − kᵢ) mod 26

where kᵢ is the integer value of the key letter at position i (repeating cyclically).

This means the Vigenère cipher is not one substitution alphabet but *as many substitution alphabets as the key length*. If the key has length N, then position 1 uses one substitution, position 2 uses a different substitution, ..., position N uses yet another substitution, then position N+1 cycles back to the same substitution as position 1, and so on.

#### Why This Defeats Frequency Analysis

Consider what happens to the letter 'E'. In English, 'E' is the most common letter, appearing ~12.7% of the time. In a monoalphabetic cipher, 'E' always maps to the same ciphertext letter, so that ciphertext letter appears ~12.7% of the time — easily detected.

In the Vigenère cipher with key `abracadabra` (length 11), the letter 'E' is encrypted differently depending on which position in the key it aligns with:
- If aligned with key letter 'a' (shift 0): 'E' → 'E'
- If aligned with key letter 'b' (shift 1): 'E' → 'F'
- If aligned with key letter 'r' (shift 17): 'E' → 'V'
- If aligned with key letter 'c' (shift 2): 'E' → 'G'
- etc.

Now the 12.7% frequency of 'E' in the plaintext is *distributed* across multiple different ciphertext letters. No single ciphertext letter accumulates the full frequency of 'E'. The frequency distribution of individual ciphertext letters is flattened — it looks more uniform, more random.

The longer the key, the better this masking effect. With a key of length N, the cipher consists of N independent monoalphabetic substitutions interleaved. An attacker who guesses N correctly can separate the ciphertext into N groups (positions 1, N+1, 2N+1, ... form group 1; positions 2, N+2, 2N+2, ... form group 2; etc.) and apply frequency analysis to each group independently. But they first need to determine N.

#### The Limit of Security: The One-Time Pad

You might wonder: what if the key were as long as the entire message and was never reused? With a key of length N equal to the plaintext length, there is no repetition — the cipher never cycles back to any previously used substitution. If additionally the key letters are chosen *uniformly at random*, then every plaintext letter is shifted by an independent random amount, and the resulting ciphertext is statistically **perfectly random**. No amount of ciphertext analysis can reveal *any* information about the plaintext, because for any possible plaintext P, there exists exactly one key K that would encrypt P into the observed ciphertext C.

This is the **one-time pad**, and it is the *only* cryptographic scheme proven to be **information-theoretically secure** — secure against an attacker with unlimited computational power. The Vigenère cipher with a random key as long as the message IS a one-time pad.

The exam question about this is direct: "What happens if the keyword in the Vigenère Cipher is as long as the plaintext and never reused?" The answer is that it becomes unbreakable — equivalent to a one-time pad.

The practical problem with the one-time pad is key management: the key must be as long as the message, it must be truly random, and it must never be reused. Sharing a one-time pad key securely requires a secure channel, which defeats the purpose if you already have a secure channel. These practical limitations mean the one-time pad is used in only very specific, high-security scenarios.

---

#### Worked Example — Vigenère Encryption (from lecture slide)

Key: `abracadabra...` (repeating)  
Plaintext: `canyoumeetmeatmidnightihavethegoods`

Let's compute the first 14 characters:

| Position | Plaintext | pt# | Key letter | key# | Sum | mod 26 | Ciphertext |
|----------|-----------|-----|-----------|------|-----|--------|-----------|
| 1 | c | 2 | a | 0 | 2 | 2 | **C** |
| 2 | a | 0 | b | 1 | 1 | 1 | **B** |
| 3 | n | 13 | r | 17 | 30 | 4 | **E** |
| 4 | y | 24 | a | 0 | 24 | 24 | **Y** |
| 5 | o | 14 | c | 2 | 16 | 16 | **Q** |
| 6 | u | 20 | a | 0 | 20 | 20 | **U** |
| 7 | m | 12 | d | 3 | 15 | 15 | **P** |
| 8 | e | 4 | a | 0 | 4 | 4 | **E** |
| 9 | e | 4 | b | 1 | 5 | 5 | **F** |
| 10 | t | 19 | r | 17 | 36 | 10 | **K** |
| 11 | m | 12 | a | 0 | 12 | 12 | **M** |
| 12 | e | 4 | a | 0 | 4 | 4 | **E** |
| 13 | a | 0 | b | 1 | 1 | 1 | **B** |
| 14 | t | 19 | r | 17 | 36 | 10 | **K** |

First 14 characters of ciphertext: **`CBEYQUPEFKMEBK`** ✓ (matches the lecture slide exactly)

Notice how the letter 'e' (plaintext positions 8, 9, 12) produces three different ciphertext letters: E, F, E — because the key letters at those positions are 'a', 'b', 'a'. This is the polyalphabetic effect in action.

---

#### Practice Problems — Vigenère

**Easy:** Decrypt `ZBHKM` with key `KEY`.  
Key in order: K(10), E(4), Y(24), K(10), E(4).  
Z(25)−10=15→P, B(1)−4=−3 mod 26=23→X, H(7)−24=−17 mod 26=9→J, K(10)−10=0→A, M(12)−4=8→I  
**Plaintext: PXJAI**

**Medium:** Encrypt `ATTACK` with key `SECRET`.  
Key: S(18), E(4), C(2), R(17), E(4), T(19)  
A(0)+18=18→S, T(19)+4=23→X, T(19)+2=21→V, A(0)+17=17→R, C(2)+4=6→G, K(10)+19=29 mod 26=3→D  
**Ciphertext: SXVRGD**

**Hard:** A Vigenère cipher uses a key of length 3. The ciphertext is `ZXZXZX`. What can you infer about the plaintext? (Hint: think about what must be true for every 3rd letter to encrypt identically.)  
**Answer:** With key length 3, positions 1,4 use key[0], positions 2,5 use key[1], positions 3,6 use key[2]. The ciphertext is Z at positions 1,3,5 and X at positions 2,4,6. This means plaintext positions 1 and 3 (and 5) are the same letter, and positions 2,4 (and 6) are the same letter. We can't determine the actual letters without knowing the key, but we know the plaintext has the pattern ABABAB where A and B are specific letters.

---

### Exam Practice — Vigenère
*(2025 Midterm Exam, Question 8)*

**Q8:** What happens if the keyword in the Vigenère Cipher is as long as the plaintext and never reused?  
a) It becomes unbreakable (like a one-time pad)  
b) It becomes equivalent to a Caesar cipher  
c) It turns into a transposition cipher  
d) It loses its polyalphabetic property

**Answer: (a).** When the key is as long as the plaintext and consists of random characters that are never reused, every position is encrypted with an independent random shift — this is precisely the one-time pad, the only provably unbreakable cipher. (b) is wrong: a Caesar cipher uses the same shift everywhere; this uses a unique shift everywhere. (c) is wrong: this is still a substitution cipher, not transposition. (d) is wrong: this *maximizes* polyalphabetic properties — every character uses a different substitution alphabet.

---

### 4.6 Transposition Techniques

#### What Transposition Does Differently

We have studied several substitution ciphers now — ciphers that change *what* letters appear. Transposition takes the opposite approach: it changes *where* letters appear, while leaving the letters themselves unchanged.

In a **transposition cipher** (also called a permutation cipher), the plaintext letters are rearranged in some order determined by the key. Every letter in the plaintext appears exactly once in the ciphertext — in a different position.

The simplest variant is the **column transposition cipher**:
1. Write the plaintext across the rows of a matrix of fixed width.
2. The key determines the order in which to read the columns.
3. Read down the columns in the key's order to produce the ciphertext.

#### Worked Example 4.6 — Column Transposition (Encryption)

Let's do a full example the way you would do it in an exam.

**Plaintext:** `MEETMEAFTERNOON`  
**Key order:** `3 1 4 2` (4 columns)

In this lecture's convention, the key digits are the **direct read sequence** of columns.

So for `3 1 4 2`, you read columns in this exact order: **column 3, then column 1, then column 4, then column 2**.

**Step 1 — Write plaintext row-wise in a 4-column grid**

```
Col#:   1   2   3   4
Key#:   3   1   4   2

        M   E   E   T
        M   E   A   F
        T   E   R   N
        O   O   N   X
```

We added filler `X` to complete the final row.

**Step 2 — Read columns in key order (3 → 1 → 4 → 2)**

- Column 3: `E A R N` → `EARN`
- Column 1: `M M T O` → `MMTO`
- Column 4: `T F N X` → `TFNX`
- Column 2: `E E E O` → `EEEO`

**Ciphertext:** `EARNMMTOTFNXEEEO`

You might notice this feels very mechanical compared to Vigenere. That is true. Transposition ciphers are algorithmically simple; their security comes only from position rearrangement, not symbol transformation.

#### Worked Example 4.6b — Column Transposition (Decryption)

Now reverse the process.

**Ciphertext:** `EARNMMTOTFNXEEEO`  
**Key order:** `3 1 4 2`

**Step 1 — Determine grid dimensions**

Ciphertext length = 16, columns = 4, so rows = 4.

**Step 2 — Fill columns in key order with ciphertext chunks**

Since rows = 4, each column gets 4 letters:

- Column 3 gets first 4 chars: `EARN`
- Column 1 gets next 4 chars: `MMTO`
- Column 4 gets next 4 chars: `TFNX`
- Column 2 gets last 4 chars: `EEEO`

So grid becomes:

```
Col#:   1   2   3   4
Key#:   3   1   4   2

        M   E   E   T
        M   E   A   F
        T   E   R   N
        O   O   N   X
```

**Step 3 — Read row-wise to recover plaintext**

`MEET` + `MEAF` + `TERN` + `OONX` = `MEETMEAFTERNOONX`  
Remove filler `X` → **`MEETMEAFTERNOON`**

---

#### Practice Problems — Transposition

**Easy:**  
1. Encrypt `ATTACKATDAWN` using 3 columns and key order `2 3 1`.  
   **Answer:** Length is 12, so no filler is needed (exactly 4 rows × 3 columns).  
   Grid (row-wise):
   `A T T / A C K / A T D / A W N`  
   Read columns in direct key order `2 → 3 → 1`:
   - Col 2: `T C T W` → `TCTW`
   - Col 3: `T K D N` → `TKDN`
   - Col 1: `A A A A` → `AAAA`  
   Ciphertext: **`TCTWTKDNAAAA`**.

**Medium:**  
2. Decrypt `TCTWTKDNAAAA` with the same key order `2 3 1`.  
   **Answer:** 12 chars, 3 columns → 4 rows, so each column gets 4 chars.  
   Fill by direct key sequence:
   - Col 2 = `TCTW`
   - Col 3 = `TKDN`
   - Col 1 = `AAAA`  
   Rebuild grid and read row-wise → **`ATTACKATDAWN`**.

**Hard (conceptual):**  
3. Why does column transposition resist direct substitution-frequency attacks better than Caesar, but still fail as a modern cipher?  
   **Answer:** It does not preserve letter positions, so simple substitution assumptions fail. But it preserves exact letter counts (all unigram frequencies), and enough ciphertext reveals permutation patterns. Without substitution/non-linearity, it lacks confusion and is breakable with modern cryptanalysis.

#### Why Transposition Alone Is Insufficient

Here is the critical weakness: transposition **preserves all individual letter frequencies**. If 'E' appears 12.7% of the time in the plaintext, it still appears 12.7% of the time in the ciphertext — just in different positions. A frequency attack on individual letters still reveals the language and provides useful constraints on the structure of the message. An analyst who observes that the ciphertext has English letter frequencies can be confident the letters haven't been substituted.

But that doesn't mean transposition is useless. Combined with substitution, transposition becomes powerful. Substitution destroys frequency information; transposition then scrambles position information. When you apply both — especially in multiple rounds — each operation amplifies the obscuring effect of the other. This is precisely the design principle of modern block ciphers: AES uses substitution (SubBytes) and permutation (ShiftRows, MixColumns) in ten consecutive rounds, and each round makes it harder to trace any relationship between input and output.

#### The Lesson

The takeaway from studying all these classical ciphers is this: security requires **both** confusion (obscuring the key–ciphertext relationship, achieved by substitution) and **diffusion** (spreading the influence of each plaintext bit across many ciphertext bits, achieved by transposition/permutation). These two properties, formalized by Claude Shannon in 1949, are the foundation of modern block cipher design. You will see them again in DES and AES.

---

## Part 5: The Ideal Block Cipher — Perfect Security, Impossible in Practice

### Thinking About What "Perfect" Means

Before we design a real block cipher, let's think about what the *ideal* block cipher would look like. This is not just an academic exercise — it defines the security target that practical ciphers like AES are trying to approximate.

Consider a block cipher that processes N-bit blocks. The cipher takes an N-bit input block and produces an N-bit output block, controlled by a key. The **ideal** block cipher would have this property: for each possible key, the relationship between input blocks and output blocks is a **completely random, independent permutation** of all possible N-bit strings. There would be no patterns, no correlations, no algebraic relationships — just pure randomness (subject only to the constraint that the mapping is invertible, so decryption works).

You might wonder: how would an attacker even approach such a cipher? They couldn't use frequency analysis (there's no statistical structure). They couldn't use algebraic attacks (there's no mathematical relationship). They couldn't use codebook attacks without having a complete codebook. The only option is brute-force over the key space.

### The Problem: The Key Is the Codebook

In an ideal block cipher, the encryption key is precisely the specification of which input maps to which output — i.e., the complete codebook. For a 64-bit block cipher, there are 2⁶⁴ possible input blocks. For each one, you need to specify a 64-bit output block. The codebook consists of 2⁶⁴ entries of 64 bits each:

**Codebook size = 64 bits × 2⁶⁴ entries ≈ 10²¹ bits ≈ 100 billion terabytes**

You cannot store this key. You cannot transmit it. You cannot compute it quickly. The ideal block cipher is *conceptually* perfect but *practically* completely infeasible. This is the fundamental tension in block cipher design: we want the behavior to look like an ideal block cipher (completely random-looking output), but we need to achieve this with a tiny key (128 bits for AES).

### The Goal for Real Ciphers

A real cipher like AES uses a 128-bit key and an algorithm (10 rounds of specific operations) to deterministically produce an output from any input. The output looks, to any computationally bounded observer, completely indistinguishable from truly random output — even though it is actually deterministically computed. This is what we mean by a "pseudorandom permutation" (PRP): it behaves like a random permutation to any efficient attacker, even though it isn't truly random.

The practical question is: does AES actually achieve this? The answer, based on decades of cryptanalysis by the world's best researchers, is yes — no one has found a computationally efficient way to distinguish AES-encrypted data from truly random data.

---

### Knowledge Check 5

**Q:** Why is the ideal block cipher impractical despite being the theoretically perfect design?  
**A:** Because the encryption key *is* the codebook — the complete table mapping every possible input block to its output. For a 64-bit block cipher, this requires 64 × 2⁶⁴ ≈ 10²¹ bits of storage, which is completely infeasible. AES solves this by using a compact 128-bit key together with a specific mathematical algorithm that produces output that *looks* like a random permutation.

---

## Part 6: The Feistel Structure — Elegance in Block Cipher Design

### The Engineering Problem

Imagine you are a cryptographer at IBM in 1971, tasked with designing a cipher for banking. You have a clear wish-list: strong non-linear substitution operations for security, fast execution, and — critically for hardware — the *same circuitry* running both encryption and decryption.

One natural choice for the substitution step is a lookup table — you feed in a fixed-width input, and the table gives you a scrambled output. These lookup tables are called **substitution boxes** or **S-boxes**. We will study them in detail when we reach AES, but for now think of an S-box as a fixed, pre-designed scramble table. The most security-effective S-boxes tend to be **non-invertible**: they deliberately map a larger input (e.g. 6 bits) to a smaller output (e.g. 4 bits), losing information on purpose to make the mapping harder to analyze. The problem: you cannot run such a table in reverse. How do you build a cipher that uses non-invertible operations but still decrypts correctly?

Horst Feistel's answer was one of the most elegant ideas in all of cryptography, and it is the foundation of DES.

### The Analogy: One-Way Masking

Think of it this way. Suppose Alice writes a secret message on the left page of a notebook. On the right page she writes some random-looking scribbles (a hash of the page number and a shared secret). She XORs the left page with those scribbles and sends both pages. Bob receives them. He can reproduce the *exact same scribbles* (same right page, same shared secret) and XOR again — because XOR is its own inverse. He recovers the left page without ever needing to "un-hash" anything.

The Feistel structure does exactly this, but with the two halves of a block, over many rounds.

### The Core Mechanism: Split, Mix, Swap

Split the N-bit block into two equal halves:

$$\text{Block} = (L_0 \;|\; R_0), \quad \text{each } N/2 \text{ bits}$$

For each round $i = 1, 2, \ldots, n$:

$$\boxed{L_i = R_{i-1}}$$

$$\boxed{R_i = L_{i-1} \oplus F(R_{i-1},\; K_i)}$$

where $F$ is the **Feistel round function** (an arbitrary, possibly non-invertible transformation) and $K_i$ is the round key derived from the master key.

In plain English:
- The right half **passes through unchanged** and becomes the new left half.
- The left half gets **XORed with a scrambled version of the right half**, and the result becomes the new right half.

$F$ can be anything — a cascade of substitution tables, bit permutations, modular arithmetic. It is not required to be invertible. This freedom is the whole point.

### Deriving Decryption: A Step-by-Step Proof

This is the beautiful part, and it is worth working through carefully. Given the outputs of round $i$ — that is, $(L_i,\; R_i)$ — and knowing the round key $K_i$, we want to recover the inputs $(L_{i-1},\; R_{i-1})$.

**Step 1 — Recover $R_{i-1}$:**

The first encryption equation says $L_i = R_{i-1}$. Trivially:

$$R_{i-1} = L_i$$

The old right half is just sitting there, untouched, as the current left half. No computation needed.

**Step 2 — Recover $L_{i-1}$:**

The second encryption equation is:

$$R_i = L_{i-1} \oplus F(R_{i-1},\; K_i)$$

We already know $R_{i-1} = L_i$, so substitute:

$$R_i = L_{i-1} \oplus F(L_i,\; K_i)$$

XOR both sides with $F(L_i,\; K_i)$. Since $x \oplus x = 0$ and $x \oplus 0 = x$:

$$L_{i-1} = R_i \oplus F(L_i,\; K_i)$$

**That is the entire decryption step.** We evaluated $F$ in the **same forward direction** as encryption — no inverse of $F$ is ever needed. The XOR self-cancellation does all the work.

> **Key Insight:** You only need $R_i$, $L_i$, and $K_i$ — all three available to the decryptor. $F$ is called with the same inputs in both directions; XOR cancels itself. Non-invertible substitution operations are perfectly fine.

### The Punchline: Same Hardware, Reversed Keys

Putting it all together, the decryption algorithm is **structurally identical** to the encryption algorithm. The only difference is that the round keys are applied in reverse order.

| Direction | Round key sequence |
|---|---|
| Encryption | $K_1,\; K_2,\; \ldots,\; K_{16}$ |
| Decryption | $K_{16},\; K_{15},\; \ldots,\; K_1$ |

DES has 16 rounds. The same 16-round circuit that encrypts when fed $K_1 \to K_{16}$ decrypts when fed $K_{16} \to K_1$. No separate decrypt chip. No separate inverse lookup tables. This was a major practical advantage in 1977 when DES was standardised, and the principle influenced cipher design for decades.

> **Coming up:** When we study AES later, you will see that AES takes a fundamentally different structural approach — every transformation must have an explicit mathematical inverse, and the decryption algorithm is genuinely different from the encryption algorithm. That contrast will make the elegance of Feistel's design even clearer. For now, the key takeaway is this: the Feistel structure's split-and-XOR construction gives you decryption for free, without requiring any operation in $F$ to be invertible.

---

### Knowledge Check 6

**Q:** In the Feistel structure, why does the right half pass through each round unchanged?  
**A:** The right half passes through unchanged because it needs to serve as the input to the Feistel function F during both encryption and decryption. By keeping it unchanged for the current round (and then swapping so it becomes the new left half), we ensure that during decryption we always know the inputs to F — without needing to invert F itself.

**Q:** Why is it an advantage that the Feistel function $F$ does not need to be invertible?  
**A:** Non-invertible functions can be cryptographically stronger. A substitution table that maps 6 bits to 4 bits (losing 2 bits of information) is much harder to reverse-engineer than one that maps 6 bits to 6 bits bijectively — the information loss creates ambiguity that helps resist analysis. The Feistel structure's XOR construction allows decryption without ever calling $F^{-1}$, so you can use the most aggressive, hard-to-analyze substitutions without any constraint of reversibility.

**Q:** In a Feistel cipher, what happens if the round function $F$ is accidentally designed to be invertible — does that break anything?  
**A:** No, nothing breaks. Invertibility of $F$ is simply not required by the Feistel construction. The decryption derivation only relies on XOR being self-inverse and on the right half passing through unchanged — it never calls $F^{-1}$. If $F$ happens to be invertible, decryption still works identically. The Feistel structure gives you decryption regardless of $F$'s properties.

---

### Exam Practice — Feistel Structure *(Related to 2025 Midterm)*

**Practice Q:** In a Feistel cipher with 3 rounds, you encrypt a block. During decryption, in what order are the three round keys applied?  
**Answer:** If encryption used keys K₁, K₂, K₃ (in that order), decryption applies them in the reverse order: K₃, K₂, K₁. This is the only difference between the encrypt and decrypt procedures.

**True/False:** In a Feistel cipher, the Feistel function F must be a one-to-one (invertible) function.  
**Answer: False.** This is precisely the beauty of the Feistel structure — $F$ does not need to be invertible. The XOR structure and the fact that the right half passes through unchanged allow decryption to work with any $F$. DES's round function uses substitution tables that map 6-bit inputs to 4-bit outputs — clearly non-invertible — yet DES decryption works perfectly, because $F$ is never inverted, only reapplied.

---

## Part 7: DES — Data Encryption Standard

### Historical Context

DES was adopted by NIST (then called the National Bureau of Standards) in 1977 as the United States standard for data encryption. It was the first public, standardized encryption algorithm — a major milestone. For nearly two decades, DES was used to secure banking transactions, government communications, and commercial data worldwide.

| Parameter | Value |
|-----------|-------|
| Block size | 64 bits |
| Key size | 56 bits |
| Number of rounds | 16 |
| Structure | Feistel network |
| Status | Retired — broken in 1998 |

### Confusion and Diffusion: Shannon's Two Properties

In 1949, Claude Shannon — the founder of information theory — published a paper identifying the two properties that make a cipher secure. Understanding these properties is essential for understanding why DES (and later AES) are designed the way they are.

**Diffusion** means that a change in any bit of the plaintext (or key) should affect as many bits as possible of the ciphertext. Ideally, flipping one plaintext bit should change, on average, half of all ciphertext bits. This destroys the statistical structure of the plaintext: even if the plaintext has patterns (like English text), those patterns should not survive into the ciphertext because they get "diffused" across the entire block. In DES, the permutation (transposition) operations achieve diffusion.

**Confusion** means that the relationship between the ciphertext and the key should be as complex as possible — each bit of the ciphertext should depend on many bits of the key in a complicated, non-linear way. This ensures that even if an attacker has many ciphertext samples, they cannot deduce the key because the relationship is too complex to untangle. In DES, the S-box (substitution) operations achieve confusion.

You might wonder why you need both. Consider a cipher with only diffusion (pure transposition): you can permute the bits, but the attacker still sees the same set of bits — just rearranged. The statistical properties of the plaintext survive. Now consider a cipher with only confusion (pure substitution): you change the bits, but the substitution is applied independently to small groups, so patterns can re-emerge. It is the *combination* — substitution that creates confusion, followed by permutation that spreads that confusion across the entire block — that produces the **avalanche effect**: changing one input bit causes approximately half of all output bits to change in an unpredictable way.

In DES, a single plaintext bit change affects an average of **31 bit positions** in the ciphertext after 16 rounds. This is the avalanche effect at work. AES, which we study next, achieves a much stronger avalanche — a single bit change affects all 128 output bit positions — thanks to a more effective combination of confusion and diffusion steps. Understanding why requires understanding AES's specific design, which we will cover in detail shortly.

### Why DES Was Retired

DES's 56-bit key was its critical weakness. When DES was designed in the 1970s, 2⁵⁶ ≈ 7.2 × 10¹⁶ keys seemed safely beyond reach. By the late 1990s, hardware had improved dramatically. In 1998, the Electronic Frontier Foundation built a dedicated machine called "Deep Crack" for $250,000 that could crack DES in under 56 hours. Today, cloud computing resources could do it far faster and for far less money.

NIST recognized this problem and in 1997 launched a competition to select a new standard. Five finalist algorithms were evaluated over three years, and in 2001, the Rijndael algorithm (designed by Belgian cryptographers Joan Daemen and Vincent Rijmen) was selected as the **Advanced Encryption Standard (AES)**.

---

### Exam Practice — DES
*(2025 Midterm Exam, Questions 6, 7)*

> **Note:** Questions on 2DES, 3DES, Meet-in-the-Middle, and Block Cipher Modes are covered in the **Lecture 02** study guide.

**Q6:** What is the main function of confusion in a block cipher?  
a) To shuffle bits within the block  
b) To expand the key into multiple subkeys  
c) To compress the plaintext before encryption  
d) To obscure the relationship between the key and ciphertext

**Answer: (d).** Confusion specifically means making the key–ciphertext relationship as complex and opaque as possible. (a) describes diffusion (shuffling/permuting bits). (b) describes the key schedule. (c) is not a goal of block ciphers.

---

**Q7:** Why do modern block ciphers use multiple rounds?  
a) To increase key size  
b) To enhance confusion and diffusion  
c) To allow variable block sizes  
d) To reduce encryption speed for security

**Answer: (b).** Each individual round provides limited confusion and diffusion — after one substitution step, most bits are still quite predictable from the plaintext. Multiple rounds stack the effect: the output of round 1 becomes the input to round 2, and any residual patterns from round 1 are scrambled further. By round 10 of AES, a single-bit change has diffused to affect all 128 output bits. (a) is false — rounds don't change the key size. (c) is false — block size is fixed regardless of round count. (d) is false — speed is a cost of rounds, not a goal.

---

## Part 8: AES — The Advanced Encryption Standard

### 8.1 Why AES Was Needed

DES failed primarily because its 56-bit key was too short — by 1998, dedicated hardware could brute-force it in under 56 hours. But NIST did not just want a cipher with a longer key. It wanted a fundamentally better design: more mixing per round, operations that are fast in software (not just hardware), and a larger block size. In 1997, NIST launched an open international competition. Teams worldwide submitted algorithms, publicly attacked each other's designs for three years, and in 2001 the winner — **Rijndael**, designed by Belgian cryptographers Joan Daemen and Vincent Rijmen — became the **Advanced Encryption Standard (AES)**.

| Parameter | AES | DES |
|-----------|-----|-----|
| Block size | 128 bits | 64 bits |
| Key sizes | 128, 192, or 256 bits | 56 bits |
| Rounds (128-bit key) | 10 | 16 |
| Structure | Substitution-Permutation Network | Feistel Network |
| Operations | Byte-level | Bit-level |
| Decryption | Separate inverse algorithm | Same algorithm, reversed keys |

Three reasons AES was adopted over DES (exam Question 10 — answer is **(d) all of the above**):
- **Larger block and key sizes:** more security margin
- **Faster in software:** AES is byte-oriented; modern CPUs work naturally with bytes, while DES's 6→4-bit S-boxes require awkward bit-level manipulation
- **Public competition:** years of global cryptanalysis scrutiny before standardization

---

### 8.2 The Big Picture — Four Problems, Four Operations

Before studying each operation in detail, it helps enormously to understand what problem each one solves. AES uses a **Substitution-Permutation Network (SPN)**: every round transforms the entire block through the same four operations in sequence. Unlike a Feistel cipher (which leaves half the block unchanged per round), an SPN transforms every byte every round — which is why AES achieves stronger security in fewer rounds.

Here is the design logic, stated plainly:

**Problem 1: Without non-linear operations, the cipher is algebraically breakable.**  
If all operations are linear, an attacker with enough plaintext–ciphertext pairs can set up a system of linear equations and solve for the key — just like the Hill cipher attack. Solution: **SubBytes** injects non-linearity by applying a carefully constructed lookup table (the S-box) to each byte.

**Problem 2: After SubBytes, each byte has been scrambled independently — but there is no interaction between bytes in different columns.**  
If bytes never interact across columns, an attacker can attack each column independently. Solution: **ShiftRows** rotates rows by different amounts, so that when the next operation (MixColumns) processes each column, it is now mixing bytes that came from different original columns.

**Problem 3: After ShiftRows, bytes have been rearranged but not actually mixed — a column still contains independent values.**  
Solution: **MixColumns** performs matrix multiplication over a finite field, mixing all four bytes of each column together so that every output byte depends on all four input bytes.

**Problem 4: All three operations above are public. Without the key, anyone can decrypt.**  
Solution: **AddRoundKey** XORs the state with a key-derived value at each round. Without knowing the key, the cipher is uncomputable.

Each round applies them in this order: SubBytes → ShiftRows → MixColumns → AddRoundKey. The order matters — SubBytes provides confusion per byte, then ShiftRows and MixColumns together spread that confusion across the full block, and finally AddRoundKey ties everything to the secret key.

---

### 8.3 The State Array

When AES receives a 128-bit input block, it organizes the 16 bytes into a **4×4 state array**, filled **column-major** (down column 0, then down column 1, etc.):

```
Input bytes:  b₀  b₁  b₂  b₃  b₄  b₅  b₆  b₇  b₈  b₉  b₁₀ b₁₁ b₁₂ b₁₃ b₁₄ b₁₅

State array:
         Col 0    Col 1    Col 2    Col 3
Row 0:    b₀       b₄       b₈      b₁₂
Row 1:    b₁       b₅       b₉      b₁₃
Row 2:    b₂       b₆      b₁₀      b₁₄
Row 3:    b₃       b₇      b₁₁      b₁₅
```

Why this shape? Because ShiftRows operates on rows and MixColumns operates on columns — a 4×4 matrix makes both operations clean and efficient. Each round reads the current 4×4 state, transforms it, and passes the result to the next round. The final state is converted back to 128 bits by reading in the same column-major order.

---

### 8.4 GF(2⁸) Arithmetic — The Shared Mathematical Foundation

Before studying SubBytes and MixColumns in detail, you need a working understanding of **GF(2⁸)** — the mathematical framework both operations rely on. It sounds intimidating, but the practical rules are simple.

#### What GF(2⁸) Is

GF(2⁸) is a mathematical field with exactly 256 elements — one for each possible byte value (0x00 through 0xFF). A "field" means that addition and multiplication are both defined, and every non-zero element has a multiplicative inverse. The special property of GF(2⁸) is that all arithmetic is done with polynomials whose coefficients are just 0 or 1 (mod 2), which maps perfectly onto bits.

The field is constructed modulo an **irreducible polynomial**: $x^8 + x^4 + x^3 + x + 1$ (hex $0\text{x}11\text{b}$). Think of this as the "modulus" for multiplication — like how clock arithmetic works mod 12. When a multiplication result exceeds 8 bits, you reduce it by this polynomial.

You will never need to factor or reason about the polynomial in this course. What you need are the three practical arithmetic rules below.

#### Rule 1: Addition = XOR

In GF(2⁸), adding two bytes means XORing them bit-by-bit. There is no carry.

$$a \oplus b = c \quad \text{(ordinary bitwise XOR)}$$

Example: $0\text{x}53 \oplus 0\text{xCA} = 0\text{x}99$

This makes sense because in polynomial terms, adding coefficients mod 2 means $0+0=0$, $1+0=1$, $1+1=0$ — exactly XOR.

#### Rule 2: Multiplication by 2 (xtime)

Multiplying a byte $b$ by 2 in GF(2⁸):

1. **Left-shift $b$ by one bit position** (this is multiplying the polynomial by $x$).
2. **If bit 7 of the original $b$ was 1** (meaning the shift would produce a 9-bit result), **XOR the result with $0\text{x}1\text{b}$**. This XOR with $0\text{x}1\text{b}$ is the reduction step — it subtracts (mod 2 addition) the irreducible polynomial's lower 8 bits ($x^4 + x^3 + x + 1 = 0\text{x}1\text{b}$) to bring the result back within 8 bits.
3. If bit 7 was 0, no adjustment needed.

$$\text{xtime}(b) = \begin{cases} b \ll 1 & \text{if bit 7 of } b = 0 \\ (b \ll 1) \oplus 0\text{x}1\text{b} & \text{if bit 7 of } b = 1 \end{cases}$$

**Example:** $\text{xtime}(0\text{x}57)$:
- $0\text{x}57 = 0101\;0111_2$. Bit 7 is 0.
- Left-shift: $1010\;1110_2 = 0\text{xAE}$.
- No adjustment needed. Result: $0\text{xAE}$.

**Example:** $\text{xtime}(0\text{xAE})$:
- $0\text{xAE} = 1010\;1110_2$. Bit 7 is 1.
- Left-shift: $\text{0101\;1100}_2 = 0\text{x5C}$ (low 8 bits after dropping the carry).
- XOR with $0\text{x}1\text{b}$: $0\text{x5C} \oplus 0\text{x}1\text{b} = 0\text{x}47$.
- Result: $0\text{x}47$.

#### Rule 3: Multiplication by 3

$3b = 2b \oplus b$, i.e., apply xtime once, then XOR with the original.

$$3 \times b = \text{xtime}(b) \oplus b$$

**Example:** $3 \times 0\text{x}57 = \text{xtime}(0\text{x}57) \oplus 0\text{x}57 = 0\text{xAE} \oplus 0\text{x}57 = 0\text{xF9}$.

These three rules (addition=XOR, ×2=xtime, ×3=xtime⊕original) are all you need to evaluate MixColumns by hand. Higher multiplications (×9, ×11, ×13, ×14 used in InvMixColumns) are computed by repeated application of xtime.

---

### 8.5 SubBytes — Confusion Through Non-Linear Substitution

#### Why SubBytes Is the Only Non-Linear Step

Every other AES operation is linear over GF(2⁸): ShiftRows permutes bytes (linear), MixColumns is matrix multiplication (linear), AddRoundKey is XOR (linear). A cipher built entirely from linear operations is vulnerable to linear algebra: given enough plaintext–ciphertext pairs, an attacker sets up and solves a linear system to recover the key — exactly what breaks the Hill cipher. SubBytes is the **sole non-linear operation** in AES, and it is placed first in each round specifically so its non-linearity propagates through ShiftRows, MixColumns, and AddRoundKey in that round.

#### The S-box Lookup

SubBytes replaces each byte of the 4×4 state with its value from the **16×16 AES S-box**:

1. The byte's upper 4 bits (high nibble) give the row index (0–15).
2. The byte's lower 4 bits (low nibble) give the column index (0–15).
3. Look up the S-box at (row, column) and replace the byte.

**Example:** byte $0\text{x}95$ → high nibble 9, low nibble 5 → look up S-box[9][5].

The entire S-box is pre-computed and stored as a 256-entry lookup table. In hardware and software, SubBytes is simply 16 parallel table lookups — very fast.

#### S-box Construction — Why These Specific Values?

The S-box is not random. It is constructed by a two-step process chosen for provable mathematical security properties.

**Step 1 — Multiplicative Inverse in GF(2⁸):**

For each byte $b$ from $0\text{x}01$ to $0\text{xFF}$, find its multiplicative inverse $b^{-1}$ in GF(2⁸), meaning the byte $b^{-1}$ such that $b \times b^{-1} = 1$ (where $\times$ is GF(2⁸) multiplication). The byte $0\text{x}00$ has no inverse (analogous to $0$ in ordinary arithmetic), so it is mapped to $0\text{x}00$ by convention.

Why the multiplicative inverse? Because this mapping is **maximally non-linear**: no polynomial of degree less than 8 over GF(2) approximates it accurately. It is the hardest-to-approximate function available in GF(2⁸), which is exactly what you want for the non-linear substitution step.

**Step 2 — Affine Bit Transformation:**

After taking the multiplicative inverse, each byte undergoes an affine transformation. For each bit $b_i$ ($i = 0 \ldots 7$) of the inverse byte:

$$b'_i \;=\; b_i \;\oplus\; b_{(i+4)\bmod 8} \;\oplus\; b_{(i+5)\bmod 8} \;\oplus\; b_{(i+6)\bmod 8} \;\oplus\; b_{(i+7)\bmod 8} \;\oplus\; c_i$$

where $c = 0\text{x}63 = 01100011_2$, and $c_i$ is the $i$-th bit of $c$.

Why this affine transformation? Two reasons:
- **Eliminates fixed points:** Without the constant $c$, the byte $0\text{x}00$ (whose inverse is itself $0\text{x}00$) would pass through unchanged. A fixed point means a plaintext byte survives SubBytes visibly — a serious weakness. The constant $0\text{x}63$ ensures $0\text{x}00 \mapsto 0\text{x}63$, and in fact the full combined mapping has no fixed points or anti-fixed points at all.
- **Defeats correlation attacks:** The XOR pattern of four adjacent bits was chosen so that the complete S-box construction (inverse + affine) has provably low correlation between input and output bits, maximizing resistance to linear and differential cryptanalysis.

**Summary of S-box construction:**
1. Initialize: cell $(r,c)$ contains byte $\text{0xrc}$
2. Replace each non-zero byte with its multiplicative inverse in GF(2⁸) using $x^8+x^4+x^3+x+1$. ($0\text{x}00 \to 0\text{x}00$)
3. Apply affine transformation with constant $0\text{x}63$

**InvSubBytes** (decryption) reverses this: first apply the inverse affine transformation (constant $0\text{x}05$), then take the multiplicative inverse in GF(2⁸).

---

#### Assignment Exercise — S-box Construction
*(Assignment 2, Question 1)*

**Q:** What are the steps that go into constructing the 16×16 S-box?

**Step 1 — Initialize:** Create a 16×16 table. Cell $(r, c)$ holds byte $\text{0xrc}$ (e.g., cell $(9,5)$ holds $0\text{x}95$).

**Step 2 — Multiplicative inverse:** For each non-zero byte $b$, replace with $b^{-1}$ in GF(2⁸) using irreducible polynomial $x^8+x^4+x^3+x+1$. $0\text{x}00$ stays $0\text{x}00$.

**Step 3 — Affine transform:** Apply $b'_i = b_i \oplus b_{(i+4)\bmod 8} \oplus b_{(i+5)\bmod 8} \oplus b_{(i+6)\bmod 8} \oplus b_{(i+7)\bmod 8} \oplus c_i$ with $c = 0\text{x}63$. Eliminates fixed points, adds constant non-linearity.

---

### Exam Practice — SubBytes
*(2025 Midterm Exam, Question 12)*

**Q12:** How is the AES S-box constructed?  
a) By rotating bits in each byte  
b) By shuffling bytes randomly  
c) Through a lookup table provided by NIST  
d) Using a mathematical formula involving multiplicative inverses in GF(2⁸)

**Answer: (d).** The S-box is mathematically derived — not random. The two-step construction (GF(2⁸) multiplicative inverse + affine transform with $0\text{x}63$) gives it provable resistance to algebraic and differential attacks.

**Q:** Why is $0\text{x}00$ a special case in SubBytes?  
**A:** Zero has no multiplicative inverse in GF(2⁸) (same reason $0$ has no reciprocal in ordinary arithmetic). AES defines $0\text{x}00 \to 0\text{x}00$ for the inverse step. Then the affine transform with constant $0\text{x}63$ ensures $0\text{x}00$ does not become a fixed point — it maps to $0\text{x}63$.

**Q:** What would happen if the affine constant were $0\text{x}00$ instead of $0\text{x}63$?  
**A:** With constant zero, the affine transform is purely linear. The byte $0\text{x}00$ (whose inverse is $0\text{x}00$) passes through unchanged — a fixed point. An attacker could detect $0\text{x}00$ plaintext bytes in ciphertext, a meaningful information leak.

---

### 8.6 ShiftRows — Breaking Column Independence

#### The Problem ShiftRows Solves

Imagine for a moment that AES skipped ShiftRows entirely: SubBytes → MixColumns → AddRoundKey. SubBytes substitutes each byte independently, so it creates no interaction between bytes in different columns. MixColumns then mixes bytes within each column, but only within that column. The four columns of the state would be **completely independent** throughout all 10 rounds — an attacker could attack each of the four columns separately, reducing the problem from a 128-bit cipher to four independent 32-bit problems.

ShiftRows breaks this independence by *offset-shifting the rows* before MixColumns runs, so that when MixColumns processes each column, it finds bytes from four different original columns in that column.

#### How ShiftRows Works

ShiftRows performs a **cyclic left shift** on each row:

- **Row 0:** no shift (stays the same)
- **Row 1:** shift left by **1** byte (first byte wraps to the end)
- **Row 2:** shift left by **2** bytes
- **Row 3:** shift left by **3** bytes

```
Before ShiftRows:           After ShiftRows:
s₀₀  s₀₁  s₀₂  s₀₃        s₀₀  s₀₁  s₀₂  s₀₃   ← row 0: no shift
s₁₀  s₁₁  s₁₂  s₁₃   →    s₁₁  s₁₂  s₁₃  s₁₀   ← row 1: shift left 1
s₂₀  s₂₁  s₂₂  s₂₃        s₂₂  s₂₃  s₂₀  s₂₁   ← row 2: shift left 2
s₃₀  s₃₁  s₃₂  s₃₃        s₃₃  s₃₀  s₃₁  s₃₂   ← row 3: shift left 3
```

After ShiftRows, every new column contains exactly one byte from each of the four original columns. For instance, new column 0 contains $\{s_{00}, s_{11}, s_{22}, s_{33}\}$ — one byte from original column 0, 1, 2, and 3 respectively. When MixColumns now processes column 0, it mixes together bytes that came from all four original columns — achieving global diffusion.

**InvShiftRows** (decryption) is simply the reverse: cyclic **right** shifts of 0, 1, 2, and 3 bytes per row.

#### The Two-Round Avalanche

ShiftRows alone only rearranges bytes — no information is gained or lost. MixColumns alone mixes within one column but columns are still independent after a single application. Together, across two rounds:

- **Round 1:** One changed byte → MixColumns spreads the change to all 4 bytes of that column → ShiftRows distributes those 4 bytes into 4 different columns.
- **Round 2:** MixColumns processes each of those 4 affected columns → $4 \times 4 = 16$ bytes affected.

After just two rounds, a single changed input byte affects all 16 bytes of the state. After 10 rounds, every output bit depends on every input bit — the full AES avalanche effect.

---

### 8.7 MixColumns — Column-Level Diffusion

#### What MixColumns Does

MixColumns processes each column of the 4×4 state **independently**. It treats the 4 bytes of a column as a vector and multiplies it by a fixed $4 \times 4$ matrix — all arithmetic in GF(2⁸) (so addition = XOR, and the xtime rules from Section 8.4 apply).

The result: every output byte in a column depends on all four input bytes of that column. If you change even a single input byte, all four output bytes change — this is the **MDS (Maximum Distance Separable) property** of the chosen matrix.

#### The MixColumns Matrix

$$\begin{bmatrix} s'_0 \\ s'_1 \\ s'_2 \\ s'_3 \end{bmatrix} = \begin{bmatrix} 2 & 3 & 1 & 1 \\ 1 & 2 & 3 & 1 \\ 1 & 1 & 2 & 3 \\ 3 & 1 & 1 & 2 \end{bmatrix} \begin{bmatrix} s_0 \\ s_1 \\ s_2 \\ s_3 \end{bmatrix}$$

Expanded (where $\times$ is GF(2⁸) multiplication = xtime for ×2, xtime⊕original for ×3, and ×1 = identity):

$$s'_0 = (2 \times s_0) \oplus (3 \times s_1) \oplus s_2 \oplus s_3$$
$$s'_1 = s_0 \oplus (2 \times s_1) \oplus (3 \times s_2) \oplus s_3$$
$$s'_2 = s_0 \oplus s_1 \oplus (2 \times s_2) \oplus (3 \times s_3)$$
$$s'_3 = (3 \times s_0) \oplus s_1 \oplus s_2 \oplus (2 \times s_3)$$

#### Why Coefficients 1, 2, and 3?

You might wonder: why these specific coefficients and not others? Two reasons:
1. **Simplicity:** coefficients 1, 2, and 3 in GF(2⁸) are the cheapest to compute — ×1 is free, ×2 is one shift + one conditional XOR, ×3 is ×2 then XOR. Using larger coefficients (like 7 or 11) would require more computation.
2. **MDS property:** the specific $\{1,2,3\}$ pattern gives the matrix the MDS property — changing any single input byte changes all four output bytes. This is the strongest possible diffusion guarantee for a $4\times 4$ matrix with those coefficients.

#### InvMixColumns

Decryption uses the inverse matrix with coefficients $\{14, 11, 13, 9\}$:

$$\begin{bmatrix} s_0 \\ s_1 \\ s_2 \\ s_3 \end{bmatrix} = \begin{bmatrix} 14 & 11 & 13 & 9 \\ 9 & 14 & 11 & 13 \\ 13 & 9 & 14 & 11 \\ 11 & 13 & 9 & 14 \end{bmatrix} \begin{bmatrix} s'_0 \\ s'_1 \\ s'_2 \\ s'_3 \end{bmatrix}$$

These larger coefficients (14, 11, 13, 9) require chains of xtime applications, making InvMixColumns computationally more expensive than forward MixColumns — but identical in security.

---

### Exam Practice — MixColumns
*(2025 Midterm Exam, Question 11)*

**Q11:** What is the purpose of the MixColumns step in AES?  
a) To substitute bytes using a lookup table  
b) To shift rows cyclically  
c) To diffuse data across columns via matrix multiplication  
d) To XOR the round key

**Answer: (c).** (a) = SubBytes. (b) = ShiftRows. (d) = AddRoundKey. MixColumns provides column-level diffusion through GF(2⁸) matrix multiplication.

---

### 8.8 AddRoundKey — Introducing the Secret Key

#### Why It Exists

SubBytes, ShiftRows, and MixColumns are all deterministic, public operations — anyone who knows the AES algorithm can run them. Without AddRoundKey, AES would produce the same output for every user with the same input, making it trivially breakable. AddRoundKey is where the secret key enters every round.

#### What It Does

AddRoundKey XORs the entire 4×4 state with the current round's 4×4 key matrix (4 words = 16 bytes from the key schedule):

$$\text{state}[r][c] = \text{state}[r][c] \oplus \text{roundkey}[r][c]$$

It is the simplest of the four operations — a straightforward XOR — but conceptually the most important: it is the only step that depends on the secret.

#### Why XOR?

XOR is self-inverse: $A \oplus K \oplus K = A$. This means the decryption step for AddRoundKey is **identical** to the encryption step: just XOR the same round key again. No inverse operation needed. This is why AddRoundKey is the one AES operation that is the same in both the encryption and decryption algorithms.

#### AddRoundKey and Confusion

Recall Shannon's principle: confusion means each ciphertext bit should depend on many key bits in a complex way. After AddRoundKey, every state byte depends on the corresponding round key byte. And because each round key byte was derived from the master key through the key schedule — which uses SubWord (applying the S-box to key bytes) and XOR — each state byte indirectly depends on many master key bits in a non-trivial way.

> **Key Exam Distinction:** The **key schedule** *generates* round keys. AddRoundKey *applies* them. The *confusion* property comes from AddRoundKey, not from the key schedule. The key schedule's only job is expansion. (This is what Question 5 tests.)

---

### 8.9 The Key Schedule — Why Every Decision Was Made This Way

Now that you understand the rounds, you are ready to understand the key schedule — because now you know what round keys are for, how many are needed, and why they must look different from each other.

#### Why Not Use the Master Key Directly in Every Round?

If every round used the same master key $K$, the cipher would have dangerous symmetry: any pattern in round 1 would repeat identically in round 5 and round 9. An attacker could XOR together the inputs and outputs of corresponding rounds and cancel out the key, effectively attacking each round independently. Different round keys break this symmetry — a change in any bit of the master key should propagate unpredictably to many round keys.

#### How Many Round Keys Are Needed?

For AES-128: one pre-whitening key ($w_0$–$w_3$) plus one key for each of 10 rounds — that is **11 four-word round keys**. Since the master key is 4 words and we need 44 words total ($4 \times 11$), the key schedule must expand 4 words into 44 words.

#### The Two-Case Expansion

The 44 words are generated one at a time. The rule depends on whether the index $i$ is a multiple of 4:

**If $i$ is not a multiple of 4 (simple case):**
$$w[i] = w[i-4] \oplus w[i-1]$$

**If $i$ is a multiple of 4 (every fourth word):**
$$w[i] = w[i-4] \oplus g(w[i-1])$$

---

#### Why g() at Multiples of 4 — Not Every Word, Not Every Other?

This is the question the textbook never answers. Here is the full reasoning.

Consider what happens with the simple case alone: $w[i] = w[i-4] \oplus w[i-1]$. This is a linear recurrence. If you know any four consecutive key words $w[0]$–$w[3]$, you can compute all subsequent words purely through XOR operations — the key schedule is completely **linear**. A linear key schedule means the entire expanded key is an affine function of the master key, and there exist related-key attacks that exploit this linearity to reduce the effective key space.

To break this linearity, AES injects a non-linear transformation exactly once per "generation" of 4 words — at $i = 4, 8, 12, \ldots$ By applying $g()$ to the last word of the previous group (and XORing it into the first word of the new group), every group of 4 words gets at least one non-linear anchor. Why not apply $g()$ to every word? That would be redundant and slow. The critical property is that **each group of 4 words contains at least one non-linear input**, which is achieved by applying $g()$ exactly once per group.

#### Why Each Step of g() Exists

**Step 1 — RotWord: $[a_0, a_1, a_2, a_3] \to [a_1, a_2, a_3, a_0]$**

The four key words $w[4k], w[4k+1], w[4k+2], w[4k+3]$ form a group, with $w[4k+3]$ being the last word. Without rotation, $g()$ would always process the last word's bytes in the same positional slots — byte 0 of $w[4k+3]$ would always feed byte 0 of $w[4k+4]$, byte 1 to byte 1, and so on. This alignment means a known pattern in one byte position would propagate predictably across the key schedule. The rotation *shuffles the byte positions*, ensuring bytes from the last word of one group feed into different positional slots of the first word of the next group — breaking alignment patterns that could enable byte-by-byte analysis.

**Step 2 — SubWord: apply the S-box to each byte**

The simple XOR chain is linear. Even with RotWord, the function is still linear (rotation is a linear operation). SubWord applies the AES S-box — which is non-linear by construction — to each byte of the rotated word. This is the non-linearity injection that breaks the linear key schedule. It is the same S-box used in SubBytes, which means: attacking the key schedule algebraically requires attacking the same mathematical structure that resists attacks in the cipher itself. One design, two security benefits.

**Step 3 — XOR with Rcon: $\oplus\; [\text{Rcon}[j],\; 0\text{x}00,\; 0\text{x}00,\; 0\text{x}00]$**

Even with RotWord and SubWord, there is a subtle remaining risk: if two consecutive applications of $g()$ ever happened to receive the same input (due to a particular master key choice), they would produce the same output, causing round keys to repeat. Rcon prevents this by XORing a **different constant into each application of g()**. The constants are powers of 2 in GF(2⁸): $\text{Rcon}[1] = 0\text{x}01$, $\text{Rcon}[2] = 0\text{x}02$, $\text{Rcon}[3] = 0\text{x}04$, … Each $g()$ call sees a unique Rcon, so even if two calls had identical inputs after RotWord and SubWord, the final XOR would differ — guaranteeing that all 10 applications of $g()$ produce distinct outputs.

The first four expanded words:

$$w_4 = w_0 \oplus g(w_3), \qquad w_5 = w_1 \oplus w_4, \qquad w_6 = w_2 \oplus w_5, \qquad w_7 = w_3 \oplus w_6$$

and so on up to $w_{43}$.

> **Critical exam distinction:** The key schedule **expands** the master key — this is its sole function. The **confusion** property of AES comes from AddRoundKey *applying* the expanded keys. These are two separate operations. (Question 5 answer: (a).)

---

#### Assignment Exercise — Key Schedule
*(Assignment 2, Questions 2 & 3)*

**Q2:** Given $w_0, w_1, w_2, w_3$, how do we obtain $w_4, w_5, w_6, w_7$?

$$w_4 = w_0 \oplus g(w_3)$$
$$w_5 = w_1 \oplus w_4$$
$$w_6 = w_2 \oplus w_5$$
$$w_7 = w_3 \oplus w_6$$

$g()$ is used only when $i$ is a multiple of 4. For all other $i$: $w[i] = w[i-4] \oplus w[i-1]$.

**Q3:** What does $g()$ compute?

Given $w = [a_0, a_1, a_2, a_3]$:
1. **RotWord:** $[a_0, a_1, a_2, a_3] \to [a_1, a_2, a_3, a_0]$
2. **SubWord:** apply S-box to each byte
3. **XOR with Rcon:** $\oplus\; [\text{Rcon}[j], 0\text{x}00, 0\text{x}00, 0\text{x}00]$, where $\text{Rcon}[j] = 2^{j-1}$ in GF(2⁸): $0\text{x}01, 0\text{x}02, 0\text{x}04, 0\text{x}08, \ldots$

---

#### Worked Example — First Key Expansion

**Key:** `2b 7e 15 16 | 28 ae d2 a6 | ab f7 15 88 | 09 cf 4f 3c`

So: $w_0 = [\text{2b, 7e, 15, 16}]$, $w_1 = [\text{28, ae, d2, a6}]$, $w_2 = [\text{ab, f7, 15, 88}]$, $w_3 = [\text{09, cf, 4f, 3c}]$

**Computing $g(w_3) = g([\text{09, cf, 4f, 3c}])$:**

1. RotWord: $[\text{09, cf, 4f, 3c}] \to [\text{cf, 4f, 3c, 09}]$

2. SubWord: $S(\text{0xcf})=\text{0x8a}$, $S(\text{0x4f})=\text{0x84}$, $S(\text{0x3c})=\text{0xeb}$, $S(\text{0x09})=\text{0x01}$  
   → $[\text{8a, 84, eb, 01}]$

3. XOR with $\text{Rcon}[1] = [\text{0x01, 0x00, 0x00, 0x00}]$:  
   $[\text{8a}\oplus\text{01},\; \text{84}\oplus\text{00},\; \text{eb}\oplus\text{00},\; \text{01}\oplus\text{00}] = [\text{8b, 84, eb, 01}]$

**$w_4 = w_0 \oplus g(w_3) = [\text{2b,7e,15,16}] \oplus [\text{8b,84,eb,01}] = [\text{a0, fa, fe, 17}]$** ✓

**$w_5 = w_1 \oplus w_4 = [\text{28,ae,d2,a6}] \oplus [\text{a0,fa,fe,17}] = [\text{88, 54, 2c, b1}]$**

---

### Exam Practice — Key Schedule
*(2025 Midterm Exam, Question 5)*

**Q5:** What is the role of the key schedule in a block cipher?  
a) To expand the master key into multiple round keys  
b) To perform byte substitutions  
c) To add confusion to the cipher  
d) To break plaintext into fixed-size blocks

**Answer: (a).** The key schedule only *expands* the master key. The confusion comes from AddRoundKey *applying* those keys. These are different operations.

---

### 8.10 The Full AES Encryption and Decryption Flow

Before any round-based processing, the state is XORed with $w_0$–$w_3$ — **pre-whitening** (also called initial key addition). This ensures an attacker cannot examine the unmodified plaintext at any point in the cipher, even the very first step.

#### Encryption

```
Input: 128-bit plaintext block
   ↓
1. Arrange as 4×4 state (column-major)
   ↓
2. AddRoundKey with w[0..3]          ← pre-whitening
   ↓
3. For rounds 1 to 9:
   a. SubBytes       (non-linear byte substitution — confusion)
   b. ShiftRows      (row offsets to enable cross-column mixing)
   c. MixColumns     (GF(2⁸) matrix multiply per column — diffusion)
   d. AddRoundKey    (XOR with w[4i..4i+3] — key injection)
   ↓
4. Round 10 (final round — no MixColumns):
   a. SubBytes
   b. ShiftRows
   c. AddRoundKey with w[40..43]
   ↓
Output: 128-bit ciphertext (read column-major)
```

> **Why no MixColumns in round 10?** Including it would require InvMixColumns at the start of decryption, adding computation with zero security benefit — the two operations would cancel. Omitting it is an efficiency optimization with no effect on security.

#### Decryption

AES decryption is **not** the encryption algorithm run in reverse — it requires explicit inverse operations because AES is an SPN (unlike DES's Feistel structure, which is naturally self-inverse). The inverse operations are applied in a mirrored order:

```
Input: 128-bit ciphertext block
   ↓
1. Arrange as 4×4 state
   ↓
2. AddRoundKey with w[40..43]        ← undo the last encryption step first
   ↓
3. For rounds 9 down to 1:
   a. InvShiftRows   (cyclic right shifts: 0, 1, 2, 3 bytes per row)
   b. InvSubBytes    (inverse S-box lookup)
   c. AddRoundKey    (XOR with w[4i..4i+3] — same as encryption, XOR is self-inverse)
   d. InvMixColumns  (inverse matrix multiply)
   ↓
4. Final inverse round (no InvMixColumns):
   a. InvShiftRows
   b. InvSubBytes
   c. AddRoundKey with w[0..3]
   ↓
Output: 128-bit plaintext
```

Note: AddRoundKey is identical in both directions — XOR is its own inverse.

---

### Knowledge Check 8

**Q:** List the four operations of an AES round and the ONE security problem each solves.  
**A:** SubBytes: non-linearity (prevents algebraic attacks). ShiftRows: cross-column byte distribution (so MixColumns can achieve global diffusion). MixColumns: column-level mixing (every output byte depends on all four input bytes of the column). AddRoundKey: key injection (ties the computation to the secret).

**Q:** Why is round 10 missing MixColumns?  
**A:** It would add computation without security — decryption would immediately undo it with InvMixColumns. Omitting it saves a step in both encryption and decryption.

**Q:** Why does $g()$ apply SubWord (the S-box)?  
**A:** Without SubWord, the key schedule would be a purely linear operation (RotWord is linear, XOR is linear). A linear key schedule is vulnerable to related-key attacks. The S-box is non-linear by construction, so SubWord breaks the algebraic structure of the key expansion.

**Q:** Now that you've studied both: how does AES decryption structurally differ from DES decryption?  
**A:** DES (Feistel): the same algorithm runs for both — only key order reverses. AES (SPN): every operation has an explicit mathematical inverse, and the decryption algorithm applies these inverses in a different order. AES decryption is genuinely a different algorithm from encryption.

---

## Part 9: Assignment Solutions

### Assignment 1 (Chapters 2 & 3)

**Q3 (Codebook encryption of "hello"):**

A text file `myfile.txt` contains `hello`. Hexdump gives bytes: `68 65 6C 6C 6F 0A`. Encrypt with a 4-bit block cipher whose codebook is `[6, 0, 13, 4, 3, 1, 14, 8, 7, 12, 9, 15, 5, 2, 11, 10]` (entry at position i is the output for input i).

The codebook maps input nibble → output nibble:
- 0→6, 1→0, 2→13(D), 3→4, 4→3, 5→1, 6→14(E), 7→8
- 8→7, 9→12(C), 10→9, 11→15(F), 12→5, 13→2, 14→11(B), 15→10(A)

Split each byte into two 4-bit nibbles and apply the codebook to each:

| Byte | Hex | High nibble | Encrypted | Low nibble | Encrypted | Result byte |
|------|-----|------------|-----------|-----------|-----------|------------|
| h | 0x68 | 6 | 14 (E) | 8 | 7 | **0xE7** |
| e | 0x65 | 6 | 14 (E) | 5 | 1 | **0xE1** |
| l | 0x6C | 6 | 14 (E) | 12 | 5 | **0xE5** |
| l | 0x6C | 6 | 14 (E) | 12 | 5 | **0xE5** |
| o | 0x6F | 6 | 14 (E) | 15 | 10 (A) | **0xEA** |
| \n | 0x0A | 0 | 6 | 10 | 9 | **0x69** |

**Hexdump of encrypted file: `E7 E1 E5 E5 EA 69`**

---

**Q1 (Base64 encoding of "hello\njello"):**

"hello\njello" → ASCII bytes: `68 65 6C 6C 6F 0A 6A 65 6C 6C 6F` (11 bytes = 88 bits)

Base64 encodes every 3 bytes (24 bits) as 4 characters (each character represents 6 bits). With 11 bytes = 3 groups of 3 + one group of 2, we get 3×4 + 3 + 1 padding = 16 characters total.

| Bytes | Binary bits | 6-bit groups | Values | Base64 |
|-------|------------|-------------|--------|--------|
| h(68) e(65) l(6C) | 01101000 01100101 01101100 | 011010 | 26 | a |
| | | 000110 | 6 | G |
| | | 010101 | 21 | V |
| | | 101100 | 44 | s |
| l(6C) o(6F) \n(0A) | 01101100 01101111 00001010 | 011011 | 27 | b |
| | | 000110 | 6 | G |
| | | 111100 | 60 | 8 |
| | | 001010 | 10 | K |
| j(6A) e(65) l(6C) | 01101010 01100101 01101100 | 011010 | 26 | a |
| | | 100110 | 38 | m |
| | | 010101 | 21 | V |
| | | 101100 | 44 | s |
| l(6C) o(6F) (pad) | 01101100 01101111 xxxxxx | 011011 | 27 | b |
| | | 000110 | 6 | G |
| | | 111100 | 60 | 8 |
| (padding) | | | | = |

**Base64 result: `aGVsbG8KamVsbG8=`**

---

### Assignment 2 (Chapters 8 & 9) — Full Solutions

**Q1:** Steps to construct the 16×16 AES S-box — see Section 8.5 above for the complete answer.

**Q2 & Q3:** Key schedule expansion (w₄–w₇) and g() function — see Section 8.3 above for the complete answer with worked example.

---

## Part 10: Final Exam Preparation

### Relevant 2025 Midterm Questions from Lecture 01 — Topic Index

| Question | Topic | Location in Guide |
|----------|-------|------------------|
| Q1 | Primary goal of cryptanalysis | Part 3 |
| Q2 | Stream vs. block ciphers | Part 1 |
| Q3 | Kerckhoffs's Principle | Part 2 |
| Q4 | Codebook attack requirements | Part 3 |
| Q5 | Role of key schedule (**disputed — see Q5 note**) | Part 8.9 |
| Q6 | Confusion in block cipher | Part 7 |
| Q7 | Why multiple rounds? | Part 7 |
| Q8 | Vigenère with long key | Part 4.5 |
| Q9 | Doubling key length effect (**disputed**) | Part 3 |
| Q10 | AES vs DES (**all of the above**) | Part 8.1 |
| Q11 | Purpose of MixColumns | Part 8.7 |
| Q12 | AES S-box construction | Part 8.5 |
| Q13–Q15 | 2DES, DES group, 3DES EDE | **See Lecture 02 guide** |
| Q16–Q20 | Block cipher modes | **See Lecture 02 guide** |

### Summary of "Questions of Doubt" Clarifications from Lecture 01

| Q# | What the clarification says |
|----|---------------------------|
| **Q5** | Key schedule = **expands** master key. AddRoundKey = **adds confusion**. Answer is (a). |
| **Q9** | Doubling once → effort squares. Long run → exponential. Both (a) and (d) could be correct; (a) is more accurate overall. |
| **Q10** | AES is faster, has larger block/keys, AND is publicly designed. Answer = **(d) all of the above**. |

### Comprehensive Review Questions

**1.** A cipher has a 64! key space (larger than DES's 2⁵⁶). Can it still be broken quickly? Explain.  
**Answer:** Yes. Key space determines only brute-force resistance. A 64! key space is astronomically large, but the cipher might still preserve statistical structure (like monoalphabetic ciphers) and be broken by frequency analysis in minutes. Security requires both large key space AND destruction of statistical/algebraic structure.

**2.** A student implements AES but accidentally applies MixColumns in the final round as well. What happens to security?  
**Answer:** No security change. The extra MixColumns in the final encryption round would be immediately undone by InvMixColumns at the start of decryption. The plaintext recovered would be identical. However, performance is slightly worse, and the correct decryption algorithm now requires an extra InvMixColumns step.

**3.** Why do AES and DES differ in their decryption algorithms, even though both are block ciphers?  
**Answer:** DES uses a Feistel structure, which is inherently self-inverse: run the same structure with round keys in reverse order and you decrypt. AES uses a substitution-permutation network (SPN), where each operation (SubBytes, ShiftRows, MixColumns) requires an explicit mathematical inverse. The SPN doesn't have the Feistel structure's self-inverse property, so a separate decryption algorithm using InvSubBytes, InvShiftRows, InvMixColumns is needed.

**4.** Explain the relationship between ShiftRows and MixColumns in achieving the AES avalanche effect.  
**Answer:** MixColumns mixes bytes within a single column — without ShiftRows, each column would be independent. ShiftRows ensures that after the shift, each column contains bytes that came from different original columns. When MixColumns then operates on this shifted state, it is mixing bytes from all four original columns together. Over two rounds, a change in one byte propagates to all four columns (via ShiftRows) and then mixes within those columns (via MixColumns), ultimately affecting all 16 bytes of the state.

**5.** What is the significance of the irreducible polynomial x⁸ + x⁴ + x³ + x + 1 in AES?  
**Answer:** This polynomial defines the structure of GF(2⁸) — the Galois Field with 256 elements in which all AES arithmetic (SubBytes multiplicative inverses, MixColumns multiplications) is performed. Its choice is not arbitrary: it must be an irreducible polynomial over GF(2) for GF(2⁸) to be a valid field (so that every non-zero element has a multiplicative inverse). The particular polynomial was chosen among all suitable irreducible polynomials for having desirable efficiency and security properties.

---

*End of Lecture 01 Study Guide*  
*Next: Lecture 02 — Block Ciphers (2DES/3DES/MITM, Block Cipher Modes, RC4, Stream Ciphers, Needham-Schroeder, Kerberos)*
