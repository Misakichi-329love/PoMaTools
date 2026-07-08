# skills.json 調査

## 確定事項

現行公開GitHubリポジトリには `skills.json` は存在しない。PoMaTools内部のスキルID、効果式、重複処理は直接確認できない。

## 推奨スキーマ

```ts
type Skill = {
  skillId: string;
  name: Record<Locale, string>;
  description: Record<Locale, string>;
  category: 'passive' | 'lucky' | 'grid' | 'superAwakening' | 'innate' | 'theme';
  effectTags: string[];
  effects: SkillEffect[];
  stacking?: 'stack' | 'max' | 'unique' | 'replace';
};
```

## BSBとの関連

BSBの「パッシブ追加」パネルは、パネル自体に効果式を重複して持たせるより、`skillId` 参照にする。

```ts
type GridPassiveEffect = {
  kind: 'grantSkill';
  skillId: string;
};
```

## 同一効果の重複処理

PoMa Gymでは、スキル辞書側に `stacking` と `stackGroup` を持たせることで、UIと計算エンジンが同じルールを共有できる。

```ts
type SkillEffect = {
  kind: string;
  params: Record<string, unknown>;
  stackGroup?: string;
  stacking?: 'additive' | 'multiplicative' | 'maxOnly' | 'unique';
};
```

## 改善ポイント

- 表示文から効果をパースしない。
- 効果式は構造化し、表示文はローカライズ専用にする。
- 未対応スキルは `kind: 'unknown'` で保持し、UI表示と保存を壊さない。
