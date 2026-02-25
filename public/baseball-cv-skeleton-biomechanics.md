---
title: Driveline C3Dデータで投球・打撃の3D骨格検知をやってみた
tags:
  - Python
  - 野球
  - OpenCV
  - データ分析
  - バイオメカニクス
private: false
updated_at: ''
id: ''
organization_url_name: null
slide: false
ignorePublish: false
---

> **ライセンス表記**: 本記事で使用しているモーションキャプチャデータは [Driveline OpenBiomechanics Project](https://www.openbiomechanics.org/) のものです（**CC BY-NC-SA 4.0**）。
> **引用**: Wasserberger KW, Brady AC, Besky DM, Jones BR, Boddy KJ. *The OpenBiomechanics Project: The open source initiative for anonymized, elite-level athletic motion capture data.* (2022).
> ライセンス: https://creativecommons.org/licenses/by-nc-sa/4.0/
> 本記事の派生物（グラフ・GIF等）も同ライセンスに従います。

## はじめに

野球選手のフォームを3Dで可視化し、関節の動きと球速の関係を探ってみました。

使ったのは以下の3つです。

- **Driveline OpenBiomechanics Project (OBP)** — プロレベルのモーションキャプチャC3Dデータ（100投手＋98打者）
- **ezc3d** — C3Dファイルの読み書きライブラリ（MITライセンス、[GitHub](https://github.com/pyomeca/ezc3d)）
- **matplotlib** — 3D可視化・アニメーション

→ **GitHub**: https://github.com/yasumorishima/baseball-cv

### ezc3dとの関わり

ezc3dには[PR #384](https://github.com/pyomeca/ezc3d/pull/384)でバグ修正を貢献しています。自分がOSSに参加したライブラリを使って実際に分析する流れになりました。

---

## Step 1: C3Dデータから3D骨格を可視化

C3Dファイルにはモーションキャプチャで取得した各マーカーの3D座標が記録されています。

- **投球データ**: 45マーカー、360Hz、約726フレーム
- **打撃データ**: 55マーカー（体45＋バット10）、360Hz、約804フレーム

```python
import ezc3d

c3d = ezc3d.c3d("pitching_sample.c3d")
points = c3d["data"]["points"]  # shape: (4, n_markers, n_frames)
labels = c3d["parameters"]["POINT"]["LABELS"]["value"]
```

### 投球フォームの骨格アニメーション

![投球骨格アニメーション](https://raw.githubusercontent.com/yasumorishima/baseball-cv/master/data/output/skeleton_pitching_anim.gif)

45個のマーカーを結んで骨格として描画しています。

### 打撃フォームの骨格アニメーション

![打撃骨格アニメーション](https://raw.githubusercontent.com/yasumorishima/baseball-cv/master/data/output/skeleton_hitting_anim.gif)

打撃側は55マーカーで、バットの10マーカーは赤色で表示しています。

---

## Step 2: MediaPipeによる動画からの骨格検知

C3Dデータとは別に、通常の動画からも骨格検知ができます。GoogleのMediaPipe Poseを使えば、一般的な動画から33個のランドマークを検出できます。

```python
import mediapipe as mp

mp_pose = mp.solutions.pose
pose = mp_pose.Pose(
    static_image_mode=False,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5
)
```

モーションキャプチャ設備がなくても、スマホの動画から骨格抽出ができる点がMediaPipeの強みです。

---

## Step 3: 関節角度・角速度の抽出

骨格座標から以下の関節角度を時系列で算出しました。

### 投球（Pitching）

| 関節角度 | 最小値 | 最大値 |
|---|---|---|
| 肘屈曲 (Elbow Flexion) | 50.5° | 156.7° |
| 肩外転 (Shoulder Abduction) | 4.6° | 117.7° |
| 体幹回旋 (Trunk Rotation) | 0° | 58° |
| 膝屈曲 (Knee Flexion) | 99.1° | 163.8° |

### 角速度の時系列

![関節角速度の時系列](https://raw.githubusercontent.com/yasumorishima/baseball-cv/master/data/output/kinematic_sequence_pitching.png)

各関節の角速度をフレームごとにプロットしたものです。投球動作のどのフェーズで、どの関節が最も速く動くかが分かります。

---

## Step 4: 骨格特徴量 × 球速の相関分析

Driveline OBPのC3Dデータにはファイル名に球速情報が含まれています（例: `..._809.c3d` → 80.9 mph）。

16投手分のデータで骨格特徴量と球速の相関を調べました。

### 相関結果

![相関散布図](https://raw.githubusercontent.com/yasumorishima/baseball-cv/master/data/output/scatter_pitching.png)

![相関行列](https://raw.githubusercontent.com/yasumorishima/baseball-cv/master/data/output/correlation_pitching.png)

| 特徴量 | r | p値 |
|---|---|---|
| Peak Trunk Angular Velocity | 0.119 | 0.673 |
| Peak Elbow Angular Velocity | 0.094 | 0.739 |
| Peak Shoulder Abduction | 0.180 | 0.520 |
| **Trunk Rotation Range** | **0.425** | **0.114** |

16サンプルでは統計的有意水準には達していませんが、**体幹回旋のレンジが球速との最強の正の相関**（r=0.425）を示しました。

これは「体幹をどれだけ大きく回転させられるか」が球速に寄与する可能性を示唆しています。

---

## まとめ

- Driveline OBPのC3Dデータからezc3dで3D骨格を可視化できた
- 関節角度・角速度の時系列を抽出し、投球フェーズごとの特徴を観察できた
- 体幹回旋のレンジが球速と最も高い相関（r=0.425）を示した
- MediaPipeを使えばモーションキャプチャ設備なしでも骨格検知が可能

→ **GitHub**: https://github.com/yasumorishima/baseball-cv

**データ**: [Driveline OpenBiomechanics Project](https://www.openbiomechanics.org/)（CC BY-NC-SA 4.0）
**ezc3d**: [pyomeca/ezc3d](https://github.com/pyomeca/ezc3d)（MIT License）
