---
layout: lecture
title: "デバッグとプロファイリング"
description: >
  ロギングとデバッガを使ったプログラムのデバッグ方法と、パフォーマンスのためのコードのプロファイリング方法を学びます。
thumbnail: /static/assets/thumbnails/2026/lec4.png
date: 2026-01-15
ready: true
panopto: "https://mit.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=a72c48e3-5eb2-46fa-aa03-b3b700e1ca8d"
video:
  aspect: 56.25
  id: 8VYT9TcUmKs
---

プログラミングにおける黄金律は、コードはあなたが期待した通りには動かず、あなたが命じた通りに動く、というものです。そのギャップを埋めるのは、時にかなり難しい仕事になります。この講義では、バグを含むコードやリソースを大量に消費するコードに対処するための有用な技術、すなわちデバッグとプロファイリングを扱います。

# デバッグ

## printfデバッグとロギング

> 「最も効果的なデバッグツールは、今でも注意深い思考と、要所要所に配置されたprint文である」 — Brian Kernighan, _Unix for Beginners_

プログラムをデバッグする最初のアプローチは、問題を検出したあたりにprint文を追加し、問題の原因を理解するのに十分な情報が得られるまで繰り返すことです。

2つ目のアプローチは、その場しのぎのprint文の代わりに、プログラムでロギングを使うことです。ロギングは本質的に「より丁寧なprint」であり、通常は次のような機能を組み込みで備えたロギングフレームワークを通じて行われます。

- ログ(またはログの一部)を別の出力先に振り分ける機能
- 深刻度レベル(INFO、DEBUG、WARN、ERRORなど)を設定し、それに応じて出力をフィルタリングできる機能
- ログエントリに関連するデータを構造化してロギングし、後から容易に抽出できるようにするサポート

また、ログ出力の文は、デバッグに必要なデータが最初から揃っているように、プログラミング中に先回りして入れておくことが多いものです。実際、print文を使って問題を見つけて修正したら、それらを削除する前に、適切なログ文に変換しておく価値がしばしばあります。こうしておけば、将来同じようなバグが発生したとき、コードを変更することなく、必要な診断情報がすでに手元にあることになります。

> **サードパーティのログ**: 多くのプログラムは、実行時により多くの情報を出力する `-v` や `--verbose` フラグをサポートしています。これは、あるコマンドがなぜ失敗するのかを突き止めるのに役立ちます。フラグを繰り返し指定することで、さらに詳細な情報が得られるものもあります。サービス(データベース、Webサーバーなど)の問題をデバッグするときは、そのログを確認しましょう。Linuxでは多くの場合 `/var/log/` にあります。systemdのサービスのログを見るには `journalctl -u <service>` を使います。サードパーティのライブラリについては、環境変数や設定でデバッグロギングをサポートしていないか確認してみてください。

## デバッガ

printデバッグは、何を出力すればよいかが分かっていて、コードを簡単に変更・再実行できる場合にはうまく機能します。デバッガが真価を発揮するのは、どんな情報が必要か分からないとき、バグが再現しにくい条件でしか発生しないとき、あるいはプログラムの変更や再起動のコストが高い(起動時間が長い、再現すべき状態が複雑など)ときです。

デバッガは、プログラムの実行にリアルタイムで介入できるプログラムで、次のようなことができます。

- 特定の行に到達したときに実行を停止する。
- 一度に1命令ずつステップ実行する。
- クラッシュ後に変数の値を調べる。
- 特定の条件が満たされたときに条件付きで実行を停止する。
- その他多くの高度な機能。

ほとんどのプログラミング言語は、何らかの形のデバッガをサポート(または同梱)しています。最も汎用性が高いのは、[`gdb`](https://www.gnu.org/software/gdb/)(GNU Debugger)や [`lldb`](https://lldb.llvm.org/)(LLVM Debugger)のような**汎用デバッガ**で、あらゆるネイティブバイナリをデバッグできます。多くの言語には、ランタイムとより緊密に統合された**言語固有のデバッガ**もあります(Pythonのpdb、Javaのjdbなど)。

`gdb` は、C、C++、Rustなどのコンパイル言語における事実上の標準デバッガです。ほぼあらゆるプロセスを調査して、レジスタ、スタック、プログラムカウンタなど、現在のマシン状態を取得できます。

便利なGDBコマンドをいくつか挙げます。

- `run` - プログラムを開始する
- `b {function}` または `b {file}:{line}` - ブレークポイントを設定する
- `c` - 実行を継続する
- `step` / `next` / `finish` - ステップイン / ステップオーバー / ステップアウト
- `p {variable}` - 変数の値を表示する
- `bt` - バックトレース(コールスタック)を表示する
- `watch {expression}` - 値が変化したときに停止する

> GDBのTUIモード(`gdb -tui`、またはGDB内で `Ctrl-x a` を押す)を使うと、ソースコードとコマンドプロンプトを並べて表示する分割画面ビューになるので、検討してみてください。

### 記録・再生デバッグ

最も苛立たしいバグのひとつが _ハイゼンバグ(Heisenbug)_ です。これは、観察しようとすると消えたり振る舞いが変わったりするように見えるバグのことです。競合状態(race condition)、タイミングに依存するバグ、特定のシステム条件でしか現れない問題がこれに該当します。プログラムを再実行するたびに違う振る舞いをするため(例えば、print文がコードを十分に遅くして競合が起きなくなってしまうなど)、従来のデバッグはここではしばしば役に立ちません。

**記録・再生デバッグ(record-replay debugging)** は、プログラムの実行を記録し、必要な回数だけ決定的に再生できるようにすることで、この問題を解決します。さらに、実行を _逆方向_ にたどって、何が悪かったのかを正確に見つけることもできます。

[rr](https://rr-project.org/) は、プログラムの実行を記録し、完全なデバッグ機能を備えた決定的な再生を可能にする、Linux向けの強力なツールです。GDBと連携して動作するので、インターフェースはすでにおなじみのものです。

基本的な使い方は次の通りです。

```bash
# Record a program execution
rr record ./my_program

# Replay the recording (opens GDB)
rr replay
```

魔法が起こるのは再生時です。実行が決定的であるため、**逆方向デバッグ(reverse debugging)** コマンドが使えます。

- `reverse-continue` (`rc`) - ブレークポイントに到達するまで逆方向に実行する
- `reverse-step` (`rs`) - 1行逆方向にステップする
- `reverse-next` (`rn`) - 関数呼び出しをスキップしながら逆方向にステップする
- `reverse-finish` - 現在の関数に入るところまで逆方向に実行する

これはデバッグにおいて信じられないほど強力です。例えばクラッシュが起きたとします。バグの場所を推測してブレークポイントを設定する代わりに、次のようにできます。

1. クラッシュまで実行する
2. 壊れた状態を調べる
3. 壊れた変数にウォッチポイントを設定する
4. `reverse-continue` で、それがどこで壊されたのかを正確に見つける

**rrを使うべき場面:**
- 断続的に失敗する不安定なテスト
- 競合状態やスレッド関連のバグ
- 再現が難しいクラッシュ
- 「時間を巻き戻せたら」と思うようなあらゆるバグ

> 注意: rrはLinux上でのみ動作し、ハードウェアパフォーマンスカウンタを必要とします。これらのカウンタを公開していないVM(ほとんどのAWS EC2インスタンスなど)では動作せず、GPUアクセスもサポートしていません。macOSでは [Warpspeed](https://warpspeed.dev/) をチェックしてみてください。

> **rrと並行処理**: rrは実行を決定的に記録するため、スレッドのスケジューリングを直列化します。つまり、特定のタイミングに依存する競合状態は、rrの下では発生しないことがあります。それでもrrは競合のデバッグに有用です。失敗した実行を一度捕まえてしまえば、確実に再生できるからです。ただし、断続的なバグを捕まえるには、複数回の記録の試行が必要になるかもしれません。並行処理が関係しないバグに対しては、rrは最も輝きます。常に全く同じ実行を再現でき、逆方向デバッグを使ってデータ破壊を追い詰めることができます。

## システムコールのトレース

プログラムがオペレーティングシステムとどのようにやり取りしているかを理解する必要がある場合もあります。プログラムは、ファイルを開く、メモリを割り当てる、プロセスを作成するといったカーネルのサービスを要求するために、[システムコール](https://en.wikipedia.org/wiki/System_call)を発行します。これらの呼び出しをトレースすることで、プログラムがなぜハングしているのか、どのファイルにアクセスしようとしているのか、どこで待ち時間を費やしているのかが明らかになります。

### strace (Linux) と dtruss (macOS)

[`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) を使うと、プログラムが発行するすべてのシステムコールを観察できます。

```bash
# Trace all system calls
strace ./my_program

# Trace only file-related calls
strace -e trace=file ./my_program

# Follow child processes (important for programs that start other programs)
strace -f ./my_program

# Trace a running process
strace -p <PID>

# Show timing information
strace -T ./my_program
```

> macOSとBSDでは、同様の機能を持つ [`dtruss`](https://www.manpagez.com/man/1/dtruss/)(`dtrace` のラッパー)を使ってください。

> `strace` をより深く知りたい場合は、Julia Evansによる素晴らしい [strace zine](https://jvns.ca/strace-zine-unfolded.pdf) をチェックしてみてください。

### bpftrace と eBPF

[eBPF](https://ebpf.io/)(extended Berkeley Packet Filter)は、サンドボックス化されたプログラムをカーネル内で実行できるようにする、Linuxの強力な技術です。[`bpftrace`](https://github.com/iovisor/bpftrace) は、eBPFプログラムを書くための高水準な構文を提供します。これらはカーネル内で動く任意のプログラムであり、それゆえ非常に大きな表現力を持ちます(ただし、awkに似たやや扱いにくい構文でもあります)。最も一般的なユースケースは、どのシステムコールが呼び出されているのかを調査することで、集計(カウントやレイテンシの統計など)や、システムコールの引数の中身を調べる(あるいはそれによるフィルタリング)も含まれます。

```bash
# Trace file opens system-wide (prints immediately)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# Count system calls by name (prints summary on Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }'
```

しかし、[`bcc`](https://github.com/iovisor/bcc) のようなツールチェーンを使えば、eBPFプログラムをC言語で直接書くこともできます。bccには、ディスク操作のレイテンシ分布を表示する `biosnoop` や、開かれたすべてのファイルを表示する `opensnoop` など、[多くの便利なツール](https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html)も付属しています。

`strace` が「とりあえずすぐに使い始められる」点で有用であるのに対し、`bpftrace` は、より低いオーバーヘッドが必要なとき、カーネル関数の中までトレースしたいとき、何らかの集計を行いたいときなどに手を伸ばすべきツールです。ただし、`bpftrace` は `root` として実行する必要があり、また一般に特定のプロセスだけでなくカーネル全体を監視することに注意してください。特定のプログラムを対象にするには、コマンド名やPIDでフィルタリングできます。

```bash
# Filter by command name (prints summary on Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /comm == "bash"/ { @[probe] = count(); }'

# Trace a specific command from startup using -c (cpid = child PID)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == cpid/ { @[probe] = count(); }' -c 'ls -la'
```

`-c` フラグは指定したコマンドを実行し、そのPIDを `cpid` に設定します。これは、プログラムを起動した瞬間からトレースするのに便利です。トレース対象のコマンドが終了すると、bpftraceは集計結果を出力します。

### ネットワークのデバッグ

ネットワークの問題については、[`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) と [Wireshark](https://www.wireshark.org/) を使ってネットワークパケットをキャプチャして分析できます。

```bash
# Capture packets on port 80
sudo tcpdump -i any port 80

# Capture and save to file for Wireshark analysis
sudo tcpdump -i any -w capture.pcap
```

HTTPSトラフィックについては、暗号化のためtcpdumpの有用性は下がります。[mitmproxy](https://mitmproxy.org/) のようなツールは、傍受プロキシとして動作し、暗号化されたトラフィックを検査できます。Webアプリケーションから発行されるHTTPSリクエストをデバッグするには、ブラウザの開発者ツール(Networkタブ)が最も手軽な方法であることが多く、復号されたリクエスト/レスポンスのデータ、ヘッダ、タイミング情報を表示してくれます。

## メモリのデバッグ

メモリのバグ、すなわちバッファオーバーフロー、解放後使用(use-after-free)、メモリリークは、最も危険でデバッグが難しいバグの部類に入ります。これらは多くの場合、すぐにはクラッシュせず、ずっと後になって問題を引き起こすような形でメモリを破壊します。

### サニタイザ

メモリのバグを見つけるひとつのアプローチは、**サニタイザ**を使うことです。これは、実行時にエラーを検出できるようコードを計装するコンパイラの機能です。例えば、広く使われている **AddressSanitizer (ASan)** は次のものを検出します。
- バッファオーバーフロー(スタック、ヒープ、グローバル)
- 解放後使用(use-after-free)
- リターン後使用(use-after-return)
- メモリリーク

```bash
# Compile with AddressSanitizer
gcc -fsanitize=address -g program.c -o program
./program
```

有用なサニタイザは他にも各種あります。

- **ThreadSanitizer (TSan)**: マルチスレッドコードのデータ競合を検出します(`-fsanitize=thread`)
- **MemorySanitizer (MSan)**: 未初期化メモリの読み取りを検出します(`-fsanitize=memory`)
- **UndefinedBehaviorSanitizer (UBSan)**: 整数オーバーフローなどの未定義動作を検出します(`-fsanitize=undefined`)

サニタイザは再コンパイルを必要としますが、CIパイプラインや日々の開発で使えるほど十分に高速です。

### Valgrind: 再コンパイルできないとき

[Valgrind](https://valgrind.org/) は、代わりに一種の仮想マシンのような環境でプログラムを実行してメモリエラーを検出します。サニタイザより低速ですが、再コンパイルは不要です。

```bash
valgrind --leak-check=full ./my_program
```

Valgrindを使うべき場面は次の通りです。
- ソースコードがないとき
- 再コンパイルできないとき(サードパーティのライブラリ)
- サニタイザにはない特定のツールが必要なとき

Valgrindは実は非常に強力な制御された実行環境であり、プロファイリングの話に入るときに再登場します!

## AIによるデバッグ

大規模言語モデル(LLM)は、驚くほど有用なデバッグアシスタントになりました。従来のツールを補完する特定のデバッグタスクにおいて、LLMは力を発揮します。

**LLMが得意なこと:**

- **不可解なエラーメッセージの説明**: コンパイラのエラー、特にC++のテンプレートやRustの借用チェッカが出すものは、悪名高いほど分かりにくいことがあります。LLMはそれらを平易な言葉に翻訳し、修正案を提示できます。

- **言語や抽象化の境界をまたぐこと**: 複数の言語にまたがる問題(例えば、Pythonバインディングを通じて顕在化するCライブラリのバグ)をデバッグしている場合、LLMは異なるレイヤの間を行き来する手助けをしてくれます。FFIの境界、ビルドシステムの問題、言語をまたいだデバッグ(例:プログラムがエラーになるが、原因は依存ライブラリのバグだと思われる、など)の理解が特に得意です。

- **症状と根本原因の関連付け**: 「プログラムは正常に動くが、想定の10倍のメモリを使う」といった漠然とした症状は、LLMが調査を助けられる類のもので、ありそうな原因や着目すべき点を提案してくれます。

- **クラッシュダンプやスタックトレースの分析**: スタックトレースを貼り付けて、何が原因になり得るかを尋ねてみましょう。

> **デバッグシンボルに関する注意**: 意味のあるスタックトレースとデバッグのためには、バイナリ(およびリンクされるライブラリ)がデバッグシンボル付き(`-g` フラグ)でコンパイルされていることを確認してください。デバッグ情報は通常DWARF形式で格納されます。さらに、フレームポインタ付き(`-fno-omit-frame-pointer`)でコンパイルすると、特にプロファイリングツールにとって、スタックトレースの信頼性が高まります。これらがないと、スタックトレースがメモリアドレスしか表示しなかったり、不完全になったりすることがあります。これはPythonやJavaよりも、ネイティブにコンパイルされるプログラム(C++、Rust)でより重要です。

**心に留めておくべき限界:**
- LLMは、もっともらしく聞こえるが間違った説明をでっち上げる(ハルシネーションを起こす)ことがあります
- バグを修正するのではなく、覆い隠すだけの修正を提案することがあります
- 提案は必ず実際のデバッグツールで検証しましょう
- LLMは、コードを理解することの代替ではなく、補完として使うのが最も効果的です

> これは、開発環境の講義で扱った[AIコーディング全般の機能]({{ site.baseurl }}/2026/development-environment/#ai-powered-development)とは別の話です。ここでは特に、デバッグの補助としてLLMを使うことについて述べています。

# プロファイリング

たとえコードが機能的に期待通りに動作していても、その過程でCPUやメモリを食い尽くしてしまうなら、それでは不十分かもしれません。アルゴリズムの授業ではビッグ _O_ 記法は教えても、プログラムのホットスポットの見つけ方は教えないことが多いものです。[早すぎる最適化は諸悪の根源](https://wiki.c2.com/?PrematureOptimization)なので、プロファイラと監視ツールについて学ぶべきです。これらは、プログラムのどの部分が時間やリソースの大半を消費しているのかを理解する助けになり、最適化すべき部分に集中できるようになります。

## 時間計測

パフォーマンスを測定する最も単純な方法は、時間を計ることです。多くの場面では、2点間でコードにかかった時間を出力するだけで十分なこともあります。

しかし、実時間(wall clock time)は誤解を招くことがあります。コンピュータが同時に他のプロセスを実行していたり、イベントの発生を待っていたりするかもしれないからです。`time` コマンドは _Real_、_User_、_Sys_ の時間を区別します。

- **Real** - 開始から終了までの実時間。待ち時間を含む
- **User** - CPUがユーザーコードの実行に費やした時間
- **Sys** - CPUがカーネルコードの実行に費やした時間

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real	0m0.272s
user	0m0.079s
sys	    0m0.028s
```

この例では、リクエストに300ミリ秒近く(real時間)かかっていますが、CPU時間(user + sys)は107ミリ秒だけです。残りはネットワークの待ち時間でした。

## リソース監視

プログラムのパフォーマンス分析の第一歩は、実際のリソース消費が何であるかを理解することである場合があります。プログラムは、リソースの制約を受けているときに遅くなることがよくあります。

- **全般的な監視**: [`htop`](https://htop.dev/) は `top` の改良版で、現在実行中のプロセスに関するさまざまな統計情報を表示します。便利なキー操作: `<F6>` でプロセスのソート、`t` でツリー階層表示、`h` でスレッド表示の切り替え。さらに _はるかに_ 多くの項目を監視できる [`btop`](https://github.com/aristocratos/btop) もあります。

- **I/O操作**: [`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) は、リアルタイムのI/O使用状況を表示します。

- **メモリ使用量**: [`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) は、空きメモリと使用中メモリの合計を表示します。

- **オープン中のファイル**: [`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) は、プロセスが開いているファイルの情報を一覧表示します。特定のファイルをどのプロセスが開いているかを調べるのに便利です。

- **ネットワーク接続**: [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) を使うと、ネットワーク接続を監視できます。よくあるユースケースは、あるポートをどのプロセスが使っているのかを突き止めることです: `ss -tlnp | grep :8080`。

- **ネットワーク使用量**: [`nethogs`](https://github.com/raboof/nethogs) と [`iftop`](https://pdw.ex-parrot.com/iftop/) は、プロセスごとのネットワーク使用量を監視するのに適した対話型CLIツールです。

## パフォーマンスデータの可視化

人間は、数値の表よりもグラフのほうがはるかに速くパターンを見つけられます。パフォーマンスを分析するとき、データをプロットすると、生の数値では見えなかった傾向、スパイク、異常がしばしば明らかになります。

**データをプロット可能にする**: デバッグのためにprint文やログ文を追加するときは、後で簡単にグラフ化できるように出力をフォーマットすることを考えてみてください。CSV形式のシンプルなタイムスタンプと値(`1705012345,42.5`)は、散文の文章よりもはるかにプロットしやすいものです。JSONで構造化されたログも、最小限の手間でパースしてプロットできます。言い換えれば、データは[整然(tidy)な形で](https://vita.had.co.nz/papers/tidy-data.pdf)ログに残しましょう。

**gnuplotによる手軽なプロット**: コマンドラインでの簡単なプロットには、[`gnuplot`](http://www.gnuplot.info/) を使ってデータファイルから直接グラフを生成できます。

```bash
# Plot a simple CSV with timestamp,value
gnuplot -e "set datafile separator ','; plot 'latency.csv' using 1:2 with lines"
```

**matplotlibとggplot2による反復的な探索**: より深い分析には、Pythonの [`matplotlib`](https://matplotlib.org/) とRの [`ggplot2`](https://ggplot2.tidyverse.org/) が反復的な探索を可能にします。一回限りのプロットとは異なり、これらのツールを使えば、仮説を検証するためにデータを素早くスライスしたり変換したりできます。ggplot2のファセットプロットは特に強力で、単一のデータセットをカテゴリごとに複数のサブプロットに分割することで(例えば、リクエストのレイテンシをエンドポイント別や時間帯別にファセット化する)、そうしなければ隠れたままだったパターンをあぶり出すことができます。

**ユースケースの例:**
- リクエストのレイテンシを時系列でプロットすると、生のパーセンタイルでは分からない周期的な遅延(ガベージコレクション、cronジョブ、トラフィックパターン)が明らかになる
- 成長するデータ構造への挿入時間を可視化すると、アルゴリズム的な計算量の問題が露わになることがある。ベクタへの挿入をプロットすると、内部の配列がサイズを倍にするタイミングで特徴的なスパイクが現れる
- メトリクスをさまざまな次元(リクエスト種別、ユーザーコホート、サーバー)でファセット化すると、「システム全体」の問題が実は特定のカテゴリだけに限られていることがしばしば分かる

## CPUプロファイラ

_プロファイラ_ という言葉が使われるとき、ほとんどの場合それは _CPUプロファイラ_ を指しています。主に2つのタイプがあります。

- **トレーシングプロファイラ**は、プログラムが行うすべての関数呼び出しを記録します
- **サンプリングプロファイラ**は、プログラムを定期的に(一般的には1ミリ秒ごとに)調べて、プログラムのスタックを記録します

サンプリングプロファイラはオーバーヘッドが小さく、本番環境での使用には一般にこちらが好まれます。

### perf: サンプリングプロファイラ

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) はLinuxの標準プロファイラです。再コンパイルなしにあらゆるプログラムをプロファイリングできます。

`perf stat` は、どこに時間が費やされているかの概要を手早く教えてくれます。

```bash
$ perf stat ./slow_program

 Performance counter stats for './slow_program':

         3,210.45 msec task-clock                #    0.998 CPUs utilized
               12      context-switches          #    3.738 /sec
                0      cpu-migrations            #    0.000 /sec
              156      page-faults               #   48.587 /sec
   12,345,678,901      cycles                    #    3.845 GHz
    9,876,543,210      instructions              #    0.80  insn per cycle
    1,234,567,890      branches                  #  384.532 M/sec
       12,345,678      branch-misses             #    1.00% of all branches
```

実世界のプログラムに対するプロファイラの出力は、大量の情報を含みます。人間は視覚的な生き物であり、大量の数値を読むのはかなり苦手です。[フレームグラフ(Flame graph)](https://www.brendangregg.com/flamegraphs.html)は、プロファイリングデータをはるかに理解しやすくする可視化手法です。

フレームグラフは、Y軸に関数呼び出しの階層を、X軸に所要時間に比例した幅を表示します。インタラクティブなので、クリックしてプログラムの特定の部分にズームインできます。

[![FlameGraph](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

`perf` のデータからフレームグラフを生成するには、次のようにします。

```bash
# Record profile
perf record -g ./my_program

# Generate flame graph (requires flamegraph scripts)
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

> インタラクティブなWebベースのフレームグラフビューアとして [Speedscope](https://www.speedscope.app/) を、包括的なシステムレベルの分析には [Perfetto](https://perfetto.dev/) を検討してみてください。

### ValgrindのCallgrind: トレーシングプロファイラ

[`callgrind`](https://valgrind.org/docs/manual/cl-manual.html) は、プログラムの呼び出し履歴と命令数を記録するプロファイリングツールです。サンプリングプロファイラとは異なり、正確な呼び出し回数を提供し、呼び出し元と呼び出し先の関係を表示できます。

```bash
# Run with callgrind
valgrind --tool=callgrind ./my_program

# Analyze with callgrind_annotate (text) or kcachegrind (GUI)
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid>
```

Callgrindはサンプリングプロファイラより低速ですが、正確な呼び出し回数を提供し、必要であればキャッシュの挙動のシミュレーション(`--cache-sim=yes`)もオプションで行えます。

> 特定の言語を使っているなら、より専門化されたプロファイラがあるかもしれません。例えば、Pythonには [`cProfile`](https://docs.python.org/3/library/profile.html) と [`py-spy`](https://github.com/benfred/py-spy) が、Goには [`go tool pprof`](https://pkg.go.dev/cmd/pprof) が、Rustには [`cargo-flamegraph`](https://github.com/flamegraph-rs/flamegraph)(実は任意のコンパイル済みプログラムで動きます!)があります。

## メモリプロファイラ

メモリプロファイラは、プログラムが時間とともにどのようにメモリを使うかを理解し、メモリリークを見つける助けになります。

### ValgrindのMassif

[`massif`](https://valgrind.org/docs/manual/ms-manual.html) はヒープメモリの使用状況をプロファイリングします。

```bash
valgrind --tool=massif ./my_program
ms_print massif.out.<pid>
```

これはヒープ使用量の時間変化を表示し、メモリリークや過剰なアロケーションの特定に役立ちます。

> Pythonでは、[`memory-profiler`](https://pypi.org/project/memory-profiler/) が行単位のメモリ使用量情報を提供します。

## ベンチマーク

異なる実装やツールのパフォーマンスを比較する必要があるとき、コマンドラインプログラムのベンチマークには [`hyperfine`](https://github.com/sharkdp/hyperfine) が最適です。

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

> Web開発では、ブラウザの開発者ツールに優れたプロファイラが含まれています。[Firefox Profiler](https://profiler.firefox.com/docs/) と [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/rendering-tools) のドキュメントを参照してください。

# 演習

## デバッグ

1. **ソートアルゴリズムのデバッグ**: 次の擬似コードはマージソートを実装していますが、バグが含まれています。好きな言語で実装した上で、デバッガ(gdb、lldb、pdb、またはIDEのデバッガ)を使ってバグを見つけ、修正してください。

   ```
   function merge_sort(arr):
       if length(arr) <= 1:
           return arr
       mid = length(arr) / 2
       left = merge_sort(arr[0..mid])
       right = merge_sort(arr[mid..end])
       return merge(left, right)

   function merge(left, right):
       result = []
       i = 0, j = 0
       while i < length(left) AND j < length(right):
           if left[i] <= right[j]:
               append result, left[i]
               i = i + 1
           else:
               append result, right[i]
               j = j + 1
       append remaining elements from left and right
       return result
   ```

   テストベクタ: `merge_sort([3, 1, 4, 1, 5, 9, 2, 6])` は `[1, 1, 2, 3, 4, 5, 6, 9]` を返すはずです。ブレークポイントを使ってmerge関数をステップ実行し、誤った要素が選択されている箇所を見つけてください。

1. [`rr`](https://rr-project.org/) をインストールし、逆方向デバッグを使ってデータ破壊のバグを見つけてください。このプログラムを `corruption.c` として保存してください。

   ```c
   #include <stdio.h>

   typedef struct {
       int id;
       int scores[3];
   } Student;

   Student students[2];

   void init() {
       students[0].id = 1001;
       students[0].scores[0] = 85;
       students[0].scores[1] = 92;
       students[0].scores[2] = 78;

       students[1].id = 1002;
       students[1].scores[0] = 90;
       students[1].scores[1] = 88;
       students[1].scores[2] = 95;
   }

   void curve_scores(int student_idx, int curve) {
       for (int i = 0; i < 4; i++) {
           students[student_idx].scores[i] += curve;
       }
   }

   int main() {
       init();
       printf("=== Initial state ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       curve_scores(0, 5);

       printf("\n=== After curving ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       if (students[1].id != 1002) {
           printf("\nERROR: Student 1's ID was corrupted! Expected 1002, got %d\n",
                  students[1].id);
           return 1;
       }
       return 0;
   }
   ```

   `gcc -g corruption.c -o corruption` でコンパイルして実行してください。Student 1のIDが破壊されますが、その破壊はStudent 0にしか触れていないはずの関数の中で起きています。`rr record ./corruption` と `rr replay` を使って犯人を見つけてください。`students[1].id` にウォッチポイントを設定し、破壊が起きた後に `reverse-continue` を使って、どの行がそれを上書きしたのかを正確に突き止めてください。

1. AddressSanitizerでメモリエラーをデバッグしてください。これを `uaf.c` として保存してください。

   ```c
   #include <stdlib.h>
   #include <string.h>
   #include <stdio.h>

   int main() {
       char *greeting = malloc(32);
       strcpy(greeting, "Hello, world!");
       printf("%s\n", greeting);

       free(greeting);

       greeting[0] = 'J';
       printf("%s\n", greeting);

       return 0;
   }
   ```

   まずサニタイザなしでコンパイルして実行してください: `gcc uaf.c -o uaf && ./uaf`。一見正常に動くように見えるかもしれません。次にAddressSanitizer付きでコンパイルしてください: `gcc -fsanitize=address -g uaf.c -o uaf && ./uaf`。エラーレポートを読んでください。ASanはどんなバグを見つけましたか? 指摘された問題を修正してください。

1. `strace`(Linux)または `dtruss`(macOS)を使って、`ls -l` のようなコマンドが発行するシステムコールをトレースしてください。どんなシステムコールを発行していますか? もっと複雑なプログラムもトレースして、どんなファイルを開いているか見てみましょう。

1. LLMを使って不可解なエラーメッセージのデバッグを手伝ってもらいましょう。コンパイラのエラー(特にC++のテンプレートやRustのもの)をコピーして、説明と修正を求めてみてください。`strace` やAddressSanitizerの出力の一部を入力してみるのも試してください。

## プロファイリング

1. `perf stat` を使って、好きなプログラムの基本的なパフォーマンス統計を取得してください。それぞれのカウンタは何を意味していますか?

1. `perf record` でプロファイリングしてください。これを `slow.c` として保存してください。

   ```c
   #include <math.h>
   #include <stdio.h>

   double slow_computation(int n) {
       double result = 0;
       for (int i = 0; i < n; i++) {
           for (int j = 0; j < 1000; j++) {
               result += sin(i * j) * cos(i + j);
           }
       }
       return result;
   }

   int main() {
       double r = 0;
       for (int i = 0; i < 100; i++) {
           r += slow_computation(1000);
       }
       printf("Result: %f\n", r);
       return 0;
   }
   ```

   デバッグシンボル付きでコンパイルしてください: `gcc -g -O2 slow.c -o slow -lm`。`perf record -g ./slow` を実行し、続いて `perf report` でどこに時間が費やされているかを見てください。flamegraphスクリプトを使ってフレームグラフの生成も試してみましょう。

1. `hyperfine` を使って、同じタスクの2つの異なる実装のベンチマークを取ってください(例: `find` と `fd`、`grep` と `ripgrep`、あるいは自分のコードの2つのバージョン)。

1. リソースを大量に消費するプログラムを実行しながら、`htop` を使ってシステムを監視してください。`taskset` を使ってプロセスが使用できるCPUを制限してみましょう: `taskset --cpu-list 0,2 stress -c 3`。なぜ `stress` は3つのCPUを使わないのでしょうか?

1. よくある問題として、リッスンしたいポートがすでに別のプロセスに取られている、というものがあります。そのプロセスを発見する方法を学びましょう。まず `python -m http.server 4444` を実行して、ポート4444で最小限のWebサーバーを起動します。別のターミナルで `ss -tlnp | grep 4444` を実行してプロセスを見つけます。`kill <PID>` でそれを終了させましょう。
