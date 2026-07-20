# Life OS

人生まるごと（キャリア・学習・コンペ・競馬投資・旅行・生活）を一元管理するリポジトリ。

すべて Markdown + Git で管理。数値も記録もバージョン管理され、オフラインでも grep 検索できる。

> 持ち主のプロフィール（人物像・思考特性・文体の好み）は [PROFILE.md](PROFILE.md) を参照。

## ダッシュボード
- **[いま](00_dashboard/now.md)** — 今フォーカスしていること
- [週次振り返り](00_dashboard/weekly-review.md)

## 領域
| 領域 | 内容 | 入口 |
|---|---|---|
| キャリア | 中長期目標・スキル・意思決定ログ | [10_career/](10_career/README.md) |
| 学習 | ロードマップ・学習ログ | [20_learning/](20_learning/README.md) |
| コンペ | Kaggle / SIGNATE 管理 | [30_competitions/](30_competitions/README.md) |
| 競馬投資 | 収支台帳・資金管理・回収率 | [40_keiba/](40_keiba/README.md) |
| 旅行 | 旅行計画 | [50_travel/](50_travel/README.md) |
| 生活 | 習慣・タスク | [60_life/](60_life/README.md) |

## 使い方の思想
- **1ファイル1テーマ**。増えたら分割。
- **記録は追記、判断は decision-log に**。あとで「なぜそう決めたか」を辿れるように。
- **数値管理は表 (Markdown table) で**。集計が必要になったらスクリプトを足す。
- 迷ったら Claude Code に「〇〇を更新して」「今月の回収率を出して」と頼む。

## 命名ルール
- フォルダは `NN_name/`（並び順を固定）
- 日付は `YYYY-MM-DD`
- コンペ/レース/旅行など「案件」は1件1ファイル
