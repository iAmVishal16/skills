# Changelog

All notable changes to `swiftui-microinteractions` are documented here.

Format: `[version] — date — summary`

---

## [1.3.0] — 2026-05-30

Learnings from building GlassMorphActionView — an iOS 26 capsule ↔ red circle morphing toggle.

- **`GlassEffectContainer` morphing clusters**: documented pattern for fusing/separating multiple glass elements with metaball physics; uses `@Namespace` + `.glassEffectID()` for identity across state changes
- **Fusion-threshold spacing trap**: `GlassEffectContainer(spacing:)` is the fusion threshold — layout spacing between elements MUST exceed it or shapes blob together permanently at rest. Rule: `layoutSpacing > containerSpacing + 6pt`
- **Tinted glass**: `.glassEffect(.regular.tint(color), in: shape)` for colored states — never layer a colored shape on top of glass
- **Edge distortion trap**: never apply `.scaleEffect` with `.leading`/`.trailing` anchors to individual glass elements (subpixel artifacts distort edges even at scale 1.0). Apply container-wide stretch with `.center` anchor instead
- **3-phase rubber-band toggle**: pre-stretch (0.28/0.55) → morph (0.55/0.72) → release with overshoot (0.45/0.55) — stacked with `asyncAfter` for tactile rubber feel
- **Light gradient backgrounds for glass demos**: dark backgrounds hide refraction — use `LinearGradient([white:0.97, white:0.88])` to showcase iOS 26 glass
- **No redundant `.clipShape`** on top of `.glassEffect(in:)` — glassEffect already clips

---

## [1.2.0] — 2026-05-27

- **SF Symbol replace transition**: `.contentTransition(.symbolEffect(.replace.downUp))` pattern for toggle buttons (more ↔ close, play ↔ pause). Button stays visible — icon morphs in place. Never hide/show a separate close button.
- Version output bumped to `⚙️ swiftui-microinteractions v1.2.0`

---

## [1.1.0] — 2026-05-26

Improvements from real-world LiquidGlassTabBar development.

- **iOS 26 Liquid Glass**: `glassShape(radius:)` `@ViewBuilder` helper — `.glassEffect(in:)` on iOS 26+, `.ultraThinMaterial` fallback on older OS
- **ZStack overflow trap**: documented rule — conditionally render tall children + add `.clipped()`; `.opacity(0)` alone does NOT prevent frame overflow
- **Sliding tab indicator**: `GeometryReader` + `offset(x: tabW * selectedIndex)` pattern with exact fill/padding/radius spec matching iOS 26 HIG
- **Active indicator fidelity**: `Color.white.opacity(0.14)` solid fill, 5pt inset, corner radius = outerRadius − 5 — matches Apple Music / iOS 26 reference
- **Multi-component bar layout**: `HStack` of independent pill + circle components, not one monolithic ZStack
- **ContentView date**: use today's actual date in snippet, not hardcoded past date
- **Version line**: bumped to `⚙️ swiftui-microinteractions v1.1.0`

---

## [1.0.0] — 2026-05-21

Initial public release.

- Create mode: generates complete SwiftUI animation files from plain-English prompts
- Edit mode: reads existing file, applies targeted change, rewrites in place
- Asset scanning: detects `.imageset` folders in `Assets.xcassets` before generating
- Haptics check: uses shared `HapticFeedback.swift` if present, falls back to inline UIKit calls
- Xcode registration: auto-adds generated file to `.pbxproj` (PBXFileReference, PBXBuildFile, group child, Sources phase)
- Spring preset table: 7 presets mapped from feel words to exact `response`/`dampingFraction` values
- Progress output: streams `⚙️ v1.0.0` + 6 status lines so long generations feel responsive
- Supports 3 prompt levels: plain English · English + tech hints · fully technical
