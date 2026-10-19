# EJF_dataprivacy
# Crew Testimony Analysis — privacy-first prototype

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR-USERNAME/YOUR-REPO/blob/main/crew_testimony_demo.ipynb)

> **Dummy data only.** All 1,000 interviews are synthetic. Real flag states and ports appear only so the demo reads realistically; no figure is a real finding.

A working prototype of a system for analysing crew testimony about IUU fishing and forced labour **without exposing the people who gave it**. Staff can ask questions in plain English, explore dashboards and export aggregates, while the system enforces consent, small-cell suppression, role-based access and a full audit trail.

## Run it

1. Click **Open in Colab** above (after replacing `YOUR-USERNAME/YOUR-REPO`).
2. `Runtime → Change runtime type → T4 GPU`.
3. `Runtime → Run all`. The app opens in the last cell (first run downloads a ~3 GB model, about 2–3 minutes).

No GPU? Set `USE_LLM = False` in the configuration cell. Everything works; answers come from templates instead of the model.

## Demo logins

| User | Role | Password | Sees |
|---|---|---|---|
| Dana / Sam | Data custodian | `custodian1` / `custodian2` | Everything except identities · settings · audit log · ingestion · approvals |
| Ivo | Investigator | `investigator` | Port- and month-level aggregates · redacted records · can request identity files |
| Ana | Analyst | `analyst` | Region/year aggregates and themes · interview IDs only |
| Cam | Comms viewer | `viewer` | Aggregate figures only |

## Things to try

1. As **Ivo**, ask the three example questions. The third is withheld (fewer than *k* people) and automatically broadened to the region.
2. Ask *"Which crew members on Taiwanese-flagged vessels were in Port Louis in 2021?"* → refused and logged.
3. Request an identity file for `INT-0042` in **Identity vault**. Sign in as **Sam** to approve it (you can't approve your own).
4. As **Dana**, raise *k* to 10 in **Privacy settings** and re-ask question 1: more years disappear. Turn on differential-privacy noise.
5. As **Dana**, paste a transcript in **Ingest & review**, run redaction, tick *Other* consent, submit. Sign in as **Sam** to review it.
6. Check the **Audit log** and the **energy meter** beside the chat.

## Privacy design

- **Identities are never digitised.** The system holds only a vault reference to a locked physical file; release needs a reason and a second person.
- **Standardised consent** recorded at ingestion (aggregate analysis, internal paraphrase, public quotation, partner sharing, legal use, withdrawn, or *Other: reason*, which is held back until a second custodian reviews it).
- **Small-cell suppression** with adjustable *k*; totals are rounded when a cell is hidden so it can't be recovered by subtraction.
- **Generalisation levels** (port/month → region/year → 3-year bands); analysts and viewers never get below region/year.
- **Optional differential privacy** (Laplace noise, adjustable ε).
- **Deterministic query parsing** turns questions into filters, so the privacy rules can't be talked around; singling-out questions are refused.
- **The LLM sits after the privacy layer.** It only sees facts that already passed suppression, runs locally (no data leaves the runtime), and any answer containing a number not in those facts is discarded.
- **Accountability:** purpose statement on every query, full audit log, watermarked aggregate-only exports.

## Production path

Swap the stand-ins for: Microsoft Presidio + human review (redaction), PostgreSQL (store), SSO + MFA (login), a self-hosted model on EJF-controlled hardware (chat), and persistent, tamper-evident audit logging. Colab is fine for a demo with dummy data but **not** for real testimony.
