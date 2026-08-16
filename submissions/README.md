# Submission Schema

Files in this folder follow this schema:

```yaml
# Required fields
repo_url: https://github.com/owner/repo  # Must be a public GitHub repo
category: economy  # See category list below

# Optional fields
icon_path: assets/icon.png  # Path to icon in repo (optional)
keywords:  # Max 10 keywords for search
  - economy
  - shop
  - money
```

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `repo_url` | string | Full GitHub URL to the plugin repository |
| `category` | string | One of the accepted categories |

## Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `icon_path` | string | Relative path to icon in repo (e.g., `assets/icon.png`) |
| `keywords` | array | List of keywords for search (max 10) |

## Categories

- `admin` — Administration and server management
- `economy` — Economy, shops, currencies
- `chat` — Chat formatting, channels, ranks
- `world` — World generation, editing, protection
- `protection` — Claim systems, anti-grief
- `gameplay` — Game modes, minigames, RPG
- `fun` — Entertainment, custom features
- `api` — Libraries, frameworks, virions
- `tools` — Utilities, developer tools
- `misc` — Everything else

## Notes

- This file will be **deleted** after your plugin is approved
- You can submit again later for version updates
- Complex fields (version, author, etc.) are auto-generated from GitHub API
