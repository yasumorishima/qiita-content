---
title: BTC → ckBTC & ETH → ckETH を簡単に行う方法【初心者向け】
tags:
  - ICP
  - HiÐΞ
private: false
updated_at: '2025-02-08T08:44:15+09:00'
id: 374aeb5dd3dbc16add34
organization_url_name: null
slide: false
ignorePublish: false
---
***

# **BTC → ckBTC & ETH → ckETH を簡単に行う方法【初心者向け】**

> **海外取引所を使わず、日本国内でBTC・ETHを使ってICPエコシステムへ接続！**

## **はじめに**

ICP（Internet Computer）では、ビットコインやイーサリアムと直接統合する仕組みがあり、それぞれの資産をICP上で扱えるようにする\*\*「Chain-Key トークン」\*\*（ckBTC・ckETH）が存在します。

✅ **ckBTC（Chain-Key Bitcoin）** → BTCと1:1対応\
✅ **ckETH（Chain-Key Ethereum）** → ETHと1:1対応

この仕組みを活用すると、日本国内の取引所で購入したBTCやETHをICPネットワークに持ち込み、DEXを通じてICPへ交換できます。\
\*\*「なるべく簡単に、手間をかけずにBTC・ETHをICPへ変換したい」\*\*という方のために、最もシンプルな方法を解説します。

***

# **1. BTC → ckBTC を簡単に行う方法**

### **🔹 最も簡単な方法 → 公式のNNSウォレットを使う**

BTCをckBTCに変換するには、**ICP公式の「NNSウォレット」**（Network Nervous System）が最も簡単です。NNSウォレットを使えば、**専用のビットコインアドレスが発行され、送るだけでckBTCが自動ミント**されます。

### **💡 NNSウォレットを使ったckBTC取得の手順**

1.  **NNSウォレットを開く**
    *   [NNS公式サイト](https://nns.ic0.app/) にアクセスし、**Internet Identity**（生体認証やセキュリティキー）でログイン。
    *   初めての方は、簡単なセットアップでアカウントを作成できます。

2.  **ckBTCの入金アドレスを取得**
    *   ログイン後、**「My Tokens」** 画面で **ckBTC** を選択。
    *   「Receive（受け取る）」ボタンを押すと、専用の **BTC入金アドレス** が表示されます。
    *   **⚠️ 重要** → このアドレスは**あなた専用**のビットコインアドレスです。

3.  **国内取引所からBTCを送金**
    *   bitFlyer、GMOコイン、コインチェックなどの**国内取引所から、NNSで取得したBTCアドレス宛に送金**。
    *   **送金額は自由**ですが、取引所の最低出金額に注意。
    *   送金手数料（例：bitFlyerなら0.0004 BTC）も確認してください。

4.  **ckBTCが自動的にミントされる**
    *   ビットコインネットワークで\*\*12回のブロック承認（約1時間）\*\*が完了すると、NNSウォレット内に **ckBTC** が自動的に反映されます。
    *   **「Refresh（更新）」ボタン**を押すと最新残高が表示されます。

✅ **この方法のメリット**

*   **最も簡単** → 送金するだけでckBTCが自動ミント！
*   **セキュア** → 公式NNSが対応しており、スマートコントラクトベースで安全。
*   **手数料が安い** → 送金時のBTCネットワーク手数料のみ（ICPネットワーク上の手数料はほぼゼロ）。

✅ **この方法の注意点**

*   BTCの承認には時間がかかる（約1時間程度）。
*   BTC送金時のネットワーク手数料が発生する。

💡 **これが最も手軽な方法なので、BTC→ckBTCはNNSウォレット一択でOK！**

***

# **2. ETH → ckETH を簡単に行う方法**

### **🔹 ICLighthouse を使う（Etherscan不要！）**

ckETHの取得方法には **ICLighthouse** という公式推奨のWebサービスがあり、Etherscanを使わずに簡単にETH→ckETH変換ができます。

### **💡 ICLighthouseを使ったckETH取得の手順**

1.  **ICLighthouseのWebサイトにアクセス**
    *   [ICLighthouse公式サイト](https://iclight.io/) にアクセス。
    *   **「Connect Wallet」ボタン**を押し、ETHウォレット（**MetaMask**）とICPウォレット（**Plug Wallet** など）を接続。

2.  **ckETHのブリッジを選択**
    *   **「Ethereum → ICP」** のブリッジを選択し、**ETH → ckETH** を選ぶ。

3.  **ETHを送金（最低 0.03 ETH 以上）**
    *   **送金するETHの金額を入力（最低 0.03 ETH 以上）**
    *   「Confirm（確認）」ボタンを押し、MetaMaskでトランザクションを承認。
    *   送信先のスマートコントラクトが自動で自分のICPアドレス（Plugウォレット）を識別。

4.  **ckETHが自動的にICPウォレットに反映**
    *   Ethereumチェーン上でブロック承認されると（通常約10～20分）、**ICP側のウォレットにckETHが自動入金**されます。
    *   Plugウォレットの「Refresh（更新）」でckETH残高を確認。

✅ **この方法のメリット**

*   **Etherscan不要！** → Web UI上で簡単に操作できる。
*   **セキュア** → ICLighthouseがスマートコントラクトを正しく制御。
*   **高速処理** → 10～20分程度で完了（BTCより早い）。

✅ **この方法の注意点**

*   **ガス代（ETHのネットワーク手数料）が発生する** → 取引混雑時は高くなるので注意。
*   **最低0.03 ETH以上が必要** → 少額は対応不可。

💡 **ETH→ckETHは、IClighthouseを使うのが圧倒的に簡単！**

***

# **3. 重要ポイント：BTCとETHのブリッジの違い**

| 比較項目 | BTC → ckBTC | ETH → ckETH |
|----------|------------|------------|
| どのチェーンを利用？ | **Bitcoin チェーン** | **Ethereum チェーン** |
| 承認時間 | 約1時間（12回承認） | 10～20分（ガス代次第） |
| 手数料 | BTCのネットワーク手数料（取引所による） | ETHのガス代（変動あり） |
| 最小送金額 | なし（0.0001 BTC でも可） | **0.03 ETH 以上** |
| 簡単な取得方法 | **NNSウォレットを使う** | **ICLighthouseを使う** |

🔹 **BTCは「NNSウォレット」、ETHは「ICLighthouse」を使えばOK！**\
🔹 **ETH → ckETH は Ethereum チェーン上の処理**（Ethereum のネットワークでガス代がかかる）。\
🔹 **BTC → ckBTC は Bitcoin チェーン上の処理**（ビットコインのマイナー承認が必要）。

***

# **まとめ**

### **✅ BTC → ckBTC を簡単に行うなら？**

👉 **NNSウォレット（公式サイト）を使うのが最も簡単！**\
　→ NNSで**専用BTCアドレスを発行 → BTCを送るだけ！**

### **✅ ETH → ckETH を簡単に行うなら？**

👉 **ICLighthouse（公式サイト）を使えばEtherscan不要！**\
　→ Web UIで**MetaMaskを接続し、ETHを送るだけ！**

### **✅ どちらを選ぶべき？**

💡 **BTCを持っているなら、NNSウォレットでckBTC化が最も簡単！**\
💡 **ETHを持っているなら、ICLighthouseでckETH化するのがベスト！**

🚀 **これで、海外取引所を使わずに簡単にICPエコシステムへ参加できます！**
