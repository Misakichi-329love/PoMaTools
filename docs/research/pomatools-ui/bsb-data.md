# BSB データ構造

## 確定事項

現行公開GitHubリポジトリにはBSB JSON、パネルID、座標、接続情報、unlock条件、必要エネルギー、パネル効果定義は含まれていない。

## 推定されるBSB要素

PoMaToolsの共有URLには `g=AAECAw...` のような圧縮されたグリッド選択状態が含まれる。これは選択パネルIDリストを短い文字列にエンコードしたものと推定される。

BSBデータとして最低限必要な要素は以下。

- board/grid ID
- panel ID
- hex座標またはピクセル座標
- 隣接/接続情報
- unlock条件: わざレベル、拡張、特定パネル前提など
- energy cost
- orb cost
- panel type
- effects

## 推奨スキーマ

```ts
type GridDefinition = {
  gridId: string;
  pairId: string;
  schemaVersion: number;
  layout: {
    coordinateSystem: 'axial' | 'offset' | 'pixel';
    originPanelId: string;
  };
  panels: GridPanel[];
};

type GridPanel = {
  panelId: string;
  label: Record<Locale, string>;
  type: 'stat' | 'movePower' | 'moveAccuracy' | 'grantSkill' | 'mpr' | 'mgr' | 'syncPower' | 'special';
  position: { q: number; r: number } | { x: number; y: number };
  neighbors?: string[];
  unlock: {
    moveLevel?: number;
    requiresPanels?: string[];
    requiresExpansion?: string;
  };
  cost: { energy: number; orbs?: number };
  effects: GridPanelEffect[];
  icon?: string;
};
```

## 接続判定

推奨ルール:

1. 中央パネルは常に選択済みまたはコスト0。
2. パネルが選択可能になる条件は `unlock` と接続性のAND。
3. 接続性は「選択済みパネル集合 + 中央」から対象パネルまで隣接選択パネルが存在すること。
4. パネルOFF時は、OFF後に残る選択済み集合がすべて中央に連結しているか検証する。

## データ構造上の改善余地

- `neighbors` を手書きすると更新漏れが起きるため、hex座標から自動導出できる場合はビルド時生成する。
- ただし特殊な非標準接続がある場合に備え、`neighborsOverride` を許容する。
- パネル効果は構造化し、表示文・計算式・UIアイコンを分離する。
