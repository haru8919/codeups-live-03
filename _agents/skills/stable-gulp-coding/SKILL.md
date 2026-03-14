---
name: Stable Gulp Coding
description: Guidelines and workflows for coding with the WordPress theme's Gulp environment, including strict adherence to Sass, BEM, and margin-less block standards.
---

# Stable Gulp Coding Skill

このスキルは、本プロジェクト（WordPressテーマ）におけるGulp環境を用いたコーディングの標準作業手順（SOP）を定義するものです。
AIエージェントがコードを生成・編集する際は、本スキルに記載されたGulpの構成、BEMアーキテクチャ、およびコーディング規約を厳格に遵守します。

## 1. 環境とセットアップ (Environment & Setup)

- **プロジェクトのルート**: テーマのルートディレクトリ（`_gulp`や`src`フォルダが配置されている場所）を基準とします。
- **Gulpのセットアップと操作**:
  - Gulp設定ファイル（`gulpfile.js`）および`package.json`は `_gulp` ディレクトリ内に配置されています。
- **Gulpのセットアップと操作（静的コーディングフェーズ）**:
  - 依存関係のインストール: `cd _gulp && npm i`
  - 開発の開始（監視モード・ブラウザ同期）: `cd _gulp && npx gulp`
  - 本番用ビルド（ファイル整理・圧縮等）: `cd _gulp && npx gulp build`
- **Gulpのセットアップと操作（WordPress化フェーズ）**:
  - 静的コーディング完了後、`gulpfile.js`内のWordPress反映用設定（`browserSyncOption`の`proxy`や保存先パスなど）を有効化／書き換えます。
  - その後、node_modulesの再構築を行うワークフローとなります。
- **注意点**: タスクランナーはSassのコンパイル、BabelによるJSのトランスパイル、画像の圧縮、HTML/PHPファイルのコピーとブラウザ同期を自動的に行います。

## 2. ディレクトリ構成 (Directory Architecture)

- **`src/` フォルダ**: 編集・作成を行うのは**必ず**このディレクトリ内のファイルのみです。
  - `src/sass/**/*.scss` : Sassファイル
  - `src/js/**/*` : JavaScriptファイル
  - `src/images/**/*` : 画像ファイル
  - `src/**/*.html` : HTMLファイル
- **`dist/` および `WordPressTheme/assets/` フォルダ**: 
  - これらはGulpによって自動生成・出力されるフォルダです。**直接編集してはいけません**。
- **WordPress用ファイル**: `WordPressTheme/**/*.php` （必要に応じて編集可ですが、Gulpの管轄内で同期されます）
- **`base` フォルダ**: 基本的な設定ファイルなどが入っているため、安易に変更を加えないこと。

## 3. コーディング規約 (Coding Standards)

### 3.1 CSS / Sass のガイドライン
- **アーキテクチャ**: FLOCSS に準拠したディレクトリ構造（`global`, `base`, `module`, `page`など）を使用します。
- **BEM 命名規則**: Block, Element, Modifier（BEM）の命名規則を厳格に使用します（例: `.block`, `.block__element`, `.block--modifier`）。
  - ネストは過度に深くせず、フラットな構造（Flat BEM）を心がけます。
- **マージンを持たないブロック（Margin-less Blocks）**: 
  - Blockコンポーネント自身には外側のマージン（`margin-top`, `margin-bottom` 等）を直接設定してはいけません。
  - コンポーネント間の余白は、ラッパー・コンテナ要素や、親のレイアウト要素（レイアウトモジュール）側で制御します。（このラッパー要素を配置することによって生じるHTML階層の深化は、コンポーネントの独立性を保つための意図された設計です）
- **ブレイクポイント / メディアクエリ**:
  - `src/sass/global/_breakpoints.scss` の変数を元に、スマホファースト（`sp`）かPCファースト（`pc`）かを切り替える仕様になっています（初期値は`sp`）。
  - メディアクエリは必ず共通のmixin（例: `@include mq(md) { ... }`）を用いて記述します。
    - **`mq($mediaquery)`**: ブレイクポイント（sm, md, lg, xl）を指定してメディアクエリを展開します（初期値はmd）。
- **頻出する関数 (Functions)**:
  - `src/sass/global/_function.scss` に定義されている便利な関数を積極的に使用してください。
    - **`vw($window_width, $size)`**: ピクセル値をVW（ビューポート幅）単位に変換します。レスポンシブなサイズ指定に有用です。
    - **`rem($pixels)`**: ピクセル値をREM単位に変換します。フォントサイズや余白の指定に有用です（16px基準）。
    - **`strip-unit($number)`**: 数値から単位を取り除きます。

### 3.2 HTML / JS のガイドライン
- HTMLのclass名は、上記CSS/SassのBEM規約に基づいた名前を割り当てます。
- **堅牢なHTML構造設計（意図的な階層化の許容）**:
  - **責務の分離（レイアウトと装飾）**: セクション構築時は、全体の大枠（背景や上下余白用）、最大幅制限・中央揃え用（`.inner`等）、内部要素の配置・グループ化用（`.wrapper`等）の役割ごとに `div` 等タグを分割し、1つの要素に複数の責務を持たせないこと。
  - **動的コンテンツへの耐性（WordPress化前提）**: 将来のCMS組み込みや運用によるテキスト文字数の増減、画像の比率変更時にレイアウトが崩れないよう、HTMLを細かく箱（タグ）で区切り、堅牢な構造にすること。
  - **表現力の担保（アニメーションと重なり）**: 高度なアニメーションや複雑なz-indexの重なり制御のために、`<span>` 要素や擬似要素を積極的に配置すること。これによりHTMLのネストが深くなるのは、保守性と表現力を両立するための正しいアプローチとする。
- **レスポンシブなヒーロー画像（FVなど）の実装**: 
  - メインビジュアルなどで画面幅に応じて画像を出し分ける際は、`<picture>` タグを使用して実装することを推奨します。
  - HTMLの記述例（`765px` をブレイクポイントとする場合）:
    ```html
    <picture>
      <source srcset="./assets/images/common/mv-pc4.jpg" media="(min-width:765px)" />
      <img src="./assets/images/common/mv4.jpg" alt="海亀とダイバーが泳いでいる様子" />
    </picture>
    ```
- JavaScriptはES6+のモダンな構文で記述します。保存時にGulp（Babel）が自動で各ブラウザ対応のためのトランスパイルを行います。

## 4. エージェントの実行ワークフロー (Agent Execution Workflow)

1. **要件分析**: 指示されたデザインや機能要件を確認する。
2. **ファイル編集**: `src/` フォルダ内の該当ファイル（HTML, SCSS, JS）のみを編集する。
3. **規約チェック**: コードがBEM構造に従っているか、Blockがマージンを持っていないかなどを自己レビューする。
4. **反映確認**: ユーザー環境で `npx gulp` が実行されていることを前提に、編集内容が正しく `dist/` に出力される状態にする。
