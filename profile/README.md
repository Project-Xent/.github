# Project Xent

**A pure-C UI layout engine — currently under active development.**  
[![License](https://img.shields.io/badge/License-0BSD-228B22.svg)](https://opensource.org/licenses/0BSD)
[![Standard](https://img.shields.io/badge/Standard-C23-00599C.svg)](https://www.iso.org/standard/82075.html)
[![ISPC](https://img.shields.io/badge/Parallel-ISPC-0071C5.svg)](https://ispc.github.io/)

---

## Overview

Project Xent is split into two repositories with a clear separation of concerns:

- **[xent-core](https://github.com/Project-Xent/xent-core)** — cross-platform UI layout engine written in C. Handles node tree management, layout computation, text measurement, accessibility semantics, and SIMD-accelerated calculations. No rendering, no OS dependencies.
- **[fluxent](https://github.com/Project-Xent/fluxent)** — Windows-only Fluent Design UI framework built on top of xent-core. Handles rendering (Direct2D / DirectWrite / DirectComposition), the full control library, input, theming, and animation.

```
┌────────────────────────────────────────────────────────┐
│                       fluxent                          │
│  Win32 · Direct2D · DirectWrite · DirectComposition   │
│  Controls (18) · Input · Theme · Animation · Popup    │
└───────────────────────┬────────────────────────────────┘
                        │ XentNodeId / XentContext
                        ▼
┌────────────────────────────────────────────────────────┐
│                      xent-core                         │
│  Layout: Flex · SwiftStack · Grid · Absolute          │
│  Text measurement · ISPC SIMD · Semantics · Plugins   │
└────────────────────────────────────────────────────────┘
```

---

## xent-core

**Repository**: [github.com/Project-Xent/xent-core](https://github.com/Project-Xent/xent-core)  
**Language**: C · **Build**: xmake  
**Latest commit**: Replace Highway SIMD backend with ISPC (2026-04-13)

### Layout protocols

| Protocol | Description |
|----------|-------------|
| Flex | Full Flexbox — row/column, wrap, justify, align, grow/shrink |
| SwiftStack | Priority-based stack with spacers and `layout_priority` |
| Grid | CSS Grid subset — AUTO/PIXEL/STAR tracks, row/column span (up to 16×16) |
| Absolute | Explicit position/size |

Incremental layout is supported via dirty-flag propagation (`DIRTY_SELF`, `DIRTY_SUBTREE`, `DIRTY_LAYOUT`). The engine skips clean subtrees on re-layout.

### Node storage

Nodes are stored in a Structure-of-Arrays (`XentNodeStore`) for cache-friendly access. Fields cover tree structure, layout inputs/outputs, Flex and SwiftStack properties, Grid definitions, text attributes, accessibility semantics, and focus metadata (`focusable`, `tab_index`).

### SIMD acceleration

ISPC kernels (runtime SSE4/AVX2 dispatch) accelerate:
- Flex line statistics (four-way parallel reduction)
- Flex grow/shrink distribution
- SwiftStack spacer distribution (masked add, FMA, proportional reduce)
- Dirty-flag batch OR / clear
- Pixel quantization

### Text subsystem

- Pluggable `XentTextBackend` interface
- Built-in Mono backend (no external dependencies): word-wrap, char-wrap, multi-line measurement
- Two-level cache: `XentTextCache` + `XentShapeCache`

### Control types (24)

`Container` · `Text` · `Button` · `ToggleButton` · `Checkbox` · `Radio` · `Switch` · `Slider` · `TextInput` · `Scroll` · `Image` · `Progress` · `List` · `Tab` · `Card` · `Divider` · `Canvas` · `PasswordBox` · `NumberBox` · `Hyperlink` · `RepeatButton` · `ProgressRing` · `InfoBadge` · `Tooltip` · `Custom`

### Other features

- Accessibility / semantic tree (`role`, `label`, `checked`, `value`, …)
- Focus management (`focusable`, `tab_index`)
- Plugin system (`xent_plugins.h`)
- CLI tooling (`xent_cli.h`)

### Public headers

`xent.h` · `xent_types.h` · `xent_layout.h` · `xent_text.h` · `xent_plugins.h` · `xent_cli.h`

---

## fluxent

**Repository**: [github.com/Project-Xent/fluxent](https://github.com/Project-Xent/fluxent)  
**Language**: C · **Platform**: Windows 10/11 · **Build**: xmake  
**Latest commit**: Large-scale new controls + focus navigation (2026-04-12)

### Rendering stack

- **Win32** window management
- **Direct2D** 2D drawing primitives
- **DirectWrite** text rendering
- **DirectComposition** composited windows
- **Backdrop effects**: Mica / Mica Alt / Acrylic (via `DwmSetWindowAttribute` + DComp)

Two-phase rendering pipeline: *collect* (walk xent-core node tree, build render command list) → *execute* (dispatch to per-control renderers).

### Control library (18 controls)

| Control | Notes |
|---------|-------|
| Button | Standard / Subtle / Text / Accent styles |
| ToggleButton | Checked/unchecked state |
| RepeatButton | Initial delay + repeat interval |
| Checkbox | Three-state: Unchecked / Checked / Indeterminate |
| RadioButton | Mutually exclusive group selection |
| ToggleSwitch | Animated knob (normal / hover / press sizes) |
| Slider | Horizontal, animated thumb scaling |
| TextBox | Full editor: UTF-8/16, cursor, selection, multi-line, Undo/Redo (50 levels), IME, auto-scroll, right-click menu |
| PasswordBox | TextBox core + `●` mask, press-to-reveal |
| NumberBox | min / max / step, SpinButtons in Inline / Compact / Hidden modes |
| ProgressBar | Determinate and indeterminate modes |
| ProgressRing | Circular progress indicator |
| ScrollView | Horizontal and vertical, Auto/Always/Never scrollbar policy |
| Card | Container with background and shadow |
| Divider | Separator line |
| HyperlinkButton | Visited / unvisited color distinction |
| InfoBadge | Dot / Number / Icon variants |
| Text | Static multi-line text with alignment and weight |

### Input system

- Mouse: hit testing, hover, click, double/triple click, scroll wheel
- Keyboard: full `WM_KEYDOWN` routing to focused node
- IME: `WM_IME_COMPOSITION` with cursor position feedback (`ImmSetCompositionWindow`)
- Right-click context menu
- Tab / Arrow key focus navigation ordered by `tab_index`

### Theme system

- Modes: Light / Dark / System (reads Windows preference)
- 35+ Fluent color semantic tokens (control fill, stroke, accent, text, card, focus, background, …)
- Version-number cache so controls refresh colors only when the theme changes

### Animation system

Frame-driven `FluxTween` (float) and `FluxColorTween` (RGBA), no external dependencies.

| Constant | Duration | Used for |
|----------|----------|----------|
| `FLUX_ANIM_DURATION_FAST` | 83 ms | Checkbox check mark |
| `FLUX_ANIM_DURATION_NORMAL` | 167 ms | Hover / focus ring |
| `FLUX_ANIM_DURATION_SLOW` | 250 ms | Color transitions |
| `FLUX_ANIM_DURATION_PRESS` | 110 ms | Press feedback |
| `FLUX_ANIM_DURATION_SLIDER` | 83 ms | Slider thumb scale |

Easing: `ease_out_quad`, `ease_in_out_cubic`.

### Other features

- Popup / ContextMenu system (borderless child window)
- Plugin interface
- `FluxNodeStore`: per-node UI state (visuals, state flags, event callbacks) keyed by `XentNodeId`

---

## Building

Both repositories use [xmake](https://xmake.io).

```bash
git clone https://github.com/Project-Xent/xent-core.git
git clone https://github.com/Project-Xent/fluxent.git

cd fluxent
xmake
xmake run hello-fluxent
```

ISPC must be available on `PATH` to compile the SIMD kernels in xent-core.
