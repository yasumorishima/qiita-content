---
title: 【tesseract OCR メモ3】 PDFからテキストを抽出して自動でファイル名を付け、整理するPythonスクリプト
tags:
  - Python
  - OCR
  - HiÐΞ
private: false
updated_at: '2024-02-29T22:31:33+09:00'
id: b23abb5adcab6d6654e0
organization_url_name: null
slide: false
ignorePublish: false
---
# python tesseract OCR するメモ3

今回は、左下の文字を読んで、それをファイル名にして、各ページごとに分割する

以下PDFで実施

*   横浜市のPDF・18区おすすめスポット<br><https://www.city.yokohama.lg.jp/city-info/koho-kocho/koho/insatsubutsu/lifeguide.html>

この左下を読む

![](https://pbs.twimg.com/media/GHgcorjbMAAzajH?format=jpg\&name=small)

今回の結果

![](https://pbs.twimg.com/media/GHgc6iZboAAsMoA?format=png\&name=900x900)

おしい。

やり方は以下。

***

### PDFからテキストを抽出して自動でファイル名を付け、整理するPythonスクリプト

#### 目的と背景

日々の業務や研究でPDFファイルを扱う際、特定の情報を基にファイルを自動整理したい場面があります。特に、PDFの特定領域にあるテキスト（例えば、タイトルや日付など）を読み取り、それをファイル名に反映させたいというニーズがあります。この記事では、PDFから特定の領域のテキストを抽出し、抽出したテキストを基にファイル名を生成してPDFを保存するPythonスクリプトについて説明します。

#### 使用技術

*   **pdf2image**: PDFファイルを画像に変換するライブラリ。
*   **PIL (Python Imaging Library)**: 画像処理を行うライブラリ。
*   **pytesseract**: OCR（光学的文字認識）を利用して画像からテキストを読み取るライブラリ。
*   **os** と **datetime**: ファイル操作と日付の取り扱いに使用。

#### スクリプトの説明

今回、日本語だけど、英語にしたいときは、

```python
# 切り取った画像からテキストを読み取る、英語は、ENG
text = pytesseract.image_to_string(cropped_image, lang='ENG').strip()
```

では、日本語verで書きます。JPN

```python
from pdf2image import convert_from_path
import pytesseract
from PIL import Image
import os
import re
from datetime import datetime

# Tesseract OCRの実行ファイルへのパスを設定
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'

input_dir = r"C:\Users\_\Documents\Python\OCR\Input"
output_dir = r"C:\Users\_\Documents\Python\OCR\output"

# 今日の日付を取得し、日付のフォルダを作成
today = datetime.now().strftime("%Y%m%d")
today_output_dir = os.path.join(output_dir, today)
if not os.path.exists(today_output_dir):
    os.makedirs(today_output_dir)

dpi = 300
mm_to_pixel = dpi / 25.4

pdf_files = [file for file in os.listdir(input_dir) if file.endswith('.pdf')]

file_counter = 1

for pdf_file in pdf_files:
    pdf_path = os.path.join(input_dir, pdf_file)
    images = convert_from_path(pdf_path, dpi=dpi)

    for page_number, image in enumerate(images, start=1):
        width, height = image.size
        # 左下の領域を指定（横12cm、縦2cm）
        x = 120 * mm_to_pixel
        y = 20 * mm_to_pixel
        area = (0, height - y, x, height)  # 左下の領域を計算

        # OCR用の画像を切り取る
        cropped_image = image.crop(area)
        # 切り取った画像からテキストを読み取る
        text = pytesseract.image_to_string(cropped_image, lang='jpn').strip()

        # 読み取ったテキストを基にファイル名を生成
        if text:
            base_filename = re.sub(r'[\\/*?:"<>|]', "", text.splitlines()[0])[:15]
        else:
            base_filename = "no_text"

        filename = f"{file_counter}_{base_filename}_{today}"
        
        unique_filename = f"{filename}_1.pdf"
        counter = 1
        while os.path.exists(os.path.join(today_output_dir, unique_filename)):
            counter += 1
            unique_filename = f"{filename}_{counter}.pdf"

        file_path = os.path.join(today_output_dir, unique_filename)
        # 元の画像全体をPDFとして保存
        image.save(file_path, "PDF", resolution=dpi)
        print(f"PDF saved as {file_path}")

        file_counter += 1
```

#### コードの解説
1. **Tesseract OCRの設定**: Tesseract OCRの実行ファイルへのパスを設定します。これにより、`pytesseract`がOCR処理を実行できるようになります。
2. **入出力ディレクトリの設定**: 処理するPDFが置かれた入力ディレクトリと、処理結果を保存する出力ディレクトリを設定します。
3. **日付フォルダの作成**: 処理結果を保存する際、今日の日付のフォルダを作成し、整理された形で保存します。
4. **PDFの読み込みと画像への変換**: `convert_from_path`を用いてPDFを画像に変換します。DPIを設定して、変換される画像の解像度を調整します。
5. **テキスト抽出領域の指定と抽出**: 特定の領域（例：左下の12cm x 2cm）を指定して、そこから画像を切り取り、`pytesseract`を用いてテキストを抽出します。
6. **ファイル名の生成と保存**: 抽出したテキストを基にファイル名を生成し、元の画像をPDFとして新しいファイル名で保存します。

#### 応用例と注意点
このスクリプトは、特に請求書や契約書など、特定のフォーマットを持つPDFファイルの自動整理に役立ちます。ただし、Tesseract OCRの精度は画像のクオリティに依存するため、可能な限り高解像度の画像で処理を行うことが重要です。

#### まとめと今後の展望
この記事では、PDFから特定のテキストを抽出し、それをファイル名に反映させることでファイルの自動整理を行うPythonスクリプトを紹介しました。今後、OCRの精度向上や複数言語対応など、さらに汎用性の高いスクリプトの開発が期待されます。

------

です。
