---
title: Baseball SavantのBat Trackingデータを直接取得してKaggleに公開した話
tags:
  - Python
  - データ分析
  - Kaggle
  - baseball
  - mlb
private: false
updated_at: '2026-02-08T13:52:45+09:00'
id: 26881cbcacbafc4656a7
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

MLBは2024年シーズンから「Bat Tracking」という新機能を導入しました。高速カメラでバッターのスイングを計測し、バットスピードやスイングの軌道長などの新指標を提供しています。

この記事では、pybaseballに未実装の機能をBaseball Savantから直接取得してKaggleデータセットとして公開した経験を記録します。

---

## 📊 作成したデータセット

https://www.kaggle.com/datasets/yasunorim/mlb-bat-tracking-2024-2025

**MLB Bat Tracking Leaderboard 2024-2025**

- **打者数**: 452人（2024年: 226人、2025年: 226人）
- **カラム数**: 19カラム
- **ファイルサイズ**: 107KB
- **期間**: 2024年〜2025年（最小スイング数: 50）

---

## 🔍 Bat Trackingとは

2024年シーズンからMLBが導入した新しいStatcast機能で、以下の指標を計測します：

- **avg_bat_speed**: バットスピード平均（mph）
- **swing_length**: スイング軌道の長さ（feet）
- **squared_up_per_bat_contact**: スイートスポットでの打球率
- **hard_swing_rate**: 高速スイング率
- **blast_per_bat_contact**: 理想的な打球角度＋速度での打球率
- **swords**: ゾーン外での空振り数

---

## 🚨 pybaseballの限界

通常、MLB StatcastデータはPythonライブラリ `pybaseball` で取得できます。しかし、Bat Trackingは2024年の新機能であり、**pybaseball 2.2.7時点では未実装**です。

```python
from pybaseball import statcast_batter_bat_tracking

# ❌ ImportError: cannot import name 'statcast_batter_bat_tracking'
```

そのため、**Baseball Savantから直接CSVをダウンロードする**方法を使いました。

---

## 🛠️ Baseball Savantから直接取得する方法

### Baseball SavantのCSVエンドポイント

Baseball Savantのリーダーボードページには、CSVダウンロード用のクエリパラメータがあります。

**URL構造**:
```
https://baseballsavant.mlb.com/leaderboard/{type}?year={year}&csv=true
```

Bat Trackingの場合:
```
https://baseballsavant.mlb.com/leaderboard/bat-tracking?year=2024&csv=true
```

### 方法1: curlでダウンロード

最も簡単な方法は `curl` を使う方法です。

```bash
# 2024年データ
curl -s "https://baseballsavant.mlb.com/leaderboard/bat-tracking?year=2024&csv=true" \
  -o mlb_bat_tracking_2024.csv

# 2025年データ
curl -s "https://baseballsavant.mlb.com/leaderboard/bat-tracking?year=2025&csv=true" \
  -o mlb_bat_tracking_2025.csv
```

### 方法2: Pythonでダウンロード

Python環境では `requests` ライブラリと `StringIO` を使います。

:::note alert
`pandas.read_csv(url)` で直接読み込むと **403 Forbidden** エラーになります。`requests` を使ってUser-Agentを設定する必要があります。
:::

```python
import pandas as pd
import requests
from io import StringIO

# 2024年データ
url_2024 = "https://baseballsavant.mlb.com/leaderboard/bat-tracking?year=2024&csv=true"
response_2024 = requests.get(url_2024)
df_2024 = pd.read_csv(StringIO(response_2024.text))
df_2024['season'] = 2024

# 2025年データ
url_2025 = "https://baseballsavant.mlb.com/leaderboard/bat-tracking?year=2025&csv=true"
response_2025 = requests.get(url_2025)
df_2025 = pd.read_csv(StringIO(response_2025.text))
df_2025['season'] = 2025

# 結合
df_combined = pd.concat([df_2024, df_2025], ignore_index=True)
df_combined.to_csv('mlb_bat_tracking_2024_2025.csv', index=False)

print(f"Total batters: {len(df_combined)}")
print(f"Columns: {len(df_combined.columns)}")
```

---

## 📦 Kaggleデータセットの作成

### 1. dataset-metadata.json作成

```json
{
  "title": "MLB Bat Tracking Leaderboard 2024-2025",
  "id": "your-username/mlb-bat-tracking-2024-2025",
  "licenses": [{"name": "CC0-1.0"}],
  "keywords": ["baseball", "sports"]
}
```

### 2. Kaggle CLIでアップロード

```bash
kaggle datasets create -p mlb-bat-tracking/
```

---

## 📁 データの中身

### 主要カラム

| カラム | 説明 |
|---|---|
| `id` | Baseball Savant player ID |
| `name` | 選手名 |
| `swings_competitive` | 競争的スイング数 |
| `avg_bat_speed` | 平均バットスピード（mph） |
| `swing_length` | スイング軌道長（feet） |
| `hard_swing_rate` | 高速スイング率 |
| `squared_up_per_bat_contact` | スイートスポット打球率 |
| `blast_per_bat_contact` | 理想打球率 |
| `swords` | ゾーン外空振り数 |
| `whiff_per_swing` | 空振り率 |
| `batted_ball_event_per_swing` | 打球化率 |
| `season` | シーズン年（2024 or 2025） |

---

## 🔧 Kaggleノートブックでの使い方

### 1. kernel-metadata.jsonでデータセットを指定

```json
{
  "id": "your-username/your-notebook-slug",
  "title": "Your Notebook Title",
  "code_file": "your-notebook.ipynb",
  "dataset_sources": ["yasunorim/mlb-bat-tracking-2024-2025"]
}
```

### 2. ノートブック内でデータを読み込み

```python
import pandas as pd

df = pd.read_csv('/kaggle/input/mlb-bat-tracking-2024-2025/mlb_bat_tracking_2024_2025.csv')

print(f'Total records: {len(df)}')
print(f'Unique batters: {df["id"].nunique()}')
```

---

## 🔗 このデータセットを使った分析ノートブック

日本ゆかりの3選手（大谷翔平、鈴木誠也、ラース・ヌートバー）のBat Tracking分析ノートブックを公開しています。

https://www.kaggle.com/code/yasunorim/bat-tracking-japanese-mlb-batters-2024-2025

---

## 💡 他のBaseball Savantリーダーボード

同じ方法で他のリーダーボードも取得できます。

```bash
# Expected Stats (xBA, xwOBA等)
curl -s "https://baseballsavant.mlb.com/leaderboard/expected_statistics?year=2024&csv=true"

# Outs Above Average（守備指標）
curl -s "https://baseballsavant.mlb.com/leaderboard/outs_above_average?year=2024&csv=true"

# Pitch Arsenal（投手球種データ）
curl -s "https://baseballsavant.mlb.com/leaderboard/pitch-arsenal-stats?year=2024&csv=true"
```

---

## 📝 学んだこと

### pybaseball未実装機能への対処法

- 最新のMLB統計機能はpybaseballに反映されるまで時間がかかる
- Baseball SavantはCSVエンドポイントを提供しているため直接取得できる
- `requests` + `StringIO` でUser-Agent問題を回避できる

### Kaggleデータセットの価値

- 既存データセットにない新しい指標は注目される（Bat Trackingは2024年新機能）
- 小さなデータセット（107KB）でも、話題性があれば価値がある
- データセット＋分析ノートブックのセットで公開すると相乗効果

---

## 🔗 リンク

- **データセット**: https://www.kaggle.com/datasets/yasunorim/mlb-bat-tracking-2024-2025
- **分析ノートブック**: https://www.kaggle.com/code/yasunorim/bat-tracking-japanese-mlb-batters-2024-2025
- **GitHubリポジトリ**: https://github.com/yasumorishima/kaggle-datasets
- **Baseball Savant Bat Tracking**: https://baseballsavant.mlb.com/leaderboard/bat-tracking
