# Sync Pair データ構造

## 確定事項

現行公開GitHubリポジトリには、Sync Pair辞書、Pair ID定義、キャラクターJSON、ステータスJSON、ロール/EX/超覚醒データは含まれていない。

## 推定されるPoMaTools上のキー構造

流通URLでは `/pairs/900000/2500` や `/pairs/4/600` のように2つの数値セグメントでバディーズを識別している。これは以下のどちらかと推定される。

1. `trainerId` + `pokemonId/formId`
2. `baseTrainerId` + `pokemonVariantId`
3. `pairId` を2分割した互換キー

PoMa Gymでは、ゲーム内更新や別ソース取り込みに強くするため、内部キーは単一の安定IDに正規化し、外部キーをaliasesとして保持するのが望ましい。

## 推奨スキーマ

```ts
type SyncPair = {
  pairId: string;
  sourceIds: {
    pomatools?: { pathA: string; pathB: string };
    game?: string;
    datamine?: string;
  };
  trainer: {
    id: string;
    name: Record<Locale, string>;
    tags: string[];
  };
  pokemon: {
    id: string;
    name: Record<Locale, string>;
    form?: string;
    type: PokemonType;
    weakness?: PokemonType;
  };
  role: {
    primary: 'strike' | 'tech' | 'support' | 'sprint' | 'field';
    exRole?: 'strike' | 'tech' | 'support' | 'sprint' | 'field';
  };
  rarity: { base: number; max: number };
  stats: {
    byLevel: Record<number, Stats>;
    lv150?: Stats;
    lv200?: Stats;
  };
  moves: Array<{ slot: number; moveId: string; user: 'pokemon' | 'trainer' }>;
  syncMove: { moveId: string; exEffectId?: string };
  passives: string[];
  gridId?: string;
  superAwakening?: {
    maxLevel: number;
    passiveByLevel?: Record<number, string[]>;
  };
};
```

## UI利用フロー案

1. Pair selectorで `pairId` を選ぶ。
2. Adapterが外部IDを内部 `pairId` に変換。
3. `SyncPair`、`MoveDictionary`、`SkillDictionary`、`GridDefinition` をまとめてロード。
4. UI初期値としてレベル、★、わざレベル、EX、超覚醒、BSB初期状態をセット。
5. 計算エンジンにはUI生データではなく正規化済み `CalculationBuild` を渡す。

## 改善ポイント

- Pair IDをURL互換IDに依存させず、内部IDと外部IDを分ける。
- レベル別ステータスは事前計算済みテーブルと計算式のどちらにも対応できるようにする。
- EX/EXロール/超覚醒は可変機能なので、nullableフィールドではなくFeature配列として扱う設計も有効。
