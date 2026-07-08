# BSB 入力UI

## 確定事項

現行公開GitHubリポジトリには、BSBクリック処理、選択状態管理、エネルギー計算、接続判定、React Componentは含まれていない。

## 推奨状態モデル

```ts
type GridUiState = {
  pairId: string;
  moveLevel: number;
  selectedPanelIds: Set<string>;
  energyUsed: number;
  energyLimit: number;
  validation: {
    disconnectedPanelIds: string[];
    overEnergy: boolean;
    lockedPanelIds: string[];
  };
};
```

## クリック処理

```text
onPanelClick(panelId)
  ├─ panel = grid.panels[panelId]
  ├─ if selected: tryDeselect(panelId)
  │    └─ OFF後の連結性を検証
  └─ else: trySelect(panelId)
       ├─ unlock条件を検証
       ├─ 接続条件を検証
       ├─ energy上限を検証
       └─ selectedPanelIdsに追加
```

## エネルギー計算

`selectedPanelIds` から毎回reduceで再計算する方式を推奨する。差分更新だけに依存すると、データ更新やundo/redoで不整合が起きやすい。

```ts
const energyUsed = selectedPanelIds.reduce(
  (sum, id) => sum + grid.panelsById[id].cost.energy,
  0,
);
```

## 有効・無効判定

パネル表示状態は保存せず、派生値として計算する。

```ts
type PanelViewState = 'selected' | 'available' | 'locked' | 'blockedByEnergy' | 'blockedByConnection';
```

## UI更新フロー

1. Pairまたはわざレベル変更。
2. GridDefinitionをロード。
3. 既存selectedPanelIdsを再検証。
4. PanelViewModelを再生成。
5. SVG/Panelコンポーネントが再描画。
6. AggregatedEffectsを計算エンジン入力へ反映。

## React設計上の改善案

- `useGridSelection(grid, moveLevel, energyLimit)` のようなhookにロジックを集約。
- Panelコンポーネントは表示専用にする。
- 接続判定・エネルギー判定は純関数化し、単体テストする。
- URLエンコード/JSON保存はUI stateとは別モジュールにする。
