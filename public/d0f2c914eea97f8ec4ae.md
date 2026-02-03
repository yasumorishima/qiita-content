---
title: 和訳 & トラブル対応メモ / Starknetノードインストール
tags:
  - HiÐΞ
private: false
updated_at: '2022-10-24T12:55:44+09:00'
id: d0f2c914eea97f8ec4ae
organization_url_name: null
slide: false
ignorePublish: false
---
# 和訳 & トラブル対応メモ<br><br>初心者向けStarknetネットワーク（GoerliまたはMainnet）にノードをインストールする方法

チュートリアルがなくなると困ってしまうのでメモとして和訳を残しておく

チュートリアル - 初心者向けStarknetネットワーク（GoerliまたはMainnet）にノードをインストールする方法

*   以下の記事<br>
    <https://medium.com/@akabane.kurodo786/tutorial-how-to-install-a-node-on-the-starknet-network-for-beginners-goerli-or-mainnet-c1a162885bea>

***

# 手順・抜粋

準備<br>
パッケージのアップデートを行います

`sudo apt update && sudo apt upgrade -y`

必要なライブラリは、以下のコマンドでインストールします

`sudo apt install pkg-config curl git build-essential libssl-dev`

スクリーンをインストール

`sudo apt install screen`

Linux distroに開発ツールをインストールする<br>
まず、このコマンドでシステムにインストールされている Python 3 のバージョンを確認します。

`python3 -V`

次にpipをインストールします<br>これは将来的にパッケージのインストールと管理を可能にするツールです。

`sudo apt install -y python3-pip`

いくつかのツールを追加でインストールする
あああああ

`sudo apt install -y build-essential libssl-dev libffi-dev python3-dev`

それから

`sudo apt-get install libgmp-dev`

さらに

`pip3 install fastecdsa`

これで fastecdsa (楕円曲線暗号、特に電子署名を高速に行うための python ツール) がインストールされました。

後で何かを見逃さないようにするために

`sudo apt-get install -y pkg-config`

Rust インストール

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y`

もし、1分経ってもインストールが始まらない場合（上の画像のように）、同じコマンドを入力し直せばうまくいくはずですので注意してください

Rust インストール完了

`source $HOME/.cargo/env`

このコマンドを実行するには、少なくともrustc version 1.58以上の最新版が必要です。

`rustc --version`

必要であれば、次のコマンドでアップグレードできます。

`rustup update stable --force`

pathfinder の github リポジトリをクローンする<br>
このコマンドで、pathfinderのgithubリポジトリのローカルコピーを作成します。

`git clone --branch v0.3.7 https://github.com/eqlabs/pathfinder.git`

v0.3.7 の所は最新verを適宜入力<br>
<https://github.com/eqlabs/pathfinder/tags>

ノード用の仮想環境を作成する<br>
python3.8-venvをインストールします。

`sudo apt install python3.8-venv`

そして

`cd pathfinder/py`

次に、このコマンドでvenvという仮想環境を作成します。

`python3 -m venv .venv`

そして、それを起動する

`source .venv/bin/activate`

ノードに必要なツールをインストールします。

`PIP_REQUIRE_VIRTUALENV=true pip install --upgrade pip`

それから

`PIP_REQUIRE_VIRTUALENV=true pip install -r requirements-dev.txt`

テスト開始

`pytest`

ノードのアセンブルと実行<br>
このステップでは、仮想環境(venv)に滞在して、以下のコマンドを実行してノードをコンパイルしてください。

`cargo build --release --bin pathfinder`

別スクリーン作成

`screen -S mystarknet`

"mystarknet"の文字は、何でもOK。このコマンドは初めにしてもOK

そして最後に、Alchemy経由で作成したAPIを使って、以下のコマンドでノードを起動することができます。

`cargo run --release --bin pathfinder -- --ethereum.url XXXXXXXXXXX`

xxxxxxxxはalchemy上のあなたのHTTP APIに置き換えてください

例<br>
`cargo run --release --bin pathfinder -- --ethereum.url https://eth-mainnet.alchemyapi.io/v2/BVqqXjf5MmRNp8LYoQwYj7xOjNO`

祝！ノードが起動しました<br>

ノードの同期の進捗を確認する<br>
5分ほど待ってから、Alchemyアカウントのダッシュボードに戻りましょう。

***

## 補足

実行のままにしたいので、Ctrl + A + D で閉じる

また開きたいときは、<br>`screen -r mystarknet`

***

完全に同期させるためには、最後の1つまでのすべてのブロックをスキャンする必要があります。

starknetですでに生成されたブロックの正確な数を知るには、ここに直接アクセスすることができます。

<https://voyager.online/>

Goerliではなく、Mainnetであることを確認してください。

***

# トラブル対応

*   強制終了<br>kill プロセス番号(PID)

更新の際に古いものが残ってしまって、強制的に消したいときに使う

<https://eng-entrance.com/linux-command-kill#kill>

実行プロセスの確認

`$ ps -a`

具体的に数字を打って消す

`$ kill 24573`

*   fatal: destination path '　' already exists and is not an empty directory <br>の対応

更新の際にこの表示が出てくることがある

対応方法<br>
<https://qiita.com/ochun/items/163abb37339458b3b3db>

`% rm -fr pathfinder`

*   screenを使ったserial接続と終了の方法メモ

<https://qiita.com/135yshr/items/b52c3bf36e27bdf44497>

のような感じ

***

その他のチュートリアル<br><br>StarkNet fullnodeの建て方<br><https://mirror.xyz/0xFA72ba6a332B196fC62bC221E4D32Cd166D0a9aF/rhdfungKnfA-6JTkOZlkccut8DX0bXDbO8fEXd28IEk>

![](https://starkware.co/wp-content/uploads/2021/07/Group-177.svg)
