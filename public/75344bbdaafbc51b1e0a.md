---
title: PDFにまとめた古い図面さん、各ページを分割、図番でファイル名とする (Tesseract OCR)
tags:
  - Python
  - OCR
  - HiÐΞ
private: false
updated_at: '2024-02-24T13:32:38+09:00'
id: 75344bbdaafbc51b1e0a
organization_url_name: null
slide: false
ignorePublish: false
---
Tesseract OCR メモの続きです。

Tesseract OCR メモ<br><https://hide.ac/articles/MAg5H2KlK>

***

古い図面データをまとめてスキャナーでPDFにしたいときなど

例えば、右下の図面番号を拾って、各ページを分割、ファイル名にすることを想定しています。

***

# Pythonを使用したPDFからのテキスト抽出とページ別PDFの保存

## 導入

この記事では、Pythonを用いてPDFファイルから特定の領域のテキストを抽出し、そのテキストを基に新しいPDFファイルの名前を生成する方法を紹介します。この技術は、文書管理システムにおける自動ファイリングや、大量のPDFファイルを扱う際の作業効率化に役立ちます。

## 必要なツール

*   **Python**: プログラミング言語
*   **Tesseract OCR**: テキスト抽出のためのオープンソースOCRエンジン
*   **PikePDF & PDF2Image**: PDFファイルの処理と画像変換に使用
*   **Pillow (PIL)**: 画像処理ライブラリ

## ステップバイステップのガイド

1.  **環境設定**: 必要なライブラリのインストールとTesseract OCRのセットアップ。
2.  **PDFから画像への変換**: `pdf2image`を使用してPDFファイルを画像に変換します。
3.  **テキスト抽出**: `Tesseract OCR`と`Pillow`を使用して、変換された画像から特定領域のテキストを抽出します。
4.  **新しいPDFの生成と保存**: 抽出したテキストをファイル名に使用し、`PikePDF`で元のPDFのページを新しいPDFとして保存します。

### コードの詳細解説

このスクリプトは、Pythonを使用してPDFファイルから特定の領域のテキストを抽出し、そのテキストをもとにして各ページを個別のPDFファイルとして保存するプロセスを自動化します。以下、コードの主要部分を順に説明します。

#### 必要なライブラリのインポート

```python
from pdf2image import convert_from_path
import pytesseract
from PIL import Image
import os
import re
import pikepdf
```

*   `pdf2image`: PDFを画像に変換するためのライブラリ。
*   `pytesseract`: Tesseract OCRをPythonから利用するためのラッパー。
*   `Pillow (PIL)`: 画像処理ライブラリ。ここでは、画像の特定の領域をクロップするために使用します。
*   `os`: ファイルパスの操作やディレクトリの作成に使用します。
*   `re`: 正規表現を使ってファイル名から不適切な文字を除去します。
*   `pikepdf`: PDFファイルの操作に使用します。

#### Tesseract OCRのパス設定

```python
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'
```

Tesseract OCRがインストールされているパスを指定します。これにより、`pytesseract`がTesseract OCRを呼び出す際に正しい場所を参照できるようになります。

#### PDFから画像への変換

```python
pdf_path = r"C:\Users\******\Desktop\OCRtest\your_pdf_file.pdf"
output_dir = r"C:\Users\******\Desktop\OCRtest\output"
images = convert_from_path(pdf_path)
```

*   `pdf_path`: 処理するPDFファイルのパス。
*   `output_dir`: 生成されるPDFファイルを保存するディレクトリのパス。
*   `images`: `convert_from_path`関数を使用して、PDFファイルを画像のリストに変換します。

#### テキストの抽出とPDFファイルの保存

area = (width \* 0.82, height \* 0.93, width, height)

右下の図面番号を拾って、ファイル名にすることを想定しています。

```python
with pikepdf.open(pdf_path) as pdf:
    for i, image in enumerate(images):
        width, height = image.size
        area = (width * 0.82, height * 0.93, width, height)
        cropped_image = image.crop(area)
        text = pytesseract.image_to_string(cropped_image, lang='eng').strip()

        if text:
            filename = re.sub(r'[\\/*?:"<>|]', "", text.splitlines()[0])[:15]
        else:
            filename = "no_text"

        unique_filename = f"{i + 1}_{filename}.pdf"
        file_path = os.path.join(output_dir, unique_filename)

        new_pdf = pikepdf.new()
        new_pdf.pages.append(pdf.pages[i])
        new_pdf.save(file_path)
        print(f"PDF saved as {file_path}")
```

*   各ページをループ処理し、画像の特定領域からテキストを抽出します。
*   抽出したテキストをファイル名に使用し、それぞれのページを新しいPDFファイルとして保存します。テキストが抽出されなかった場合は、デフォルトのファイル名を使用します。
*   `pikepdf.new()`で新しいPDFを作成し、`pdf.pages[i]`で元のPDFから特定のページを新しいPDFに追加します。そして、指定されたパスにPDFを保存します。

## 使用例と応用

古い図面を複数枚まとめてスキャナーでPDFにしたとき、それぞれのページを図番を振って保存したいときなど

今回は、右下の図面番号を拾って、ファイル名にすることを想定しています。

## まとめ

この方法を使えば、大量のPDFファイルを効率的に管理し、必要な情報に迅速にアクセスするための作業を自動化できます。

***

という感じ
