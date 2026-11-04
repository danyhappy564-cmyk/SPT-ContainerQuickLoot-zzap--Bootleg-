<p align="center">
  <a href="README.md"><img alt="한국어" src="https://img.shields.io/badge/%ED%95%9C%EA%B5%AD%EC%96%B4-%ED%95%9C%EA%B5%AD%EC%96%B4%EB%A1%9C%20%EB%B3%B4%EA%B8%B0-6e7681?style=for-the-badge"></a>
  <a href="README.en.md"><img alt="English (this page)" src="https://img.shields.io/badge/English-This%20page-1f6feb?style=for-the-badge"></a>
</p>

### ⚠️ IMPORTANT NOTICE / DISCLAIMER

- **Original Author:** CactusPie
- **Original Repository:** SPT-ContainerQuickLoot
- **Original Link:** https://github.com/CactusPie/SPT-ContainerQuickLoot
- **License:** GPL-3.0
- **This Port By:** R_F (danyhappy564-cmyk) — unofficial, AI-assisted port. Not affiliated with or endorsed by the original author.

1. **Reflection & Take-Downs:** I deeply reflect on the ECOT incident. As an AI-assisted "vibe coder," I will immediately delete files if the original authors ask.
2. **No Re-Distribution:** These ported builds are unverified, temporary fixes. Please do NOT re-upload or share them anywhere else.
3. **Do Not Pester Original Authors:** Never report bugs or pester original modders regarding issues from my unofficial ports.
4. **Full Credit & Respect:** I will always credit original creators on GitHub and prioritize their decisions above all else.
5. **Support Original Creators:** Instead of using my ports, please visit the original authors' Forge pages to leave kind words or tips.

---

## Changelog

The full changelog (1 entry) is in the [`changelog/`](changelog/) folder, 10 per file (higher number = newer, latest: [`CHANGELOG-001.md`](changelog/CHANGELOG-001.md)). Per-release summaries: [Releases](../../releases).

# ContainerQuickLoot (fork)

> **Original author · Original repository**
> **CactusPie** — https://github.com/CactusPie/SPT-ContainerQuickLoot
>
> This repository is a **fork** of the original above. Its features are unchanged; it has only
> been **ported to build and run on SPT 4.1**.

Automatically places items in containers tagged `@loot`. Works with both Ctrl+click transfers
and loose loot pickups, and automatically merges stacks (money, ammunition, etc.). Adding a number
to the tag (`@loot1`, `@loot2` …) sets its priority.

Currently targets **SPT 4.1**.

---

## Changes in the 4.1 port

**8 type names** changed. All were renamed when 4.1 deobfuscated the client.

| 4.0 | 4.1 |
|---|---|
| `InteractionsHandlerClass` | `EFT.InventoryLogic.ItemManipulator` |
| `TraderControllerClass` | `EFT.InventoryLogic.ItemController` |
| `StashGridClass` | `EFT.InventoryLogic.Grid` |
| `GStruct154<T>` | `Diz.LanguageExtensions.OperationResult<T>` |
| `GInterface424` | `EFT.InventoryLogic.IItemOperationResult` |
| `GClass3417` | `EFT.InventoryLogic.MergeResult` |
| `GClass3411` | `EFT.InventoryLogic.MoveResult` |
| `GClass3248` | `EFT.InventoryLogic.ContainerCollection` |

**All were cross-checked against two independent sources** — the SPT 4.1 wiki's `Class_Name_Mappings.md`
and assembly-tool's mapping json5. Both sources agree on all 8.

`EMoveItemOrder` (nested enum), `Item`, `Inventory`, `ItemAddress`, `TagComponent`,
`CompoundItem`, `LocationInGrid`, and `InventoryEquipment` are absent from the rename table, so they are unchanged.
The evidence is that **all renamed nested types** of `InteractionsHandlerClass` are listed in that table,
and `EMoveItemOrder` is absent.

`Grid` is **fully qualified** as `EFT.InventoryLogic.Grid`. Its name is common enough that adding
`using UnityEngine;` later would immediately make it ambiguous (UnityEngine also has `Grid`).

### Evidence that these 8 are all the changes needed

The repository included the 4.0 `Assembly-CSharp.dll`, so both states were actually compiled:

| | Result |
|---|---|
| Original source × 4.0 assembly | **Build succeeded** |
| Renamed source × 4.0 assembly | Only the renamed types were reported as "not found" |

This means the names that fail against 4.0 are exactly those changed here, and the rest of the code
does not need modification.

---

## Build configuration was also overhauled

- Old-style csproj → SDK style; `net472` → `netstandard2.1`
- **Deleted the repository's `libs/` folder (27MB).** It contained 4.0 DLLs, so building against it
  **silently** produces a plugin compiled against types that do not exist in 4.1 (the baseline build
  above demonstrates this — the 4.0 assemblies resolve every name in this project without complaint).
  References now come from the actual installation
- **Removed the `BepInEx.Core` NuGet package**; BepInEx is also referenced from the installation.
  That package required `nuget.bepinex.dev`, so builds failed entirely where that host was unreachable.
  There was also no guarantee that its pinned version matched the BepInEx version actually loading
  the plugin. That source was removed from `NuGet.Config` too
- Changed release zip output to a location **outside** the folder being archived

## Build

```
dotnet build CactusPie.ContainerQuickLoot.sln
```

Paths come from `SptRoot`. Its default is `E:\SPT 4.1`; override it with `-p:SptRoot=...`
or an environment variable with the same name. An incorrect path produces an explanatory error.

Building copies the plugin directly into `$(SptRoot)\BepInEx\plugins\` and creates a release zip.

## Settings (F12 — BepInEx Configuration Manager)

| Setting | Default |
|---|---|
| Enable for Ctrl+click | On |
| Enable for loose loot | On |
| Merge stacks | On |
| Merge stacks for non-loot containers | On |
| 루팅 컨테이너 태그 키워드 (Loot container tag keyword) | `@loot` |

## License

**GPL-3.0.** The original [CactusPie/SPT-ContainerQuickLoot](https://github.com/CactusPie/SPT-ContainerQuickLoot)
is GPL-3.0, and this repository follows the same license as a derivative. See `LICENSE` for the full text.

Redistribution must include **the full license text and source code** as required by GPL-3.0.

## ⚠️ Build it yourself before use

The SPT launcher reads the **`spt-reflection` assembly version** referenced by a plugin to determine
which SPT version it was built for, and blocks the game from launching if it does not match.
This mod's DLL must therefore be **built against your actual SPT installation**.

```
dotnet build CactusPie.ContainerQuickLoot.sln -c Release
```

The default `SptRoot` is `E:\SPT 4.1`, so you can build as is. If yours differs, use `-p:SptRoot=...`.

Building against a stub `spt-reflection.dll` (version 1.0.0.0) produces a DLL that **compiles but is blocked
only by the launcher**, so the build is configured to stop with an error in that case.
