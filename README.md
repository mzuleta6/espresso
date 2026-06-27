<div align="center">

<img src=".github/banner.svg" alt="espresso" width="800"/>

# espresso

![version](https://img.shields.io/badge/version-1.1.0-BA7517?style=flat-square&labelColor=1c1917)
![license](https://img.shields.io/badge/license-MIT-57534e?style=flat-square&labelColor=1c1917)
![works with](https://img.shields.io/badge/works_with-Claude_Code-f5f5f4?style=flat-square&labelColor=1c1917)

*Claude without the noise.*

</div>

---

## Install

### Option A — Marketplace
```bash
/plugin marketplace add mzuleta6/espresso
```

### Option B — Manual
```bash
mkdir -p ~/.claude/plugins/espresso/skills/espresso
curl -o ~/.claude/plugins/espresso/skills/espresso/SKILL.md \
  https://raw.githubusercontent.com/mzuleta6/espresso/main/SKILL.md
curl -o ~/.claude/plugins/espresso/plugin.json \
  https://raw.githubusercontent.com/mzuleta6/espresso/main/plugin.json
```

---

MIT © 2026 [mzuleta6](https://github.com/mzuleta6)