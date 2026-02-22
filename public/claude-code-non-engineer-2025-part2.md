---
title: 非エンジニアがClaude Codeで未知の言語（Motoko）のOSSにPRを出した話
tags:
  - OSS
  - 生成AI
  - Claude
  - ClaudeCode
  - FindyTeamPlus_AI_2025
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

[前回の記事](https://qiita.com/ussu_ussu_ussu/items/33ba41fadad02215aede)で、「非エンジニアがClaude Codeを使って1ヶ月でできたこと」を書きました。

あれから少し経って、さらに踏み込んだ体験をしたので続編です。今回のテーマは**「自分が全く知らない言語で書かれたOSSプロジェクトにPRを出せた」**という話です。

---

## 何をしたのか

**dfinity/pic-js** というOSSプロジェクトにプルリクエストを出しました。

https://github.com/dfinity/pic-js/pull/235

これはInternet Computer（ICP）というブロックチェーンプラットフォームのテスト用ライブラリです。本体はTypeScriptですが、テスト用のCanister（スマートコントラクト）は**Motoko**という言語で書かれています。

Motokoは、私にとって完全に未知の言語でした。

---

## 何が「未知」だったのか

正直に書きます。

- **Motoko**: 見たことも触ったこともない。構文もわからない
- **dfx**: ICP開発用のCLIツール。インストール方法も知らなかった
- **Candid**: ICPのインターフェース記述言語。エンコード・デコードの仕組みは初見
- **PocketIC**: テスト用のローカルIC環境。存在自体をこの作業で知った
- **GitHub Codespace上での開発**: ローカルPCがRAM 4GBのため、クラウド開発環境を使う必要があった

要するに、プロジェクトの技術スタック全体が未知でした。

---

## どう進めたのか

### 1. Issueを見つける

GitHub上でdfinity organizationのopen issueを調べました。[#68](https://github.com/dfinity/pic-js/issues/68)は「canisterのtrapログが表示されない」というバグで、メンテナーが**「PRs are welcome」**と明言していたものです。

### 2. 実現可能性を調査する

PRを出すと宣言する前に、まず「本当にできるのか」を確認しました。

- リポジトリの構造を読む
- 既存のコードパターン（management canister呼び出しの書き方）を理解する
- Rust版の参考実装（[dfinity/ic #2533](https://github.com/dfinity/ic/pull/2533)）を読む
- IC Interface Specの`fetch_canister_logs`仕様を確認する

この調査はClaude Codeに任せました。自分では何を調べればいいかすら分からなかったからです。

### 3. Codespaceで環境構築する

ローカルPCのスペックが足りないため、GitHub Codespaceを使いました。

```
# Codespace上での作業
pnpm install          # 依存インストール → 12.8秒
pnpm run build        # ビルド確認
pnpm run test:pic     # 既存テスト11件 → 全パス
dfxvm install 0.29.1  # Motoko canisterのビルドに必要
pnpm run build:test-canister  # テスト用canisterのビルド
```

dfxのインストールも、バージョン指定も、全部Claude Codeが進めました。私はエラーが出たら「どうする？」と聞いて、「OK」と答えるだけでした。

### 4. 実装する

変更したのは以下のファイルです。

| ファイル | 内容 |
|---|---|
| `management-canister.ts` | Candid型定義 + エンコード/デコード関数 |
| `pocket-ic-types.ts` | 公開型（`FetchCanisterLogsOptions`等） |
| `pocket-ic.ts` | `fetchCanisterLogs()`メソッド本体 |
| `main.mo` | テスト用canisterに`print_log`メソッド追加 |
| `fetchCanisterLogs.spec.ts` | テスト2件 |

Motokoのコードは3行だけ追加しました。

```motoko
public func print_log(message : Text) : async () {
    Debug.print(message);
};
```

この3行すら、自分では書けなかったと思います。

### 5. テストで引っかかる

最初のテスト実行で失敗しました。

```
Caller 2vxsx-fae is not allowed to access canister logs.
```

anonymous principalではcanisterログにアクセスできない、というエラーでした。テストコードを修正して、controllerのprincipalを渡すようにしたら全パス。

こういう「実行してみないと分からない」問題は、仕様書を読むだけでは気づけません。

### 6. CI全パス → PR提出

フォーク内でDraft PRを作り、以下を確認してからupstream PRを出しました。

- ビルド: パス
- ユニットテスト: 13件全パス（既存11 + 新規2）
- e2eテスト: Node.js + Bun、Ubuntu + macOS、全パス
- フォーマットチェック: パス
- CodeQL: パス
- CodeRabbit: 指摘ゼロ

---

## 私が実際にやったこと

振り返ると、私がやったのはこういうことでした。

- **「このissueやってみたい」と決めた**
- **「先に調査してから」と判断した**
- **エラーが出たら「どうする？」と聞いた**
- **テスト結果を見て「OK」と言った**
- **「PR出して」と言った**

コードの実装はClaude Codeがやりました。でも、「何をやるか」「いつやるか」「出していいか」を決めたのは自分です。

---

## 前回の記事で書いた「感じたこと」の答え合わせ

前回、こう書きました。

> **「分からないまま進める」ができるようになった**

今回はまさにそれの極端なケースでした。Motokoもdfxも知らないまま、PRまで出せています。

> **任せすぎない方がいい**

これは今回も痛感しました。最初のテスト失敗（anonymous principal問題）は、Claude Codeが「コントローラーとして送る必要がある」とすぐ修正してくれましたが、**なぜそうなるのかを自分でも理解しておかないと、次に似た問題が出たとき対処できません**。

> **「自分が何をしたいか」を言語化するのは自分の仕事**

今回、ICP系のOSSをいくつか調べた中で、「このissueは保留にしよう」「こっちはメンテナーとの関係を考えて今は出さない方がいい」という判断がありました。AIに「全部やって」と言っても、こういう文脈の読み取りはできません。

---

## 数字で見る

| 項目 | 値 |
|---|---|
| 作業時間（体感） | 約1時間 |
| 追加コード行数 | 約140行 |
| Motokoの知識 | ゼロのまま |
| dfxの知識 | ゼロのまま |
| Codespace費用 | 無料枠内（2コア） |

---

## まとめ

非エンジニアが、知らない言語のOSSにPRを出せる時代になっています。

ただし、「AIが全部やってくれる」という話ではありません。issueの選定、メンテナーとのコミュニケーション、PRを出すタイミングの判断——これらは人間の仕事として残っています。

技術的な壁が下がった分、**「何をやるか」を決める力**の方が重要になったと感じています。

---

前回の記事:
https://qiita.com/ussu_ussu_ussu/items/33ba41fadad02215aede
