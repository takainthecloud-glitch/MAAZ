# MAAZ — Zero Trust Maturity Atelier

**English summary:** MAAZ is a single-file HTML assessment tool that scores an organization's zero-trust maturity against the **CISA Zero Trust Maturity Model v2** (6 pillars × 18 controls × 4 levels) and cross-maps that maturity onto threat coverage from **MITRE ATT&CK** and **MITRE ATLAS** (63 techniques). You set an AS-IS and TO-BE level for each control, optionally narrow the scope per pillar to compute an effective coverage factor, and the tool derives prioritized actions, a threat-coverage delta, and a print-ready executive report. It runs entirely in the browser with no data leaving the machine. The UI is in Japanese.

---

## 概要

MAAZ（マーズ）は、**ゼロトラストの成熟度評価と、それが実際にどの脅威を止めるのかを一枚の HTML でつなぐ**アセスメントツールです。

成熟度モデルの評価は「レベルが上がった」で終わりがちですが、経営層が知りたいのは「それで何が防げるようになるのか」です。MAAZ は成熟度軸（CISA ZTMM）と脅威軸（MITRE ATT&CK / ATLAS）をクロスマッピングし、AS-IS と TO-BE の差分から投資優先度を導きます。

### 評価の骨格

| 軸 | 内容 |
|---|---|
| 成熟度 | CISA ZTMM v2 — 6 ピラー（Identity / Devices / Networks / App & Workload / Data / Cross-Cutting）× 18 コントロール × 4 レベル（Traditional / Initial / Advanced / Optimal） |
| 脅威 | 計 63 技法（MITRE ATT&CK 50 + MITRE ATLAS 12 + 独自定義 1）。フェーズ別・リスク別に整理。独自定義の 1 件は公式フレームワークに対応技法が存在しない項目で、画面上でもその旨を明示 |
| 対応関係 | ZTMM コントロール ↔ 技法のマッピング表に基づき、AS-IS / TO-BE それぞれで「緩和できる技法」を算出 |

### 画面構成（v5.0.0 — 問い駆動の4区画）

v4.0.0 までの機能別の並列画面（Dashboard / Assessment / Scope / Mapping / Threat Map / Action Plan）を、左サイドナビから**経営の問い4つ**を切り替える構成に再編しました。評価ロジック・スコア・エクスポート形式は v4.0.0 と同一で、置き場所だけを変えています。

| 区画 | 問い | 内容 |
|---|---|---|
| **① 現在地**（Now） | いま、どこまで守れているか。 | 総合成熟度スコア・脅威カバレッジ・ピラー別進捗を俯瞰する現在地ビュー（HORIZON 帯・成熟度ゲージ・レーダー凡例は AS-IS と TO-BE を併記し、6 Pillars の行は AS-IS 専用） |
| **② 到達点**（Target） | TO-BE にすると何が良くなるか。 | ピラー別の TO-BE 進捗（Pillar Progress）とボトルネックピラーの指摘。AS-IS との距離を投資判断の根拠として示す |
| **③ 着手順**（Order） | 何から着手するか。 | ギャップから導いた対策を優先度つきに整理。ロードマップ表とカンバン表示を切り替え可能。各アクションには対象レベルの**達成基準**（そのレベルで実装すべきこと）と、対応するソリューションカテゴリ（IDaaS、EDR、DLP、XDR、SOAR 等の一般名称）を併記 |
| **④ 記入**（Input） | 評価をここで入力する。 | 18 コントロールの AS-IS / TO-BE 評価、**Scope（SCF）**の AS-IS / TO-BE 別スコープ設定、業界プリセット（金融・保険 / 医療・製薬 / 製造・インフラ / IT・SaaS / 政府・公共機関 / 中小企業）、前提情報を全幅で収容（440px のドロワーには収まらないため） |

区画をまたいで参照する根拠は、次の2面に集約しています。

| 面 | 内容 |
|---|---|
| **根拠センター**（Evidence・下からのシート・5タブ） | **カバレッジ一望**（63技法 × カバレッジ・マトリクス）／**脅威マップ**（技法一覧・フィルタ・代表的な脅威アクターのプロファイル）／**マッピング**（ZTMM コントロール ↔ MITRE 技法の対応表を全行公開。どの統制からも到達しない技法や新規緩和が発生しないレベルも明示し、スコアの出どころを外部から検証できる）／**算出ロジック**（脆弱性係数の4ステップ算出）／**出典・免責**（母集合の限定・参照フレームワークの版・開示範囲） |
| **文書・出力**（Documents・全幅オーバーレイ） | **CISO Executive Report** — A4 縦（表紙 / サマリ / 現状評価 / ピラー別改善内訳 / 脅威ランドスケープ / ロードマップ / 投資提言 / 付録）を印刷・PDF 保存。ロードマップとピラー別改善内訳は内容量に応じてページ数が変動／**連携 JSON**（下記「入出力」）／**スナップショット**（評価時点の状態をブラウザ内に保存し、時系列で比較） |

画面下部の出所チップから、根拠センターの該当タブまたは④記入区画へ直接ジャンプできます。ARIA tabs・フォーカストラップ・`inert` によるキーボード操作に対応しています。

### Scope（SCF）— 実効カバレッジの考え方

ピラーごとに対象リソースを Y / P / N / NA で評価し、**Scope Coverage Factor** を算出します（NA は分母から除外）。成熟度が Optimal でも対象範囲が部分的なら攻撃面は残る、という実効カバレッジを反映する仕組みです。スコープは **AS-IS（現在の ZT 制御対象）と TO-BE（計画後の ZT 制御対象）を別々に**指定し、それぞれの SCF を対応する側の実効カバレッジに適用します。これにより「成熟度が上がった分」と「制御対象を広げた分」を分けて示せます（TO-BE でスコープを広げない場合は「AS-IS からコピー」で揃えます）。

### 入出力

評価の保存・再開と下流ツール連携を兼ねた4系統です（文書・出力オーバーレイに集約）。

- **OVERDUE 連携 (JSON)** — 評価全体（AS-IS / TO-BE、脆弱性係数、脅威カバレッジ、アクション）を書き出す標準形式。ZT 負債の投資対効果を試算する [OVERDUE](https://github.com/takainthecloud-glitch/OVERDUE) に渡せる。**JSONインポート**でそのまま読み戻して続きから再開できる
- **CSF評価連携 (JSON)** — コントロール別の AS-IS / TO-BE と総合ティアに絞った軽量形式。CSF 成熟度側のツールに渡す用途
- **CSV (脅威)** — 63 技法の緩和状況を表計算ソフト向けに出力
- **スナップショット** — 評価時点の状態をブラウザ内に保存し、時系列で比較

3 色のテーマ（paper / indigo / beige）を切替できます。選んだテーマはブラウザの `localStorage` に保存され、次回起動時に復元されます（値は端末内のみに保存され、外部には送信されません）。

### データの扱い

すべての計算はブラウザ内で完結し、入力内容が外部に送信されることはありません。スナップショット以外はページを閉じると消えるため、評価を残すには JSON エクスポートまたは PDF 出力を使用してください。

## 使い方

1. `MAAZ_ztelier_v5_0_0.html`（または `index.html`）をダウンロードする
2. ブラウザでファイルを開く

ビルド不要・サーバー不要です。GitHub Pages を有効にした場合は `index.html` がそのまま表示されます。

> **ネットワークについて**: 画面描画に React / Babel を CDN（unpkg）から、フォントを Google Fonts から読み込みます。初回表示時はインターネット接続が必要です。完全オフラインで使う場合は、これらを同梱した形に改変してご利用ください。

### 動作環境

Chrome / Edge / Firefox / Safari の最新版。JavaScript を有効にしてください。PDF 出力はブラウザの印刷機能を使用します。

## バージョン

- アプリケーション: **v5.0.0**（HTML 内の `const APP_VER` が唯一の版数の出所）
- 準拠フレームワーク: CISA ZTMM v2 / MITRE ATT&CK / MITRE ATLAS（ATLAS は 2026.07 版）
- 設計システム: Ztelier Design System v2 — Ztelier Console UI v2.0

エクスポートする JSON のスキーマ版数はアプリ版数とは別軸で管理されています。

### 変更履歴

- **v5.0.0** — UI を**問い駆動のコックピット**へ全面再構成（メジャー変更）。(a) 並列画面（Dashboard / Assessment / Scope / Mapping / Threat Map / Action Plan）を廃止し、左サイドナビから切り替える**4区画**に再編 — ①現在地（いま、どこまで守れているか）／②到達点（TO-BE にすると何が良くなるか）／③着手順（何から着手するか）／④記入（評価をここで入力する）。参照専用の情報は**根拠センター**（カバレッジ一望・脅威マップ・マッピング・算出ロジック・出典と免責の5タブ）と**文書・出力**（CISO Executive Report・連携 JSON・スナップショット）のオーバーレイに集約した。(b) ARIA tabs・フォーカストラップ・`inert` によるキーボード操作に対応。(c) アクセシビリティを是正 — 状態を意味で運んでいた `--ink-3` の使用箇所を用途別トークンへ再分類（CISO Executive Report 内の固定リテラル色を含む）、非テキスト要素の最小コントラストを確保、①現在地が AS-IS と TO-BE の両方を表示していた箇所を AS-IS 専用の表記に是正。(d) CISO Executive Report 内の旧い番号表記（旧画面の識別番号）への参照を新しい区画名へ置換。あわせてロードマップのフェーズ列表示を内部識別子から表示ラベルへ変更した（金額・様式・ページ構成は不変。実装差分は文言置換 5 箇所・色指定 7 箇所・表示ラベル化 1 箇所の計 13 行）。**評価ロジック・スコア・エクスポート形式（`maaz-v4.4.0` / `maaz-v4-csfa`）に変更はない**（v4.0.0 以前の保存 JSON はそのまま読み込めます）
- **v4.0.0** — UI を **Ztelier Console UI v2.0** シェルへ刷新。264px 固定サイドナビ＋ヒーロー帯を導入し、メインビジュアルをカバレッジ・マトリクス主役の構成に変更しました。配色トークンを正典 Ztelier Design System v2（Future Blue / Cyber Black）に統一し、新たに3色目のテーマ **Frontier Beige（beige）** を追加（paper → indigo → beige の順に切り替え）。表示のみの刷新であり、**評価ロジック・スコア・エクスポート形式（`maaz-v4.4.0` / `maaz-v4-csfa`）に変更はない**
- **v3.7.2** — paper テーマの `--topbar-bg` を UI Kit v1.0 の正準値（`rgba(247, 249, 251, .88)`）に統一（Kit 監査で v3.7.0 移行時の未追随を検出したもの。透過度 0.86 → 0.88）。見た目の差はトップバーの透け具合がごくわずかに変わるのみで、**評価ロジック・スコア・エクスポート形式（`maaz-v4.4.0` / `maaz-v4-csfa`）に変更はない**
- **v3.7.1** — indigo テーマの `--danger-ink` を UI Kit v1.0 の AA 検証値（`#E38A8A`）に統一（実測8面の最悪値 4.55:1、従来値は一部の面で AA 未達だった）。あわせて UI Kit 移行（v3.7.0）以降どこからも参照されていなかった `.zt2-*` 系 CSS（未使用コンポーネント・ユーティリティ、約295行）を削除。**評価ロジック・スコア・エクスポート形式（`maaz-v4.4.0` / `maaz-v4-csfa`）に変更はない**
- **v3.7.0** — UI を **Ztelier UI Kit v1.0** に一本化。旧 Common Kit 由来の重複定義と、一度も参照されていなかったコンポーネント定義を削除した。あわせて (a) フォントサイズの下限を画面・印刷とも **11px** に統一（従来 9px まで縮んでいた技法チップ等を是正）、(b) 本文・補助テキストのコントラストを WCAG AA まで引き上げ、(c) `:focus-visible` のフォーカスリングを全操作要素で明示、(d) CISO Executive Report の A4 横溢れ（技法ビュー）と改ページ位置を修正。装飾（影・グラデーション・色付き縦バー・装飾グリフ）は罫線と面に置き換えており、**評価ロジック・スコア・エクスポート形式に変更はない**
- **v3.6.0** — マッピング表の構造を拡張し、**同一のコントロール × 同一レベルでも重みの異なる行を分割できる**ようにした。1 つの成熟度レベルの中に「防止まで到達する技法」と「検知どまりの技法」が混在するケースを、まとめて高い重みで数えずに書き分けるため。技法ごとに最良経路（max）を採る算出方式のため行分割による二重計上は起きない。あわせて個別精度パッチ 6 件（根本統制の欠落補完、インライン検査系技法の担当コントロール移設、検知クラス相当への重み是正など）
- **v3.5.0** — **マッピング透明性チャプター（Mapping）を新設**し、ZTMM コントロール ↔ MITRE 技法の対応表を全行公開。あわせてマッピング監査の是正を反映した: (a) レベル 2 に置かれていた重み 1.0 のエントリをレベル 3 へ移設し、**11 コントロールはレベル 2 到達では新規緩和が発生しない**設計に変更（コントロール本文が要求するレベルと一致させるため）、(b) MITRE ATLAS の技法 ID を現行版（2026.07）へ再マッピング、(c) 公式フレームワークに対応技法が存在しない独自定義項目（シャドー AI 利用）を「独自定義」バッジで明示、(d) 脆弱性係数の母集合を限定する注記を顧客提示物に常時表示、(e) CISO Executive Report の脅威ランドスケープ節を 2 ページに分割。**(a) によりレベル 2 中心の TO-BE を置いていた場合、脅威カバレッジの数値が従来より下がります**
- **v3.4.0** — Scope（SCF）を **AS-IS / TO-BE の二重スコープ**に拡張。従来は 1 本のスコープを両側に適用していたため、TO-BE で ZT 制御対象を広げる計画が実効カバレッジに反映されず過小評価になっていた。AS-IS 側には AS-IS スコープの SCF を、TO-BE 側には TO-BE スコープの SCF を適用し、あわせて「スコープ拡張による改善（pt）」を成熟度向上分と分離して表示する。エクスポート JSON は追加のみの後方互換変更（従来の `scf` は AS-IS 値のまま、TO-BE 側は `scfToBe` を新設）
- **v3.3.1** — CISO Executive Report の「ピラー別改善内訳」が A4 の紙面を超えることがあった問題を修正。内容量に応じて自動改ページするようにした
- **v3.3.0** — Action Plan と CISO Executive Report の各アクションに、対象レベルの達成基準を併記。あわせて Executive Report のロードマップをアクション件数に応じて自動改ページ
- **v3.2.1** — 下流ツール連携の記述を整理し、OVERDUE を連携先として明示
- **v3.2.0** — 初回公開

## ライセンス

Apache License 2.0 — 詳細は [LICENSE](./LICENSE) を参照してください。

```
Copyright 2026 takainthecloud-glitch

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
```

CISA ZTMM、MITRE ATT&CK、MITRE ATLAS はそれぞれの権利者に帰属します。本ツールはこれらを参照した独自の整理であり、各機関による承認・保証を受けたものではありません。

## 免責

本ツールは成熟度の把握と議論を支援する情報提供を目的としています。算出されるスコア・カバレッジ・推奨施策は入力値と公開情報に基づく目安であり、実際の防御効果や特定の結果を保証するものではありません。
