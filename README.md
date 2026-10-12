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

# ContainerQuickLoot (fork)

> **원작자 · 원본**
> **CactusPie** — https://github.com/CactusPie/SPT-ContainerQuickLoot
>
> 이 레포는 위 원작의 **포크**입니다. 기능은 그대로고, **SPT 4.1에서 빌드·동작하도록
> 포팅**한 것이 전부입니다.

`@loot` 태그를 붙인 컨테이너로 아이템이 자동으로 들어가게 해줍니다. Ctrl+클릭 이동과
바닥 루팅 양쪽에 적용되고, 스택(돈·탄약 등)은 자동으로 합쳐집니다. 태그에 숫자를 붙이면
(`@loot1`, `@loot2` …) 우선순위가 됩니다.

현재 기준 **SPT 4.1**.

---

## 4.1 포팅에서 바뀐 것

타입 이름 **8개**입니다. 4.1이 클라이언트를 역난독화하면서 전부 바뀌었습니다.

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

**독립된 두 출처로 전부 교차 확인했습니다** — SPT 4.1 wiki의 `Class_Name_Mappings.md`와
assembly-tool의 매핑 json5. 8개 모두 두 곳이 일치합니다.

`EMoveItemOrder`(중첩 열거형), `Item`, `Inventory`, `ItemAddress`, `TagComponent`,
`CompoundItem`, `LocationInGrid`, `InventoryEquipment`는 리네임 표에 없습니다 = 그대로.
`InteractionsHandlerClass`의 중첩 타입 중 **바뀐 것들은 표에 전부 나열되어 있고**
`EMoveItemOrder`는 거기 없다는 게 근거입니다.

`Grid`는 `EFT.InventoryLogic.Grid`로 **완전 수식**해뒀습니다. 이름이 너무 흔해서 나중에
누가 `using UnityEngine;`을 추가하면(거기도 `Grid`가 있습니다) 바로 모호해집니다.

### 이 8개가 전부라는 근거

레포에 4.0 시절 `Assembly-CSharp.dll`이 들어 있어서 양쪽 상태를 실제로 컴파일해봤습니다:

| | 결과 |
|---|---|
| 원본 소스 × 4.0 어셈블리 | **빌드 성공** |
| 리네임 후 × 4.0 어셈블리 | 리네임한 이름들만 "찾을 수 없음" |

4.0에서 깨지는 이름이 정확히 우리가 바꾼 것들뿐이고, 나머지 코드는 손댈 필요가 없다는
뜻입니다.

---

## 빌드 설정도 갈아엎었습니다

- 구식 csproj → SDK 스타일, `net472` → `netstandard2.1`
- **레포의 `libs/` 폴더(27MB) 삭제.** 4.0 DLL이라 그걸로 빌드하면 4.1에 없는 타입을 향해
  컴파일된 플러그인이 **조용히** 나옵니다 (위 베이스라인 빌드가 그 증거입니다 — 4.0
  어셈블리는 이 프로젝트의 모든 이름을 아무 불평 없이 해결해줍니다). 참조는 실제
  설치본에서 가져옵니다
- **`BepInEx.Core` NuGet 패키지 제거**, BepInEx도 설치본에서 참조합니다. 그 패키지는
  `nuget.bepinex.dev`가 필요해서 그 호스트에 못 닿는 환경에서는 빌드가 아예 실패했고,
  패키지가 고정한 버전이 게임이 실제로 이 플러그인을 로드하는 BepInEx와 같다는 보장도
  없었습니다. `NuGet.Config`에서도 그 소스를 뺐습니다
- 릴리스 zip을 압축 대상 폴더 **밖에** 쓰도록 수정

## 빌드

```
dotnet build CactusPie.ContainerQuickLoot.sln
```

경로는 `SptRoot`에서 나옵니다. 기본값 `E:\SPT 4.1`, `-p:SptRoot=...` 또는 동명의
환경변수로 덮어쓸 수 있습니다. 경로가 틀리면 이유를 말해줍니다.

빌드하면 `$(SptRoot)\BepInEx\plugins\`로 바로 복사되고 릴리스 zip도 만들어집니다.

## 설정 (F12)

| 항목 | 기본값 |
|---|---|
| Enable for Ctrl+click | 켜짐 |
| Enable for loose loot | 켜짐 |
| Merge stacks | 켜짐 |
| Merge stacks for non-loot containers | 켜짐 |
| 루팅 컨테이너 태그 키워드 | `@loot` |

## 라이선스

**GPL-3.0.** 원작 [CactusPie/SPT-ContainerQuickLoot](https://github.com/CactusPie/SPT-ContainerQuickLoot)
가 GPL-3.0 이고, 이 저장소는 그 파생물이라 같은 라이선스를 따릅니다. 전문은 `LICENSE` 참고.

재배포하실 때는 GPL-3.0 조건대로 **라이선스 전문과 소스를 같이** 제공해야 합니다.

## ⚠️ 직접 빌드해서 쓰세요

SPT 런처는 플러그인이 참조하는 **`spt-reflection` 어셈블리 버전**을 읽어서 "몇 버전용으로
빌드됐는지"를 판정하고, 안 맞으면 게임 실행을 막습니다. 그래서 이 모드의 DLL은 **실제 SPT
설치본에 대고 빌드해야** 합니다.

```
dotnet build CactusPie.ContainerQuickLoot.sln -c Release
```

`SptRoot` 기본값이 `E:\SPT 4.1` 이라 그대로 빌드하면 됩니다. 다르면 `-p:SptRoot=...`.

껍데기 `spt-reflection.dll`(버전 1.0.0.0)에 대고 빌드하면 **컴파일은 되고 런처에서만 막히는**
DLL이 나오기 때문에, 그 경우 빌드가 에러로 멈추도록 해뒀습니다.
