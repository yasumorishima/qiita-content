---
title: WBC準々決勝 日本vsベネズエラをMLB Statcastデータで分析するStreamlitアプリを作った
tags:
  - Python
  - データ分析
  - baseball
  - Streamlit
  - statcast
private: false
updated_at: '2026-03-14T16:48:39+09:00'
id: bd48585300431c2d5939
organization_url_name: null
slide: false
ignorePublish: false
---

## 作ったもの

WBC 2026 準々決勝・日本 vs ベネズエラ戦に特化したスカウティングダッシュボードです。

**アプリ**: https://wbc-qf-jpn-ven.streamlit.app/
**GitHub**: https://github.com/yasumorishima/wbc-scouting

![予想スタメンテーブル](https://raw.githubusercontent.com/yasumorishima/zenn-content/master/images/wbc-qf-lineup-table.png)

1次ラウンドでは各チーム単位の打者・投手ダッシュボード（全20チーム30アプリ）を作りましたが、準々決勝は対戦カードが確定しています。「この打者にはどの球種が有効か」「この投手のどのゾーンが被打率が高いか」を1アプリで見られるようにしました。

## 5タブ構成

### 🎯 タブ1: 対戦プレビュー

ベネズエラの予想スタメン9人のテーブル、マチャド（NPB所属）の注意書き、控え・代打候補テーブルが並びます。

各打者をexpanderで展開すると、1人あたり以下のデータが全て表示されます。

- 主要指標6つ（AVG/OBP/SLG/OPS/K%/BB%）とMLB平均との比較
- レーダーチャート（5軸、MLB平均ライン付き）
- ゾーンヒートマップ（3×3・5×5）— 打率・xwOBAのゾーン別分布、vs左投手/vs右投手で分割
- スプレーチャート — vs左投手/vs右投手で分割

![スプレーチャート（左右投手別）](https://raw.githubusercontent.com/yasumorishima/zenn-content/master/images/wbc-qf-spray-chart.png)

- 左右投手別成績（OPS/打率/K%/BB%を並列表示）
- 配球プラン — 全体 + vs左投手 + vs右投手の3パターン
- 守備シフト推奨 — 打球方向データから自動生成、左右投手別に表示
- 球種別パフォーマンステーブル（打率・長打率・空振率・チェイス率）
- カウント別パフォーマンス（有利=緑・不利=赤・イーブン=琥珀で色分け）

タブの最後に先発投手（Ranger Suárez・左腕）の分析セクションがあります。

### 📋 タブ2: ゲームプラン

試合のフェーズ別にStatcastデータをまとめたタブです。

![チーム弱点分析](https://raw.githubusercontent.com/yasumorishima/zenn-content/master/images/wbc-qf-weakness-analysis.png)

- **チーム弱点分析** — K% ≥ 22.4%（MLB平均）の打者、BB% < 8.3%の打者、左右別OPSに80pts以上の差がある打者を自動抽出
- **1〜3回 vs スアレス（先発）** — 先発のK%/BB%/Whiff%/平均球速・球種配分、打順グループ別の打者成績
- **4〜5回（2巡目 or 中継ぎ移行）** — 中継ぎ投手のK%/BB%/Whiff%/球速
- **6回以降（勝ちパターン）** — 各リリーフのK%/Whiff%/Chase%/球速、左右別マッチアップデータ
- **代打候補** — 控え選手のAVG/OPS/K%

全テキストはMLB Statcastの数値のみ。コーチング口調は入れず、データだけを提示する方針です。

### ⚔️ タブ3: 打線スカウティング

チーム打線のレーダーチャート（AVG/OBP/SLG/K%/BB%の5軸、MLB平均ライン付き）と、全打者の個別分析。

### 🎱 タブ4: 先発投手分析

予想先発Ranger Suárezの球種テーブル（球速 mph/km/h 両表記・変化量・空振率・決め球成功率）、球種使用率の円グラフ、変化量チャート、配球ヒートマップ、左右打者別成績、カウント別球種選択（ドーナツチャート）。

![カウント別球種選択](https://raw.githubusercontent.com/yasumorishima/zenn-content/master/images/wbc-qf-pitch-selection.png)

### 🔥 タブ5: ブルペン分析

ブルペンの概要と個別投手の詳細分析。

## 技術的なポイント

### 分析テキストの動的生成

6つの関数がStatcastの生データから分析テキストを自動生成します。

| 関数 | 用途 |
|------|------|
| `generate_player_summary()` | 打者のスカウティング要約 |
| `generate_pitcher_summary()` | 投手のスカウティング要約 |
| `generate_pitching_plan()` | 配球プラン（球種・ゾーン・カウント・左右別） |
| `generate_hitting_plan()` | 打撃アプローチ（被打率の高い球種・ゾーン等） |
| `generate_defensive_positioning()` | 守備シフト推奨 |
| `generate_sp_pitch_analysis()` | 先発投手の球種分析 |

各関数が球種別の空振率、ゾーン別打率、カウント別OPS等を計算し、閾値を超えた項目だけをテキスト化します。

```python
# 例: 被打率が最も高い球種を特定
hittable = sorted(
    [p for p in pt_stats if p["ba"] is not None],
    key=lambda x: x["ba"], reverse=True
)
if hittable and hittable[0]["ba"] >= 0.250:
    h = hittable[0]
    lines.append(
        f"- **被打率が最も高い球種:** {h['label']}"
        f"（被打率 .{int(h['ba']*1000):03d}）"
    )
```

### 全指標にMLB平均値を並記

SLG .476 だけでは高いかどうかわかりません。全ての数値にMLB平均を添えています。

```
K% 28.3% (MLB avg 22.4%)
BB% 6.1% (MLB avg 8.3%)
```

### ゾーン名の打席側自動反転

左打者と右打者では「インコース」「アウトコース」が逆になります。`_zone_names_for_bats()`で打席に応じてゾーン名を自動反転しています。

### 用語解説

全指標の横に **?** アイコンがあり、ホバーで意味とMLB平均値を表示。カウント表記にも読み方ガイドと色凡例付きです。

## データソース

- [Baseball Savant](https://baseballsavant.mlb.com/) Statcast データ（2024-2025 MLBレギュラーシーズン）
- [pybaseball](https://github.com/jldbc/pybaseball) 経由で取得

## 関連記事

- [WBC 2026 スカウティングダッシュボードとStatcastデータセットを公開した話](https://qiita.com/shogaku/items/ce89231a145ddb37c712)
