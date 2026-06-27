# Security Audit — mzuleta6/espresso

**Date:** 2026-06-26  
**Auditor:** Claude Sonnet 4.6  
**Scope:** SKILL.md, plugin.json, marketplace.json, .github/workflows/, CODEOWNERS, git history, cross-file consistency  
**Primary attack vector:** Malicious PR modifies SKILL.md → merges without review → all users with `always: true` receive the modified skill  
**Result:** 0 critical · 2 high · 5 medium · 8 low

---

## Summary

| Severity | Count | Worst finding |
|---|---|---|
| High | 2 | CI workflow is untracked — GitHub has no validation on any PR |
| Medium | 5 | `actions/checkout` not SHA-pinned; root plugin.json path broken |
| Low | 8 | Missing SECURITY.md, PR template, workflow permissions block |
| Clean | — | No secrets in history, no injection, SKILL.md copies in sync |

---

## High

### H1 — CI workflow exists locally but has never been committed

**File:** `.github/workflows/validate.yml` · (`git status: ??` — untracked)

`validate.yml` exists on disk but is untracked. GitHub has never seen it. Every push and every PR to the remote repo runs with zero CI: no JSON validation, no skill path resolution check, no SKILL.md sync check, no banner check. Combined with CODEOWNERS not being enforced (see H2), this means a malicious PR that modifies `SKILL.md` can be merged without any automated or human gate. All users with `always: true` would silently receive the modified skill on their next install or update.

Same status applies to `.github/ISSUE_TEMPLATE.MD` — untracked, never committed.

**Attack path:** Fork → modify `SKILL.md` with adversarial instructions → open PR → merge (no CI blocks, no required reviews) → users pull from `main` → poisoned skill loads into every Claude Code session.

**Fix:**

```bash
git add .github/workflows/validate.yml .github/ISSUE_TEMPLATE.MD
git commit -m "chore: add CI workflow and issue template"
git push
```

Then enable branch protection (see H2).

---

### H2 — No branch protection: CODEOWNERS committed but not enforced

**File:** `.github/CODEOWNERS` · (committed in `e8629819`, not yet pushed to remote)

`CODEOWNERS` was committed locally but the commit has not been pushed. Even once pushed, CODEOWNERS has no enforcement power without branch protection rules enabled in GitHub repository settings. Without branch protection requiring code owner review, any contributor with write access can merge to `main` without approval. Without branch protection blocking PRs when CI fails (and CI doesn't exist on GitHub per H1), the `validate.yml` checks are irrelevant even after being committed.

**Fix:**

1. Push pending commits to remote.
2. In GitHub → Settings → Branches → Add rule for `main`:
   - ✅ Require a pull request before merging
   - ✅ Require approvals (minimum 1)
   - ✅ Require review from Code Owners
   - ✅ Require status checks to pass before merging → add `validate` job
   - ✅ Restrict who can push to matching branches
   - ✅ Do not allow force pushes
   - ✅ Do not allow deletions

---

## Medium

### M1 — `actions/checkout` pinned to tag, not commit SHA

**File:** `.github/workflows/validate.yml` · line 13

```yaml
- uses: actions/checkout@v4
```

Tags are mutable. A compromised `actions/checkout` repo could push a malicious commit to `v4` and it would silently run in CI on the next trigger. GitHub's 2026 supply chain guidance requires pinning third-party actions to their full commit SHA.

**Fix:**

```yaml
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
```

Verify the SHA at [github.com/actions/checkout/releases](https://github.com/actions/checkout/releases) and update with each major release.

---

### M2 — Root `plugin.json` skill path does not exist; CI step will always fail

**File:** `plugin.json` · line 12 · `validate.yml` · lines 30–37

`plugin.json` declares:

```json
"skills": [{ "path": "skills/espresso/SKILL.md" }]
```

Resolved from the repo root, `skills/espresso/SKILL.md` does not exist. The `validate.yml` step "Verify skill paths resolve" runs `test -f "$ROOT_PATH"` against this path and exits 1. CI would fail on every run the moment the workflow is committed. The nested plugin (`plugins/espresso/.claude-plugin/plugin.json`) uses the same path string, but resolves it correctly relative to `plugins/espresso/`, so that path does work.

**Fix — Option A:** Delete the root `plugin.json` (the canonical copy is `plugins/espresso/.claude-plugin/plugin.json`). Update the CI step to only validate the nested copy.

**Fix — Option B:** Keep the root copy but change its path to be repo-root-relative:

```json
"skills": [{ "path": "plugins/espresso/skills/espresso/SKILL.md" }]
```

---

### M3 — All install paths pin to mutable `main` branch

**Files:** `marketplace.json` · line 14 · `plugin.json` · line 19 · `README.md` · lines 28, 30

`marketplace.json` sets `"ref": "main"` and every `raw.githubusercontent.com` URL references `main`. A push to `main` — including by an attacker who compromised the account — immediately changes what all users install or see. There is no versioning barrier between a bad commit and all active users.

**Fix:** Cut a `v1.1.0` git tag and pin all refs to it:

```json
// marketplace.json
"source": { "ref": "v1.1.0" }
```

```bash
# README / marketplace.json raw URLs
https://raw.githubusercontent.com/mzuleta6/espresso/v1.1.0/SKILL.md
```

Update refs only on intentional releases, not on every commit to `main`.

---

### M4 — `marketplace.json` manual install places SKILL.md at wrong path

**File:** `marketplace.json` · line 22

```bash
mkdir -p ~/.claude/plugins/espresso && \
  curl -o ~/.claude/plugins/espresso/SKILL.md ...
```

`plugin.json` declares the skill at `skills/espresso/SKILL.md` (relative to the plugin root). The manual command drops SKILL.md directly into `~/.claude/plugins/espresso/`, producing a flat structure that doesn't match the manifest. The skill will not load. The README has the correct path (`skills/espresso/SKILL.md` subdirectory).

**Fix:**

```json
"manual": "mkdir -p ~/.claude/plugins/espresso/skills/espresso && curl -o ~/.claude/plugins/espresso/skills/espresso/SKILL.md https://raw.githubusercontent.com/mzuleta6/espresso/v1.1.0/SKILL.md && curl -o ~/.claude/plugins/espresso/plugin.json https://raw.githubusercontent.com/mzuleta6/espresso/v1.1.0/plugin.json"
```

---

### M5 — `validate.yml` plugin.json sync check is non-blocking

**File:** `.github/workflows/validate.yml` · lines 58–64

The step that compares both `plugin.json` copies emits a `WARN` and does not call `exit 1`:

```bash
echo "WARN: plugin.json copies differ in shared fields (non-blocking)."
```

If the two files diverge, CI passes. This is the only consistency check that silently succeeds on failure. Combined with the fact that the workflow is currently untracked, this means plugin.json divergence would never be caught.

**Fix:** Change the conditional to fail on divergence:

```bash
if [ "$ROOT" = "$NESTED" ]; then
  echo "plugin.json shared fields are in sync."
else
  echo "FAIL: plugin.json copies have diverged." && exit 1
fi
```

---

## Low

### L1 — `ISSUE_TEMPLATE.MD` untracked and wrong filename extension

**File:** `.github/ISSUE_TEMPLATE.MD` · (`git status: ??` — untracked)

The file uses an uppercase `.MD` extension. GitHub's issue template system recognizes `.md` (lowercase) only. The file will be ignored even after being committed. It also needs to be committed first (see H1).

**Fix:** Rename and commit:

```bash
git mv .github/ISSUE_TEMPLATE.MD .github/ISSUE_TEMPLATE.md
git add .github/ISSUE_TEMPLATE.md
```

---

### L2 — Workflow has no explicit `permissions` block

**File:** `.github/workflows/validate.yml` · (no `permissions` key)

Without an explicit `permissions` block, the job inherits the repository's default token permissions. For public repos this is typically `read` across all scopes, which is acceptable here since the workflow doesn't use `GITHUB_TOKEN`. However, omitting it means the behavior depends on repository settings that can be changed externally. Explicit minimal permissions are defensive.

**Fix:** Add to the job or top level:

```yaml
permissions:
  contents: read
```

---

### L3 — CODEOWNERS does not explicitly cover `.github/workflows/`

**File:** `.github/CODEOWNERS` · line 2

The `* @mzuleta6` wildcard covers all files including workflow files, so there is no gap in ownership coverage. However, `.github/workflows/` is the highest-risk directory in any repo — a malicious workflow can exfiltrate secrets, run arbitrary code, and modify the repo. An explicit rule makes the protection intent visible to reviewers and tooling.

**Fix:** Add:

```
.github/workflows/ @mzuleta6
```

---

### L4 — No `SECURITY.md` / responsible disclosure policy

**Files:** `SECURITY.md` (absent) · `.github/SECURITY.md` (absent)

GitHub displays a "Report a vulnerability" button and links to `SECURITY.md` when it exists. For a public plugin that runs code inside user Claude sessions, a disclosure path matters. Without it, security reporters have no channel other than opening a public issue, which would expose a vulnerability before it's fixed.

**Fix:** Create `.github/SECURITY.md`:

```markdown
# Security Policy

## Supported versions
Only the latest release (currently 1.1.0) receives security fixes.

## Reporting a vulnerability
Do **not** open a public issue for security vulnerabilities.
Email: [your address] or use GitHub's private vulnerability reporting.
Expected response: within 72 hours.
```

Also enable private vulnerability reporting: GitHub → Settings → Security → Private vulnerability reporting → Enable.

---

### L5 — No `PULL_REQUEST_TEMPLATE.md`

**File:** `.github/PULL_REQUEST_TEMPLATE.md` (absent)

Without a PR template, contributors don't have a structured checklist. For a plugin where the primary concern is SKILL.md content integrity, a template that requires reviewers to verify they've read the SKILL diff adds a meaningful human gate.

**Fix:** Create `.github/PULL_REQUEST_TEMPLATE.md` with a checklist that includes: SKILL.md change description, whether `always: true` behavior is affected, whether both copies are in sync, and version bump if applicable.

---

### L6 — CHANGELOG mentions "Plan mode rules" not implemented in SKILL.md

**File:** `CHANGELOG.md` · line 8

The 1.1.0 entry lists "Plan mode rules" as added. Neither `SKILL.md` copy contains any plan mode section. This is either a dropped feature or premature changelog entry.

**Fix:** Add the plan mode section to `SKILL.md` and sync both copies, or remove the entry from the changelog. Stale changelogs erode trust for external users evaluating the plugin.

---

### L7 — "Ejecutar sin preguntar permiso" — skip-confirmation instruction

**File:** `SKILL.md` · line 41 · `plugins/espresso/skills/espresso/SKILL.md` · line 41

The EXCEPCION TOTAL section instructs Claude to execute artifact generation "sin preguntar permiso" (without asking permission). The explicit safety carve-outs below it (irreversible ops, production, security) limit the blast radius. This is not a jailbreak pattern — it's a UX optimization. The concern is that users who install via `always: true` without reading the SKILL may not realize confirmation is being suppressed.

**Fix:** Tighten the language to scope the exception explicitly:

```
Para artifacts: generar y presentar directamente sin pedir confirmación
de diseño o layout. Las excepciones críticas aplican igual.
```

---

### L8 — `validate.yml` only validates one skill path from root `plugin.json`

**File:** `.github/workflows/validate.yml` · lines 30–47

The step reads only `skills[0]` (the first skill). If additional skills are added to the plugin, their paths won't be checked. This is a latent gap rather than an active vulnerability.

**Fix:** Loop over all declared skills:

```python
import json, sys
with open('plugin.json') as f:
    d = json.load(f)
for skill in d.get('skills', []):
    path = skill['path']
    import os
    if not os.path.isfile(path):
        print(f"FAIL: {path} does not exist")
        sys.exit(1)
```

---

## Clean — no issues found

| Area | Status | Notes |
|---|---|---|
| SKILL.md — prompt injection | Clean | No adversarial patterns. Behavioral rules are restrictive (remove filler), not permissive. |
| SKILL.md — jailbreak language | Clean | No attempts to override safety guidelines. Safety carve-outs are explicit and correct. |
| SKILL.md — overly broad grants | Clean | `always: true` is documented plugin behavior, not a hidden permission. |
| Secrets in git history | Clean | Full history scanned (5 commits). No tokens, API keys, or credentials in any diff. |
| `plugin.json` unexpected fields | Clean | Both copies contain only expected fields. No external script URLs. |
| `marketplace.json` unexpected fields | Clean | All 12 fields are standard. No unexpected callbacks or script references. |
| Both `SKILL.md` copies in sync | Clean | Byte-identical (`diff` exits 0). |
| Both `plugin.json` copies in sync | Clean | Byte-identical. |
| Version numbers consistent | Clean | `1.1.0` across SKILL.md (×2), plugin.json (×2), marketplace.json, README badge, examples (×2), CHANGELOG. |
| `banner.svg` | Clean | Static SVG, no embedded scripts, no external resource references. |
| `ISSUE_TEMPLATE.MD` content | Clean | Legitimate bug report template. No injection surface. |
| `always: true` implications | Clean | Activates the skill automatically. Behavior is fully documented in SKILL.md and README. |

---

## Remediation priority

| Priority | Action |
|---|---|
| Immediate | Commit and push `validate.yml` (H1) |
| Immediate | Enable branch protection on `main` with required reviews + CI (H2) |
| Before next release | Pin `actions/checkout` to commit SHA (M1) |
| Before next release | Fix root `plugin.json` path or delete it (M2) |
| Before next release | Pin all install refs to `v1.1.0` tag (M3) |
| Before next release | Fix `marketplace.json` manual install path (M4) |
| Before next release | Make plugin.json sync check blocking (M5) |
| Soon | Commit `ISSUE_TEMPLATE.md` with correct lowercase extension (L1) |
| Soon | Add `SECURITY.md` and enable private vulnerability reporting (L4) |
| Soon | Add explicit `permissions: contents: read` to workflow (L2) |
| Backlog | Add `PULL_REQUEST_TEMPLATE.md` (L5) |
| Backlog | Resolve CHANGELOG / SKILL.md "Plan mode rules" discrepancy (L6) |
| Backlog | Tighten artifact exception phrasing (L7) |
