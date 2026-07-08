# moves.json 調査

## 確定事項

現行公開GitHubリポジトリには `moves.json` は存在しない。したがってPoMaTools内部のID体系、フィールド名、UI参照方法は直接確認できない。

## 推奨ID体系

技は同名・別性能・強化後・B技・シンクロ技で衝突しやすい。PoMa Gymでは表示名をIDにしない。

```ts
type Move = {
  moveId: string;
  sourceIds?: Record<string, string>;
  name: Record<Locale, string>;
  category: 'physical' | 'special' | 'status' | 'sync' | 'max' | 'buddy';
  type: PokemonType | 'trainer';
  target: string;
  gauge?: number;
  uses?: number;
  power?: { base: number; min?: number; max?: number; byMoveLevel?: number[] };
  accuracy?: number;
  effectTags: string[];
  skillEffectIds?: string[];
  flags?: string[];
};
```

## バディーズとの関連付け

`SyncPair.moves[].moveId` から辞書参照する。BSBの「技威力+N」「Move Gauge Refresh」「命中率+」などは、パネル側で `targetMoveId` を持たせる。

```ts
type GridMovePowerEffect = {
  kind: 'movePowerUp';
  targetMoveId: string;
  amount: number;
};
```

## UIでの利用方法

- Pair詳細: スロット順にmoveIdを解決して表示。
- BSB tooltip: `targetMoveId` を名前に解決し、「Thunderbolt: Power +3」のように表示。
- 計算エンジン: 選択中技IDとBSB効果配列から最終威力補正を算出。

## 移植時の注意点

- 同名技が別IDで存在する可能性を前提にする。
- B技・シンクロ技・Buddy Moveは通常技と同じ辞書で扱いつつ、`category` で分岐する。
- `power` の変動は「わざレベルによる基礎値」と「BSBパネルによる加算」を分ける。
