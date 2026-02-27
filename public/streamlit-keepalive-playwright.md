---
title: Streamlit CloudのZzzを自動で防ぐ — HTTP GETでは起きない理由とPlaywrightでの解決法
tags:
  - Python
  - RaspberryPi
  - GitHubActions
  - Playwright
  - Streamlit
private: false
updated_at: '2026-02-27T23:39:43+09:00'
id: 7a30abcf48b482a0d5dc
organization_url_name: null
slide: false
ignorePublish: false
---

## 結論から

Streamlit Community Cloud のアプリがスリープ（Zzz）したとき、`urllib` や `requests` で HTTP GET を送っても**アプリは起きません**。HTTP 200 は返りますが、中身は静的 HTML シェル（4,271 bytes）だけで、Python プロセスは起動しません。

**Playwright（ヘッドレス Chromium）** でブラウザとして訪問すれば、JavaScript が実行されて WebSocket 接続が確立し、アプリが実際に起動します。

---

## 背景

WBC 2026 のスカウティングダッシュボードを 30 個デプロイしています。しばらくアクセスがないとスリープし、「Zzzz — This app has gone to sleep」と表示されます。

Raspberry Pi で定期的に HTTP GET を送って keepalive にしていましたが、**200 が返っているのにアプリは寝たまま**でした。

---

## なぜ HTTP GET では起きないのか

Streamlit のレスポンスを確認したところ、全 URL が同じ 4,271 bytes の HTML を返していました。

```html
<!doctype html>
<html lang="en">
  <head>...</head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```

これは SPA（Single Page Application）の典型的な構造です。

```
HTTP GET → 静的 HTML シェルを返す（ここで終わり）
ブラウザ → HTML → JS 実行 → WebSocket (/_stcore/stream) → Python 起動
```

HTTP GET では JavaScript が実行されないため、WebSocket 接続も発生せず、Python アプリは起動しません。HTTP 200 は「静的ファイルの配信に成功した」という意味でしかありません。

:::note info
`/_stcore/health` エンドポイントも試しましたが、アプリがスリープ中は 303 リダイレクトになり、最終的に同じ HTML シェルが返るだけでした。
:::

---

## 解決: Playwright でブラウザ訪問

Playwright を使えばヘッドレス Chromium で実際にページを開き、JavaScript を実行できます。スリープ中の場合は「Yes, get this app back up!」ボタンも自動クリックします。

```python
from playwright.async_api import async_playwright

async def visit(page, url):
    await page.goto(url, wait_until="domcontentloaded", timeout=120_000)
    await page.wait_for_timeout(5000)

    wake_btn = page.get_by_role("button", name="Yes, get this app back up!")
    if await wake_btn.count() > 0:
        print(f"  WAKE  {url}")
        await wake_btn.click()
        await page.wait_for_timeout(60_000)
    else:
        print(f"  OK    {url}")
```

実行結果:

```
  OK    https://npb-prediction.streamlit.app/
  WAKE  https://wbc-cuba-batters.streamlit.app/    ← 寝てたのを起こした
  WAKE  https://wbc-can-batters.streamlit.app/
  OK    https://wbc-can-pitchers.streamlit.app/
```

---

## 実装

### Raspberry Pi Docker（6 時間ごと）

```dockerfile
FROM python:3.12-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    libnss3 libatk-bridge2.0-0 libdrm2 libxkbcommon0 \
    libgbm1 libpango-1.0-0 libcairo2 libasound2 \
    libatspi2.0-0 libcups2 libxcomposite1 libxdamage1 \
    libxfixes3 libxrandr2 libgtk-3-0 libdbus-glib-1-2 \
    fonts-liberation xdg-utils \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir playwright \
    && playwright install chromium

WORKDIR /app
COPY keepalive.py .
CMD ["python", "keepalive.py"]
```

Raspberry Pi 5（ARM64）でも `playwright install chromium` で問題なく動きました。

### GitHub Actions（バックアップ）

```yaml
name: Streamlit Keepalive

on:
  schedule:
    - cron: '0 */6 * * *'
  workflow_dispatch:

jobs:
  keepalive:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: |
          pip install playwright
          playwright install chromium --with-deps
      - run: python scripts/keepalive.py
```

---

## 試行錯誤の記録

| 試したこと | 結果 | 学び |
|---|---|---|
| `urllib.request.urlopen` | 303 エラー | 認証リダイレクトがある |
| `CookieJar` + `build_opener` | 200 だが寝たまま | 静的シェルが返っているだけ |
| `/_stcore/health` | 303 → HTML シェル | スリープ中はヘルスチェックも効かない |
| **Playwright** | **起きた** | JS + WebSocket が必要 |

---

## この知見が活きる場面

- **Render / Railway / Koyeb の無料枠**: 同様のスリープ機能がある
- **ポートフォリオ・デモアプリ**: 採用担当が見に来た時に Zzz ではもったいない
- **SPA の監視**: HTTP 200 = 正常、ではない。ブラウザレンダリング後の状態を確認すべき

---

## リポジトリ

- GitHub: https://github.com/yasumorishima/wbc-scouting
- keepalive スクリプト: [`scripts/keepalive.py`](https://github.com/yasumorishima/wbc-scouting/blob/master/scripts/keepalive.py)
- GitHub Actions: [`.github/workflows/keepalive.yml`](https://github.com/yasumorishima/wbc-scouting/blob/master/.github/workflows/keepalive.yml)
