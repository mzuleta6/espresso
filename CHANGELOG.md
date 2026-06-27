# Changelog

All notable changes to espresso are documented in this file.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)  
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

---

## [1.1.0] — 2026-06-26

### Added
- **EXCEPCION TOTAL for Artifacts**: All React components, HTML widgets, SVGs, dashboards, animations, and visualizations are now fully exempt from every brevity instruction. Artifacts execute without asking permission and receive full design treatment — elaborate layouts, rich palettes, micro-interactions, complete CSS.
- **`/espresso ultra` level**: Strips all unsolicited explanations, unprompted examples, and unrequested alternatives. Returns only the exact answer requested.
- **Narration of tool calls** added to the elimination list — Claude no longer announces what it's about to do before doing it.
- **Transition vacías** (e.g., "Dicho esto,", "Con eso en mente,") added to the elimination list.
- **Plan mode rules**: Plans are always fresh, never reference previous plans, always start with `## PLAN:` header.
- **Multi-language enforcement**: Response language now strictly follows the user's message language, never mixed.

### Changed
- `always: true` frontmatter now set by default — espresso activates for every conversation, no explicit trigger needed.
- Skill description updated to emphasize the core invariant: precision is never sacrificed for brevity.
- Levels table reordered: `full` is now explicitly labeled as the default.

### Fixed
- Clarified that `/normal` fully deactivates espresso (was ambiguous in v1.0.0 — it now means complete off, not just lite).
- Critical exceptions section now explicitly lists all four categories: irreversible ops, production changes, security, confused user.

---

## [1.0.0] — 2026-05-01

### Added
- Initial release of espresso skill.
- Core elimination rules: cortesías, hedging, relleno, re-contexto, meta-comentarios.
- Protection rules: code blocks, errors, APIs, CLI, TypeScript types, React hooks, Tailwind classes, paths, URLs are never truncated or paraphrased.
- Two levels: `/espresso lite` and `/espresso full`.
- Critical exceptions: irreversible operations, production changes, security contexts.
- Language enforcement: respond in the user's language.
- `plugin.json` with `always: true` activation.
- MIT license.
- README with before/after examples.
- `examples/basic.md` with 3 real comparisons.

[1.1.0]: https://github.com/mzuleta6/espresso/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/mzuleta6/espresso/releases/tag/v1.0.0