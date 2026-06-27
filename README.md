<div align="center">

<img src=".github/banner.svg" alt="espresso — Claude without the noise." width="800"/>

<br/>

# espresso

![version](https://img.shields.io/badge/version-1.1.0-BA7517?style=flat-square&labelColor=1c1917)
![license](https://img.shields.io/badge/license-MIT-57534e?style=flat-square&labelColor=1c1917)
![works with](https://img.shields.io/badge/works_with-Claude_Code-f5f5f4?style=flat-square&labelColor=1c1917)

*Claude without the noise.*

</div>

---

## The problem

I was building a form in Next.js. The mutation was failing silently. I asked Claude:

> "Why is my Server Action not throwing when Prisma fails?"

Claude responded with 4 sentences of setup — explaining what a Server Action is. Then 2 sentences of hedging: *"this might be because..."*, *"it's possible that..."*. Then a suggestion to wrap things in try/catch — which I already had. Then *"I hope this helps! Let me know if you'd like me to elaborate."*

The actual answer — **errors in Server Actions are swallowed unless you rethrow** — was sentence 6 of 9. I wasted 30 seconds scanning noise to find the signal.

Then I installed espresso.

Same question. Claude responded:

```ts
// Server Actions swallow errors by default.
// In Next.js 14+, errors inside "use server" functions are caught
// by the framework and returned as ErrorBoundary triggers, not thrown.
// Rethrow explicitly to surface them:

export async function updateUser(data: FormData) {
  "use server"
  try {
    await prisma.user.update({ where: { id: data.get("id") as string }, data: { name: data.get("name") as string } })
  } catch (err) {
    throw new Error(`Failed to update user: ${err instanceof Error ? err.message : String(err)}`)
  }
}
```

Sentence 1 was the answer. No hedging. No re-explaining what a Server Action is. No "I hope this helps."

That's what espresso does.

---

## Before / After

| Scenario | Without espresso | With espresso | Saved |
|---|---|---|---|
| **React re-render** — why is my component re-rendering on every parent render? | 214 tokens · 9 sentences · 2 were the answer | 91 tokens · answer first | **−57%** |
| **TypeScript TS2345** — `string \| undefined` not assignable to `string` | 231 tokens · "that's a great question" opener | 78 tokens · 3 concrete fixes | **−66%** |
| **Server Actions** — form validation with `useActionState` | 246 tokens · pseudocode · "would you like an example?" | 198 tokens · complete implementation | **−19%** |

> espresso never reduces code. If the baseline only had prose and the correct response has code, espresso generates more tokens — because the correct response is more useful, not less. Noise reduction ≠ answer reduction.

---

## Install

### Option A — Marketplace (one command)

```bash
/plugin marketplace add mzuleta6/espresso
```

### Option B — Manual

```bash
# Create the plugin directory
mkdir -p ~/.claude/plugins/espresso/skills/espresso

# Download the skill
curl -o ~/.claude/plugins/espresso/skills/espresso/SKILL.md \
  https://raw.githubusercontent.com/mzuleta6/espresso/v1.1.0/SKILL.md

# Download the plugin manifest
curl -o ~/.claude/plugins/espresso/plugin.json \
  https://raw.githubusercontent.com/mzuleta6/espresso/v1.1.0/plugin.json
```

Then restart Claude Code. espresso activates automatically (`always: true`).

---

## Levels

| Command | Level | What changes |
|---|---|---|
| `/espresso lite` | **Lite** | Removes cortesías and meta-comments. Keeps some context for complex topics. |
| `/espresso full` | **Full** *(default)* | Removes all 7 noise categories. Direct, precise responses. |
| `/espresso ultra` | **Ultra** | Full + removes unsolicited explanations, unprompted examples, unrequested alternatives. |
| `/normal` | **Off** | Deactivates espresso completely. Standard Claude behavior. |

Levels persist for the conversation until changed.

---

## What never changes

These are **never** truncated, paraphrased, or shortened:

- **Code blocks** — complete, no `// ... rest is the same`
- **Error messages and stack traces** — verbatim
- **API names and endpoints** — exact casing, exact paths
- **CLI commands** — all flags, full paths when they matter
- **TypeScript types** — complete interfaces, generics, union types
- **React hooks** — full implementations with correct dependency arrays
- **Tailwind classes** — full class lists, never collapsed
- **File paths and URLs** — exact, never shortened

---

## Artifacts are fully exempt

Components, dashboards, visualizations, animations — all of them.

Artifacts are deliverables, not illustrations. You use them directly. Asking Claude to "build a data table with sorting and filtering" and getting back a skeleton with `// TODO: add sorting` is not a token optimization — it's a broken deliverable.

espresso's brevity rules apply to prose. Artifacts get full design treatment: elaborate layouts, rich color palettes, micro-interactions, hover/focus/loading/empty/error states, complete CSS, and intentional typography. Tokens in artifacts are investment, not waste.

---

## Stack

![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=flat-square&logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)

Built and tested against a React/Next.js/TypeScript/Tailwind stack. Works with any stack — the noise elimination rules are language-agnostic.

---

## Contributing

Found a case where espresso didn't eliminate noise it should have — or eliminated content it shouldn't? [Open an issue](.github/ISSUE_TEMPLATE.md) with the exact prompt and response.

---

<div align="center">

MIT © 2026 [mzuleta6](https://github.com/mzuleta6)

</div>