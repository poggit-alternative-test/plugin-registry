# Axolotl Plugin Repository — Roadmap

> Companion document to `ARCHITECTURE.md`. That document defines *what*
> the system is and the constraints it must respect; this document defines
> *what to build, in what order* — each phase below is scoped to be a single
> focused unit of work, assuming earlier phases are already in place.

---

## Prerequisites (must be done manually, before Phase 1)

These require a human logged into GitHub — Claude cannot do these for you:

- [ ] Ensure the `axolotl-pm/plugin-registry` repository exists with GitHub Pages enabled.
- [ ] Decide the final Pages domain / repo name for the site.

---

## Phase 1 — Registry repo skeleton

**Goal**: the registry repo has the structure to receive submissions.

- [x] `submissions/` folder for PR-based submissions.
- [x] `data/index.json` placeholder (empty array) — the file that scheduled Actions will regenerate later.
- [x] `schema/plugin-entry.schema.json` — JSON Schema for a single registry index entry (based on architecture §5.3), used to validate output in later phases.
- [x] `CONTRIBUTING.md` explaining the submission flow at a high level, linking to `pmmp-plugin-actions` for developers who don't yet have CI.

## Phase 2 — Validation workflow

**Goal**: when a PR is opened/synced with a submission file, it gets automatically validated.

- [x] `.github/workflows/validate-pr.yml`, triggered on `pull_request: opened/synchronize/reopened`.
- [x] Steps (all read-only, per architecture §6.3):
  - Find submission files in `submissions/` folder
  - Parse `repo_url` from the YAML file.
  - Verify submitter has permission to the submitted repository.
  - Fetch `plugin.yml` via the Contents API, validate required fields exist.
  - Call `/releases/latest` — fail with guidance if it returns 404 (no stable release) or if the latest release has no `.phar` asset. Pre-release / nightly builds alone are not sufficient.
  - Check for a SLSA attestation on the asset; determine badge tier (`verified` / `unverified`).
  - Security scan (Semgrep PHP analysis).
- [x] Explicit `permissions:` block, scoped per architecture §6.3.

## Phase 3 — Approval → index write workflow

**Goal**: when a maintainer adds the `approved` label, the plugin gets written into `data/index.json`.

- [x] `.github/workflows/auto-approve.yml`, triggered on `pull_request: labeled` (filtered to `approved`).
- [x] `.github/workflows/approve-pr.yml`, triggered on `pull_request: closed` and `workflow_run` (from auto-approve).
- [x] Re-run the same read-only checks from Phase 2 (don't trust that nothing changed since validation).
- [x] Append/update the corresponding entry in `data/index.json`, matching the schema from Phase 1.
- [x] Commit with a scoped token (`contents: write` only, only in the write job).
- [x] Delete submission file after approval.

## Phase 4 — Scheduled sync workflow

**Goal**: keep the index fresh for all already-approved plugins without requiring resubmission.

- [x] `.github/workflows/sync-index.yml` on a `schedule:` trigger (every 6 hours, off-minute mark) + `workflow_dispatch`.
- [x] For every entry in `data/index.json`: refresh `download_count`, `download_total`, `icon_url`, `stargazers_count`, `unavailable`, `last_synced_at`.
- [x] Fields intentionally NOT updated by sync (reflect approval-time state): `tag`, `version`, `download_url`, `build_tier`, `attestation_checked_at`, `all_tags`, `dev_build`.
- [x] Resolve `icon_url` on every sync per the three-tier fallback (submitted `icon_path` → `icon.png` → `assets/icon.png` → null). Construct the raw.githubusercontent.com URL with the literal `/HEAD/` segment in place of a branch name — GitHub resolves this server-side; no API call to look up the plugin's default branch is needed.
- [x] Handle the "repo went private / release removed" edge case — mark the entry `unavailable` rather than silently dropping stale data.
- [x] Commit only if something actually changed (avoid empty commits / unnecessary Pages redeploys).

## Phase 5 — Site (React + Vite + TypeScript + Tailwind)

**Goal**: a static site that reads `data/index.json` and renders a plugin list. No authentication required.

- [x] Vite + React + TypeScript project scaffold (served from the registry repo via GitHub Pages).
- [x] Tailwind CSS v3 configured with Axolotl Blue brand colors (`#084DE6`).
- [x] Fetch and render `data/index.json` (runtime fetch from registry repo).
- [x] Plugin list/grid view with search, category filter.
- [x] Plugin detail view with download button, badges.
- [x] UI component library: Button, Card, Badge, Spinner.
- [x] Dark/light theme with ThemeContext.
- [x] Responsive layout (mobile/tablet/desktop).
- [x] Deployed via GitHub Actions to GitHub Pages.

> **Note**: This is a read-only data viewer. No authentication is required — submission is done via fork+PR on GitHub.

## Phase 8 — Polish / hardening pass

- [x] Re-audit every workflow against architecture §6 (no direct `${{ }}` interpolation in `run:`, explicit `permissions:` blocks, no `pull_request_target` + fork execution).
- [x] Sanitize all externally-sourced text rendering (README/description) per §6.2.
- [x] Workflow fixes and comment cleanup (all issues resolved).

- [ ] Add featured plugin monitor workflow (`featured-monitor.yml`).
- [ ] Add post-security-report workflow (`post-security-report.yml`).

---

## Explicitly out of scope for this roadmap

- Any form of authentication / login (submission is via fork+PR only)
- Any external service beyond GitHub
- Anything listed in architecture §9 (non-goals)