# Doc Testing Hub — Claude Code Instructions

## Project purpose
This project builds and maintains QA testing documentation for Ethos insurance
product documents. It extracts form data from PCS configuration files and
individual form PDFs, then generates a QA Testing Hub (HTML artifact) that a
QA tester with no insurance background can use to test document packets before
any PROD deployment.

---

## Compliance SOP — core rules (always apply)

These rules come from the Compliance SOP and govern everything built in this
project. Apply them to every artifact generated.

### Testing framework — 13 SOP test areas
Every product hub must cover all 13 areas:
1. End-to-end funnel and doc testing
2. Application generation — field mappings, checkboxes, labels
3. Policy doc generation — state-specific forms, business rules, calculations
4. Authorizations — carrier-agnostic HIPAA matches Legal Center, signed by insured/agent
5. Options coverage — Lump Sum, Tax Test, Death Benefit logic
6. Carrier HIPAA/Auth — correct carrier-specific authorization forms
7. Replacement forms — trigger logic (replacing-existing-insurance = yes)
8. State-specific forms — local mandates per state
9. Effective dating — EFT form policy effective date rules
10. Business rules — triggers for Riders, Disclosures, specific conditions
11. Military forms — logic for military status applicants
12. E-sign scenarios — all digital signature confirmation flows
13. Buyer's Guide — carrier-agnostic documents appended correctly

### Business rule testing requirement
Every conditional form must be tested BOTH ways:
- Condition triggered → form must be PRESENT
- Condition not triggered → form must be ABSENT
Both results must be logged as separate test runs in the M&T log.

### M&T evidence requirements (quarterly)
Per SOP Section 5, every test run must produce:
- Pass/Fail log per State/Product
- Insurance Team sign-off documented
- Carrier sign-off documented (for new or updated forms)
- Jira ticket number for any FAIL before PROD promotion

### Sign-off gates — mandatory before PROD
- All changes: Insurance Team sign-off required
- New forms or form updates: Carrier sign-off required
- Fix for a production error: Must pass Regression Suite + expedited Product team sign-off

### Field check rules — critical
- Field checks are RULES, not values. Never hardcode expected values from a
  previous test run.
- Fields sourced from "funnel" must be verified against actual applicant funnel
  responses — never assume standard answers.
- Medical question responses must be verified individually — mismatch is a
  contestability risk.
- Calculations must be computed fresh from current test policy data.

### Form status
- Only include forms with status "Active" (Pending = active) in test checklists
- Forms marked "Obsolete" must be excluded unless explicitly noted otherwise

---

## Project file structure

```
/doc-testing-hub/
  CLAUDE.md                    ← this file — loaded automatically
  /shared/
    /forms/                    ← canonical location for carrier-agnostic PDFs shared
    |                             across ALL products (43 files):
    |                             Authorizations: 22-ETH E-SIGN, HIPAA, FCRA, NOIP, TXT,
    |                               MIBN, RTA
    |                             Replacement: 22-ETH RPLC B, 21-ETH RPLC A (state variants),
    |                               21-ETH RPLC (state variants), 24-ETH RPLC C (state variants)
    |                             Buyer's Guide: NAIC, ME, NJ
    glossary.md
    sop_md.md
  /lga-prime/
    pcs_lga.csv                ← LGA-relevant rows from Forms sheet of PCS
    /forms/                    ← LGA-specific form PDFs (product-specific only)
    /output/
  /[next-product]/             ← add new product folders here
    pcs_[product].csv
    /forms/                    ← product-specific PDFs only; shared forms live in /shared/forms/
    /output/
  /shared/
    sop.docx                   ← Compliance SOP (source: Downloads)
    glossary.md                ← shared insurance terms
```

### PCS source file
The full multi-sheet Excel source is:
`~/Downloads/Copy of Ethos Product Configuration Tables Working Document v2.0.xlsx`

To re-export `pcs_lga.csv` (Forms sheet, LGA rows only):
```python
python3 -c "
import openpyxl, csv
from datetime import datetime
wb = openpyxl.load_workbook('~/Downloads/Copy of Ethos Product Configuration Tables Working Document v2.0.xlsx', read_only=True, data_only=True)
# ... filter Forms sheet for LGA controls ...
"
```
See conversation history for full export script.

---

## How to build a product hub

When asked to build or update a hub for a product:

1. Read the PCS CSV for that product — extract all rows where status = Pending/Active/Implemented
2. Group rows by: combined doc assignment (Policy/App/Auth), state, form type
3. For each form with a DocuSign link, note the form key/UUID
4. Read individual form PDFs where available — extract field names and
   signature requirements
5. Build the hub covering all 13 SOP test areas
6. Include a scenario matrix with minimum 20 scenarios covering all business
   rule combinations (ADB, RUW, replacement, military, EFT, e-sign variants,
   key state variants)
7. Include M&T log with fields: date, envelope ID, tester, scenario ID, state,
   result, Jira ticket, insurance sign-off, carrier sign-off, PROD promoted,
   notes
8. Output as a single self-contained HTML file

---

## How to update an existing hub

When a form changes, a new state is added, or rates are updated:
1. Identify which PCS rows changed
2. Update only the affected FORMS array entries or FIELD_CHECKS
3. Bump the version number (e.g. v1.1 → v1.2)
4. Update the last-updated date
5. Add an entry to the changelog at the bottom of the HTML file

---

## Source of truth hierarchy

When there is a conflict between sources, resolve in this order:
1. Carrier-approved form (highest authority)
2. PCS configuration
3. Compliance SOP
4. Prior hub version

---

## Products currently configured

All products are consolidated into a single unified hub: `/doc-testing-hub/output/ethos_qa_hub.html`

| Product | Carrier | Type | Form Control | PCS file | Status |
|---------|---------|------|--------------|----------|--------|
| LGA Prime | Banner Life | 30-yr Term | LGA_Reprice_Prime, LGA_Price_Elasticity_Prime | lga-prime/pcs_lga.csv | Active |
| Ameritas Protection IUL | Ameritas | Universal Life (IUL) | Ethos_IUL_v1 | ameritas-iul/pcs_ameritas_iul.csv | Active |
| SFG / North American Accumulation IUL | North American | Universal Life (IUL) | IUL_ACCUM_v1 | sfg-iul/pcs_sfg_iul.csv | Active |
| Banner GIWL (Final Expense) | Banner Life | Guaranteed Issue WL | Banner_FEGI | banner-fegi/pcs_banner_fegi.csv | Active |
| Banner SIWL (Final Expense) | Banner Life | Simplified Issue WL | Banner_FESI | banner-fesi/pcs_banner_fesi.csv | Active |
| Ameritas Choice Term | Ameritas | Term Life | Ameritas_Choice_v1 | ameritas-choice/pcs_ameritas_choice.csv | Active |

Add new products to this table when onboarded. Re-export PCS CSV and regenerate `ethos_qa_hub.html` using `/tmp/build_hub.py` + `/tmp/gen_hub.py`.

---

## Changelog

| Version | Date | Changed by | Summary |
|---------|------|-----------|---------|
| v1.0 | 2026-04-02 | Product Documents Team | Initial LGA Prime hub |
| v2.0 | 2026-04-08 | Product Documents Team | Unified multi-product hub — added Ameritas IUL, SFG IUL, Banner GIWL, Banner SIWL, Ameritas Choice. Single ethos_qa_hub.html with product selector. |
