---
layout: lecture
title: "コマンドライン環境"
description: >
  入出力ストリーム、環境変数、SSHによるリモートマシンなど、コマンドラインプログラムの仕組みを学びます。
thumbnail: /static/assets/thumbnails/2026/lec2.png
date: 2026-01-13
ready: true
video:
  aspect: 56.25
  id: ccBGsPedE9Q
---

前回の講義で説明したように、ほとんどのシェルは他のプログラムを起動するだけの単なるランチャーではなく、実際には共通のパターンや抽象化を豊富に備えた、ひとつの完全なプログラミング言語を提供しています。
ただし、大多数のプログラミング言語とは異なり、シェルスクリプトではすべてが「プログラムを実行し、それらを簡潔かつ効率的に連携させること」を中心に設計されています。

とりわけ、シェルスクリプトは「_慣習(コンベンション)_」と深く結びついています。コマンドラインインターフェース(CLI)プログラムが、より広いシェル環境の中でうまく振る舞うためには、従うべき共通のパターンがいくつかあります。
これから、コマンドラインプログラムがどのように動作するのかを理解するために必要な概念の数々と、それらを使い、設定するための広く行き渡った慣習について説明していきます。

# コマンドラインインターフェース

ほとんどのプログラミング言語では、関数を書くと次のようになります。

```
def add(x: int, y: int) -> int:
    return x + y
```

ここでは、プログラムの入力と出力が明示的に見て取れます。
それに対して、シェルスクリプトは一見するとかなり違った見た目をしています。

```shell
#!/usr/bin/env bash

if [[ -f $1 ]]; then
    echo "Target file already exists"
    exit 1
else
    if $DEBUG; then
        grep 'error' - | tee $1
    else
        grep 'error' - > $1
    fi
    exit 0
fi
```

このようなスクリプトで何が起きているのかを正しく理解するには、まず、シェルプログラム同士やシェル環境との間でやり取りが行われる際に頻繁に登場する、いくつかの概念を導入する必要があります。

- 引数
- ストリーム
- 環境変数
- リターンコード
- シグナル

## 引数

シェルプログラムは、実行時に引数のリストを受け取ります。
シェルにおいて引数は単なる文字列であり、それをどう解釈するかはプログラム次第です。
たとえば `ls -l folder/` を実行するとき、私たちはプログラム `/bin/ls` を引数 `['-l', 'folder/']` 付きで実行していることになります。

シェルスクリプトの中からは、特別なシェル構文を使ってこれらの引数にアクセスします。
最初の引数には変数 `$1`、2番目の引数には `$2`、というように `$9` までアクセスできます。すべての引数をリストとして取得するには `$@` を使い、引数の個数を取得するには `$#` を使います。さらに、`$0` でプログラム名にもアクセスできます。

ほとんどのプログラムでは、引数は「_フラグ_」と通常の文字列が混在したものになります。
フラグは、ダッシュ(`-`)またはダブルダッシュ(`--`)で始まることで見分けられます。
フラグは通常は省略可能で、その役割はプログラムの挙動を変更することです。
たとえば `ls -l` は、`ls` の出力形式を変更します。

`--all` のような長い名前のダブルダッシュフラグと、`-a` のような、多くの場合1文字が続くシングルダッシュフラグを目にするでしょう。
同じオプションが両方の形式で指定できることもあり、`ls -a` と `ls --all` は等価です。
シングルダッシュのフラグはまとめて書けることが多く、`ls -l -a` と `ls -la` も等価です。
フラグの順序も通常は問題にならず、`ls -la` と `ls -al` は同じ結果になります。
よく使われるフラグはかなり定番化しており、シェル環境に慣れてくると自然に手が伸びるようになるでしょう(たとえば `--help`、`--verbose`、`--version` など)。

> フラグは、シェルの慣習の好例のひとつです。シェル言語は、プログラムが `-` や `--` をこのような特定の方法で使うことを要求しているわけではありません。
`myprogram +myoption myfile` のような構文のプログラムを書くことを妨げるものは何もありませんが、ダッシュを使うのが当然という期待があるため、混乱を招くことになるでしょう。
> 実際には、ほとんどのプログラミング言語がCLIフラグを解析するライブラリを提供しています(たとえばPythonでダッシュ構文の引数を解析する `argparse` など)。

CLIプログラムにおけるもうひとつの一般的な慣習は、同じ種類の引数を可変個受け取れるようにすることです。このように引数が与えられると、コマンドはそれぞれの引数に対して同じ操作を実行します。

```shell
mkdir src
mkdir docs
# is equivalent to
mkdir src docs
```

この糖衣構文は最初は不要に思えるかもしれませんが、「_グロブ(globbing)_」と組み合わせると非常に強力になります。
グロブとは、シェルがプログラムを呼び出す前に展開する特別なパターンのことです。

たとえば、現在のフォルダにあるすべての .py ファイルを(再帰せずに)削除したいとしましょう。前回の講義で学んだことを使えば、次のように実行することで実現できます。

```shell
for file in $(ls | grep -P '\.py$'); do
    rm "$file"
done
```

しかし、これは `rm *.py` と書くだけで置き換えられるのです!

ターミナルに `rm *.py` と入力したとき、シェルは `/bin/rm` プログラムを引数 `['*.py']` で呼び出すわけではありません。
その代わり、シェルは現在のフォルダから `*.py` というパターンにマッチするファイルを探します。ここで `*` は、任意の種類の0文字以上の文字列にマッチします。
したがって、フォルダに `main.py` と `utils.py` があれば、`rm` プログラムは引数 `['main.py', 'utils.py']` を受け取ることになります。

もっともよく見かけるグロブは、ワイルドカードの `*`(任意の文字が0個以上)、`?`(任意の文字がちょうど1個)、そして波括弧です。
波括弧 `{}` は、カンマ区切りのパターンのリストを複数の引数に展開します。

実際のところ、グロブは具体的な例で理解するのが一番です。

```shell
touch folder/{a,b,c}.py
# Will expand to
touch folder/a.py folder/b.py folder/c.py

convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{setup,build,deploy}.sh /newpath
# Will expand to
cp /path/to/project/setup.sh /path/to/project/build.sh /path/to/project/deploy.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files
```

> 一部のシェル(たとえばzsh)は、再帰的なパスにまで展開される `**` のような、さらに高度な形式のグロブをサポートしています。したがって `rm **/*.py` は、すべての .py ファイルを再帰的に削除します。


## ストリーム

次のようなプログラムのパイプラインを実行するとき、

```shell
cat myfile | grep -P '\d+' | uniq -c
```

`grep` プログラムは `cat` と `uniq` の両方のプログラムと通信していることがわかります。

ここで重要な観察は、3つのプログラムがすべて同時に実行されているという点です。
つまり、シェルはまずcatを呼び、次にgrep、それからuniqを呼ぶ、というふうにはしていません。
そうではなく、3つのプログラムがすべて生成され、シェルがcatの出力をgrepの入力に、grepの出力をuniqの入力に接続しているのです。
パイプ演算子 `|` を使うとき、シェルは、チェーン内のあるプログラムから次のプログラムへと流れるデータの「ストリーム」を扱っています。

この並行性を実際に確かめてみましょう。パイプライン内のすべてのコマンドは即座に開始されます。

```console
$ (sleep 15 && cat numbers.txt) | grep -P '^\d$' | sort | uniq  &
[1] 12345
$ ps | grep -P '(sleep|cat|grep|sort|uniq)'
  32930 pts/1    00:00:00 sleep
  32931 pts/1    00:00:00 grep
  32932 pts/1    00:00:00 sort
  32933 pts/1    00:00:00 uniq
  32948 pts/1    00:00:00 grep
```

`cat` を除くすべてのプロセスがすぐに実行されていることがわかります。シェルは、どれかのプロセスが終了する前に、すべてのプロセスを生成してそれらのストリームを接続します。`cat` はsleepが終了して初めて起動され、`cat` の出力がgrepへ送られ、以降も同様に続いていきます。

すべてのプログラムには、stdin(standard input、標準入力)と呼ばれる入力ストリームがあります。パイプを使うと、stdinは自動的に接続されます。スクリプト内では、多くのプログラムがファイル名として `-` を受け付け、「stdinから読み込む」という意味になります。

```shell
# These are equivalent when data comes from a pipe
echo "hello" | grep "hello"
echo "hello" | grep "hello" -
```

同様に、すべてのプログラムには2つの出力ストリーム、stdoutとstderrがあります。
標準出力(stdout)はもっとも頻繁に目にするもので、プログラムの出力をパイプラインの次のコマンドへ渡すために使われるストリームです。
標準エラー出力(stderr)はもうひとつのストリームで、プログラムが警告やその他の問題を報告するためのものであり、その出力がチェーン内の次のコマンドに解析されてしまうことを避けられます。

```console
$ ls /nonexistent
ls: cannot access '/nonexistent': No such file or directory
$ ls /nonexistent | grep "pattern"
ls: cannot access '/nonexistent': No such file or directory
# The error message still appears because stderr is not piped
$ ls /nonexistent 2>/dev/null
# No output - stderr was redirected to /dev/null
```

シェルは、これらのストリームをリダイレクトするための構文を提供しています。以下にわかりやすい例をいくつか示します。

```shell
# Redirect stdout to a file (overwrite)
echo "hello" > output.txt

# Redirect stdout to a file (append)
echo "world" >> output.txt

# Redirect stderr to a file
ls foobar 2> errors.txt

# Redirect both stdout and stderr to the same file
ls foobar &> all_output.txt

# Redirect stdin from a file
grep "pattern" < input.txt

# Discard output by redirecting to /dev/null
cmd > /dev/null 2>&1
```

Unix哲学を体現するもうひとつの強力なツールが、ファジーファインダーの [`fzf`](https://github.com/junegunn/fzf) です。stdinから行を読み込み、絞り込みと選択のためのインタラクティブなインターフェースを提供します。

```console
$ ls | fzf
$ cat ~/.bash_history | fzf
```

`fzf` は、多くのシェル操作と統合できます。シェルのカスタマイズについて説明するときに、さらなる活用例を見ていきます。


## 環境変数

bashで変数に代入するには `foo=bar` という構文を使い、変数の値には `$foo` という構文でアクセスします。
`foo = bar` は不正な構文であることに注意してください。シェルはこれを、プログラム `foo` を引数 `['=', 'bar']` で呼び出すものとして解析します。
シェルスクリプトにおいて、スペース文字の役割は引数の分割を行うことです。
この挙動は紛らわしく、慣れるまでは厄介なので、頭に入れておいてください。

シェル変数に型はなく、すべて文字列です。
シェルで文字列式を書く際、シングルクォートとダブルクォートは交換可能ではないことに注意してください。
`'` で囲まれた文字列はリテラル文字列であり、変数の展開、コマンド置換、エスケープシーケンスの処理は行われませんが、`"` で囲まれた文字列ではこれらが行われます。

```shell
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```

コマンドの出力を変数に取り込むには、「_コマンド置換_」を使います。
次を実行すると、
```shell
files=$(ls)
echo "$files" | grep README
echo "$files" | grep ".py"
```
lsの出力(具体的にはstdout)が変数 `$files` に格納され、後からアクセスできるようになります。
`$files` 変数の内容にはlsの出力に含まれる改行がそのまま含まれており、これによって `grep` のようなプログラムは各項目を個別に処理できるのです。

これに似た、あまり知られていない機能に「_プロセス置換_」があります。`<( CMD )` は `CMD` を実行し、その出力を一時ファイルに書き出したうえで、`<()` の部分をそのファイル名に置き換えます。
これは、コマンドがSTDINではなくファイル経由で値を渡されることを期待している場合に便利です。
たとえば `diff <(ls src) <(ls docs)` は、`src` ディレクトリと `docs` ディレクトリのファイルの違いを表示します。

シェルプログラムが別のプログラムを呼び出すときは、しばしば「_環境変数_」と呼ばれる一連の変数を一緒に渡します。
シェルの中からは、`printenv` を実行することで現在の環境変数を確認できます。
環境変数を明示的に渡すには、コマンドの前に変数代入を付けます。

> 環境変数は、慣習的にすべて大文字で書かれます(例:`HOME`、`PATH`、`DEBUG`)。これは技術的な要件ではなく慣習ですが、これに従うことで、通常は小文字で書かれるローカルなシェル変数と環境変数を区別しやすくなります。

```shell
TZ=Asia/Tokyo date  # prints the current time in Tokyo
echo $TZ  # this will be empty, since TZ was only set for the child command
```

あるいは、組み込みの `export` を使うこともできます。これは現在の環境自体を変更するため、以降のすべての子プロセスがその変数を継承します。

```shell
export DEBUG=1
# All programs from this point onwards will have DEBUG=1 in their environment
bash -c 'echo $DEBUG'
# prints 1
```

変数を削除するには、組み込みコマンドの `unset` を使います(例:`unset DEBUG`)。

> 環境変数もまた、シェルの慣習のひとつです。環境変数は、多くのプログラムの挙動を明示的にではなく暗黙的に変更するために使えます。たとえばシェルは、現在のユーザーのホームフォルダのパスを `$HOME` 環境変数に設定します。これにより、プログラムは `--home /home/alice` のような明示的な指定を要求する代わりに、この変数にアクセスして情報を得ることができます。もうひとつのよくある例は `$TZ` で、多くのプログラムが、指定されたタイムゾーンに従って日付や時刻をフォーマットするためにこれを使います。

## リターンコード

先ほど見たように、シェルプログラムの主要な出力はstdout/stderrストリームとファイルシステムへの副作用によって伝えられます。

デフォルトでは、シェルスクリプトは終了コードとしてゼロを返します。
慣習として、ゼロは「すべて問題なく完了した」ことを意味し、ゼロ以外は「何らかの問題が発生した」ことを意味します。
ゼロ以外の終了コードを返すには、シェル組み込みの `exit NUM` を使う必要があります。
最後に実行したコマンドのリターンコードには、特別な変数 `$?` でアクセスできます。

シェルには、AND演算とOR演算をそれぞれ行うブール演算子 `&&` と `||` があります。
通常のプログラミング言語で見られるものとは異なり、シェルのこれらの演算子はプログラムのリターンコードに対して作用します。
どちらも[短絡評価](https://en.wikipedia.org/wiki/Short-circuit_evaluation)を行う演算子です。
つまり、直前のコマンドの成功・失敗(リターンコードがゼロかどうかで判定されます)に基づいて、条件付きでコマンドを実行するために使えるということです。いくつか例を挙げます。

```shell
# echo will only run if grep succeeds (finds a match)
grep -q "pattern" file.txt && echo "Pattern found"

# echo will only run if grep fails (no match)
grep -q "pattern" file.txt || echo "Pattern not found"

# true is a shell program that always succeeds
true && echo "This will always print"

# and false is a shell program that always fails
false || echo "This will always print"
```

同じ原則が `if` 文と `while` 文にも当てはまります。どちらもリターンコードを使って判断を行います。

```shell
# if uses the return code of the condition command (0 = true, nonzero = false)
if grep -q "pattern" file.txt; then
    echo "Found"
fi

# while loops continue as long as the command returns 0
while read line; do
    echo "$line"
done < file.txt
```

## シグナル

実行中のプログラムを中断しなければならない場合があります。たとえば、コマンドの完了に時間がかかりすぎているときなどです。
プログラムを中断するもっとも簡単な方法は `Ctrl-C` を押すことで、これでたいていのコマンドは停止します。
しかし、これは実際にはどのような仕組みで動いているのでしょうか。そして、なぜプロセスの停止に失敗することがあるのでしょうか。

```console
$ sleep 100
^C
$
```

> ここで `^C` は、ターミナルで `Ctrl-C` を入力したときの表示です。

裏側では、次のようなことが起きています。

1. `Ctrl-C` を押す
2. シェルがこの特別な文字の組み合わせを識別する
3. シェルプロセスが `sleep` プロセスにSIGINTシグナルを送る
4. シグナルが `sleep` プロセスの実行を中断する

シグナルは特別な通信メカニズムです。
プロセスはシグナルを受け取ると実行を止め、シグナルを処理し、シグナルが伝えた情報に基づいて実行の流れを変えることもあります。このため、シグナルは「_ソフトウェア割り込み_」と呼ばれます。


今回のケースでは、`Ctrl-C` を入力するとシェルはプロセスに `SIGINT` シグナルを届けます。
以下は、`SIGINT` を捕捉して無視し、停止しなくなるPythonプログラムの最小例です。このプログラムを終了させるには、代わりに `Ctrl-\` を入力して `SIGQUIT` シグナルを使います。

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

このプログラムに `SIGINT` を2回送り、続けて `SIGQUIT` を送るとどうなるかを示します。なお、`^` はターミナルで `Ctrl` を入力したときの表示です。

```console
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

`SIGINT` と `SIGQUIT` はどちらも通常ターミナル関連の要求と結びついていますが、プロセスに正常終了を求めるためのより汎用的なシグナルとして `SIGTERM` シグナルがあります。
このシグナルを送るには [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) コマンドを、`kill -TERM <PID>` という構文で使います。

シグナルには、プロセスを終了させる以外の用途もあります。たとえば `SIGSTOP` はプロセスを一時停止します。ターミナルで `Ctrl-Z` を入力すると、シェルは `SIGTSTP` シグナルを送ります。これはTerminal Stopの略です(つまり `SIGSTOP` のターミナル版です)。

一時停止したジョブは、[`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) または [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html) を使って、それぞれフォアグラウンドまたはバックグラウンドで再開できます。

[`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) コマンドは、現在のターミナルセッションに関連付けられた未完了のジョブを一覧表示します。
これらのジョブは、pidを使って参照できます(pidは [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) で調べられます)。
より直感的には、パーセント記号にジョブ番号(`jobs` で表示されます)を続けてプロセスを参照することもできます。最後にバックグラウンドに送ったジョブを参照するには、特別なパラメータ `$!` が使えます。

もうひとつ知っておくべきことは、コマンドの末尾に `&` を付けるとそのコマンドがバックグラウンドで実行され、プロンプトがすぐに戻ってくるという点です。ただし、そのコマンドは依然としてシェルのSTDOUTを使うため、煩わしいことがあります(その場合はシェルのリダイレクトを使いましょう)。同様に、すでに実行中のプログラムをバックグラウンドに送るには、`Ctrl-Z` に続けて `bg` を実行します。


バックグラウンドのプロセスは依然としてターミナルの子プロセスであり、ターミナルを閉じると死んでしまうことに注意してください(このとき、さらに別のシグナルである `SIGHUP` が送られます)。
これを防ぐには、プログラムを [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html)(`SIGHUP` を無視するラッパー)付きで実行するか、すでに開始してしまったプロセスには `disown` を使います。
あるいは、次のセクションで見るように、ターミナルマルチプレクサを使うこともできます。

これらの概念のいくつかを実際に示すサンプルセッションを以下に載せます。

```
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ kill -SIGHUP %2   # nohup protects from SIGHUP

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000
```

`SIGKILL` は特別なシグナルです。プロセスはこれを捕捉できず、必ず即座に終了させられます。ただし、孤児となった子プロセスを残すなど、望ましくない副作用を引き起こす可能性があります。

これらのシグナルやその他のシグナルについては、[こちら](https://en.wikipedia.org/wiki/Signal_(IPC))を読むか、[`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) あるいは `kill -l` を実行することで詳しく学べます。

シェルスクリプト内では、組み込みの `trap` を使って、シグナルを受け取ったときにコマンドを実行できます。これはクリーンアップ処理に便利です。

```shell
#!/usr/bin/env bash
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT  # Run cleanup when script exits
trap cleanup SIGINT SIGTERM  # Also on Ctrl-C or kill
```
{% comment %}
### Users, Files and Permissions

Lastly, another way programs have to indirectly communicate with each other is using files.
For a program to be able to correctly read/write/delete files and folders, the file permissions must allow the operation.

Listing a specific file will give the following output

```console
$ ls -l notes.txt
-rw-r--r--  1 alice  users  12693 Jan 11 23:05 notes.txt
```

Here `ls` is listing what is the owner of the file, user `alice`, and the group `users`. Then the `rw-r--r--` are a shorthand notation for the permissions.
In this case, the file `notes.txt` has read/write permissions for the user alice `rw-`, and only read permissions for the group and the rest of users in the file system.

```console
$ ./script.sh
# permission denied
$ chmod +x script.sh
$ ls -l script.sh
-rwxr-xr-x  1 alice  users  3125 Jan 11 23:07 script.sh
$ ./script.sh
```

For a script to be executable, the executable rights must be set, hence why we had to use the `chmod` (change mode) program.
`chmod` syntax, while intuitive, is not obvious when first encountered.
If you, like me, prefer to learn by example, this is a good usecase of the `tldr` tool (note that you need to install it first).

```console
❯ tldr chmod
  Change the access permissions of a file or directory.
  More information: <https://www.gnu.org/software/coreutils/chmod>.

  Give the [u]ser who owns a file the right to e[x]ecute it:

      chmod u+x path/to/file

  Give the [u]ser rights to [r]ead and [w]rite to a file/directory:

      chmod u+rw path/to/file_or_directory

  Give [a]ll users rights to [r]ead and e[x]ecute:

      chmod a+rx path/to/file
```

Run `tldr chmod` to see more examples, including recursive operations and group permissions.

> Your shell might show you something like `command not found: tldr`. That is because it is a more modern tool and it is not pre-installed in most systems. A good reference for how to install tools is the [https://command-not-found.com](https://command-not-found.com) website. It contains instructions for a huge collection of CLI tools for popular OS distributions.

Each program is run as a specific user in the system. We can use the `whoami` command to find our user name and `id -u` to find our UID (user id) which is the integer value that the OS associates with the user.

When running `sudo command`, the `command` is run as the root user which can bypass most permissions in the system.
Try running `sudo whoami` and `sudo id -u` to see how the output changes (you might be prompted for your password).
To change the owner of a file or folder, we use the `chown` command.

You can learn more about UNIX file permissions [here](https://en.wikipedia.org/wiki/File-system_permissions#Traditional_Unix_permissions)

So far we've focused on your local machine, but many of these skills become even more valuable when working with remote servers.

{% endcomment %}

# リモートマシン

プログラマーが日々の仕事でリモートサーバーを扱うことは、ますます一般的になっています。この用途でもっともよく使われるツールがSSH(Secure Shell)で、リモートサーバーに接続し、もうおなじみとなったシェルインターフェースを提供してくれます。サーバーには次のようなコマンドで接続します。

```bash
ssh alice@server.mit.edu
```

ここでは、ユーザー `alice` としてサーバー `server.mit.edu` にsshで接続しようとしています。

`ssh` の見落とされがちな機能に、コマンドを非対話的に実行できるというものがあります。`ssh` はコマンドへのstdinの送信とstdoutの受信を正しく処理してくれるので、他のコマンドと組み合わせることができます。

```shell
# here ls runs in the remote, and wc runs locally
ssh alice@server ls | wc -l

# here both ls and wc run in the server
ssh alice@server 'ls | wc -l'

```

> SSHの代替として [Mosh](https://mosh.org/) をインストールしてみてください。切断、スリープへの出入り、ネットワークの切り替え、高遅延の回線などにうまく対処できます。

`ssh` にリモートサーバーでのコマンド実行を許可してもらうには、自分にその権限があることを証明する必要があります。
これはパスワードまたはssh鍵で行えます。
鍵ベースの認証は公開鍵暗号を利用して、秘密鍵そのものを明かすことなく、クライアントがその秘密鍵を所有していることをサーバーに証明します。
鍵ベースの認証はより便利かつ安全なので、こちらを優先すべきです。
秘密鍵(多くの場合 `~/.ssh/id_rsa`、最近では `~/.ssh/id_ed25519`)は実質的にあなたのパスワードなので、パスワードと同様に扱い、その内容を決して共有しないでください。

鍵ペアを生成するには [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html) を実行します。
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

SSH鍵を使ってGitHubへプッシュする設定をしたことがあるなら、おそらく[こちら](https://help.github.com/articles/connecting-to-github-with-ssh/)に書かれている手順をすでに済ませており、有効な鍵ペアを持っているはずです。パスフレーズが設定されているかを確認・検証するには、`ssh-keygen -y -f /path/to/key` を実行します。

サーバー側では、`ssh` は `.ssh/authorized_keys` を参照して、どのクライアントを受け入れるべきかを判断します。公開鍵をコピーするには次のようにします。

```bash
cat .ssh/id_ed25519.pub | ssh alice@remote 'cat >> ~/.ssh/authorized_keys'

# or more simply (if ssh-copy-id is available)

ssh-copy-id -i .ssh/id_ed25519 alice@remote
```

コマンドの実行だけでなく、sshが確立する接続は、サーバーとの間で安全にファイルを転送するためにも使えます。[`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) がもっとも伝統的なツールで、構文は `scp path/to/local_file remote_host:path/to/remote_file` です。[`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) は `scp` を改良したもので、ローカルとリモートで同一のファイルを検出し、それらを再度コピーするのを防ぎます。また、シンボリックリンクやパーミッションをより細かく制御でき、中断されたコピーを途中から再開できる `--partial` フラグのような追加機能もあります。`rsync` の構文は `scp` に似ています。

SSHクライアントの設定は `~/.ssh/config` にあり、ホストを宣言してそのデフォルト設定を定義できます。この設定ファイルは `ssh` だけでなく、`scp`、`rsync`、`mosh` など他のプログラムからも読み込まれます。

```bash
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Configs can also take wildcards
Host *.mit.edu
    User alice
```




# ターミナルマルチプレクサ

コマンドラインインターフェースを使っていると、複数のことを同時に実行したくなる場面がよくあります。
たとえば、エディタとプログラムを並べて動かしたい、といった場合です。
これは新しいターミナルウィンドウを開くことでも実現できますが、ターミナルマルチプレクサを使うほうがより柔軟な解決策です。

[`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) のようなターミナルマルチプレクサを使うと、ペインやタブでターミナルウィンドウを多重化し、複数のシェルセッションを効率よく操作できます。
さらに、ターミナルマルチプレクサは、現在のターミナルセッションをデタッチして、後で再アタッチすることも可能にします。
このため、ターミナルマルチプレクサはリモートマシンでの作業時に非常に便利で、`nohup` などの小技を使う必要がなくなります。

現在もっとも人気のあるターミナルマルチプレクサは [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) です。`tmux` は高度にカスタマイズ可能で、対応するキーバインドを使って複数のタブやペインを作成し、それらの間をすばやく移動できます。

`tmux` はそのキーバインドを覚えていることを前提としており、キーバインドはすべて `<C-b> x` という形をしています。これは、(1) `Ctrl+b` を押し、(2) `Ctrl+b` を離してから、(3) `x` を押す、という意味です。`tmux` のオブジェクトには次のような階層があります。
- **セッション** - セッションは、1つ以上のウィンドウを持つ独立したワークスペースです
    + `tmux` で新しいセッションを開始します。
    + `tmux new -s NAME` で名前を付けて開始します。
    + `tmux ls` で現在のセッションを一覧表示します
    + `tmux` 内で `<C-b> d` を入力すると、現在のセッションをデタッチします
    + `tmux a` で最後のセッションにアタッチします。`-t` フラグでどのセッションかを指定できます

- **ウィンドウ** - エディタやブラウザのタブに相当するもので、同じセッションの中の視覚的に分かれた部分です
    + `<C-b> c` 新しいウィンドウを作成します。閉じるには、`<C-d>` でシェルを終了すれば十分です
    + `<C-b> N` _N_ 番目のウィンドウへ移動します。ウィンドウには番号が付いています
    + `<C-b> p` 前のウィンドウへ移動します
    + `<C-b> n` 次のウィンドウへ移動します
    + `<C-b> ,` 現在のウィンドウの名前を変更します
    + `<C-b> w` 現在のウィンドウを一覧表示します

- **ペイン** - vimのスプリットのように、ペインを使うと同じ画面内に複数のシェルを表示できます。
    + `<C-b> "` 現在のペインを水平に分割します
    + `<C-b> %` 現在のペインを垂直に分割します
    + `<C-b> <direction>` 指定した _方向_ のペインへ移動します。ここでの方向とは矢印キーのことです。
    + `<C-b> z` 現在のペインのズームを切り替えます
    + `<C-b> [` スクロールバックを開始します。その後 `<space>` を押すと選択を開始し、`<enter>` でその選択をコピーできます。
    + `<C-b> <space>` ペインの配置を順に切り替えます。

> tmuxについてさらに学びたい場合は、[こちら](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)の手軽なチュートリアルと、[こちら](https://linuxcommand.org/lc3_adv_termmux.php)のより詳しい解説を読んでみてください。

tmuxとSSHが道具箱に揃ったら、次はどのマシンでも自分の環境を「我が家」のように感じられるようにしたくなるでしょう。そこで登場するのがシェルのカスタマイズです。

# シェルのカスタマイズ

数多くのコマンドラインプログラムが、「_ドットファイル_」と呼ばれるプレーンテキストのファイルで設定されます
(ファイル名が `.` で始まるためこう呼ばれます。たとえば `~/.vimrc` のように、
デフォルトでは `ls` のディレクトリ一覧に表示されず隠されます)。

> ドットファイルもまた、シェルの慣習のひとつです。先頭のドットは、一覧表示の際にファイルを「隠す」ためのものです(そう、これもまた慣習です)。

シェルは、このようなファイルで設定されるプログラムの一例です。起動時に、シェルは多くのファイルを読み込んで設定をロードします。
シェルの種類や、ログインセッションおよび/または対話的セッションとして起動しているかどうかによって、この一連の処理はかなり複雑になることがあります。
このトピックに関する優れた資料が[こちら](https://web.archive.org/web/20260329133158/https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)にあります。

`bash` の場合、`.bashrc` または `.bash_profile` を編集すれば、ほとんどのシステムでうまくいきます。
ドットファイルで設定できるツールの例をほかにも挙げると、

- `bash` - `~/.bashrc`、`~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` と `~/.vim` フォルダ
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

よくある設定変更のひとつが、シェルがプログラムを探す場所を追加することです。ソフトウェアをインストールする際に、このパターンに出会うことになるでしょう。

```shell
export PATH="$PATH:path/to/append"
```

ここでは、$PATH変数の値を「現在の値に新しいパスを加えたもの」に設定し、すべての子プロセスにこの新しいPATHの値を継承させるようシェルに指示しています。
これにより、子プロセスは `path/to/append` 以下にあるプログラムを見つけられるようになります。


シェルのカスタマイズは、多くの場合、新しいコマンドラインツールのインストールを伴います。これを簡単にしてくれるのがパッケージマネージャです。パッケージマネージャは、ソフトウェアのダウンロード、インストール、更新を担ってくれます。オペレーティングシステムごとにパッケージマネージャは異なります。macOSは [Homebrew](https://brew.sh/)、Ubuntu/Debianは `apt`、Fedoraは `dnf`、Archは `pacman` を使います。パッケージマネージャについては、コードの配布に関する講義でより詳しく扱います。

macOSでHomebrewを使って、便利なツールを2つインストールする方法を示します。

```shell
# ripgrep: a faster grep with better defaults
brew install ripgrep

# fd: a faster, user-friendly find
brew install fd
```

これらをインストールすれば、`grep` の代わりに `rg` を、`find` の代わりに `fd` を使えるようになります。

> **`curl | bash` に関する警告**: `curl -fsSL https://example.com/install.sh | bash` のようなインストール手順をよく見かけるでしょう。このパターンはスクリプトをダウンロードして即座に実行するもので、便利ではありますがリスクがあります。中身を確認していないコードを実行することになるからです。より安全なやり方は、まずダウンロードし、確認してから実行することです。
> ```shell
> curl -fsSL https://example.com/install.sh -o install.sh
> less install.sh  # review the script
> bash install.sh
> ```
> 一部のインストーラは、少しだけ安全なバリアントを使っています。`/bin/bash -c "$(curl -fsSL https://url)"` という形式で、少なくとも現在使用中のシェルではなくbashがスクリプトを解釈することが保証されます。

インストールされていないコマンドを実行しようとすると、シェルは `command not found` と表示します。ウェブサイト [command-not-found.com](https://command-not-found.com) は便利なリソースで、任意のコマンドを検索して、さまざまなパッケージマネージャやディストリビューションでのインストール方法を調べられます。

もうひとつの便利なツールが [`tldr`](https://tldr.sh/) です。これは、例を中心とした簡潔なmanページを提供してくれます。長いドキュメントを読み通す代わりに、よくある使い方のパターンをすばやく確認できます。

```console
$ tldr fd
  An alternative to find.
  Aims to be faster and easier to use than find.

  Recursively find files matching a pattern in the current directory:
      fd "pattern"

  Find files that begin with "foo":
      fd "^foo"

  Find files with a specific extension:
      fd --extension txt
```

まったく新しいプログラムが必要なわけではなく、特定のフラグを付けた既存コマンドのショートカットがほしいだけ、ということもあります。そこで登場するのがエイリアスです。

シェル組み込みの `alias` を使って、独自のコマンドエイリアスを作ることもできます。
シェルエイリアスとは別のコマンドの短縮形で、シェルが式を評価する前に自動的に置き換えてくれます。
たとえば、bashのエイリアスは次のような構造をしています。

```bash
alias alias_name="command_to_alias arg1 arg2"
```

> 等号 `=` の前後にスペースがないことに注意してください。[`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) は単一の引数を取るシェルコマンドだからです。

エイリアスには便利な使い方がたくさんあります。

```bash
# Make shorthands for common flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gs="git status"
alias gc="git commit"

# Save you from mistyping
alias sl=ls

# Overwrite existing commands for better defaults
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or disable an alias altogether with unalias
unalias la

# To get an alias definition just call it with alias
alias ll
# Will print ll='ls -lh'
```

エイリアスには制約もあります。コマンドの途中に引数を取ることはできません。より複雑な挙動が必要な場合は、代わりにシェル関数を使うべきです。

ほとんどのシェルは、履歴の逆方向検索のための `Ctrl-R` をサポートしています。`Ctrl-R` を押して入力を始めると、過去のコマンドを検索できます。先ほどファジーファインダーとして `fzf` を紹介しましたが、fzfのシェル統合を設定すると、`Ctrl-R` が履歴全体に対するインタラクティブなあいまい検索になり、デフォルトよりもはるかに強力になります。

ドットファイルはどのように整理すべきでしょうか?専用のフォルダにまとめてバージョン管理下に置き、スクリプトを使って所定の場所に**シンボリックリンク**を張るのがよいでしょう。これには次のような利点があります。

- **簡単なインストール**: 新しいマシンにログインしたとき、自分のカスタマイズの適用が1分で済みます。
- **ポータビリティ**: どこでもツールが同じように動作します。
- **同期**: どこにいてもドットファイルを更新でき、すべてを同期した状態に保てます。
- **変更の追跡**: ドットファイルはおそらくプログラマー人生を通してメンテナンスし続けることになるので、長寿命のプロジェクトにはバージョン履歴があると心強いものです。

ドットファイルには何を書けばよいのでしょうか?
ツールの設定については、オンラインのドキュメントや
[manページ](https://en.wikipedia.org/wiki/Man_page)を読んで学べます。もうひとつの優れた方法は、
特定のプログラムについてのブログ記事をインターネットで検索することです。著者が
自分のお気に入りのカスタマイズを紹介してくれています。さらに別の方法として、他の人のドットファイルを
覗いてみるという手もあります。GitHubには膨大な数の
[dotfilesリポジトリ](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)
があります --- もっとも人気のあるものは
[こちら](https://github.com/mathiasbynens/dotfiles)です(ただし、設定をやみくもに
コピーするのはおすすめしません)。
このトピックに関するもうひとつの良い資料が[こちら](https://dotfiles.github.io/)にあります。

本講座の講師は全員、自分のドットファイルをGitHubで公開しています: [Anish](https://github.com/anishathalye/dotfiles)、
[Jon](https://github.com/jonhoo/configs)、
[Jose](https://github.com/jjgo/dotfiles)。

**フレームワークやプラグイン**もシェルを改善してくれます。人気のある汎用フレームワークには [prezto](https://github.com/sorin-ionescu/prezto) や [oh-my-zsh](https://ohmyz.sh/) があり、特定の機能に特化した小さなプラグインには次のようなものがあります。

- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - 入力中のコマンドを有効/無効に応じて色付けします
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) - 入力中に履歴からコマンドを提案します
- [zsh-completions](https://github.com/zsh-users/zsh-completions) - 補完定義を追加します
- [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) - fish風の履歴検索を提供します
- [powerlevel10k](https://github.com/romkatv/powerlevel10k) - 高速でカスタマイズ可能なプロンプトテーマです

[fish](https://fishshell.com/) のようなシェルは、これらの機能の多くをデフォルトで備えています。

> こうした機能を得るために、oh-my-zshのような巨大なフレームワークは必要ありません。個別のプラグインをインストールするほうが、たいてい高速で、より細かくコントロールできます。大きなフレームワークはシェルの起動時間を大幅に遅くすることがあるため、実際に使うものだけをインストールすることを検討してください。


# シェルにおけるAI

シェルにAIツールを組み込む方法はたくさんあります。統合の度合いが異なる例をいくつか紹介します。

**コマンド生成**: [`simonw/llm`](https://github.com/simonw/llm) のようなツールは、自然言語の説明からシェルコマンドを生成するのに役立ちます。

```console
$ llm cmd "find all python files modified in the last week"
find . -name "*.py" -mtime -7
```

**パイプラインへの統合**: LLMはシェルのパイプラインに組み込んで、データの処理や変換に使えます。正規表現では骨が折れるような、形式の不揃いなデータから情報を抽出する必要があるときに特に便利です。

```console
$ cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor
$ INSTRUCTIONS="Extract just the username from each line, one per line, nothing else"
$ llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor
```

変数にスペースが含まれるため `"$INSTRUCTIONS"` のように引用符で囲んでいること、そして `< users.txt` でファイルの内容をstdinにリダイレクトしていることに注目してください。

**AIシェル**: [Claude Code](https://docs.anthropic.com/en/docs/claude-code) のようなツールは、英語のコマンドを受け取り、それをシェル操作やファイル編集、さらに複雑なマルチステップのタスクへと翻訳するメタシェルとして機能します。

# ターミナルエミュレータ

シェルのカスタマイズと合わせて、**ターミナルエミュレータ**の選択とその設定にも時間をかける価値があります。
ターミナルエミュレータとは、シェルが動作するテキストベースのインターフェースを提供するGUIプログラムです。
世の中には数多くのターミナルエミュレータがあります。

ターミナルの中で何百時間、何千時間と過ごすことになるかもしれないのですから、その設定を調べておくことは十分に元が取れます。ターミナルで調整したくなるであろう項目には、次のようなものがあります。

- フォントの選択
- カラースキーム
- キーボードショートカット
- タブ/ペインのサポート
- スクロールバックの設定
- パフォーマンス([Alacritty](https://github.com/alacritty/alacritty) や [Ghostty](https://ghostty.org/) のような比較的新しいターミナルは、GPUアクセラレーションを提供しています)



# 演習

## 引数とグロブ

1. `cmd --flag -- --notaflag` のようなコマンドを見かけることがあります。`--` は特別な引数で、プログラムにフラグの解析をやめるよう伝えます。`--` 以降のものはすべて位置引数として扱われます。これはなぜ便利なのでしょうか?`touch -- -myfile` を実行してみて、その後 `--` を使わずにそのファイルを削除してみてください。

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

1. プロセス置換 `<(command)` を使うと、コマンドの出力をあたかもファイルであるかのように扱えます。プロセス置換を使った `diff` で、`printenv` と `export` の出力を比較してください。なぜ両者は異なるのでしょうか?(ヒント: `diff <(printenv | sort) <(export | sort)` を試してみてください)。

## 環境変数

1. 次のように動作するbash関数 `marco` と `polo` を書いてください。`marco` を実行すると、そのときのカレントワーキングディレクトリが何らかの形で保存されます。その後 `polo` を実行すると、どのディレクトリにいようとも、`marco` を実行したディレクトリに `cd` で戻ります。デバッグしやすいように、コードは `marco.sh` というファイルに書き、`source marco.sh` を実行することで定義をシェルに(再)読み込みできます。

{% comment %}
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}
{% endcomment %}

## リターンコード

1. めったに失敗しないコマンドがあるとしましょう。これをデバッグするには失敗時の出力を捕捉する必要がありますが、失敗するまで実行し続けるのは時間がかかります。以下のスクリプトを失敗するまで実行し続け、その標準出力と標準エラー出力をファイルに記録して、最後にすべてを表示するbashスクリプトを書いてください。スクリプトが失敗するまでに何回実行されたかも報告できればボーナスポイントです。

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ $n -eq 42 ]]; then
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

## シグナルとジョブ制御

1. ターミナルで `sleep 10000` ジョブを開始し、`Ctrl-Z` でバックグラウンドに移して、`bg` で実行を再開してください。次に [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) を使ってそのpidを見つけ、[`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html) を使って、pidを一度も自分で入力することなくこのジョブをkillしてください。(ヒント: `-lf` フラグを使いましょう)。

1. あるプロセスが完了するまで、別のプロセスを開始したくないとします。どうすればよいでしょうか?この演習では、待たされる側の起点となるプロセスは常に `sleep 60 &` とします。ひとつの方法は [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) コマンドを使うことです。sleepコマンドを起動し、そのバックグラウンドプロセスが終了するまで `ls` を待たせてみてください。

    しかし、この戦略は、別のbashセッションから始めた場合には失敗します。`wait` は子プロセスに対してしか機能しないからです。本文で触れなかった機能のひとつに、`kill` コマンドの終了ステータスは成功時にゼロ、それ以外は非ゼロになる、というものがあります。`kill -0` はシグナルを送信しませんが、プロセスが存在しない場合に非ゼロの終了ステータスを返します。pidを受け取り、そのプロセスが完了するまで待つ `pidwait` というbash関数を書いてください。CPUを無駄に消費しないように `sleep` を使うこと。

## ファイルとパーミッション

1. (上級)ディレクトリ内でもっとも最近変更されたファイルを再帰的に見つけるコマンドまたはスクリプトを書いてください。より一般化して、すべてのファイルを新しい順に一覧表示できますか?

## ターミナルマルチプレクサ

1. こちらの `tmux` [チュートリアル](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)に従って学び、その後[こちらの手順](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)に従って基本的なカスタマイズの方法を学んでください。

## エイリアスとドットファイル

1. `cd` を打ち間違えたときのために、`dc` を `cd` に解決するエイリアスを作ってください。

1. `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` を実行して、もっともよく使っているコマンドのトップ10を調べ、それらに短いエイリアスを作ることを検討してください。注: これはBashで動作します。ZSHを使っている場合は、`history` の代わりに `history 1` を使ってください。

1. ドットファイル用のフォルダを作り、バージョン管理をセットアップしてください。

1. 少なくとも1つのプログラム(たとえばシェル)の設定を、何らかのカスタマイズ付きで追加してください(手始めとしては、`$PS1` を設定してシェルのプロンプトをカスタマイズするような簡単なものでかまいません)。

1. 新しいマシンにドットファイルをすばやく(かつ手作業なしで)インストールする方法をセットアップしてください。各ファイルに対して `ln -s` を呼ぶだけの単純なシェルスクリプトでもよいですし、[専用のユーティリティ](https://dotfiles.github.io/utilities/)を使ってもかまいません。

1. まっさらな仮想マシンで、インストールスクリプトをテストしてください。

1. 現在使っているすべてのツールの設定を、ドットファイルのリポジトリに移行してください。

1. 自分のドットファイルをGitHubで公開してください。

## リモートマシン (SSH)

これらの演習のために、Linuxの仮想マシンをインストールしてください(既存のものがあればそれを使ってもかまいません)。仮想マシンに馴染みがない場合は、[こちら](https://hibbard.eu/install-ubuntu-virtual-box/)のインストール手順のチュートリアルを確認してください。

1. `~/.ssh/` を開いて、SSH鍵のペアがあるかどうかを確認してください。なければ、`ssh-keygen -a 100 -t ed25519` で生成してください。パスワードを設定し、`ssh-agent` を使うことをおすすめします。詳しくは[こちら](https://www.ssh.com/ssh/agent)。

1. `.ssh/config` を編集して、次のようなエントリを追加してください。

    ```bash
    Host vm
        User username_goes_here
        HostName ip_goes_here
        IdentityFile ~/.ssh/id_ed25519
        LocalForward 9999 localhost:8888
    ```

1. `ssh-copy-id vm` を使って、ssh鍵をサーバーにコピーしてください。

1. VM内で `python -m http.server 8888` を実行してウェブサーバーを起動してください。手元のマシンで `http://localhost:9999` にアクセスして、VMのウェブサーバーに接続してください。

1. `sudo vim /etc/ssh/sshd_config` でSSHサーバーの設定を編集し、`PasswordAuthentication` の値を変更してパスワード認証を無効化してください。さらに `PermitRootLogin` の値を変更してrootログインを無効化してください。`sudo service sshd restart` で `ssh` サービスを再起動し、もう一度sshで接続してみてください。

1. (チャレンジ)VMに [`mosh`](https://mosh.org/) をインストールして接続を確立してください。その後、サーバー/VMのネットワークアダプタを切断してみてください。moshは正しく復帰できるでしょうか?

1. (チャレンジ)`ssh` の `-N` フラグと `-f` フラグが何をするのか調べて、バックグラウンドでポートフォワーディングを行うコマンドを考えてください。
