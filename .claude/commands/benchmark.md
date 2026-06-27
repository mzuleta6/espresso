# espresso benchmark

Run an in-session A/B benchmark comparing responses with and without espresso active.
No external API calls. No extra cost. Uses this Claude Code session directly.

## Flow

For each of the 3 questions below, run two separate responses in sequence:

**Round A — baseline:**
1. Run `/normal` to deactivate espresso
2. Ask the question
3. Record the response character count

**Round B — with espresso:**
1. Run `/espresso full` to reactivate espresso
2. Ask the same question
3. Record the response character count

After all 3 questions, reactivate `/espresso full` permanently.

## Questions

1. "Why does my React component re-render on every parent render even when props haven't changed?"
2. "TypeScript error TS2345: Type 'string | undefined' is not assignable to type 'string'. Fix?"
3. "How do I handle form validation errors in a Next.js Server Action?"

## Metrics per question

- `baseline_chars`: character count of response without espresso
- `espresso_chars`: character count of response with espresso
- `reduction_pct`: Math.round((1 - espresso_chars / baseline_chars) * 100)
- `label`: short version of the question for display (max 6 words)

## Output — React artifact

Generate a beautiful, fully interactive React artifact using the frontend-design skill.

**Header:**
- Large "☕" icon
- Title: "espresso benchmark"
- Subtitle: "in-session A/B · 3 questions · zero extra cost"

**Results — one card per question:**
- Question label (short, human-readable)
- Two horizontal bars side by side:
  - Bar A: gray, labeled "without espresso", width proportional to baseline_chars
  - Bar B: amber (#BA7517), labeled "with espresso", width proportional to espresso_chars
- Reduction percentage large and prominent between the bars
- Character counts below each bar in muted text

**Summary section:**
- 3 metric cards:
  1. Average reduction — large % in amber
  2. Characters saved total — number in green
  3. Human label: "That's X fewer words per answer on average" — calculated as Math.round(total_chars_saved / 3 / 5)
- All cards use CSS variables, amber hardcoded only as specified

**Design rules:**
- CSS variables only except amber: #BA7517 fill, #FAEEDA bg, #EF9F27 border, #854F0B text
- Tabler icons outline (ti-*)
- Smooth bar animations on load (width transition 0.6s ease)
- Responsive down to 400px
- No localStorage, no external calls
- Every state styled: loading per question, complete, summary revealed

**Footer:**
- "Run again" button in amber
- Small disclaimer: "Measured by response length · same session · same model"

## After the artifact

### 1. Save results

Save to `docs/benchmark-results.json`:
```json
{
  "generated": "<ISO date>",
  "model": "claude-sonnet-4-6",
  "method": "in-session character count",
  "results": [
    {
      "question": "<label>",
      "baseline_chars": 0,
      "espresso_chars": 0,
      "reduction_pct": 0
    }
  ],
  "summary": {
    "avg_reduction_pct": 0,
    "total_chars_saved": 0,
    "avg_words_saved_per_answer": 0
  }
}
```

### 2. Screenshot instructions

Tell the user exactly this after generating the artifact:

---
**Para agregar el benchmark al README:**

1. Toma un screenshot del artifact (Mac: Cmd+Shift+4)
2. Guárdalo como `docs/benchmark-preview.png`
3. Corre en terminal:
```bash
git add docs/
git commit -m "docs: add benchmark results $(date +%Y-%m-%d)"
git push origin main
```
---

### 3. Update README.md

Replace the existing benchmark or Before/After section with:

```markdown
## Benchmark

> Last run: <date> · Model: claude-sonnet-4-6 · Method: in-session A/B

![reduction](https://img.shields.io/badge/avg_reduction-XX%25-BA7517?style=flat-square&labelColor=1c1917)
![saved](https://img.shields.io/badge/chars_saved-XXXX-22c55e?style=flat-square&labelColor=1c1917)
![words](https://img.shields.io/badge/fewer_words_per_answer-XX-57534e?style=flat-square&labelColor=1c1917)

![benchmark preview](docs/benchmark-preview.png)
```

Replace XX values with the actual numbers from the run.