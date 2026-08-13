# Contributing to the Axolotl Plugin Registry

Thank you for your interest in adding a plugin to the Axolotl registry!

This document explains the two ways to submit a plugin and what happens
after you submit.

---

## Submission paths

### 1. Via the registry site (recommended)

If you have [GitHub Actions CI][pmmp-plugin-actions] set up for your plugin
repo, the fastest path is to use the submission form on the registry site.

The site will:
1. Help you pick your repository from GitHub.
2. Verify your repo has a Release with a `.phar` asset.
3. Ask you to choose a category and (optionally) provide an icon path.
4. Open a structured Issue in this repository on your behalf.

**You do not need to interact with this repository directly.**

### 2. Via the Issue Form (manual fallback)

If you cannot set up CI yet, you can still submit manually by opening a new
Issue on this repository and selecting the **"Submit a Plugin"** template.

> ⚠️ **Important:** Your repository **must** have at least one GitHub Release
> containing a `.phar` file, regardless of how it was built (manual upload,
> custom CI, or `pmmp-plugin-actions`). The registry does not build anything
> on your behalf.

For the fastest path to getting CI set up, see
[axolotl-pm/pmmp-plugin-actions][pmmp-plugin-actions].

---

## What happens after you submit

Every submission goes through the same review pipeline:

1. **Automated validation** — A GitHub Actions workflow runs a set of
   read-only checks on your repository:
   - Parses your `plugin.yml` via the GitHub Contents API.
   - Confirms a Release with a `.phar` asset exists.
   - Checks for an SLSA attestation (if your CI uses `pmmp-plugin-actions`
     with `attest: true`).
   The workflow posts a comment on your Issue with the results.

2. **Maintainer review** — A human maintainer reads your Issue, reviews
   the plugin, and decides whether to approve it.

3. **Approval** — If approved, a maintainer adds the `approved` label.
   Your plugin is then added to `data/index.json` and appears on the site.

4. **Periodic sync** — Every few hours, a scheduled workflow refreshes
   data for all approved plugins (latest version, download counts,
   attestation status). You don't need to resubmit for updates.

---

## Issue body format

> **For developers building automated submission tools (Phase 7 integrators):**
>
> When submitting via the GitHub API (`POST /repos/{owner}/{repo}/issues`),
> construct the issue body as plain markdown using the format below.
> The validation workflow parses this format — it does not process the
> Issue Form YAML schema (which is only used by the manual fallback path).
>
> **⚠ plugin.yml-derived fields are for reviewer reference only.**
> The validation workflow **always re-fetches `plugin.yml`** from `repo_url`
> via the Contents API as the source of truth. It does **not** trust the
> copy of these fields that you put in the issue body. Do not try to
> override or omit fields — they will be overwritten on validation.

```
## Required

**Repository URL:** https://github.com/owner/plugin-name

**Category:** economy

**Icon Path:** assets/icon.png
```
_(leave blank to use `assets/icon.png` from your repo automatically,
 if it exists. If neither is available, the registry's default icon
 is used instead.)_

---

## plugin.yml reference

> The fields below are copied from your `plugin.yml` for reviewer
> convenience. They are **re-verified from the repository** automatically
> and cannot be used to override anything.

**Name:**
**Version:**
**API:**
**Author:**
**Description:**

**Dependencies:**
```

---

## Plugin requirements

To be listed in the registry, a plugin must:

- Be for **PocketMine-MP**.
- Be in a **public** GitHub repository.
- Have at least one **GitHub Release** with a **`.phar`** file.
- Have a valid **`plugin.yml`** in the repository root.

There is no requirement that the `.phar` be built in any particular way.
Plugins built with `pmmp-plugin-actions` + `attest: true` receive a
"Verified build" badge. Plugins with a `.phar` uploaded by GitHub Actions
receive a "Built via CI" badge. All other plugins are listed without a badge.

---

## Badges explained

| Badge | Meaning |
|---|---|
| **Verified build** | `.phar` has a valid SLSA attestation from `pmmp-plugin-actions` with `attest: true` |
| **Built via CI** | `.phar` was uploaded by the `github-actions[bot]` account |
| *(no badge)* | `.phar` was uploaded manually by a human |

---

## Category list

Accepted categories:

- `admin` — Administration and server management
- `economy` — Economy, shops, currencies
- `chat` — Chat formatting, channels, ranks
- `world` — World generation, editing, protect
- `protection` — Claim systems, anti-grief
- `gameplay` — Game modes, minigames, RPG
- `fun` — Entertainment, custom features
- `api` — Libraries, frameworks, virions
- `tools` — Utilities, developer tools
- `misc` — Everything else

---

## Keeping your listing up to date

You don't need to resubmit when you release a new version. The sync
workflow automatically picks up new releases and updates the listing.

If your repository goes **private**, your listing will be marked
`unavailable` rather than removed, so the entry is not lost if you
re-open it later.

[pmmp-plugin-actions]: https://github.com/axolotl-pm/pmmp-plugin-actions
