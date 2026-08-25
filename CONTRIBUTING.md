# Contributing to the Axolotl Plugin Registry

Thank you for your interest in adding a plugin to the Axolotl registry!

---

## Submission via Pull Request

All plugins must be submitted via Pull Request. This keeps the registry clean and allows automated validation.

### ⚠️ One Plugin Per PR

**Only ONE plugin per PR is allowed.** Each plugin must have its own separate Pull Request.

```
✅ Correct:
  PR #1: submissions/my-plugin.yaml       → 1 plugin

❌ Wrong:
  PR #1: submissions/plugin-a.yaml + plugin-b.yaml + plugin-c.yaml  → 3 plugins
```

### Prerequisites

Your plugin repository must:
- Be **public** on GitHub
- Have a valid **`plugin.yml`** in the repository root
- Have at least one **GitHub Release** with a **`.phar`** file

### Steps

1. **Fork** this repository

2. **Create a submission file** in the `submissions/` folder:
   ```yaml
   # submissions/owner-repo.yaml
   repo_url: https://github.com/owner/repo
   category: economy
   icon_path: assets/icon.png
   keywords:
     - economy
     - shop
   ```

3. **Open a Pull Request** to this repository

4. Wait for automated validation

5. A maintainer will review and add the `approved` label

6. Once approved, your plugin will be added to the site

7. Your submission file will be **automatically deleted** after approval

---

## Submission File Format

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `repo_url` | string | Full GitHub URL to your plugin repository |
| `category` | string | One of the accepted categories |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `icon_path` | string | Path to icon in your repo (e.g., `assets/icon.png`) |
| `keywords` | array | Search keywords (max 10) |

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

## What happens after you submit

1. **Automated validation** — The workflow validates your submission:
   - Checks YAML format
   - Verifies you have permission to the repository
   - Fetches `plugin.yml` from your repo
   - Confirms a Release with `.phar` exists
   - Security scan (Semgrep PHP analysis)

2. **Security report** — A report is posted as a PR comment with:
   - Attestation check result
   - PHP security findings (errors block approval)

3. **Maintainer review** — A maintainer reviews and adds the `approved` label

4. **Auto-merge** — The PR is automatically merged

5. **Approval** — Your plugin is added to the index and appears on the site

6. **Submission file deleted** — Your submission file is removed (you can submit again for updates)

---

## Updating your plugin

When you release a new version, create a **new PR** with your submission file.

```
1. Create a new submission file: submissions/my-plugin.yaml
2. Open a new Pull Request
3. Wait for validation and approval
4. Your entry is updated with the new version
```

Each version update requires its own PR.

---

## Re-submission for updates

You can submit the same plugin again for version updates:

1. Create a new PR with a submission file
2. The approval workflow detects the existing entry
3. Your entry is **updated** (not duplicated)
4. The submission file is deleted

---

## Featured Plugins

Maintainers can mark plugins as "featured" by adding the `featured` label to the submission PR.

- Featured plugins appear at the top of the list
- Maximum **10** featured plugins at a time
- The oldest featured plugin is automatically unfeatured when a new one is added

---

## Plugin requirements

To be listed in the registry, a plugin must:

- Be for **PocketMine-MP**
- Be in a **public** GitHub repository
- Have a valid **`plugin.yml`** in the repository root
- Have at least one **GitHub Release** with a **`.phar`** file

There is no requirement that the `.phar` be built in any particular way. Plugins built with `pmmp-plugin-actions` + `attest: true` receive a "Verified build" badge. All other plugins are listed with an "Unverified" badge.

---

## Build tiers (automatically determined)

| Badge | Requirement |
|-------|-------------|
| **Verified build** | Built with `pmmp-plugin-actions` + `attest: true` (SLSA attestation verified) |
| **Unverified** | No valid SLSA attestation |

Note: `github-actions[bot]` is not treated as a trust signal because it can be impersonated by anyone. Only SLSA attestation provides cryptographic proof of build origin.

---

## Need help with CI?

For the fastest path to getting your `.phar` built and attested, see:
https://github.com/axolotl-pm/pmmp-plugin-actions
