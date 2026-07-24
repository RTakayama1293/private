# やるべきこと（アクションプラン）

> 前提: これは「良い予測モデル」ではなく「有効な積付を返し続けるエージェント」を作る問題。
> まず**減点(無効積付)を潰して有効なベースラインを出す** → **スコア項目を1つずつ上げる**、の順。
> 締切 2026-10-19。チーム結成期限 2026-09-04。投稿は1日5回まで。仮説→検証で回す。
> 仕様の詳細は [simulator-notes.md](simulator-notes.md)。CLI手順は [../../signate-cli.md](../../signate-cli.md)。

## 現状（2026-07-23）
- [x] SIGNATE CLI 導入・トークン取得（Anaconda）。task_key/file_key 特定済み（[../../signate-cli.md](../../signate-cli.md)）。
- [x] simulator.zip DL・展開・仕様解析済み（`data/` 配下、gitignore）。仕様は [simulator-notes.md](simulator-notes.md)。
- [x] **環境方針＝Docker に決定**（下記 環境方針）。ただし WSL2/Docker は未導入。
- [ ] ↓ 次: WSL2+Docker導入 → サンプル実行 → 有効ベースラインを1本提出する。

## 環境方針（意思決定 2026-07-23）
- **決定**: 開発・検証環境は Docker に統一する。
- **理由**: 配布の Dockerfile が評価基盤そのもの（ubuntu24.04 + py3.12 + gymnasium1.2.3/pybullet3.2.7/pillow10.3.0/torch2.7.0cpu）。バージョン一致で「ローカルで積めたのに提出で落ちる/スコアがずれる」を構造的に防ぐ。PyBulletは物理が環境依存でぶれ得るため一致が効く。冪等性・再現性も高い。
- **前提コスト**: WSL2 + Docker Desktop の導入（管理者権限・再起動が必要。ユーザー自身で実施）。
- **GUI**: X11不要。`"visualizer":{"vis":true}` でステップPNGを出力し、ボリュームマウント経由でWindowsから確認する。
- **懸念/保留**: RL学習はCPU版torchのため遅い。学習に入るなら別途GPU環境を検討（推論・評価はDockerのまま）。

### 再開手順（セッションが消えても、ここから）
> 2026-07-24: WSL2 は導入済み（v2.7.10、既定v2）。`wsl --install`のUbuntu DLは不要でキャンセル。Docker DesktopだけでOK。
1. Docker Desktop 導入（WSL2 backend既定ON。自前の `docker-desktop` ディストロを使うのでUbuntu不要）。`docker --version` が通ればOK。
2. `cd data/simulator/simulator && docker compose up -d`（評価基盤と同環境のコンテナ `gh_env` が起動、`.` が `/workspace` にマウント）
3. コンテナ内で `python -m scripts.run_test`（サンプル疎通）→ 成功したらフェーズ1へ
4. `data/` が消えていたら simulator.zip を CLI で再取得（signate-cli.md のキー使用）

## 最優先（今すぐ〜数日）— ローカルで一周させる
- [ ] 実行環境を用意（Python 3.10〜3.12）。Anaconda に専用env推奨:
  - `conda create -n nedo python=3.11 -y && conda activate nedo`
  - `pip install gymnasium==1.2.3 pybullet==3.2.7 pillow==10.3.0`
  - `pip install torch==2.7.0+cpu --extra-index-url https://download.pytorch.org/whl/cpu`
  - ※ pybullet は Windows だとビルドで詰まることがある。ダメなら Docker（`docker compose up -d`）に切替。
- [ ] サンプルをそのまま動かして疎通確認: `cd data/simulator/simulator && python -m scripts.run_test --render-mode human`
  - サンプルは固定座標で置くだけなので、2個目以降で検証NGになるはず。まず「動く／results が出る」ことを確認。
- [ ] `agents/submit/agent.py` を作り、**有効な積付を返すベースライン**を実装（下記フェーズ1）。
- [ ] ローカルでスコアが出たら zip 化して**1回だけCLI投稿**し、LB反映まで疎通（`signate submit`）。

## フェーズ1: 有効なベースライン（Week 1-2）
> 目標: NG(無効積付)を出さず、最後まで積める土台。「積めない＝fill以外0点」なのでここが全て。
> 設計方針: policy 内で「置ける場所」を自前探索して返す。サンプルの固定座標は捨てる。
- [ ] `optimize`: まず体積大きい順ソート（サンプル流用でOK）
- [ ] `policy` ベースライン（貪欲・Bottom-Left系）:
  - [ ] `container_list[c].packed_items` で現在の占有を把握（各荷物 pos/orn/寸法）
  - [ ] 内寸（length/width/height − thickness）＋**上部の斜め切り欠き cut_x/cut_y** を制約に入れる
  - [ ] 候補位置を離散走査（床から、奥Y+→手前Y-、左右X、低Z優先）× orientation 0〜5
  - [ ] 各候補で「内包する／既配置と safety_margin 0.015 以上離れる／搬入軌道(Y→X押し込み)が干渉しない」を自前簡易チェックし、最初の有効候補を返す
  - [ ] place_pos は**荷物中心の相対座標**（原点 offset_x=container_idx*spacing）・np.float32
  - [ ] item_idx は pool_list 範囲内（終盤プールが縮む点に注意）
- [ ] タイムアウト対策: policy を 8秒/1手 に収める（全探索は不可）。**高速な有効手のフォールバック**必須（超過＝ランダム強制で崩壊）
- [ ] `status` が "completed successfully" になる config を1つ作り、回帰確認用にする

## フェーズ2: スコアを上げる（Week 3-5）
> 100点＝重み付き平均。効く順に潰す。experiments.md にスコア内訳で記録。
- [ ] 空間充填率↑: Extreme Point / skyline / DBLF 系のヒューリスティクスで隙間を減らす
- [ ] 重心↓: 重い荷物を下・奥へ優先配置
- [ ] ペナルティ回避: 優先手荷物・ソフト手荷物の**上に他を載せない**制約、優先コンテナ指定の遵守
- [ ] 動的安定性: 接地面積・かみ合わせ・積み上げ高さを意識（揺らしてずれない配置）
- [ ] 2台コンテナの割り振りロジック（属性に応じてどちらに積むか）

## フェーズ3: 高度化（Week 6-10）
- [ ] オンラインの先読み: プール情報を使ったビームサーチ / ロールアウト評価
- [ ] ハイブリッド設計: オフラインの順序・計画をオンラインのガイドとして引き継ぐ
- [ ] 8秒制約内での探索深さ管理（近似・枝刈り・計算量の予算配分）
- [ ] 課題A/B/Cすべてでロバストに（棚有無・初期荷物有り・優先指定などの組合せ）
- [ ] （必要なら）強化学習エージェント化を検討。ただし評価基盤4vCPU/16GBで動く軽量さが条件

## フェーズ4: 仕上げ（最終盤・〜10/19）
- [ ] **Public過学習に注意**（最終は別シーンで評価）。汎化を優先し、極端なチューニングを避ける
- [ ] 投稿ファイル2枠の使い分け戦略（安定版＋攻め版など）
- [ ] コード整理。通過したら10/26までに提出する書類（概要・実行環境・ソース説明）の下書き
- [ ] 9/4のチーム結成期限までに「個人 or チーム」を判断

## 判断メモ / 論点
- ソロで行くか、チームを組むか（9/4期限）→ [決めたら overview か下に記録]
- RLに踏み込むか、ヒューリスティクスで押すか（工数と評価基盤の制約で判断）

## リスク
- Public/Private乖離で終了後に順位変動 → 汎化重視・シーン別に崩れないか確認
- 時間制約超過→ランダム強制で大崩れ → フォールバックの高速性が生命線
- 1日5投稿の上限 → ローカル評価を整えて投稿を無駄打ちしない
