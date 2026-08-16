# Axolotl Plugin Repository — Architecture & Project Brief

> This document is the complete architecture reference for the **Axolotl Plugin
> Repository** project: a PocketMine-MP plugin registry that runs entirely on
> free GitHub features (Actions, Releases, Pages, API) as a GitHub-native
> alternative to Poggit.

---

## 1. Project goals

Build a PocketMine-MP plugin registry that:

1. **Requires no paid server/backend** — entirely GitHub Pages + GitHub Actions + GitHub API.
2. **Never hosts `.phar` files** — download links always resolve directly to the GitHub Release of the plugin's own repository.
3. **Never executes anyone else's plugin code** — the system only *reads* (`plugin.yml`, README) as data; it never clones and runs/builds a developer's repository.
4. **Always has a human review gate** before a plugin goes public — automated validation is never sufficient for auto-approval.

## 2. Finalized design decisions (do not change without a strong reason)

These decisions were reached after extensive discussion — treat them as
**constraints**, not suggestions:

- **Option A is chosen**: the system does **not** provide its own build service (unlike Poggit-CI). If a developer's plugin repo has no `.phar` in GitHub Releases yet, submission is automatically rejected with a message pointing to CI setup (see §7).
- **Only public repositories** are eligible for registration. Private repos are filtered out at query level (`is:public`).
- **Developer authentication** uses a **GitHub App**, with the exact flow still to be confirmed between **PKCE** and **Device Flow** — not a classic OAuth App used purely as a confidential client. **Open item, verify before Phase 6**: as of GitHub's July 2025 PKCE changelog, GitHub does not yet distinguish public vs. confidential clients, and still expects a client secret to be sent during token exchange even when PKCE is used — unlike the "no secret needed" model PKCE provides on most other OAuth providers. Confirm GitHub's current behavior before implementing; if a secret must still be shipped client-side, treat it as a "public client secret" (GitHub's own term for a non-sensitive, expected-to-be-public secret in this scenario) rather than a confidential one, and prefer Device Flow if it avoids this requirement entirely for this use case.
- **GitHub App permission scope is minimal**: `issues: write` on the registry repo, and `contents: read` (read-only) on whichever plugin repos the user grants during App installation — enough to read `plugin.yml`/README, never `contents: write` to a developer's repo.
- **Developer tokens are kept in-memory only** in the browser (never `localStorage`/`sessionStorage`).
- **Submissions come in as a structured GitHub Issue** (using Issue Forms), not a direct PR — simpler to implement and sufficient for current needs. (Can be migrated to a fork+PR model later if a stronger audit trail is needed — see §9 as *future work*; do not implement unless explicitly requested.)
- **Approval is always manual**, done by a maintainer adding the `approved` label to the Issue — there is no auto-merge path of any kind.
- **Attestation (SLSA build provenance) is a bonus, not a requirement.** Plugins built via `axolotl-pm/pmmp-plugin-actions` with `attest: true` get a "Verified build" badge. Other plugins remain eligible but without that badge.
- **The registry index is generated periodically by a scheduled Action**, not queried live per site visitor (this avoids the 60 req/hour GitHub API rate limit for unauthenticated client-side calls).
- **Icon resolution is a three-tier fallback**: (1) developer-submitted `icon_path` is used if provided; (2) if `icon_path` is null/blank, the sync workflow fetches `assets/icon.png`; (3) if that returns 404, the registry's own default icon (constant TBD, defined in Phase 5) is used instead. `icon_url` in the index always reflects the highest-priority available tier. The raw.githubusercontent.com URL uses the literal `HEAD` segment (e.g. `/owner/repo/HEAD/assets/icon.png`); GitHub resolves this server-side to the current default branch, so no separate API call to look up the default branch is needed.

## 3. Repository structure

The project is split across repos with clearly separated responsibilities:

| Repo | Purpose |
|---|---|
| `poggit-alternative-test/poggit-alternative-test.github.io` (or similar) | React/Vite site source, deployed to GitHub Pages |
| `poggit-alternative-test/plugin-registry` | The "database" — Issue Forms for submission, plus the auto-generated index JSON |
| `axolotl-pm/pmmp-plugin-actions` (already exists) | Reusable workflows for building/releasing PocketMine-MP plugins, used by plugin developers (not by Axolotl itself) |
| Individual plugin repos (owned by each developer, outside the Axolotl org) | Plugin source + `plugin.yml` + GitHub Releases containing the `.phar` |

## 4. End-to-end system flow

```
Developer                          Axolotl (registry repo + Pages)
──────────                         ────────────────────────────────
1. Log in via GitHub App
   (PKCE/Device Flow)
   → token kept in-memory, scope
     limited to granted repos

2. System uses Search API to
   find the developer's repos
   that are:
   - is:public
   - contain filename:plugin.yml
   → shown as a picker

3. Developer selects a
   repo/plugin
   → picks a category
   → enters the icon path

4. System checks that repo has
   a GitHub Release with a
   .phar asset
   → if NOT FOUND: submission
     is rejected, developer is
     pointed to CI setup (§7)
   → if FOUND: proceed

5. Click submit
   → calls the GitHub API to
     open a new Issue (Issue
     Form) in the registry repo
                                    6. Automated Actions validation:
                                       - parse plugin.yml (read-only,
                                         via Contents API, NEVER
                                         clone & run the code)
                                       - check Release + .phar asset
                                         exist
                                       - check for attestation
                                         → set verified/community badge
                                       - post validation results as
                                         an Issue comment

                                    7. Maintainer manual review
                                       → reads the repo, skims the
                                         code
                                       → adds the `approved` label

                                    8. Actions triggered by the
                                       `approved` label:
                                       - writes an entry to the index
                                         JSON
                                       - commits (minimal-scope token,
                                         contents: write only within
                                         the registry repo)

                                    9. Scheduled Actions (e.g. every
                                       6 hours) refresh data for all
                                       registered plugins: latest
                                       version, download count,
                                       attestation status → regenerate
                                       the index

                                    10. GitHub Pages auto-redeploys
                                        → plugin appears, download
                                          button points directly to
                                          the asset on the plugin's
                                          own GitHub Release (never
                                          hosted by Axolotl)
```

## 5. Data schemas

### 5.1 `plugin.yml` (standard PocketMine-MP file, primary source of truth)

Fields that MUST be read automatically (never ask the developer to re-enter these manually):
`name`, `version`, `api` (PM API version), `main`, `author`/`authors`, `description`, `depend` (virion/dependency).

### 5.2 Fields requested manually from the developer via the form (not present in `plugin.yml`)

- `repo_url` (selected from the picker, not typed manually)
- `category` (dropdown, list of categories defined by Axolotl)
- `icon_path` (relative path to the icon file inside the repo, live-validated via the Contents API before submit)

### 5.3 Registry index JSON entry structure (example)

```json
{
  "id": "owner/repo-name",
  "name": "PluginName",
  "version": "1.2.0",
  "api": ["5.x"],
  "author": ["username"],
  "description": "A short description of what this plugin does.",
  "category": "economy",
  "icon_path": null,
  "icon_url": "https://raw.githubusercontent.com/owner/repo/HEAD/assets/icon.png",
  "repo_url": "https://github.com/owner/repo",
  "download_url": "https://github.com/owner/repo/releases/download/v1.2.0/Plugin.phar",
  "download_count": 1234,
  "download_total": 5678,
  "build_tier": "verified",
  "attestation_checked_at": "2026-08-12T00:00:00Z",
  "approved_at": "2026-08-01T00:00:00Z",
  "last_synced_at": "2026-08-12T00:00:00Z",
  "all_tags": ["v1.2.0", "v1.1.0"],
  "featured": false
}
```

`download_count` is the download count for the current approved stable version. `download_total` is the cumulative download count across all stable releases ever submitted. Frontend displays these as `{download_count}/{download_total}`.

`build_tier` is one of: `"verified"` (has SLSA attestation), `"built-via-ci"` (`.phar` uploaded by `github-actions[bot]`), `"unverified"` (manually uploaded), or `null` (unknown / release unavailable). `icon_path` is the developer-submitted path or `null`; `icon_url` uses `HEAD` in place of a branch name so GitHub resolves it server-side to the current default branch — no API call to look up the branch is needed. If neither the submitted path nor `assets/icon.png` resolves, the registry's own default icon (constant TBD) is used. `unavailable: true` may be set by the sync workflow when a repo or release becomes inaccessible rather than silently dropping the entry. `all_tags` tracks all stable release tags that have been submitted (oldest last), used for cumulative download counting.

## 6. Security model (defense in depth — 6 layers)

Do not skip or simplify the following layers during implementation:

1. **Auth**: GitHub App + PKCE/Device Flow, in-memory token, exact-match `redirect_uri` whitelist, minimal App permissions with explicit user-side repository selection.
2. **Frontend**: all text from external sources (`description`, README) is rendered as escaped text / via a sanitized Markdown renderer (e.g. `rehype-sanitize`). Never raw `innerHTML`.
3. **Data entry point**: use GitHub Issue Forms (`.github/ISSUE_TEMPLATE/*.yml`) — structured submissions, not free text.
4. **GitHub Actions**:
   - `permissions:` at the workflow level defaults to `contents: read`; elevate scope only per-job, only where needed.
   - Never interpolate `${{ github.event.issue.body }}` directly into `run:` — always pass through `env:` first.
   - Never combine `pull_request_target` with checkout-and-execute of fork code.
   - Validation of `plugin.yml`/README is always read-only via the API — never clone-and-execute a developer's repo.
5. **Human review**: mandatory, cannot be bypassed by any automated validation. This is the only layer that catches semantic threats (a malicious plugin that passes schema validation).
6. **Post-publish**: the index published to Pages only ever contains data that has passed sanitization + approval. Attestation (if present) is re-verified on every periodic sync, not just once at approval time.

## 7. Handling plugins without `pmmp-plugin-actions` / without CI

- **Hard requirement**: the repo must have at least one **stable** (non-prerelease) GitHub Release with a `.phar` asset — built by any means (Axolotl's recommended CI, a custom CI, or a manual upload). GitHub's `/releases/latest` endpoint is used to find the candidate release; this specifically excludes pre-releases. Since `pmmp-plugin-actions` marks nightly/development builds as pre-releases, a repo that only has pre-release `.phar` assets does **not** satisfy this requirement and is rejected with guidance to cut a stable release.
- The system **never builds** anything on a developer's behalf.
- The badge shown is determined by:
  - A valid SLSA attestation exists → **"Verified build"**
  - No attestation, but the asset was uploaded by `github-actions[bot]` → **"Built via CI"**
  - Asset uploaded manually by a human → no badge / **"Unverified"**
- If the repo has no stable Release with a `.phar` at submission time → **reject automatically**, with a message pointing to the `pmmp-plugin-actions` template/docs as the fastest path to setting up CI.

## 8. Label taxonomy

Workflows assign the following labels to submission Issues. Labels are mutually exclusive per workflow run:

| Label | Applied by | Meaning |
|---|---|---|
| `pending` | Issue Form (manual submission only) | Submission received, validation not yet run |
| `validation-success` | Phase 2 workflow | All automated checks passed |
| `validation-failure` | Phase 2 workflow | One or more automated checks failed |
| `approved` | Phase 3 workflow (maintainer-triggered) | Approved for inclusion in the index |

## 9. Non-goals (do not implement unless explicitly requested)

- A centralized build service (like Poggit-CI).
- Hosting/proxying `.phar` files on Axolotl's own infrastructure.
- Live GitHub API queries per page-view from visitors (must go through the statically-generated index instead).
- Permanently storing a developer's personal token anywhere in server/secrets storage.
- Auto-approving/auto-merging submissions without human review.

## 10. Future work (optional, not a current priority)

- Migrate submissions from Issue-based to fork+PR-based for a stronger commit-level audit trail.
- Client-side full-text search (Lunr.js/Fuse.js) over the static index once the plugin count grows large enough to warrant it.
- A developer dashboard (download stats, list of owned plugins) — kept read-only from public data, requiring no additional token scope.
- Automatic discovery via GitHub Topics (e.g. topic `axolotl-plugin`) as a complement to manual submission, while still preserving the manual approval gate.

## 11. Tech stack

- **Frontend**: React + Vite, **TypeScript** (not plain JS).
- **Styling**: **Tailwind CSS**.
- **Hosting**: GitHub Pages (static export from Vite build).
- **CI/build**: GitHub Actions for both the site build/deploy and the registry validation/index workflows.
- Package manager, testing framework, and linting setup: not yet decided — confirm with the project owner before introducing a default, rather than assuming.