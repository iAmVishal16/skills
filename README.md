# Skills by Vishal Paliwal

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![skills.sh](https://skills.sh/b/iAmVishal16/skills)](https://skills.sh/iAmVishal16/skills/swiftui-microinteractions)
[![GitHub Stars](https://img.shields.io/github/stars/iAmVishal16/skills?style=flat)](https://github.com/iAmVishal16/skills/stargazers)

Premium SwiftUI animation and interaction skills for AI coding agents — generate production-ready micro-interactions from plain English prompts.

Built from [legendary-Animo](https://github.com/iAmVishal16/legendary-Animo): 84 hand-crafted SwiftUI animation demos.

---

## swiftui-microinteractions

Generate premium SwiftUI animations in the legendary-Animo style — spring physics, CoreHaptics, glass morphism, and complete compilable files — from a plain English description.

```
/swiftui-microinteractions iOS toggle but the track floods with liquid when switched
/swiftui-microinteractions a notification card you can rip apart by pulling both edges
/swiftui-microinteractions edit ViscousButtonView.swift — increase tear threshold to 160pt
```

Each prompt writes a complete `.swift` file directly to your project. Supports both create and edit modes.

---

## Who This Is For

- iOS developers who want premium micro-interactions without spending days on physics tuning
- Designers prototyping gesture-driven interactions in SwiftUI
- Teams who want consistent animation quality across their app
- Anyone who wants to ship the kind of interactions that make users say "how did they do that?"

---

## Installation

### Option A: skills.sh CLI

```bash
npx skills add iAmVishal16/skills
```

Then use in your agent:
```
/swiftui-microinteractions iOS toggle but the track floods with liquid when switched
```

[View on skills.sh →](https://skills.sh/iAmVishal16/skills/swiftui-microinteractions)

### Option B: Claude Code Plugin

**Add to your `.claude/settings.json`:**

```json
{
  "enabledPlugins": {
    "swiftui-microinteractions@iamvishal16-skills": true
  },
  "extraKnownMarketplaces": {
    "iamvishal16-skills": {
      "source": {
        "source": "github",
        "repo": "iAmVishal16/skills"
      }
    }
  }
}
```

**Or via Claude Code CLI:**
```
/plugin marketplace add iAmVishal16/skills
/plugin install swiftui-microinteractions@iamvishal16-skills
```

### Option C: Manual Install

```bash
curl -o ~/.claude/commands/swiftui-microinteractions.md \
  https://raw.githubusercontent.com/iAmVishal16/skills/main/swiftui-microinteractions/SKILL.md
```

Skill is then available as `/swiftui-microinteractions` in any Claude Code session.

---

## What's Inside

The skill encodes the full legendary-Animo aesthetic without requiring knowledge of the codebase:

- **Spring physics library** — 7 tuned presets (snap, pop, settle, morph, stiff, dial) with exact `response` + `dampingFraction` values
- **Haptic grammar** — 4-event ladder tied to interaction phases (drag start, threshold cross, commit, destroy)
- **Visual DNA** — dark background, glass morphism surfaces, 7-level opacity hierarchy, two-tone gradient system
- **Liquid metaball pattern** — `Canvas` + `.blur()` + `.contrast()` + `.blendMode(.screen)` recipe
- **Multi-phase animation chains** — stacked `DispatchQueue.main.asyncAfter` for choreographed sequences
- **State architecture tiers** — Simple (2–4 `@State`) / Medium (5–8) / Complex (10+) with property type rules
- **Code structure law** — mandatory `MARK` layout, camelCase tokens, no magic numbers
- **Create + Edit modes** — generates new files or modifies existing ones, writes directly to disk

---

## Skill Structure

```
swiftui-microinteractions/
  SKILL.md
```

---

## Contributing

Contributions welcome. Open a PR to improve the skill content, add new animation patterns, or fix incorrect physics values.

---

## About

Built by [Vishal Paliwal](https://twitter.com/iamvishal16_ios) — iOS developer and creator of [legendary-Animo](https://github.com/iAmVishal16/legendary-Animo).

Support the work: [Patreon](https://www.patreon.com/c/iamvishal16)

---

## License

MIT License. See [LICENSE](LICENSE) for details.
