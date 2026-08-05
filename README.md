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
| 脅威 | 63 技法（MITRE ATT&CK 50 + MITRE ATLAS 13）。フェーズ別・リスク別に整理 |
| 対応関係 | ZTMM コントロール ↔ 技法のマッピング表に基づき、AS-IS / TO-BE それぞれで「緩和できる技法」を算出 |

### 主な機能

- **業界プリセット**（金融・保険 / 医療・製薬 / 製造・インフラ / IT・SaaS / 政府・公共 / 中小企業）で TO-BE 目標値を一括適用し、後からコントロール単位で微調整
- **Dashboard** — 総合成熟度スコア、ピラー別レーダー、脅威カバレッジのドーナツ、ボトルネックピラーの指摘、業界ベンチマーク比較
- **Assessment** — 18 コントロールを AS-IS / TO-BE のデュアルスライダーで評価
- **Scope（SCF）** — ピラーごとに対象リソースを Y / P / N / NA で評価し、**Scope Coverage Factor** を算出。成熟度が Optimal でも対象範囲が部分的なら攻撃面は残る、という実効カバレッジを反映する（NA は分母から除外）。スコープは **AS-IS（現在の ZT 制御対象）と TO-BE（計画後の ZT 制御対象）を別々に**指定し、それぞれの SCF を対応する側の実効カバレッジに適用する。これにより「成熟度が上がった分」と「制御対象を広げた分」を分けて示せる（TO-BE でスコープを広げない場合は「AS-IS からコピー」で揃える）
- **Threat Map** — 63 技法それぞれについて、AS-IS で緩和済 / TO-BE で新たに緩和 / 未カバー を可視化。代表的な脅威アクターのプロファイル（使用技法つき）も収録
- **Action Plan** — Quick Win（0–3ヶ月）/ Short-term（3–6ヶ月）/ Strategic（6–12ヶ月）の三層で施策を優先度つきに整理。各アクションには対象レベルの**達成基準**（そのレベルで実装すべきこと）を併記し、各コントロールに対応するソリューションカテゴリ（IDaaS、EDR、DLP、XDR、SOAR 等の一般名称）も表示
- **CISO Executive Report** — A4 縦（表紙 / サマリ / 現状評価 / ピラー別改善内訳 / 脅威ランドスケープ / ロードマップ / 投資提言 / 付録）を印刷・PDF 保存。ロードマップとピラー別改善内訳は内容量に応じてページ数が変動します
- **入出力** — 評価の保存・再開と下流ツール連携を兼ねた 4 系統
  - **OVERDUE 連携 (JSON)** — 評価全体（AS-IS / TO-BE、脆弱性係数、脅威カバレッジ、アクション）を書き出す標準形式。ZT 負債の投資対効果を試算する [OVERDUE](https://github.com/takainthecloud-glitch/OVERDUE) に渡せる。**JSONインポート**でそのまま読み戻して続きから再開できる
  - **CSF評価連携 (JSON)** — コントロール別の AS-IS / TO-BE と総合ティアに絞った軽量形式。CSF 成熟度側のツールに渡す用途
  - **CSV (脅威)** — 63 技法の緩和状況を表計算ソフト向けに出力
  - **スナップショット** — 評価時点の状態をブラウザ内に保存し、時系列で比較
- **ライト / ダークテーマ**切替（paper / indigo）

### データの扱い

すべての計算はブラウザ内で完結し、入力内容が外部に送信されることはありません。スナップショット以外はページを閉じると消えるため、評価を残すには JSON エクスポートまたは PDF 出力を使用してください。

## 使い方

1. `MAAZ_ztelier_v3_4_0.html`（または `index.html`）をダウンロードする
2. ブラウザでファイルを開く

ビルド不要・サーバー不要です。GitHub Pages を有効にした場合は `index.html` がそのまま表示されます。

> **ネットワークについて**: 画面描画に React / Babel を CDN（unpkg）から、フォントを Google Fonts から読み込みます。初回表示時はインターネット接続が必要です。完全オフラインで使う場合は、これらを同梱した形に改変してご利用ください。

### 動作環境

Chrome / Edge / Firefox / Safari の最新版。JavaScript を有効にしてください。PDF 出力はブラウザの印刷機能を使用します。

## バージョン

- アプリケーション: **v3.4.0**（HTML 内の `const APP_VER` が唯一の版数の出所）
- 準拠フレームワーク: CISA ZTMM v2 / MITRE ATT&CK / MITRE ATLAS
- 設計システム: Ztelier Edition

エクスポートする JSON のスキーマ版数はアプリ版数とは別軸で管理されています。

### 変更履歴

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
