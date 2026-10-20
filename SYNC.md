# How this repo stays in sync

This repo is the **canonical, publishable** source for the Turtini Claude Code plugin. It was bootstrapped from `packages/claude-code-skill/` in the [`turtini` monorepo](https://github.com/Turtini/turtini), but going forward all edits should land here first.

## Updating the SKILL or plugin manifest

1. Edit files in this repo directly (`skills/turtini/SKILL.md`, `.claude-plugin/plugin.json`, `README.md`).
2. Bump `version` in `.claude-plugin/plugin.json` per semver.
3. `git commit` + `git push` to `main`. Anyone with the plugin installed picks up the update on the next `claude plugin update` (or on plugin reinstall).

## Mirroring back into the monorepo (optional)

If you want the monorepo copy to stay current for offline browsing or in-monorepo greps:

```bash
rsync -a --delete \
  ~/turtini-claude-code-plugin/ \
  ~/turtini/packages/claude-code-skill/ \
  --exclude .git --exclude .gitignore --exclude SYNC.md
```

The monorepo copy is for reference only — never publish from there.

## Releases

Tag releases with the same version as `plugin.json`:

```bash
git tag v0.1.0
git push --tags
```

The Claude Code plugin manager honors the tag if a user installs with `github.com/Turtini/claude-code-plugin@v0.1.0`.

## Background sync of the tool list

A scheduled remote agent runs weekly to compare the SKILL's documented tool list against `npx -y @turtini/mcp --list-tools`. If they diverge it opens a PR with the corrected list — see the routine named `turtini-mcp-skill-sync`.
