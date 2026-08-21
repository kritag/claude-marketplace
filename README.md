# kritag-tools

A curated collection of third-party Claude Code skills, each pinned to a commit
I've reviewed. Nothing here is vendored and nothing is a submodule — each entry
in `.claude-plugin/marketplace.json` points at an upstream repo and a `sha`.
Claude Code fetches the content at install time.

## Install on a new machine

```bash
claude plugin marketplace add kritag/claude-marketplace
claude plugin install stop-slop@kritag-tools --scope user
```

Or declare it in `~/.claude/settings.json` and skip the commands entirely:

```json
{
  "extraKnownMarketplaces": {
    "kritag-tools": {
      "source": { "source": "github", "repo": "kritag/claude-marketplace" }
    }
  },
  "enabledPlugins": { "stop-slop@kritag-tools": true }
}
```

## Adding a skill

```bash
bin/add-skill owner/repo                       # SKILL.md at the repo root
bin/add-skill owner/repo --path skills/pdf     # skill inside a monorepo
bin/add-skill owner/repo --ref v2.0.0          # pin a tag instead of main
bin/add-skill owner/repo --name better-name --no-install
```

It resolves the current sha, shallow-fetches the repo to inspect it, and works
out how the entry has to be shaped:

| What's in the repo | How it's added |
| --- | --- |
| `SKILL.md` at the root | `strict: false`, `skills: "."` |
| `skills/<name>/SKILL.md` | `strict: false`, `skills: "./skills"` |
| `.claude-plugin/plugin.json` | left alone — its own manifest defines components |

The name and description come from the `SKILL.md` frontmatter (or `plugin.json`),
`--path` switches the source to `git-subdir` so a monorepo sparse-clones instead
of pulling everything, and the entry is validated before anything is committed.
Then it commits, pushes, refreshes the marketplace, and installs.

It refuses to do anything if the ref doesn't resolve, the `--path` doesn't exist,
the repo has no skill in it, or the name is already taken — and leaves the
manifest untouched when it refuses.

## Updating

```bash
bin/update-skills --check        # report only, change nothing
bin/update-skills               # prompt per skill, with a compare URL to review
bin/update-skills stop-slop     # just one
bin/update-skills --yes         # take everything without prompting
```

Anything behind prints a `compare/<pinned>...<upstream>` URL. Read the diff
before you accept it — that review step is the entire reason the pins exist.
Accepting bumps the sha, commits, pushes, refreshes the marketplace and updates
the installed plugin.

## Why pushing matters

The marketplace is registered by its git URL, not by the local path, so Claude
Code reads its own clone under `~/.claude/plugins/marketplaces/kritag-tools`.
Editing `marketplace.json` here changes nothing until it's pushed. Both scripts
push for you, which is why they exist.

After either script, restart Claude Code — skills are discovered at session
start.
