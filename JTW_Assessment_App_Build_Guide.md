# JTW Consulting — Bicycle Assessment & Repair App
## Build Guide for Claude Code

## 1. Purpose

Build a single app that replaces JTW Consulting's current workbook-based workflow (separate Excel/Word files per job) with one connected system. Today, each bicycle job produces four related documents that are filled in manually and copy-pasted between: a **Damage Assessment**, a **Repair Quote**, a **Client Acceptance**, and **Warranty/Liability Terms**. The app should let JTW staff create a **Job**, fill in each document as a form, attach photos, and see every job's documents (assessment, quote, acceptance, warranty, invoice, handover) in one place, filterable by status.

This is a working tool for a small repair shop — prioritize simplicity, speed of data entry, and reliable calculations over polish. Single-user or small-team use is fine; no need for complex auth unless you judge it necessary.

## 2. Core Concept: the Job

Everything in the current spreadsheets is keyed off the same identifiers (Reference/Job No., Claim/Policy No., Owner/Custodian, bicycle identification). The app should make **Job** the central record, with every document type hanging off it, rather than treating each document as a standalone file. This is the main gap in the current workbook process — data is retyped across files instead of shared.

**Job fields (entered once, reused everywhere):**
- Reference / Job No. (unique, e.g. `JTW-2026-0088`)
- Assessment date
- Assessor name
- Location
- Owner / Custodian
- Reported by
- Purpose of assessment
- Insurer
- Claim / Policy No.
- Bicycle: Make, Model, Colour, Wheel size, Frame material, Frame size, Serial/Frame No., Bicycle type (road/MTB/hybrid/kids), Asset/Tag No.
- Job status (see Section 6)

Each Job can have one Damage Assessment, one or more Quotes (a job might be re-quoted), one Acceptance, one Warranty record, and later an Invoice and Handover doc.

## 3. Document Types & Their Fields

Build a form for each, mirroring the source workbooks. Pull shared fields (Job No., Claim No., bicycle ID, owner) from the parent Job automatically instead of re-entering them.

### 3.1 Damage Assessment
- **Component Condition Checklist**: repeatable rows grouped under fixed categories — Frame & Fork, Wheels & Tyres, Drivetrain, Brakes, Cockpit & Seating, Accessories & Other. Each row: Component name, Condition (dropdown: OK / Damaged / Missing), Recommendation (dropdown: Repairable / Replace, only relevant if not OK), Parts Available (dropdown: Yes / No), Notes. Allow adding custom rows beyond the standard checklist.
- **Overall photos**: 4 fixed slots — Front view, Rear view, Left side, Right side.
- **Damage photos**: repeatable — photo + "Affected part" caption per item (source has 6 slots but should be unlimited/repeatable in the app).
- **Assessor notes / observations**: free text.
- **Summary & sign-off**: Overall Rating (dropdown: Roadworthy/Good, Repairable, Uneconomical to Repair, Dispose/Scrap), Assessment notes/justification, Estimated Repair Cost, Estimated Repair Time, Assessor signature, Date.
- **Liability disclaimer**: fixed reference text (assessment is a point-in-time judgement, not a warranty, doesn't cover latent defects) — display as read-only boilerplate, not editable per job.

### 3.2 Repair Quote
- Repeatable line items: Component/Item, Work Description, Labour Hrs, Labour Rate (R), Labour Cost (auto), Parts Cost, Line Total (auto).
- Totals: Subtotal (auto = sum of line totals), Contingency % (editable, default 10%), Contingency amount (auto), VAT % (editable, default 15%), VAT amount (auto), Estimated Total (auto).
- A quote should be creatable directly from the assessment's checklist items marked Damaged/Missing, pre-filling the Component/Item column, to avoid retyping.

### 3.3 Client Acceptance
- Auto-pulled: Job No., Claim/Policy No., Owner, Insurer, bicycle make/model, Serial/Frame No., and the quoted totals (subtotal, contingency, VAT, total) — read-only, sourced live from the linked Quote.
- Fixed terms-of-acceptance text (client confirms they've read the assessment/quote, accepts the total including contingency, authorises work, understands variance beyond contingency needs written approval, and that warranty terms apply separately).
- Signature capture: client name + signature + date, JTW representative name + signature + date. (A simple typed-name + drawn/uploaded signature + timestamp is sufficient — no need for e-signature infrastructure unless requested.)

### 3.4 Warranty Terms
Two related but distinct warranty documents exist in the source material — keep them as separate templates the app can attach to a job:
- **Standard repair warranty summary** (general mechanical repairs): workmanship warranty period (default 6 months), parts warranty (per manufacturer), wheel truing/spoke tension warranty (default 3 months), what's covered/not covered, how to claim, response time.
- **Carbon frame structural repair warranty** (longer-form, from the Warranty & Liability Terms doc): lifetime warranty on the specific repaired area only, scope limitations, exclusions, claims process, limitation of liability, and the manufacturer-warranty-voiding notice. This is a heavier legal document — treat it as a template with bracketed fields ([Jurisdiction], [X] business days) that get filled per job, not something rebuilt from scratch each time.
- Both warranty types should clearly state they are separate from the assessment liability disclaimer (Section 3.1) — the assessment disclaims the assessment itself; the warranty covers repair workmanship only.

### 3.5 Invoice & Handover Doc (new — not in source files, but requested)
These weren't in the source workbooks, so design them consistently with the pattern above:
- **Invoice**: pulls Job No., Claim/Policy No., bicycle ID, and line items/totals from the accepted Quote; add Invoice No., Invoice date, Payment status (Unpaid/Partially Paid/Paid), Payment terms, Amount paid, Balance due.
- **Handover doc**: confirms return of the repaired bicycle to the client — Job No., date of handover, condition-on-return notes, confirmation the client was given the warranty summary, client signature, JTW representative signature.

## 4. Calculations (must match the source workbooks exactly)

- Labour Cost = Labour Hrs × Labour Rate
- Line Total = Labour Cost + Parts Cost
- Subtotal = sum of all Line Totals
- Contingency amount = Subtotal × Contingency %
- VAT amount = (Subtotal + Contingency amount) × VAT %
- Estimated Total = Subtotal + Contingency amount + VAT amount
- Contingency % and VAT % should be set once (defaults 10% and 15%) but editable per job/quote, and every dependent figure should recalculate live.

## 5. Photos

- Allow photo upload/attach at both the assessment level (overall + damage photos) and, optionally, per checklist row.
- Store a caption/"affected part" label with each damage photo.
- Photos should stay attached to the Job permanently and be viewable from the Job's page, not just embedded once in an export.

## 6. Categorization & Dashboard

Central list/dashboard view of all jobs, filterable and sortable by:
- Job No., Owner, Insurer, Claim/Policy No.
- Document status per job: Assessment (Draft/Complete), Quote (Draft/Sent/Accepted), Acceptance (Pending/Signed), Warranty (Not issued/Issued), Invoice (Unpaid/Paid), Handover (Pending/Complete)
- Overall job stage, e.g.: Assessment → Quoted → Accepted → In Repair → Invoiced → Handed Over
- A simple status badge/kanban or filterable table view is enough — no need for complex workflow automation, but the status should update sensibly (e.g. job can't move to "Handed Over" without a completed Invoice).

## 7. Export

- Each document (Assessment, Quote, Acceptance, Warranty, Invoice, Handover) should be exportable as a clean PDF for signing/sending, matching the layout/fields of the source spreadsheets.
- Signed Acceptance and Handover docs should be stored back against the Job once completed.

## 8. Non-functional notes

- Keep it simple: this is a small business tool, not a multi-tenant SaaS product. A local-first or lightweight single-database web app is appropriate unless you judge otherwise.
- Persist data reliably between sessions (a real database, not in-memory) since this holds ongoing job/financial records.
- No need for user-facing AI features — this is a forms + records + calculations + PDF export app.

## 9. Suggested build order

1. Job model + create/list/view Job.
2. Damage Assessment form (checklist, photos, sign-off) linked to a Job.
3. Repair Quote form with live calculations, optionally pre-filled from the assessment.
4. Acceptance page pulling live totals from the Quote, with signature capture.
5. Warranty template(s) attachable to a Job.
6. Invoice and Handover doc.
7. Dashboard with filtering/status across all jobs.
8. PDF export for each document type.

## Source material

This guide was derived from three existing JTW Consulting workbooks/documents used as the current manual process:
- `JTW_Bicycle_Damage_Assessment.xlsx`
- `JTW_Bicycle_Repair_Quote_and_Acceptance.xlsx`
- `Warranty_Liability_Terms_Draft.docx`

Field names, dropdown options, and calculation logic above are taken directly from these files so the new app produces equivalent output.
