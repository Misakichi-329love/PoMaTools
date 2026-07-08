# BSB 画像・描画方式

## 確定事項

現行公開GitHubリポジトリには、BSBパネル画像、アイコンセット、Canvas/SVG/React描画コンポーネントは含まれていない。含まれる画像は移転告知ページ用の背景とアイコンのみである。

## 推定

PoMaToolsは「BSB画像による直感的な入力UI」を提供していたため、実装方式は以下のいずれかと考えられる。

- SVG hex gridを動的生成。
- Canvasにhex/アイコン/選択状態を描画。
- CSS absolute positioningで各パネルを配置。
- ゲーム内BSB画像を背景として重ね、透明クリック領域を配置。

現行リポジトリだけではSVG/Canvas/CSSのどれかは未確認。

## PoMa Gym向け推奨方式

Reactとの相性、アクセシビリティ、状態同期、テスト容易性を考えると、**SVGベースのhex grid** が最も扱いやすい。

```text
<GridSvg viewBox="...">
  <ConnectionLines />
  <PanelHex id="..." state="selected|available|locked" />
  <PanelIcon />
  <PanelLabel />
</GridSvg>
```

## JSONから動的生成する設計

- `position.q/r` からSVG座標を算出。
- `type` から色・アイコンを決定。
- `selectedPanelIds` から選択状態を決定。
- `unlock` と接続判定から `available/locked` を算出。

## 改善ポイント

- ゲーム内画像を背景にしたクリックマップは見た目再現性が高いが、レスポンシブ・保守・座標調整が難しい。
- SVGならパネルIDをDOMに持たせられ、テストとアクセシビリティが容易。
- Canvasを使う場合はヒットテストとキーボード操作を別実装する必要がある。
