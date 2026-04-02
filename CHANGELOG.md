# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

---

## 0.1.1

### Changed

#### Lecture 02 — pedagogy and alignment with peer review (`lecture02-feedback.md`)
- **`lectures/lecture02-study-guide.md`** — Major rewrite of explanatory prose (exam Q&A and structure preserved):
  - Opening framed as **four recurring design pressures** (repeated structure, keystream reuse, replay, scale) that unify the whole lecture.
  - **2DES / MITM** — Two distinct failure modes (DES-as-group vs. meet-in-the-middle); attack explained via the **searchable seam** at the intermediate state and named as a **time–memory tradeoff**.
  - **Block modes** — CBC justified as “XOR before encryption” (what the cipher sees); CFB/OFB motivated by byte-at-a-time use, ciphertext vs. keystream feedback, and noisy channels; explicit **keystream-then-XOR** unifier for CFB, OFB, CTR, and RC4 (**two-time pad**); contrast of **CBC IV unpredictability** vs. **CTR nonce uniqueness**.
  - **RC4** — Contrasted with a naive failing design; rationale for KSA **j** accumulation and PRGA output indirection; “what breaks if” for KSA/PRGA variants.
  - **Needham-Schroeder** — Built from broken simpler attempts (ticket, N₁, N₂); **Denning–Sacco** residual replay called out in the main narrative.
  - **Kerberos** — Presented as **NS under operational pressure** (four problems → three design decisions: TGT/SSO, timestamps vs. nonces and clock sync tradeoff, AS/TGS separation); NS ticket vs. TGT pattern connected.
  - **Exam prep** — Understanding-oriented review questions; **“What breaks if…”** master list.
- **`lectures/lecture02-study.html`** — Content-only updates (no full rewrite): overview four-problems grid; 2DES framing and time–memory tradeoff; CBC “XOR before” callout; CFB ciphertext-feedback rationale; OFB two-time-pad callout; CTR keystream-unifier callout; RC4 weak-alternative and KSA/PRGA rationale; NS ticket/freshness narrative and residual-replay callout; Kerberos evolutionary narrative, timestamp tradeoffs, “what breaks if” callouts, NS↔TGT connection; exam-prep understanding questions and “What breaks if” list.

---

## [0.1.0] — 2026-03-31

Initial release. Complete study hub for CSE 464 Computer System Security.

### Added

#### Study Guides (Markdown source)
- `lecture01-study-guide.md` — AES & Classical Cryptography
- `lecture02-study-guide.md` — Block Ciphers, Stream Ciphers & Key Distribution
- `lecture03-study-guide.md` — Public-Key Cryptography & RSA
- `lecture04-study-guide.md` — More Public-Key Cryptography (PKI, DH, ElGamal)
- `lecture05-study-guide.md` — Cryptographically Secure Hashing
- `lecture06-study-guide.md` — Secure Random Number Generation

#### Interactive HTML Study Tools
- `lecture01-study.html` through `lecture06-study.html`
- Each tool includes: prose-first explanations, embedded worked examples, interactive quiz cards with immediate feedback, KaTeX math rendering, sidebar navigation

#### Exam Files
- `exam-2025-midterm.html` — Full 2025 Midterm (all 60 questions, 60-minute timer, grade calculation)
- `exam-l01.html` — L01 practice exam (Q1–Q12)
- `exam-l02.html` — L02 practice exam (Q13–Q29)
- `exam-l03.html` — L03 practice exam (Q30–Q38)
- `exam-l04.html` — L04 practice exam (Q39–Q48)
- `exam-l05.html` — L05 practice exam (Q49–Q60)
- `exam-common.css` — Shared styles for all exam files

#### Site Infrastructure
- `index.html` — Landing page with lecture cards, exam section, scroll progress bar
- Version badge (`v0.1.2`) in nav bar and footer
- `README.md` — Repository documentation
- `CHANGELOG.md` — This file
- `.gitignore`

### Notes on 2025 Midterm Questions

The following questions are flagged based on the official *Questions of Doubt* clarification document:

| Question | Issue | Resolution |
|----------|-------|------------|
| Q20 | No mode propagates an error to ALL subsequent blocks | Marked **disputed** — no correct answer exists among options |
| Q29 | Both (b) and (c) are valid answers | Both marked as correct in exam HTML |
| Q52 | No option correctly defines weak collision resistance | Marked **disputed** — no correct answer exists among options |
| Q18 | (d) was clarified as correct over (b) | (d) marked as correct |
| Q44 | (a) "key exchange" confirmed as correct | (a) marked as correct |
| Q54 | (c) confirmed correct | (c) marked as correct |

---

## Future Work

- [ ] Add L06 exam (currently no midterm questions cover L06)
- [ ] Add generated practice questions (non-midterm) per lecture
- [ ] Add a combined practice exam across all lectures
- [ ] Dark/light theme toggle
- [ ] Print-to-PDF stylesheet for study guides
