# Project Xent

**Modern, Native, Zero-Compromise UI Framework for C++**

> Write once in C++, render natively on any OS.

[![License](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)](LICENSE)
[![Standard](https://img.shields.io/badge/C%2B%2B-20-00599C.svg)]()

---

![FluXent-Example](https://github.com/Project-Xent/fluxent/blob/main/examples/hello_fluxent/Hello-FluXent.png?raw=true)

## What is Project Xent?

Project Xent is an architectural pattern for building high-performance native applications.

Unlike other frameworks that ship a bundled rendering engine, Xent bridges a high-level C++ reactive DSL directly to the OS's most performant native compositor.

```
Shared Logic (xent-core) + Native Rendering (per platform) = True Native Experience
```

Xent allows each platform to use its optimal rendering technology:
- **Windows**: DirectComposition + Fluent Design (FluXent)
- **Linux**: Wayland/X11 + EFL (LuXent)
- **macOS**: SwiftUI Integration (NeXent)

---

## Architecture

```
                Your Application Code
                  (One codebase)
                        ↓
        ┌───────────────────────────────────┐
        │         xent-core                 │
        │  • Layout (Yoga Flexbox)          │
        │  • Reactive View Tree             │
        │  • Signal-based State             │
        │  (Zero OS dependencies)           │
        └───────────────────────────────────┘
                        ↓
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
  ┌─────────┐     ┌─────────┐     ┌─────────┐
  │ FluXent │     │ LuXent  │     │ NeXent  │
  │(Windows)│     │(Linux)* │     │(macOS)* │
  └─────────┘     └─────────┘     └─────────┘
      ↓               ↓               ↓
 Native API      Native API      Native API
```

*\*Implementation in progress*

---

## Design Principles

### 1. **Zero Overhead**
- No virtual machine, no heavy runtime.
- Compiles to direct platform API calls.
- Cold start < 50ms.

### 2. **Strict Architecture**
- **Ownership**: Explicit `std::unique_ptr` hierarchy. No hidden shared state or ambiguous lifecycles.
- **Purity**: `xent-core` is purely data-driven. Zero platform dependencies (no COM/D2D pointers).
- **Safety**: Compile-time enforcement of layout and event handlers.

### 3. **Native First**
- No "painted" widgets that feel wrong.
- Windows uses DirectComposition + Direct2D for true native performance.
- Linux uses Wayland/EFL.
- macOS uses SwiftUI/Metal.

### 4. **No Compromise**
- Rejects "lowest common denominator" features.
- Exposes platform-specific capabilities (like Mica on Windows) where available.

---

## Naming

```
Xent    = eXtensible Native Toolkit

FluXent = Fluent + Xent (Windows Implementation)
          Uses DirectComposition and Win32 APIs to deliver 
          next-gen Fluent Design.

LuXent  = Lux (Light) + Xent (Linux/Unix Implementation)
          Leverages EFL and Wayland for a lightweight, 
          fast Unix experience.

NeXent  = NeXT + Xent (macOS Implementation)
          Bridges C++ logic with SwiftUI/Cocoa.
```

---

## Platform Status

| Platform | Implementation | Status |
|----------|---------------|--------|
| **Windows 10/11** | FluXent | 🚧 **In Development** (Fluent v2/Mica supported) |
| **Linux (Wayland)** | LuXent | 📅 **Planned** |
| **macOS** | NeXent | 📅 **Planned** |

---

## Why Xent?

### vs Qt
**Qt** renders its own style everywhere, often looking alien on modern systems.
**Xent** delegates rendering to the native OS compositor, ensuring your app always looks native and respects system updates automatically.

### vs Flutter
**Flutter** uses a custom rendering engine (Skia/Impeller), adding size overhead and potential input lag.
**Xent** is a thin C++ layer over native APIs. Zero runtime overhead.

### vs Electron
**Electron** ships an entire browser.
**Xent** produces single-file binaries (KB, not MB) with instant startup.

---

## Getting Started

### Prerequisites
- C++20 Compiler (MSVC, GCC, or Clang)
- xmake

### internal-preview build (Windows-only)

```bash
# Clone the repository
git clone https://github.com/Project-Xent/xent-core.git
git clone https://github.com/Project-Xent/fluxent.git

# Build and run the example
cd fluxent
xmake
xmake run hello-fluxent
```

---

## Component Tree (DSL)

Xent uses a declarative, fluent syntax to build UI hierarchies with strict ownership semantics.

```cpp
// Example: A simple counter component
using namespace xent::dsl;

return Create<VStack>({
    Create<Text>("Counter App") // Temporary Text created
        .FontSize(24)
        .Margin(10),

    Create<HStack>({
        Create<Button>("-").OnClick(&State::decrement, &state),
        Create<Text>(state.count_string),
        Create<Button>("+").OnClick(&State::increment, &state)
    }).Gap(20)

}).AlignItems(YGAlignCenter);
```

### Core Components
- **Layouts**: `VStack`, `HStack`, `Spacer`
- **Controls**: `Text`, `Button`, `CheckBox`, `RadioButton`, `ToggleButton`
- **Modifiers**: `.Padding()`, `.Background()`, `.CornerRadius()`, `.OnClick()`

---

### Core Components (updated)
- **Layouts**: `VStack`, `HStack`, `Spacer`, `Grid`
- **Controls**: `Text`, `TextBox`, `Button`, `ToggleButton`, `ToggleSwitch`, `CheckBox`, `RadioButton`, `Slider`, `ScrollView`
- **Visuals**: Icon support for controls, theming and WinUI3-style visuals
- **Modifiers**: `.Padding()`, `.Background()`, `.CornerRadius()`, `.OnClick()`, `.Icon()`

---

## Recent Implementations

The repository has recently landed several features and refactors. Notable items from the commit history:

- `TextBox` control with IME support and input/IME integration
- `Slider` and `ScrollView` controls plus animation utilities
- `ToggleSwitch` control and improved control rendering
- Checkbox visual states and improved Button/Toggle visuals (WinUI3-style)
- Icon support for `Button` and `ToggleButton`
- Plugin system added
- Replaced `std::function` with `xent::Delegate` for callback efficiency
- Added `tl::expected` third-party header
- Large refactors of controls, rendering and input handling

## Roadmap (updated)

- [x] **Core**: Reactivity system and Yoga layout integration.
- [x] **Windows**: DirectComposition rendering pipeline.
- [x] **Windows**: Basic Fluent controls (Button, Toggle, Checkbox).
- [x] **Windows**: TextBox, IME support, ToggleSwitch, Slider, ScrollView, icons and improved button visuals.
- [x] **Windows**: Plugin system and callback refactor (`xent::Delegate`).
- [ ] **Windows**: Complete advanced layout components (Grid polishing, virtualization)
- [ ] **Linux**: Initial Wayland surface integration and LuXent port.
- [ ] **macOS**: Metal/SwiftUI bridge prototype and NeXent port.

---

## License

BSD 3-Clause License

---

## Inspiration

- **OpenStep**: The original native cross-platform UI (NeXTSTEP heritage)
- **React**: Component-based architecture
- **SwiftUI**: Declarative syntax and reactive updates
- **Yoga**: Fast Flexbox layout engine
- **Enlightenment**: Unix GUI philosophy and EFL architecture
- **WinUI**: Fluent Design System
