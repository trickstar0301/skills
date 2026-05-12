---
name: html-deck
description: HTML 単一ファイルで完結する slide deck を生成する skill。CSS と最小限の JavaScript (~25 行) のみ、reveal.js / Slidev / Spectacle 等のフレームワークは入れない。Tariq Krim "The Unreasonable Effectiveness of HTML" のミニマル流儀。frontend-design skill と併用して見た目品質を上げる。「スライド作って」「プレゼン作って」「HTML でデッキを」「週次レポートをスライドで」「成果報告をブラウザで見れる形で」「PDF にもしたい」のような依頼で必ずトリガー。逆に登壇用の重い presentation 機能（speaker notes / 複雑 transition / live drawing）が必須なら本 skill は使わず reveal.js 直書きを案内する。
---

# HTML Deck

Tariq Krim 流の **単一 HTML ファイル + 最小限の JS** で slide deck を作る skill。CSS だけで完結する 2 つの variant (scroll-snap / 固定サイズ) を持ち、後者は PDF 出力にも素直に対応する。
ブラウザで開けば即動く、S3 や GitHub Pages にアップすればすぐ共有できる軽量プレゼン形式。

## いつ使うか

- 週次・月次の社内共有、PR explainer、incident report、研究まとめ
- ブラウザを開くだけで読める形にしたいレポート
- スライドという形をした「読み物」（登壇 != 読まれる）

使わない場面:
- 登壇現場で speaker view・複雑な transition・live drawing が必要 → reveal.js を案内
- 数十枚を長期メンテする社内資料 → Slidev / Keynote を案内
- インタラクティブな explorer/プレイグラウンド → 別 skill (playground 等)

PDF 出力は固定サイズ variant でカバーできる（後述）。

## 原則 (必ず守る)

1. **単一 HTML ファイル**。CSS と JavaScript は `<style>` / `<script>` で同梱。**外部依存禁止**（フォントは system stack で代用、CDN も入れない）
2. **フレームワーク禁止**。reveal.js / Slidev / Spectacle / impress.js は使わない。Tailwind CDN すら入れない（CSS custom properties で代用）
3. **JavaScript は ~25 行が目安**。variant ごとに 3 部品（実装はテンプレ参照）:
   - scroll-snap: `scroll-snap-type` + `scrollIntoView` + `IntersectionObserver`
   - 固定サイズ: `transform: scale()` で viewport fit + `--current` CSS var + `#track` の `translateY`
4. **frontend-design skill を併用**。視覚デザインの品質はこのスキルの責務外、frontend-design 側で担保

## アーキテクチャ

```html
<!-- scroll-snap variant (default) -->
<!DOCTYPE html>
<html>
<head>
  <style>
    :root { /* design tokens */ }
    .slide { width: 100vw; height: 100vh; scroll-snap-align: start; }
    body { scroll-snap-type: y mandatory; }
  </style>
</head>
<body>
  <section class="slide" id="s1">...</section>
  <section class="slide invert" id="s2">...</section>
  <section class="slide" id="s3">...</section>
  <div id="counter">1 / 3</div>
  <script>(function(){ /* nav + observer */ })();</script>
</body>
</html>
```

固定サイズ variant の骨組みは `template-fixed.html` (1280×720 stage + `transform: scale()` + `#track` translate + footer HUD + `@media print`)。

詳細は同梱の 3 つのテンプレを参照:

- `template-minimal.html` — 最低限の動く骨組み（~80 行）。scroll-snap、素早く始めるならこれをコピー
- `template-full.html` — 完全版（~400 行）。design tokens、複数スライドタイプの component（title / list / metric / decision / next-steps）、レスポンシブ対応つき。scroll-snap ベース
- `template-fixed.html` — 固定 1280×720 + transform scale + footer HUD + `@media print` 同梱（~215 行）。PDF も毎回出す運用ならこれ。component は `template-full.html` から借りる

## Variant 選択: scroll-snap vs 固定サイズ

| | scroll-snap (default) | 固定サイズ + scale |
|---|---|---|
| Layout | `vh / vw / clamp()` で流動 | `1280×720 px` 固定（1 座標系） |
| 画面 = PDF | × （`@media print` で破綻しがち） | ◯ （same rendering engine） |
| PDF 出力 | 非推奨 | 1 slide = 1 page で素直 |
| 配布が HTML のみ | ◯ | ◯ （footer HUD 込み） |
| 元テンプレ | minimal / full | fixed |

判定:
- **「毎回 PDF も配布する」なら fixed**、それ以外は scroll-snap
- 迷うなら scroll-snap で始める。後から fixed に移行する場合、design tokens と component CSS はそのまま流用できる（座標系の置換だけが必要）
- 両者を混ぜない（座標系が増えて 2 重管理になる）

## ワークフロー

### Step 1: 入力整理

- ユーザーから内容（要点・データ・引用）を受け取る
- **配布形式を確認**: HTML だけ / PDF も毎回必要か → variant が決まる（未確認なら聞く）
- スライド枚数の目安: **5〜8 枚**（多すぎると単一 HTML としては重くなる）
- 各スライドに何を入れるか箇条書きで一覧化してから着手

### Step 2: frontend-design を呼ぶ

視覚デザインの方針（色・タイポグラフィ・コンポーネント構成）は frontend-design skill に委譲。
**呼び方**: Skill tool で `frontend-design:frontend-design` を invoke、または LLM の内部判断で frontend-design 風の設計を行う。design tokens（`--primary`, `--bg`, `--text` 等）は CSS custom properties で定義し、ブランド／トーンに合わせる。

### Step 3: テンプレを基に書く

variant が確定していれば対応テンプレをコピーして content を差し替える:

- **scroll-snap**: 軽量なら `template-minimal.html`、component 多めなら `template-full.html`
- **固定サイズ**: `template-fixed.html` をコピー → 必要な component CSS を `template-full.html` から借りて貼り込む（design tokens は :root に併合）

未決なら Variant 選択セクションを参照して決める。

### Step 4: 出力先

- ユーザーが場所を指定したらそこに
- 指定がない場合: `./<topic-slug>.html` をプロジェクト直下に
- 出力後、Claude Code 内で `open <path>` を提案するか、ファイルパスのみ返す

### Step 5: 仕上げの check

- ブラウザでロード時に矢印キーで移動できるか
- 1 枚目に title、最後の枚に decision / next steps / questions
- 各スライドは **単独で読めるか**（前のスライドの文脈なしで理解できるか）。スレッド型 X 投稿と同じ評価基準
- counter が右下（または footer HUD）に出ているか、`1 / N` 形式
- 固定サイズ variant なら手元で `just pdf` 実行 → 生成された PDF を視覚確認（背景色保持、改ページ、font tofu の有無）

## PDF として書き出す（固定サイズ variant のみ）

`template-fixed.html` は `@media print` で 1 slide = 1 page になる。headless Chromium で render すれば運用が楽。

**最小スクリプト** `scripts/render-pdf.mjs` (Playwright):
```js
import { chromium } from 'playwright';
import { resolve } from 'node:path';
import { pathToFileURL } from 'node:url';
const [input, output] = process.argv.slice(2);
const browser = await chromium.launch();
const page = await browser.newPage({ viewport: { width: 1280, height: 720 } });
await page.goto(pathToFileURL(resolve(input)).href, { waitUntil: 'load' });
await page.pdf({ path: resolve(output), width: '1280px', height: '720px', printBackground: true, margin: { top: 0, right: 0, bottom: 0, left: 0 } });
await browser.close();
```

**justfile レシピ**:
```just
pdf:
    pnpm add --silent playwright
    pnpm exec playwright install chromium
    pnpm exec node scripts/render-pdf.mjs deck.html deck.pdf
```

注意:
- `@page size: 1280px 720px` は Chromium 前提（Playwright で使うので運用は OK）。Firefox / Safari の print プレビューでは layout が崩れる可能性あり
- フォント: system stack は PDF embed されない → 配布先で見た目が変わる。Linux Chromium に CJK fallback が無い環境では日本語 tofu リスク。必要なら Noto Sans JP の woff2 を `data:` URI で `@font-face` 同梱（サイズ膨らむので default は off）
- Claude Code のサンドボックスでは Chromium が起動できないので、PDF 検証はユーザー手元で `just pdf` する旨を伝える
- 用紙サイズ: 1280×720 px ≈ 338×190 mm (16:9)。A4/Letter とは違うので、物理印刷時は viewer の「用紙に合わせる (Fit to page)」が必要。スクリーン閲覧のみなら問題なし

## 落とし穴

- **コンテンツを詰め込みすぎる**: 1 枚 1 メッセージが基本。3 つ以上 bullet を並べるなら別スライドに分けるか、横並びの grid にする
- **font の依存**: Google Fonts や Adobe Fonts を CDN で入れない。`system-ui, -apple-system, "Segoe UI", Roboto, sans-serif` などのスタックで完結させる。serif/mono も同様
- **build process を提案する**: 「Vite/Webpack でビルドして…」は本 skill の範囲外。1 HTML で完結
- **JavaScript が肥大化**: ~25 行の目安を超えそうになったら設計を疑う。scroll-snap が解決する問題を JS で書き直していないか
- **mobile を放置 (scroll-snap variant)**: viewport meta を入れ、`@media (max-width: 640px)` で grid を 1 列に畳むなど、最低限のレスポンシブ対応
- **固定サイズ variant の mobile**: 意図的に縮小表示する設計（1 座標系を維持するため）。phone は横画面 or pinch zoom 前提となるので、ユーザーにその旨伝える
- **「ですます」一辺倒**: 文体は読み物の性格に合わせる。週次 status なら「だ・である」が締まる
- **アクセシビリティ無視**: alt 属性、`aria-hidden="true"` を装飾 SVG に、見出し階層 (h1 → h2 → h3) を守る
- **letterbox 余白が目立つ (固定サイズ variant)**: body 背景を slide と違う色にすると 16:9 が viewport にフィットしない領域が「無駄領域」に見える。`body { background: var(--ivory) }` のように slide と同色にし、`#stage` に `box-shadow` で縁取りを入れると消える
- **counter/hint が viewport サイズで隠れる (固定サイズ variant)**: `position: fixed` だけで `slide` 内の content と座標が衝突したり、`mix-blend-mode: difference` が背景色で破綻したりする。viewport 下端の footer bar (`#hud`) に counter + hint をまとめ、`fit()` で footer 高さを scale 計算から引く。`@media print { #hud { display: none } }`
- **初回ロードで巨大な #stage が一瞬見える (固定サイズ variant)**: `--scale` の初期値 1 のまま `fit()` 実行前に paint されるため。`#stage { visibility: hidden }` をデフォルトにし、`fit()` 末尾で `document.documentElement.classList.add('ready')` → `html.ready #stage { visibility: visible }` で隠す
- **slide content が 720px 高さを超える (固定サイズ variant)**: `#stage { overflow: hidden }` で見切れる。1 slide の content は padding (56×80) 込みで 608px × 1120px 以内に収まるよう設計。超えるなら別スライドに分割

## frontend-design との連携パターン

```
ユーザー: "Q2 振り返りのスライド作って、データはこれ：..."
↓
Step 1. 要点整理（agent が自動でやる）
Step 2. frontend-design skill を invoke。design tokens 設計、palette 提案、typography 決定
Step 3. template-full.html ベース、Step 2 の tokens を :root に注入、各スライドの content 流し込み
Step 4. ./q2-retrospective.html に書き出し
Step 5. ユーザーに開いて確認してもらう
```

## 関連

- `blog-style` — 長文記事向け文体評価。本 skill とは出力形式が違う（HTML deck vs zenn/dev.to 記事）
- `frontend-design:frontend-design` — 視覚デザイン担当、本 skill が必ず併用する相棒
- `playground` plugin の `playground` skill — インタラクティブ explorer 生成、deck より構築が大きい
- 参考: [thariqs.github.io/html-effectiveness/09-slide-deck.html](https://thariqs.github.io/html-effectiveness/09-slide-deck.html) — 元ネタの slide demo
- 参考: [Tariq Krim "The Unreasonable Effectiveness of HTML"](https://x.com/trq212/status/2052809885763747935) — 設計思想の出典
