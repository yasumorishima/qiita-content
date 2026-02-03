---
title: Google Colab で一括 OCR — YomiToku テンプレートコード
tags:
  - OCR
  - HiÐΞ
  - YomiToku
private: false
updated_at: '2025-04-21T11:10:29+09:00'
id: ac083c7d39df0a1dddf5
organization_url_name: null
slide: false
ignorePublish: false
---
コード

````markdown
# Google Colab で一括 OCR — YomiToku テンプレートコード

> **この記事のゴール**  
> - PDF をブラウザにドラッグ＆ドロップ  
> - YomiToku が **HTML + CSV** を自動生成  
> - すべて ZIP にまとめて **ワンクリック DL**

---

## 1. Colab ノートブックを準備

1. [Google Colab](https://colab.research.google.com/) を開く  
2. メニュー **Runtime → Change runtime type** で **GPU** を選択（なくても動きます）  
3. 新しいノートブックを作成

---

## 2. コード全文（コピペ OK）

```python
# =========================
# ① ライブラリをインストール
# =========================
!pip install -q yomitoku           # 日本語特化OCR
!apt-get -yqq install poppler-utils # PDF ⇄ 画像変換

# =========================
# ② PDF をアップロード
# =========================
from google.colab import files
uploaded = files.upload()          # 画面でファイルを選ぶだけ

# =========================
# ③ YomiToku で一括 OCR
# =========================
import os, subprocess, shlex, zipfile, pathlib, uuid
from tqdm.auto import tqdm         # 進捗バー表示用

out_root = f"ocr_results_{uuid.uuid4().hex[:8]}"
os.makedirs(out_root, exist_ok=True)

for fn in tqdm(uploaded.keys(), desc="OCRing"):
    in_path = f"/content/{fn}"
    if not in_path.lower().endswith(".pdf"):
        print(f"⚠️  Skipping non-PDF: {fn}")
        continue

    safe = pathlib.Path(fn).stem
    outdir = os.path.join(out_root, safe)
    os.makedirs(outdir, exist_ok=True)

    for fmt in ("html", "csv"):    # 読む用＆加工用
        cmd = (
            f'yomitoku {shlex.quote(in_path)} '
            f'-f {fmt} -o {shlex.quote(outdir)} '
            f'-v --figure --encoding utf-8-sig --combine'
        )
        subprocess.run(cmd, shell=True, check=True)

# =========================
# ④ ZIP に固めてダウンロード
# =========================
zip_name = f"{out_root}.zip"
with zipfile.ZipFile(zip_name, "w", zipfile.ZIP_DEFLATED) as zf:
    for root, _, files_in_dir in os.walk(out_root):
        for file in files_in_dir:
            zf.write(os.path.join(root, file))

print(f"📦  OCR 結果を {zip_name} にまとめました")
files.download(zip_name)
````

***

## 3. コードのポイント

| ここが便利 | 説明 |
|------------|------|
| **ファイル名を毎回書かなくていい** | `files.upload()` で選んだ PDF を自動でループ処理 |
| **HTML + CSV を同時生成** | 読みやすい表示用と、Excel で開ける表データを一括取得 |
| **進捗バー付き** | `tqdm` がページ数の多い PDF でも残り時間を見せてくれる |
| **ZIP 一発ダウンロード** | 出力フォルダを丸ごと圧縮 → ワンクリックで PC に保存 |
| **毎回ユニークな出力フォルダ** | `uuid` で衝突防止。過去の結果を間違って上書きしない |

***

## 4. よくある質問

<details>
<summary>手書き文字も読めますか？</summary>
YomiToku は活字向けに最適化されていますが、整った手書きなら一定の精度で認識できます。崩れた筆跡やクセ字は誤読が出やすいので、まずは試して確認しましょう。
</details>

<details>
<summary>料金は？ 商用利用は？</summary>
個人・非商用なら無料。ビジネス利用は開発元へライセンス申請が必要です。
</details>

<details>
<summary>処理が遅い／落ちる</summary>
- まず Colab を **GPU ランタイム** に  
- 高解像度 PDF はページ数を分割してアップロード  
- 可視化不要なら `-v` を外すとメモリ節約
</details>

***

## 5. 使い道アイデア

*   ◎ 雑誌や小説の **縦書き OCR**
*   ◎ バス・電車の **時刻表 → CSV**
*   ◎ **野球のスコアブック** をデータベース化
*   ◎ 会議資料を **HTML 化して全文検索**

***

> **まとめ**\
> YomiToku は「日本語にめっぽう強い」OCR。\
> Colab テンプレを使えば、**アップロード → 待つ → ダウンロード** の 3 ステップで完了です。\
> ぜひ手元の PDF で試して、日本語 OCR の精度を体感してみてください！

```

---

