# Open Questions

## 要検証事項

1. 実アプリ本体のソースコード所在。
   - 現行 `stdk12/PoMaTools` には存在しない。
   - `pomatools.github.io` のデプロイ元、旧 `ng-pomatools.web.app`、または非公開リポジトリの確認が必要。

2. `g=` パラメータのエンコード方式。
   - panelId配列、bitset、差分、Base64風エンコードのいずれか要検証。

3. `/pairs/{a}/{b}` のID意味。
   - trainerId/pokemonIdなのか、pairId/formIdなのか要検証。

4. moves/skills/BSB JSONの実フィールド。
   - 現行公開GitHubリポジトリにはJSONがない。

5. BSB描画方式。
   - SVG/Canvas/CSS/画像マップのいずれか未確認。

6. 超覚醒データの有無。
   - 現行公開GitHubリポジトリでは確認不可。

## 次の調査候補

- ブラウザDevToolsで `https://pomatools.github.io` のNetworkタブを確認し、JS bundleとJSON asset URLを特定する。
- 過去の `ng-pomatools.web.app` のアセットやソースマップが残っているか確認する。
- GitHub Pagesの別organization/repositoryに実アプリ成果物が存在するか確認する。
- PoMaTools作者またはコミュニティが公開しているデータ更新手順を探す。

## ダメージロール完全互換で追加検証が必要な事項

- 現行 `https://pomatools.github.io/#/pairs/28701/102400` のbundleまたは通信JSONを保存し、`damageRoll` が `rawDamage` 経由ではなく `attack` / `battlePower` / `defense` / `finalMultiplier` から直接計算されるか確認する。
- `Math.fround` が適用される正確な位置と、11ロール目だけ直接 `Math.floor` されるという先行解析の配列インデックスを確認する。
- ブレイク、テラスタル、EXロールが `battlePower`、ステータス、防御側補正、`finalMultiplier`、表示専用のどこに注入されるかを確認する。
- `ge` と `pe` の重複・集約・floor順が、最新版PoMaTools bundleでも先行解析通りか確認する。
