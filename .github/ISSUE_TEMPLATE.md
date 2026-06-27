---
name: espresso issue
about: Report unexpected verbosity, broken behavior, or suggest improvements
title: "[TYPE] brief description"
labels: ""
assignees: ""
---

## Type

- [ ] Claude ignored espresso and added hedging/filler
- [ ] Technical content was incorrectly truncated or paraphrased
- [ ] Wrong level behavior (lite / full / ultra)
- [ ] Artifact exception not applied correctly
- [ ] Language mixing (responding in wrong language)
- [ ] Critical exception not triggered when it should have been
- [ ] Suggestion / improvement

---

## Claude version & model

- Model: `claude-sonnet-4-6` / `claude-opus-4-6` / other: ___
- Claude Code version: ___
- espresso version: `1.1.0` / other: ___

---

## Reproduction

**Prompt sent:**
```
[paste exact prompt]
```

**What Claude responded:**
```
[paste exact response]
```

**What espresso should have produced:**
```
[what you expected]
```

---

## Context

- Active level (`/espresso lite` / `/espresso full` / `/espresso ultra`): ___
- Was this inside an artifact context? Yes / No
- Was this a critical exception context (production / security / irreversible)? Yes / No

---

## Additional notes

<!-- Anything else that helps reproduce or understand the issue -->