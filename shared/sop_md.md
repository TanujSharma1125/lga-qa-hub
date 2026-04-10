# Compliance SOP: Document Service & Form Lifecycle Management

## 1. Purpose

To outline the end-to-end lifecycle of insurance forms (Applications, Disclosures, and Policy Contracts) ensuring alignment with carrier, product, and state-specific regulatory requirements through digital configuration and automated testing.

---

## 2. Scope

This SOP covers the Product Documents team's management of:

- State-specific Applications and Supporting Disclosures
- Carrier-agnostic forms (Buyer's Guides, Guarantee Association Notices, Replacement Notices)
- Authorizations (HIPAA, E-Sign, MIB, Release Transfer Authorization)
- Dynamic Policy Contract pages (including TPA-specific Premium/NFO calculations)
- DocuSign field mappings and Business Rule triggers

---

## 3. Roles and Responsibilities

| Role | Responsibility |
|------|---------------|
| **Product Documents Team** | Manages forms configuration in PCS, DocuSign mapping, and JSON template delivery to Engineering |
| **Insurance Team** | Provides approved/unbracketed forms; provides final sign-off on "Test" environment outputs |
| **Engineering** | Integrates JSON templates into the Interview Funnel, Final look, and Member Portal |
| **Carriers** | Provide final external sign-off on forms for new product onboarding and updates to existing forms such as rebranding, rate revisions, and latest forms updates |

---

## 4. Document Lifecycle & Integration Process

### 4.1 Implementation Workflow

- **Ingestion:** Receive approved, unbracketed production versions of forms from the Insurance team.
- **Configuration (PCS):** Store source links in the **Forms Config** within the Product Configuration System (PCS).
- **Mapping:** Build DocuSign field mappings (AOR, Medical & Financial Questions, etc.) and establish **Business Rules** for form triggering (e.g., Replacement triggers, Rider selections).
- **Engineering Hand-off:** Store JSON Template IDs in PCS and pass to Engineering for Funnel/Portal integration.

### 4.2 Testing & Quality Assurance

- **Internal Testing:** Verification of form generation logic against state/business rules and field-level accuracy against the Interview Funnel.
- **Automation:** Maintain an **Automation Regression Suite** in PCS to ensure 100% form presence against specific Product/State combinations.

---

## Detailed Testing Framework (Function & Test Areas)

To ensure regulatory alignment, the following test areas must be validated for every new product onboard or any feature added and quarterly review:

| Function / Test Area | Details / Notes |
|---|---|
| **End-to-End Funnel and Doc Testing** | Validate full flow from Interview Funnel to final PDF generation. |
| **Application Generation (PCS)** | Validate field mappings, Questions checkboxes, and labels. |
| **Policy Doc Generation (PCS)** | Validate correct state-specific forms generating and Business rules associated to trigger specific forms. Ensure Policy Spec pages printing correct calculations and summary. |
| **Authorizations** | Ensure Carrier Agnostic HIPAA matches to Legal Center and signed by Insured/Agent. |
| **Options Coverage** | Validate Lump Sum, Tax Test, and Death Benefit logic. |
| **Carrier HIPAA/Auth** | Ensure correct inclusion of carrier-specific authorization forms. |
| **Replacement Forms** | Trigger logic verification (Existing Coverage > 0) for NAIC states. |
| **State-Specific Forms** | Validate local mandates (e.g., Arbitration Disclosure in AL). |
| **Effective Dating** | Validate EFT form policy effective date rules. |
| **Business Rules** | Validate triggers for Riders, Disclosures, and specific conditions. |
| **Military Forms** | Confirm logic for military status applicants. |
| **E-sign Scenarios** | Validate all digital signature confirmation flows. |
| **Buyer's Guide** | Ensure carrier-agnostic documents are appended correctly. TruStage has their own carrier-specific Buyer's Guide. |

---

## 5. Ongoing Oversight & Process to Cure

### 5.1 Periodic Review (M&T Evidence)

To ensure ongoing compliance and address "M&T" (Monitoring & Testing) requirements:

- **Quarterly Product Docs Testing:** Conducted for certain products (LGA Prime, Ameritas Choice, IUL, Protective, TruStage) rotating through each quarter to address no bugs or impacts to current product docs with addition of new products and forms. There are 20–30 policies per product that we test covering:
  - State-specific forms
  - Forms associated with each channel and product variation
  - Business rules forms triggering
  - Carrier-agnostic forms which are used across other products
  
  All products will be tested at least once annually.

- **Insurance Team Sign-off:** All changes (New or Existing) require a documented sign-off from Ethos Insurance Team and the Carrier in the **TEST environment** prior to Production deployment.

### 5.2 Process to Cure (Failure Remediation)

In the event a form failure is identified (e.g., incorrect logic in the Funnel or missing state disclosure):

1. **Identification:** Failure logged via Automation Suite or Manual Audit.
2. **Correction:** If a Production error occurs, the team must log a P0 bug in the CM board to fix forward along with backfill. If required, update the PCS configuration or DocuSign mapping.
3. **Cure Execution:** For impacted customers, Compliance will determine if a "re-sign" is required or if a corrected disclosure must be sent via the LC comms.
4. **Validation:** The fix must pass the Regression Suite and receive an expedited Product team sign-off.

---

## 6. Documentation of Evidence

For Audit/M&T purposes, the following must be archived:

- **PCS Logs:** History of Form Config changes.
- **Regression Reports:** Results of the automated testing suites (Pass/Fail logs by State/Product).
- **Sign-off Artifacts:** Jira tickets showing Carrier and Compliance approval of the "TEST" environment before "PROD" implementation.
- **Calculation Validation:** For TPA products, evidence of Premium/NFO calculation testing.

> Be sure to include edit/approval log.

---

## Edit / Approval Log

| Review Date | Title Updates / Changes / Notes | Approver |
|-------------|----------------------------------|----------|
| | | |
| | | |
| | | |
| | | |
