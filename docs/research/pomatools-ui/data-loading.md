# 外部データ参照・ロード

## 確定事項

現行公開GitHubリポジトリの `docs/index.html` は、Bootstrap CSSとローカル画像のみを参照している。バディーズJSON、moves JSON、skills JSON、BSB JSONをロードするコードは存在しない。

## 推奨データロード層

```text
DataRepository
  ├─ loadPair(pairId)
  ├─ loadMoves(moveIds)
  ├─ loadSkills(skillIds)
  ├─ loadGrid(gridId)
  └─ searchPairs(query)

Adapters
  ├─ PoMaToolsUrlAdapter
  ├─ DatamineJsonAdapter
  └─ LegacyBuildAdapter
```

## キャッシュ戦略

- 辞書JSONはアプリ起動時にmanifestだけロード。
- ペア選択後に該当pair/grid/move/skillを遅延ロード。
- `schemaVersion` と `dataVersion` をmanifestに持たせる。
- Service WorkerまたはHTTP cacheを使う場合も、アプリ内でversion mismatchを検出する。

## Pair選択後の自動入力

```text
selectPair(pairId)
  ├─ load SyncPair
  ├─ load referenced moves/passives
  ├─ load grid
  ├─ set default rarity/level/moveLevel
  ├─ reset or migrate selected panels
  └─ project to calculation input
```

## データ正規化

外部データをそのままUIに渡さず、Adapterで内部スキーマに変換する。

```ts
type NormalizedPairBundle = {
  pair: SyncPair;
  moves: Record<string, Move>;
  skills: Record<string, Skill>;
  grid?: GridDefinition;
};
```
