# AGENTS

## Overview
- This repo is a Dalamud plugin fork of `Simple Tweaks`.
- The fork currently builds as `SimpleTweaksFork`, not `SimpleTweaksPlugin`.
- The fork manifest files are `SimpleTweaksFork.yaml` and `pluginmaster.json`.

## Build And Packaging
- Local build: `dotnet build -c Debug`
- Output DLL: `bin/Debug/SimpleTweaksFork.dll`
- Output plugin package metadata: `bin/Debug/SimpleTweaksFork.json`
- GitHub installable build is published by `.github/workflows/publish-installable.yml`.

## Local Dalamud Environment
- Dalamud dev runtime is available at `~/.xlcore/dalamud/Hooks/dev/`.
- Local installed plugins live under `~/.xlcore/installedPlugins/<InternalName>/<Version>/`.
- For this fork, the live local install path is typically `~/.xlcore/installedPlugins/SimpleTweaksFork/1.14.1.0/`.
- Per-tweak config files live under `~/.xlcore/pluginConfigs/SimpleTweaksFork/`.

## Local Deploy
- Build first, then copy artifacts second. Do not run build and copy in parallel.
- Typical local deploy sequence:
  `dotnet build -c Debug && cp bin/Debug/SimpleTweaksFork.dll ~/.xlcore/installedPlugins/SimpleTweaksFork/1.14.1.0/ && cp bin/Debug/SimpleTweaksFork.deps.json ~/.xlcore/installedPlugins/SimpleTweaksFork/1.14.1.0/ && cp bin/Debug/SimpleTweaksFork.json ~/.xlcore/installedPlugins/SimpleTweaksFork/1.14.1.0/`
- For hook-heavy changes, a full game restart is more reliable than assuming a hot reload worked.

## Tweak Architecture
- Tweaks mostly live under `Tweaks/` and are auto-discovered from classes derived from `Tweak`.
- Common attributes:
  `TweakName`, `TweakDescription`, `TweakCategory`, `TweakAutoConfig`, `TweakReleaseVersion`.
- Auto-config files are keyed by tweak class key/name and saved in `pluginConfigs`.
- Many low-level hooks use `HookWrapper<T>` and helper methods in `Utility/Common.cs`.

## Hooking Notes
- `TweakHook(typeof(...), nameof(...), ...)` works when the target type exposes a generated nested `Addresses` type through ClientStructs interop.
- Not every ClientStructs type/method exposes that. If address resolution is unavailable, use a manual hook path instead, usually via `Common.Hook(address, detour)` or an existing vtable pointer.

## Safety And Review Tips
- Native crashes may not show a managed stack. Check `dalamud.log`, `dalamud.boot.log`, and crash packs together.
- If a change introduces repeated CTDs, assume low-level hooks are suspect until isolated.
- For code review in this repo, pay extra attention to hook lifetime, `FrameworkUpdate` subscriptions, and local deploy race conditions.
