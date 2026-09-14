# Setting up a new mod from this template

This template captures the administrative/repo layer that's shared across
`kam_compat_zen`, `civilian_presence_extended`, and `Camo_Faces_Redux`: CI,
branch protection, funding, contributor docs, editor config, and the
Steam Workshop description convention. It does **not** include an addon
code skeleton — for that, scaffold with `hemtt new <addon_name>` once
you've cloned the new repo, or start from
[DartsArmaMods/ModTemplate](https://github.com/DartsArmaMods/ModTemplate).

## 1. Create the repo

Click "Use this template" on GitHub, or:

```
gh repo create Andx667/<new-repo> --template Andx667/arma-mod-template --public --clone
```

## 2. Find-and-replace placeholders

| Placeholder | Where | Replace with |
|---|---|---|
| `MOD_NAME` | `README.md`, `mod.cpp`, `.hemtt/project.toml`, `.github/workflows/release-drafter.yml`, `.github/workflows/release.yml`, `workshop/steam_description.txt`, `MOD_NAME.code-workspace` (filename too) | The mod's display name |
| `PREFIX` | `.hemtt/project.toml`, `.github/workflows/release.yml` (`FOLDER: '@PREFIX'`), `tools/stringtable_validator.py` (`PROJECT_NAME`) | HEMTT prefix, e.g. `mymod` — lowercase, matches your addon folders' `#define COMPONENT` values |
| `ABBR` | `README.md`, `workshop/steam_description.txt` | Short mod abbreviation |
| Workshop ID (`0` / `WORKSHOPID`) | `README.md` badges, `.hemtt/project.toml` (once you have `meta.cpp`'s `publishedid`), `.github/workflows/release.yml`, `workshop/steam_description.txt` | The Steam Workshop item ID once the mod is published there |
| `discord.gg/REPLACE_ME` | `README.md`, `workshop/steam_description.txt` | Real Discord invite, or delete the line if there isn't one yet |
| `Andx667/MOD_NAME` in `release-drafter.yml`'s `if:` | `.github/workflows/release-drafter.yml` | `Andx667/<new-repo-name>` |
| Dependencies line | `README.md`, `.github/release-drafter.yml`'s `template:` | Actual required addons, or the "no hard dependencies" wording if there are none |

## 3. Add repo secrets (for release.yml)

Settings → Secrets and variables → Actions:
- `STEAM_USERNAME`
- `STEAM_PASSWORD`

(Skip if the mod won't auto-upload to Steam Workshop on release.)

## 4. Apply branch protection

Template repos don't carry rulesets over. Once the repo exists, apply the
same ruleset the other three repos share — block deletion/force-push,
require the `check` and `validate` status checks, 0 required approvals,
repo-admin bypass:

```bash
gh api --method POST -H "Accept: application/vnd.github+json" \
  repos/Andx667/<new-repo>/rulesets --input - <<'JSON'
{
  "name": "default",
  "target": "branch",
  "enforcement": "active",
  "conditions": {"ref_name": {"exclude": [], "include": ["~DEFAULT_BRANCH"]}},
  "rules": [
    {"type": "deletion"},
    {"type": "non_fast_forward"},
    {"type": "pull_request", "parameters": {"required_approving_review_count": 0, "dismiss_stale_reviews_on_push": false, "required_reviewers": [], "require_code_owner_review": false, "require_last_push_approval": false, "required_review_thread_resolution": false, "require_extra_approval_for_unattributed_changes": true, "allowed_merge_methods": ["merge", "squash", "rebase"]}},
    {"type": "required_status_checks", "parameters": {"strict_required_status_checks_policy": true, "do_not_enforce_on_create": true, "required_status_checks": [{"context": "check", "integration_id": 15368}, {"context": "validate", "integration_id": 15368}]}}
  ],
  "bypass_actors": [{"actor_id": 5, "actor_type": "RepositoryRole", "bypass_mode": "always"}]
}
JSON
```

## 5. Everything else

- Add real `img/icon.png` / `img/icon_ca.paa` (referenced by `mod.cpp` and the README)
- Scaffold your first addon (`hemtt new <name>`) — `check`/`validate`/`build` will have nothing to check until one exists
- Fill in `workshop/` with real screenshots once you have them
- Set repo topics (at minimum: `arma3`) and the repo description/homepage to the Steam Workshop URL once published
