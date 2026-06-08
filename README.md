<div align="center">

<img src="https://raw.githubusercontent.com/FortAwesome/Font-Awesome/refs/heads/6.x/svgs/solid/arrows-rotate.svg" width="64" height="64" alt="updock logo" style="filter: invert(1)"/>

# updock

**A Rust control plane for macOS Apple Silicon updates.**  
Scan, classify, and upgrade your dev machine — with intent.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](#)
[![Platform: macOS](https://img.shields.io/badge/platform-macOS%20Apple%20Silicon-black?logo=apple)](#)
[![Built with Rust](https://img.shields.io/badge/built%20with-Rust-orange?logo=rust)](#)

</div>

---

> **Status:** 🚧 Work in Progress — v0.1 in development

---

## Why Updock?

Your Mac update surface is fragmented. You manually run `brew upgrade`, `npm update`, `bun update`, `mas upgrade`, and `softwareupdate` — at different times, in different terminals, with no unified view of what's actually outdated or risky.

Updock unifies all of that into a single CLI with three principles:

- **Scan first, act second.** Always preview before changing anything.
- **Policy over impulse.** Auto-update patches, prompt for majors.
- **Apple Silicon aware.** Know which apps are still Intel-only on your M-series Mac.

---

## Supported Sources

| Source | Command | Status |
|--------|---------|--------|
| Homebrew Formulae | `brew upgrade` | ✅ v0.1 |
| Homebrew Casks | `brew upgrade --cask` | ✅ v0.1 |
| App Store | `mas upgrade` | ✅ v0.1 |
| npm (global) | `npm update -g` | ✅ v0.1 |
| bun | `bun upgrade` | ✅ v0.1 |
| macOS System | `softwareupdate` | ✅ v0.1 |
| `/Applications` Audit | Unmanaged app scan | 🚧 v0.2 |
| Apple Silicon Audit | Intel-only detection | 🚧 v0.2 |

---

## Installation

```bash
# Via Homebrew (coming soon)
brew install updock

# Or build from source
git clone https://github.com/yourusername/updock.git
cd updock
cargo build --release
cp target/release/updock /usr/local/bin/
```

---

## Quick Start

```bash
# Audit everything — no changes made
updock scan

# Show outdated packages only
updock outdated

# Run upgrade based on your policy
updock upgrade

# Check your environment health
updock doctor

# Audit /Applications for unmanaged, Intel-only, or duplicate apps
updock app-audit

# Machine-readable output for automation
updock report --json
```

---

## Commands

### `updock scan`
Detects all active update sources and shows their current state. **Read-only.** No system changes.

```
✔ brew         12 outdated
✔ brew cask     3 outdated
✔ mas           1 outdated
✔ npm           5 outdated
✔ bun           up to date
✔ system        1 pending restart
```

### `updock outdated`
Lists all outdated items with metadata: source, current version, latest version, update tier (patch/minor/major), and architecture.

```
NAME              SOURCE     CURRENT    LATEST     TIER     ARCH
───────────────────────────────────────────────────────────────
git               brew       2.44.0     2.46.0     minor    arm64
node              brew       20.11.0    22.2.0     major  ⚠ arm64
Notion            mas        3.1.2      3.4.0      minor    universal
macOS Sequoia     system     15.3       15.4       patch    arm64
```

### `updock upgrade`
Runs updates based on your active policy. Patch and minor are executed automatically; major versions prompt for confirmation.

```bash
updock upgrade            # follow policy
updock upgrade --dry-run  # preview only
updock upgrade --profile nightly  # override to aggressive profile
```

### `updock app-audit`
Scans `/Applications` and classifies every app by architecture (arm64 native, universal, intel-only/Rosetta) and ownership (brew cask, App Store, unmanaged DMG).

```
APP              ARCH          OWNED BY
──────────────────────────────────────────────
Firefox          universal     brew cask
Notion           arm64         mas
Figma            arm64         unmanaged
OldTool.app      intel-only ⚠  unmanaged
```

### `updock doctor`
Checks that all required CLI tools are installed and functional: `brew`, `mas`, `npm`, `bun`, `softwareupdate`.

### `updock report --json`
Outputs full scan results as JSON. Useful for piping into Raycast scripts, SketchyBar, cron jobs, or shell wrappers.

---

## Configuration

Updock uses a TOML config file at `~/.config/updock/config.toml`.

```toml
[policy]
# auto: run silently
# prompt: ask before running
# skip: never touch this
patch  = "auto"
minor  = "auto"
major  = "prompt"

[schedule]
enabled = true
interval = "daily"   # daily | weekly | manual

[exclude]
formulae = ["ruby", "python@3"]
casks    = ["docker"]
mas      = []

[profile.nightly]
# Override policy for aggressive updates
patch = "auto"
minor = "auto"
major = "auto"
```

---

## Architecture Overview

```
updock
├── cli.rs                  # clap command definitions
├── config/                 # TOML schema + loader
├── adapters/               # Per-source adapters
│   ├── brew.rs
│   ├── mas.rs
│   ├── npm.rs
│   ├── bun.rs
│   ├── softwareupdate.rs
│   └── apps.rs             # /Applications scanner
├── domain/                 # Internal package, policy, report models
├── engine/                 # Discovery, normalizer, executor
└── output/                 # Table, JSON, TUI renderers
```

Each adapter implements a shared `Adapter` trait with `list_outdated()`, `upgrade(pkg)`, and `dry_run(pkg)` methods, making it easy to add new sources without touching core logic.

---

## Roadmap

- **v0.1** — Core adapters (brew, mas, npm, bun, softwareupdate), TOML config, dry-run, table output, JSON output
- **v0.2** — Apple Silicon audit, `/Applications` scan, ownership mapping, unmanaged app detection
- **v0.3** — Policy engine, `launchd` scheduling, macOS local notifications
- **v1.0** — Plugin adapter system, rich TUI, self-update binary

---

## Compared to Alternatives

| | updock | topgrade | MacUpdater |
|--|--------|----------|------------|
| CLI-first | ✅ | ✅ | ❌ (GUI) |
| Apple Silicon audit | ✅ | ❌ | Partial |
| Policy engine | ✅ | Partial | ❌ |
| Unmanaged app scan | ✅ | ❌ | ❌ |
| JSON output | ✅ | ❌ | ❌ |
| Mac-only focus | ✅ | ❌ (cross-platform) | ✅ |
| Built in Rust | ✅ | ✅ | ❌ |

---

## Contributing

Contributions are welcome. If you want to add an adapter for a new package source, implement the `Adapter` trait in `src/adapters/` and open a PR.

```bash
# Run tests
cargo test

# Run with verbose output
UPDOCK_LOG=debug cargo run -- scan
```

---

## License

MIT © 2026 — made with ☕ on a Mac.

---

<div align="center">
<sub>Built for Apple Silicon. Runs fast. Ships intent.</sub>
</div>
