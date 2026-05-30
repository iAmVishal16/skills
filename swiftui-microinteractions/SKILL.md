---
name: swiftui-microinteractions
description: Generate premium SwiftUI animations in the legendary-Animo style — spring physics, CoreHaptics, glass morphism, complete compilable files.
---

Generate a complete, compilable SwiftUI animation file in the legendary-Animo style. No placeholders. No TODO comments.

## When to Use
- Building a SwiftUI button, gesture, slider, or liquid effect with premium physics
- Need haptic feedback timed correctly to animation phases
- Creating a drag interaction with resistance, snap, or threshold trigger
- Editing an existing SwiftUI animation file

## Mode Detection
- **Edit mode**: prompt contains "edit", "update", "add X to", "change", or a `.swift` filename → read file first, change only what's asked, overwrite file on disk
- **Create mode**: everything else → generate a new complete file and write it to disk

---

## Asset Scanning (run before generating any image-dependent animation)

If the animation involves images, photos, or cards, scan for existing assets first:

1. Check these paths in order:
   - `legendary-Animo/Res/Assets.xcassets/Images/`
   - `Assets.xcassets/Images/`
   - `Assets.xcassets/`

2. List `.imageset` folder names — strip `.imageset` suffix to get the `Image("name")` string

3. **If found** → use them directly. Print:
   ```
   🖼️  Assets found: photo1, photo2 … (using in file)
   ```

4. **If not found** → use `Image(systemName: "photo.fill")` in a colored `RoundedRectangle`. Include Asset Notes at the end.

Never invent asset names. Only reference names confirmed to exist on disk.

---

## Haptics — check project first

Before including haptic code, check if `HapticFeedback.swift` exists anywhere in the project:
- **If found** → call `HapticFeedback.lightImpact()` etc. directly. Do NOT re-declare the struct.
- **If not found** → call `UIImpactFeedbackGenerator` / `UISelectionFeedbackGenerator` directly inline, and add `import UIKit` at the top.

---

## Style Rules

**Backgrounds:** `Color(white: 0.06).ignoresSafeArea()` standalone · `Color("BgColor").ignoresSafeArea()` in-project

**Glass surfaces (pre-iOS 26):** `.ultraThinMaterial` or `.thickMaterial` clipped to shape · track background `.white.opacity(0.06)` + 1pt stroke `.white.opacity(0.08)`

**Opacity levels:** ghost `0.06–0.08` · subtle `0.12` · inactive `0.3` · secondary `0.5` · active `0.8–0.9` · full `1.0`

**Colors:** white + opacity dominant · cyan `Color(red: 0.45, green: 0.65, blue: 1.0)` · green `Color(red: 0.55, green: 0.95, blue: 0.75)` · never `.blue/.green/.red` on dark bg

**Gradients:** two-tone only · blob fill `[.white, Color(white: 0.88)]` · progress `[cyan, green]`

---

## iOS 26 Liquid Glass

Use `.glassEffect()` on iOS 26+, fall back to `.ultraThinMaterial` on older OS. Always wrap in a `@ViewBuilder` helper so both paths share the same call site:

```swift
@ViewBuilder
private func glassShape(radius: CGFloat) -> some View {
    if #available(iOS 26.0, *) {
        RoundedRectangle(cornerRadius: radius, style: .continuous)
            .fill(.clear)
            .glassEffect(in: RoundedRectangle(cornerRadius: radius, style: .continuous))
    } else {
        RoundedRectangle(cornerRadius: radius, style: .continuous)
            .fill(.ultraThinMaterial)
            .overlay(
                RoundedRectangle(cornerRadius: radius, style: .continuous)
                    .strokeBorder(Color.white.opacity(0.12), lineWidth: 1)
            )
    }
}
```

Apply to any shape — pill, capsule, circle. Never hardcode `.ultraThinMaterial` for new components when iOS 26 is a target.

### Tinted glass (colored states)

For destructive / accent buttons, use the built-in tint API — never layer a colored shape on top of glass:

```swift
.glassEffect(.regular.tint(dangerColor), in: Circle())
```

Danger red: `Color(red: 0.87, green: 0.32, blue: 0.42)` · Accent blue: `Color(red: 0.45, green: 0.65, blue: 1.0)`.

### `GlassEffectContainer` — morphing clusters

When multiple glass elements should fuse and separate with the iOS 26 metaball effect (e.g. a capsule that morphs into a separate circle), wrap them in a `GlassEffectContainer`:

```swift
@Namespace private var glassNS

GlassEffectContainer(spacing: 18) {
    HStack(spacing: 28) {                 // see spacing trap below
        capsule
            .glassEffect(in: Capsule())
            .glassEffectID("capsule", in: glassNS)

        if isExpanded {
            circle
                .glassEffect(.regular.tint(.red), in: Circle())
                .glassEffectID("circle", in: glassNS)
        }
    }
}
```

**Fusion-threshold spacing trap** — `GlassEffectContainer(spacing:)` is the fusion threshold, not visual padding. Any two glass elements within that distance will visually blob together permanently, even at rest. Always make the layout spacing **greater than** `containerSpacing`:

| containerSpacing | HStack spacing | Result |
|---|---|---|
| 18 | 10 | ❌ Permanent fusion — edges distorted at rest |
| 18 | 28 | ✅ Clean shapes at rest, metaball morph only during transition |

Rule: `layoutSpacing > containerSpacing + 6pt` for safe margin.

### Edge distortion trap

Never apply `.scaleEffect(x:y:anchor:)` with offset anchors (`.leading` / `.trailing`) to **individual** glass elements — subpixel rendering at the anchor edge causes visible edge distortion even when scale = 1.0. Apply container-wide rubber stretch instead:

```swift
GlassEffectContainer(spacing: 18) { ... }
    .scaleEffect(x: 1.0 + stretch * 0.05, y: 1.0 - stretch * 0.025, anchor: .center)
```

Also: don't add redundant `.clipShape(Capsule())` on top of `.glassEffect(in: Capsule())` — `glassEffect` already clips.

### 3-phase rubber-band toggle

For glass elements that pop apart or fuse on tap, stack 3 animations to feel like real rubber:

```swift
private func toggle() {
    HapticFeedback.mediumImpact()

    // Phase 1 — pre-stretch (rubber tensions)
    withAnimation(.spring(response: 0.28, dampingFraction: 0.55)) {
        stretchAmount = 1.0
    }
    // Phase 2 — morph fires at peak tension
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.05) {
        withAnimation(.spring(response: 0.55, dampingFraction: 0.72)) {
            isExpanded.toggle()
        }
    }
    // Phase 3 — release snap-back with overshoot
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.28) {
        withAnimation(.spring(response: 0.45, dampingFraction: 0.55)) {
            stretchAmount = 0
        }
    }
}
```

### Light backgrounds for glass demos

To showcase iOS 26 glass effects, use a light gradient background — dark backgrounds hide the refraction. Standalone demo:

```swift
LinearGradient(colors: [Color(white: 0.97), Color(white: 0.88)],
               startPoint: .topLeading, endPoint: .bottomTrailing)
    .ignoresSafeArea()
```

---

## Spring Presets

| Feel | Value |
|---|---|
| Snap / bounce | `.spring(response: 0.35, dampingFraction: 0.5)` |
| UI pop | `.spring(response: 0.35, dampingFraction: 0.6)` |
| Standard settle | `.spring(response: 0.4, dampingFraction: 0.65)` |
| Physics settle | `.spring(response: 0.45, dampingFraction: 0.7)` |
| Slow morph | `.spring(response: 0.6, dampingFraction: 0.8)` |
| Precision stiff | `.interpolatingSpring(stiffness: 220, damping: 22)` |
| Dial / scrub | `.interactiveSpring(response: 0.3, dampingFraction: 0.7)` |

Never use bare `.spring()` — always explicit `response` + `dampingFraction`. If user supplies values, use them verbatim. Map feel words: "stretchy" → 0.4/0.5, "snappy" → 0.35/0.6, "melts" → 0.6/0.8.

---

## State & Code Rules

- Default: pure `@State` for all gesture tracking · `@GestureState` only if value must auto-reset
- Constants: camelCase `private let` at struct level (2–5 values) — never SCREAMING_SNAKE_CASE
- Liquid metaball: `Canvas` (black fill → white circles) + `.blur(20)` + `.contrast(50)` + `.blendMode(.screen)`
- Multi-phase sequences: stacked `DispatchQueue.main.asyncAfter` with overlapping delays

**ZStack frame trap — always apply both rules together:**
When a ZStack has a fixed `.frame(height:)` AND contains a `LazyVGrid`, `List`, or any tall component:
1. Conditionally render the tall child only when needed (`if isExpanded || childVisible`)
2. Add `.clipped()` to the ZStack

Relying on `.opacity(0)` alone does NOT prevent overflow — the component still renders outside the frame and changing `height` constants has no visual effect.

**Floating bar layout:** Build multi-component bars (pill + separate action button) as `HStack` of independent views, not one monolithic ZStack. Each component gets its own glass background and shadow.

**File layout (mandatory MARK order):**
```
// MARK: - Model
// MARK: - Main View   (tokens → @State → body → subviews → gesture → actions)
// MARK: - Supporting Shapes
// MARK: - Preview
```

---

## Tab Bar Patterns

**Sliding selection indicator** — use `GeometryReader` + `offset(x:)`, never manual position math:

```swift
GeometryReader { geo in
    let tabW = geo.size.width / CGFloat(tabCount)
    RoundedRectangle(cornerRadius: indicatorRadius, style: .continuous)
        .fill(Color.white.opacity(0.14))
        .padding(5)
        .frame(width: tabW)
        .offset(x: tabW * CGFloat(selectedIndex))
        .animation(.spring(response: 0.35, dampingFraction: 0.65), value: selectedIndex)
}
```

**Active indicator spec** (matches Apple Music / iOS 26 HIG):
- Fill: `Color.white.opacity(0.14)` — solid presence, clearly distinct from glass background
- Padding: `5pt` inset on all sides (fills full tab height minus 5pt each edge)
- Corner radius: `outerRadius - 5` (softer than the outer pill, independent value ~16–18pt)
- Width: exactly `pillWidth / tabCount` — crisp slot division, no guessing

**Tab cell layout** (icon + label, per HIG):
- Icon: 22pt, `.semibold` when active · `.regular` inactive
- Label: 10–11pt, `.semibold` when active · `.regular` inactive
- Both tinted with `activeColor` when selected · `white.opacity(0.5)` inactive
- `VStack(spacing: 4)` inside `.frame(maxWidth: .infinity).frame(height: barHeight)`

**SF Symbol replace transition** — for any button that toggles state and changes its icon (e.g. more → close, play → pause, add → done):

```swift
Image(systemName: isExpanded ? "xmark" : "ellipsis")
    .contentTransition(.symbolEffect(.replace.downUp))
    .animation(.spring(response: 0.35, dampingFraction: 0.6), value: isExpanded)
```

- `.replace.downUp` — old icon scales down, new scales up. Use for open/close toggles.
- `.replace.upUp` — both scale up. Use for sequential forward actions.
- The button action must toggle: `isExpanded ? collapse() : expand()` — never hide the button to show a separate close button.
- Available iOS 17+. No fallback needed for legendary-Animo (iOS 16+ minimum → gate with `if #available(iOS 17, *)` only if targeting iOS 16).

**Standard heights:** `barHeight = 49pt` (icon-only) · `barHeight = 68pt` (icon + label)

---

## Output (Create mode)

Stream these progress lines one by one:

```
⚙️  swiftui-microinteractions v1.3.0
🖼️  Assets: <found: name1, name2… · or · none found, using placeholders>
🎯  Archetype: <archetype name>
⚡  Physics: <spring preset and why — one phrase>
🎮  Haptics: <2–3 haptic moments>
🏗️  State: <tier> — <@State var names>
✍️  Writing <FileName>.swift…
```

**Step 1 — Write the file to disk:**
- Carousel → `legendary-Animo/Carousels/` if exists, else `Carousels/` if exists, else cwd
- Other → `legendary-Animo/Animations/` if exists, else `Animations/` if exists, else cwd

Print: `✅  Saved to <full relative path>`

---

**Step 2 — Xcode project registration:**

Check for a `.xcodeproj` file:
```bash
find . -maxdepth 3 -name "*.xcodeproj" -type d | head -1
```

**If `.xcodeproj` found** → register the file in `project.pbxproj` using this Python script:

```python
import uuid, re

pbxproj  = "<xcodeproj_path>/project.pbxproj"
filename = "<FileName>.swift"
filepath = "<full relative path written above>"
# Determine target group: "Carousels" if saved in Carousels/, else "Animations"
group_name = "Carousels"  # or "Animations"

FILE_REF   = uuid.uuid4().hex[:24].upper()
BUILD_FILE = uuid.uuid4().hex[:24].upper()

with open(pbxproj) as f:
    content = f.read()

# Find an existing file in the same group as an insertion anchor
# Look for the last .swift entry in PBXFileReference section for that group
# Insert PBXBuildFile, PBXFileReference, group child, and Sources build phase entry
# Use the same indentation and formatting as surrounding entries
```

Run the script with the actual values. Use an existing same-group file as the anchor for each insertion point. Print:

```
📦  Registered in Xcode project: <xcodeproj name>
    └─ PBXFileReference  ✓
    └─ PBXBuildFile      ✓
    └─ Group child       ✓
    └─ Sources phase     ✓
```

**If no `.xcodeproj` found** → print:

```
⚠️  No Xcode project found. Add the file manually:
    File path: <full path>
    In Xcode: File → Add Files to "<ProjectName>"
              or drag into the Navigator → check "Add to target"
```

---

**Step 3 — ContentView registration snippet** (```swift block, paste manually):

Use today's actual date for the `date:` field — never hardcode a past date.

```swift
DemoItem(
    row: RowView(icon: "💧", title: "Title Here", desc: "Feature · feature · feature"),
    destination: wrappedDestination { YourView() },
    date: "<today's date e.g. May 26, 2026>",
    hasDateHeader: true
)
```

**Step 4 — Asset notes** (only if placeholders were used):
```
ASSET NOTES:
- Add portrait images to Assets.xcassets/Images/ named: photo1, photo2…
  → Recommended size: 400×600pt, portrait ratio
- Remove placeholder RoundedRectangle once real assets are added
```

---

## Output (Edit mode)

Stream before editing:
```
📂  Reading <FileName>.swift…
✏️  Applying: <one-line summary of change>
✍️  Writing updated file…
```

Overwrite the file at its existing path, then print:
```
✅  Updated <full relative path>
```

Then output a **Changes** bullet list of what was modified and why.
(No pbxproj update needed for edits — file is already registered.)
