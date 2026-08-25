# Axolotl Plugin Repository — Architecture & Project Brief

> This document is the complete architecture reference for the **Axolotl Plugin
> Repository** project: a PocketMine-MP plugin registry that runs entirely on
> free GitHub features (Actions, Releases, Pages, API) as a GitHub-native
> alternative to Poggit.

> **Note**: This is a fully GitHub-native system. No external authentication
> (GitHub App, OAuth, etc.) is required. The frontend is a read-only data
> viewer that simply renders the registry index.

---

## 1. Project goals

Build a PocketMine-MP plugin registry that:

1. **Requires no paid server/backend** — entirely GitHub Pages + GitHub Actions + GitHub API.
2. **Requires no external authentication** — fully GitHub-native. Developers submit via fork+PR, no login required.
3. **Never hosts `.phar` files** — download links always resolve directly to the GitHub Release of the plugin's own repository.
4. **Never executes anyone else's plugin code** — the system only *reads* (`plugin.yml`, README) as data; it never clones and runs/builds a developer's repository.
5. **Always has a human review gate** before a plugin goes public — automated validation is never sufficient for auto-approval.

## 2. Finalized design decisions (do not change without a strong reason)

These decisions were reached after extensive discussion — treat them as
**constraints**, not suggestions:

- **Option A is chosen**: the system does **not** provide its own build service (unlike Poggit-CI). If a developer's plugin repo has no `.phar` in GitHub Releases yet, submission is automatically rejected with a message pointing to CI setup (see §7).
- **Only public repositories** are eligible for registration. Private repos are filtered out at query level (`is:public`).
- **No external authentication required** — this is a fully GitHub-native system. Developers submit via fork+PR, which provides authentication via their GitHub account.
- **Submissions come in as a fork + Pull Request** with a YAML file in `submissions/`, not a direct PR to the registry repo.
- **Approval is always manual**, done by a maintainer adding the `approved` label to the PR — there is no auto-merge path of any kind. The auto-approve workflow merges the PR upon label addition.
- **Attestation (SLSA build provenance) is a bonus, not a requirement.** Plugins built via `axolotl-pm/pmmp-plugin-actions` with `attest: true` get a "Verified build" badge. Other plugins remain eligible but are marked "Unverified".
- **The registry index is generated periodically by a scheduled Action**, not queried live per site visitor (this avoids the 60 req/hour GitHub API rate limit for unauthenticated client-side calls).
- **Sync workflow updates specific fields only**: `download_count`, `download_total`, `icon_path`, `icon_url`, `stargazers_count`, `unavailable`, `last_synced_at`. Fields that reflect approval-time state (`tag`, `version`, `download_url`, `build_tier`, `attestation_checked_at`, `all_tags`, `dev_build`) are never updated by sync.
- **Icon resolution is a three-tier fallback**: (1) developer-submitted `icon_path` is used if provided; (2) if `icon_path` is null/blank, check for `icon.png`; (3) if that returns 404, check for `assets/icon.png`; (4) if all fail, null (frontend uses default). `icon_url` in the index always reflects the highest-priority available tier. The raw.githubusercontent.com URL uses the literal `HEAD` segment (e.g. `/owner/repo/HEAD/assets/icon.png`); GitHub resolves this server-side to the current default branch, so no separate API call to look up the default branch is needed.

## 3. Repository structure

The project is split across repos with clearly separated responsibilities:

| Repo | Purpose |
|---|---|
| `axolotl-pm/plugin-registry` | The "database" — submissions via PR, workflow validation, auto-generated index JSON, deployed to GitHub Pages |
| `axolotl-pm/pmmp-plugin-actions` (already exists) | Reusable workflows for building/releasing PocketMine-MP plugins, used by plugin developers |
| Individual plugin repos (owned by each developer, outside the Axolotl org) | Plugin source + `plugin.yml` + GitHub Releases containing the `.phar` |

## 4. End-to-end system flow

```
Developer                          Axolotl (registry repo + Pages)
──────────                         ────────────────────────────────
1. Fork the registry repo

2. Add a submission file:
   submissions/owner-repo.yaml
   ├── repo_url
   ├── category
   └── icon_path (optional)

3. Open a Pull Request

4. Automated validation runs:
   - Parse plugin.yml (read-only,
     via Contents API, NEVER
     clone & run the code)
   - Check Release + .phar asset
     exist
   - Check for attestation
     → set verified/unverified badge
   - Verify submitter has permission
     to the repository
   - Security scan (Semgrep)

5. Security report posted as PR
   comment with findings

6. Maintainer manual review
   → reads the repo, skims the
     code
   → adds the `approved` label

7. Actions triggered by the
   `approved` label:
   - auto-approve workflow merges
     the PR
   - approve-pr workflow writes
     entry to index JSON
   - commits (minimal-scope token,
     contents: write only within
     the registry repo)

8. Scheduled Actions (every
   6 hours) refresh data for all
   registered plugins:
   - download_count
   - download_total
   - icon_path/icon_url
   - stargazers_count
   - unavailable status
   → regenerate the index
   (NOT: tag, version,
   download_url, build_tier)

9. GitHub Pages auto-redeploys
   → plugin appears, download
     button points directly to
     the asset on the plugin's
     own GitHub Release (never
     hosted by Axolotl)

─────────────────────────────────────────────

Frontend (plugin-registry GitHub Pages):
- Read-only data viewer
- Fetches data/index.json
- No authentication required
- Simply renders the plugin list
```

## 5. Data schemas

### 5.1 `plugin.yml` (standard PocketMine-MP file, primary source of truth)

Fields that MUST be read automatically (never ask the developer to re-enter these manually):
`name`, `version`, `api` (PM API version), `main`, `author`/`authors`, `description`.

Note: `depend` (virion/dependency) is available in plugin.yml but not currently displayed in the registry.

### 5.2 Fields in submission YAML file (developer-submitted)

- `repo_url` (required) — GitHub URL to the plugin repository
- `category` (required) — dropdown, list of categories defined by Axolotl
- `icon_path` (optional) — relative path to the icon file inside the repo
- `keywords` (optional) — search keywords (max 10)

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
  "icon_path": "assets/icon.png",
  "icon_url": "https://raw.githubusercontent.com/owner/repo/HEAD/assets/icon.png",
  "repo_url": "https://github.com/owner/repo",
  "download_url": "https://github.com/owner/repo/releases/download/v1.2.0/Plugin.phar",
  "download_count": 1234,
  "download_total": 5678,
  "stargazers_count": 42,
  "build_tier": "verified",
  "attestation_checked_at": "2026-08-12T00:00:00Z",
  "approved_at": "2026-08-01T00:00:00Z",
  "last_synced_at": "2026-08-12T00:00:00Z",
  "unavailable": false,
  "all_tags": ["v1.2.0", "v1.1.0"],
  "dev_build": null,
  "featured": false,
  "featured_marked_at": null,
  "submission_pr_number": 42,
  "keywords": ["economy", "shop"],
  "fork": false,
  "forked_from": null
}
```

**Fields refreshed by sync workflow** (every 6 hours):
- `download_count` — download count for current approved version
- `download_total` — cumulative downloads across all stable releases
- `icon_path`, `icon_url` — re-resolved per 3-tier fallback
- `stargazers_count` — from GitHub API
- `unavailable` — marked true if repo/release inaccessible
- `last_synced_at` — timestamp of last sync

**Fields NOT modified by sync workflow** (reflect approval-time state):
- `tag`, `version`, `download_url` — reflect version at approval time
- `build_tier`, `attestation_checked_at` — reflect attestation at approval time
- `all_tags`, `dev_build` — managed by approval workflow only
- `approved_at` — never changes after first approval

`download_count` is the download count for the current approved stable version. `download_total` is the cumulative download count across all stable releases ever submitted. Frontend displays these as `{download_count}/{download_total}`.

`build_tier` is one of: `"verified"` (has valid SLSA attestation from pmmp-plugin-actions), `"unverified"` (no valid attestation — CI-built without attestation, manual upload, etc.), or `null` (unknown / release unavailable). Note: `github-actions[bot]` is not a trust signal because it can be impersonated by anyone — only SLSA attestation provides cryptographic proof of build origin. `icon_path` is the developer-submitted path or `null`; `icon_url` uses `HEAD` in place of a branch name so GitHub resolves it server-side to the current default branch — no API call to look up the branch is needed. If neither the submitted path nor `icon.png` nor `assets/icon.png` resolves, the registry's own default icon (constant TBD) is used. `unavailable: true` may be set by the sync workflow when a repo or release becomes inaccessible rather than silently dropping the entry. `all_tags` tracks all stable release tags that have been submitted (newest first), used for cumulative download counting.

## 6. Security model (defense in depth — 5 layers)

Do not skip or simplify the following layers during implementation:

1. **Frontend**: all text from external sources (`description`, README) is rendered as escaped text / via a sanitized Markdown renderer (e.g. `rehype-sanitize`). Never raw `innerHTML`.
2. **Data entry point**: use structured YAML files in `submissions/` via Pull Requests — structured submissions, not free text.
3. **GitHub Actions**:
   - `permissions:` at the workflow level defaults to `contents: read`; elevate scope only per-job, only where needed.
   - Never interpolate workflow inputs directly into `run:` — always pass through `env:` first.
   - Never combine `pull_request_target` with checkout-and-execute of fork code.
   - Validation of `plugin.yml`/README is always read-only via the API — never clone-and-execute a developer's repo.
   - Verify submitter has permission to the submitted repository (owner or collaborator).
4. **Human review**: mandatory, cannot be bypassed by any automated validation. This is the only layer that catches semantic threats (a malicious plugin that passes schema validation).
5. **Post-publish**: the index published to Pages only ever contains data that has passed sanitization + approval. Attestation (if present) is re-verified on every periodic sync, not just once at approval time.

## 7. Handling plugins without `pmmp-plugin-actions` / without CI

- **Hard requirement**: the repo must have at least one **stable** (non-prerelease) GitHub Release with a `.phar` asset — built by any means (Axolotl's recommended CI, a custom CI, or a manual upload). GitHub's `/releases/latest` endpoint is used to find the candidate release; this specifically excludes pre-releases. Since `pmmp-plugin-actions` marks nightly/development builds as pre-releases, a repo that only has pre-release `.phar` assets does **not** satisfy this requirement and is rejected with guidance to cut a stable release.
- The system **never builds** anything on a developer's behalf.
- The badge shown is determined by:
  - A valid SLSA attestation exists → **"Verified build"**
  - No valid attestation (CI-built without attestation, manual upload, etc.) → **"Unverified"**
- Note: `github-actions[bot]` is **not** treated as a trust signal because it can be impersonated by anyone. Only SLSA attestation provides cryptographic proof of build origin.
- If the repo has no stable Release with a `.phar` at submission time → **reject automatically**, with a message pointing to the `pmmp-plugin-actions` template/docs as the fastest path to setting up CI.

## 8. Label taxonomy

Workflows assign the following labels. Labels are mutually exclusive per workflow run:

| Label | Applied by | Meaning |
|---|---|---|
| `featured` | Maintainer (on PR) | Plugin is featured, sorted to top |
| `approved` | Maintainer (on PR) | Approved for inclusion in the index |

Note: Submission validation labels are posted as PR comments, not as GitHub labels.

## 9. Non-goals (do not implement unless explicitly requested)

- A centralized build service (like Poggit-CI).
- Hosting/proxying `.phar` files on Axolotl's own infrastructure.
- Live GitHub API queries per page-view from visitors (must go through the statically-generated index instead).
- Permanently storing a developer's personal token anywhere in server/secrets storage.
- Auto-approving/auto-merging submissions without human review (maintainer must add `approved` label).

## 10. Future work (optional, not a current priority)

- Client-side full-text search (Lunr.js/Fuse.js) over the static index once the plugin count grows large enough to warrant it.
- A developer dashboard (download stats, list of owned plugins) — kept read-only from public data, requiring no additional token scope.
- Automatic discovery via GitHub Topics (e.g. topic `axolotl-plugin`) as a complement to manual submission, while still preserving the manual approval gate.

## 11. Tech stack

- **Frontend**: React + Vite, **TypeScript** (not plain JS).
- **Styling**: **Tailwind CSS**.
- **Hosting**: GitHub Pages (static export from Vite build, served from the registry repo).
- **CI/build**: GitHub Actions for both the site build/deploy and the registry validation/index workflows.
- **Authentication**: None (fully GitHub-native via fork+PR submission).