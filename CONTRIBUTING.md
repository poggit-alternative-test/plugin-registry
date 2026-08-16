# Contributing to the Axolotl Plugin Registry

Thank you for your interest in adding a plugin to the Axolotl registry!

---

## Submission via Pull Request

All plugins must be submitted via Pull Request. This keeps the registry clean and allows automated validation.

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
   - Fetches `plugin.yml` from your repo
   - Confirms a Release with `.phar` exists

2. **Maintainer review** — A maintainer reviews and adds the `approved` label

3. **Approval** — Your plugin is added to the index and appears on the site

4. **Submission file deleted** — Your submission file is removed (you can submit again for updates)

---

## Updating your plugin

When you release a new version:

1. Open a **new PR** with your submission file again
2. The workflow detects it's an update
3. Your entry is updated with the new version

---

## Re-submission for updates

You can submit the same plugin again for version updates:

1. Create a new PR with a submission file
2. The approval workflow detects the existing entry
3. Your entry is **updated** (not duplicated)
4. The submission file is deleted

---

## Featured Plugins

Maintainers can mark plugins as "featured" by adding the `featured` label to the submission issue.

- Featured plugins appear at the top of the list
- Maximum **10** featured plugins at a time
- The oldest featured plugin is automatically unfla when a new one is added

---

## Plugin requirements

To be listed in the registry, a plugin must:

- Be for **PocketMine-MP**
- Be in a **public** GitHub repository
- Have a valid **`plugin.yml`** in the repository root
- Have at least one **GitHub Release** with a **`.phar`** file

There is no requirement that the `.phar` be built in any particular way. Plugins built with `pmmp-plugin-actions` + `attest: true` receive a "Verified build" badge. Plugins with a `.phar` uploaded by GitHub Actions receive a "Built via CI" badge. All other plugins are listed without a badge.

---

## Build tiers (automatically determined)

| Badge | Requirement |
|-------|-------------|
| **Verified build** | Built with `pmmp-plugin-actions` + `attest: true` |
| **Built via CI** | `.phar` uploaded by `github-actions[bot]` |
| *(no badge)* | `.phar` uploaded manually |

---

## Need help with CI?

For the fastest path to getting your `.phar` built and attested, see:
https://github.com/axolotl-pm/pmmp-plugin-actions
