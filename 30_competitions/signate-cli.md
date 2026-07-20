# SIGNATE CLI メモ（共通リファレンス）

SIGNATE コンペのダウンロード・投稿をコマンドラインで行うための公式CLI(Beta)。

## 環境（この端末での解決済み情報）
- インストール済み: **Anaconda の Python**（`pip install signate` を Anaconda Prompt で実行）
- 実体: `C:\Users\yaman\anaconda3\Scripts\signate.exe`
- 注意: **通常のPowerShellではPATHに無い**。基本は **Anaconda Prompt** で実行する。
  （PowerShellから使うなら `conda activate` するか、フルパスで `& "C:\Users\yaman\anaconda3\Scripts\signate.exe" ...`）

## 制約
- **Google Sign-In 登録のアカウントは使用不可**。メール＋パスワード認証のみ。
- 投稿はコンペ規定の上限あり（このNEDOコンペは**1日5回**）。

## コマンド
```bash
# 1. 認証（初回1回。パスワードは対話入力＝履歴に残らない）
signate token -e ryodora211293@gmail.com
#   → ~/.signate/signate.json にトークン保存。以降パスワード不要。

# 2. キーを調べる
signate competition-list                                  # public_key, title, remaining, reward, entry_count
signate task-list --competition_key=<competition-public-key>   # task の public_key, task_name
signate file-list --task_key=<task-public-key>            # file の public_key, file_name, size （要参加同意）

# 3. データDL
signate download --task_key=<task-public-key> --file_key=<file-public-key> --path=./data/simulator.zip

# 4. 投稿
signate submit --task_key=<task-public-key> --path=./submit.zip --memo="baseline v1"
```

## このコンペ（NEDO積付）のキー（判明済み）
- competition_key: `0f73f92293664c48b365e08ec15161d9`
- task_key: `54eb698257344d638111c73d49664ddb`
- file_key (simulator.zip): `a83ba77834ce4d4db059791824b91212`

投稿の例（フルパス版）:
```powershell
& "C:\Users\yaman\anaconda3\Scripts\signate.exe" submit `
  --task_key=54eb698257344d638111c73d49664ddb --path=.\submit.zip --memo="baseline v1"
```

## 出典
- 公式サポート: https://support-competition.signate.jp/en/support/solutions/articles/157000231065-i-want-to-know-how-to-use-signate-cli-
- PyPI: https://pypi.org/project/signate/
