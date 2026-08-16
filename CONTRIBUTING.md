# Contributing to the Axolotl Plugin Registry

Thank you for your interest in adding a plugin to the Axolotl registry!

---

## Submission via Pull Request (required)

All plugins must be submitted via Pull Request to this repository.

### Prerequisites

Your plugin repository must:
- Be **public** on GitHub
- Have a valid **`plugin.yml`** in the repository root
- Have at least one **GitHub Release** with a **`.phar`** file (built any way you prefer)

> ⚠️ The registry does **not** build anything on your behalf. Your `.phar` must already exist in a Release.

### Steps

1. **Fork** this repository
2. **Add your plugin** to `data/index.json` (see format below)
3. **Open a Pull Request**
4. Wait for automated validation and maintainer review

---

## What happens after you submit

Every submission goes through the same review pipeline:

1. **Automated validation** — A GitHub Actions workflow runs read-only checks:
   - Parses your `plugin.yml` via the GitHub Contents API
   - Confirms a Release with a `.phar` asset exists
   - Checks for SLSA attestation (if built with `pmmp-plugin-actions` + `attest: true`)
   The workflow posts the results as a PR comment.

2. **Maintainer review** — A human reviews your PR and decides whether to merge.

3. **Merge** — Your plugin is added to the index and appears on the site.

4. **Periodic sync** — Every few hours, data is refreshed automatically (latest version, download counts, attestation status).

---

## Plugin entry format

Add an entry to `data/index.json` with this structure:

```json
{
  "id": "owner/repo-name",
  "name": "PluginName",
  "version": "1.0.0",
  "api": ["5.0.0"],
  "author": ["username"],
  "description": "A short description of what this plugin does.",
  "category": "economy",
  "icon_path": null,
  "icon_url": "https://raw.githubusercontent.com/owner/repo/HEAD/assets/icon.png",
  "repo_url": "https://github.com/owner/repo",
  "download_url": "https://github.com/owner/repo/releases/latest/download/Plugin.phar",
  "download_count": 0,
  "build_tier": null,
  "attestation_checked_at": null,
  "approved_at": null,
  "last_synced_at": null
}
```

### Fields explained

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | `owner/repo-name` (same as GitHub repo) |
| `name` | Yes | Plugin display name (from `plugin.yml`) |
| `version` | Yes | Latest version (from latest Release) |
| `api` | Yes | Supported API versions (from `plugin.yml`) |
| `author` | Yes | Array of author names/IDs |
| `description` | Yes | Short description (from `plugin.yml`) |
| `category` | Yes | One of the accepted categories (see below) |
| `icon_path` | No | Path to icon in repo (e.g. `assets/icon.png`), or `null` |
| `icon_url` | Auto | Auto-generated from `icon_path` or `assets/icon.png` |
| `repo_url` | Yes | Full GitHub URL |
| `download_url` | Yes | URL to the `.phar` in latest Release |
| `download_count` | Auto | Updated by sync workflow |
| `build_tier` | Auto | `verified`, `built-via-ci`, `unverified`, or `null` |
| `attestation_checked_at` | Auto | Updated by sync workflow |
| `approved_at` | Auto | Set by maintainer |
| `last_synced_at` | Auto | Updated by sync workflow |

### Categories

| Category | Description |
|----------|-------------|
| `admin` | Administration and server management |
| `economy` | Economy, shops, currencies |
| `chat` | Chat formatting, channels, ranks |
| `world` | World generation, editing, protection |
| `protection` | Claim systems, anti-grief |
| `gameplay` | Game modes, minigames, RPG |
| `fun` | Entertainment, custom features |
| `api` | Libraries, frameworks, virions |
| `tools` | Utilities, developer tools |
| `misc` | Everything else |

---

## Build tiers (automatically determined)

| Badge | Requirement |
|-------|-------------|
| **Verified build** | Built with `pmmp-plugin-actions` + `attest: true` |
| **Built via CI** | `.phar` uploaded by `github-actions[bot]` |
| *(no badge)* | `.phar` uploaded manually |

---

## Keeping your listing up to date

You don't need to submit updates manually. The sync workflow automatically picks up new releases and updates the listing.

If your repository goes **private**, your listing will be marked `unavailable` rather than removed.

---

## Need help with CI?

For the fastest path to getting your `.phar` built and attested, see:
https://github.com/axolotl-pm/pmmp-plugin-actions
