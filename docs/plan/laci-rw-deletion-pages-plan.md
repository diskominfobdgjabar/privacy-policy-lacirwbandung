---
goal: "Publish Laci RW Bandung Deletion Request Page + upgraded Privacy Policy Page (Play Store compliance fix)"
version: "1.2.0"
date_created: "2026-10-16"
last_updated: "2026-10-19"
status: "In Progress" # Planned | In Progress | Completed | Paused
spec_ref: "docs/spec/laci-rw-deletion-pages-spec.md"
tags: ["polya", "plan", "tracer-bullets", "vertical-slicing", "static-site"]
---
<!-- markdownlint-disable -->

# Implementation Plan: Laci RW Bandung Public Deletion Pages (Play Store Compliance Fix)

> [!IMPORTANT]
> **THE PAUSE RULE & VERTICAL SLICING MANDATE:**
> All tasks in this plan are organized into **Vertical Tracer Bullets** — each
> slice ends in a published-and-verifiable state (artifact → commit → deploy →
> live HTTP verification). Horizontal layer-by-layer slicing is strictly
> prohibited. After each phase is verified, execution MUST halt for explicit
> user approval before proceeding to the next phase. Functional code MUST NOT
> be written until this plan is approved.

**Revision 1.1.0 note:** task matrix decomposed per operator request — the
oversized `index.html` authoring task is split into two Play-critical-first
sub-tasks, verification is split into HTTP/live vs parity/provenance suites,
and the AC-007 negative-gate simulation is scheduled explicitly in both
verification phases. Traceability coverage is now exhaustive (see §3.4).

---

## 1. Overview & High-Level Mental Model

Two vertical slices turn the approved spec (v1.5.0) into two published GitHub
Pages resources: the **Deletion Request Page** (`account-deletion.html`, new)
and the upgraded **Privacy Policy Page** (`index.html`), plus the site glossary
seed. Each slice is derived by copying the frozen JDIH reference design
(§4.3 Tables A–C) and applying only the closed substitution set (Table D) and
copy provenance anchors (Table E) — then verified live against the spec's
deterministic AC suite before the operator fills the Play Console form.

### Topological Data Flow Diagram

```text
[Operator git push (main)] ──▶ [GitHub Pages deploy] ──▶ [Raw static DOM]
                                                                 │
                              ┌──────────────────────────────────┤
                              ▼                                  ▼
                  [index.html (PrivacyPolicyPage)]   [account-deletion.html
                   links ─▶ account-deletion.html     (DeletionRequestPage)]
                   mailto ─▶ android@diskominfo...    mailto CTA ─▶ D2 email
                                                                 │
                                                                 ▼
                                    [Play Console verifier: HTTP 200 + content]
                                                 │
                                                 ▼
                              [Operator fills data-safety URLs (OC-4/OC-5)]
```

---

## 2. 📏 Task Sizing Matrix

| Size | Files Impacted | Scope | Example |
| :--- | :------------: | :---- | :------ |
| **XS** | 1 | Single config/seed change | CONTEXT.md glossary seed |
| **S** | 1–2 | Focused page slice | One HTML artifact / region group per spec contract |
| **M** | 3–5 | Multi-file vertical slice | Page + assets + deploy + verify |
| **L** | 5–8 | Multi-component flow | n/a in this plan |
| **XL** | 8+ | **Prohibited** | n/a |

---

## 3. Phased Implementation Sequence (Vertical Tracer Bullets)

> **EXECUTION DIRECTIVE FOR AI AGENTS:**
> Execute phase by phase. Run the verification tasks at the end of each phase.
> After verification passes, **STOP AND WAIT** for user approval before moving
> to the next phase. Git commits/pushes happen only with explicit operator
> confirmation at the commit step.

### 3.1 Implementation Phase 1: Land — Deletion Request Page Tracer Bullet
*Goal: the artifact that directly cures both Play Console issues is published,
verified live — both positively (HTTP/content) and negatively (the gate must
be able to fail hard) — and design/copy contract-compliant.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-001 | Seed site glossary: create root `CONTEXT.md` with the four canonical terms + `_Avoid_` lists exactly as staged in spec §2 | SPEC §2 glossary mandate | — | - | 1 (XS) | [x] | |
| TASK-002 | Create `account-deletion.html` — full DeletionRequestPage per spec §4.2 Artifact 1 (six ordered sections), design copied byte-for-byte from JDIH reference applying §4.3 Table D substitutions + Table E anchors (D1–D5, D7 identity, REQ-003 mailto endpoint with `%20`-encoded subject, zero `<script>`, no build tooling) | REQ-001, REQ-002, REQ-003, REQ-008, REQ-009, REQ-010, CON-003, CON-004, CON-006 | AC-001, AC-003, AC-009, AC-010 | TASK-001 | 1 (S) | [x] | |
| TASK-003 | **COMMIT & PUSH (with operator confirmation):** one changeset = `account-deletion.html` + `assets/diskominfo_kota.png` + `assets/ic_profile.png` + `CONTEXT.md` → `main`; wait for GitHub Pages redeploy (CON-005 atomicity; `assets/` is currently untracked) | REQ-008, CON-005 | AC-004 | TASK-002 | 3 (S) | [x] | 2026-10-19 |
| TASK-004 | **VERIFY-1a (HTTP / live):** plain GET, no cookies/JS — `/account-deletion.html` returns exactly 200 with `<main>` + H1 + literal email (AC-001); both asset URLs return 200 non-zero-length (AC-004); **AC-007 negative simulation — fetch `Account-Deletion.html` (wrong case) expecting 404 and record the gate's hard-failure verdict** (proving the gate cannot silently pass a corrupt state) | CON-001, CON-002 | AC-001, AC-004, AC-007 | TASK-003 | — | [x] | 2026-10-19 |
| TASK-005 | **VERIFY-1b (Parity / Provenance audit):** raw-HTML byte-diff of `<head>` pins, CSS tokens, and skeleton classes vs the JDIH reference (§4.3 Tables A–C, delta window = Table D only) (AC-009); Table E anchor audit incl. zero transplanted factual claims (AC-010); deletion-side coverage of AC-003 (D5 class-naming + D3 field list present) | REQ-009, REQ-010 | AC-003, AC-009, AC-010 | TASK-004 | — | [x] | 2026-10-19 |
| TASK-006 | **APPROVAL:** 🛑 Stop and wait for explicit operator confirmation to proceed to Phase 2 | — | — | TASK-005 | — | [x] | 2026-10-19 |

### 3.2 Implementation Phase 2: Expand — Upgraded Privacy Policy Page Tracer Bullet
*Goal: the policy page is upgraded to the JDIH-grade structure with Laci RW
copy, dual deletion CTAs, and the full D6 data fidelity — authored in two
Play-critical-first sub-tasks, deployed atomically, and verified in full.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-007 | Author `index.html` — **Play-critical regions first** (§4.2 Artifact 2 rows 1–3 + 10): hero (emblem, eyebrow, H1, lead, dual CTAs per REQ-003/004), "Quick Summary" panel, "Account Deletion Request" section, "Account Deletion Request Procedure" (D3 fields, ≤ 14 business days, permanence warning); design per §4.3 Tables A–C; zero `<script>` | REQ-003, REQ-004, REQ-005, REQ-009, CON-002, CON-004, CON-006 | AC-002, AC-003 | TASK-006 | 1 (S) | [x] | |
| TASK-008 | Author `index.html` — **remaining policy regions** (§4.2 Artifact 2 rows 4–9, 11–16 + footer): Information Collection and Use; Third-Party Services (7 providers — exact existing-page order and hrefs per Table E); Log Data and Cookies; Disclosure of Information + Service Providers; Opt-Out Rights; Data Retention (soft phrasing pinned per clarification Item 4 Option A); Children's Privacy; Security; Links to Other Sites; Changes to This Privacy Policy (**effective date = deploy date**, ASSUMPTION-004 Option B); Your Consent; Contact Us (mailto D2 + Diskominfo attribution); footer `<small>` — generator attribution retired | REQ-004, REQ-006, REQ-007, REQ-009, REQ-010, CON-003 | AC-002, AC-008, AC-010 | TASK-007 | 1 (S — second half of the same page) | [x] | |
| TASK-009 | **COMMIT & PUSH (with operator confirmation):** `index.html` → `main`; wait for redeploy | CON-005 | AC-002 | TASK-008 | 1 (XS) | [x] | 2026-10-19 |
| TASK-010 | **VERIFY-2a (HTTP / live suite):** plain GET, no cookies/JS — site root `/` returns exactly 200 with H1 + `account-deletion.html` link + mailto link + deletion section headings + `<small>DISKOMINFO KOTA BANDUNG</small>` footer and no generator attribution (AC-002); full section manifests present with JS disabled (AC-006); dual-issue coverage across both pages: D5 data-class naming + D3 field lists (AC-003); **AC-007 re-run — deliberate stale-state simulation** (fetch the pre-phase-1 404 shape of a temporarily broken path, confirm hard-failure verdict) | REQ-002, REQ-004, REQ-005, CON-001, CON-002 | AC-002, AC-003, AC-006, AC-007 | TASK-009 | — | [x] | 2026-10-19 |
| TASK-011 | **VERIFY-2b (Fidelity / Parity suite):** every external link carries `target="_blank"` + `rel="noopener noreferrer"`; every mailto query `%20`-encoded with no raw spaces/newlines (AC-005); provider fidelity — 7 names, exact order, exact hrefs (AC-008); byte-parity diff of both pages vs reference Tables A–C (AC-009); full Table E provenance audit on both pages — no transplanted JDIH factual claims (AC-010) | REQ-006, REQ-010, CON-004, CON-006 | AC-005, AC-008, AC-009, AC-010 | TASK-010 | — | [x] | 2026-10-19 |
| TASK-012 | **APPROVAL:** 🛑 Stop and wait for explicit operator confirmation to declare the build complete and hand over to the operator-side checklist | — | — | TASK-011 | — | [ ] | |

### 3.3 Implementation Phase 3: Handover — Operator-Side Play Console Steps
*Goal: the submission blocker is cleared. These are operator actions (spec §9);
the coding agent assists with verification only.*

| Task | Description | Ref ID | AC Ref | Dep | Files | Completed | Date |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| TASK-013 | **OPERATOR:** fill Play Console data safety form with the two verified URLs — site root `/` and `/account-deletion.html` — exactly, case-sensitively (also align store data-safety categories with D6 while there: the listing currently reads "No data collected") | ASSUMPTION-001, CON-001 | AC-001, AC-002 | TASK-012 | — | [ ] | |
| TASK-014 | **OPERATOR:** resubmit the update in Play Console; if the deletion issues re-appear, re-run TASK-010 live checks before any content edits | SPEC §9 OC-5 | — | TASK-013 | — | [ ] | |

### 3.4 Spec ↔ Plan Traceability Coverage Matrix (Zero-Orphan Proof)

| Spec element (v1.5.0) | Covered by plan tasks |
| :-------------------- | :-------------------- |
| REQ-001 / REQ-002 / REQ-003 | TASK-002, TASK-004, TASK-005 |
| REQ-004 / REQ-005 / REQ-006 / REQ-007 | TASK-007, TASK-008, TASK-010, TASK-011 |
| REQ-008 | TASK-002 (head pins), TASK-003 (assets committed), TASK-004 (AC-004) |
| REQ-009 / REQ-010 | TASK-002, TASK-005, TASK-007, TASK-008, TASK-011 |
| CON-001 / CON-002 | TASK-004, TASK-010 |
| CON-003 | TASK-002, TASK-008 |
| CON-004 | TASK-002, TASK-007, TASK-011 |
| CON-005 | TASK-003, TASK-009 |
| CON-006 | TASK-002, TASK-007, TASK-011 |
| AC-001 | TASK-004 (also TASK-013 evidence) |
| AC-002 | TASK-007, TASK-010 (also TASK-013 evidence) |
| AC-003 | TASK-005, TASK-010 |
| AC-004 | TASK-003, TASK-004 |
| AC-005 | TASK-011 |
| AC-006 | TASK-010 |
| AC-007 | TASK-004, TASK-010 (explicit negative simulations) |
| AC-008 | TASK-011 |
| AC-009 | TASK-005, TASK-011 |
| AC-010 | TASK-005, TASK-011 |
| SPEC §2 glossary mandate | TASK-001 |
| SPEC §9 OC-1…OC-5 | TASK-003/009 (OC-1/OC-2), TASK-004/010 (OC-3), TASK-013 (OC-4), TASK-014 (OC-5) |

Every REQ-001…010, CON-001…006, and AC-001…010 is consumed — no orphans, no
drift.

---

## 4. Contingency Plan B (Have Two Strings to Your Bow)

- **Risk / Failure Point 1:** GitHub Pages redeploy stalls or serves a cached
  404 after push (CDN propagation lag).
  - **Primary Strategy (Plan A):** push to `main`, wait for the Pages build to
    complete, then verify (TASK-004/TASK-010).
  - **Contingency Strategy (Plan B):** inspect the repo's Pages build status
    (Actions/Settings → Pages); re-trigger with an empty commit
    (`git commit --allow-empty -m "chore: retrigger Pages build"`); re-run
    verification after propagation.
  - **Trigger Condition:** AC-001/AC-002 returns non-200 more than one
    propagation cycle after a green push.
- **Risk / Failure Point 2:** Subresource integrity mismatch or CDN outage
  breaks styling (page returns 200 but renders unstyled).
  - **Primary Strategy (Plan A):** byte-for-byte copy of the reference
    `<head>` pins (§4.3 Table A) — the reference stack is live-proven.
  - **Contingency Strategy (Plan B):** verify the pinned Bootstrap integrity
    hash against jsdelivr's published SRI for 5.3.8; if jsdelivr itself is
    down at verification time, defer AC-009 parity verdict until CDN recovers
    (visual parity is judged on markup, not CDN availability).
  - **Trigger Condition:** console/SRI error observed during verification, or
    the parity diff flags the `<head>` block.
- **Risk / Failure Point 3:** Play Console re-reports the deletion issues
  after resubmission.
  - **Primary Strategy (Plan A):** operator fills both form fields with the
    two URLs verified in TASK-010 (TASK-013).
  - **Contingency Strategy (Plan B):** re-run the AC-001…010 suite live; if
    all green, the declaration URL in the form mismatches the verified path —
    read the exact URLs back from the form and align (case-sensitivity per
    §7 URL dimension); only then edit content.
  - **Trigger Condition:** TASK-014 outcome repeats either Play issue.
- **Risk / Failure Point 4:** The two-half authoring of `index.html`
  (TASK-007/008) produces an internally inconsistent page (e.g., anchors or
  classes drift between halves).
  - **Primary Strategy (Plan A):** TASK-007 defines the full doc skeleton
    (head + footer placeholder) in sub-task one so TASK-008 only fills
    regions — never re-structures.
  - **Contingency Strategy (Plan B):** if TASK-011 parity/provenance audit
    flags drift, redo TASK-008 against the TASK-007 skeleton rather than
    patching in place; re-commit once green.
  - **Trigger Condition:** AC-009/AC-010 failure attributable to inter-region
    drift.

---

## 5. Symmetry, Invariants & Problem Variation (Pólya, p. 199–214)

- **Symmetric Operations & Round-Trip Invertibility ($f^{-1}(f(x)) = x$):**
  - `index.html → account-deletion.html` link $\leftrightarrow$
    `account-deletion.html → index.html` backlink (REQ-002 hero CTA /
    REQ-004/005 cross-links) — both directions verified in TASK-004/010/011.
  - `mailto:` subject `%20`-encoding $\leftrightarrow$ mailto client
    decoding — encoding audited in AC-005 (TASK-011).
  - `subscribe`-style pairing n/a (no runtime state).
- **Variation of the Problem (Property-Based & Boundary Exploration):**
  - *URL variations:* correct path, wrong case (`Account-Deletion.html`),
    trailing-slash variant, with/without `assets/` — all bounded by AC-001's
    exact-path comparison and the AC-007 negative simulations in
    TASK-004/TASK-010.
  - *Fetch variations:* cookies/JS enabled vs disabled (AC-006), CDN warm vs
    cold, first-load vs cached — content must be identical raw DOM (CON-002).
- **SOLID Checks at Blueprint Stage:**
  - *SRP:* one concern per file — `index.html` presents policy + cross-links;
    `account-deletion.html` presents the deletion flow; `CONTEXT.md` holds
    only glossary.
  - *DIP:* both pages depend outward only on CDN assets and the `mailto:`
    protocol; contact identity is a single seam (D2) per page.

---

## 6. Risks & Extracted Assumptions

*Extracted from upstream Spec §3 (all four closed during `/to-clarify` —
recorded here with their standing mitigations):*

- **[ASSUMPTION-001]** RESOLVED (operator): Play data-safety form has no URLs
  yet; both fields will be set to site root `/` and `/account-deletion.html`.
  — *Mitigation:* exact filenames are hard requirements (TASK-002/TASK-007–008);
  OC-4 (TASK-013) fills the form post-verification.
- **[ASSUMPTION-002]** RESOLVED (verified-by-precedent + Play listing reality
  check): email-based deletion mechanism to D2; no in-app deletion screen in
  scope. — *Mitigation:* copy per Table E; optional future line only if the
  app ships a self-service flow.
- **[ASSUMPTION-003]** RESOLVED (verified live): GitHub Pages from `main`, no
  custom domain. — *Mitigation:* Phase 1/2 deploy steps target `main` only.
- **[ASSUMPTION-004]** RESOLVED (operator, Option B): effective date =
  deploy/revision date. — *Mitigation:* TASK-008 renders
  "This policy is effective as of {deploy date}" (Table D row 3 / E7).
- **[RISK-001]:** `assets/` is currently untracked — pushing HTML alone
  yields 200 pages with broken images. — *Mitigation:* CON-005 atomicity;
  TASK-003 bundles page + both assets in one changeset; AC-004 verifies.
- **[RISK-002]:** Filename case sensitivity was the original 404 root cause
  class. — *Mitigation:* exact-path AC-001 comparison; no renames (§9
  hands-off rule); AC-007 wrong-case simulation in TASK-004.
- **[RISK-003]:** Effective-date literal must be set at authoring time (it
  equals the deploy date, not a placeholder). — *Mitigation:* TASK-008 fills
  the real date; AC-002/AC-009 diff against Table D row 3.
- **[RISK-004]:** Store listing currently displays "No data collected" while
  D6 declares collection categories (operator-side metadata mismatch). —
  *Mitigation:* flagged in TASK-013 (align data-safety categories during the
  same Play Console session); out of this repo's artifact scope.
- **[RISK-005]:** Split authoring of `index.html` across TASK-007/008 could
  drift the shared skeleton. — *Mitigation:* TASK-007 owns head+footer
  skeleton ownership; Plan B point 4 governs recovery; AC-009/AC-010 audits
  catch divergence.

---

## 7. Rollback & Recovery Strategy

1. Revert the offending changeset: `git revert <commit>` (one revert per
   phase — Phase 1 changeset, Phase 2 changeset), push to `main`.
2. GitHub Pages redeploys the prior stable state automatically (no cache
   invalidation step exists for Pages content beyond redeploy; re-verify with
   TASK-004-style live fetches).
3. If only one page is defective, revert that page's file only
   (`git revert` scoped commit or `git checkout <prev-sha> -- <file>`) —
   the surviving page remains valid because cross-links are checked by
   AC-002/AC-003 (a reverted deletion page re-creates the original 404 state;
   resubmission must be blocked until re-verified — spec AC-007).

---

## 8. 🚩 Pre-Flight Self-Correction Checklist (Anti-Patterns)

- [x] **No Horizontal Slicing:** tasks are vertical published-state slices
      (artifact → commit → deploy → live verify); the TASK-007/008 split is
      within one tracer bullet and fixed by one deploy + verify cycle.
- [x] **No Bloated Tasks (XL):** largest task touches 3 files (TASK-003);
      authoring tasks touch exactly 1 file each with bounded region scopes.
- [x] **Strict Traceability:** every actionable task carries `Ref ID` /
      `AC Ref` (or an explicit SPEC §-mandate for the glossary seed and
      operator-side tasks per spec §9); zero orphaned tasks (§3.4 matrix).
- [x] **Deterministic Dependencies:** `Dep` column is strictly sequential and
      bottom-up (TASK-001 → 002 → 003 → 004 → 005 → 006 → 007 → 008 → 009 →
      010 → 011 → 012 → 013 → 014).
- [x] **Mandatory Verification & Approval:** every phase ends with `VERIFY`
      and `APPROVAL` checkpoints (TASK-004/005/006, TASK-010/011/012).
