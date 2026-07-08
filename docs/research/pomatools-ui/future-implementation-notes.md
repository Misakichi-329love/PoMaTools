# PoMa Gym Damage Calculator v3 実装メモ

## PoMaToolsから取り入れるべき長所

- URLでビルドを共有できる点。
- バディーズ選択後にデータが自動入力される点。
- BSBを直感的にON/OFFできる点。
- わざ・パッシブ・BSBを一体として扱う体験。

## 保守しづらそうな点（推定）

- URLパラメータが短く強力な反面、意味が不透明。
- データ・UI・計算が密結合している場合、新パネル追加時の影響範囲が広い。
- 表示文ベースで効果を扱うと多言語化や計算反映が困難。

## 推奨実装方針

1. データ辞書は `pair/move/skill/grid` に分割。
2. IDは内部安定IDと外部互換IDを分離。
3. BSBパネルは `panelId` のみ保存し、効果は辞書から復元。
4. 未対応パネルを壊さず保持できる `unknown` effectを用意。
5. UI状態、保存形式、計算エンジン入力を別型にする。
6. SVGベースのBSB UIでアクセシブルかつテスト可能にする。
7. manifestによるデータバージョン管理を行う。

## JSON保存/読込

```ts
type SavedBuild = {
  schemaVersion: number;
  dataVersion?: string;
  pairId: string;
  build: {
    level: number;
    rarity: number;
    moveLevel: number;
    ex?: boolean;
    exRole?: boolean;
    superAwakeningLevel?: number;
    selectedGridPanelIds: string[];
  };
};
```

## 新規パネル追加への対応

- 既知の `kind` は型付きで処理。
- 未知の `kind` はUI表示と保存のみ行う。
- 計算未対応の場合は警告を出し、ダメージ値には反映しない。
- データ更新テストで「全panel effect kindが登録済みか」を検査する。
