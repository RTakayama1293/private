# シミュレータ仕様メモ（simulator.zip 解析）

> PyBullet物理演算 + Gymnasium API の横空きコンテナ積載環境。配布物本体は `data/`（gitignore）。ここは要点の抜き書き。

## 実装するもの
`agent.py`（クラス名 `Agent` 固定）に3メソッド。提出はこのファイルを含むディレクトリをzip。

| メソッド | タイミング | 入力 | 出力 |
|---|---|---|---|
| `__init__(self, module_path)` | 開始前1回 | agent.pyのディレクトリパス | None |
| `get_init_states(self, init_states)` | 開始前1回 | dict: `optimize`, `lookahead_k`, `container_list` | None |
| `optimize(self, item_list)` | optimize有り時のみ・開始直後1回 | 全荷物 list[dict] | **list[int]**（全indexを過不足なく並べ替え） |
| `policy(self, observation)` | 毎ステップ | dict: `optimize`,`lookahead_k`,`depth_map`,`pool_list`,`container_list` | **action dict**（下記） |

### action フォーマット（policyの戻り値）
```python
{
  'item_idx': int,          # pool_list の何番目 (0 ~ lookahead_k-1、終盤はプールが縮むので注意)
  'container_idx': int,     # 0 ~ num_containers-1
  'place_pos': np.array([x,y,z], dtype=np.float32),  # 荷物中心の「相対座標」。原点は (offset_x,0,0)
  'orientation': int        # 0~5
}
```
- orientation: 0=[L,W,H] / 1=[L,H,W] / 2=[H,W,L] / 3=[W,L,H] / 4=Y90→Z90 / 5=X90→Z90
- 座標系: **Z-up 右手系**。X=幅(左右)、Y=奥行(手前〜奥, **開口部はY-側**)、Z=高さ(上+)。単位は m / kg / s。
- offset_x = container_idx * spacing（2台目は原点がX方向にずれる）

### 観測データの構造
- `container_list[i]`: index, length(X幅), width(Y奥行), height(Z高), cut_x, cut_y(開口の切り欠き), thickness, center, n_vec(内壁法線), points(面代表点), volume(有効体積), shelf(棚有無), is_prioritized, **packed_items**(既配置荷物)
- `item`(item_list/pool_list/packed_items共通): index, length,width,height, mass, is_prioritized, is_soft, belongs_to, pos, orn(quaternion), 各種摩擦係数。ソフト時のみ contactStiffness/Damping, linearDamping。
- `depth_map`: shape `(num_containers, 64, 64)`。開口部から見た「荷物までの距離」。小さい=手前に荷物、大きい=奥に隙間。
  - ピクセル(u,v)→相対座標: `x = pos_low[0] + (u/img_w)*(pos_high[0]-pos_low[0])`, `y = pos_low[1] + (v/img_h)*(...)`

## 評価基盤の実パラメータ（ローカルsample_configとは別物！ここに合わせる）
- タイムアウト: init 10s / **policy 8s** / **optimize 180s** / max_mem **12GB**
  - optimizeタイムアウト → 元のストリーム順のまま。policyタイムアウト → **ランダム行動が強制**。
  - 推奨: 各制限より**1秒以上速く**。
- validator: inclusion_margin -0.005 / **safety_margin 0.015**（1.5cm以内で衝突扱い）/ ceiling_margin 0.018 / displacement_threshold 0.3m / angle 45deg / **start_z 0.08**（8cm上から落とす）/ settle_wait_step 300
- depth_map 解像度 64×64
- 搬入軌道: 目標より start_z 高い位置から、まずY(奥行)方向へ、次にX方向へ押し込む。途中でNGならそのエピソード終了→その時点で評価。

## スコア（加重平均で100点）
- `fill_score` 充填率（有効容積に対する完全に収まった体積割合。内包判定は検証より緩め）
- `cog_score` 重心の低さ（mass×位置）
- `placement_score` 優先手荷物の配置（他属性の下敷き/優先コンテナ違反で減点）
- `soft_item_score` ソフト貨物の配置（上に非ソフトが載ると減点）
- `stability_score` 揺らした時の荷崩れの少なさ
- `num_placed_items` 積載できた割合
- **注意**: 一定数以上積めないと fill_score 以外は0。→ まず「積める」ことが最優先。

## ローカル実行（Python 3.10〜3.12）
```bash
cd data/simulator/simulator     # ← README等がある階層（zipが1階層ネスト）
pip install gymnasium==1.2.3 pybullet==3.2.7 pillow==10.3.0
pip install torch==2.7.0+cpu --extra-index-url https://download.pytorch.org/whl/cpu

# 自作エージェントを agents/submit/ に置いて実行
python -m scripts.run_test --module-path agents/submit/
# → results/evaluation_results.json にスコア・処理時間が出る
# GUIで見る: --render-mode human / configで "visualizer":{"vis":true} でステップ画像保存
```
Docker（評価基盤と同環境）: `cd simulator && docker compose up -d`

## 提出
- `agent.py`（class Agent 必須）＋ 任意 `requirements.txt`（Dockerfile に無い追加ライブラリ用）＋その他ファイルを1ディレクトリにまとめ zip。
- **評価基盤はインターネット接続不可**。外部通信を入れない。
- **sample_config は開発用**。本番はコンテナ数/サイズ・荷物種類/数/順番が全く別。特定条件に依存しない汎用アルゴリズムにする。

## 提出フィードバック例
`fill_score, cog_score, stability_score, placement_score, soft_item_score, num_placed_items, time_results{optimization, policy}, status`
- 成功: "The packing has been completed successfully."
- 途中失敗: "Stopped in the middle. Did not satisfy {is_included / is_valid / is_placed_safe}"
