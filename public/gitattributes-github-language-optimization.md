---
title: .gitattributesでGitHubの言語表示を最適化する
tags:
  - Git
  - GitHub
  - Jupyter
  - データ分析
  - Python
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

データサイエンス系のリポジトリをGitHubに置いていると、こんな表示になることがあります。

**Jupyter Notebook 98.3% / Python 1.7%**

Pythonで書いたコードなのに、Jupyter Notebookがほぼすべてを占有してしまう問題です。`.ipynb`ファイルは中身がJSON（セルのソース・出力・メタデータの塊）なので、ファイルサイズも言語カウントも圧倒的になりがちです。

この記事では、`.gitattributes`を使ってGitHubの言語表示を制御する方法と、その他の実用的な使い方をまとめます。

---

## .gitattributes とは

`.gitattributes`は、Gitがファイルをどう扱うかをパスパターンで定義するファイルです。リポジトリのルートに置くことで、そのリポジトリ全体に適用されます。

```
*.ipynb  linguist-detectable=false
*.csv    diff=csv
*.sh     eol=lf
```

改行コードの統一、差分表示の制御、そしてGitHubの言語検出（GitHub Linguist）の制御もここで行えます。

---

## GitHub Linguist の制御（メイン）

GitHubはリポジトリの言語統計を表示するために **[GitHub Linguist](https://github.com/github-linguist/linguist)** というライブラリを使っています。`.gitattributes`にLinguist用の属性を書くことで、この検出結果を調整できます。

### 属性一覧

| 属性 | 効果 |
|---|---|
| `linguist-detectable=false` | 言語統計から除外する |
| `linguist-vendored` | 外部ライブラリ（vendor）扱いで除外 |
| `linguist-documentation` | ドキュメント扱いで除外 |
| `linguist-generated` | 自動生成ファイル扱いで除外 |
| `linguist-language=Python` | 言語を手動で上書き |

### notebookを除外する（今回のメインユースケース）

```gitattributes
# *.ipynb を言語統計から除外する
*.ipynb linguist-detectable=false
```

これだけで、`.ipynb`ファイルが言語統計から除外され、実際のPythonコードの比率が表示されるようになります。

**`linguist-vendored`との違い**

同じ結果を得るもう1つの書き方として `linguist-vendored` があります。

```gitattributes
*.ipynb linguist-vendored
```

`linguist-vendored`は「外部から持ってきたコード（サードパーティライブラリ等）」という意味合いです。自分で書いたnotebookに適用するのは語義がずれるため、意図を明確にしたい場合は `linguist-detectable=false` の方が適切です。

### その他のユースケース

```gitattributes
# docs/ 以下をドキュメント扱いにして除外
docs/**  linguist-documentation

# ビルド成果物を自動生成扱いにして除外
dist/**  linguist-generated
build/** linguist-generated

# .js ファイルを TypeScript として表示
*.js  linguist-language=TypeScript
```

---

## その他の実用的な使い方

### 改行コードの統一（Windows/Linux混在チーム向け）

WindowsとLinuxが混在するプロジェクトでよく発生する「改行コードが混ざる問題」を防げます。

```gitattributes
# デフォルト: テキストファイルはLFに統一
* text=auto eol=lf

# Windowsでしか使わないファイルはCRLFのまま
*.bat eol=crlf
*.ps1 eol=crlf

# シェルスクリプトは必ずLF
*.sh eol=lf
```

### git archive からファイルを除外

`git archive`（リリース用の配布物作成）から除外したいファイルを指定できます。

```gitattributes
# 開発時のみ必要なファイルをリリース配布から除外
.github/         export-ignore
tests/           export-ignore
*.test.py        export-ignore
CONTRIBUTING.md  export-ignore
```

### バイナリファイルの扱いを明示

```gitattributes
# 画像はバイナリとして扱う（不要な差分を出さない）
*.png  binary
*.jpg  binary
*.gif  binary

# モデルファイルもバイナリ扱い
*.pkl  binary
*.h5   binary
```

---

## データサイエンス系リポジトリのテンプレート

以上をまとめた `.gitattributes` のテンプレートです。

```gitattributes
# ===========================
# Language Detection (GitHub)
# ===========================

# Jupyter Notebookを言語統計から除外
*.ipynb  linguist-detectable=false

# 自動生成ファイルを除外
*.min.js  linguist-generated

# ===========================
# Line Endings
# ===========================

* text=auto eol=lf
*.sh  eol=lf

# ===========================
# Binary Files
# ===========================

*.png   binary
*.jpg   binary
*.jpeg  binary
*.gif   binary
*.pkl   binary
*.parquet  binary
```

---

## まとめ

`.gitattributes`でできることをまとめます。

- **言語表示の最適化**: `linguist-detectable=false` で不要なファイルを除外し、実態に合った言語表示にする
- **改行コードの統一**: Windows/Linux混在環境での `CRLF/LF` 問題を防ぐ
- **バイナリ管理**: 画像・モデルファイルをバイナリとして明示することで余計な差分を出さない
- **リリース配布の整理**: `export-ignore` で開発用ファイルをリリース物から除外する

`.gitattributes`はリポジトリのルートに置くだけで機能するシンプルな仕組みですが、チーム開発・公開リポジトリで地味に効いてくる設定ファイルです。

---

参考:
- [GitHub Linguist - Overrides](https://github.com/github-linguist/linguist/blob/master/docs/overrides.md)
- [gitattributes - Git Documentation](https://git-scm.com/docs/gitattributes)
