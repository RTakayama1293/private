# いま (Now)

> このページは「今の自分が何にフォーカスしているか」の1枚。週次で見直す。
> 最終更新: 2026-07-23

## 今のフォーカス（最大3つ）
1. **SIGNATE NEDO積付コンペ** — ベースライン提出まで。締切 2026-10-19。
2. （Life OS の土台づくりは一巡。競馬の資金設定はまだ空欄）
3.

## 進行中
| 領域 | 案件 | 状態 | 次の一手 | 期限 |
|---|---|---|---|---|
| コンペ | NEDO積付アルゴリズム(SIGNATE) | 環境構築の直前 | **WSL2+Docker導入** → サンプル実行 | 2026-10-19 |
| 競馬 | 資金管理ルール設定 | 未着手 | rules.md の空欄を埋める | - |
| 学習 | - | - | - | - |

## 今週やること
- [ ] （最優先）NEDOコンペ: WSL2 + Docker Desktop を導入して環境を通す（下の再開手順）
- [ ] サンプルエージェントを Docker 内で `run_test` 実行して疎通確認
- [ ] ベースライン `agent.py`（Bottom-Left貪欲）の実装に着手

## セッション引き継ぎ（履歴が消えても、ここから再開できる）
> 新しいClaude Codeを立ち上げたら、この節と競馬/コンペの各READMEを読めば経緯が分かる。

**これまでの経緯（2026-07-20〜23）**
- この「Life OS」リポジトリを新規作成し GitHub(private) に同期済み。
- ユーザープロフィールを `PROFILE.md` に配置、絵文字は使わない方針を `CLAUDE.md` に明記。
- SIGNATE の NEDO積付コンペに参加中。CLI導入・トークン取得・simulator.zip解析まで完了。
  - 詳細: [コンペ概要](../30_competitions/signate/nedo-baggage-loading-2/overview.md) / [仕様メモ](../30_competitions/signate/nedo-baggage-loading-2/simulator-notes.md) / [やること](../30_competitions/signate/nedo-baggage-loading-2/plan.md) / [CLI](../30_competitions/signate-cli.md)
- **環境方針を Docker に決定**（評価基盤と完全一致・冪等性のため）。ただし WSL2/Docker未導入なのでこれから入れる。

**再開手順（次にやること）**
> 2026-07-24 時点: WSL2 は導入済み（v2.7.10、既定v2）。`wsl --install` のUbuntu DLは不要だったのでキャンセル済み。
1. Docker Desktop を導入（WSL2 backend既定ON）。`docker --version` が通ればOK。Ubuntuディストロは不要。
2. `cd 30_competitions/signate/nedo-baggage-loading-2/data/simulator/simulator`
3. `docker compose up -d` でビルド＆起動 → コンテナ内(`/workspace`)で `python -m scripts.run_test`
   （simulator.zip は `data/` にDL済み。gitignoreなので新セッションで消えていたら CLI で再取得: [signate-cli.md](../30_competitions/signate-cli.md) 参照）
4. サンプルが動いたら、ベースライン `agent.py` 実装へ（plan.md フェーズ1）

## 気になっていること / あとで考える
- コンペ: 個人参加 or チーム（結成期限 2026-09-04）
- RLに踏み込むか、ヒューリスティクスで押すか
