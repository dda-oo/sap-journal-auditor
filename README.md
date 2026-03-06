<<<<<<< HEAD
# 🔍 SAP Journal Auditor

> **An OpenClaw skill that audits SAP FI/CO journal entry exports for anomalies, duplicate postings, approval bypass patterns, and period-end irregularities — and delivers a professional audit memo in English or German.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-blue)](https://openclaw.dev)
[![Built by RadarRoster](https://img.shields.io/badge/Built%20by-RadarRoster-orange)](https://radarroster.com)
[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D16-green)](https://nodejs.org)

---

## What It Does

Upload a SAP FI/CO journal export (CSV or Excel) to your OpenClaw-connected messaging app (Telegram, WhatsApp, Slack, or CLI) and receive a structured audit memo with flagged entries within seconds.

**Audit Checks Performed:**

| Check | Description | Risk Levels |
|---|---|---|
| 🔁 Duplicate Postings | Same amount + account + cost center within ±2 days | HIGH / MEDIUM |
| 💶 Round Amount Anomalies | Round figures on accrual accounts at period-end | HIGH / MEDIUM / LOW |
| 📅 Period-End Outliers | Accruals posted in last 3 days of fiscal period | HIGH |
| 🏗️ Cost Center Mismatch | Account outside typical range for its cost center | MEDIUM |
| 🔓 Approval Bypass | Missing documentation, backdated postings | HIGH / MEDIUM |
| 🏢 Intercompany / GR-IR | IC postings without reference, open GR/IR > 60 days | HIGH / MEDIUM |
| 👤 User Pattern Analysis | High-volume users, unusually large single postings | MEDIUM / LOW |

---

## Supported SAP Export Formats

| SAP Transaction | Description | Format |
|---|---|---|
| `FAGLL03` | General Ledger Line Items | CSV / Excel |
| `FB03` | Document Display | CSV |
| `KSB1` | Cost Center Line Items | CSV / Excel |
| ACDOCA extract | Universal Journal (S/4HANA) | CSV / Excel |
| Any FI/CO export | Custom column mapping supported | CSV / Excel |

The parser handles both **English and German SAP column headers** and both **European** (`1.234,56`) and **US** (`1,234.56`) number formats.

---

## Installation

### OpenClaw (primary usage)

```bash
# Install OpenClaw if not already done
npm install -g openclaw@latest
openclaw onboard

# Install this skill
openclaw skill install sap-journal-auditor
```

### Standalone (CLI / GitHub testing)

```bash
git clone https://github.com/dda-oo/sap-journal-auditor.git
cd sap-journal-auditor
npm install        # optional — skill works without npm packages too
node tests/test.js # run the full test suite
```

> **Zero-dependency mode:** The skill runs without `npm install` using built-in CSV and XLSX parsers. Installing npm packages (`csv-parse`, `xlsx`) enables full format support and is recommended for production.

---

## Usage

### Via OpenClaw Messaging (Telegram / Slack / WhatsApp)

```
You:   [upload journal_march_2025.csv]
       Audit this SAP journal export for March

Bot:   ⏳ Running audit checks... please wait.
       ✅ Audit Complete
       📊 Entries analyzed: 32
       🚩 Total findings: 15
       🔴 Critical: 2 | 🟠 High: 6
       📋 Overall risk: CRITICAL
       [audit_memo.md attached]
       [flagged_entries.csv attached]
```

### With Options

```
# German output
Prüfe diese Buchungen auf Deutsch

# Filter to specific period
Audit this file for period 2025-03

# Lower duplicate threshold
Audit with threshold 500
```

### CLI / Standalone

```javascript
const { parseJournalFile } = require('./lib/parser');
const { runAllChecks }     = require('./lib/auditor');
const { generateMemo }     = require('./lib/reporter');
const { exportFlaggedCSV } = require('./lib/exporter');
const fs = require('fs');

const entries = await parseJournalFile('./your_export.csv', { period: '2025-03' });
const results = runAllChecks(entries, { duplicateThreshold: 1000 });
const memo    = generateMemo(results, entries, { language: 'en' });

fs.writeFileSync('audit_memo.md', memo);
exportFlaggedCSV(results.findings, 'flagged_entries.csv');

console.log(`Found ${results.findings.length} issues. Overall risk: ${results.overallRisk}`);
```

---

## Output

### 1. Audit Memo (`audit_memo.md`)

A professional Markdown document containing:
- **Metadata** — date, period, entry count, overall risk level
- **Executive Summary** — 3–5 sentence narrative
- **Findings by Category** — summary table
- **Full Finding Table** — ID, risk, type, document numbers, amounts, accounts
- **Detailed Descriptions** — per-finding explanation and recommendation
- **Recommendations** — prioritized action list for HIGH/CRITICAL items

### 2. Flagged Entries CSV (`flagged_entries.csv`)

Machine-readable output with columns:
`FindingID`, `RiskLevel`, `CheckType`, `DocumentNumbers`, `Amount`, `Currency`, `Account`, `CostCenter`, `PostingDate`, `User`, `Description`, `Recommendation`

---

## Sample Output

```markdown
# SAP FI/CO Journal Entry Audit Report

| | |
|---|---|
| **Audit Date** | 2025-03-06 |
| **Entries Analyzed** | 32 |
| **Total Findings** | 15 |
| **Overall Risk** | 🔴 CRITICAL |

## Executive Summary

A total of 32 journal lines were analyzed. The audit identified 15 findings
(2 critical, 6 high, 5 medium, 2 low). High and critical findings require
immediate attention from the controller or internal audit team.

## Findings

| ID | Risk | Check Type | Document(s) | Amount | Account |
|---|---|---|---|---|---|
| FND-0001 | 🟠 HIGH | Duplicate Posting | 1800000001, 1800000002 | 15.000,00 EUR | 400000 |
| FND-0002 | 🔴 CRITICAL | Backdated Posting | 1800000032 | 18.500,00 EUR | 500000 |
...
```

---

## Running Tests

```bash
node tests/test.js
```

Expected output: **34 tests, all passing** ✅

The test suite validates:
- CSV parsing with SAP-format dates and amounts
- All 6 audit check types against realistic sample data (`sample-data/journal_march_2025.csv`)
- English and German memo generation
- CSV export format
- Period filtering

---

## Project Structure

```
sap-journal-auditor/
├── skill.json           # OpenClaw skill metadata
├── instructions.md      # Agent system prompt
├── index.js             # OpenClaw entrypoint
├── package.json
├── lib/
│   ├── parser.js        # CSV/Excel parser with SAP column normalization
│   ├── auditor.js       # All 6 audit check implementations
│   ├── reporter.js      # Markdown memo generator (EN/DE)
│   ├── exporter.js      # CSV findings export
│   ├── csv-parse-lite.js # Zero-dependency CSV parser fallback
│   └── xlsx-lite.js      # Zero-dependency XLSX reader fallback
├── sample-data/
│   ├── journal_march_2025.csv   # Realistic test data (32 SAP-style entries)
│   ├── sample_audit_memo.md     # Sample output memo
│   └── sample_flagged.csv       # Sample flagged entries
└── tests/
    └── test.js          # Full test suite (34 assertions)
```

---

## Configuration

| Parameter | Type | Default | Description |
|---|---|---|---|
| `language` | `en` / `de` | `en` | Output language for memo |
| `period` | `YYYY-MM` | all | Filter to specific fiscal period |
| `threshold_duplicate_amount` | number | `1000` | Min amount to flag duplicates |

---

## Requirements

- Node.js ≥ 16
- OpenClaw ≥ 0.9.0 (for messaging integration)
- No mandatory npm packages — zero-dependency fallbacks included

---

## Author

**Daryoosh** — Enterprise Data & AI Architect  
[RadarRoster](https://radarroster.com) · [GitHub: dda-oo](https://github.com/dda-oo)

> Built with deep knowledge of SAP FI/CO, S/4HANA Universal Journal (ACDOCA),  
> and hands-on controlling experience across DACH enterprise environments.

---

## License

MIT © 2025 RadarRoster

---

## Contributing

Issues and PRs welcome. If you work with SAP and have edge cases not covered by the current checks, please open an issue with an anonymized sample.

### Planned Features
- [ ] ACDOCA-native column mapping (direct S/4HANA extract)
- [ ] Profit Center dimension checks
- [ ] Intercompany reconciliation across two company codes
- [ ] HTML report output option
- [ ] Integration with SAP Analytics Cloud export format
=======
# sap-journal-auditor
OpenClaw skill: Audit SAP FI/CO journal entries for anomalies, duplicates &amp; compliance risks
>>>>>>> 801082df444436ece4f4b59dcf67151a2395b222
