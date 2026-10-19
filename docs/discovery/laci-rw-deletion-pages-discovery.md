# Discovery Draft: Fix Play Store Rejection — Laci RW Bandung Deletion Pages

**Date:** 2026-10-16
**Status:** Draft
**Phase:** 0 — Problem Discovery & Exploration
**Source:** Google Play Store submission rejection report (2 issues found)

---

## 1. Problem Statement

A Google Play Store update submission for the application
`gov.bdg.lacirw` (Laci RW Bandung) is blocked by two policy issues reported
by Play Console:

| # | Play Console Issue | Message |
| :- | :---------------- | :------ |
| 1 | **Account deletion page returns a page not found error** | Apps that allow users to create accounts must include a valid URL that links users to a place where they can delete their account. The link provided in the data safety declaration cannot be accessed because of a page not found error. |
| 2 | **Data deletion page returns a page not found error** | Apps that allow users to request that some account data is deleted can provide a valid URL that links to the place where they can go to complete this request. The link provided in the data safety declaration cannot be accessed because of a page not found error. |

### Pólya Problem Triad

- **The Unknown:** Two reachable (HTTP 200), policy-compliant public deletion
  pages published at GitHub Pages URLs that GitHub Pages currently serves
  as **404**, so the Play Console data safety declaration verifies successfully.
- **The Data:**
  - This repository: `diskominfobdgjabar/privacy-policy-lacirwbandung`
    (route: `https://diskominfobdgjabar.github.io/privacy-policy-lacirwbandung/`).
  - Working reference implementation:
    `https://diskominfobdgjabar.github.io/privacypolicy-jdihmobile/`
    (source: `D:\DeployAplikasi\privacy-policy-jdih\privacypolicy-jdihmobile`).
  - App contact channel: `android@diskominfo.bandung.go.id`.
  - App data categories declared in the current policy: Internet, Network
    Status, Gallery / Image Photo; third-party SDKs: Google Play Services,
    Firebase Analytics, Firebase Crashlytics, Facebook, Fabric, Matomo, Mapbox.
- **The Condition:**
  - Both declared URLs must return **HTTP 200** when Play Console verifies them.
  - The account deletion page must describe a viable deletion mechanism
    (email-based request is accepted practice, as proven by the JDIH reference
    app that passed review).
  - Deployment target is **GitHub Pages from the `main` branch** — everything
    that must be publicly reachable has to be committed and pushed.
  - Player policy baseline: Google Play "Account deletion" requirement
    (data safety form) — web resources must resolve and match the declared app.

---

## 2. Root Cause Analysis (Verified by Live Inspection)

Live HTTP checks performed on 2026-10-16:

| URL | Status | Interpretation |
| :-- | :----- | :------------- |
| `https://diskominfobdgjabar.github.io/privacy-policy-lacirwbandung/` | **200 OK** | Existing privacy policy page is live and serves the old 2023 template content. |
| `https://diskominfobdgjabar.github.io/privacy-policy-lacirwbandung/account-deletion.html` | **404 Not Found** | The file required by Issue 1 does not exist in the repository — Play Console is pointing at a URL that was never published. |
| `https://diskominfobdgjabar.github.io/privacypolicy-jdihmobile/account-deletion.html` | **200 OK** | The reference app's deletion page (the pattern to replicate) is live and passed review. |

**Root cause:** The repository contains only `index.html`. The deletion
URL(s) entered in the Play Console data safety declaration
(`/account-deletion.html` and, per Issue 2, the equivalent data deletion
target) point to files that were never created in this repo. Play Console's
automated verifier fetched the 404 and blocked the review.

### Secondary Findings (Tech Debt Critique)

1. **`assets/` is untracked.** `git status` shows `assets/` as untracked while
   `index.html` is committed. The new pages reference
   `assets/diskominfo_kota.png` / `ic_profile.png`; if the commit+push covers
   only HTML files, the pages will return 200 but render broken images. The
   assets directory must be committed with the pages.
2. **Content is stale.** The live policy is the auto-generated 2023-01-20
   template (generator footer visible, no deletion section, no coherent
   structure). The JDIH reference shows the quality bar Diskominfo wants:
   structured sections, a dedicated "Account Deletion Request" section in the
   main policy, and a hero-page deletion flow.
3. **Naming ambiguity for Issue 2.** The JDIH reference uses a single
   `account-deletion.html` that satisfies both "account deletion" and "data
   deletion" declarations (both issues were resolved by one page there).
   The safe default is to replicate exactly that pattern rather than invent
   a second URL (`data-deletion.html`) that was never validated by review.

---

## 3. Analogy & Prior Art (Pólya: "Do you know a related problem?")

Yes — the same Diskominfo organization solved this exact problem for
**JDIH Mobile Kota Bandung**, and that app's declaration passed Play review.
Reference topology (all within one GitHub Pages repo):

```text
privacypolicy-jdihmobile/          (passed Play review)
├── index.html                     Privacy policy + prominent
│                                  "Account Deletion Request" buttons
│                                  linking to account-deletion.html
│                                  + mailto fallback
└── account-deletion.html          Public deletion resource:
                                   - mailto deletion request button
                                   - How to Request Deletion
                                   - Processing Time (<= 14 business days)
                                   - What Will Be Deleted
                                   - What May Be Retained
                                   - Need Help (contact email)
```

Both issues in this rejection (account deletion 404 + data deletion 404)
were resolved there by this **single shared deletion page** linked prominently
from the main policy.

Candidate solution stack (from the reference, already proven):

- GitHub Pages static hosting (no backend, no build step, zero dependencies).
- Bootstrap 5.3.x CDN for responsive layout.
- Static, server-side rendered HTML so the automated verifier sees the full
  content (no client-side rendering — verifiers fetch raw HTML).
- `mailto:` endpoint as the actionable deletion mechanism.

---

## 4. The Inventor's Paradox — Candidate Architectures

Three candidate shapes were considered:

### Option A — Minimal Patch (NOT RECOMMENDED)

Just create a bare `account-deletion.html` with a single sentence and an
email link; leave `index.html` untouched.

- **Pros:** Smallest possible diff.
- **Cons:** Existing policy page remains a weak 2023 generator template with
  no linking seam to the deletion page; Play auditors (and the app policy
  itself) give no discoverable path to deletion; looks non-compliant if a
  human reviewer opens the site.

### Option B — Mirror the Proven JDIH Pattern (RECOMMENDED)

Replicate the topology that already passed review, adapted for Laci RW:

- `account-deletion.html`: dedicated deletion request page (hero section,
  mailto CTA, How to Request / Processing Time / What Deleted / What retained
  / Need Help cards), branded for Laci RW Bandung, reusing this repo's own
  `assets/` images and contact email.
- `index.html` upgraded to the JDIH-grade structure: prominent
  "Account Deletion Request" section with buttons to the deletion page and to
  email, Quick Summary panel, structured sections (Information Collection,
  Third-Party Services, Log Data & Cookies, Data Retention, Deletion
  Procedure, Children's Privacy, Security, Changes, Contact), and removal of
  the template-generator footer.
- Commit **both HTML files and `assets/`**, push to `main` so GitHub Pages
  redeploys.
- In Play Console, point both data-safety URLs at the verified
  `/account-deletion.html` URL (human action outside this repo).

- **Pros:** Lowest verification risk (pattern proven by review); satisfies both
  reported issues with one artifact; aligns visual/structural quality with the
  other Diskominfo properties; fixes all three findings above (missing page,
  untracked assets, stale content).
- **Cons:** Slightly larger diff than Option A; requires the user to update
  the Play Console declaration afterward (operational, not code).

### Option C — Separate Account-Deletion and Data-Deletion Pages

Create `account-deletion.html` **and** `data-deletion.html` as distinct
resources and point each Play issue to its own URL.

- **Pros:** Extremely literal mapping from error message to artifact.
- **Cons:** The data-deletion URL is *unknown* (the exact URL Play echoed is
  inside the user's Play Console form, not this rejection text); a second
  unverified page adds duplication without evidence that Play demands it —
  the JDIH precedent resolved both issues with one page; risk of the second
  URL being wrong and re-triggering a rejection cycle.

**Recommendation:** **Option B.** The JDIH reference is the strongest prior
art — it is the same organization, the same hosting, the same reviewer, and
it already passed. Keep one canonical deletion page; if the user knows the
Play Console declares a distinct data-deletion URL, the same artifact can be
declared for both fields (its content covers data deletion explicitly in the
"What Will Be Deleted" section).

> [!WARNING]
> [ASSUMPTION-001]: The data-safety URLs declared in Play Console follow the
> pattern `https://diskominfobdgjabar.github.io/privacy-policy-lacirwbandung/account-deletion.html`
> (404-verified live). Downstream risk: if Play Console actually declares a
> different path/file name, the artifact name must be adjusted to match.
> Default boundary: replicate the exact reference filename `account-deletion.html`
> and have the user confirm the two URLs in Play Console before resubmission.

> [!WARNING]
> [ASSUMPTION-002]: The deletion mechanism can remain an email-based request
> (`android@diskominfo.bandung.go.id`) with in-app data supplied by the user
> (name, app name, identifier, statement, contact). The JDIH reference passed
> review with this mechanism. Downstream risk: if `gov.bdg.lacirw` actually
> exposes a self-service in-app deletion flow, the page wording should
> reference that flow; current evidence (reference app passing with mailto)
> supports email-only. Default boundary: mailto-based flow, mirroring the
> reference verbatim in structure.

> [!WARNING]
> [ASSUMPTION-003]: Deployment is GitHub Pages serving the `main` branch of
> this repository (live 200 + 404 behavior observed). Default boundary: no
> custom-domain or branch switch; push to `main` triggers redeploy.

> [!WARNING]
> [ASSUMPTION-004]: Domain vocabulary for this project (Context Seeding
> candidate for `CONTEXT.md`): **Deletion Request Page** — a public,
> permanently hosted web page that explains and accepts account/data deletion
> requests. _Avoid_: deletion portal, GDPR removal tool, cancel-account URL.

---

## 5. Feasibility Verdict

**Feasible and low-risk.** No backend, no build pipeline, no new dependencies;
the pattern has already passed the identical review one repository away. The
only external dependencies are (a) the user updating two URL fields in Play
Console, and (b) a push to `main` and page redeploy before resubmission.

## 6. Downstream Scope Boundary

What the next phase must do (and this phase must NOT do):

- **In scope:** Create/upgrade `account-deletion.html` and `index.html`
  (static HTML only), adapt JDIH content to Laci RW naming and data
  categories, commit `assets/`, verify live URLs return 200 via post-deploy
  fetch, document checklist for Play Console resubmission.
- **Out of scope:** Any backend/email workflow automation, modifying the app
  binary (`gov.bdg.lacirw`), Play Console form edits (human step),
  in-app deletion API design.
