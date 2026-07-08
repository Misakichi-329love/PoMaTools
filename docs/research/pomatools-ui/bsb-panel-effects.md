# BSB Panel Effects

## 確定事項

PoMaTools公開GitHubリポジトリからは、パネル効果の実データ構造や重複処理は確認できない。

## 推奨効果モデル

```ts
type GridPanelEffect =
  | { kind: 'statUp'; stat: keyof Stats; amount: number }
  | { kind: 'movePowerUp'; targetMoveId: string; amount: number }
  | { kind: 'syncMovePowerUp'; targetMoveId?: string; amount: number }
  | { kind: 'moveAccuracyUp'; targetMoveId: string; amount: number }
  | { kind: 'grantSkill'; skillId: string }
  | { kind: 'moveGaugeRefresh'; targetMoveId: string; chanceRank: number }
  | { kind: 'mpRefresh'; targetMoveId: string; chanceRank: number }
  | { kind: 'unknown'; raw: unknown };
```

## パネル種類ごとの扱い

### 能力値パネル

- UI: HP/攻撃/防御/特攻/特防/素早さを色分け。
- 計算: `statUp` を合算。
- 保存: panelIdのみ保存し、効果は辞書から復元。

### パッシブ追加

- UI: `skillId` をスキル辞書で解決して説明表示。
- 計算: `Skill.effects` を展開。
- 重複: `Skill.stackGroup` と `stacking` に従う。

### 技威力上昇

- UI: 対象技名 + `Power +N`。
- 計算: 基礎威力に加算。倍率補正と混ぜない。

### Move Gauge Refresh / MP Refresh

- UI: 対象技と確率ランクを表示。
- 計算: ダメージ計算対象外なら、行動期待値・補助評価用タグとして保持。

### その他特殊パネル

未知パネルは `unknown` として保存可能にする。これによりデータ更新直後でもUIが壊れず、計算未対応であることを明示できる。

## 同一効果の重複処理

PoMa Gymでは次の3層に分ける。

1. **Panel aggregation**: 選択panelIdからeffectsを集める。
2. **Effect normalization**: 同種効果を `target`, `kind`, `stackGroup` でグループ化。
3. **Engine projection**: ダメージ計算に必要な効果だけ抽出。
