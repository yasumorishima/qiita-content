---
title: ICP-USD / ICP-JPY データ取得の備忘録
tags:
  - ICP
  - HiÐΞ
private: false
updated_at: '2023-03-05T16:04:05+09:00'
id: f1dd7f36ceb3f9d3a3e2
organization_url_name: null
slide: false
ignorePublish: false
---
Google Colaboratory で実施した例です<br><br>
Google Colaboratory → <https://colab.research.google.com/?hl=ja#>

***

# ICP-USD データの取得

***

## Historic_Cryptoという名前のPythonパッケージを使用

このパッケージは、暗号通貨のライブデータを取得するためのAPIを提供するもの

<pre><code>!pip install historic_crypto
from Historic_Crypto import LiveCryptoData
from Historic_Crypto import HistoricalData
new = LiveCryptoData('ICP-USD').return_data()</code></pre>

「LiveCryptoData」クラスを使用して、「ICP-USD」という暗号通貨ペアのライブデータを取得しています。<br>「return_data()」メソッドを呼び出すことで、このデータを取得する

ICP以外も取得できます。以下参照<br>
<https://pypi.org/project/Historic-Crypto/>

***

## 期間設定

<pre><code>start_date = '2020-01-01-00-00' # 開始日
end_date = '2024-01-01-00-00' # 終了日（任意）
new = HistoricalData('ICP-USD', 86400, start_date, end_date).retrieve_data()</code></pre>

「HistoricalData」クラスを使用して、「ICP-USD」という暗号通貨ペアのデータを取得しています。<br>「86400」は、1日を秒単位で表したものであり、このような時間間隔でデータを取得することを指示しています。<br>「retrieve_data()」メソッドを呼び出すことで、このデータを取得することができます。

## データ確認

<pre><code>import pandas as pd

# convert the returned data to a pandas dataframe
df = pd.DataFrame(new)

print(df)</code></pre>

![](https://pbs.twimg.com/media/FonimogaIAAoPvb?format=png\&name=small)

***

## ICP-USD グラフ化

取得したデータをグラフで見ると以下のような感じ

<pre><code>import matplotlib.pyplot as plt

plt.plot(df.index, df['close'])
plt.xlabel('Time')
plt.ylabel('USD Close Price')
plt.title('ICP-USD')
plt.xticks(rotation=45)
plt.grid(True)

plt.show()</code></pre>

![](https://pbs.twimg.com/media/FonjOicakAAqhpq?format=png\&name=small)

***

## ICP-USD データ (CSV) をGoogle Driveへ

もし、ここでもう保存したい人がいたらのために書いておきます

<pre><code># Google Driveへ保管

df.to_csv('ICP-USD.csv')

from google.colab import drive
drive.mount('/content/drive')

df.to_csv('/content/drive/My Drive/ICP-USD.csv')</code></pre>

こんなデータ

![](https://pbs.twimg.com/media/FonkEwqaIAAI-i4?format=png\&name=small)

***

# ICP-JPY データの取得

引き続き、ICP-JPYデータ取得

***

## FREDからUSD-JPYを取得

「pandas_datareader」を使って「Federal Reserve Economic Data」（FRED）と呼ばれるデータベースから「DEXUSEU」（ユーロ/米ドル為替レート）と「DEXJPUS」（米ドル/日本円為替レート）のデータを取得し、「df_JPY」という名前のデータフレームに格納。

<pre><code># データ取得に使うライブラリをimport
import pandas_datareader.data as web
import pandas as pd

# 2020-5-1から現在までのデータを取得する
df_JPY = web.DataReader(['DEXUSEU','DEXJPUS'],'fred',start='2020-05-01')

# 表示される列の名前を変更
df_JPY.columns = ['EURUSD','USDJPY']

# 時間インデックスを列に追加する
df_JPY['time'] = df_JPY.index

print(df_JPY.head(10))
print(df_JPY.tail(10))</code></pre>

![](https://pbs.twimg.com/media/Fonk3qRaIAAlouP?format=png\&name=small)

***

## ICP-JPYに変換

<pre><code># ICP-JPYに変換
df['ICP-JPY'] = df['close'] * df_JPY['USDJPY']

print(df['ICP-JPY'].head(10))
print(df['ICP-JPY'].tail(10))</code></pre>

![](https://pbs.twimg.com/media/Fonl6KbaAAEYUgT?format=png\&name=small)

価格の差！

***

## ICP-JPY グラフ

<pre><code>import matplotlib.pyplot as plt
df['ICP-JPY'].plot()

plt.xlabel('Time')
plt.ylabel('JPY Price')
plt.title('ICP-JPY')
plt.grid(True)

plt.show()</code></pre>

![](https://pbs.twimg.com/media/FonmRAtaEAAzQ7z?format=png\&name=small)

***

## データ整理

<pre><code># 列名を変更する
df.rename(columns={'low':'low(ICP-USD)', 'high':'high(ICP-USD)', 'open':'open(ICP-USD)', 'close':'close(ICP-USD)', 'volume':'volume(ICP-USD)'}, inplace=True)

# 時間インデックスを列に追加する
df['time'] = df.index

# CSVファイルとして保存する
df.to_csv('ICP_data.csv', index=False)

print(df.head(10))
print(df.tail(10))</code></pre>

![](https://pbs.twimg.com/media/Fonm3\_laAAECkTq?format=png\&name=small)

***

## ここで保存しても良いかな、という人向け

Googleドライブへ保存

<pre><code>from google.colab import drive
drive.mount('/content/drive')

df.to_csv('/content/drive/My Drive/ICP-JPY.csv', index=False)</code></pre>

![](https://pbs.twimg.com/media/FonnV1ZaUAM-iCy?format=png\&name=small)

***

# 全部載せておこうというデータ

<pre><code># EURUSD、USDJPY 追加
df.rename(columns={'time':'df_time'}, inplace=True)
df = pd.merge(df, df_JPY, left_on='df_time', right_on='time', how='left')

# 列削除
df.drop(['df_time', 'volume(ICP-USD)'], axis=1, inplace=True)

print(df.head(10))
print(df.tail(10))</code></pre>

![](https://pbs.twimg.com/media/FonnxUtaEAA_dqz?format=png\&name=small)

***

## Googleドライブへデータ保存

<pre><code>from google.colab import drive
drive.mount('/content/drive')

df.to_csv('/content/drive/My Drive/ICP-JPY-USD-EUR.csv', index=False)</code></pre>

![](https://pbs.twimg.com/media/FonoObeaAAAn1lC?format=png\&name=small)

***

備忘録、以上

今度はトランザクションデータ使って、税金計算しやすくできれば。
