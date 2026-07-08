# Architecture 調査

## 関連ファイル

- `README.md`
- `docs/index.html`
- `docs/assets/theme.png`
- `docs/assets/icon-152x152.png`
- `docs/favicon.ico`

## 確定事項

現行リポジトリは、GitHub Pages用の静的サイトとして構成されている。`docs/index.html` はPoMaToolsのメタ情報、Bootstrap CDN、背景画像、移転告知を含むが、アプリ本体のJavaScriptバンドルやデータJSONをロードしていない。

確認できる外部依存はBootstrap CDNのみで、アプリ本体に相当するReact Component、Angular Component、データロード処理、BSB描画処理は含まれていない。

## 推定アーキテクチャ

PoMaToolsの実アプリは、少なくとも以下の責務を持つ別成果物として存在していた可能性が高い。

```text
App Router
  ├─ Pair route: /#/pairs/:trainerId/:pokemonId?...&g=...
  ├─ Team route: /#/team?...g0=...
  ├─ Pair selector
  ├─ Sync grid / BSB editor
  ├─ Damage/result side panel
  └─ Move/Skill finder

Data layer
  ├─ Sync pair dictionary
  ├─ Moves dictionary
  ├─ Skills dictionary
  ├─ Sync grid/BSB boards
  └─ image/icon assets
```

## URLから推定できる状態モデル

公開例として流通しているPoMaTools URLは次の形式を持つ。

```text
/#/pairs/{trainerOrPairId}/{pokemonOrFormId}?s=5&l=140&r=5&p=0&a=0&g=AAECAw...
/#/team?v0=...&s0=...&l0=...&r0=...&g0=...
```

**推定**:

- `pairs/{id1}/{id2}` がバディーズ特定キー。
- `s` は★数または初期レアリティ/現在レアリティ。
- `l` はレベル。
- `r` はわざレベルまたはロール/EXロール関連値の可能性。
- `p` はポテンシャルまたはパッシブ/ポテンシャル選択。
- `a` は追加状態、超覚醒、EXロール、テーマ等の圧縮値の可能性。
- `g` は選択済みBSBパネルの圧縮エンコード。

## PoMa Gym向け改善案

PoMaToolsのURL圧縮は共有性に優れる一方、意味がURLから読み取りにくい。PoMa Gymでは以下を分離するのが望ましい。

- 永続保存用: version付きJSON。
- 共有URL用: JSONを圧縮した短縮表現。
- 内部状態用: 型付きオブジェクト。
- 計算エンジン入力用: 正規化済み `CalculationBuild`。

```ts
type BuildSave = {
  schemaVersion: 3;
  pairKey: string;
  level: number;
  rarity: number;
  moveLevel: number;
  ex: { enabled: boolean; role?: string };
  superAwakening?: { level: number; passiveId?: string };
  grid: { selectedPanelIds: string[] };
};
```
