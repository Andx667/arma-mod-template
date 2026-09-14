# Setting up a new mod from this template

This template captures the administrative/repo layer that's shared across
`kam_compat_zen`, `civilian_presence_extended`, `Camo_Faces_Redux`,
`tactical-tarps`, and `clear-hud-rewrite`: CI, branch protection, funding,
contributor docs, editor config, and the Steam Workshop description
convention. It also ships a minimal, dependency-free
`addons/main` so a fresh clone actually builds (`hemtt check` / `hemtt build`
both pass out of the box) — it's intentionally not a full CBA/ACE-integrated
skeleton. If your mod depends on CBA_A3, swap `addons/main/script_mod.hpp`'s
minimal macro block for CBA's real `script_macros_common.hpp` (see
[DartsArmaMods/ModTemplate](https://github.com/DartsArmaMods/ModTemplate) for
what that looks like). `hemtt new` only scaffolds a whole new project, not a
single addon within an existing one — copy `addons/main` as a starting point for each new
component instead.

## 1. Create the repo

Click "Use this template" on GitHub, or:

```
gh repo create Andx667/<new-repo> --template Andx667/arma-mod-template --public --clone
```

## 2. Find-and-replace placeholders

Four distinct tokens, deliberately not sharing text with any real macro name
(`MOD_NAME`/`PREFIX` are actual CBA/HEMTT macro names used in the addon code,
so the placeholder values below avoid colliding with them):

| Placeholder | Meaning | Where |
|---|---|---|
| `MOD_TITLE` | Human display name, e.g. `KAM Compat ZEN` | `README.md`, `mod.cpp`, `.hemtt/project.toml`, `workshop/steam_description.md`, `addons/main/script_mod.hpp` (`#define MOD_NAME MOD_TITLE`) |
| `MOD_REPO` | GitHub repo slug (URL-safe), e.g. `kam_compat_zen` | GitHub URLs in `README.md`/`mod.cpp`/`workshop/steam_description.md`, `.github/workflows/release-drafter.yml`'s `if:`, `MOD_REPO.code-workspace` (filename too) |
| `MOD_PREFIX` | HEMTT prefix / code namespace, e.g. `kcz` — lowercase, matches every addon's `#define COMPONENT` | `.hemtt/project.toml` (`prefix`), `addons/main/$PBOPREFIX$`, `addons/main/script_mod.hpp` (`#define PREFIX MOD_PREFIX`), `addons/main/stringtable.xml`, `.github/workflows/release.yml` (`FOLDER: '@MOD_PREFIX'`), `tools/stringtable_validator.py` (`PROJECT_NAME`) |
| `MOD_ABBR` | Short abbreviation, e.g. `KCZ` | `README.md`, `workshop/steam_description.md` |

Also:

| Placeholder | Where | Replace with |
|---|---|---|
| Workshop ID (`0`) | `README.md` badges, `meta.cpp` (`publishedid`), `workshop/steam_description.md` | The Steam Workshop item ID once the mod is published there |
| `discord.gg/REPLACE_ME` | `README.md`, `workshop/steam_description.md` | Real Discord invite, or delete the line if there isn't one yet |
| Dependencies line | `README.md`, `.github/release-drafter.yml`'s `template:` | Actual required addons, or the "no hard dependencies" wording if there are none |

## 3. Apply branch protection

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

## 4. Keep CHANGELOG.md current, and know how hemtt publish uses it

This template ships `CHANGELOG.md` (Keep a Changelog format) and a
`workshop/steam_description.md` already wired into `.hemtt/project.toml`'s
`[hemtt.publish]` section. Once your Steam Workshop item exists
(`meta.cpp` has a real `publishedid`), running `hemtt publish` locally
(Steam must be running and logged in — it uses the desktop client, not
a username/password secret) builds the mod, converts both Markdown
files to Steam Workshop BBCode, and uploads — no more hand-maintained
BBCode, and no CI secrets to manage for it.

`release.yml` (CI) only builds the mod and attaches the zip to the
GitHub release on publish — it no longer touches Steam Workshop at
all, so there's nothing to configure there for that.

The changelog step specifically looks up the entry whose heading
*exactly* matches the current project version (from
`addons/main/script_version.hpp`) — so right before you publish a
release, rename `## [Unreleased]` to `## [X.Y.Z] - YYYY-MM-DD` matching
that version, and start a fresh empty `## [Unreleased]` above it. If
there's no matching entry, `hemtt publish` fails with "No changelog
entry found for version X.Y.Z".

Add an entry under `## [Unreleased]` for every user-facing change as
you make it — don't leave it to write itself at release time.

## 5. Everything else

- Add real `img/icon.png` / `img/icon_ca.paa` (referenced by `mod.cpp` and the README)
- Fill in `workshop/` with real screenshots once you have them
- Set repo topics (at minimum: `arma3`) and the repo description/homepage to the Steam Workshop URL once published
