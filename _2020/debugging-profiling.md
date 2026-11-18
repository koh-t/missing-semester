---
layout: lecture
title: "デバッグとプロファイリング"
description: >
  ロギング、デバッガ、静的解析を使ったプログラムのデバッグ方法と、パフォーマンスのためのコードのプロファイリング方法を学びます。
thumbnail: /static/assets/thumbnails/2020/lec7.png
date: 2020-01-23
ready: true
video:
  aspect: 56.25
  id: l812pUnKxME
---

プログラミングにおける黄金律は、コードはあなたが期待した通りには動かず、あなたが命じた通りに動く、というものです。そのギャップを埋めるのは、時にかなり難しい仕事になります。この講義では、バグを含むコードやリソースを大量に消費するコードに対処するための有用な技術、すなわちデバッグとプロファイリングを扱います。

# デバッグ

## printfデバッグとロギング

「最も効果的なデバッグツールは、今でも注意深い思考と、要所要所に配置されたprint文である」 — Brian Kernighan, _Unix for Beginners_

プログラムをデバッグする最初のアプローチは、問題を検出したあたりにprint文を追加し、問題の原因を理解するのに十分な情報が得られるまで繰り返すことです。

2つ目のアプローチは、その場しのぎのprint文の代わりに、プログラムでロギングを使うことです。ロギングは、いくつかの理由で通常のprint文より優れています。

- 標準出力の代わりに、ファイルやソケット、さらにはリモートサーバーにもログを出力できます。
- ロギングは深刻度レベル(INFO、DEBUG、WARN、ERRORなど)をサポートしており、それに応じて出力をフィルタリングできます。
- 新たに発生した問題であっても、何がおかしいのかを突き止めるのに十分な情報がログに含まれている可能性がかなりあります。

メッセージをログに出力するコードの例は[こちら]({{ site.baseurl }}/static/files/logger.py)です。

```bash
$ python logger.py
# Raw output as with just prints
$ python logger.py log
# Log formatted output
$ python logger.py log ERROR
# Print only ERROR levels and above
$ python logger.py color
# Color formatted output
```

ログを読みやすくするための私のお気に入りのコツのひとつは、色分けすることです。ここまでで、ターミナルが読みやすさのために色を使っていることには、おそらく気づいているでしょう。しかし、どのようにして実現しているのでしょうか? `ls` や `grep` のようなプログラムは、[ANSIエスケープコード](https://en.wikipedia.org/wiki/ANSI_escape_code)を使っています。これは、出力の色を変えるようシェルに指示するための特殊な文字の並びです。たとえば、`echo -e "\e[38;2;255;0;0mThis is red\e[0m"` を実行すると、ターミナルが[トゥルーカラー](https://github.com/termstandard/colors#truecolor-support-in-output-devices)をサポートしている限り、`This is red` というメッセージが赤色で表示されます。ターミナルがこれをサポートしていない場合(たとえばmacOSのTerminal.app)は、より広くサポートされている16色のエスケープコードを使えます。たとえば `echo -e "\e[31;1mThis is red\e[0m"` のようにします。

次のスクリプトは、多数のRGBカラーをターミナルに表示する方法を示しています(これも、トゥルーカラーをサポートしている場合に限ります)。

```bash
#!/usr/bin/env bash
for R in $(seq 0 20 255); do
    for G in $(seq 0 20 255); do
        for B in $(seq 0 20 255); do
            printf "\e[38;2;${R};${G};${B}m█\e[0m";
        done
    done
done
```

## サードパーティのログ

より大きなソフトウェアシステムを構築するようになると、別のプログラムとして動作する依存先にほぼ確実に出くわします。Webサーバー、データベース、メッセージブローカーは、この種の依存先のよくある例です。こうしたシステムとやり取りする際には、クライアント側のエラーメッセージだけでは不十分なことがあるため、それらのログを読む必要がしばしばあります。

幸い、ほとんどのプログラムは自分のログをシステムのどこかに書き出しています。UNIXシステムでは、プログラムがログを `/var/log` 以下に書き出すのが一般的です。たとえば、Webサーバーの[NGINX](https://www.nginx.com/)は、ログを `/var/log/nginx` 以下に置きます。最近では、システムは**システムログ**を使うようになっており、すべてのログメッセージがますますここに集まるようになってきています。すべてではないもののほとんどのLinuxシステムは `systemd` を使っています。これはシステムデーモンで、どのサービスが有効化され実行されているかなど、システムの多くのものを制御しています。`systemd` はログを専用のフォーマットで `/var/log/journal` 以下に置き、[`journalctl`](https://www.man7.org/linux/man-pages/man1/journalctl.1.html) コマンドを使ってそのメッセージを表示できます。同様に、macOSにも依然として `/var/log/system.log` がありますが、システムログを使うツールが増えており、これは [`log show`](https://www.manpagez.com/man/1/log/) で表示できます。ほとんどのUNIXシステムでは、[`dmesg`](https://www.man7.org/linux/man-pages/man1/dmesg.1.html) コマンドを使ってカーネルログにアクセスすることもできます。

システムログにログを出力するには、シェルプログラムの [`logger`](https://www.man7.org/linux/man-pages/man1/logger.1.html) が使えます。以下は、`logger` の使用例と、そのエントリがシステムログに入ったことを確認する方法です。さらに、ほとんどのプログラミング言語には、システムログに出力するためのバインディングがあります。

```bash
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

データラングリングの講義で見たように、ログはかなり冗長になることがあり、欲しい情報を得るにはある程度の加工とフィルタリングが必要です。`journalctl` や `log show` の出力を大量にフィルタリングしていることに気づいたら、それらのフラグを使うことを検討してみてください。フラグを使えば、出力に対する最初のフィルタリングを行えます。また、[`lnav`](https://lnav.org/) のように、ログファイルの表示とナビゲーションを改善してくれるツールもあります。

## デバッガ

printfデバッグでは不十分なときは、デバッガを使うべきです。デバッガは、プログラムの実行に介入できるプログラムで、次のようなことができます。

- 特定の行に到達したときにプログラムの実行を停止する。
- プログラムを一度に1命令ずつステップ実行する。
- プログラムがクラッシュした後に変数の値を調べる。
- 特定の条件が満たされたときに条件付きで実行を停止する。
- その他多くの高度な機能

多くのプログラミング言語には、何らかの形のデバッガが付属しています。Pythonの場合は、Pythonデバッガの [`pdb`](https://docs.python.org/3/library/pdb.html) です。

`pdb` がサポートするコマンドのいくつかを簡単に説明します。

- **l**(ist) - 現在の行の前後11行を表示するか、前回の表示の続きを表示します。
- **s**(tep) - 現在の行を実行し、最初に停止可能な場所で停止します。
- **n**(ext) - 現在の関数の次の行に到達するか、関数が戻るまで実行を続けます。
- **b**(reak) - ブレークポイントを設定します(場所は与えた引数によって決まります)。
- **p**(rint) - 現在のコンテキストで式を評価し、その値を表示します。代わりに [`pprint`](https://docs.python.org/3/library/pprint.html) を使って表示する**pp**もあります。
- **r**(eturn) - 現在の関数が戻るまで実行を続けます。
- **q**(uit) - デバッガを終了します。

`pdb` を使って、次のバグのあるPythonコードを修正する例を見ていきましょう(講義動画を参照してください)。

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(n):
            if arr[j] > arr[j+1]:
                arr[j] = arr[j+1]
                arr[j+1] = arr[j]
    return arr

print(bubble_sort([4, 2, 1, 8, 7, 6]))
```


Pythonはインタプリタ言語なので、`pdb` シェルを使ってコマンドや命令を実行できることに注意してください。[`ipdb`](https://pypi.org/project/ipdb/) は改良版の `pdb` で、[`IPython`](https://ipython.org) のREPLを使うことで、`pdb` モジュールと同じインターフェースを保ちながら、タブ補完、シンタックスハイライト、より分かりやすいトレースバック、より優れたイントロスペクションを可能にします。

より低レベルなプログラミングでは、おそらく [`gdb`](https://www.gnu.org/software/gdb/)(およびその使い勝手を向上させる改造版である [`pwndbg`](https://github.com/pwndbg/pwndbg))や [`lldb`](https://lldb.llvm.org/) を調べてみるとよいでしょう。これらはC系言語のデバッグ向けに最適化されていますが、ほぼあらゆるプロセスを調査して、レジスタ、スタック、プログラムカウンタなど、現在のマシン状態を取得できます。


## 専用ツール

デバッグしようとしているものがブラックボックスのバイナリであっても、役に立つツールがあります。プログラムは、カーネルにしかできない操作を行う必要があるときは必ず[システムコール](https://en.wikipedia.org/wiki/System_call)を使います。プログラムが発行するシステムコールをトレースできるコマンドがあります。Linuxには [`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) があり、macOSとBSDには [`dtrace`](https://dtrace.org/about/) があります。`dtrace` は独自の `D` 言語を使うため扱いが難しいことがありますが、[`dtruss`](https://www.manpagez.com/man/1/dtruss/) というラッパーがあり、`strace` により近いインターフェースを提供してくれます(詳細は[こちら](https://8thlight.com/blog/colin-jones/2015/11/06/dtrace-even-better-than-strace-for-osx.html))。

以下は、`strace` または `dtruss` を使って、`ls` の実行における [`stat`](https://www.man7.org/linux/man-pages/man2/stat.2.html) システムコールのトレースを表示する例です。`strace` をさらに深く知るには、[この記事](https://blogs.oracle.com/linux/strace-the-sysadmins-microscope-v2)と[このzine](https://jvns.ca/strace-zine-unfolded.pdf)が良い読み物です。

```bash
# On Linux
sudo strace -e lstat ls -l > /dev/null
# On macOS
sudo dtruss -t lstat64_extended ls -l > /dev/null
```

状況によっては、プログラムの問題を突き止めるためにネットワークパケットを調べる必要があるかもしれません。[`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) や[Wireshark](https://www.wireshark.org/)のようなツールはネットワークパケットアナライザで、ネットワークパケットの内容を読み取り、さまざまな条件でフィルタリングできます。

Web開発では、Chrome/Firefoxの開発者ツールがとても便利です。次のような多数のツールを備えています。
- ソースコード - 任意のWebサイトのHTML/CSS/JSのソースコードを調べます。
- HTML、CSS、JSのライブ編集 - Webサイトの内容、スタイル、挙動を変更してテストします(Webサイトのスクリーンショットが有効な証拠にならないことを、自分で確かめられます)。
- JavaScriptシェル - JSのREPLでコマンドを実行します。
- ネットワーク - リクエストのタイムラインを分析します。
- ストレージ - Cookieやアプリケーションのローカルストレージを調べます。

## 静的解析

問題によっては、コードをまったく実行する必要がありません。たとえば、コードを注意深く眺めるだけで、ループ変数が既存の変数や関数名をシャドーイングしていることや、プログラムが変数を定義する前に読み出していることに気づけるでしょう。ここで役立つのが[静的解析](https://en.wikipedia.org/wiki/Static_program_analysis)ツールです。静的解析プログラムはソースコードを入力として受け取り、コーディング規則を使って解析することで、その正しさを推論します。

次のPythonのコード片には、いくつかの間違いがあります。まず、ループ変数 `foo` が、それより前にある関数 `foo` の定義をシャドーイングしています。また、最後の行で `bar` の代わりに `baz` と書いてしまったので、プログラムは(1分かかる)`sleep` の呼び出しが終わった後にクラッシュします。

```python
import time

def foo():
    return 42

for foo in range(5):
    print(foo)
bar = 1
bar *= 0.2
time.sleep(60)
print(baz)
```

静的解析ツールは、この種の問題を特定できます。このコードに対して [`pyflakes`](https://pypi.org/project/pyflakes) を実行すると、両方のバグに関するエラーが得られます。[`mypy`](https://mypy-lang.org/) は、型チェックの問題を検出できるもうひとつのツールです。ここでは `mypy` が、`bar` が最初は `int` であり、その後 `float` にキャストされていると警告してくれます。繰り返しになりますが、これらの問題はすべて、コードを実行することなく検出されたことに注目してください。

```bash
$ pyflakes foobar.py
foobar.py:6: redefinition of unused 'foo' from line 3
foobar.py:11: undefined name 'baz'

$ mypy foobar.py
foobar.py:6: error: Incompatible types in assignment (expression has type "int", variable has type "Callable[[], Any]")
foobar.py:9: error: Incompatible types in assignment (expression has type "float", variable has type "int")
foobar.py:11: error: Name 'baz' is not defined
Found 3 errors in 1 file (checked 1 source file)
```

シェルツールの講義では [`shellcheck`](https://www.shellcheck.net/) を扱いました。これはシェルスクリプト向けの同様のツールです。

ほとんどのエディタやIDEは、これらのツールの出力をエディタ内に表示し、警告やエラーの場所をハイライトすることをサポートしています。これはよく**コードリンティング**と呼ばれ、スタイル違反や安全でない構文など、他の種類の問題を表示するためにも使えます。

vimでは、[`ale`](https://vimawesome.com/plugin/ale) や [`syntastic`](https://vimawesome.com/plugin/syntastic) といったプラグインでこれが可能になります。Pythonでは、[`pylint`](https://github.com/PyCQA/pylint) や [`pep8`](https://pypi.org/project/pep8/) がスタイルリンタの例で、[`bandit`](https://pypi.org/project/bandit/) はよくあるセキュリティ上の問題を見つけるために設計されたツールです。その他の言語については、[Awesome Static Analysis](https://github.com/mre/awesome-static-analysis)(_Writing_ のセクションを見てみるとよいでしょう)のように、便利な静的解析ツールの包括的なリストを有志がまとめており、リンタについては[Awesome Linters](https://github.com/caramelomartins/awesome-linters)があります。

スタイルのリンティングを補完するツールが、Python向けの [`black`](https://github.com/psf/black)、Go向けの `gofmt`、Rust向けの `rustfmt`、JavaScript・HTML・CSS向けの [`prettier`](https://prettier.io/) といったコードフォーマッタです。これらのツールは、対象のプログラミング言語の一般的なスタイルのパターンに沿うよう、コードを自動整形します。自分のコードのスタイルを自分で決められなくなるのは気が進まないかもしれませんが、コードのフォーマットを標準化すれば、他の人があなたのコードを読みやすくなり、あなた自身も他の人の(スタイルが標準化された)コードを読むのがうまくなります。

# プロファイリング

たとえコードが機能的に期待通りに動作していても、その過程でCPUやメモリを食い尽くしてしまうなら、それでは不十分かもしれません。アルゴリズムの授業ではビッグ _O_ 記法は教えても、プログラムのホットスポットの見つけ方は教えないことが多いものです。[早すぎる最適化は諸悪の根源](https://wiki.c2.com/?PrematureOptimization)なので、プロファイラと監視ツールについて学ぶべきです。これらは、プログラムのどの部分が時間やリソースの大半を消費しているのかを理解する助けになり、その部分の最適化に集中できるようになります。

## 時間計測 {#timing}

デバッグの場合と同様に、多くの場面では、コードの2点間でかかった時間を表示するだけで十分なことがあります。以下は、Pythonで [`time`](https://docs.python.org/3/library/time.html) モジュールを使った例です。

```python
import time, random
n = random.randint(1, 10) * 100

# Get current time
start = time.time()

# Do some work
print("Sleeping for {} ms".format(n))
time.sleep(n/1000)

# Compute time between start and now
print(time.time() - start)

# Output
# Sleeping for 500 ms
# 0.5713930130004883
```

しかし、実時間(wall clock time)は誤解を招くことがあります。コンピュータが同時に他のプロセスを実行していたり、イベントの発生を待っていたりするかもしれないからです。ツールは _Real_、_User_、_Sys_ の時間を区別するのが一般的です。一般に、_User_ + _Sys_ が、プロセスが実際にCPU上で費やした時間を示します(より詳しい説明は[こちら](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1))。

- _Real_ - プログラムの開始から終了までの実時間。他のプロセスに使われた時間や、ブロックされていた時間(たとえばI/Oやネットワークの待ち時間)を含みます
- _User_ - CPU上でユーザーコードの実行に費やされた時間
- _Sys_ - CPU上でカーネルコードの実行に費やされた時間

たとえば、HTTPリクエストを行うコマンドの先頭に [`time`](https://www.man7.org/linux/man-pages/man1/time.1.html) を付けて実行してみてください。接続が遅いと、以下のような出力になるかもしれません。ここでは、リクエストの完了に2秒以上かかりましたが、プロセスが使ったのはユーザーCPU時間15ミリ秒とカーネルCPU時間12ミリ秒だけでした。

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real    0m2.561s
user    0m0.015s
sys     0m0.012s
```

## プロファイラ

### CPU

人々が _プロファイラ_ と言うとき、たいていは実際には _CPUプロファイラ_ のことを指しており、これが最も一般的です。CPUプロファイラには主に2種類あります。_トレーシング_ プロファイラと _サンプリング_ プロファイラです。トレーシングプロファイラはプログラムが行うすべての関数呼び出しを記録するのに対し、サンプリングプロファイラはプログラムを定期的に(一般的には1ミリ秒ごとに)調べて、プログラムのスタックを記録します。プロファイラはこれらの記録を使って、プログラムが何に最も時間を費やしたかについての集計統計を提示します。このトピックについてさらに詳しく知りたい場合は、[こちら](https://jvns.ca/blog/2017/12/17/how-do-ruby---python-profilers-work-)が良い入門記事です。

ほとんどのプログラミング言語には、コードの分析に使える何らかのコマンドラインプロファイラがあります。これらは本格的なIDEと統合されていることも多いのですが、この講義ではコマンドラインツールそのものに焦点を当てます。

Pythonでは、`cProfile` モジュールを使って関数呼び出しごとの時間をプロファイルできます。以下は、Pythonで初歩的なgrepを実装した簡単な例です。

```python
#!/usr/bin/env python

import sys, re

def grep(pattern, file):
    with open(file, 'r') as f:
        print(file)
        for i, line in enumerate(f.readlines()):
            pattern = re.compile(pattern)
            match = pattern.search(line)
            if match is not None:
                print("{}: {}".format(i, line), end="")

if __name__ == '__main__':
    times = int(sys.argv[1])
    pattern = sys.argv[2]
    for i in range(times):
        for file in sys.argv[3:]:
            grep(pattern, file)
```

このコードは、次のコマンドでプロファイルできます。出力を分析すると、IOが時間の大半を占めていること、そして正規表現のコンパイルにもかなりの時間がかかっていることが分かります。正規表現は一度だけコンパイルすれば済むので、forの外に出すことができます。

```
$ python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py

[omitted program output]

 ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     8000    0.266    0.000    0.292    0.000 {built-in method io.open}
     8000    0.153    0.000    0.894    0.000 grep.py:5(grep)
    17000    0.101    0.000    0.101    0.000 {built-in method builtins.print}
     8000    0.100    0.000    0.129    0.000 {method 'readlines' of '_io._IOBase' objects}
    93000    0.097    0.000    0.111    0.000 re.py:286(_compile)
    93000    0.069    0.000    0.069    0.000 {method 'search' of '_sre.SRE_Pattern' objects}
    93000    0.030    0.000    0.141    0.000 re.py:231(compile)
    17000    0.019    0.000    0.029    0.000 codecs.py:318(decode)
        1    0.017    0.017    0.911    0.911 grep.py:3(<module>)

[omitted lines]
```


Pythonの `cProfile` プロファイラ(さらに言えば多くのプロファイラ)の注意点は、関数呼び出しごとの時間を表示することです。これはあっという間に直感的でなくなります。特に、コードでサードパーティのライブラリを使っている場合は、内部の関数呼び出しも計上されるためです。プロファイリング情報をより直感的に表示する方法は、コードの行ごとにかかった時間を含めることで、これを行うのが _ラインプロファイラ_ です。

たとえば、次のPythonコードは、この授業のWebサイトにリクエストを送り、レスポンスをパースしてページ内のすべてのURLを取得します。

```python
#!/usr/bin/env python
import requests
from bs4 import BeautifulSoup

# This is a decorator that tells line_profiler
# that we want to analyze this function
@profile
def get_urls():
    response = requests.get('https://missing.csail.mit.edu')
    s = BeautifulSoup(response.content, 'lxml')
    urls = []
    for url in s.find_all('a'):
        urls.append(url['href'])

if __name__ == '__main__':
    get_urls()
```

Pythonの `cProfile` プロファイラを使った場合、2500行を超える出力が得られ、ソートしたとしても、どこに時間が費やされているのかを理解するのは難しいでしょう。[`line_profiler`](https://github.com/pyutils/line_profiler) でさっと実行すると、行ごとにかかった時間が表示されます。

```bash
$ kernprof -l -v a.py
Wrote profile results to urls.py.lprof
Timer unit: 1e-06 s

Total time: 0.636188 s
File: a.py
Function: get_urls at line 5

Line #  Hits         Time  Per Hit   % Time  Line Contents
==============================================================
 5                                           @profile
 6                                           def get_urls():
 7         1     613909.0 613909.0     96.5      response = requests.get('https://missing.csail.mit.edu')
 8         1      21559.0  21559.0      3.4      s = BeautifulSoup(response.content, 'lxml')
 9         1          2.0      2.0      0.0      urls = []
10        25        685.0     27.4      0.1      for url in s.find_all('a'):
11        24         33.0      1.4      0.0          urls.append(url['href'])
```

### メモリ

CやC++のような言語では、メモリリークによって、もう必要のないメモリをプログラムがいつまでも解放しないことがあります。メモリのデバッグ作業を助けるために、[Valgrind](https://valgrind.org/)のようなツールを使ってメモリリークを特定できます。

Pythonのようなガベージコレクションのある言語でも、メモリプロファイラを使うことはやはり有用です。メモリ上のオブジェクトへのポインタを持っている限り、それらはガベージコレクションされないからです。以下は、プログラムの例と、それを[memory-profiler](https://pypi.org/project/memory-profiler/)で実行したときの出力です(`line-profiler` と同様のデコレータに注目してください)。

```python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

```bash
$ python -m memory_profiler example.py
Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```

### イベントプロファイリング

デバッグにおける `strace` の場合と同じように、プロファイリングの際にも、実行しているコードの詳細は無視して、ブラックボックスのように扱いたいことがあります。[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) コマンドはCPUの違いを抽象化し、時間やメモリを報告する代わりに、プログラムに関連するシステムイベントを報告します。たとえば `perf` は、キャッシュ局所性の悪さ、大量のページフォールト、ライブロックを簡単に報告できます。このコマンドの概要は次のとおりです。

- `perf list` - perfでトレースできるイベントを一覧表示します
- `perf stat COMMAND ARG1 ARG2` - プロセスまたはコマンドに関連するさまざまなイベントの回数を取得します
- `perf record COMMAND ARG1 ARG2` - コマンドの実行を記録し、統計データを `perf.data` というファイルに保存します
- `perf report` - `perf.data` に収集されたデータを整形して表示します


### 可視化

実世界のプログラムに対するプロファイラの出力は、ソフトウェアプロジェクトに本質的に備わる複雑さのために、大量の情報を含みます。人間は視覚的な生き物であり、大量の数値を読んでその意味を理解するのはかなり苦手です。そのため、プロファイラの出力をより解釈しやすい形で表示するツールが数多くあります。

サンプリングプロファイラのCPUプロファイリング情報を表示する一般的な方法のひとつが、[フレームグラフ(Flame Graph)](https://www.brendangregg.com/flamegraphs.html)です。これは、Y軸に関数呼び出しの階層を、X軸に所要時間に比例した幅を表示します。インタラクティブでもあり、プログラムの特定の部分にズームインして、そのスタックトレースを取得できます(下の画像をクリックしてみてください)。

[![フレームグラフ](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

コールグラフや制御フローグラフは、関数をノードとして、関数間の呼び出しを有向エッジとして含めることで、プログラム内のサブルーチン間の関係を表示します。呼び出し回数や所要時間といったプロファイリング情報と組み合わせると、コールグラフはプログラムの流れを解釈するのにとても役立ちます。Pythonでは、[`pycallgraph`](https://pycallgraph.readthedocs.io/) ライブラリを使って生成できます。

![コールグラフ](https://upload.wikimedia.org/wikipedia/commons/2/2f/A_Call_Graph_generated_by_pycallgraph.png)


## リソース監視

プログラムのパフォーマンスを分析する最初の一歩が、実際のリソース消費量を把握することである場合もあります。プログラムは、メモリが足りない、ネットワーク接続が遅いなど、リソースが制約されているときに遅くなることがよくあります。CPU使用率、メモリ使用量、ネットワーク、ディスク使用量など、さまざまなシステムリソースを調べて表示するためのコマンドラインツールは無数にあります。

- **全般的な監視** - おそらく最も人気があるのは [`htop`](https://htop.dev/) で、これは [`top`](https://www.man7.org/linux/man-pages/man1/top.1.html) の改良版です。`htop` は、システム上で現在実行中のプロセスについてさまざまな統計を表示します。`htop` には無数のオプションとキーバインドがあり、便利なものとしては、プロセスをソートする `<F6>`、ツリー階層を表示する `t`、スレッドの表示を切り替える `h` などがあります。優れたUIを備えた同様の実装として、[`glances`](https://nicolargo.github.io/glances/) も参照してください。全プロセスにまたがる集計値を得るには、[`dool`](https://github.com/scottchiefbaker/dool) も気の利いたツールで、I/O、ネットワーク、CPU使用率、コンテキストスイッチなど、多数のサブシステムのリソース指標をリアルタイムで計算します。
- **I/O操作** - [`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) はI/Oの使用状況をリアルタイムで表示し、プロセスがディスクI/Oを大量に行っていないかを確認するのに便利です
- **ディスク使用量** - [`df`](https://www.man7.org/linux/man-pages/man1/df.1.html) はパーティションごとの指標を表示し、[`du`](https://man7.org/linux/man-pages/man1/du.1.html) はカレントディレクトリのファイルごとのディスク使用量(**d**isk **u**sage)を表示します。これらのツールでは、`-h` フラグを付けると、人間が読みやすい(**h**uman readable)形式で出力します。`du` のよりインタラクティブなバージョンが [`ncdu`](https://dev.yorhel.nl/ncdu) で、フォルダ間を移動しながら、ファイルやフォルダを削除できます。
- **メモリ使用量** - [`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) は、システムの空きメモリと使用中メモリの総量を表示します。メモリは `htop` のようなツールでも表示されます。
- **開いているファイル** - [`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) は、プロセスが開いているファイルについての情報を一覧表示します。特定のファイルをどのプロセスが開いているかを確認するのにとても便利です。
- **ネットワーク接続と設定** - [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) を使うと、送受信されるネットワークパケットの統計やインターフェースの統計を監視できます。`ss` のよくある使い道は、マシン上の特定のポートをどのプロセスが使っているかを突き止めることです。ルーティング、ネットワークデバイス、インターフェースを表示するには、[`ip`](https://man7.org/linux/man-pages/man8/ip.8.html) が使えます。なお、`netstat` と `ifconfig` は非推奨となり、それぞれ前述のツールに置き換えられています。
- **ネットワーク使用量** - [`nethogs`](https://github.com/raboof/nethogs) と [`iftop`](https://pdw.ex-parrot.com/iftop/) は、ネットワーク使用量を監視するための優れたインタラクティブCLIツールです。

これらのツールを試したい場合は、[`stress`](https://linux.die.net/man/1/stress) コマンドを使ってマシンに人為的に負荷をかけることもできます。


### 専用ツール

どのソフトウェアを使うかを決めるのに、ブラックボックスのベンチマークさえあれば十分なこともあります。[`hyperfine`](https://github.com/sharkdp/hyperfine) のようなツールを使えば、コマンドラインプログラムのベンチマークをすばやく取れます。たとえば、シェルツールとスクリプトの講義では、`find` よりも `fd` をおすすめしました。`hyperfine` を使えば、よく実行するタスクでこれらを比較できます。たとえば以下の例では、私のマシンで `fd` は `find` より20倍高速でした。

```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

デバッグの場合と同じく、ブラウザにもWebページの読み込みをプロファイリングするための素晴らしいツール群が付属しており、どこに時間が費やされているのか(読み込み、レンダリング、スクリプト実行など)を突き止められます。詳しくは[Firefox](https://profiler.firefox.com/docs/)と[Chrome](https://developers.google.com/web/tools/chrome-devtools/rendering-tools)の情報を参照してください。

# 演習

## デバッグ
1. Linuxでは `journalctl`、macOSでは `log show` を使って、過去1日間のスーパーユーザーによるアクセスとコマンドを取得してください。ひとつもなければ、`sudo ls` のような無害なコマンドを実行してから、もう一度確認してください。

1. [こちら](https://github.com/spiside/pdb-tutorial)の `pdb` ハンズオンチュートリアルに取り組んで、コマンドに慣れてください。より詳しいチュートリアルとしては、[こちら](https://realpython.com/python-debugging-pdb)を読んでください。

1. [`shellcheck`](https://www.shellcheck.net/) をインストールして、次のスクリプトをチェックしてみてください。このコードのどこが間違っているでしょうか? 修正してください。警告が自動的に得られるように、エディタにリンタのプラグインをインストールしてください。

   ```bash
   #!/bin/sh
   ## Example: a typical script with several problems
   for f in $(ls *.m3u)
   do
     grep -qi hq.*mp3 $f \
       && echo -e 'Playlist $f contains a HQ file in mp3 format'
   done
   ```

1. (上級)[リバースデバッグ](https://undo.io/resources/reverse-debugging-whitepaper/)について読み、[`rr`](https://rr-project.org/) または [`RevPDB`](https://morepypy.blogspot.com/2016/07/reverse-debugging-for-python.html) を使って簡単な例を動かしてください。
## プロファイリング

1. [こちら]({{ site.baseurl }}/static/files/sorts.py)に、いくつかのソートアルゴリズムの実装があります。[`cProfile`](https://docs.python.org/3/library/profile.html) と [`line_profiler`](https://github.com/pyutils/line_profiler) を使って、挿入ソートとクイックソートの実行時間を比較してください。それぞれのアルゴリズムのボトルネックは何でしょうか? 次に `memory_profiler` を使ってメモリ消費量を確認してください。なぜ挿入ソートのほうが優れているのでしょうか? 今度はインプレース版のクイックソートを確認してください。チャレンジ: `perf` を使って、各アルゴリズムのサイクル数とキャッシュのヒット・ミスを見てください。

1. 以下は、数ごとにひとつの関数を使ってフィボナッチ数を計算する(回りくどいと言えなくもない)Pythonコードです。

   ```python
   #!/usr/bin/env python
   def fib0(): return 0

   def fib1(): return 1

   s = """def fib{}(): return fib{}() + fib{}()"""

   if __name__ == '__main__':

       for n in range(2, 10):
           exec(s.format(n, n-1, n-2))
       # from functools import lru_cache
       # for n in range(10):
       #     exec("fib{} = lru_cache(1)(fib{})".format(n, n))
       print(eval("fib9()"))
   ```

   このコードをファイルに保存して、実行可能にしてください。前提となる [`pycallgraph`](https://lewiscowles1986.github.io/py-call-graph/) と [`graphviz`](https://graphviz.org/) をインストールしてください(`dot` を実行できるなら、GraphVizはすでに入っています)。`pycallgraph graphviz -- ./fib.py` でコードをそのまま実行し、`pycallgraph.png` ファイルを確認してください。`fib0` は何回呼ばれているでしょうか? 関数をメモ化すれば、これを改善できます。コメントアウトされている行のコメントを外し、画像を再生成してください。今度は、各 `fibN` 関数を何回呼んでいるでしょうか?

1. よくある問題として、待ち受けたいポートがすでに別のプロセスに使われていることがあります。そのプロセスのpidを調べる方法を学びましょう。まず `python -m http.server 4444` を実行して、ポート `4444` で待ち受ける最小限のWebサーバーを起動してください。別のターミナルで `lsof | grep LISTEN` を実行し、待ち受け中のすべてのプロセスとポートを表示してください。そのプロセスのpidを見つけ、`kill <PID>` を実行して終了させてください。

1. プロセスのリソースを制限することも、道具箱に入れておくと便利な手段のひとつです。`stress -c 3` を実行し、`htop` でCPU消費を可視化してみてください。次に `taskset --cpu-list 0,2 stress -c 3` を実行して、可視化してください。`stress` は3つのCPUを使っているでしょうか? なぜ使っていないのでしょうか? [`man taskset`](https://www.man7.org/linux/man-pages/man1/taskset.1.html) を読んでください。チャレンジ: [`cgroups`](https://www.man7.org/linux/man-pages/man7/cgroups.7.html) を使って同じことを実現してください。`stress -m` のメモリ消費を制限してみてください。

1. (上級)コマンド `curl ipinfo.io` はHTTPリクエストを行い、あなたのパブリックIPについての情報を取得します。[Wireshark](https://www.wireshark.org/)を開いて、`curl` が送受信したリクエストとレスポンスのパケットをスニッフィングしてみてください(ヒント: `http` フィルタを使うと、HTTPパケットだけを見られます)。

