---
title: Entrepot NFT volume (ICP)
tags:
  - ICP
  - HiÐΞ
private: false
updated_at: '2023-03-05T15:59:57+09:00'
id: c0da1e86b3b6ae6afa5d
organization_url_name: null
slide: false
ignorePublish: false
---
Google Colaboratory で実施した例です<br><br>
Google Colaboratory → <https://colab.research.google.com/?hl=ja#>

***

# Entrepot NFT volume (ICP)

***

## データ取得

<pre><code>!rm -r nfts_data
!git clone https://github.com/lukasvozda/nfts_data.git
!pip install -r nfts_data/requirements.txt

from ic.agent import *
from ic.identity import *
from ic.client import *
from ic.candid import Types, encode, decode

# Inicialisation
client = Client(url = "https://ic0.app")
iden = Identity()
agent= Agent(iden, client)

# Endpoint that we want to pull data from
function_to_call = 'transactions'

# List of NFTs canisters we want to pull data from
# You can find it on Entrepot clicking on NFTs 
canisters = [
    {
        'name': 'Poked bots',
        'id': 'bzsui-sqaaa-aaaah-qce2a-cai'
    },
    {
        'name': 'OG MEDALS',
        'id': 'rw623-hyaaa-aaaah-qctcq-cai'
    },
    {
        'name': 'ICPuppies',
        'id': '5movr-diaaa-aaaak-aaftq-cai'
    },
    {
        'name': 'Dfinity Space Apes',
        'id': '3mttv-dqaaa-aaaah-qcn6q-cai'
    },
    {
        'name': 'IC Dinos',
        'id': 'yrdz3-2yaaa-aaaah-qcvpa-cai'
    },
    {
        'name': 'ICPCS',
        'id': 'j3dqa-byaaa-aaaah-qcwfa-cai'
    },
    {
        'name': 'D-City',
        'id': 'gtb2b-tiaaa-aaaah-qcxca-cai'
    },
    {
        'name': 'Meme Cake',
        'id': 'txr2a-fqaaa-aaaah-qcmkq-cai'
    },
    {
        'name': 'Pineapple Punks',
        'id': 'skjpp-haaaa-aaaae-qac7q-cai'
    },
    {
        'name': 'BTC Flower',
        'id': 'pk6rk-6aaaa-aaaae-qaazq-cai'
    },
    {
        'name': 'ETH Flower',
        'id': 'dhiaa-ryaaa-aaaae-qabva-cai'
    },
    {
        'name': 'ICP Flower',
        'id': '4ggk4-mqaaa-aaaae-qad6q-cai'
    },
    {
        'name': 'Motoko',
        'id': 'oeee4-qaaaa-aaaak-qaaeq-cai'
    },
    {
        'name': 'Motoko Mechs',
        'id': 'ugdkf-taaaa-aaaak-acoia-cai'
    },
    {
        'name': 'Cubetopia Islands',
        'id': '3vdxu-laaaa-aaaah-abqxa-cai'
    },

    {
        'name': 'ICKitties',
        'id': 'rw7qm-eiaaa-aaaak-aaiqq-cai'
    },
    {
        'name': 'Cubetopia Islands',
        'id': '3vdxu-laaaa-aaaah-abqxa-cai'
    },
    {
        'name': 'Poked Bots Mutant Army',
        'id': 'jv55j-riaaa-aaaal-abvnq-cai'
    },
    {
        'name': 'ICPunks',
        'id': 'bxdf4-baaaa-aaaah-qaruq-cai'
    },
    
    {
        'name': 'Pet bots',
        'id': 't2mog-myaaa-aaaal-aas7q-cai'
    },
    {
        'name': 'IC BUCKS',
        'id': '6wih6-siaaa-aaaah-qczva-cai'
    },
    {
        'name': 'MoonWalkers',
        'id': 'er7d4-6iaaa-aaaaj-qac2q-cai'
    },
    
    {
        'name': 'Finterest',
        'id': '4fcza-biaaa-aaaah-abi4q-cai'
    },  
    {
        'name': 'ICmojis',
        'id': 'gevsk-tqaaa-aaaah-qaoca-cai'
    },  

    {
        'name': 'ICTuTs',
        'id': 'ahl3d-xqaaa-aaaaj-qacca-cai'
    },
    {
        'name': 'Starverse',
        'id': 'nbg4r-saaaa-aaaah-qap7a-cai'
    },

    {
        'name': 'ICP.DOG',
        'id': '3bqt5-gyaaa-aaaah-qcvha-cai'
    },
    {
        'name': 'ICApes',
        'id': 'zvycl-fyaaa-aaaah-qckmq-cai'
    },
    {
        'name': 'Boxy Dude',
        'id': 's36wu-5qaaa-aaaah-qcyzq-cai'
    },
    {
        'name': 'ICPets',
        'id': 'unssi-hiaaa-aaaah-qcmya-cai'
    },
    {
        'name': 'Pet bots',
        'id': 't2mog-myaaa-aaaal-aas7q-cai'
    },
    {
        'name': 'CrowdFund NFT',
        'id': '2glp2-eqaaa-aaaak-aajoa-cai'
    },
    {
        'name': 'Astronauts',
        'id': 'sr4qi-vaaaa-aaaah-qcaaq-cai'
    },   


]

# File to write results to
f  = open('result.txt','w')
# Header row
f.write(f"collection,index,token,icp,time_updated,timestamp,seller,buyer")

for c in canisters:
    params = []

    params = encode(params)

    response = agent.query_raw(c["id"], function_to_call, params)

    print("Getting collection:", c["name"])

    trans = response[0]["value"]
    for i,t in enumerate(trans):
        price = str(t['_3364572809']/100000000)
        timestamp = t['_1291635725']
        token = t['_338395897']
        seller = t['_1782082687']
        buyer = t['_3136747827']
        f.write(f"\n{c['name']},{i},{token},{price},,{timestamp},{seller},{buyer}")

f.close()</code></pre>

***

## 取得したデータ確認

<pre><code>import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns


df = pd.read_csv("result.txt", header=0)
df['timestamp'] = pd.to_datetime(df['timestamp'])

sns.scatterplot(x='timestamp', y='icp', hue='collection', legend=True, data=df)
plt.grid(linestyle='dotted')

#レジェンド
#plt.legend(loc='upper left')

#X軸のラベルを変更
import matplotlib.dates as mdates

plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%y/%m'))
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())
plt.gca().xaxis.set_major_locator(mdates.MonthLocator(interval=3))
plt.gcf().autofmt_xdate()

#Y軸 Max1000
import matplotlib.pyplot as plt

# Place legend outside of the plot
plt.legend(loc='center left', bbox_to_anchor=(1.0, 0.5))

#plt.ylim(0, 1000)

plt.show()
df = df.sort_values(by='timestamp')</code></pre>

![](https://pbs.twimg.com/media/FoXyGS2aYAQbfBo?format=png\&name=small)

***

## ICP積算

<pre><code>df = df.sort_values(by='timestamp')

df['volume_icp'] = df.groupby(['collection'])['icp'].cumsum()
</code></pre>

***

## ICP積算データを使って、各日付のMax値だけを使う

データ多すぎるのを減らすため

<pre><code>import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df['day'] = df['timestamp'].dt.date
df_max = df.groupby(['day', 'collection'])['volume_icp'].max()

# ffillメソッドを使用して、前日のデータで空欄を埋める
df_max = df_max.ffill()

# プロットする
sns.scatterplot(x=df_max.index.get_level_values(0), y=df_max, hue=df_max.index.get_level_values(1), legend=True)

plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%y/%m'))
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())
plt.gca().xaxis.set_major_locator(mdates.MonthLocator(interval=3))
plt.gcf().autofmt_xdate()

# Place legend outside of the plot
plt.legend(loc='center left', bbox_to_anchor=(1.0, 0.5))

plt.grid(linestyle='dotted')

plt.show()</code></pre>

![](https://pbs.twimg.com/media/FoX1dHcacAETMX\_?format=png\&name=small)

***

## 保存

<pre><code># CSV形式で保存する
df_max.to_csv("max_icp.csv")</code></pre>

***

## ピポットテーブル&空欄を埋める

<pre><code># CSVファイルを読み込む
df = pd.read_csv('max_icp.csv')

# ピボットテーブルを作成する
df = df.pivot_table(index='day', 
    columns='collection',
    values='volume_icp')

# ffillメソッドを使用して、前日のデータで空欄を埋める
df = df.ffill()

print(df.head())</code></pre>

***

## Bar Chart Race作成

<pre><code>!pip install bar_chart_race

import pandas as pd
import bar_chart_race as bcr

bcr.bar_chart_race(df=df, n_bars=10)</code></pre>

<iframe width="560" height="315" src="https://www.youtube.com/embed/KpmSt2aD-AI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
