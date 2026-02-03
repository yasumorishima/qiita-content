---
title: 世界の通貨：USドルに対する相対価値の時間経過による変動
tags:
  - HiÐΞ
private: false
updated_at: '2023-06-10T06:09:19+09:00'
id: 6e1d5b72adad80e2fd2e
organization_url_name: null
slide: false
ignorePublish: false
---








Google Colaboratory（グーグル・コラボレイトリー）で実行しています

-------------------
# 通貨の為替レートデータを取得

通貨の為替レートデータを取得するために、Federal Reserve Bank of St. Louis（通称：FRED）が提供するデータベースを利用しています。FREDは、数千もの経済・金融データを公開しており、このようなデータを活用することで、経済の状態を把握したり、トレンド分析を行ったりすることが可能になります。

コード中では、pandas_datareaderライブラリのget_data_fred関数を使用して、FREDのデータベースから通貨の為替レートデータを直接ダウンロードしています。各国の通貨コード（例：'DEXJPUS'は日本、'DEXCHUS'は中国など）を指定することで、それぞれの国の通貨の為替レートデータを取得できます。



<pre><code>!pip install pandas_datareader

import pandas_datareader as pdr

start_date = '2019-01-01'
end_date = '2023-06-09'  # or other end date

# currency codes and country names dictionary
country_names = {
    'DEXJPUS': 'Japan',
    'DEXCHUS': 'China',
    'DEXCAUS': 'Canada',
    'DEXUSUK': 'UK',
    'EXMXUS': 'Mexico',
    'DEXINUS': 'India',
    'EXBZUS': 'Brazil',
    'DEXKOUS': 'South Korea',
    'DEXUSEU': 'Eurozone',
    'DEXSZUS': 'Switzerland', 
    'DEXSDUS': 'Sweden',  
    'DEXUSAL': 'Australia',  
    'DEXUSNZ': 'New Zealand',
    'DEXHKUS': 'Hong Kong', 
    'DEXSIUS': 'Singapore',  
    'DEXTHUS': 'Thailand', 
    'DEXMAUS': 'Malaysia', 
    'CCUSMA02RUM618N': 'Russia', 
    'DEXSFUS': 'South Africa', 
    'CCUSMA02TRM618N': 'Turkey', 
    'DEXVZUS': 'Venezuela'
}

# Retrieve exchange rates for each country's currency code
currency_codes = list(country_names.keys())

# Dictionary to store dataframes for each currency
dataframes = {}

for code in currency_codes:
    dataframes[code] = pdr.get_data_fred(code, start_date, end_date)

# Display the dataframe for each currency
for code, df in dataframes.items():
    print(f"{country_names[code]} ({code}):\n{df}\n")

</code></pre>

-----------------------

# 2019年1月1日を基準に各国の為替レートを正規化し、その変化をプロット

次に、各国の通貨の為替レートの時間経過による変化を視覚化します。

為替レートを特定の日付（ここでは2019年1月2日）のレートで正規化します。これにより、各国の通貨の時間経過によるパフォーマンスを一覧表示できます。

<pre><code>import numpy as np
import matplotlib.pyplot as plt

plt.figure(figsize=(14,8))

# 指定した日付の為替レートで正規化とプロット
# データフレームと国名をソート
sorted_items = sorted(dataframes.items(), key=lambda x: country_names[x[0]])

table_data = []

for code, df in sorted_items:
    initial_date = None
    try:
        specific_date_rate = df.loc['2019-01-01', code]
        initial_date = '2019-01-01'
    except KeyError:
        # 指定した日付のデータが存在しない場合、直近の有効なデータを使用
        specific_date_rate = df[code].loc[df[code].first_valid_index()]
        initial_date = df[code].first_valid_index().strftime('%Y-%m-%d')
    
    if np.isnan(specific_date_rate):
        # 指定した日付のデータがNaNの場合、直近の有効なデータを使用
        specific_date_rate = df[code].loc[df[code].first_valid_index()]
        initial_date = df[code].first_valid_index().strftime('%Y-%m-%d')
        
    df[code] = df[code] / specific_date_rate
    plt.plot(df.index, df[code], label=f"{country_names[code]} ({code})")

    # 表のデータを集める
    final_date = df.index[-1].strftime('%Y-%m-%d')  # 最終日の日付
    final_value = round(df[code].iloc[-1], 3)  # 最終日の値を小数点以下3桁に丸める
    table_data.append([country_names[code], initial_date, specific_date_rate, final_date, final_value])

# Place the legend to the right of the plot
plt.legend(loc='center left', bbox_to_anchor=(1, 0.5))

# 表のデータを最終値で降順ソート
table_data.sort(key=lambda x: x[4], reverse=True)

# 表の作成と表示
table_columns = ['Country', 'Initial Date', 'Initial Value', 'Final Date', 'Final Value']
plt.table(cellText=table_data, colLabels=table_columns, loc='top')

plt.show()
</code></pre>

![](https://pbs.twimg.com/media/FyL2tCZXoAUH-xJ?format=jpg&name=large)

ベネズエラのデータがおかしくなっているのは、

2021年10月1日にベネズエラは過去3年間で2回目のデノミネーション（通貨単位の切り下げ）を実施しました。これはハイパーインフレに対応するための行動で、通貨ボリバルからゼロを6つ取り除き、ボリバルの価値を100万分の1に切り下げました。

[https://www.afpbb.com/articles/-/3369068](https://www.afpbb.com/articles/-/3369068)

この影響かと思います。

いったん、ベネズエラ除外しましょう

---------------------------
# 2019年1月1日を基準に各国の為替レートを正規化し、その変化をプロット(ベネズエラ除外)

同じ内容なので、コード省略

![](https://pbs.twimg.com/media/FyL2HM7WIAQjF7k?format=jpg&name=large)

トルコのインフレが目立ちます

記事によると、

[https://www3.nhk.or.jp/news/special/international_news_navi/articles/feature/2022/05/29/21526.html](https://www3.nhk.or.jp/news/special/international_news_navi/articles/feature/2022/05/29/21526.html)

トルコリラの価値が急速に下落した原因、主な要素として以下の点が挙げられます。

1. 政策の問題：トルコの中央銀行が高インフレに対処するために通常は利上げを行うべきなのですが、トルコの大統領エルドアンは一貫して低金利政策を推進してきました。この政策は短期的には経済活動を刺激しますが、長期的にはインフレを助長し、通貨の価値を下げる結果となります。

2. 経済の基礎的な問題：トルコは大量の外国資本を必要としていますが、その資本を引きつけるためには安定した政策と信頼性が必要です。しかし、政策の不安定さや法の支配の欠如などが外国からの投資を阻害し、結果としてリラの価値を下げる要因となっています。

3. 地政学的リスク：トルコは地政学的に不安定な地域に位置しており、そのために経済にも影響が出ています。例えば、トルコはシリアとの長期的な紛争やクルド人問題など、内外の安全保障課題に直面しています。これらの問題は経済に不確実性をもたらし、リラの価値を下げる要因となっています。


いったん、トルコも除外しましょう

-----------------------------
# 2019年1月1日を基準に各国の為替レートを正規化し、その変化をプロット(トルコ、ベネズエラ除外)

同じ内容なので、コード省略


![](https://pbs.twimg.com/media/FyL3MA1XgAAmMaX?format=jpg&name=large)


2020年1月頃からのコロナ爆発により、各国動きが大きくなる

- ロシア<br>ルーブル　2021年3月、ウクライナ進行をしたため金融制裁を受ける。ルーブル暴落、しかし戻る

- 日本<br>円安進行

- 南アフリカ<Br>最近インフレ

-------------------------------

# Bar Chart Race

見づらいけど、試しに。適当

<pre><code># CSV形式で保存
# 2023-03-01までのデータのみを含むようにデータフレームを切り詰めます。
end_date = '2023-03-01'
normalized_df_combined = normalized_df_combined[normalized_df_combined.index <= end_date]
normalized_df_combined.to_csv('exchange_rate_data.csv')

# bar_chart_raceをインストール
!pip install bar_chart_race

import pandas as pd
import bar_chart_race as bcr

# CSVファイルを読み込む
df = pd.read_csv('exchange_rate_data.csv', index_col=0)

# 日付をDatetime型に変換
df.index = pd.to_datetime(df.index)

# データフレームの各値をパーセンテージ表示に変換（小数点以下2位まで丸める）
df_percentage = df * 100
df_rounded = df_percentage.round(2)

# バーグラフを作成する
bcr.bar_chart_race(df=df_rounded, n_bars=10)

</code></pre>

<iframe width="560" height="315" src="https://www.youtube.com/embed/lMPbBPj-CtM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

test です
