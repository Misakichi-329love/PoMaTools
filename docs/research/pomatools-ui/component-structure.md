# Component Structure 案

## 確定事項

現行公開GitHubリポジトリにはPoMaTools本体のComponentは存在しない。以下はPoMa Gym Damage Calculator v3向けの設計提案である。

## 推奨Component構成

```text
PairBuildPage
  ├─ PairSearchPanel
  ├─ PairSummaryCard
  ├─ BuildControls
  │   ├─ LevelInput
  │   ├─ RaritySelect
  │   ├─ MoveLevelSelect
  │   ├─ ExToggle
  │   └─ SuperAwakeningSelect
  ├─ SyncGridEditor
  │   ├─ GridToolbar
  │   ├─ GridEnergyMeter
  │   ├─ GridSvg
  │   │   ├─ GridConnections
  │   │   └─ GridPanelNode
  │   └─ GridPanelDetails
  └─ CalculationPreview
```

## Hooks/Services

```text
usePairBundle(pairId)
useGridSelection(grid, constraints)
useBuildSerialization(build)
useCalculationProjection(build, selectedPanels)
```

## 呼び出し関係

- `PairBuildPage` がpairIdとbuild stateを所有。
- `usePairBundle` が必要JSONをロード。
- `SyncGridEditor` は `grid`, `selectedPanelIds`, `onTogglePanel` を受け取る。
- `useGridSelection` が接続判定とエネルギー判定を行う。
- `useCalculationProjection` が選択パネル効果を計算エンジン用に正規化。

## UIとドメインの分離

UI Componentにゲームルールを埋め込まない。以下を純関数として切り出す。

- `canSelectPanel`
- `canDeselectPanel`
- `calculateEnergy`
- `getConnectedPanelIds`
- `aggregateGridEffects`
- `encodeBuildToUrl`
- `decodeBuildFromUrl`
