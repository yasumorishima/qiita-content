---
title: Raspberry Pi 5でホルムズ海峡の船舶をリアルタイム追跡するシステムを構築した
tags:
  - Python
  - RaspberryPi
  - Docker
  - ais
  - FastAPI
private: false
updated_at: '2026-03-14T16:48:39+09:00'
id: 259af7d5ca5e12aa5240
organization_url_name: null
slide: false
ignorePublish: false
---

## 概要

aisstream.ioのWebSocket APIでホルムズ海峡のAISデータをリアルタイム収集し、Raspberry Pi 5上のDocker環境でSQLite保存・FastAPI配信・Leaflet.js地図表示・matplotlibスナップショット生成まで行うシステムを構築しました。

この記事は構築時の振り返りメモです。

## AIS（自動船舶識別装置）の基礎

AIS（Automatic Identification System）は、船舶が自動的に位置・速度・針路・船名・船種などの情報をVHF帯で送信する国際安全システムです。

今回使うメッセージは2種類です。

| メッセージ | 内容 | 送信間隔 |
|---|---|---|
| PositionReport | 緯度、経度、速度、針路、船首方位 | 数秒〜数分 |
| ShipStaticData | 船名、船種コード、目的地、サイズ、喫水 | 数分 |

## システム構成

```
aisstream.io (WebSocket)
       |
       v
 Raspberry Pi 5 (Docker)
 +-----------------------+    +--------------------+
 | ais-collector          |    | snapshot-cron       |
 |  collector.py (WS受信) |    |  snapshot.py        |
 |  api.py (FastAPI)      |    |  auto_push.sh       |
 |  main.py (統合起動)    |    |  (6時間ごとcron)    |
 +-----------------------+    +--------------------+
       |                              |
   SQLite (ais.db)               GitHub README
   Leaflet.js (port 8002)       (スナップショット画像)
```

このRaspberry Pi 5では既にNPB予測API（port 8000）、MLB勝利確率API（port 8001）、Streamlit keepaliveの3コンテナが稼働しています。ship trackerは追加の2コンテナ（port 8002）です。

## データソース: aisstream.io

[aisstream.io](https://aisstream.io/)はリアルタイムAISデータをWebSocketで配信するサービスです。GitHubアカウントで登録してAPIキーを取得できます。

バウンディングボックスでフィルタリングできるため、ホルムズ海峡周辺だけのデータを受信します。

## 実装

### collector.py: WebSocketクライアント

aisstream.ioに接続し、ホルムズ海峡のバウンディングボックス内のAISメッセージを受信してSQLiteに保存します。

```python
BBOX = [[23.5, 54.0], [27.5, 58.5]]

subscribe_msg = {
    "APIKey": API_KEY,
    "BoundingBoxes": [BBOX],
    "FilterMessageTypes": ["PositionReport", "ShipStaticData"],
}
```

ShipStaticDataはインメモリのdictにキャッシュし、PositionReport受信時にMMSIをキーにして紐づけます。

```python
static_cache: dict[int, dict] = {}

if msg_type == "ShipStaticData":
    meta = msg.get("Message", {}).get("ShipStaticData", {})
    mmsi = msg.get("MetaData", {}).get("MMSI")
    if mmsi:
        static_cache[mmsi] = {
            "ship_name": meta.get("Name", "").strip(),
            "ship_type": meta.get("Type"),
            "destination": meta.get("Destination", "").strip(),
            "draught": meta.get("MaximumStaticDraught"),
            "length": meta.get("Dimension", {}).get("A", 0)
                    + meta.get("Dimension", {}).get("B", 0),
            "width": meta.get("Dimension", {}).get("C", 0)
                    + meta.get("Dimension", {}).get("D", 0),
        }
```

AIS仕様では船のサイズはA/B/C/Dの4値で表現されます。A+Bが全長、C+Dが全幅です。

接続切断時は自動再接続します。

```python
except (websockets.exceptions.ConnectionClosed, OSError) as e:
    logger.warning("Connection lost: %s -- reconnecting in 10s", e)
    await asyncio.sleep(10)
```

### api.py: FastAPIエンドポイント

3つのエンドポイントを提供します。

| エンドポイント | 用途 |
|---|---|
| `GET /api/latest` | 直近30分の各船舶の最新位置 |
| `GET /api/tracks/{mmsi}?hours=6` | 特定船舶の航跡履歴 |
| `GET /api/stats` | 船種別の統計情報 |

AIS船種コードを人間が読めるラベルに変換しています。

```python
SHIP_TYPE_LABELS = {
    range(70, 80): "Cargo",
    range(80, 90): "Tanker",
    range(60, 70): "Passenger",
    range(30, 36): "Fishing/Towing/Dredging",
    range(36, 40): "Military/Sailing/Pleasure",
    range(40, 50): "HSC",
}
```

### main.py: 統合起動

asyncio.gatherでコレクターとFastAPIサーバーを1プロセスで並行実行します。

```python
async def main():
    await asyncio.gather(
        collect(),
        run_server(),
    )

if __name__ == "__main__":
    asyncio.run(main())
```

### map.html: Leaflet.jsリアルタイム地図

CARTO darkタイルを使ったダークテーマの地図です。

```javascript
L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
    maxZoom: 19,
}).addTo(map);
```

特徴:
- 船種ごとの色分け（Tanker=オレンジ, Cargo=青, Passenger=緑, Fishing=紫, Military=赤）
- クリックでポップアップ（船名、速度、針路、国旗、目的地、サイズ）
- 「Show Track (6h)」ボタンで航跡表示
- 30秒ごとに自動更新

![地図のスクリーンショット](https://raw.githubusercontent.com/yasumorishima/hormuz-ship-tracker/master/docs/screenshot.png)

### snapshot.py: matplotlibスナップショット

6時間ごとにSQLiteからデータを読み出し、ダークテーマの静的マップ画像を生成します。

```python
fig, ax = plt.subplots(figsize=(14, 9), facecolor="#0a0a1a")
ax.set_facecolor("#0d1b2a")

# 海岸線の近似ポリゴンで地理的コンテキストを描画
for segment in COASTLINE_SEGMENTS:
    lats, lons = zip(*segment)
    ax.plot(lons, lats, color="#2a3a4a", linewidth=1.2, zorder=2)
    ax.fill(lons, lats, color="#111822", alpha=0.6, zorder=1)
```

shapefileライブラリに依存せず、手書きの近似座標で海岸線を描画しています。

### auto_push.sh: SHA256差分検出 + git push

```bash
NEW_HASH=$(sha256sum "$SNAPSHOT" | cut -d' ' -f1)
OLD_HASH=$(sha256sum "$DEST_IMG" | cut -d' ' -f1)

if [ "$NEW_HASH" = "$OLD_HASH" ]; then
    echo "No change in snapshot -- skipping push"
    exit 0
fi

git commit -m "snapshot: ${VESSEL_COUNT} vessels at ${TIMESTAMP}"
git push origin HEAD
```

画像が変わっていない場合（夜間など）は無駄なcommitを生成しません。

## Docker構成

```yaml
services:
  ais-collector:
    build: .
    container_name: hormuz-tracker
    restart: unless-stopped
    ports:
      - "8002:8002"
    volumes:
      - ./data:/app/data
      - .:/repo

  snapshot-cron:
    build: .
    container_name: hormuz-snapshot
    restart: unless-stopped
    entrypoint: /bin/bash
    command:
      - -c
      - |
        apt-get update -qq && apt-get install -y -qq git cron >/dev/null 2>&1
        echo "0 0,6,12,18 * * * /bin/bash /app/src/auto_push.sh" | crontab -
        /bin/bash /app/src/auto_push.sh || true
        cron -f
    depends_on:
      - ais-collector
```

SQLiteファイルはボリュームマウント（`./data:/app/data`）で両コンテナ間で共有しています。

## 起動手順

```bash
# .envファイルの作成
echo "AISSTREAM_API_KEY=your-api-key" > .env
echo "GITHUB_TOKEN=your-github-token" >> .env
echo "GITHUB_REPO=your-username/hormuz-ship-tracker" >> .env

# 起動
docker compose up -d

# ブラウザで http://<raspberry-pi-ip>:8002 にアクセス
```

## 設計判断

| 判断 | 理由 |
|---|---|
| SQLite | 単一ファイルで管理が簡単。RPiの制約上PostgreSQLは不要 |
| インメモリキャッシュ | StaticDataとPositionReportが別メッセージで到着するため |
| SHA256比較 | 船が動いていない時間帯の無駄なgit pushを防止 |
| 1プロセスでcollector+API | asyncio.gatherで十分。コンテナを分けるほどの規模ではない |
| 近似ポリゴン海岸線 | shapefileライブラリ依存を避けた |

## まとめ

aisstream.ioのWebSocket APIとRaspberry Pi 5を組み合わせることで、特定海域の船舶リアルタイム追跡システムを比較的少ないコードで構築できました。

データソース: [aisstream.io](https://aisstream.io/)
