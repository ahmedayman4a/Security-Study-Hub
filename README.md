# CSE 464 — Computer System Security · Study Hub

> **Interactive study guides and practice exams** for the CS464 Computer System Security course at Alexandria University.
> Deployed as a static site on GitHub Pages — no build step required.

[![Version](https://img.shields.io/badge/version-v0.1.0-a78bfa?style=flat-square)](CHANGELOG.md)
[![Lectures](https://img.shields.io/badge/lectures-6-6c8aff?style=flat-square)](#lectures)
[![Exam Questions](https://img.shields.io/badge/exam%20questions-60-34d399?style=flat-square)](#exam-files)

---

## What's in here

Each lecture has two companion files — a **Markdown source** (the content source of truth) and an **HTML study tool** (interactive, rendered in the browser).

| Lecture | Topic | Study Guide | Study Tool |
|---------|-------|-------------|------------|
| 01 | AES & Classical Cryptography | `lecture01-study-guide.md` | `lecture01-study.html` |
| 02 | Block Ciphers, Stream Ciphers & Key Distribution | `lecture02-study-guide.md` | `lecture02-study.html` |
| 03 | Public-Key Cryptography & RSA | `lecture03-study-guide.md` | `lecture03-study.html` |
| 04 | More Public-Key Cryptography (PKI, DH, ElGamal) | `lecture04-study-guide.md` | `lecture04-study.html` |
| 05 | Cryptographically Secure Hashing | `lecture05-study-guide.md` | `lecture05-study.html` |
| 06 | Secure Random Number Generation | `lecture06-study-guide.md` | `lecture06-study.html` |

---

## Exam Files

All questions are drawn from the **2025 Midterm Exam** and cross-referenced with the *Questions of Doubt* clarifications document. Answers are revealed immediately on selection.

### Disputed questions

Two questions have **no correct answer** among the given options, as officially clarified:

- **Q20** — "Which mode propagates a single-bit error to ALL subsequent blocks?" → No mode does this; CBC propagates to the next block only.
- **Q52** — "What is weak collision resistance?" → None of the options correctly defines second pre-image resistance.

**Q29** — Both (b) and (c) are valid. Ticket encryption keys (K_TGS, K_Service) are long-term keys.

---

## File Structure

```
study_security/
│
├── index.html                    ← Landing page / study hub (deploy this)
├── README.md                     ← This file
├── CHANGELOG.md                  ← Version history
├── .gitignore
│
├── lectures/                     ← Lecture study materials
│   ├── lecture01-study-guide.md  ┐
│   ├── lecture01-study.html      │
│   ├── lecture02-study-guide.md  │
│   ├── lecture02-study.html      │
│   ├── lecture03-study-guide.md  ├── Markdown source + HTML tool per lecture
│   ├── lecture03-study.html      │
│   ├── lecture04-study-guide.md  │
│   ├── lecture04-study.html      │
│   ├── lecture05-study-guide.md  │
│   ├── lecture05-study.html      │
│   ├── lecture06-study-guide.md  │
│   └── lecture06-study.html      ┘
│
├── exams/                        ← Exam & practice files
│   ├── exam-common.css           ← Shared stylesheet
│   ├── exam-2025-midterm.html    ← Full 60-question timed midterm
│   ├── exam-l01.html             ┐
│   ├── exam-l02.html             │
│   ├── exam-l03.html             ├── Per-lecture practice exams
│   ├── exam-l04.html             │
│   └── exam-l05.html             ┘
│
└── source/                       ← Raw PDFs (gitignored, not served)
    ├── Lectures/
    ├── Exams/
    └── sheets/
```

---

## Deploying on GitHub Pages

1. Push the repository to GitHub.
2. Go to **Settings → Pages**.
3. Set **Source** to `Deploy from a branch`, branch `main`, folder `/ (root)`.
4. After a minute, the site is live at `https://<username>.github.io/<repo-name>/`.
5. The entry point is `index.html` — share that URL.

> **No build step needed.** All HTML files are self-contained (KaTeX loaded from CDN).

---

## Authoring Notes

- **Markdown files are the source of truth.** The HTML files mirror the markdown content.
- When editing a lecture, update the `.md` first, then reflect changes in the corresponding `.html`.
- Math is rendered via **[KaTeX](https://katex.org/)** using `\(inline\)` and `\[display\]` delimiters.
- The `checkAnswer` / `ca` JavaScript function in each study/exam HTML handles immediate answer feedback. The pattern used in exam files is the same across all per-lecture files — see `exam-l02.html` for the version that also handles disputed questions.

---

## Versioning

Version numbers follow [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`):

| Part | When to increment |
|------|-------------------|
| PATCH | Fix typos, fix wrong answers, minor clarifications |
| MINOR | Add new content, new questions, new lecture coverage |
| MAJOR | Complete overhaul of structure or scope |

Version is displayed in:
- The `<meta name="version">` tag of each HTML file
- The nav bar badge in `index.html`
- The footer of `index.html`
- The comment header of `exam-common.css`

See [CHANGELOG.md](CHANGELOG.md) for the full history.

---

## Course Info

- **Course:** CSE 464 — Computer System Security
- **University:** Alexandria University, Faculty of Engineering
- **Semester:** Fall 2025
