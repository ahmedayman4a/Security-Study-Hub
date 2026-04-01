# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

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
- Version badge (`v0.1.0`) in nav bar and footer
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
