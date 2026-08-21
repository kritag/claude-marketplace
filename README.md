# kritag-tools

Bootstraps Claude Code on a new machine, and curates third-party skills that
don't ship a marketplace of their own.

Two layers, because they solve different problems:

| File | Handles |
| --- | --- |
| `marketplaces.txt` + `plugins.txt` | Plugins that ship their own marketplace (`claude-mem`, Anthropic's official ones). Nothing to package — just register and install. |
| `.claude-plugin/marketplace.json` | Loose skills that are a bare `SKILL.md` in someone's repo, with no marketplace to install them from. Pinned to a reviewed `sha`. |

A marketplace catalogs plugins; it can't register another marketplace. That's why
the text files exist alongside it.

## Bootstrap a machine

```bash
git clone https://github.com/kritag/claude-marketplace
claude-marketplace/bin/bootstrap
```

Idempotent — re-run it any time. It reports per line whether something was added
or already present, keeps going past failures, and exits non-zero if any step
failed. Restart Claude Code afterward; skills and plugins load at session start.

## Adding a plugin that has its own marketplace

Add its marketplace source to `marketplaces.txt` and the `plugin@marketplace` id
to `plugins.txt`, then run `bin/bootstrap`. Install it the author's way rather
than re-listing it here — plugins often hardcode their own marketplace name in
hook paths, so repackaging them under this one breaks on their next release.

## Adding a loose skill

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

## Using these skills on claude.ai, mobile, and in the browser

Claude Code skills live on the machine that runs the CLI. Nothing syncs them up
to your claude.ai account, so the phone app and browser can't see them. Syncing
only runs the other way: skills enabled on claude.ai can be pulled down into
Claude Code with `CLAUDE_CODE_SYNC_SKILLS=1` on a non-interactive run.

To use them elsewhere, upload each one to your account:

```bash
bin/export-zips --lean          # writes dist-lean/<skill>.zip
```

Then on claude.ai: **Customize > Skills > + > Create skill > Upload a skill.**

`--lean` drops screenshots and repo-level docs, which matters more than it
sounds: `task-observer` ships 3.2 MB of PNGs and goes from 3.1 MB to 32 KB.
Omit the flag for a faithful copy of the pinned skill.

Cowork and cloud sessions also read skills from your claude.ai account rather
than from this machine, so uploading covers those too.
