# LGA Prime QA Testing Hub — SKILL.md

## Overview

This skill guides the AI in assisting with the **LGA Prime QA Testing Framework** — a compliance-driven
document testing process for insurance product packets issued by Banner Life Insurance Company and
administered by Ethos Technologies Inc.

When this skill is invoked, the AI must follow all sections below before performing any QA task.
This is a single source of truth combining: Live PDF Testing, Auth Packet Validation, State Form
Variants, Business Rules Testing, DocuSign Envelope Routing, and M&T Log Reporting.

---

## Product Reference

| Field                  | Value                                               |
|------------------------|-----------------------------------------------------|
| Product Name           | LGA Prime (Term Life Insurance)                     |
| Primary Term           | 30-year (10, 15, 20, 25, 30, 35, 40 yr available)  |
| Carrier                | Banner Life Insurance Company                       |
| Carrier Address        | 3275 Bennett Creek Avenue, Frederick, Maryland 21704|
| Carrier Phone          | (800) 638-8428                                      |
| Policy Administrator   | Ethos Technologies Inc.                             |
| Admin Address          | 1606 Headway Circle #9013, Austin, TX 78754         |

### Active Form Controls

| Form Control                  | Use Case                                        |
|-------------------------------|-------------------------------------------------|
| `LGA_Reprice_Prime`           | Primary reprice policy (ICC23-P contracts)      |
| `LGA_Price_Elasticity_Prime`  | Price elasticity policy (ICC24-P2 contracts)    |
| `Agnostic_Replacements_DTC`   | DTC channel replacement forms                   |
| `Agnostic_Replacements_AA`    | AA channel replacement forms                    |
| `Life`                        | Carrier-agnostic auth forms                     |

> ⚠️ Legacy controls `LGA_ER_DTC_v1` and `LGA_ER_DTC_v2` are NOT the current active controls.

---

## Section 1 — Live PDF Document Testing

### Purpose
Upload actual DocuSign packet PDFs and run AI-powered real-time pass/fail verification against the
LGA Prime PCS configuration.

### Test Configuration (Required Before Every Run)
Before analyzing any PDF, confirm and record:
- [ ] **State** — which US state the policy is issued in
- [ ] **Form Control** — `LGA_Reprice_Prime` or `LGA_Price_Elasticity_Prime`
- [ ] **Channel** — DTC or AA
- [ ] **Replacement** — YES or NO
- [ ] **RUW Policy** — YES or NO
- [ ] **Scenario ID** — SC-01 through SC-30 (optional but recommended)
- [ ] **Envelope ID** — from DocuSign
- [ ] **Tester Name** — for M&T log attribution

### Packet Types to Upload

| Packet             | Forms Included                                              |
|--------------------|-------------------------------------------------------------|
| Auth Packet        | E-Sign · HIPAA · FCRA · NOIP · TXT · MIB · RTA · RPLC B   |
| Application Packet | Application · Banner HIPAA · ADB Disc · Buyer's Guide       |
| Policy Packet      | Contract · ADB Rider · Perks · SOH · Delivery Receipt       |

### Critical Field-Check Rule (ALWAYS APPLY)
> ⚠️ **Field checks are RULES, not hardcoded expected values.**
> - NEVER assert "expected value = X"
> - ALWAYS verify each field matches the actual funnel response for that specific test applicant
> - Medical question responses (Q1–Q19) must each be verified individually — a mismatch is a contestability risk
> - Calculations (premiums, schedule sections) must be computed fresh from current test policy data

### Live Test Output
Each PDF analysis must produce:
1. **Pass/Fail table** — one row per form checked
2. **Flagged issues list** — with page, field, expected vs actual
3. **Auto-populated M&T log entry** — see Section 15

---

## Section 2 — DocuSign Envelope Routing

### Packet Order (CRITICAL — Never Deviate)

```
1 — AUTH Packet  →  2 — APPLICATION Packet  →  3 — POLICY Packet
```

- Auth packet **must appear FIRST** in the signing ceremony — no exceptions
- Application packet appears second
- Policy packet is generated **only after** the underwriting decision

### Envelope Routing Checks
- [ ] Correct form control name present in envelope metadata
- [ ] AUTH packet is first in signing order
- [ ] All required docs for the test state are present in each packet
- [ ] No extra or unexpected forms included

---

## Section 3 — Auth Packet Validation (Area 4)

### Rule: All 8 Forms Are ALWAYS Present — No Exceptions

| Form Name                        | Form Number          | Always Present? |
|----------------------------------|----------------------|-----------------|
| E-Sign Consent                   | 22-ETH ESIGN         | ✅ YES           |
| HIPAA Authorization              | 25-ETH HIPAA RTA     | ✅ YES           |
| Consumer Report Auth (FCRA)      | 22-ETH FRCA          | ✅ YES           |
| Notice of Insurance Practices    | 22-ETH NOIP          | ✅ YES           |
| Phone/Text/SMS Consent           | 22-ETH TXT           | ✅ YES           |
| MIB Notice & Authorization       | 24-ETH MIBN (10/24)  | ✅ YES           |
| HIPAA Release Transfer Auth      | 25-ETH RTA v.2       | ✅ YES           |
| Replacement Disclosure Part B    | 22-ETH RPLC B        | ✅ YES — unconditional |

> All forms must match Legal Center carrier-agnostic versions.

### Auth Packet Checks
- [ ] All 8 forms present
- [ ] Form numbers match exactly (check version/date codes)
- [ ] Auth packet appears before application and policy packets
- [ ] No duplicate forms

---

## Section 4 — Application Packet Validation (Area 2)

### Base Application Form Numbers by State

| State              | Application Form Number                              |
|--------------------|------------------------------------------------------|
| All compact states | ICC19-ETH Version 0519.1                             |
| CA                 | ETH-CA (7-19) + LU-1340-CA-A Fraud Statement         |
| ND                 | ETH-ND (7-19)                                        |
| DE                 | ETH-DE (7-19)                                        |
| SD                 | ETH-SD (7-19)                                        |
| FL                 | ETH-FL                                               |

### Application Field Mapping Checks
- [ ] All fields map exactly from the funnel response
- [ ] Medical questions Q1–Q19 verified individually
- [ ] No hardcoded expected values used
- [ ] No placeholder or default values remaining in fields

---

## Section 5 — Policy Packet Validation (Area 3)

### LGA_Reprice_Prime Contracts

| State          | Contract Form |
|----------------|---------------|
| Compact states | ICC23-P       |
| CA             | CA23-P        |
| FL             | FL23-P        |
| ND             | ND23-P        |
| SC             | SC23-P        |
| SD             | SD23-P        |

### LGA_Price_Elasticity_Prime Contracts

| State          | Contract Form |
|----------------|---------------|
| Compact states | ICC24-P2      |
| CA             | CA24-P2       |
| FL             | FL24-P2       |
| SC             | SC24-P2       |

### Policy Packet Checks
- [ ] Correct contract form for state + form control combination
- [ ] Schedule sections 2a/2b and 4a/4b tested both ways (see Section 10)
- [ ] ADB Rider present (always required — see Section 6)
- [ ] Perks Benefit included where applicable (CA and FL only; ND has ETHLP21-ND)
- [ ] Delivery Receipt present

---

## Section 6 — ADB Rider Validation (Area 5)

### Rule: ADB Rider is ALWAYS Included — No Scenario Where It Should Be Absent

| State(s)           | ADB Rider Form                                             |
|--------------------|------------------------------------------------------------|
| All compact states | ICC10 ADB                                                  |
| CA                 | ADB-CA (2-17) — title MUST read "due to TERMINAL ILLNESS"  |
| ND, SD             | ADB (06-10)                                                |
| FL                 | ADB-FL                                                     |

### ADB Checks
- [ ] ADB Rider present in policy packet
- [ ] ADB Disclosure present in application packet
- [ ] Correct state variant used
- [ ] CA title verified: must say "due to TERMINAL ILLNESS" exactly

---

## Section 7 — Carrier HIPAA Validation (Area 6)

### Rule: Banner HIPAA (ICC20 LU-1250) is ALWAYS in the Application Packet

| State(s)                           | Form Number             |
|------------------------------------|-------------------------|
| All compact states                 | ICC20 LU-1250 (5-20)    |
| CA (`LGA_Reprice_Prime`)           | LU-1250-CA (3-24)       |
| CA (`LGA_Price_Elasticity_Prime`)  | LU-1250-CA (4-23)       |
| DE, ND, SD                         | LU-1250 (5-20)          |
| FL                                 | LU-1250-FL (5-20)       |
| ME                                 | ICC20 LU-1250ME (5-20)  |
| VT                                 | ICC20 LU-1250VT (5-20)  |

### Carrier HIPAA Checks
- [ ] Banner HIPAA present in application packet
- [ ] Correct state variant used
- [ ] Carrier name, address, and phone verified exactly against product reference

---

## Section 8 — Replacement Forms (Area 7)

### 22-ETH RPLC B — ALWAYS in Auth Packet (Unconditional)

### State RPLC Forms — CONDITIONAL (Test Both Sides)

**DTC Channel — `Agnostic_Replacements_DTC`**

| Condition                                | Expected Result                                        |
|------------------------------------------|--------------------------------------------------------|
| `replacing-existing-insurance = yes`     | State RPLC C (or 21-ETH RPLC) form **MUST BE PRESENT** |
| `replacing-existing-insurance = no`      | State RPLC form **MUST NOT BE PRESENT**                |

**AA Channel — `Agnostic_Replacements_AA`**

| Condition              | Expected Result                     |
|------------------------|-------------------------------------|
| Coverage > 0 AND = YES | RPLC A form **MUST BE PRESENT**     |
| Coverage = 0 or NO     | RPLC A form **MUST NOT BE PRESENT** |

### Replacement Checks
- [ ] 22-ETH RPLC B always present in auth packet
- [ ] Conditional state RPLC form tested BOTH ways (present + absent)
- [ ] Correct channel (DTC vs AA) replacement logic applied

---

## Section 9 — State-Specific Form Variants (Area 8)

### California (CA)
- Application: `ETH-CA (7-19)`
- CA Fraud Statement: `LU-1340-CA-A`
- ADB Disc: `ADB DISC-CA`
- HIPAA: `LU-1250-CA (3-24)` [Reprice] / `LU-1250-CA (4-23)` [Price Elasticity]
- Contract: `CA23-P` / `CA24-P2`
- ADB Rider: `ADB-CA (2-17)` — title must say "due to TERMINAL ILLNESS"
- Perks Benefit: `21-ETH PERKS` — CA and FL ONLY

### Florida (FL)
- Application: `ETH-FL`
- ADB Disc: `ADB DISC FL`
- HIPAA: `LU-1250-FL (5-20)`
- Contract: `FL23-P` / `FL24-P2`
- ADB Rider: `ADB-FL`
- Perks Benefit: `21-ETH PERKS`

### North Dakota (ND) · South Dakota (SD) · Delaware (DE)
- Applications: `ETH-ND/SD/DE (7-19)`
- ADB Disc: `ADB DISC-1`
- HIPAA: `LU-1250 (5-20)`
- ND Contract: `ND23-P` · SD Contract: `SD23-P`
- ADB Rider ND/SD: `ADB (06-10)`
- ND Perks Rider: `ETHLP21-ND`

### Maine (ME) · New Jersey (NJ) · Vermont (VT) · Minnesota (MN)
- ME HIPAA: `ICC20 LU-1250ME (5-20)` · Endorsement: `LU1305 ME (9-11)` · Buyer's Guide: `21-ETH LIFE GUIDE ME`
- NJ Buyer's Guide: `21-ETH LIFE GUIDE NJ`
- VT HIPAA: `ICC20 LU-1250VT (5-20)`
- MN: `21-ETH GAN MN` must appear in **BOTH** auth AND application packets

### State Endorsements (Policy Packet)

| State | Endorsement Form         |
|-------|--------------------------|
| KS    | LU1257-KS                |
| ME    | LU1305 ME (9-11)         |
| MO    | LU1283-MO                |
| PA    | ICC13AMTPA + LU-1185-PA  |
| VA    | LU1306-VA                |
| WY    | LU1288-WY                |

---

## Section 10 — Business Rules Testing (Area 10)

### Rule: Test BOTH Sides of Every Conditional Rule

| Rule            | Trigger Condition                       | Form(s)        | Test Present             | Test Absent              |
|-----------------|-----------------------------------------|----------------|--------------------------|--------------------------|
| Schedule 2a     | Year ≤ `premium.total.37.policyYear`    | Section 2a     | Year = policyYear value  | Year > policyYear value  |
| Schedule 2b     | Year > same label                       | Section 2b     | Year > policyYear value  | Year ≤ policyYear value  |
| Schedule 4a/4b  | Year vs `premium.renewal.38.policyYear` | Sections 4a/4b | Year ≤ / > renewal label | Opposite condition       |
| RUW Amendment   | RUW policy                              | LU-14(54-62)   | Use RUW case             | Use STP case             |
| DTC Replacement | `replacing-existing-insurance = yes`    | State RPLC C   | replacing = yes          | replacing = no           |
| AA Replacement  | Coverage > 0 AND = YES                  | RPLC A         | Coverage > 0 + YES       | Coverage = 0 or NO       |
| MN GAN          | State = MN                              | 21-ETH GAN MN  | MN test case             | Non-MN state             |

### Business Rule Checks
- [ ] Both sides of every rule tested in the same test cycle
- [ ] No hardcoded values — always derive from funnel response
- [ ] Schedule section logic (2a/2b, 4a/4b) tested in both directions
- [ ] RUW amendment tested with a RUW case AND a standard STP case

---

## Section 11 — EFT Effective Dating (Area 9)

### Three In-Force Conditions — ALL Must Be Met Simultaneously

| # | Condition |
|---|-----------|
| ① | Policy delivered to and accepted by owner |
| ② | Full first premium paid while insured is alive at time of payment |
| ③ | No change in health or habits since application date |

### EFT Checks
- [ ] Effective date computed from actual test payment data — never hardcoded
- [ ] All three conditions verified simultaneously
- [ ] EFT date logic tested at boundary conditions (day before / day of / day after)

---

## Section 12 — Military Applicant Form Logic (Area 11)

- Military-specific forms must be verified when applicant is identified as active military
- Test both: military applicant present AND military applicant absent
- Confirm correct form triggers based on military status flag in funnel response

---

## Section 13 — DocuSign E-Sign Scenarios (Area 12)

- All e-sign scenario variants (remote, in-person, delegated) must be tested
- Signature field mapping must be verified against the DocuSign template
- Envelope routing order must be maintained across all e-sign modes
- Test both completed and incomplete signing scenarios

---

## Section 14 — Buyer's Guide (Area 13)

### Rules
- Buyer's Guide is **always included** — verify presence in every test run
- Buyer's Guide does **NOT require a signature** — verify it is a non-signature form
- State variants apply for ME and NJ (see Section 9)

| State      | Buyer's Guide Form        |
|------------|---------------------------|
| All others | Standard Buyer's Guide    |
| ME         | 21-ETH LIFE GUIDE ME      |
| NJ         | 21-ETH LIFE GUIDE NJ      |

---

## Section 15 — M&T Log Reporting

### Every Test Run Must Capture

| Field        | Description                                      |
|--------------|--------------------------------------------------|
| Envelope ID  | DocuSign envelope ID for the test packet         |
| Scenario ID  | SC-01 through SC-30                              |
| Tester Name  | Person running the test                          |
| State        | Test state                                       |
| Channel      | DTC or AA                                        |
| Form Control | Active form control used                         |
| Result       | PASS / FAIL / BLOCKED                            |
| Jira Ticket  | Required for every FAIL before PROD promotion    |
| Notes        | Any relevant observations                        |

### M&T Log Rules
- Results from live PDF testing auto-populate the M&T log
- CSV export must be available for every completed test cycle
- A **Jira ticket number is required for every FAIL** before production promotion
- BLOCKED status is used when a prerequisite is missing (e.g. missing golden file, env issue)

---

## Section 16 — Compliance Sign-Off Gates

| Gate | Requirement | When Required |
|------|-------------|---------------|
| ① Insurance Team Sign-Off | Required for ALL changes without exception | ALWAYS |
| ② Carrier Sign-Off | Required for new or updated form versions | NEW/UPDATED FORMS |
| ③ Production Error Fix | Must pass full Regression Suite + expedited Product sign-off | PROD ERRORS |
| ④ Jira Ticket for FAIL | Required before any PROD promotion | EVERY FAIL |

---

## Section 17 — Do's and Don'ts

### ✅ Do
- Always confirm state, form control, channel, and replacement scenario before starting
- Test BOTH sides of every conditional rule (form present AND form absent)
- Verify medical questions Q1–Q19 individually — never bulk-pass them
- Check carrier name, address, and phone exactly against the product reference
- Log every FAIL with a Jira ticket number before marking the cycle complete
- Re-verify state-specific form variants using Section 9 for every non-compact state test
- Treat field checks as rules — derive values from actual funnel response every time

### ❌ Don't
- Never hardcode expected values for any field
- Never skip the Auth packet order check — it must always be first
- Never assume ADB Rider is absent — it is always required
- Never mark a FAIL as PASS without a documented fix and re-test
- Never use legacy form controls `LGA_ER_DTC_v1` or `LGA_ER_DTC_v2`
- Never promote to production with an open FAIL and no Jira ticket
- Never test only one side of a conditional rule

---

## Quick Reference — Full Test Checklist

```
PRE-TEST
  [ ] State confirmed
  [ ] Form control confirmed
  [ ] Channel confirmed (DTC / AA)
  [ ] Replacement scenario confirmed (YES / NO)
  [ ] RUW status confirmed (YES / NO)
  [ ] Scenario ID assigned
  [ ] Envelope ID recorded
  [ ] Tester name logged

PACKET VALIDATION
  [ ] Auth packet — all 8 forms present
  [ ] Auth packet — appears FIRST in envelope
  [ ] Application packet — fields match funnel response
  [ ] Application packet — medical Q1–Q19 verified individually
  [ ] Policy packet — correct contract for state + form control
  [ ] ADB Rider — correct state variant, always present
  [ ] Buyer's Guide — present, no signature required
  [ ] Banner HIPAA — correct state variant present

STATE & RULES
  [ ] State-specific form variants applied (Section 9)
  [ ] State endorsements in policy packet where applicable
  [ ] Replacement logic tested both ways (Section 8)
  [ ] Business rules tested both ways (Section 10)
  [ ] Schedule sections 2a/2b and 4a/4b tested both ways

POST-TEST
  [ ] M&T log entry completed
  [ ] All FAILs have Jira ticket numbers
  [ ] CSV export saved
  [ ] Compliance sign-off gates met
```

---

*Skill version: 1.0 | Last updated: April 2026 | Product: LGA Prime | Hub version: v2.1*
