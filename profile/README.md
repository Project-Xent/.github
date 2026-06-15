<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://img.shields.io/badge/Project_Xent-pure_C_desktop_UI-e2e8f0?style=for-the-badge&labelColor=0f172a">
  <img alt="Project Xent" src="https://img.shields.io/badge/Project_Xent-pure_C_desktop_UI-334155?style=for-the-badge&labelColor=f8fafc">
</picture>

![0BSD](https://img.shields.io/badge/license-0BSD-3b82f6?style=flat-square)
![C11/C17](https://img.shields.io/badge/C11%2FC17-no_C++-475569?style=flat-square&logo=c&logoColor=white)
![xmake](https://img.shields.io/badge/build-xmake-22c55e?style=flat-square)
![MSVC · clang · MinGW](https://img.shields.io/badge/MSVC_%C2%B7_clang_%C2%B7_MinGW-passing-22c55e?style=flat-square)

A desktop UI stack written entirely in C — layout engine, platform bindings,
Fluent Design controls, and a declarative Elm-architecture view layer — with
zero C++ and zero runtime dependencies beyond the OS.

```c
int main(void) {
    Model model = { .count = 0 };
    return fx_app_run(
        &(FluxAppConfig){ .title = L"Hello", .width = 640, .height = 480 },
        &model, update, view);
}
```

---

## Architecture

```mermaid
block-beta
  columns 3

  block:desc:3
    columns 3
    fx["fx.h — Elm MVU declarative layer"]:3
    fluxent["FluXent\n(Windows)"]  nexent["NeXent\n(macOS)"]  luxent["LuXent\n(BSD · Linux)"]
    d2d["D2D / DComp\nDirectWrite\nWinRT"]  metal["Metal\nCoreText\nCoreAnimation"]  arcan["Arcan / 9P\nHarfBuzz + FreeType\nEGL"]
    cwinrt["cwinrt\nWinRT C projection"]  objc["ObjC runtime\nbridge"]  shmif["SHMIF\n9P mount"]
    xent["xent-core — layout · tree · text · semantics · SIMD"]:3
  end

  style fx fill:#3b82f6,color:#fff,stroke:none
  style xent fill:#3b82f6,color:#fff,stroke:none
  style fluxent fill:#22c55e,color:#fff,stroke:none
  style nexent fill:#a1a1aa,color:#fff,stroke:none
  style luxent fill:#a1a1aa,color:#fff,stroke:none
  style d2d fill:#22c55e,color:#fff,stroke:none
  style metal fill:#a1a1aa,color:#fff,stroke:none
  style arcan fill:#a1a1aa,color:#fff,stroke:none
  style cwinrt fill:#22c55e,color:#fff,stroke:none
  style objc fill:#a1a1aa,color:#fff,stroke:none
  style shmif fill:#a1a1aa,color:#fff,stroke:none
```

> [!NOTE]
> <b>Green</b> = shipped &nbsp; <b>Grey</b> = planned

`xent-core` and `fx.h` are platform-agnostic and shared across every target.
Each platform provides its own rendering backend and system bindings; the
declarative layer reconciles against whichever backend is underneath.

---

## Repositories

### [`xent-core`](https://github.com/Project-Xent/xent-core) &ensp; ![lines](https://img.shields.io/badge/23.5k_lines-C11-475569?style=flat-square) ![0BSD](https://img.shields.io/badge/0BSD-3b82f6?style=flat-square)

Platform-agnostic layout engine and node tree.

| | |
|---|---|
| **Layout protocols** | Flex (CSS Flexbox), Grid (WinUI Star/Auto/Pixel), SwiftStack (SwiftUI VStack/HStack semantics), Absolute |
| **Data layout** | Structure-of-Arrays — one contiguous float array per property, SIMD-friendly |
| **SIMD** | Optional ISPC backend (SSE4 + AVX2 multi-target dispatch) for hot layout paths |
| **Text** | Pluggable backend with measurement cache; built-in monospace fallback for headless use |
| **Correctness** | Yoga-ported conformance suite (13k lines), 19 unit-test targets, regression-gated benchmarks |
| **Accessibility** | Semantic roles, labels, checked/enabled/expanded states, focus + tab-index in the core tree |

<details>
<summary>Public API surface</summary>

```c
XentContext *xent_create_context(XentConfig const *config);
XentNodeId   xent_create_node(XentContext *ctx);
bool         xent_set_protocol(XentContext *ctx, XentNodeId node, XentProtocol protocol);
bool         xent_set_flex_direction(XentContext *ctx, XentNodeId node, XentFlexDirection dir);
bool         xent_set_stack_axis(XentContext *ctx, XentNodeId node, XentAxis axis);
bool         xent_layout(XentContext *ctx, XentNodeId root, float w, float h);
// … size, margin, padding, gap, min/max, percent, aspect-ratio, grid tracks,
//   direction (LTR/RTL), wrap-content, z-index, dirty flags, profiling …
```

</details>

---

### [`cwinrt`](https://github.com/Project-Xent/cwinrt) &ensp; ![lines](https://img.shields.io/badge/12k_written_%C2%B7_786k_generated-C-475569?style=flat-square) ![0BSD](https://img.shields.io/badge/0BSD-3b82f6?style=flat-square) ![CI](https://img.shields.io/github/actions/workflow/status/Project-Xent/cwinrt/ci.yml?style=flat-square&label=CI)

Pure C projection for the entire Windows Runtime.

| | |
|---|---|
| **Generator** | Reads `.winmd` as PE + ECMA-335 `#~` metadata — no `cor.h`, no COM SDK headers |
| **Coverage** | 342 `Windows.*` namespaces; one header + one impl per namespace |
| **Runtime** | `init` · `hstring` (UTF-8/16) · `factory` (cached) · `async` (event + poll) · `delegate` · `event` — 951 lines total |
| **Naming** | [Frozen mangling spec](https://github.com/Project-Xent/cwinrt/blob/main/docs/MANGLING.md); overloads keyed by type signature, not ordinal — SDK additions never rename existing symbols |
| **CI gates** | Determinism (byte-stable) · Golden (ABI SHA-256) · PIID vs cppwinrt · Slot vs ABI · All-header compile · All-impl link · clang `-Werror` · MinGW full link × 4 arch · E2E on hardware |
| **Toolchains** | MSVC · clang-cl · clang · llvm-mingw |

<details>
<summary>C++/WinRT migration cheat sheet</summary>

| C++/WinRT | cwinrt |
|---|---|
| `winrt::init_apartment()` | `cwinrt_init(RO_INIT_MULTITHREADED)` |
| `Calendar c;` | `WGL_Calendar *c; wgl_calendar_new(&c);` |
| `c.Year()` | `int32_t y; wgl_calendar_get__year(c, &y);` |
| `obj.as<T>()` | `cwinrt_query(obj, &CWINRT_IID_T, &out)` |
| `co_await op;` | `cwinrt_async_wait((IUnknown *)op, INFINITE);` |
| `event += h;` | `on_event(self, fn, ctx)` → `cwinrt_token` |
| RAII release | `((IUnknown *)p)->lpVtbl->Release((IUnknown *)p)` |

</details>

---

### [`fluxent`](https://github.com/Project-Xent/fluxent) &ensp; ![lines](https://img.shields.io/badge/45k_lines-C17-475569?style=flat-square) ![0BSD](https://img.shields.io/badge/0BSD-3b82f6?style=flat-square)

WinUI 3-class Fluent Design controls, driven by xent-core layout and cwinrt platform integration.

| | |
|---|---|
| **Controls** | Button · Checkbox · RadioButton · ToggleSwitch · Slider · ComboBox · DropDownButton · SplitButton · TextBox · PasswordBox · NumberBox · ProgressBar · ProgressRing · ScrollViewer · Card · InfoBadge · InfoBar · Divider · Expander · Hyperlink · RepeatButton · Image · NavigationView (hierarchical) · TabView (close / reorder / add) · ContentDialog (modal focus trap) · MenuFlyout · MenuBar · Tooltip |
| **Render** | Direct2D / DirectWrite with snapshot diff + per-node cache; or Windows.UI.Composition retained tree with compositor-thread animations and InteractionTracker scrolling |
| **Theming** | Live system accent + light/dark palettes using WinUI 3 design tokens |
| **Input** | Hit testing · keyboard focus · DirectManipulation inertial scroll · InteractionTracker |
| **Accessibility** | Win32 UI Automation provider — control types, patterns, focus/invoke events |

#### `fx.h` — Elm Architecture in C

Each frame the app's `view` builds a throwaway element tree from arena memory;
the reconciler diffs it against the previous frame and patches retained controls.
Controls never call back directly — they post `FxMsg` values consumed by `update`.

```c
FluxEl *view(FxUi *ui, void *model) {
    Model *m = model;
    return fx_column(ui, (FxStackDesc){.gap = 12, .padding = {32,32,32,32}}, (FluxEl*[]){
        fx_text(ui, "Counter", (FxTextDesc){.size = 24}),
        fx_text(ui, fx_fmt(ui, "Count: %d", m->count), (FxTextDesc){0}),
        fx_button(ui, "Increment", (FxButtonDesc){.on_click = fx_msg(MSG_INC)}),
        m->count > 0
            ? fx_button(ui, "Reset", (FxButtonDesc){.on_click = fx_msg(MSG_RESET)})
            : NULL,
    FX_END});
}
```

> Compound literals + designated initializers make every property optional with
> a zero default. `NULL` entries are conditional rendering. The gallery demo —
> NavigationView, 20+ control pages — is ≈ 1200 lines of view code.

---

## Platform roadmap

| Target | Backend | Binding layer | Status |
|:---|:---|:---|:---:|
| Windows 10+ | D2D · DirectWrite · DComp | cwinrt (WinRT C projection) | ✅ |
| macOS | Metal · CoreAnimation · CoreText | ObjC runtime bridge | 🔲 |
| OpenBSD | Arcan | SHMIF · 9P | 🔲 |
| FreeBSD | Arcan | SHMIF · 9P | 🔲 |
| musl Linux | Arcan | SHMIF · 9P | 🔲 |

The BSD and Linux stack targets [Arcan](https://arcan-fe.com) — a
single-protocol (`SHMIF`) display server with security-oriented composition —
and exposes UI resources over [9P](https://9p.cat-v.org) for Plan 9-style
composability and network transparency.

OpenBSD is the primary development platform for LuXent; its `pledge`/`unveil`
model and strict libc complement Arcan's design. Downstream porting follows
least-friction order: FreeBSD first, then musl-based Linux
([Chimera Linux](https://chimera-linux.org),
[Void Linux](https://voidlinux.org)).

---

## Build

All repositories use [xmake](https://xmake.io).

```bash
# xent-core
xmake && xmake test

# cwinrt (Windows, requires Windows SDK)
xmake build cwinrt-gen cwinrt-rt test_smoke
xmake run test_smoke

# fluxent (Windows 10 1903+)
xmake run hello_fluxent
```

---

<sub>Every repository under Project Xent is released under the [0BSD](https://opensource.org/license/0bsd) license.</sub>
