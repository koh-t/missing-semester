---
layout: lecture
title: "コマンドライン環境"
description: >
  ジョブ制御、ターミナルマルチプレクサ、ドットファイル、SSHによるリモートマシンについて学びます。
thumbnail: /static/assets/thumbnails/2020/lec5.png
date: 2020-01-21
ready: true
video:
  aspect: 56.25
  id: e8BO_dYxk5c
---

この講義では、シェルを使うときのワークフローを改善するいくつかの方法を見ていきます。シェルを使い始めてしばらく経ちますが、これまでは主にさまざまなコマンドを実行することに焦点を当ててきました。ここでは、複数のプロセスを把握しながら同時に実行する方法、特定のプロセスを停止または一時停止する方法、そしてプロセスをバックグラウンドで実行する方法を見ていきます。

また、エイリアスを定義したり、ドットファイルを使って設定したりすることで、シェルやその他のツールを改善するさまざまな方法も学びます。どちらも時間の節約に役立ちます。たとえば、長いコマンドを入力しなくても、すべてのマシンで同じ設定を使えるようになります。SSHを使ってリモートマシンを扱う方法も見ていきます。


# ジョブ制御

場合によっては、実行中のジョブを中断する必要があります。たとえば、コマンドの完了に時間がかかりすぎているとき(非常に大きなディレクトリ構造を検索する `find` など)です。
たいていの場合、`Ctrl-C` を押せばコマンドは停止します。
しかし、これは実際にはどのような仕組みで動いているのでしょうか。そして、なぜプロセスの停止に失敗することがあるのでしょうか。

## プロセスを終了させる

シェルは、_シグナル_ と呼ばれるUNIXの通信メカニズムを使って、プロセスに情報を伝えています。プロセスはシグナルを受け取ると実行を止め、シグナルを処理し、シグナルが伝えた情報に基づいて実行の流れを変えることもあります。このため、シグナルは _ソフトウェア割り込み_ です。

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

```
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

`SIGINT` と `SIGQUIT` はどちらも通常ターミナル関連の要求と結びついていますが、プロセスに正常終了を求めるためのより汎用的なシグナルとして `SIGTERM` シグナルがあります。
このシグナルを送るには [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) コマンドを、`kill -TERM <PID>` という構文で使います。

## プロセスの一時停止とバックグラウンド化

シグナルには、プロセスを終了させる以外の用途もあります。たとえば `SIGSTOP` はプロセスを一時停止します。ターミナルで `Ctrl-Z` を入力すると、シェルは `SIGTSTP` シグナルを送ります。これはTerminal Stopの略です(つまり `SIGSTOP` のターミナル版です)。

一時停止したジョブは、[`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) または [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html) を使って、それぞれフォアグラウンドまたはバックグラウンドで再開できます。

[`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) コマンドは、現在のターミナルセッションに関連付けられた未完了のジョブを一覧表示します。
これらのジョブは、pidを使って参照できます(pidは [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) で調べられます)。
より直感的には、パーセント記号にジョブ番号(`jobs` で表示されます)を続けてプロセスを参照することもできます。最後にバックグラウンドに送ったジョブを参照するには、特別なパラメータ `$!` が使えます。

もうひとつ知っておくべきことは、コマンドの末尾に `&` を付けるとそのコマンドがバックグラウンドで実行され、プロンプトが戻ってくるという点です。ただし、そのコマンドは依然としてシェルのSTDOUTを使うため、煩わしいことがあります(その場合はシェルのリダイレクトを使いましょう)。

すでに実行中のプログラムをバックグラウンドに送るには、`Ctrl-Z` に続けて `bg` を実行します。
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

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs

```

`SIGKILL` は特別なシグナルです。プロセスはこれを捕捉できず、必ず即座に終了させられます。ただし、孤児となった子プロセスを残すなど、望ましくない副作用を引き起こす可能性があります。

これらのシグナルやその他のシグナルについては、[こちら](https://en.wikipedia.org/wiki/Signal_(IPC))を読むか、[`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) あるいは `kill -l` を実行することで詳しく学べます。


# ターミナルマルチプレクサ

コマンドラインインターフェースを使っていると、複数のことを同時に実行したくなる場面がよくあります。
たとえば、エディタとプログラムを並べて動かしたい、といった場合です。
これは新しいターミナルウィンドウを開くことでも実現できますが、ターミナルマルチプレクサを使うほうがより柔軟な解決策です。

[`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) のようなターミナルマルチプレクサを使うと、ペインやタブでターミナルウィンドウを多重化し、複数のシェルセッションを操作できます。
さらに、ターミナルマルチプレクサは、現在のターミナルセッションをデタッチして、後で再アタッチすることも可能にします。
これにより、`nohup` などの小技を使う必要がなくなるので、リモートマシンでの作業時のワークフローが大幅に改善されます。

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

さらに学びたい場合は、[こちら](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)が `tmux` の手軽なチュートリアルで、[こちら](https://linuxcommand.org/lc3_adv_termmux.php)には元祖である `screen` コマンドも含めたより詳しい解説があります。[`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html) にも慣れておくとよいでしょう。ほとんどのUNIXシステムにインストールされているからです。

# エイリアス

多くのフラグや冗長なオプションを伴う長いコマンドを入力するのは、面倒になってくることがあります。
このため、ほとんどのシェルは _エイリアス_ をサポートしています。
シェルエイリアスとは、別のコマンドの短縮形のことで、シェルが自動的に置き換えてくれます。
たとえば、bashのエイリアスは次のような構造をしています。

```bash
alias alias_name="command_to_alias arg1 arg2"
```

等号 `=` の前後にスペースがないことに注意してください。[`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) は単一の引数を取るシェルコマンドだからです。

エイリアスには便利な機能がたくさんあります。

```bash
# Make shorthands for common flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gs="git status"
alias gc="git commit"
alias v="vim"

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

エイリアスは、デフォルトではシェルセッションをまたいで保持されないことに注意してください。
エイリアスを永続化するには、`.bashrc` や `.zshrc` のようなシェルの起動ファイルに書いておく必要があります。これらのファイルは次のセクションで紹介します。


# ドットファイル

多くのプログラムは、_ドットファイル_ と呼ばれるプレーンテキストのファイルで設定されます(`~/.vimrc` のようにファイル名が `.` で始まるためこう呼ばれます。そのおかげで、デフォルトでは `ls` のディレクトリ一覧に表示されず隠されます)。

シェルは、このようなファイルで設定されるプログラムの一例です。起動時に、シェルは多くのファイルを読み込んで設定をロードします。
シェルの種類や、ログインセッションおよび/または対話的セッションとして起動しているかどうかによって、この一連の処理はかなり複雑になることがあります。
このトピックに関する優れた資料が[こちら](https://web.archive.org/web/20260329133158/https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)にあります。

`bash` の場合、`.bashrc` または `.bash_profile` を編集すれば、ほとんどのシステムでうまくいきます。
ここには、先ほど説明したエイリアスや `PATH` 環境変数の変更など、起動時に実行したいコマンドを書いておけます。
実際、多くのプログラムは、バイナリを見つけられるように、`export PATH="$PATH:/path/to/program/bin"` のような行をシェルの設定ファイルに追加するよう求めてきます。

ドットファイルで設定できるツールの例をほかにも挙げると、次のとおりです。

- `bash` - `~/.bashrc`、`~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` と `~/.vim` フォルダ
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

ドットファイルはどのように整理すべきでしょうか? 専用のフォルダにまとめてバージョン管理下に置き、スクリプトを使って所定の場所に**シンボリックリンク**を張るのがよいでしょう。これには次のような利点があります。

- **簡単なインストール**: 新しいマシンにログインしたとき、自分のカスタマイズの適用が1分で済みます。
- **ポータビリティ**: どこでもツールが同じように動作します。
- **同期**: どこにいてもドットファイルを更新でき、すべてを同期した状態に保てます。
- **変更の追跡**: ドットファイルはおそらくプログラマー人生を通してメンテナンスし続けることになるので、長寿命のプロジェクトにはバージョン履歴があると心強いものです。

ドットファイルには何を書けばよいのでしょうか?
ツールの設定については、オンラインのドキュメントや[manページ](https://en.wikipedia.org/wiki/Man_page)を読んで学べます。もうひとつの優れた方法は、特定のプログラムについてのブログ記事をインターネットで検索することです。著者が自分のお気に入りのカスタマイズを紹介してくれています。さらに別の方法として、他の人のドットファイルを覗いてみるという手もあります。GitHubには膨大な数の[dotfilesリポジトリ](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)があります --- もっとも人気のあるものは[こちら](https://github.com/mathiasbynens/dotfiles)です(ただし、設定をやみくもにコピーするのはおすすめしません)。
このトピックに関するもうひとつの良い資料が[こちら](https://dotfiles.github.io/)にあります。

この授業の講師は全員、自分のドットファイルをGitHubで公開しています: [Anish](https://github.com/anishathalye/dotfiles)、[Jon](https://github.com/jonhoo/configs)、[Jose](https://github.com/jjgo/dotfiles)。


## ポータビリティ

ドットファイルでよくある悩みは、複数のマシンを使っていると、たとえばOSやシェルが異なる場合に、設定がうまく動かないことがあるという点です。また、ある設定を特定のマシンにだけ適用したいこともあります。

これを楽にするための工夫がいくつかあります。
設定ファイルが対応していれば、if文に相当するものを使って、マシン固有のカスタマイズを適用しましょう。たとえば、シェルの設定には次のようなものを書けます。

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# Check before using shell-specific features
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# You can also make it machine-specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

設定ファイルが対応していれば、インクルードを活用しましょう。たとえば、`~/.gitconfig` には次のような設定を書けます。

```
[include]
    path = ~/.gitconfig_local
```

そして各マシンで、`~/.gitconfig_local` にマシン固有の設定を書いておけます。マシン固有の設定用の別のリポジトリで、これらを管理することもできます。

この考え方は、異なるプログラムで一部の設定を共有したい場合にも役立ちます。たとえば、`bash` と `zsh` の両方で同じエイリアス一式を共有したいなら、それらを `.aliases` に書いておき、両方に次のブロックを入れておけます。

```bash
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

# リモートマシン {#remote-machines}

プログラマーが日々の仕事でリモートサーバーを使うことは、ますます一般的になっています。バックエンドソフトウェアをデプロイするためにリモートサーバーを使う必要がある場合や、より高い計算能力を持つサーバーが必要な場合には、Secure Shell(SSH)を使うことになります。これまで扱ってきたほとんどのツールと同様に、SSHは高度に設定可能なので、学んでおく価値があります。

サーバーに `ssh` で接続するには、次のようなコマンドを実行します。

```bash
ssh foo@bar.mit.edu
```

ここでは、ユーザー `foo` としてサーバー `bar.mit.edu` にsshで接続しようとしています。
サーバーはURL(`bar.mit.edu` など)またはIP(`foobar@192.168.1.42` のようなもの)で指定できます。あとで見るように、sshの設定ファイルを変更すれば、`ssh bar` のようなコマンドだけでアクセスできるようになります。

## コマンドの実行

`ssh` の見落とされがちな機能に、コマンドを直接実行できるというものがあります。
`ssh foobar@server ls` は、foobarのホームフォルダで `ls` を実行します。
パイプとも組み合わせられるので、`ssh foobar@server ls | grep PATTERN` はリモートの `ls` の出力をローカルでgrepし、`ls | ssh foobar@server grep PATTERN` はローカルの `ls` の出力をリモートでgrepします。


## SSH鍵

鍵ベースの認証は公開鍵暗号を利用して、秘密鍵そのものを明かすことなく、クライアントがその秘密鍵を所有していることをサーバーに証明します。こうすれば、毎回パスワードを入力し直す必要がありません。とはいえ、秘密鍵(多くの場合 `~/.ssh/id_rsa`、最近では `~/.ssh/id_ed25519`)は実質的にあなたのパスワードなので、パスワードと同様に扱ってください。

### 鍵の生成

鍵ペアを生成するには [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html) を実行します。
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```
パスフレーズは設定しておくべきです。秘密鍵を手に入れた誰かが、認可されたサーバーにアクセスできてしまうのを防ぐためです。毎回パスフレーズを入力しなくて済むように、[`ssh-agent`](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) または [`gpg-agent`](https://linux.die.net/man/1/gpg-agent) を使いましょう。

SSH鍵を使ってGitHubへプッシュする設定をしたことがあるなら、おそらく[こちら](https://help.github.com/articles/connecting-to-github-with-ssh/)に書かれている手順をすでに済ませており、有効な鍵ペアを持っているはずです。パスフレーズが設定されているかを確認・検証するには、`ssh-keygen -y -f /path/to/key` を実行します。

### 鍵ベースの認証

`ssh` は `.ssh/authorized_keys` を参照して、どのクライアントを受け入れるべきかを判断します。公開鍵をコピーするには次のようにします。

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

`ssh-copy-id` が使える環境では、より簡単に行えます。

```bash
ssh-copy-id -i .ssh/id_ed25519 foobar@remote
```

## SSH経由でのファイルコピー

ssh経由でファイルをコピーする方法はたくさんあります。

- `ssh+tee`: もっとも単純なのは、`ssh` のコマンド実行とSTDIN入力を使い、`cat localfile | ssh remote_server tee serverfile` とする方法です。[`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html) はSTDINからの出力をファイルに書き込むことを思い出してください。
- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html): 大量のファイルやディレクトリをコピーする場合は、パスを簡単に再帰的にたどれるセキュアコピーの `scp` コマンドのほうが便利です。構文は `scp path/to/local_file remote_host:path/to/remote_file` です。
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) は `scp` を改良したもので、ローカルとリモートで同一のファイルを検出し、それらを再度コピーするのを防ぎます。また、シンボリックリンクやパーミッションをより細かく制御でき、中断されたコピーを途中から再開できる `--partial` フラグのような追加機能もあります。`rsync` の構文は `scp` に似ています。

## ポートフォワーディング

多くの場面で、マシンの特定のポートで待ち受けるソフトウェアに出会うことになります。これが手元のマシンで起きているなら `localhost:PORT` や `127.0.0.1:PORT` と入力すれば済みますが、ネットワーク/インターネット経由でポートに直接アクセスできないリモートサーバーの場合はどうすればよいでしょうか?

これは _ポートフォワーディング_ と呼ばれ、ローカルポートフォワーディングとリモートポートフォワーディングの2種類があります(詳しくは図を参照してください。図の出典は[このStackOverflowの投稿](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)です)。

**ローカルポートフォワーディング**
![ローカルポートフォワーディング]({{ site.baseurl }}/static/media/images/local-port-forwarding.png)

**リモートポートフォワーディング**
![リモートポートフォワーディング]({{ site.baseurl }}/static/media/images/remote-port-forwarding.png)

もっともよくあるのはローカルポートフォワーディングで、リモートマシンのサービスがあるポートで待ち受けており、手元のマシンのポートをそのリモートのポートへ転送するようにつなぎたい、という場面です。たとえば、リモートサーバーで `jupyter notebook` を実行し、それがポート `8888` で待ち受けているとします。これをローカルのポート `9999` に転送するには、`ssh -L 9999:localhost:8888 foobar@remote_server` を実行し、手元のマシンで `localhost:9999` にアクセスします。


## SSHの設定

ここまで、渡すことのできる多くの引数を扱ってきました。ついやりたくなる方法のひとつは、次のようなシェルエイリアスを作ることです。
```bash
alias my_server="ssh -i ~/.ssh/id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server"
```

しかし、`~/.ssh/config` を使うという、より良い方法があります。

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```

エイリアスではなく `~/.ssh/config` ファイルを使うもうひとつの利点は、`scp`、`rsync`、`mosh` などの他のプログラムもこのファイルを読み込み、設定を対応するフラグに変換できることです。


`~/.ssh/config` ファイルはドットファイルの一種と見なせるので、基本的には他のドットファイルと一緒に管理してかまいません。ただし、公開する場合は、インターネット上の見知らぬ人に与えてしまうかもしれない情報について考えてください。サーバーのアドレス、ユーザー名、開いているポートなどです。これはある種の攻撃を容易にするかもしれないので、SSHの設定を共有する際は慎重に考えましょう。

サーバー側の設定は、通常 `/etc/ssh/sshd_config` で指定します。ここでは、パスワード認証の無効化、sshのポートの変更、X11フォワーディングの有効化などの変更ができます。設定はユーザーごとに指定することもできます。

## その他

リモートサーバーに接続しているときによくある悩みが、コンピュータのシャットダウンやスリープ、ネットワークの切り替えによる切断です。さらに、遅延の大きい回線では、sshを使うのがかなりもどかしくなることがあります。モバイルシェルである[Mosh](https://mosh.org/)はsshを改良したもので、ローミング接続や断続的な接続に対応し、賢いローカルエコーを提供します。

リモートのフォルダをマウントできると便利なこともあります。[sshfs](https://github.com/libfuse/sshfs)はリモートサーバー上のフォルダをローカルにマウントでき、そうすればローカルのエディタを使えます。


# シェルとフレームワーク

シェルツールとスクリプトの講義では `bash` シェルを扱いました。これが群を抜いて広く普及しているシェルであり、ほとんどのシステムでデフォルトになっているからです。とはいえ、選択肢はこれだけではありません。

たとえば `zsh` シェルは `bash` のスーパーセットであり、次のような便利な機能を標準で数多く備えています。

- より賢いグロブ、`**`
- インラインでのグロブ/ワイルドカード展開
- スペル修正
- より優れたタブ補完/選択
- パス展開(`cd /u/lo/b` は `/usr/local/bin` に展開されます)

**フレームワーク**もシェルを改善してくれます。人気のある汎用フレームワークには[prezto](https://github.com/sorin-ionescu/prezto)や[oh-my-zsh](https://ohmyz.sh/)があり、特定の機能に特化した小さなものには[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)や[zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)などがあります。[fish](https://fishshell.com/)のようなシェルは、こうしたユーザーフレンドリーな機能の多くをデフォルトで備えています。そうした機能には次のようなものがあります。

- 右プロンプト
- コマンドのシンタックスハイライト
- 履歴の部分文字列検索
- manページに基づくフラグ補完
- より賢い自動補完
- プロンプトのテーマ

これらのフレームワークを使う際にひとつ注意すべきなのは、シェルを遅くする可能性があることです。実行されるコードが適切に最適化されていなかったり、コードの量が多すぎたりする場合は特にそうです。いつでもプロファイルを取って、あまり使わない機能や、速度ほどには重視していない機能を無効にできます。

# ターミナルエミュレータ

シェルのカスタマイズと合わせて、**ターミナルエミュレータ**の選択とその設定にも時間をかける価値があります。世の中には数多くのターミナルエミュレータがあります([比較](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)はこちら)。

ターミナルの中で何百時間、何千時間と過ごすことになるかもしれないのですから、その設定を調べておくことは十分に元が取れます。ターミナルで調整したくなるであろう項目には、次のようなものがあります。

- フォントの選択
- カラースキーム
- キーボードショートカット
- タブ/ペインのサポート
- スクロールバックの設定
- パフォーマンス([Alacritty](https://github.com/jwilm/alacritty)や[kitty](https://sw.kovidgoyal.net/kitty/)のような比較的新しいターミナルは、GPUアクセラレーションを提供しています)。

# 演習

## ジョブ制御

1. これまで見てきた内容からすると、`ps aux | grep` のようなコマンドでジョブのpidを調べてからkillすることもできますが、もっと良い方法があります。ターミナルで `sleep 10000` ジョブを開始し、`Ctrl-Z` でバックグラウンドに移して、`bg` で実行を再開してください。次に [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) を使ってそのpidを見つけ、[`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html) を使って、pidを一度も自分で入力することなくこのジョブをkillしてください。(ヒント: `-af` フラグを使いましょう)。

1. あるプロセスが完了するまで、別のプロセスを開始したくないとします。どうすればよいでしょうか? この演習では、完了を待つ対象のプロセスは常に `sleep 60 &` とします。ひとつの方法は [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) コマンドを使うことです。sleepコマンドを起動し、そのバックグラウンドプロセスが終了するまで `ls` を待たせてみてください。

    しかし、この戦略は、別のbashセッションから始めた場合には失敗します。`wait` は子プロセスに対してしか機能しないからです。講義ノートで触れなかった機能のひとつに、`kill` コマンドの終了ステータスは成功時にゼロ、それ以外は非ゼロになる、というものがあります。`kill -0` はシグナルを送信しませんが、プロセスが存在しない場合に非ゼロの終了ステータスを返します。pidを受け取り、そのプロセスが完了するまで待つ `pidwait` というbash関数を書いてください。CPUを無駄に消費しないように `sleep` を使うこと。

## ターミナルマルチプレクサ

1. こちらの `tmux` [チュートリアル](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)に従って学び、その後[こちらの手順](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)に従って基本的なカスタマイズの方法を学んでください。

## エイリアス

1. `cd` を打ち間違えたときのために、`dc` を `cd` に解決するエイリアスを作ってください。

1.  `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` を実行して、もっともよく使っているコマンドのトップ10を調べ、それらに短いエイリアスを作ることを検討してください。注: これはBashで動作します。ZSHを使っている場合は、`history` の代わりに `history 1` を使ってください。


## ドットファイル

ドットファイルの扱いに慣れていきましょう。
1. ドットファイル用のフォルダを作り、バージョン管理をセットアップしてください。
1. 少なくとも1つのプログラム(たとえばシェル)の設定を、何らかのカスタマイズ付きで追加してください(手始めとしては、`$PS1` を設定してシェルのプロンプトをカスタマイズするような簡単なものでかまいません)。
1. 新しいマシンにドットファイルをすばやく(かつ手作業なしで)インストールする方法をセットアップしてください。各ファイルに対して `ln -s` を呼ぶだけの単純なシェルスクリプトでもよいですし、[専用のユーティリティ](https://dotfiles.github.io/utilities/)を使ってもかまいません。
1. まっさらな仮想マシンで、インストールスクリプトをテストしてください。
1. 現在使っているすべてのツールの設定を、ドットファイルのリポジトリに移行してください。
1. 自分のドットファイルをGitHubで公開してください。

## リモートマシン

この演習のために、Linuxの仮想マシンをインストールしてください(既存のものがあればそれを使ってもかまいません)。仮想マシンに馴染みがない場合は、[こちら](https://hibbard.eu/install-ubuntu-virtual-box/)のインストール手順のチュートリアルを確認してください。

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
1. VM内で `python -m http.server 8888` を実行してWebサーバーを起動してください。手元のマシンで `http://localhost:9999` にアクセスして、VMのWebサーバーに接続してください。
1. `sudo vim /etc/ssh/sshd_config` でSSHサーバーの設定を編集し、`PasswordAuthentication` の値を変更してパスワード認証を無効化してください。さらに `PermitRootLogin` の値を変更してrootログインを無効化してください。`sudo service sshd restart` で `ssh` サービスを再起動し、もう一度sshで接続してみてください。
1. (チャレンジ)VMに [`mosh`](https://mosh.org/) をインストールして接続を確立してください。その後、サーバー/VMのネットワークアダプタを切断してみてください。moshは正しく復帰できるでしょうか?
1. (チャレンジ)`ssh` の `-N` と `-f` フラグが何をするのかを調べ、バックグラウンドでポートフォワーディングを行うコマンドを考えてください。
