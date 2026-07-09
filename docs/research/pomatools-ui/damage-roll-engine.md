# PoMaTools 完全互換ダメージロール調査メモ

## 調査日・対象

- 調査日: 2026-07-09
- ローカル対象: このリポジトリの `docs/` 配下にある旧 `stdk12/PoMaTools` GitHub Pages成果物。
- 外部対象: 現行PoMaTools公開アプリ `https://pomatools.github.io/#/pairs/28701/102400`。

## 重要な制約

現行ローカルリポジトリには、PoMaToolsのダメージ計算エンジン本体、ビルド済みJavaScript、データJSON、通信ログは含まれていない。`docs/index.html` は移転告知ページであり、実アプリ本体のbundleを読み込まない。

外部アプリはJavaScript必須のSPAとして配信されている。調査環境の通常HTTPクライアントではGitHub PagesへのCONNECTが403で遮断され、アプリbundleやDevTools Network相当のJSONをローカル保存して静的解析するところまでは到達できなかった。一方、ブラウザ検索側ではトップページが「JavaScriptを有効化して使う」旨のHTMLだけを返すことを確認できた。

したがって本メモでは、以下を分けて扱う。

- **確定**: このリポジトリ、公開HTML、既存調査メモ、ユーザー提示の先行解析から直接言えること。
- **高確度推定**: 先行解析の関数名・演算順に整合し、PoMaTools互換エンジンで採用すべき仕様。
- **未確定**: 最新bundleまたはNetwork JSONで追加確認すべき点。

## 現時点の結論

PoMaTools互換を厳密に目指す場合、`damageRoll` は `rawDamage` 表示値から16個を再生成するのではなく、PoMaToolsの内部計算と同じ入力、同じ丸めタイミングで各ロールを直接計算する設計に寄せるべきである。

特に境界値では、以下の2経路が1ダメージずれる可能性がある。

```text
NG寄り: rawDamage = floor(base)
        rollDamage = floor(fround(rawDamage * roll / 100))

PoMaTools互換寄り: rollDamage = floor(fround(unflooredBase * roll / 100))
```

ユーザー提示の先行解析では、PoMaTools側は `attack`、`battlePower`、`defense`、`finalMultiplier`、`critical` を使って各rollを直接計算しているように見える。したがって、PoMa Gym側の実装でも `rawDamage` は表示・デバッグ用の派生値に留め、ロール配列のソースにしない方針を推奨する。

## 推奨する計算責務の分割

```text
BattlePowerCalculator
  basePower
  -> ge: 100基準の加算倍率を合算して floor
  -> pe: 1000基準の可変倍率を順次/合算ルール通り適用して floor

FinalMultiplierCalculator
  条件補正を Fraction numerator/denominator で保持
  -> 最終段まで浮動小数に潰さない

RawDamageCalculator
  UI表示・デバッグ・比較用の代表値を返す
  -> damageRoll生成の唯一ソースにはしない

DamageRollCalculator
  attack, battlePower, defense, critical, finalMultiplier を入力
  -> 各rollごとにPoMaToolsと同じ式・同じMath.froundタイミングで算出
```

## `battlePower` の扱い

先行解析に基づく高確度仕様は次の通り。

1. 技の基礎威力 `basePower` を取得する。
2. `ge` 系の100基準加算倍率を適用する。
   - 例: `basePower * geTotal / 100`。
   - 適用後に `Math.floor`。
3. `pe` 系の1000基準倍率を適用する。
   - 例: `floor(afterGe * pe / 1000)` またはPoMaToolsの順序に合わせた段階適用。
   - 適用後に `Math.floor`。

互換実装では、`ge` と `pe` を単一の浮動小数倍率にまとめない。段階ごとの `floor` がPoMaTools互換性に直結するためである。

## `finalMultiplier` の扱い

先行解析に基づく高確度仕様は次の通り。

- 最終補正は `Fraction`、つまり分子・分母で保持する。
- 各条件を早期に `number` 倍率へ丸めない。
- `damageRoll` 計算時に初めて、PoMaToolsと同じ位置で数値化する。

推奨データ構造例:

```ts
type Fraction = { numerator: number; denominator: number };

type DamageInputs = {
  attack: number;
  defense: number;
  battlePower: number;
  critical: boolean;
  finalMultiplier: Fraction;
};
```

## `damageRoll` の丸めタイミング

ユーザー提示の先行解析から、以下は高確度で採用すべきである。

- 通常ロールは `Math.fround` を通す。
- 11ロール目、すなわち100%ロール、急所時は150%相当の代表ロールだけは `Math.fround` を通さず、直接 `Math.floor` される可能性が高い。
- `rawDamage` から `damageRoll` を作らず、ロールごとに未floorの中間値を保持してから `fround` / `floor` する。

実装イメージ:

```ts
const ROLLS = [85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100];

function calcDamageRolls(input: DamageInputs): number[] {
  return ROLLS.map((roll, index) => {
    const value = calcUnflooredRollValue(input, roll);

    // 先行解析上の「11ロール目」仕様を検証可能な定数に隔離する。
    // 0-based index 10 が該当候補。もしPoMaToolsの配列が別順ならここだけ差し替える。
    if (index === 10) {
      return Math.floor(value);
    }

    return Math.floor(Math.fround(value));
  });
}
```

注意: 上記の `index === 10` は先行解析文言に合わせた互換候補である。PoMaToolsのUI表示順が `[100, 99, ..., 85]` のように逆順であれば、配列上の「11番目」と倍率値の対応が変わる。最新版bundleで、ロール配列の定義とmap順を確認する必要がある。

## ブレイクの扱い

ブレイクはゲーム上、相手の被ダメージ側状態として働く。PoMaTools互換エンジンでは、以下のどちらに分類されているかをbundleで確認する必要がある。

- **finalMultiplier系**: 最終被ダメージ補正として `Fraction` に積む。
- **defense補正系**: 防御/特防側ステータスまたは耐久補正として先に反映する。

互換実装の安全策は、ブレイクを独立した `DamageCondition` として保持し、Adapter層でPoMaToolsの分類へ変換することである。これにより、後から `finalMultiplier` 側か `defense` 側かを差し替えられる。

## テラスタルの扱い

Pokémon Masters EXのテラスタルは、通常の本編ポケモンのタイプ一致補正とは別物として扱う必要がある。PoMaTools互換観点では、以下を分けてモデル化する。

- 技タイプがテラスタイプへ変化するか。
- タイプ一致・弱点・抵抗のどの段で補正されるか。
- テラスタル専用パッシブ、フィールド、シンクロ技条件が `battlePower`、`attack`、`finalMultiplier` のどこへ入るか。

現時点では最新JSONを取得できていないため、テラスタルは `tera: { enabled, type, mode }` のような独立入力として保持し、計算式への注入位置はPoMaTools bundle確認後に固定するのがよい。

## EXロールの扱い

EXロールは、ペアのロール追加・シンクロ技効果・ステータス/パッシブ条件に影響する可変機能として扱うべきである。

推奨モデル:

```ts
type ExState = {
  exEnabled: boolean;
  exRole?: 'strike' | 'tech' | 'support' | 'field' | 'sprint' | string;
  exRoleUnlocked?: boolean;
};
```

計算エンジンへは、EXロールを直接倍率として渡すのではなく、以下へ展開して渡す。

- 追加パッシブ/条件フラグ。
- シンクロ技の対象・威力・倍率に関わる補正。
- UI表示用ロールアイコン/ラベル。

PoMaTools URLの既存調査では `r` や `a` がロール/EXロール関連値を含む可能性が示されているが、現時点では確定していない。

## 最新PoMaTools JSON/通信で確認すべき項目

DevTools Network、またはPlaywright等で現行アプリを実行できる環境を用意し、次を保存する。

1. 初回ロード時の `index.html`、`asset-manifest.json`、`service-worker.js`、main/chunk JS。
2. `/#/pairs/28701/102400` 表示時に取得されるJSON一覧。
3. ダメージ計算時に通信が発生するか、または全計算がクライアントbundle内で完結するか。
4. bundle中のキーワード検索:
   - `rawDamage`
   - `damageRoll`
   - `battlePower`
   - `finalMultiplier`
   - `Math.fround`
   - `critical`
   - `break`
   - `tera`
   - `exRole`
5. ロール配列の順序と「11ロール目」の実インデックス。
6. `ge` / `pe` の集約順序、同種補正の重複ルール。
7. ブレイク、テラスタル、EXロールが、`battlePower`、ステータス、`finalMultiplier`、表示専用のどこに分類されるか。

## PoMa Gym実装への推奨方針

- `rawDamage` は永続APIに残してよいが、`damageRolls` の入力元にしない。
- `DamageTrace` を実装し、各ロールについて `attack`、`defense`、`battlePower`、`criticalMultiplier`、`finalMultiplier`、`roll`、`froundApplied`、`result` を出す。
- PoMaTools互換モードでは丸め順を固定し、将来の検証で差分が見つかった場合はversioned behaviorとして残す。
- ブレイク/テラスタル/EXロールは最初から独立featureとして入力に持ち、PoMaTools Adapterで式注入位置を決める。
