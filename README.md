# The Missing Semester of Your CS Education

[![Build Status](https://github.com/koh-t/missing-semester/actions/workflows/build.yml/badge.svg)](https://github.com/koh-t/missing-semester/actions/workflows/build.yml) [![Links Status](https://github.com/koh-t/missing-semester/actions/workflows/links.yml/badge.svg)](https://github.com/koh-t/missing-semester/actions/workflows/links.yml)

このリポジトリは、MITの授業[The Missing Semester of Your CS Education](https://missing.csail.mit.edu/)のWebサイトを日本語に翻訳した、非公式のフォークです。原文は[missing-semester/missing-semester](https://github.com/missing-semester/missing-semester)にあります。

コントリビューションは大歓迎です! 修正や追加したい新しい内容がある場合は、issueを立てるか、プルリクエストを送ってください。

## 開発

サイトをローカルでビルドして表示するには、次を実行します。

```bash
bundle exec jekyll serve -w
```

Dockerコンテナ内でサイトを開発したい場合(たとえば、ホストマシンにRubyや依存関係をインストールしたくない場合)は、次を実行します。

```bash
docker compose up --build
```

その後、ホストマシンで <http://localhost:4000> にアクセスすると、Webサイトを表示できます。ファイルに変更を加えると、JekyllがWebサイトを再ビルドします。

## ライセンス

Webサイトのソースコード、講義ノート、演習、講義動画を含む、このコースのすべてのコンテンツは、[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)のもとでライセンスされています。コントリビューションや翻訳についての詳細は、[こちら](https://missing.csail.mit.edu/license)をご覧ください。
