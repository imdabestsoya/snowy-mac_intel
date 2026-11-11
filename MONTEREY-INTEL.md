# Snowy for Intel Macs running macOS 12 — candidate source port

This is modified source, not a compiled app. Compilation and macOS 12 runtime testing are still required. The authoring environment had Linux, no Swift compiler, no Apple SDK, and no macOS build runner. Shell syntax, plist values, and patch whitespace were checked; native tests have NOT run.

Based on https://github.com/soloiaros/snowy-mac at commit 5769b0001c62be93f238536dd1ca0dcda68eba87. Original MIT license retained.

## Build using GitHub (no Xcode needed on your Monterey Mac)

1. Fork the original repository into your GitHub account.
2. Copy this archive's source changes into your fork, including the hidden `.github/workflows/monterey-intel.yml` file. Commit and push them to the default branch. The included `monterey-intel.patch` also applies to the base commit above using `git apply`.
3. In the fork, open **Actions → Build Monterey Intel → Run workflow**. Enable Actions for the fork if GitHub asks.
4. When the run succeeds, open it and download the **Snowy-macOS12-Intel** artifact.
5. Extract the artifact ZIP, then the nested app ZIP. Move **Snowy.app** to Applications.
6. Open it. This build is locally signed, not notarized. If Gatekeeper blocks it, use Monterey's **System Preferences → Security & Privacy → General → Open Anyway**, only for the app you built.

The workflow compiles on an Intel macOS 15 runner using Xcode 16.4 while targeting macOS 12. A successful build checks compile-time API availability, architecture, deployment target, signature, and the upstream physics tests. It does not prove runtime compatibility on Monterey. If it fails, keep the build log for fixing the remaining incompatibilities.

## Build on a newer Mac

You need Xcode 16.4 with its Metal toolchain; this toolchain does not run on Monterey. From the source directory:

```sh
SNOWY_ARCH=x86_64 zsh scripts/build-app.sh
```

The app is produced at `build/Snowy.app`. Transfer the app as a ZIP created with `ditto -c -k --sequesterRsrc --keepParent build/Snowy.app Snowy-macOS12-Intel.zip` to preserve framework symlinks.

## Changes

- Target macOS 12 in SwiftPM, app metadata, and Metal shader compilation (Metal 2.4).
- Replace Observation/Bindable with Combine ObservableObject/Published/ObservedObject.
- Replace NavigationSplitView with HSplitView, unavailable empty-state UI with Label, and newer rounded-rectangle convenience APIs with explicit shapes.
- Use Monterey-compatible activation, change observation, task sleeping, and line limits. Guard hosting-controller sizing options on macOS 13+.
- Explicit Intel architecture option for the build script.
- Disable upstream updater startup for this compatibility build so it cannot replace this port with an upstream release. Sparkle remains bundled as an upstream dependency.

## Test on the target Mac before relying on it

Check launch/menu bar, snowfall and Metal rendering, window movement, bottom Dock, wind, sweeping/shovel, pause/clear, multi-display behavior, wallpapers, settings persistence, scene editing, and sleep/wake. Monterey has no Sonoma desktop widgets; those widget-specific surfaces are not expected. A Metal-capable GPU is required. Performance on older Intel graphics is unverified; start with Light snowfall.
