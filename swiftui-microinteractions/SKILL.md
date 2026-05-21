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

**Glass surfaces:** `.ultraThinMaterial` or `.thickMaterial` clipped to `Capsule()` · track background `.white.opacity(0.06)` + 1pt stroke `.white.opacity(0.08)`

**Opacity levels:** ghost `0.06–0.08` · subtle `0.12` · inactive `0.3` · secondary `0.5` · active `0.8–0.9` · full `1.0`

**Colors:** white + opacity dominant · cyan `Color(red: 0.45, green: 0.65, blue: 1.0)` · green `Color(red: 0.55, green: 0.95, blue: 0.75)` · never `.blue/.green/.red` on dark bg

**Gradients:** two-tone only · blob fill `[.white, Color(white: 0.88)]` · progress `[cyan, green]`

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

**File layout (mandatory MARK order):**
```
// MARK: - Model
// MARK: - Main View   (tokens → @State → body → subviews → gesture → actions)
// MARK: - Supporting Shapes
// MARK: - Preview
```

---

## Output (Create mode)

Stream these progress lines one by one:

```
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

```swift
DemoItem(
    row: RowView(icon: "💧", title: "Title Here", desc: "Feature · feature · feature"),
    destination: wrappedDestination { YourView() },
    date: "May 21, 2026",
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
