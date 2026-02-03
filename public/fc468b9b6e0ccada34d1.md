---
title: spotify APIで日本の曲のデータ解析
tags:
  - HiÐΞ
private: false
updated_at: '2023-03-05T12:00:48+09:00'
id: fc468b9b6e0ccada34d1
organization_url_name: null
slide: false
ignorePublish: false
---
# spotify APIで日本の曲のデータ解析

## Spotify APIを使って日本の曲のトレンドを分析する

日本の音楽シーンは、様々なジャンルの音楽が混在しているため、どの曲が最も人気があるかを把握するのは困難なことです。しかし、Spotify APIを使えば、日本の曲のトレンドを分析することが可能です。Spotify APIを使えば、日本のユーザーが最も聴いている曲を把握することができます。これにより、日本の音楽シーンのトレンドを把握することが可能になります。

次に、取得した曲を分析します。Spotify APIを使うと、曲のジャンル、曲の人気度、曲のリリース日などの情報を取得することができます。これらの情報を分析することで、日本の音楽シーンのトレンドを把握することができます。

by ChatGPT

***

## グーグルコラボを使って

### spotify API 取得方法

Spotify APIを使用するには、以下の手順が必要です。

1.  Spotify Developer Dashboardにアカウントを作成し、アプリケーションを登録します。

<https://developer.spotify.com/dashboard/>

2.  アプリケーションを登録すると、クライアントIDとクライアントシークレットが発行されます。これらの情報を使用して、Spotify APIにアクセスするためのトークンを取得します。

3.  取得したトークンを使用して、Spotify APIにアクセスします。APIには、様々なエンドポイントがあり、それぞれ異なる情報を提供します。

4.  APIから取得した情報を処理し、必要に応じて表示または保存します。

***

### 日本のユーザーが最も聴いている曲

以下のPythonコードを使用して、Spotify APIを使用して日本のユーザーが最も聴いている曲を取得することができます。

まず、Spotify Developer Dashboardでアプリケーションを作成し、必要なクライアントIDとクライアントシークレットを取得する必要があります。その後、リクエストを送信する前に、Spotify APIにアクセストークンを提供する必要があります。アクセストークンを取得するためには、OAuth 2.0フローを使用して認証する必要があります。

<pre><code>import requests
import base64

# Spotify APIのエンドポイント
SPOTIFY_TOKEN_URL = 'https://accounts.spotify.com/api/token'
SPOTIFY_API_URL = 'https://api.spotify.com/v1'

# Spotify APIに必要な認証情報
CLIENT_ID = 'your_client_id'
CLIENT_SECRET = 'your_client_secret'

# Base64エンコードされたクライアントIDとクライアントシークレット
AUTH_HEADER = base64.b64encode(f'{CLIENT_ID}:{CLIENT_SECRET}'.encode('ascii')).decode('ascii')

# OAuth 2.0フローを使用してアクセストークンを取得する
def get_access_token():
    response = requests.post(
        SPOTIFY_TOKEN_URL,
        headers={
            'Authorization': f'Basic {AUTH_HEADER}',
        },
        data={
            'grant_type': 'client_credentials',
        },
    )
    if response.status_code == 200:
        return response.json()['access_token']
    else:
        raise ValueError('Failed to get access token')

# アクセストークンを取得
access_token = get_access_token()

# 日本のSpotifyユーザーが最も聴いている曲を取得する
response = requests.get(
    f'{SPOTIFY_API_URL}/users/spotifycharts/playlists/37i9dQZEVXbKXQ4mDTEBXq/tracks',
    headers={
        'Authorization': f'Bearer {access_token}',
        'Accept': 'application/json',
    },
    params={
        'market': 'JP',
        'limit': 1,
    },
)

if response.status_code == 200:
    track_name = response.json()['items'][0]['track']['name']
    artist_name = response.json()['items'][0]['track']['artists'][0]['name']
    print(f'The most listened track in Japan is "{track_name}" by {artist_name}')
else:
    print('Failed to get top track')

</code></pre>

結果、

<pre><code>The most listened track in Japan is "Subtitle" by OFFICIAL HIGE DANDISM</code></pre>

***

### Top10

Top10を見る

<pre><code>import requests
import base64

# Spotify APIのエンドポイント
SPOTIFY_TOKEN_URL = 'https://accounts.spotify.com/api/token'
SPOTIFY_API_URL = 'https://api.spotify.com/v1'

# Spotify APIに必要な認証情報
CLIENT_ID = 'your_client_id'
CLIENT_SECRET = 'your_client_secret'

# Base64エンコードされたクライアントIDとクライアントシークレット
AUTH_HEADER = base64.b64encode(f'{CLIENT_ID}:{CLIENT_SECRET}'.encode('ascii')).decode('ascii')

# OAuth 2.0フローを使用してアクセストークンを取得する
def get_access_token():
    response = requests.post(
        SPOTIFY_TOKEN_URL,
        headers={
            'Authorization': f'Basic {AUTH_HEADER}',
        },
        data={
            'grant_type': 'client_credentials',
        },
    )
    if response.status_code == 200:
        return response.json()['access_token']
    else:
        raise ValueError('Failed to get access token')

# アクセストークンを取得
access_token = get_access_token()

# 日本のSpotifyユーザーが最も聴いているトップ10曲を取得する
response = requests.get(
    f'{SPOTIFY_API_URL}/users/spotifycharts/playlists/37i9dQZEVXbKXQ4mDTEBXq/tracks',
    headers={
        'Authorization': f'Bearer {access_token}',
        'Accept': 'application/json',
    },
    params={
        'market': 'JP',
        'limit': 10,
    },
)

if response.status_code == 200:
    tracks = response.json()['items']
    for index, track in enumerate(tracks):
        track_name = track['track']['name']
        artist_name = track['track']['artists'][0]['name']
        print(f'{index+1}. "{track_name}" by {artist_name}')
else:
    print('Failed to get top tracks')
</code></pre>

結果、

<pre><code>1. "Subtitle" by OFFICIAL HIGE DANDISM
2. "怪獣の花唄" by Vaundy
3. "KICK BACK" by Kenshi Yonezu
4. "ダンスホール" by Mrs. GREEN APPLE
5. "OMG" by NewJeans
6. "第ゼロ感" by 10-FEET
7. "W / X / Y" by Tani Yuuki
8. "Ditto" by NewJeans
9. "シンデレラボーイ" by Saucy Dog
10. "ホワイトノイズ" by OFFICIAL HIGE DANDISM</code></pre>

***

### Top10　各曲の音楽特性データ

引き続き確認

<pre><code>import requests
import base64

# Spotify APIのエンドポイント
SPOTIFY_TOKEN_URL = 'https://accounts.spotify.com/api/token'
SPOTIFY_API_URL = 'https://api.spotify.com/v1'

# Spotify APIに必要な認証情報
CLIENT_ID = 'your_client_id'
CLIENT_SECRET = 'your_client_secret'

# Base64エンコードされたクライアントIDとクライアントシークレット
AUTH_HEADER = base64.b64encode(f'{CLIENT_ID}:{CLIENT_SECRET}'.encode('ascii')).decode('ascii')

# OAuth 2.0フローを使用してアクセストークンを取得する
def get_access_token():
    response = requests.post(
        SPOTIFY_TOKEN_URL,
        headers={
            'Authorization': f'Basic {AUTH_HEADER}',
        },
        data={
            'grant_type': 'client_credentials',
        },
    )
    if response.status_code == 200:
        return response.json()['access_token']
    else:
        raise ValueError('Failed to get access token')

# アクセストークンを取得
access_token = get_access_token()

# 日本のSpotifyユーザーが最も聴いているトップ10曲を取得する
response = requests.get(
    f'{SPOTIFY_API_URL}/users/spotifycharts/playlists/37i9dQZEVXbKXQ4mDTEBXq/tracks',
    headers={
        'Authorization': f'Bearer {access_token}',
        'Accept': 'application/json',
    },
    params={
        'market': 'JP',
        'limit': 10,
    },
)

if response.status_code == 200:
    tracks = response.json()['items']
    for index, track in enumerate(tracks):
        track_id = track['track']['id']
        track_name = track['track']['name']
        artist_name = track['track']['artists'][0]['name']
        # 曲の詳細情報を取得
        track_response = requests.get(
            f'{SPOTIFY_API_URL}/tracks/{track_id}',
            headers={
                'Authorization': f'Bearer {access_token}',
                'Accept': 'application/json',
            },
        )
        if track_response.status_code == 200:
            track_data = track_response.json()
            # 曲の詳細情報を表示
            print(f'{index+1}. "{track_name}" by {artist_name}')
            print(f' - Duration: {track_data["duration_ms"] / 1000} seconds')
            print(f' - Album: {track_data["album"]["name"]}')
            print(f' - Popularity: {track_data["popularity"]}')
        else:
            print(f'Failed to get track data for "{track_name}"')
else:
    print('Failed to get top tracks')
</code></pre>

結果、

<pre><code>1. "Subtitle" by OFFICIAL HIGE DANDISM
 - Duration: 305.509 seconds
 - Album: Subtitle
 - Popularity: 76
2. "怪獣の花唄" by Vaundy
 - Duration: 224.806 seconds
 - Album: strobo
 - Popularity: 74
3. "KICK BACK" by Kenshi Yonezu
 - Duration: 193.495 seconds
 - Album: KICK BACK
 - Popularity: 83
4. "ダンスホール" by Mrs. GREEN APPLE
 - Duration: 203.149 seconds
 - Album: ダンスホール
 - Popularity: 72
5. "OMG" by NewJeans
 - Duration: 212.253 seconds
 - Album: NewJeans 'OMG'
 - Popularity: 90
6. "第ゼロ感" by 10-FEET
 - Duration: 286.0 seconds
 - Album: コリンズ
 - Popularity: 68
7. "W / X / Y" by Tani Yuuki
 - Duration: 278.118 seconds
 - Album: Memories
 - Popularity: 70
8. "Ditto" by NewJeans
 - Duration: 185.506 seconds
 - Album: Ditto
 - Popularity: 89
9. "シンデレラボーイ" by Saucy Dog
 - Duration: 234.323 seconds
 - Album: レイジーサンデー
 - Popularity: 71
10. "ホワイトノイズ" by OFFICIAL HIGE DANDISM
 - Duration: 254.838 seconds
 - Album: ホワイトノイズ
 - Popularity: 71</code></pre>

***

## ChatGPTさん的な総括

日本のSpotifyユーザーが最も聴いているトップ10曲を見ると、以下のようなトレンドが見えてきます。

*   J-POPが人気: 1位、2位、3位、4位、10位は全て日本のアーティストによるJ-POP曲です。
*   NewJeansの人気が高い: 5位と8位に同じアーティストの曲がランクインしています。
*   OFFICIAL HIGE DANDISMも人気: 1位と10位に同じアーティストの曲がランクインしています。
*   Kenshi Yonezuも健在: 3位に彼の新曲がランクインしています。
*   他のジャンルも健闘: 6位はロック、7位はアニメソング、9位はレゲエ調の音楽です。

これらのトレンドから、日本のSpotifyユーザーはJ-POPや日本のアーティストを中心に聴いている傾向があることが分かります。また、一部のアーティストや楽曲が人気が高く、他のジャンルの曲も一定の支持を得ていることがわかります。

***

まあ、Top50 くらい見たら面白いかもね
