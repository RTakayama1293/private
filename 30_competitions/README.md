# コンペ (Kaggle / SIGNATE)

参加コンペを「案件」として管理。1コンペ1フォルダ、実験ログとリーダーボード追跡を残す。

## 進行中コンペ
| プラットフォーム | コンペ | 締切 | 現在順位 | ベストスコア | フォルダ |
|---|---|---|---|---|---|
| SIGNATE | NEDO 積付アルゴリズム(コンテスト2) | 2026-10-19 | - | - | [signate/nedo-baggage-loading-2/](signate/nedo-baggage-loading-2/plan.md) |

## 過去コンペ
| プラットフォーム | コンペ | 最終順位 | メダル | 学び |
|---|---|---|---|---|

## 進め方
1. 新規参加時: `_template/` をコピーして `kaggle/<comp-name>/` や `signate/<comp-name>/` を作る
2. `overview.md` にタスク・評価指標・締切を書く
3. 実験するたびに `experiments.md` に1行追記（何を変えて、スコアがどう動いたか）
4. リーダーボードは `leaderboard.md` に定点観測

## データの扱い
- 生データ (`data/`) は `.gitignore` 済み（コミットしない）
- コード・notebook・実験ログはコミットする
- 大きな学習済みモデルもコミットしない（必要ならクラウド/DVC を検討）

## フォルダ構成
```
30_competitions/
  kaggle/
    <comp-name>/  ← _template をコピー
  signate/
    <comp-name>/
  _template/
```

> 「〇〇コンペのフォルダ作って」と言えばテンプレから生成します。
