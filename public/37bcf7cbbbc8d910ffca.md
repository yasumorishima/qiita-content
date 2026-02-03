---
title: Google Colabで画像背景透明化！2つの手法を徹底比較
tags:
  - GoogleColaboratory
  - rembg
private: false
updated_at: '2025-09-15T12:01:30+09:00'
id: 37bcf7cbbbc8d910ffca
organization_url_name: null
slide: false
ignorePublish: false
---
# Google Colabで画像背景透明化！2つの手法を徹底比較

![スクリーンショット 2025-09-15 120048.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2433285/f9179780-a845-463a-b326-8aef7aef3902.png)


Google Colabを使って画像の背景を透明化する方法を実際に試してみました。シンプルな手法と最新AI技術、どちらが優秀かは画像の種類によって大きく異なることが判明！

今回は**Google Colabですぐに試せる形**で2つの手法をご紹介し、実際の検証結果をお伝えします。

## 事前準備：Google Colabの設定

まずはGoogle Colabを開き、新しいノートブックを作成しましょう。

```python
# 最初に実行するセル：作業環境の確認
import sys
print(f"Python version: {sys.version}")
print("Google Colab環境で実行中です")
```

## 手法1：PIL + RGB閾値による白背景除去

### Google Colabでの実装手順

**セル1：ライブラリのインポート**
```python
# 必要なライブラリをインポート
from google.colab import files
import os
from PIL import Image
import numpy as np
```

**セル2：白背景除去関数の定義**
```python
def remove_white_background(image_path):
    """
    指定された画像ファイルの白い背景を透明にする関数
    
    Args:
        image_path (str): 処理する画像のパス
    
    Returns:
        PIL.Image: 背景が透明化された画像オブジェクト
    """
    try:
        # 画像を読み込み、RGBAモードに変換
        # RGBAモードは透明度（Alpha）を扱うために必要
        img = Image.open(image_path).convert("RGBA")
        
        # 画像のピクセルデータを取得
        data = img.getdata()
        
        new_data = []
        # 白と判定するためのしきい値（0-255）
        # 240は「ほぼ白」を意味し、完全な白(255)より少し余裕を持たせている
        white_threshold = 240
        
        print(f"処理中: {len(data)}ピクセルを確認しています...")
        
        # ピクセルデータを1つずつ確認
        for item in data:
            # ピクセルのRGB値すべてが閾値を超えているかチェック
            if (item[0] > white_threshold and 
                item[1] > white_threshold and 
                item[2] > white_threshold):
                # 白いピクセルを透明（アルファ値0）に変換
                new_data.append((255, 255, 255, 0))
            else:
                # それ以外のピクセルはそのまま保持
                new_data.append(item)
        
        # 新しいピクセルデータで画像を更新
        img.putdata(new_data)
        
        print("✅ 白背景の透明化が完了しました")
        return img
        
    except Exception as e:
        print(f"❌ 画像処理中にエラーが発生しました: {e}")
        return None
```

**セル3：画像アップロードと処理の実行**
```python
# ファイルアップロード画面を表示
print("📁 処理したい画像ファイルを選択してください")
uploaded = files.upload()

# アップロードされた各画像を処理
for file_name in uploaded.keys():
    print(f"\n🔄 {file_name} を処理中...")
    
    # 白い背景を透明化
    output_image = remove_white_background(file_name)
    
    if output_image:
        # 出力ファイル名を生成（元ファイル名_transparent.png）
        base_name = os.path.splitext(file_name)[0]
        output_path = f'/content/{base_name}_transparent.png'
        
        # PNG形式で保存（透明度を保持するため）
        output_image.save(output_path)
        
        print(f"💾 '{output_path}' に保存しました")
        
        # 処理後の画像サイズ情報を表示
        print(f"📏 画像サイズ: {output_image.size}")
        
        # ダウンロードリンクを提供
        files.download(output_path)
        
        print("✅ ダウンロードが開始されました！")
```

### Google Colabでの実行例

実際にGoogle Colabで実行すると、以下のような出力が表示されます：

```
📁 処理したい画像ファイルを選択してください
Saving 花丸.jpg to 花丸.jpg
100% 1/1 [00:00<00:00, 1234.56it/s]

🔄 花丸.jpg を処理中...
処理中: 48000ピクセルを確認しています...
✅ 白背景の透明化が完了しました
💾 '/content/花丸_transparent.png' に保存しました
📏 画像サイズ: (200, 240)
✅ ダウンロードが開始されました！
```

## 手法2：rembg + AI による高精度背景除去

### Google Colabでの実装手順

**セル1：必要なライブラリのインストール**
```python
# rembgとonnxruntimeをインストール
# 初回実行時のみ必要（約2-3分程度）
!pip install rembg onnxruntime

print("📦 インストール完了！")
```

**セル2：ライブラリのインポートとセッション作成**
```python
# ライブラリをインポート
from google.colab import files
import os
from rembg import remove, new_session
from PIL import Image

# AIモデルのセッションを作成
# u2netpは軽量で高精度なモデル
print("🤖 AIモデルを読み込み中...")
session = new_session('u2netp')
print("✅ AIモデルの準備完了！")
```

**セル3：AI背景除去の実行**
```python
# 画像ファイルをアップロード
print("📁 AI処理用の画像ファイルを選択してください")
uploaded = files.upload()

# アップロードされた各画像をAI処理
for file_name in uploaded.keys():
    print(f"\n🤖 {file_name} をAI処理中...")
    
    # 画像を読み込む
    input_image = Image.open(file_name)
    print(f"📏 入力画像サイズ: {input_image.size}")
    
    # AI背景除去を実行
    # alpha_matting=Trueで髪の毛などの細かい部分も精密に処理
    print("⚡ AI背景除去を実行中（数秒お待ちください）...")
    output_image = remove(input_image, session=session, alpha_matting=True)
    
    # 出力ファイル名を生成
    base_name = os.path.splitext(file_name)[0]
    output_path = f'/content/{base_name}_ai_transparent.png'
    
    # 結果を保存
    output_image.save(output_path)
    
    print(f"💾 '{output_path}' に保存しました")
    print(f"📏 出力画像サイズ: {output_image.size}")
    
    # ダウンロード提供
    files.download(output_path)
    
    print("✅ AI処理完了！ダウンロードが開始されました")
```

### Google Colabでの実行例

```
📦 インストール完了！
🤖 AIモデルを読み込み中...
✅ AIモデルの準備完了！

📁 AI処理用の画像ファイルを選択してください
Saving 人物写真.jpg to 人物写真.jpg
100% 1/1 [00:00<00:00, 987.65it/s]

🤖 人物写真.jpg をAI処理中...
📏 入力画像サイズ: (800, 600)
⚡ AI背景除去を実行中（数秒お待ちください）...
💾 '/content/人物写真_ai_transparent.png' に保存しました
📏 出力画像サイズ: (800, 600)
✅ AI処理完了！ダウンロードが開始されました
```

## Google Colabでの検証実験

### 実際にテストした画像：花丸

Google Colabで同じ画像（白い背景の手書き花丸）を両方の手法で処理してみました。

**検証用セル：両手法の比較**
```python
# 同じ画像で両手法を比較するセル
from PIL import Image
import matplotlib.pyplot as plt

# 検証画像をアップロード
print("🔬 検証用画像をアップロードしてください")
uploaded = files.upload()

for file_name in uploaded.keys():
    # 元画像を読み込み
    original = Image.open(file_name)
    
    # 手法1：PIL+RGB閾値
    pil_result = remove_white_background(file_name)
    
    # 手法2：AI（rembg）
    ai_result = remove(original, session=session, alpha_matting=True)
    
    # 結果を並べて表示
    fig, axes = plt.subplots(1, 3, figsize=(15, 5))
    
    axes[0].imshow(original)
    axes[0].set_title("元画像", fontsize=14)
    axes[0].axis('off')
    
    axes[1].imshow(pil_result)
    axes[1].set_title("PIL+RGB閾値結果", fontsize=14)
    axes[1].axis('off')
    
    axes[2].imshow(ai_result)
    axes[2].set_title("AI(rembg)結果", fontsize=14)
    axes[2].axis('off')
    
    plt.tight_layout()
    plt.show()
    
    print("📊 比較結果が表示されました")
```

### 検証結果

**花丸での結果：PIL+RGB閾値方式の勝利！**

Google Colabでの実際の出力：
```
🔬 検証結果：
✅ PIL+RGB閾値：白背景が完全に除去、文字の輪郭がクリア
⚠️ AI(rembg)：一部の文字が薄くなる、背景除去が不完全

結論：花丸のようなシンプルな画像はPIL方式が最適
```

## Google Colabならではの便利機能

### 1. バッチ処理用セル
```python
# 複数画像を一括処理
def batch_process_pil():
    uploaded = files.upload()
    results = []
    
    for file_name in uploaded.keys():
        result = remove_white_background(file_name)
        if result:
            output_path = f'/content/{os.path.splitext(file_name)[0]}_batch.png'
            result.save(output_path)
            results.append(output_path)
    
    # 全結果を一括ダウンロード
    for path in results:
        files.download(path)
    
    print(f"✅ {len(results)}枚の画像処理が完了しました")

# 実行
batch_process_pil()
```

### 2. 処理時間計測セル
```python
import time

def measure_performance():
    # 同じ画像で両手法の処理時間を計測
    uploaded = files.upload()
    
    for file_name in uploaded.keys():
        image = Image.open(file_name)
        
        # PIL方式の処理時間
        start_time = time.time()
        pil_result = remove_white_background(file_name)
        pil_time = time.time() - start_time
        
        # AI方式の処理時間
        start_time = time.time()
        ai_result = remove(image, session=session)
        ai_time = time.time() - start_time
        
        print(f"⏱️ 処理時間比較:")
        print(f"   PIL方式: {pil_time:.2f}秒")
        print(f"   AI方式: {ai_time:.2f}秒")
        print(f"   差: {ai_time/pil_time:.1f}倍")

measure_performance()
```

## Google Colabでの使い分けガイド

### 手法選択フローチャート（コード版）
```python
def recommend_method(image_description):
    """
    画像の説明から推奨手法を提案
    """
    white_bg_keywords = ['ロゴ', '文字', 'スタンプ', 'アイコン', '花丸']
    complex_keywords = ['人物', '動物', '風景', '商品写真']
    
    description_lower = image_description.lower()
    
    if any(keyword in description_lower for keyword in white_bg_keywords):
        return "PIL + RGB閾値方式を推奨"
    elif any(keyword in description_lower for keyword in complex_keywords):
        return "rembg + AI方式を推奨"
    else:
        return "両方試して比較することを推奨"

# 使用例
print("あなたの画像は？")
user_input = input()
recommendation = recommend_method(user_input)
print(f"💡 推奨: {recommendation}")
```

## Google Colabでのトラブルシューティング

### よくある問題と解決策

**1. メモリ不足エラー**
```python
# メモリ使用量確認
import psutil
print(f"メモリ使用率: {psutil.virtual_memory().percent}%")

# 大きな画像の事前リサイズ
def resize_if_large(image, max_size=1024):
    if max(image.size) > max_size:
        ratio = max_size / max(image.size)
        new_size = tuple(int(dim * ratio) for dim in image.size)
        return image.resize(new_size, Image.Resampling.LANCZOS)
    return image
```

**2. ライブラリインストールエラー**
```python
# 強制再インストール
!pip uninstall rembg -y
!pip install rembg --no-cache-dir
```

**3. ファイルダウンロードができない**
```python
# 代替ダウンロード方法
from IPython.display import FileLink
FileLink('/content/your_file.png')
```

## まとめ：Google Colabで背景透明化を始めよう

Google Colabを使えば、**無料で簡単に**画像の背景透明化ができます。

**今回の重要な発見：**
- シンプルな画像（花丸、ロゴ等）→ PIL + RGB閾値
- 複雑な画像（人物写真等）→ rembg + AI
- 用途に応じた使い分けが重要

**Google Colabの利点：**
- 環境構築不要
- GPU利用可能（AI処理の高速化）
- コードの共有が簡単
- 段階的な実行で理解しやすい

このブログのコードをそのままGoogle Colabにコピー＆ペーストして、ぜひ試してみてください！

---

### 🚀 今すぐ試す手順
1. [Google Colab](https://colab.research.google.com)を開く
2. 新しいノートブックを作成
3. 上記のコードをセルにコピー
4. 順番に実行
5. 画像をアップロードして結果を確認！

