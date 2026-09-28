# O-Banking

> **A Django banking demo — audited, modernized, and rebuilt on a permissively-licensed UI.**

[![Django](https://img.shields.io/badge/Django-5.2%20LTS-092E20)](https://www.djangoproject.com/)
[![Python](https://img.shields.io/badge/Python-3.12-blue)](https://www.python.org/)
[![Tests](https://img.shields.io/badge/tests-294%20passing-brightgreen)]()
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

---

## Table of Contents

| # | Section |
|---|---|
| 1 | [Context](#1-context) |
| 2 | [Problem Statement](#2-problem-statement) |
| 3 | [Solution Overview](#3-solution-overview) |
| 4 | [Screenshots](#4-screenshots) |
| 5 | [Architecture](#5-architecture) |
| 6 | [Key Features](#6-key-features) |
| 7 | [Technical Stack](#7-technical-stack) |
| 8 | [Engineering Decisions](#8-engineering-decisions) |
| 9 | [Results](#9-results) |
| 10 | [Quick Start](#10-quick-start) |
| 11 | [Test Credentials](#11-test-credentials) |
| 12 | [Seed Instructions](#12-seed-instructions) |
| 13 | [Project Structure](#13-project-structure) |
| 14 | [Documentation](#14-documentation) |
| 15 | [Future Work](#15-future-work) |
| 16 | [Author](#16-author) |

---

## 1. Context

O-Banking is a **server-rendered Django monolith** for a simulated bank. It covers registration and email login, KYC submission with document upload, atomic transfers, payment requests and settlement, a dashboard with KPIs and charts, statements with CSV export, saved recipients, a notification center, support tickets, user settings, spending categories, savings goals, and per-user transfer limits.

The project began as a **2022 Django 3.1 codebase** that had never been audited — imported as received and left alone. It was returned to in **2026** to fix its security problems, modernize the stack, and rebuild the interface on a theme that can legally ship.

---

## 2. Problem Statement

### 2.1 The Core Problem

> **A never-audited money-moving codebase carries vulnerabilities by default — and a web app that mishandles balances is worse than one that doesn't move money at all.**

### 2.2 Sub-problems

| # | Sub-problem | Why it's hard |
|---|---|---|
| **P1** | **Security.** Money-theft vector (negative transfer credited sender, debited recipient), IDOR on every transaction lookup, 14 views without `@login_required`, plaintext PIN printed to stdout, URL-driven account redirect | Every fix needs a test that first proves the defect, then proves the fix |
| **P2** | **UI and licensing.** Vendored commercial template, no LICENSE, GPL-v2 dependency, two divergent asset trees, dead sidebar links | Publishing was legally impossible; the interface was also half-broken |
| **P3** | **Testing and demo data.** Three stub tests totalling four lines, no README, no CI, single-user SQLite with no seed data | Nothing could be demonstrated or verified |

### 2.3 Why Traditional Solutions Failed

| Approach | Why it didn't work here |
|---|---|
| **Patch the vulnerabilities in place** | Each fix needed a regression test first, or the bug would return. Testing came first, not second. |
| **Upgrade Django in one jump** | 3.1 → 5.2 spans multiple breaking-change boundaries (`url()` removal, timezone strictness, third-party incompatibilities). Staged upgrades were safer. |
| **Keep the old theme** | No LICENSE and a GPL-v2 dependency — the project could not be published. A permissive replacement was mandatory. |
| **Ship without tests** | Every subsequent feature would have been built on unverified money movement. The audit and the suite were prerequisites, not extras. |

---

## 3. Solution Overview

Four workstreams, each with its own deliverable.

### 3.1 Security Audit — 11 phases

Each phase landed with a test that **first proved the defect, then proved the fix**. The suite that grew out of this work is **294 Django `TestCase` tests** covering money movement, authorization, the seeder, and every feature added since.

### 3.2 Stack Upgrade — Django 3.1 → 5.2 LTS

Python 3.9 → 3.12. The dependency list was cut to the **five packages the code actually imports**, then extended with two more when 2FA was added:

`Django` · `django-jazzmin` · `django-import-export` · `shortuuid` · `Pillow` · `pyotp` · `qrcode`

### 3.3 UI Migration — Commercial Theme → Tabler 1.5.1 (MIT)

Vendored as prebuilt files so the app runs offline.

| Metric | Before | After |
|---|---|---|
| `static/` size | 13.99 MB | 2.38 MB |
| File count | 206 | 11 |

A purple-accent design layer with dark mode lives in one stylesheet: `static/tabler/css/ob-theme.css`.

### 3.4 Demo Data & Feature Areas

`seed_demo` generates **150 users and 3,000 transactions over 24 months**, with balances replayed to match. Feature areas built on top include statements + CSV export, saved recipients, notification center, support tickets, and user settings.

A second round of work (**v2.2**) added: immutable audit log, rate limiting on four credential-checking endpoints, TOTP 2FA, full-text search, spending categories, savings goals, and daily / weekly / monthly transfer limits — all enforced inside the atomic money-path block.

---

## 4. Screenshots

### Dashboard

| Light mode | Dark mode |
| --- | --- |
| ![Dashboard light mode](docs/screenshots/dashboard/dashboard-light.png) | ![Dashboard dark mode](docs/screenshots/dashboard/dashboard-dark.png) |

| Charts | Tables |
| --- | --- |
| ![Dashboard charts](docs/screenshots/dashboard/dashboard-charts.png) | ![Dashboard tables](docs/screenshots/dashboard/dashboard-tables.png) |

![Transaction history](docs/screenshots/dashboard/dashboard-history.png)

8 KPIs, 5 charts, 2 summary tables, and a paginated transaction history. A single period-and-type filter bar (7 days / 30 days / 90 days / 1 year / All time · All types / Transfers / Requests) recomputes every KPI and every chart from the same query parameters.

### Security and account features

| Two-factor enrolment | Recovery codes | Two-factor challenge |
| --- | --- | --- |
| ![2FA setup](docs/screenshots/security/2fa-setup.png) | ![2FA recovery codes](docs/screenshots/security/2fa-recovery-codes.png) | ![2FA challenge](docs/screenshots/security/2fa-challenge.png) |

| Audit log (admin) | Transfer limits (admin) |
| --- | --- |
| ![Audit log](docs/screenshots/security/audit-log.png) | ![Transfer limits](docs/screenshots/security/transfer-limits.png) |

### Banking

| Transactions | Account |
| --- | --- |
| ![Transactions](docs/screenshots/banking/transactions.png) | ![Account profile](docs/screenshots/banking/account.png) |

| Statements | Recipients |
| --- | --- |
| ![Statements](docs/screenshots/banking/statements.png) | ![Recipients](docs/screenshots/banking/recipients.png) |

| Categories | Savings goals |
| --- | --- |
| ![Categories](docs/screenshots/banking/categories.png) | ![Savings goals](docs/screenshots/banking/goals.png) |

| Transfer complete | Settings | Support ticket |
| --- | --- | --- |
| ![Transfer complete](docs/screenshots/banking/transfer-complete.png) | ![Settings](docs/screenshots/banking/settings.png) | ![Support ticket](docs/screenshots/banking/support-ticket.png) |

### Public

| Landing | Sign in | Sign up |
| --- | --- | --- |
| ![Landing page](docs/screenshots/marketing/landing.png) | ![Sign in](docs/screenshots/auth/sign-in.png) | ![Sign up](docs/screenshots/auth/sign-up.png) |

---

## 5. Architecture

A server-rendered Django monolith. No separate API, no SPA, no JavaScript build step — every page is rendered by a view and returned as HTML.

| Area | Detail |
|---|---|
| **Shells** | `base.html` (public) and `dashboard-base.html` (authenticated). Their `:root` blocks are byte-identical; a probe asserts they stay that way. |
| **Apps** | `core` (money movement, limits), `userauths` (auth, 2FA), `account` (accounts, KYC, dashboard, categories, goals), `pages` (blog, contact), `audit` (append-only log) |
| **Data** | SQLite. `Account.account_balance` is the single source of truth — no ledger, no event-sourced table. |
| **Transfer direction** | transfer: sender → receiver. Settled request: receiver → sender (requester credited). |
| **Auth** | Email is the login field (`USERNAME_FIELD = 'email'`); `username` is required but not the credential. |
| **2FA** | Optional TOTP. On enable, `LoginView` parks the pending user in the session and redirects to a challenge; `login()` runs only after the code or a recovery code verifies. |
| **Limits** | Daily / weekly / monthly caps per user on outgoing transfers, checked inside the same `atomic()` block that moves the money. |
| **Charts** | Chart.js 4.4.4, vendored. Five dashboard charts: line, doughnut, grouped bar, area, spend-by-category. |

---

## 6. Key Features

### 6.1 Banking

| Feature | Detail |
|---|---|
| 🏦 **Email-login accounts** | Custom user model with `email` as the credential |
| 📄 **KYC submission** | Document upload with an admin confirmation workflow |
| 💸 **Atomic transfers** | Password re-entry, `select_for_update`, URL/transaction mismatch guard, per-user limits |
| 📩 **Payment requests** | Request + settlement with a distinct direction rule |
| 📊 **Dashboard** | 8 KPIs, 5 charts, 2 summary tables, filterable paginated history |
| 📜 **Statements** | Range selector (month / 3 months / year / 12 months) + CSV export |
| 🔍 **Full-text search** | Across description, transaction id, and counterparty |
| 🏷️ **Spending categories** | Custom categories + dashboard doughnut chart |
| 🎯 **Savings goals** | Progress tracking + dashboard widget |
| 👥 **Saved recipients** | One-click transfers |

### 6.2 Security

| Feature | Detail |
|---|---|
| 🔐 **TOTP 2FA** | Optional; session-parked challenge; single-use recovery codes |
| 📋 **Audit log** | Immutable, append-only; read-only in the admin |
| 🚦 **Rate limiting** | Login (5/15m), transfer, settlement, payment-request confirmation (10/hr) |
| 🧱 **CSRF + session hardening** | Regenerated after 2FA elevation; timing-safe comparison |
| 🧪 **294 tests** | Money movement, authorization, seeder, every feature |

### 6.3 Operations

| Feature | Detail |
|---|---|
| 🌱 **`seed_demo`** | 150 users, 3,000 transactions, 24 months, balances replayed |
| 🎨 **Dark mode** | CSS-variable driven, chart palette rebuilds on toggle |
| 🔔 **Notification center** | Bell + unread badge + dropdown + paginated list |
| 🎫 **Support tickets** | Inline create, thread view, staff replies from admin |
| 📰 **Public blog + contact** | List, detail, category filter, contact form |
| 🛠️ **Django admin** | Jazzmin theme + `ImportExportModelAdmin` |

---

## 7. Technical Stack

| Layer | Technologies |
|---|---|
| **Language** | Python 3.12 |
| **Framework** | Django 5.2 LTS (supported through April 2028) |
| **Database** | SQLite (single-file demo) |
| **UI framework** | Tabler 1.5.1 (MIT), vendored offline |
| **Charts** | Chart.js 4.4.4, vendored |
| **Admin** | django-jazzmin 3.0.5 + django-import-export 4.4.1 |
| **IDs** | shortuuid 1.0.13 |
| **Images** | Pillow 12.3.0 |
| **2FA** | pyotp 2.9.0 + qrcode[pil] 7.4.2 |
| **CI** | GitHub Actions — check, migration drift, tests |

---

## 8. Engineering Decisions

| # | Decision | Rationale |
|---|---|---|
| **a** | **PIN removed, not hashed** | A 4-digit keyspace (10^4) doesn't benefit from hashing, and it was printed to stdout on every transfer — i.e. logged. Transfers now use `check_password`. |
| **b** | **Asymmetric settlement direction** | For a payment request, the stored `sender` is the requester and `receiver` is the payer — money moves receiver → sender on settlement. The analytics layer encodes this once; templates mirror it. |
| **c** | **Seeder replays balance mutation** | No ledger exists — `account_balance` is a mutable scalar. The seeder replays the same mutation views perform, so KPIs and charts never contradict. |
| **d** | **URL/transaction mismatch rejected** | The confirmation views derived the credited account from the URL — a POST with a different number redirected funds. Both views now require the URL account to match the stored counterparty. |
| **e** | **HSTS short, preload off** | `security.W021` accepted by design. Preload at one year is right for a real bank, but a footgun for a portfolio demo. Rationale inline in `settings.py`. |
| **f** | **Two shells mirrored, not unified** | `base.html` and `dashboard-base.html` hold byte-identical `:root` blocks; a probe asserts this. A shared partial was rejected because a mid-migration edit would have silently diverged them. |
| **g** | **Chart.js reads CSS variables** | Every chart colour resolves through `getComputedStyle` against `--tblr-*` at paint time. Dark mode dispatches `ob:theme-changed`; charts rebuild against the new palette. |
| **h** | **Weaker hasher under test** | `manage.py test` swaps PBKDF2 for MD5. PBKDF2 cost ~1.7 s/hash and dominated the suite. Production hasher is unchanged. |
| **i** | **Notification signal observes, never modifies** | Status transitions are compared between `pre_save` and `post_save`. The signal reads transactions; it never changes how they're written. The money path is untouched. |
| **j** | **Audit log is append-only** | `audit.LogEntry` refuses `save()`, `delete()`, `update()`, `bulk_create()`, `get_or_create()`, and `update_or_create()`. `actor` uses `SET_NULL` so deleting a user leaves entries intact. `audit.utils.log()` never raises. |
| **k** | **Fixed-window rate limiting** | Login: 5/email/15min. Transfer/settlement/confirmation: 10/user/hour. `cache.add` sets the window; `cache.incr` never resets it. Only POSTs consume quota. LocMemCache — per-process; documented for a shared backend in production. |
| **l** | **TOTP secret not encrypted at rest** | Deliberate demo limitation; production needs KMS. Documented in `userauths/totp.py`. Recovery codes **are** hashed. |
| **m** | **2FA challenge parks pending user in session** | Correct password alone doesn't authenticate when a confirmed device exists. `login()` runs only after code verification. Pending state expires after 5 minutes. |
| **n** | **Spend is derived, not stored** | The category chart is computed from `Transaction` rows at render time. Category deletion or amount edit retroactively updates it. `SET_NULL` makes deleted-category slices appear as "Uncategorized" without a data migration. |
| **o** | **Savings goals do not move money** | A "contribute" button would need a second money-movement path — its own locking, guards, and rate limiter. Instead, the user records progress. Keeps the money path single. |
| **p** | **Transfer limit checked inside `atomic()`** | Outside the block, two concurrent confirmations could each see an under-limit total. Inside, adds five bounded queries — acceptable at demo scale, cacheable at production. |

---

## 9. Results

### 9.1 Quantified Outcomes

| Metric | Before | After | Change |
|---|---|---|---|
| `static/` asset size | 13.99 MB | 2.38 MB | **−83%** |
| `static/` file count | 206 | 11 | **−95%** |
| Backend tests | 0 (3 stubs, 4 lines) | 294 | — |
| Test suite runtime | ~dominated by PBKDF2 | ~17 s | via MD5 test-runner swap |
| Django version | 3.1 | 5.2 LTS | supported through Apr 2028 |
| Python version | 3.9 | 3.12 | — |
| Seeded demo data | single user, no history | 150 users · 3,000 transactions · 24 months | — |
| Feature areas added in v2.2 | 0 | 7 | audit log, rate limiting, 2FA, search, categories, goals, limits |

### 9.2 Correctness Proofs

| Proof | Evidence |
|---|---|
| **Critical bugs closed** | 7 from the original audit |
| **Fixes that surfaced new bugs** | 5 (balance corruption, PIN to stdout, replayable transfer, settlement KYC crash, URL-direction redirect) |
| **Money-path safety** | Balance mutation + transfer limit both inside the same `atomic()` + `select_for_update()` block |
| **Test coverage** | 294 tests — money movement, authorization, seeder, every feature |
| **Shell integrity** | Probe asserts the two shells' `:root` blocks remain byte-identical |

### 9.3 What's NOT Done (documented, not hidden)

| Gap | Reason |
|---|---|
| **SQLite in production** | Single-writer only. Postgres is the documented migration path. |
| **2FA secret at rest unencrypted** | Demo limitation; production needs a KMS-backed encrypted field. |
| **Server-side TOTP replay prevention** | A code accepted once can be reused within its 30-second window. Listed in Future Work. |
| **Rate limiter uses LocMemCache** | Per-process only. Needs a shared cache (Redis) for multi-worker deployments. |
| **Savings goals are tracking-only** | Deliberately not wired into the transfer flow — keeps the money path single. |

---

## 10. Quick Start

**Prerequisites:** Python 3.12 and Git.

```bash
git clone https://github.com/oussama-attouch/O-Banking_Payment_Project_V2.git
cd O-Banking_Payment_Project_V2
py -3.12 -m venv venv
.\venv\Scripts\pip install -r requirements.txt
$env:DJANGO_DEBUG=1
.\venv\Scripts\python.exe manage.py migrate
.\venv\Scripts\python.exe manage.py seed_demo
.\venv\Scripts\python.exe manage.py createsuperuser
.\venv\Scripts\python.exe manage.py runserver
```

Open **http://127.0.0.1:8000/**. The `DJANGO_DEBUG=1` env var is required for local work; see `.env.example` for the full list.

---

## 11. Test Credentials

```
Demo user: user1@demo.local / DemoPass123!
```

- Users **1–120** have KYC records and transaction history
- Users **121–150** are dormant accounts with no activity

**To try 2FA:** log in as the demo user → **Settings** → **Enable 2FA** → scan the QR code → confirm. The next login will ask for a code.

---

## 12. Seed Instructions

| Flag | Default | Meaning |
| --- | --- | --- |
| `--users N` | 150 | Number of users to create |
| `--transactions N` | 3000 | Number of transactions to generate |
| `--months N` | 24 | How far back to spread the history |
| `--seed N` | 42 | Random seed, for reproducible data |
| `--force` | — | Wipe existing seeded rows, then reseed |
| `--wipe` | — | Delete all seeded data and exit |

**Idempotent.** Refuses to run on a database that already contains seeded rows unless `--force` or `--wipe` is passed.

---

## 13. Project Structure

```
payment_prj/                 Django project: settings, root URLconf, WSGI/ASGI entry points
core/                        Money movement, transfer-limit model + helper, landing, seed_demo
userauths/                   Custom user model with email login, auth views, TOTP
account/                     Accounts, KYC, recipients, notifications, support, categories,
                             savings goals, statements, dashboard analytics
pages/                       Public blog and contact form
audit/                       Append-only LogEntry, middleware, log() helper, read-only admin
templates/                   All HTML; partials/ holds the two shells
static/                      Vendored Tabler 1.5.1, Chart.js, and ob-theme.css
docs/                        Screenshot library and provenance notes
venv/                        Virtual environment (gitignored)
manage.py                    Django management entry point
requirements.txt             The seven runtime dependencies, pinned
db.sqlite3                   Development database (gitignored)
media/                       User-uploaded KYC documents (gitignored)
.env.example                 Documented environment variables
LICENSE                      MIT
README.md                    This file
THIRD-PARTY.md               Third-party notices and licence provenance
.github/workflows/ci.yml     CI: check, migration drift, tests
```

---

## 14. Documentation

| Document | Purpose |
|---|---|
| [THIRD-PARTY.md](./THIRD-PARTY.md) | Third-party licences and asset provenance |
| [docs/screenshots/](./docs/screenshots/) | Full screenshot library |
| [docs/README.md](./docs/README.md) | Screenshot provenance notes |

The audit and modernization ran as a **numbered phase sequence** — from the security fixes in Phase 1 through the v2.2 hardening work. Each phase is a commit; the full sequence is in the commit log.

---

## 15. Future Work

| Feature | Why it matters |
|---|---|
| **Scheduled transfers** | A `ScheduledTransfer` model + management command that materialises a `processing` transaction at the scheduled time, then notifies the user to confirm with their password. |
| **Bill splitting** | `Bill` and `BillShare` models, with shares settled by the existing transfer flow — no new money path. |
| **PostgreSQL for production** | SQLite supports only one writer at a time. |
| **Whitenoise** | Serve static files in production without a separate web server. |
| **Shared cache (Redis) for rate limiting** | Enforce quota across multiple worker processes. |
| **Server-side TOTP replay prevention** | Cache the last accepted time-step per device so a code cannot be reused within its 30-second window. |
| **Encrypt `TOTPDevice.secret` at rest** | KMS-managed key. |
| **End-to-end settlement test** | Two browser sessions. |
| **Admin KYC confirmation workflow** | `kyc_confirmed` is currently written only through the admin list view. |
| **Bulk raw SQL for the seeder** | Replace the per-row date update — 10× scale. |
| **Dedicated test settings module** | Replace `"test" in sys.argv` with stricter test-runner detection. |

---

## 16. Author

**Oussama Attouch**

[GitHub](https://github.com/oussama-attouch) · [LinkedIn](https://linkedin.com/in/oussama-attouch-bb1558261)

Released under the **MIT License**. See [LICENSE](./LICENSE).

---

**Built with:** 🐍 Python · 🎯 Django · 🎨 Tabler · 📊 Chart.js · 🔐 pyotp
