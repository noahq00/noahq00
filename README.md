# Noah Quakkelaar

**Lead Engineer & Security Officer · ISO 27001 Lead Auditor · I build the internal tools and platforms that take manual work out of business processes.**

I work where operations, security and engineering meet, at [Capestone](https://capestone.com), an IoT connectivity company in the Netherlands. I sit with the people who do the work, find where the process breaks, and then design, build, ship and run the tool that fixes it: the database, the API integrations, the access model and the deployment included.

AI is my lever on two levels: **inside the products** (LLM parsing with a deterministic first pass, AI agents that usability-test a shop) and **in how I build** (Claude Code as a pair programmer, with specs, tests and code review as the guardrails).

## What I do

- **Process → product.** Turn a spreadsheet-and-email process into a tool people use every day: billing, customer due diligence, deliveries, pricing lookups, e-commerce.
- **Databases.** Design schemas, write migrations and run them in production (SQLite, PostgreSQL, MySQL).
- **APIs and integrations.** Carrier APIs, ERP, payment providers, chamber of commerce and EU VAT registries, SFTP feeds, and portals that have no API at all.
- **Access and security.** Central sign-on, role-based access per tool, hashed sessions, audit logging, TLS from an internal CA, least-privilege service accounts. As Security Officer and ISO 27001 Lead Auditor I also run internal security audits.
- **Deploy and operate.** Linux servers, nginx, systemd, scheduled jobs, monitoring, runbooks and rollback scripts.

## Currently working on

- **Capestone Platform**: 12 internal tools in one monorepo behind one login; 669 commits and 3,000+ automated tests since July 2026, built together with a colleague.
- **Billing correctness**: every known class of billing error mapped to a check that runs every night.
- **InstantBuyRMS**: go-live of the rebuilt B2B licence webshop (hosting, payment provider).
- **ISO 27001 and internal audit** work across group companies.

## Case studies

> [!NOTE]
> The code behind these projects belongs to my employer and stays private. I'm happy to walk through the architecture, the code and the decisions in an interview.

### 1. Capestone Hub: one login and one access model for 12 internal tools

**Problem.** The internal tools had grown one at a time. Each had its own login or none, each was reachable directly on the server, and nobody could answer the question *"who has access to what, and who changed it?"*

**What I built**

- A Flask service that is the identity of the whole platform. nginx asks the Hub on every request (`auth_request`): no session means a login page; a session without rights for that tool means a redirect back to the Hub with an explanation.
- A role model per user per tool, translated into the role names each tool uses itself.
- Single sign-on across separate hostnames. Browsers won't share one cookie across them, so the Hub hands out a one-time ticket per tool host that becomes a tool session. Sessions and tickets are stored hashed; logging out or deactivating a user ends every tool session on its next request.
- An audit log of every change (who, when, old value, new value), access requests, password reset and welcome mails through an internal mail service.
- A monitoring page with live status, uptime charts and an incident list from Uptime Kuma.
- The database design (9 tables), TLS from an internal CA, one least-privilege service account per tool, and deploy scripts that check file ownership and permissions before and after every rollout.

**Result.** 12 tools behind one gate. Access is granted or revoked in one place, with a full audit trail. 989 automated tests.

**Stack.** Python, Flask, gunicorn, SQLite, bcrypt, nginx `auth_request`, systemd, Uptime Kuma.

### 2. Billing and Rating Engine: from carrier data to proforma invoices, every night

**Problem.** Monthly invoicing for IoT SIM connectivity. Status and usage data come from eight carrier sources (REST APIs, an SFTP feed, daily ZIP files and one portal without an API) and have to be rated against price agreements that differ per customer. Doing that by hand is slow, and every mistake is money.

**What I built**

- A nightly run with seven parallel lanes, each serial inside, non-fatal and timed. It collects status and usage from every source, including incremental call-detail-record (CDR) ingestion over SFTP and a crawler for the portal that has no API.
- A rating engine with 200+ customer price tiers, proforma invoices as PDF, and a SIM management cockpit for sales and operations.
- The V2 database schema and its migrations, rebuildable from the source data at any time.
- A migration from the V1 to the V2 engine, with both engines running side by side and required to agree to four decimal places.

**The interesting part: correctness.** Comparing two engines feels safe, but it can't see an error in the input both engines share. So I designed a correctness framework: a register of checks, coverage per error class, findings that survive a database rebuild, and an independent second measurement per data source. Its first run immediately showed which sources had usage that nothing cross-checked.

**Result.** Status and usage from every source are collected and rated every night, ready for invoicing in the morning. 426 automated tests plus golden-master comparisons.

**Stack.** Python, Flask, SQLite, REST APIs, SFTP, systemd timers, PDF generation.

### 3. InstantBuyRMS: rebuilding a B2B licence webshop

**Problem.** A webshop that sells Teltonika RMS licence credits and management packs needed a rebuild. Payment, delivery of licence codes, EU VAT handling and the hand-off to the ERP all had to run without manual steps.

**What I built**

- A clean WooCommerce shop on Bedrock, with a custom plugin (checkout, VAT, licences, ERP, alerts and security modules), 16 WP-CLI commands and 8 scheduled jobs.
- Automatic licence delivery: payment, then the order is held or released, then a code is assigned atomically (lock plus conditional `UPDATE`), so one code can never be sold twice. Codes never appear in logs, CLI output or invoices.
- A B2B checkout with EU VAT rules: VIES validation with re-checks, reverse charge, a country check on the IP address, PDF invoices, English and Dutch.
- Security by design: a content security policy with nonces, rate limits, fail2ban, a read-only Store API, no customer accounts, and a risk register with 37 risks.
- A full project document set: business and software requirements, architecture, high- and low-level design, security, test and operations plans.
- **AI-driven usability testing.** Five AI persona agents (a mobile user, a foreign buyer, a fleet manager with 300 devices, a keyboard-only user and a customer after purchase) used the shop through Playwright and reported where it failed them. One finding: the pricing calculator steered buyers to the more expensive option.

**Result.** 2,400+ unit tests and 400+ end-to-end tests, 200+ of them security tests, with CI on GitHub Actions. Phase 1 is complete; go-live is in progress.

**Stack.** PHP 8.4, WordPress (Bedrock), WooCommerce, MySQL, PHPUnit, Playwright, GitHub Actions, DDEV.

## Other tools I built

| Tool | What it does |
|---|---|
| **KYC 360** | Checks new business customers against the chamber of commerce, credit data and VAT registries. |
| **SIMgate** | Looks up SIM prices, plans and coverage per country, with a price calculator for sales. |
| **Deliveries** | Tracks incoming supplier deliveries with DHL track & trace. Supplier e-mails go through a deterministic parser first; unknown formats fall back to the Claude API, with a size cap per document to bound the cost. |
| **Payment matching** | Matches payment-provider exports to SIM top-ups and reports per day, week or month. Bank data stay in memory and are never stored. |
| **Carrier export check** | Compares a carrier's IoT export with our own records and flags the differences. |
| **Custom router firmware** | A custom build of the router operating system (RutOS) with a branded web interface. |
| **AI brand guide** | The company's house style as a Claude skill and a shared Gemini Gem, so everyone writes on-brand with AI. |

## How I work

- **Spec, plan, test first, review, verify.** Every change starts on a branch, lands in `dev`, reaches `main` only when the test suite of every affected tool is green, and is deployed from `main`.
- **AI as a pair programmer, not an autopilot.** I build with Claude Code. Tests, code review and written design decisions are the guardrails, and I can explain every decision myself.
- **Documentation people actually read.** Every tool has a README that says what it does *and doesn't* do, what it depends on, and why each decision was made.
- **Measure in production.** After a deploy I check that the new version is really serving, not just that the service reports "active".

## Public example

[**internal-tools-cicd**](https://github.com/noahq00/internal-tools-cicd): the CI/CD pipeline I use for a monorepo of small internal tools, rebuilt from scratch with demo code and fully automated in GitHub Actions. It covers change detection, a test matrix per tool, one required status check, approved deploys, health checks and automatic rollback.

## Stack

Python · Flask · SQL (SQLite, PostgreSQL, MySQL) · PHP · WordPress / WooCommerce · JavaScript · REST APIs · nginx · systemd · Linux · Docker / DDEV · GitHub Actions · pytest · PHPUnit · Playwright · Claude API / Claude Code
