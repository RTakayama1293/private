# CLAUDE.md — このリポジトリでの振る舞い

これは個人の「Life OS」リポジトリ。人生の各領域を Markdown + Git で一元管理している。

## 構成
- `00_dashboard/` 今のフォーカスと週次振り返り
- `10_career/` キャリア（目標・スキル・意思決定ログ）
- `20_learning/` 学習ロードマップとログ
- `30_competitions/` Kaggle / SIGNATE。1コンペ1フォルダ（`_template/` をコピー）
- `40_keiba/` 競馬投資。`ledger.md` が収支の唯一の記録源。ROIは払戻÷投資×100
- `50_travel/` 旅行。1旅行1ファイル（`_template.md` をコピー）
- `60_life/` 生活のタスク・習慣

## 進め方
- 日本語で応答する。
- 記録は「追記」、判断は各領域の decision-log / analysis に残す。
- 集計を頼まれたら該当 Markdown の表から計算する（例: 競馬の月次回収率）。
- 個人情報・機密（口座、パスワード等）はコミットしない。`secrets/` と `.env*` は .gitignore 済み。
- 新しい案件（コンペ/旅行/レース）は対応するテンプレをコピーして作る。

## よくある依頼
- 「今月の競馬の回収率を出して」→ `40_keiba/ledger.md` の表を集計
- 「〇〇コンペのフォルダ作って」→ `30_competitions/_template/` から生成
- 「now を更新して」→ `00_dashboard/now.md`
