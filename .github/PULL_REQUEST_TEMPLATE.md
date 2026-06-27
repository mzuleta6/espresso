## What does this PR change?

<!-- Describe the change in one sentence -->

## Checklist

### If SKILL.md was modified
- [ ] I have read the full diff of `SKILL.md`
- [ ] The change does not introduce hedging, filler, or noise back into responses
- [ ] The change does not weaken or bypass the critical exceptions (irreversible ops, production, security)
- [ ] The change does not broaden the artifact exception beyond design/layout confirmation
- [ ] Both copies are in sync: `SKILL.md` and `plugins/espresso/skills/espresso/SKILL.md`

### If plugin.json was modified
- [ ] Both copies are in sync: `plugin.json` and `plugins/espresso/.claude-plugin/plugin.json`
- [ ] All declared skill paths resolve to existing files

### If .github/workflows/ was modified
- [ ] All `uses:` references are pinned to a commit SHA, not a tag or branch
- [ ] No new secrets or environment variables are introduced without justification

### Version bump
- [ ] Not needed (docs/chore only)
- [ ] Version bumped in `SKILL.md`, both `plugin.json` copies, `marketplace.json`, `README.md` badge, `CHANGELOG.md`