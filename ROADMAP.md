# Axolotl Plugin Repository — Roadmap

> Companion document to `ARCHITECTURE.md`. That document defines *what*
> the system is and the constraints it must respect; this document defines
> *what to build, in what order* — each phase below is scoped to be a single
> focused unit of work, assuming earlier phases are already in place.

---

## Prerequisites (must be done manually, before Phase 1)

These require a human logged into GitHub — Claude cannot do these for you:

- [ ] Create the `poggit-alternative-test/plugin-registry` repository (empty, or with a basic README).
- [ ] Create the GitHub App (Settings → Developer settings → GitHub Apps → New GitHub App):
  - Permissions: `Issues: Read & write` on the registry repo scope; `Contents: Read-only` for repos the installing user selects.
  - Enable Device Flow or set up the callback URL for PKCE, matching the future Pages domain.
  - Record: **App ID**, **Client ID**, and the exact **redirect URI** — Claude will need these values for the auth implementation.
- [ ] Decide the final Pages domain / repo name for the site (affects the redirect URI above).

---

## Phase 1 — Registry repo skeleton

**Goal**: the `poggit-alternative-test/plugin-registry` repo has the structure to receive submissions, even before the site exists.

- [ ] `.github/ISSUE_TEMPLATE/submit-plugin.yml` (GitHub Issue Form) with the fields defined in architecture §5.2 (`repo_url`, `category`, `icon_path`) plus fields auto-filled client-side from `plugin.yml` before submit.
- [ ] `data/index.json` placeholder (empty array) — the file that scheduled Actions will regenerate later.
- [ ] `schema/plugin-entry.schema.json` — JSON Schema for a single registry index entry (based on architecture §5.3), used to validate output in later phases.
- [ ] `CONTRIBUTING.md` explaining the submission flow at a high level, linking to `pmmp-plugin-actions` for developers who don't yet have CI.

## Phase 2 — Validation workflow

**Goal**: when an Issue is opened via the form, it gets automatically validated and commented on — no approval logic yet.

- [x] `.github/workflows/validate-submission.yml`, triggered on `issues: opened/edited`.
- [x] Steps (all read-only, per architecture §6.4):
  - Parse `repo_url` from the issue body (via `env:`, never inline in `run:`).
  - Fetch `plugin.yml` via the Contents API, validate required fields exist.
  - Call `/releases/latest` — fail with guidance if it returns 404 (no stable release) or if the latest release has no `.phar` asset. Pre-release / nightly builds alone are not sufficient.
  - Check for a SLSA attestation on the asset; determine badge tier (`verified` / `built-via-ci` / `unverified`).
  - Post a summary comment on the Issue with pass/fail per check; update/replace an existing comment on re-runs using a marker so the issue does not accumulate duplicates.
- [x] Explicit `permissions:` block, scoped to `issues: write`, `contents: read` only.

## Phase 3 — Approval → index write workflow

**Goal**: when a maintainer adds the `approved` label, the plugin gets written into `data/index.json`.

- [x] `.github/workflows/approve-submission.yml`, triggered on `issues: labeled` (filtered to `approved`).
- [x] Re-run the same read-only checks from Phase 2 (don't trust that nothing changed since validation).
- [x] Append/update the corresponding entry in `data/index.json`, matching the schema from Phase 1.
- [x] Commit with a scoped token (`contents: write` only, only in the write job).
- [x] Post approval or failure comment on the issue; remove `approved` label to prevent re-trigger.

## Phase 4 — Scheduled sync workflow

**Goal**: keep the index fresh for all already-approved plugins without requiring resubmission.

- [x] `.github/workflows/sync-index.yml` on a `schedule:` trigger (every 6 hours, off-minute mark) + `workflow_dispatch`.
- [x] For every entry in `data/index.json`: refresh version, download count, attestation status.
- [x] Resolve `icon_url` on every sync per the three-tier fallback (submitted `icon_path` → `assets/icon.png` convention → null). Construct the raw.githubusercontent.com URL with the literal `/HEAD/` segment in place of a branch name — GitHub resolves this server-side; no API call to look up the plugin's default branch is needed.
- [x] Handle the "repo went private / release removed" edge case — mark the entry `unavailable` rather than silently dropping stale data.
- [x] Commit only if something actually changed (avoid empty commits / unnecessary Pages redeploys).

## Phase 5 — Site skeleton (React + Vite + TypeScript + Tailwind)

**Goal**: a static site that reads `data/index.json` and renders a plugin list. No auth or submission yet.

- [x] Vite + React + TypeScript project scaffold (separate repo: `poggit-alternative-test/poggit-alternative-test.github.io`).
- [x] Tailwind CSS v3 configured with Axolotl Blue brand colors (`#084DE6`).
- [x] Fetch and render `data/index.json` (runtime fetch from registry repo).
- [x] Plugin list/grid view with search, category filter.
- [x] Plugin detail view with download button, badges.
- [x] UI component library: Button, Card, Badge, Spinner.
- [x] Dark/light theme with ThemeContext.
- [x] Responsive layout (mobile/tablet/desktop).
- [x] Deployed via GitHub Actions to GitHub Pages.

> **Note**: Tailwind v3 used (not v4) for consistency with reference project.
> Brand colors corrected to Axolotl Blue (`#084DE6`).
> Site repo: `poggit-alternative-test/poggit-alternative-test.github.io`

## Phase 6 — Auth flow (GitHub App, PKCE/Device Flow)

**Goal**: developers can log in from the site, token kept in-memory only.

- [x] **Before writing any code**: confirm current GitHub behavior for PKCE + client secret requirements on GitHub Apps (see architecture §2, open item). This determines whether the flow needs a "public client secret" shipped in the frontend bundle or can go fully secretless via Device Flow. Do not assume either outcome — check GitHub's current docs/changelog first.  **Result**: GitHub still requires a client secret during Device Flow token exchange even with PKCE; treated as a "public client secret" in the bundle.
- [x] Implement the chosen flow using the App ID/Client ID from the prerequisites step.
- [x] Token stored in React state only — verify it never touches `localStorage`/`sessionStorage` (architecture §6.1).
- [x] Basic "logged in as X" UI state, logout clears in-memory token.

## Phase 7 — Repo picker + submission form

**Goal**: the full submission flow described in architecture §4, steps 2–5.

- [x] Search API call: `filename:plugin.yml is:public user:{username}` (or across granted installation repos).
- [x] Picker UI showing candidate repos with `plugin.yml` preview data.
- [x] Check for existing Release + `.phar` asset before allowing submission; if missing, show the CI setup guidance instead of a submit button.
- [x] Category dropdown, icon path field with live preview (Contents API validation).
- [x] Submit → calls GitHub API to open the Issue Form-shaped Issue in `poggit-alternative-test/plugin-registry`, using the developer's own in-memory token (least privilege — Axolotl's own tokens never touch this step).

## Phase 8 — Polish / hardening pass

- [ ] Re-audit every workflow against architecture §6 (no direct `${{ }}` interpolation in `run:`, explicit `permissions:` blocks, no `pull_request_target` + fork execution).
- [ ] Sanitize all externally-sourced text rendering (README/description) per §6.2.
- [ ] Rate-limit check on the client side for repeated submissions from the same account.
- [ ] Error states: no repos found, repo has no `plugin.yml`, repo has no Release, API rate limit hit.

---

## Explicitly out of scope for this roadmap

Anything listed in architecture §8 (non-goals) and §9 (future work) — do not pull these into a phase unless the project owner decides to promote them explicitly.