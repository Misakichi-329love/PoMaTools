# PoMaTools UI・データ構造・BSBリバースエンジニアリング調査

## 調査範囲と重要な制約

本調査は、PoMaTools公開GitHubリポジトリ `stdk12/PoMaTools` の現行チェックアウトを対象にした。現行リポジトリには、公開サイト用の静的ランディングページと画像アセットのみが含まれ、React/Angular等のアプリ本体、`moves.json`、`skills.json`、バディーズ辞書、BSB JSON、BSB描画コンポーネントは含まれていない。

- **確定**: ローカルリポジトリの追跡ファイルは `README.md` と `docs/` 配下の静的ファイルのみ。
- **確定**: `docs/index.html` は `https://pomatools.github.io` への移転案内を表示するページであり、アプリ本体のバンドルやJSON参照は含まない。
- **推定**: 実際のPoMaToolsアプリ本体は、別リポジトリ・別デプロイ成果物・非公開ソース・または過去のFirebase/Angularアプリとして管理されていた可能性が高い。
- **未確認**: BSB JSON、moves/skills JSON、React Component構成、Canvas/SVG描画実装は、現行公開GitHubリポジトリだけでは直接確認できない。

## 成果物一覧

- [architecture.md](architecture.md): 現行リポジトリ構成と確認できるアーキテクチャ、制約。
- [sync-pair-data.md](sync-pair-data.md): バディーズデータ構造として期待される項目、現行リポジトリでの確認結果、移植設計案。
- [moves-json.md](moves-json.md): 技データ参照の確認結果とPoMa Gym向けスキーマ案。
- [skills-json.md](skills-json.md): パッシブ/スキル辞書の確認結果と拡張可能な設計案。
- [bsb-data.md](bsb-data.md): BSBデータ構造の調査結果、未確認点、推奨スキーマ。
- [bsb-panel-effects.md](bsb-panel-effects.md): パネル効果の分類と計算エンジン連携向け正規化案。
- [bsb-image-rendering.md](bsb-image-rendering.md): 画像・SVG・Canvas描画に関する確認結果と推奨描画方式。
- [bsb-ui.md](bsb-ui.md): クリック処理、選択状態、接続判定、エネルギー計算の設計案。
- [data-loading.md](data-loading.md): JSONロード、キャッシュ、検索、Adapter層の設計案。
- [component-structure.md](component-structure.md): PoMa Gym Damage Calculator v3向けComponent構成案。
- [future-implementation-notes.md](future-implementation-notes.md): 移植・改善・保守性向上の提案。
- [open-questions.md](open-questions.md): 追加検証が必要な事項。

## 結論

PoMaToolsの「公開GitHubリポジトリ」からは、ユーザーが期待するバディーズ辞書・わざ/パッシブ辞書・BSB JSON・BSB画像UIの実装詳細は取得できなかった。したがって本仕様書は、以下を明確に分離して記述する。

1. **確定**: 現行リポジトリ内で直接確認できた事実。
2. **推定**: 公開URL形式やPoMaToolsの用途から合理的に推測できる事項。
3. **設計提案**: PoMa Gym Damage Calculatorへ実装する際に、PoMaToolsの長所を取り込みつつ保守性・拡張性を高めるための推奨構造。
