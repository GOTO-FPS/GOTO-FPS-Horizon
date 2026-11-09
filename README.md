# GOTO-FPS Horizon

A small sandboxed Horizon Android 0.7.8 client mod that displays the game's live FPS in a shaded HUD panel.

## Compatibility

- Target: Horizon Android 0.7.8, 64-bit APKs (`arm64-v8a` and `x86_64`). The 0.7.8 release source and the pinned mod SDK provide the `render::frame_stats()` API and `render::set_corner()` HUD renderer used here.
- Format: genuine sandboxed Horizon code mod (`.hmod`), containing a WebAssembly component and root `horizon-mod.toml` manifest. This is not a content-only pack, `.hnmod`, Java mod, or renamed archive.
- FPS is sampled through Horizon's frame-statistics API (the engine refreshes it twice per second). The mod only replaces its corner HUD shapes when the displayed text changes.
- Default position is top-left. The mod settings page cycles through all four HUD corners; Horizon lays out the corner panel using its own GUI coordinates and HUD placement.
- No Android permissions, network, or filesystem access are requested. The settings use Horizon's per-mod sandbox storage.

Horizon Android 0.7.8's 32-bit APKs do not include the native `.hmod` runtime; use the 64-bit release builds. Actual rendering, safe-area placement, and touch-control overlap should still be checked on the intended phone.

## Build and validate

The project pins the official Horizon/Starling v0.7.8 source revision `68a2c4dd1b9a9e511ad3758a79ab772b6f9e5b29` for both the Rust SDK and package tool. Install Rust 1.95.0 and the WebAssembly target, then install the official builder from that exact source revision:

```sh
rustup toolchain install 1.95.0 --component rustfmt --component clippy
rustup target add wasm32-unknown-unknown --toolchain 1.95.0
# From a checkout of the exact official v0.7.8 source revision:
cargo +1.95.0 install --path crates/cargo-horizon-mod --locked
cd /path/to/GOTO-FPS-Horizon/mod
cargo +1.95.0 fmt --check
cargo +1.95.0 check --target wasm32-unknown-unknown
cargo +1.95.0 clippy --target wasm32-unknown-unknown -- -D warnings
cargo +1.95.0 horizon-mod test
cargo +1.95.0 horizon-mod build
cargo +1.95.0 horizon-mod check target/horizon-mod/goto_fps.hmod
```

The official builder creates the package at `mod/target/horizon-mod/goto_fps.hmod`. The GitHub Actions workflow runs formatting, target compilation, Clippy, sandbox mod tests, official package validation, and archive/manifest checks, then publishes the built bytes as the `GOTO-FPS-0.7.8.hmod` artifact.

## Install on Horizon Android 0.7.8

1. Download `GOTO-FPS-0.7.8.hmod` from the successful workflow run's Artifacts section.
2. In a supported 64-bit Horizon Android 0.7.8 build, import the `.hmod` from the launcher Mods page, or place it in the active profile's `horizon-mods/` directory.
3. Open the game's Options > Mods, enable GOTO-FPS Counter, and restart the game if prompted.
4. During gameplay the overlay shows `FPS: <live value>` at the top-left. The mod's settings page in Options > Mods can cycle the panel to another corner.

## Attribution and license

The API use and HUD rendering approach were checked against Horizon's official v0.7.8 FPS counter. This project has its own implementation and retains the required Horizon MIT copyright/license notice in `THIRD_PARTY_NOTICES.md`. The project source is MIT licensed; see `LICENSE`.
