---
layout: lecture
title: "シェルツールとスクリプト"
description: >
  シェルスクリプトの書き方と、強力なコマンドラインツールの使い方を学びます。
thumbnail: /static/assets/thumbnails/2020/lec2.png
date: 2020-01-14
ready: true
video:
  aspect: 56.25
  id: kgII-YWo3Zw
---

この講義では、bashをスクリプト言語として使うための基礎と、コマンドラインで絶えず行うことになるもっとも一般的な作業のいくつかをカバーする、さまざまなシェルツールを紹介します。

# シェルスクリプト

ここまで、シェルでコマンドを実行する方法と、それらをパイプでつなぐ方法を見てきました。
しかし多くの場面で、一連のコマンドを実行したり、条件分岐やループのような制御フローの式を使ったりしたくなるでしょう。

シェルスクリプトは、複雑さの次の段階です。
ほとんどのシェルには、変数、制御フロー、独自の構文を備えた独自のスクリプト言語があります。
シェルスクリプトが他のスクリプト言語と異なるのは、シェルに関連する作業を行うために最適化されている点です。
そのため、コマンドのパイプラインを作ること、結果をファイルに保存すること、標準入力から読み込むことは、シェルスクリプトではプリミティブ(基本要素)になっており、汎用のスクリプト言語よりも使いやすくなっています。
この節では、もっとも一般的であるbashスクリプトに焦点を当てます。

bashで変数に代入するには `foo=bar` という構文を使い、変数の値には `$foo` でアクセスします。
`foo = bar` は動作しないことに注意してください。これは、プログラム `foo` を引数 `=` と `bar` で呼び出すものとして解釈されるからです。
一般に、シェルスクリプトではスペース文字が引数の分割を行います。この挙動は最初のうちは紛らわしいので、常に確認するようにしてください。

bashの文字列は `'` と `"` の区切り文字で定義できますが、この2つは等価ではありません。
`'` で囲まれた文字列はリテラル文字列であり、変数の値は置換されませんが、`"` で囲まれた文字列では置換されます。

```bash
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```

多くのプログラミング言語と同様に、bashは `if`、`case`、`while`、`for` などの制御フローの手法をサポートしています。
同様に、`bash` には、引数を受け取ってそれを扱える関数があります。次の例は、ディレクトリを作成してそこに `cd` する関数です。


```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

ここで `$1` は、スクリプト/関数への最初の引数です。
他のスクリプト言語とは異なり、bashは引数、エラーコード、その他の関連する変数を参照するために、さまざまな特別な変数を使います。以下はその一部の一覧です。より網羅的な一覧は[こちら](https://tldp.org/LDP/abs/html/special-chars.html)にあります。
- `$0` - スクリプトの名前
- `$1` から `$9` - スクリプトへの引数。`$1` が最初の引数で、以下同様です。
- `$@` - すべての引数
- `$#` - 引数の個数
- `$?` - 直前のコマンドのリターンコード
- `$$` - 現在のスクリプトのプロセス識別番号(PID)
- `!!` - 引数を含む、直前のコマンド全体。よくあるパターンとして、コマンドを実行したものの権限不足で失敗する、ということがあります。そのときは `sudo !!` とすれば、sudo付きですばやくコマンドを再実行できます
- `$_` - 直前のコマンドの最後の引数。対話的なシェルでは、`Esc` に続けて `.` を入力するか、`Alt+.` を入力することでも、この値をすばやく取得できます

コマンドは多くの場合、`STDOUT` を使って出力を返し、`STDERR` を通じてエラーを返し、さらに、よりスクリプトで扱いやすい形でエラーを報告するためにリターンコードを返します。
リターンコード(終了ステータス)は、スクリプトやコマンドが実行の結果を伝えるための手段です。
値0は通常、すべてうまくいったことを意味し、0以外の値はエラーが発生したことを意味します。

終了コードは、`&&`(AND演算子)と `||`(OR演算子)を使ってコマンドを条件付きで実行するために使えます。どちらも[短絡評価](https://en.wikipedia.org/wiki/Short-circuit_evaluation)を行う演算子です。コマンドは、セミコロン `;` を使って同じ行の中で区切ることもできます。
`true` プログラムは常にリターンコード0を返し、`false` コマンドは常にリターンコード1を返します。
いくつか例を見てみましょう。

```bash
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run
```

もうひとつのよくあるパターンは、コマンドの出力を変数として取得したいというものです。これは「_コマンド置換_」で行えます。
`$( CMD )` と書くと、`CMD` が実行され、そのコマンドの出力が取得されて、その場所に置き換えられます。
たとえば `for file in $(ls)` とすると、シェルはまず `ls` を呼び出し、それからその値を順に処理します。
これに似た、あまり知られていない機能に「_プロセス置換_」があります。`<( CMD )` は `CMD` を実行し、その出力を一時ファイルに書き出したうえで、`<()` の部分をそのファイル名に置き換えます。これは、コマンドがSTDINではなくファイル経由で値を渡されることを期待している場合に便利です。たとえば `diff <(ls foo) <(ls bar)` は、ディレクトリ `foo` と `bar` のファイルの違いを表示します。


一度に大量の情報を詰め込んだので、これらの機能のいくつかを使った例を見てみましょう。この例は、与えた引数を順に処理し、文字列 `foobar` を `grep` で探して、見つからなければファイルにコメントとして追記します。

```bash
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

比較の部分では、`$?` が0と等しくないかどうかを調べました。
bashはこの種の比較を数多く実装しています。詳細な一覧は [`test`](https://www.man7.org/linux/man-pages/man1/test.1.html) のmanページにあります。
bashで比較を行うときは、単一の角括弧 `[ ]` よりも二重の角括弧 `[[ ]]` を使うようにしてください。`sh` への移植性はなくなりますが、ミスをする可能性は低くなります。より詳しい説明は[こちら](https://mywiki.wooledge.org/BashFAQ/031)にあります。

スクリプトを起動するとき、似たような引数を与えたいことがよくあります。bashには、ファイル名展開を行って式を展開することで、これを簡単にする方法があります。これらの手法は、しばしばシェルの「_グロブ_」と呼ばれます。
- ワイルドカード - 何らかのワイルドカードマッチを行いたいときは、`?` と `*` を使って、それぞれ1文字または任意の数の文字にマッチさせられます。たとえば、`foo`、`foo1`、`foo2`、`foo10`、`bar` というファイルがあるとき、コマンド `rm foo?` は `foo1` と `foo2` を削除し、`rm foo*` は `bar` 以外をすべて削除します。
- 波括弧 `{}` - 一連のコマンドに共通の部分文字列があるときは、波括弧を使ってbashに自動的に展開させることができます。これは、ファイルを移動したり変換したりするときにとても便利です。

```bash
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files


mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y
```

<!-- 最後に、パイプ `|` はスクリプトの中核となる機能です。パイプは、あるプログラムの出力を次のプログラムの入力に接続します。パイプについては、データラングリングの講義でより詳しく扱います。 -->

`bash` スクリプトを書くのは厄介で、直感に反することもあります。[shellcheck](https://github.com/koalaman/shellcheck)のように、sh/bashスクリプトの誤りを見つけるのに役立つツールがあります。

なお、ターミナルから呼び出すスクリプトは、必ずしもbashで書かれている必要はありません。たとえば、次は引数を逆順に出力する簡単なPythonスクリプトです。

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

カーネルは、スクリプトの先頭に[シェバン(shebang)](https://en.wikipedia.org/wiki/Shebang_(Unix))行が含まれているおかげで、このスクリプトをシェルコマンドではなくpythonインタプリタで実行すべきだとわかります。
シェバン行は [`env`](https://www.man7.org/linux/man-pages/man1/env.1.html) コマンドを使って書くのが良い習慣です。`env` は、そのコマンドがシステム上のどこにあっても解決してくれるので、スクリプトの移植性が高まります。場所を解決するために、`env` は最初の講義で紹介した環境変数 `PATH` を利用します。
この例では、シェバン行は `#!/usr/bin/env python` のようになります。

シェルの関数とスクリプトには、覚えておくべき違いがいくつかあります。
- 関数はシェルと同じ言語で書かなければなりませんが、スクリプトはどんな言語でも書けます。スクリプトにシェバンを含めることが重要なのは、このためです。
- 関数は、その定義が読み込まれたときに一度だけロードされます。スクリプトは、実行されるたびにロードされます。そのため関数のほうがロードはわずかに速いのですが、変更するたびに定義を再読み込みしなければなりません。
- 関数は現在のシェル環境で実行されますが、スクリプトは独自のプロセスで実行されます。したがって、関数は環境変数を変更できます(たとえば、カレントディレクトリを変更できます)が、スクリプトにはできません。[`export`](https://www.man7.org/linux/man-pages/man1/export.1p.html) を使ってエクスポートされた環境変数は、値渡しでスクリプトに渡されます。
- どんなプログラミング言語でもそうであるように、関数は、モジュール性、コードの再利用、シェルコードの明快さを実現するための強力な構成要素です。シェルスクリプトには、多くの場合、独自の関数定義が含まれます。

# シェルツール

## コマンドの使い方を調べる

ここまで来ると、エイリアスの節に出てきた `ls -l`、`mv -i`、`mkdir -p` のようなコマンドのフラグは、どうやって調べればよいのだろうと思うかもしれません。
より一般的に言えば、あるコマンドが与えられたとき、それが何をするのか、どんなオプションがあるのかを、どうやって調べればよいのでしょうか?
いつでもググることから始められますが、UNIXはStackOverflowよりも前から存在するので、この情報を得るための組み込みの方法があります。

シェルの講義で見たように、まず試すべき方法は、そのコマンドを `-h` または `--help` フラグ付きで呼び出すことです。より詳しく知るには、`man` コマンドを使います。
manualの略である [`man`](https://www.man7.org/linux/man-pages/man1/man.1.html) は、指定したコマンドのマニュアルページ(manページと呼ばれます)を表示します。
たとえば `man rm` は、`rm` コマンドの挙動と、先ほど紹介した `-i` フラグを含め、受け付けるフラグを出力します。
実のところ、ここまで各コマンドに張ってきたリンクは、そのコマンドのLinux manページのオンライン版です。
自分でインストールした標準外のコマンドであっても、開発者がmanページを書いてインストール手順に含めていれば、manページの項目があります。
ncursesベースのもののような対話的なツールでは、多くの場合、プログラム内で `:help` コマンドを使うか `?` を入力することで、コマンドのヘルプを参照できます。

manページは、コマンドの説明が詳しすぎて、よくある用途でどのフラグや構文を使えばよいのかを読み解きにくいことがあります。
[TLDR pages](https://tldr.sh/)は、コマンドの使用例を示すことに重点を置いた気の利いた補完的な解決策で、どのオプションを使えばよいかをすばやく把握できます。
たとえば私は、[`tar`](https://tldr.inbrowser.app/pages/common/tar) や [`ffmpeg`](https://tldr.inbrowser.app/pages/common/ffmpeg) については、manページよりもtldrのページを参照することのほうがずっと多いです。


## ファイルを探す

すべてのプログラマーが直面する、もっともよくある反復作業のひとつが、ファイルやディレクトリを探すことです。
UNIX系のシステムにはすべて [`find`](https://www.man7.org/linux/man-pages/man1/find.1.html) が付属しています。これはファイルを探すための優れたシェルツールです。`find` は、何らかの条件にマッチするファイルを再帰的に検索します。いくつか例を挙げます。

```bash
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '*/test/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```
ファイルを一覧表示するだけでなく、findは検索条件にマッチしたファイルに対して操作を行うこともできます。
この性質は、かなり単調になりかねない作業を簡単にするうえで、非常に役立ちます。
```bash
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;

# Find all PNG files and convert them to JPG
find . -name '*.png' -exec magick {} {}.jpg \;
```

`find` はどこにでもありますが、その構文は覚えにくいことがあります。
たとえば、あるパターン `PATTERN` にマッチするファイルを単に探すだけでも、`find -name '*PATTERN*'`(パターンマッチで大文字と小文字を区別したくない場合は `-iname`)を実行しなければなりません。
そうした場面のためにエイリアスを作り始めてもよいのですが、代替手段を探ってみるのが良いというのも、シェルの哲学の一部です。
思い出してください。シェルの最良の性質のひとつは、単にプログラムを呼び出しているだけだということです。したがって、一部のプログラムについては、代わりになるものを見つける(あるいは自分で書く)ことができます。
たとえば [`fd`](https://github.com/sharkdp/fd) は、`find` に代わる、シンプルで高速かつユーザーフレンドリーなツールです。
色付きの出力、デフォルトでの正規表現マッチ、Unicodeサポートといった、気の利いたデフォルトを備えています。また、私の意見では、構文もより直感的です。
たとえば、パターン `PATTERN` を探す構文は `fd PATTERN` です。

`find` と `fd` が優れていることにはほとんどの人が同意するでしょうが、毎回ファイルを探すのと、すばやく検索するために何らかのインデックスやデータベースを構築しておくのとでは、どちらが効率的なのかと疑問に思う人もいるかもしれません。
そのためにあるのが [`locate`](https://www.man7.org/linux/man-pages/man1/locate.1.html) です。
`locate` は、[`updatedb`](https://www.man7.org/linux/man-pages/man1/updatedb.1.html) を使って更新されるデータベースを利用します。
ほとんどのシステムでは、`updatedb` による更新は [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html) を通じて毎日行われます。
したがって、両者のトレードオフのひとつは、速度と鮮度のどちらを取るかです。
さらに、`find` やそれに類するツールは、ファイルサイズ、更新時刻、ファイルのパーミッションといった属性を使ってファイルを探すこともできますが、`locate` はファイル名しか使いません。
より踏み込んだ比較は[こちら](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)にあります。

## コードを探す

ファイルを名前で探せるのは便利ですが、ファイルの *内容* に基づいて検索したいことも多々あります。
よくあるのは、あるパターンを含むすべてのファイルを、そのパターンがファイル内のどこに現れるかとあわせて検索したいという場面です。
これを実現するために、ほとんどのUNIX系システムは [`grep`](https://www.man7.org/linux/man-pages/man1/grep.1.html) を提供しています。これは、入力テキストからパターンにマッチする部分を探す汎用ツールです。
`grep` は非常に価値の高いシェルツールで、データラングリングの講義でさらに詳しく扱います。

今のところは、`grep` には多くのフラグがあり、それによって非常に多用途なツールになっている、ということを知っておいてください。
私がよく使うのは、マッチした行の前後の文脈(**C**ontext)を取得する `-C` と、マッチを反転(in**v**ert)させる、つまりパターンにマッチ**しない**行をすべて表示する `-v` です。たとえば `grep -C 5` は、マッチの前後5行を表示します。
多数のファイルをすばやく検索したいときは、`-R` を使うとよいでしょう。ディレクトリに再帰的(**R**ecursive)に入っていき、マッチする文字列をファイルから探してくれます。

しかし `grep -R` は、`.git` フォルダを無視する、マルチCPUを活用するなど、さまざまな点で改善できます。
`grep` の代替ツールは数多く開発されており、[ack](https://github.com/beyondgrep/ack3)、[ag](https://github.com/ggreer/the_silver_searcher)、[rg](https://github.com/BurntSushi/ripgrep)などがあります。
どれも素晴らしく、ほぼ同じ機能を提供しています。
今のところ私は、その速さと直感的な使い勝手から、ripgrep(`rg`)を使い続けています。いくつか例を挙げます。
```bash
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#\!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

なお、`find`/`fd` の場合と同様に、重要なのは、こうした問題がこれらのツールのどれかを使えばすばやく解決できると知っていることであり、具体的にどのツールを使うかはそれほど重要ではありません。

## シェルコマンドを探す

ここまで、ファイルやコードを探す方法を見てきましたが、シェルで過ごす時間が長くなるにつれて、以前に入力した特定のコマンドを探したくなることがあるでしょう。
まず知っておくべきなのは、上矢印キーを押すと直前のコマンドが戻ってくること、そして押し続けるとシェルの履歴を少しずつさかのぼれることです。

`history` コマンドを使うと、シェルの履歴にプログラム的にアクセスできます。
このコマンドは、シェルの履歴を標準出力に表示します。
その中を検索したければ、出力を `grep` にパイプしてパターンを検索できます。
`history | grep find` は、「find」という部分文字列を含むコマンドを表示します。

ほとんどのシェルでは、`Ctrl+R` を使って履歴を逆方向に検索できます。
`Ctrl+R` を押したあと、履歴内のコマンドにマッチさせたい部分文字列を入力できます。
押し続けると、履歴内のマッチを順に巡っていきます。
これは、[zsh](https://github.com/zsh-users/zsh-history-substring-search)では上下の矢印キーでも有効にできます。
`Ctrl+R` をさらに便利にしてくれるのが、[fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r)のキーバインドです。
`fzf` は汎用のファジーファインダーで、多くのコマンドと組み合わせて使えます。
ここでは、履歴をあいまいにマッチさせ、結果を便利で見た目にも心地よい形で提示するために使われています。

もうひとつ、私がとても気に入っている履歴関連のクールな技が、**履歴に基づく自動サジェスト**です。
[fish](https://fishshell.com/)シェルが最初に導入したこの機能は、入力中のシェルコマンドを、共通のプレフィックスを持つ直近に入力したコマンドで動的に自動補完します。
[zsh](https://github.com/zsh-users/zsh-autosuggestions)でも有効にでき、シェルでの作業を快適にしてくれる素晴らしい技です。

シェルの履歴の挙動は変更できます。たとえば、先頭にスペースがあるコマンドを履歴に含めないようにできます。これは、パスワードやその他の機密情報を含むコマンドを入力するときに便利です。
そうするには、`.bashrc` に `HISTCONTROL=ignorespace` を、または `.zshrc` に `setopt HIST_IGNORE_SPACE` を追加します。
先頭のスペースを付け忘れてしまった場合は、`.bash_history` や `.zsh_history` を編集して、いつでも手動でその項目を削除できます。

## ディレクトリの移動

ここまでは、これらの操作を行うべき場所にすでにいることを前提としてきました。では、ディレクトリをすばやく移動するにはどうすればよいのでしょうか?
シェルのエイリアスを書いたり、[ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html)でシンボリックリンクを作ったりと、簡単な方法はいろいろありますが、実のところ、開発者たちはすでにかなり巧妙で洗練された解決策を考え出しています。

この授業のテーマにもあるように、多くの場合、よくあるケースに最適化するのが得策です。
頻繁に使うファイルやディレクトリ、最近使ったファイルやディレクトリは、[`fasd`](https://github.com/clvv/fasd) や [`autojump`](https://github.com/wting/autojump) のようなツールで見つけられます。
Fasdは、ファイルとディレクトリを[_frecency_](https://web.archive.org/web/20210421120120/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm)、つまり _頻度(frequency)_ と _新しさ(recency)_ の両方でランク付けします。
デフォルトでは、`fasd` は `z` コマンドを追加します。これを使うと、_frecent_ なディレクトリの部分文字列を指定してすばやく `cd` できます。たとえば、`/home/user/files/cool_project` によく行くのであれば、`z cool` とするだけでそこにジャンプできます。autojumpを使う場合は、同じディレクトリ移動を `j cool` で行えます。

ディレクトリ構造の概要をすばやく把握するための、より複雑なツールもあります。[`tree`](https://linux.die.net/man/1/tree)、[`broot`](https://github.com/Canop/broot)、さらには [`nnn`](https://github.com/jarun/nnn) や [`ranger`](https://github.com/ranger/ranger) のような本格的なファイルマネージャまであります。

# 演習 {#exercises}

1. [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) を読んで、次の条件でファイルを一覧表示する `ls` コマンドを書いてください。

    - 隠しファイルを含むすべてのファイルを含める
    - サイズは人間が読みやすい形式で表示する(例: 454279954ではなく454M)
    - ファイルは新しい順に並べる
    - 出力は色付きにする

    出力例は次のようになります。

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. 次のように動作するbash関数  `marco` と `polo` を書いてください。
`marco` を実行すると、そのときのカレントワーキングディレクトリが何らかの形で保存されます。その後 `polo` を実行すると、どのディレクトリにいようとも、`polo` は `marco` を実行したディレクトリに `cd` で戻してくれます。
デバッグしやすいように、コードは `marco.sh` というファイルに書き、`source marco.sh` を実行することで定義をシェルに(再)読み込みできます。

{% comment %}
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}
{% endcomment %}

1. めったに失敗しないコマンドがあるとしましょう。これをデバッグするには出力を捕捉する必要がありますが、失敗する実行に行き当たるまでには時間がかかることがあります。
以下のスクリプトを失敗するまで実行し続け、その標準出力と標準エラー出力をファイルに記録して、最後にすべてを表示するbashスクリプトを書いてください。
スクリプトが失敗するまでに何回実行されたかも報告できればボーナスポイントです。

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi

    echo "Everything went according to plan"
    ```

{% comment %}
#!/usr/bin/env bash

count=0
until [[ "$?" -ne 0 ]];
do
  count=$((count+1))
  ./random.sh &> out.txt
done

echo "found error after $count runs"
cat out.txt
{% endcomment %}

1. 講義で扱ったように、`find` の `-exec` は、検索しているファイルに対して操作を行ううえで非常に強力です。
しかし、zipファイルを作るなど、**すべて**のファイルに対して何かをしたい場合はどうでしょうか?
ここまで見てきたように、コマンドは引数とSTDINの両方から入力を受け取ります。
コマンドをパイプでつなぐときはSTDOUTをSTDINに接続しますが、`tar` のように引数から入力を受け取るコマンドもあります。
この隔たりを埋めるのが [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) コマンドで、STDINを引数として使ってコマンドを実行します。
たとえば `ls | xargs rm` は、カレントディレクトリのファイルを削除します。

    あなたの課題は、フォルダ内のすべてのHTMLファイルを再帰的に探し、それらをまとめたzipを作るコマンドを書くことです。ファイル名にスペースが含まれていてもコマンドが動作するようにしてください(ヒント: `xargs` の `-d` フラグを調べてみましょう)。
    {% comment %}
    find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf archive.tar.gz
    {% endcomment %}

    macOSを使っている場合、デフォルトのBSD版 `find` は[GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands)に含まれるものとは異なることに注意してください。`find` では `-print0` を、`xargs` では `-0` フラグを使えます。macOSユーザーは、macOSに付属するコマンドラインユーティリティがGNU版とは異なる場合があることを知っておくべきです。お望みなら、[brewを使って](https://formulae.brew.sh/formula/coreutils)GNU版をインストールできます。

1. (上級)ディレクトリ内でもっとも最近変更されたファイルを再帰的に見つけるコマンドまたはスクリプトを書いてください。より一般化して、すべてのファイルを新しい順に一覧表示できますか?
