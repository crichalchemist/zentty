# Intel Mac (x86_64) & macOS Ventura Support Design

## Overview
This design outlines the necessary changes to enable Zentty to build and run natively on Intel Macs (x86_64), specifically targeting compatibility with macOS 13 (Ventura).

## 1. Architecture & Build Configuration
* **Universal Binaries:** Add `ARCHS: "x86_64 arm64"` to the `settings > base` block in `project.yml`. This instructs Xcode to compile all targets, including Swift Package dependencies (Sentry, Sparkle, FuzzyMatch), as universal "fat" binaries.
* **macOS Ventura Support:** 
  * Lower `MACOSX_DEPLOYMENT_TARGET` to `"13.0"` in `project.yml`.
  * Lower the `deploymentTarget: macOS` to `"13.0"` in `project.yml`.
  * Lower `LSMinimumSystemVersion` to `"13.0"` in the `ZenttyCLI` target.

## 2. Dependencies & Data Flow
* **GhosttyKit Engine:** The `GhosttyKit.xcframework` must be built universally. `ghosttykit.lock` already specifies `build_target=universal`. We will verify `scripts/build_ghosttykit.sh` processes this correctly.
* **Package Managers (Homebrew & MacPorts):** Since developers may use MacPorts in addition to Homebrew, `scripts/build_ghosttykit.sh` will be updated to check for dependencies (like `gettext` and `zig`) using both `port` and `brew`, rather than strictly failing if `brew` is not the primary package manager.
* **CLI Wrappers:** Agent wrapper shell scripts remain unmodified as they are platform-agnostic `bash` scripts.

## 3. Error Handling & Testing
* **Error Handling (Compile-Time):** Lowering the deployment target will surface any macOS 14 APIs currently in use. We will resolve these compilation errors by wrapping them in Swift `#available(macOS 14.0, *)` checks and implementing appropriate macOS 13 fallbacks to gracefully degrade the UI without crashing.
* **Testing Strategy:**
  1. `scripts/test-on-virtual-display -only-testing:ZenttyLogicTests` to verify parallel-safe logic integrity.
  2. `scripts/test-hosted-on-virtual-display` to ensure window management and app lifecycle succeed on macOS 13.
  3. `scripts/test-agy-bench` to validate the shell integrations and hook pipelines.
