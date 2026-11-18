---
layout: lecture
title: "ドットファイル"
presenter: Anish
date: 2019-01-24
order: 1
video:
  aspect: 62.5
  id: YSZBWWJw3mI
---

多くのプログラムは、「ドットファイル」と呼ばれるプレーンテキストのファイルで設定されます(`~/.gitconfig` のようにファイル名が `.` で始まるためこう呼ばれます。そのおかげで、デフォルトでは `ls` のディレクトリ一覧に表示されず隠されます)。

あなたが使っているツールの多くには、おそらく、かなり細かく調整できる設定がたくさんあります。多くの場合、ツールは専用の言語でカスタマイズします。たとえばVimならVimscript、シェルならシェル自身の言語です。

ツールをカスタマイズして自分好みのワークフローに合わせれば、生産性が上がります。GitHubから他人のドットファイルをクローンするのではなく、時間をかけて自分でツールをカスタマイズすることをおすすめします。

おそらく、すでにいくつかのドットファイルが設定されているはずです。次のような場所を見てみましょう。

- `~/.bashrc`
- `~/.emacs`
- `~/.vim`
- `~/.gitconfig`

プログラムによっては、ホームフォルダの直下にファイルを置かず、代わりに `~/.config` 以下のフォルダに置くものもあります。

ドットファイルはコマンドラインアプリケーションだけのものではありません。たとえば動画プレーヤーの[MPV](https://mpv.io/)は、`~/.config/mpv` 以下のファイルを編集して設定できます。

# ツールのカスタマイズを学ぶ

ツールの設定については、オンラインのドキュメントや[manページ](https://en.wikipedia.org/wiki/Man_page)を読んで学べます。もうひとつの優れた方法は、特定のプログラムについてのブログ記事をインターネットで検索することです。著者が自分のお気に入りのカスタマイズを紹介してくれています。さらに別の方法として、他の人のドットファイルを覗いてみるという手もあります。GitHubには膨大な数の[dotfilesリポジトリ](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)があります --- もっとも人気のあるものは[こちら](https://github.com/mathiasbynens/dotfiles)です(ただし、設定をやみくもにコピーするのはおすすめしません)。

# 整理の仕方

ドットファイルはどのように整理すべきでしょうか?専用のフォルダにまとめてバージョン管理下に置き、スクリプトを使って所定の場所に**シンボリックリンク**を張るのがよいでしょう。これには次のような利点があります。

- **簡単なインストール**: 新しいマシンにログインしたとき、自分のカスタマイズの適用が1分で済みます
- **ポータビリティ**: どこでもツールが同じように動作します
- **同期**: どこにいてもドットファイルを更新でき、すべてを同期した状態に保てます
- **変更の追跡**: ドットファイルはおそらくプログラマー人生を通してメンテナンスし続けることになるので、長寿命のプロジェクトにはバージョン履歴があると心強いものです

```shell
cd ~/src
mkdir dotfiles
cd dotfiles
git init
touch bashrc
# create a bashrc with some settings, e.g.:
#     PS1='\w > '
touch install
chmod +x install
# insert the following into the install script:
#     #!/usr/bin/env bash
#     BASEDIR=$(dirname $0)
#     cd $BASEDIR
#
#     ln -s ${PWD}/bashrc ~/.bashrc
git add bashrc install
git commit -m 'Initial commit'
```

# 発展的なトピック

## マシン固有のカスタマイズ

たいていの場合、どのマシンでも同じ設定を使いたいものですが、特定のマシンでだけ少し違う設定にしたいこともあります。この状況に対処する方法をいくつか紹介します。

### マシンごとのブランチ

バージョン管理を使って、マシンごとにブランチを維持します。このやり方は論理的にはわかりやすいのですが、かなり手間がかかることがあります。

### if文

設定ファイルがサポートしていれば、if文に相当するものを使ってマシン固有のカスタマイズを適用します。たとえば、シェルでは次のように書けます。

```shell
if [[ "$(uname)" == "Linux" ]]; then {do_something else}; fi

# Darwin is the architecture name for macOS systems
if [[ "$(uname)" == "Darwin" ]]; then {do_something}; fi

# You can also make it machine specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

### インクルード

設定ファイルがサポートしていれば、インクルードを活用します。たとえば、`~/.gitconfig` には次のような設定を書けます。

```
[include]
    path = ~/.gitconfig_local
```

そして、各マシンの `~/.gitconfig_local` にマシン固有の設定を書いておけます。これらをマシン固有の設定用の別のリポジトリで管理することもできます。

この考え方は、異なるプログラムのあいだで設定の一部を共有したい場合にも便利です。たとえば、`bash` と `zsh` の両方で同じエイリアスを共有したければ、エイリアスを `.aliases` に書いておき、両方に次のブロックを入れておけばよいのです。

```bash
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

# 参考資料

- 講師たちのドットファイル: [Anish](https://github.com/anishathalye/dotfiles)、[Jon](https://github.com/jonhoo/configs)、[Jose](https://github.com/jjgo/dotfiles)
- [GitHub does dotfiles](https://dotfiles.github.io/): ドットファイルのフレームワーク、ユーティリティ、例、チュートリアル
- [シェルの起動スクリプト](https://web.archive.org/web/20260329133158/https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html): シェルで使われるさまざまな設定ファイルの解説

# 演習

1. ドットファイル用のフォルダを作り、[バージョン管理]({{ site.baseurl }}/2019/version-control/)をセットアップしてください。

1. 少なくとも1つのプログラム(たとえばシェル)の設定を、何らかのカスタマイズ付きで追加してください(手始めとしては、`$PS1` を設定してシェルのプロンプトをカスタマイズするような簡単なものでかまいません)。

1. 新しいマシンにドットファイルをすばやく(かつ手作業なしで)インストールする方法をセットアップしてください。各ファイルに対して `ln -s` を呼ぶだけの単純なシェルスクリプトでもよいですし、[専用のユーティリティ](https://dotfiles.github.io/utilities/)を使ってもかまいません。

1. まっさらな仮想マシンで、インストールスクリプトをテストしてください。

1. 現在使っているすべてのツールの設定を、ドットファイルのリポジトリに移行してください。

1. 自分のドットファイルをGitHubで公開してください。
