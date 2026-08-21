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

Get the commit you want, add an entry, validate, commit.

```bash
git ls-remote https://github.com/owner/repo main   # copy the full 40-char sha
```

For a repo with `SKILL.md` at the root and no `plugin.json` (the common shape for
a standalone skill), `strict: false` plus `"skills": "."` is required — without
them it isn't a valid plugin:

```json
{
  "name": "some-skill",
  "source": {
    "source": "github", "repo": "owner/repo",
    "ref": "main", "sha": "<full 40-char sha>"
  },
  "strict": false,
  "skills": ".",
  "category": "writing",
  "homepage": "https://github.com/owner/repo",
  "license": "MIT"
}
```

If the skill sits in a subdirectory of a monorepo, use `git-subdir` — it sparse
clones, so you don't pull the whole repo:

```json
{
  "source": "git-subdir",
  "url": "https://github.com/owner/repo.git",
  "path": "skills/some-skill",
  "ref": "main",
  "sha": "<sha>"
}
```

If the upstream repo already ships a proper `.claude-plugin/plugin.json`, drop
`strict` and `skills` and let its own manifest define the components.

Then:

```bash
claude plugin validate . --strict
```

## Updating pins

`bin/check-updates` compares every pinned sha against upstream and prints a
compare URL for anything behind:

```bash
./bin/check-updates
```

Review the diff, bump the `sha`, commit, push, then:

```bash
claude plugin marketplace update kritag-tools
claude plugin update some-skill@kritag-tools
```

Pinning is the point: an upstream force-push or a hijacked account can't change
what you run until you deliberately move the sha.
