# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clickable demo of RCMS (Registration and Capital Monitoring System, MOIC + APB, Laos). It is a single-page app in vanilla JS with no framework and no backend. All data is mock, kept in memory, and reset on reload. The UI is bilingual: Lao (the default) and English. `README.md` (in Thai) documents every screen, role and business rule. The `SPEC-*.md` / `RCMS_Phase1_Spec.md` files are the requirement notes the demo follows.

## Commands

```
npm install
npm run dev      # Vite dev server on src/ (http://localhost:5173)
npm run build    # writes the single self-contained index.html at the repo root
npm run preview  # serve the built file
```

There are no tests, linter or formatter. To verify a change, run it in a browser against `npm run dev`. After `npm run build`, the root `index.html` also opens straight from disk (`file://`), which is handy for headless browser checks. `src/index.html` itself does not: it is an ES module that imports Leaflet and the images, so it needs Vite.

Keyboard shortcuts (global `keydown` handler, ignored while typing in a field) help when testing:
- `Alt+R` resets the demo data.
- `Alt+L` switches language.
- `Alt+Q` logs out.
- `Alt+B` collapses the sidebar.
- `Alt+M` toggles the owner mobile preview.
- `Alt+/` lists every shortcut.

## Git

- Merges to `main` are squash merges, and the commit title ends with the PR number, e.g. `Rename Sign in / Sign out to Log in / Log out in English UI (#23)`.
- A push to `main` deploys to GitHub Pages, which serves the root `index.html`.

## Source vs build

- **Edit only `src/index.html`.** The root `index.html` is generated output. `vite-plugin-singlefile` inlines all JS/CSS, and images become data URIs, so the file works over `file://` and on GitHub Pages (served from `main`, repo root).
- After changing `src/index.html`, run `npm run build` and commit **both** files. Pages serves the root file, so an unbuilt change never ships.
- `vite.config.js` sets `root: 'src'` and `emptyOutDir: false`, so the build overwrites only `index.html`.
- The only runtime dependency is Leaflet (`import Leaflet from 'leaflet'` in the module script). Map tiles come from OpenStreetMap, so the map needs network access even in the built file.

## Docs to keep in sync

- `README.md` (Thai) is the per-screen spec and lists the business rules. Update it whenever a screen, role or rule changes.
- `docs/handoff/` is the guide for rebuilding the demo in React or Next.js:
  - `README.md` covers the data model, routes, business rules, components and the API draft.
  - `react.md` and `nextjs.md` cover framework-specific notes.
  - Update it when a flow, a data shape or a route changes.

## Code style

- ES5 throughout: `var`, `function(){}`, string concatenation `'a'+b`. No arrow functions, template literals or optional chaining (`SPEC-payment-fix.md` requires this).
- HTML is built as concatenated strings. Escape user text with `esc()`.
- UI text and docs use `-`, never an em-dash (README design rule).

## Architecture (all in `src/index.html`)

The `<style>` block comes first (CSS tokens in `:root`, then `/* ---------- name ---------- */` sections). The `<script type="module">` follows. Inside it, section comments (`/* ===== name ===== */`) split the code: dictionary → helpers → accounts → seed data → state → app shell → views → render/navigation → `bind()`. `reset()` runs at the end of the file to start the app.

- **i18n**
  - `D.lo` / `D.en` hold every UI string; `t('key')` looks one up.
  - Data fields carry `{lo,en}` objects and are read with `L(obj)`.
  - Any new string needs both languages.
- **State and render loop**
  - One global `S` (built by `reset()`).
  - `render()` rebuilds the whole page from `S.view` (and the role), then `bind()` re-attaches handlers.
  - Handlers are found by `data-*` attributes (`data-nav`, `data-open`, `data-task`, `data-subtab`, ...). They mutate `S`, then call `render()` or `go(view)`.
  - To add an interaction: add a `data-*` attribute in the view, then a handler in `bind()`.
  - Text inputs that re-render while typing must restore focus/caret (see the existing handlers).
- **Roles**
  - `ACCTS` lists the demo accounts: owner, maker, checker, approver, valfirm, external, admin, newuser.
  - `S.role` drives what is shown:
    - `menus()` builds the sidebar. Entries with a 5th element are sub-menus, selected via `S.dashTab` or `S.finTab` / `S.setTab`.
    - `roleHome()` picks the home view.
    - `firstTab()` picks the default sub-menu.
    - `vDash()` dispatches to the per-role dashboard (`vMakerDash`, `vCheckerDash`, `vApproverDash`, `vAdminDash`, `vOwnerList` / `vOwnerDash`).
- **Login and Create Account** (`vLogin()`, steps in `S.loginStep`: `cred` → `otp`, or `signup` → `signupDone`)
  - Login runs credentials, then OTP. Accounts with `tempPw` then get the forced set-password modal (`S.modal={type:'forgot',step:'new',first:true}`).
  - Create Account keeps its form in `S.signup`, with `S.signup.step` 1-3:
    1. Applicant details: email, Citizen ID or Passport (`idType` `cid`/`pp`), Role applied for (`SU_ROLES`), Lao and English names.
    2. Verify email: OTP via `otpWire()`.
    3. Documents, chosen by `suDocsFor(su)`. Owner gets Enterprise details (`SU_ETYPES`) plus the `DOCUMENT_TYPE` document group. Other roles upload ID/Passport plus an employee ID card (`SU_DOC`).
  - Uploads are real `<input type="file" multiple>` inside the shared upload slots (see "Document uploads"). `suFileErr()` allows PDF/JPG/PNG up to 15 MB.
  - Submitting shows a "pending document review" page. The demo skips the review: the `newuser` account becomes the applicant and can log in at once.
  - English UI says "Log in" / "Log out" and "Create Account", never "Sign in" / "Sign up".
- **Lists, search and paging** (use these instead of hand-rolled tables)
  - `listCard(o)` renders the search card (`colFilterBar` → `filterBar` with Search / Clear) plus a table card with count and pager.
    - Fields: `[col, label, options?, type?, match?]`. `options` makes a dropdown (exact match), `type:'date'` a date picker, `match(row, value)` a custom rule (`listMatch()`).
    - Typing edits `S.colDraft[key]`. Search / Enter copies it to `S.colf[key]`. Filtering happens in render, never by hiding DOM rows.
  - `paginate(key, list, sig, n0)` keeps page state in `S.pg[key]` and resets to page 1 when `sig` changes. Use it alone for tables without search.
  - `optsOf(list, getter)` builds dropdown options from values present in the data (Submit by / Verified by / Returned by).
  - Tables show Enterprise and Registration no. as separate columns, and the search splits them into separate fields too.
- **Document uploads** (one slot per document type, never a type dropdown plus one drop zone)
  - `DOC_GROUPS` is the document master from the `Doc_Type` sheet: group code → `{k, req, lo, en}`. `req` is `1`, `0` or `'foreign'` (required only when a cash payment's `src` is `foreign`). `docDefs(group, foreign)` resolves it to `[{k, lbl, req}]`.
  - `upSlot(key, def, files, err, real)` renders a slot as one compact row (label, `n/5` count, Attach button), with the attached files as wrapping chips that show the full file name. `upList(prefix, defs, owner)` stacks a group's slots top to bottom in one bordered list. Each slot takes up to `DOC_MAX_FILES` (5) files of `DOC_MAX_BYTES` (15 MB).
  - Files live in a flat `owner.files` array of `{k, name, size}`; `filesOf(owner, k)` filters one slot. Errors per slot are in `owner.upErr[k]`.
  - Slot keys route the `data-upzone` (the row, takes dropped files) / `data-updrop` (Attach button) / `data-uprm` handlers through `upOwner(key)`: `su|CODE` (signup), `pay|i|CODE` (payment block, group from `payDocGroup(p)`), `decl|CODE` (declaration-level documents in `S.form`), `val|CODE` (valuation firm result).
  - Signup opens the real file picker. The other forms attach a mock file on click (so the demo runs without files) and take real files on drop.
- **Declaration status** (the Status sheet and the user-journey flow)
  - `x.st` is one of `DST`: DRAFT, PENDING, APPRAISED_PENDING, APPRAISED_COMPLETED, APPROVED, RETURNED, COMPLETED, REJECTED. Always change it with `setSt(x, st)`, which also sets `x.stage` (`DST_STAGE`: draft / appraise / checker / approve / returned / closed). Queues filter on `x.stage`.
  - Declarations stay in `S.tasks` after they close. `LIVE_ST` / `liveTasks()` / `pendingOf()` are the ones under review; `openOf()` finds a draft or returned one, reopened in the Declare form with `S.form.edit` set to its index.
  - Reference IDs come from `capRef(n)` (`CAP-202608-000921`, counter `S.capSeq`). A draft has none.
  - A declaration with an asset becomes `APPRAISED_PENDING` and creates one valuation job per asset (`j.decl` = the reference). `valDone(j)` moves it to `APPRAISED_COMPLETED` once every job is in.
  - `doReturn()` handles Return (appraiser / checker / approver) and keeps both notes in `x.ret` (`reason` external, `internal`). Staff see both via `retNotes()` / `intNote()`; the owner sees the external note only (`isStaff()`).
  - Reject: the Approver's is final (`rejectFinal()`). The Checker's only sets `x.rejReq` and parks the declaration in the Approver's queue (`x.stage='approve'`, `x.st` unchanged, `xPill()` shows it); the Checker can cancel it (`data-rejcancel`), the Approver approves (`data-rejapprove`) or declines it (`data-rejdecline`). `approve()` signs and issues the certificate only when the capital reaches 100%; otherwise the `apconfirm` modal approves without E-Sign.
  - History wording comes from `HIST_TXT`.
  - Invoice statuses are `PENDING` / `PAID` (`WAIVED` is defined in the sheet but has no action yet).
- **Summary cards**: `sumCard({title, prog, metrics, notes})` with `metric(tone, label, value, sub)` is the shared overview pattern.
- **Mock data**
  - Enterprises: `seedEnts()` (6 hand-written demo enterprises ELN-2025-1001..1006, used by the Owner/Maker walkthroughs) plus `seedMockEnts()` from `MOCK_ENTS` (24 more, ELN-2025-1101..1124, so lists page).
  - Declarations: `seedTasks()` plus `seedMockTasks()`, plus one `RETURNED` declaration on ELN-2025-1006 from `seedReturned()`.
  - Valuation jobs: `VALJOBS` plus `MOCK_VAL`, with `mockCollateral()` on the enterprise.
  - The owner's enterprises: `OWNER_ENTS`.
  - "Today" is fixed at 04/08/2026 (`daysLeft()`).
  - Keep the six original enterprises' states intact; demo flows depend on them.
- **Dead code**: no menu leads to `vStaffDash` or `vQueue` (views `inspect` / `approveQ`). They survive only as fallbacks: `vDash()` returns `vStaffDash()` for an unmatched role, and `vTask()` returns `vQueue()` when the task is missing.
