---
layout: lecture
title: "プログラムの内部調査"
presenter: Anish
date: 2019-01-29
order: 1
video:
  aspect: 62.5
  id: 74MhV-7hYzg
---

# デバッグ

printfデバッグでは不十分なとき: デバッガを使いましょう。

デバッガを使うと、プログラムの実行に介入して、次のようなことができます。

- プログラムが特定の行に到達したときに実行を停止する
- プログラムをステップ実行する
- 変数の値を調べる
- そのほか多くの高度な機能

## GDB/LLDB

[GDB](https://www.gnu.org/software/gdb/)と[LLDB](https://lldb.llvm.org/)。多くのC系言語をサポートしています。

[example.c](/2019/files/example.c)を見てみましょう。デバッグフラグを付けてコンパイルします: `gcc -g -o example example.c`。

GDBを開きます:

`gdb example`

いくつかのコマンド:

- `run`
- `b {name of function}` - ブレークポイントを設定する
- `b {file}:{line}` - ブレークポイントを設定する
- `c` - 実行を再開する
- `step` / `next` / `finish` - ステップイン / ステップオーバー / ステップアウト
- `p {variable}` - 変数の値を表示する
- `watch {expression}` - 式の値が変化したときに発動するウォッチポイントを設定する
- `rwatch {expression}` - 値が読み取られたときに発動するウォッチポイントを設定する
- `layout`

## PDB

[PDB](https://docs.python.org/3/library/pdb.html)はPythonのデバッガです。

PDBに入りたい場所に `import pdb; pdb.set_trace()` を挿入します。PDBは、いわば(GDBのような)デバッガとPythonシェルを組み合わせたものです。

## Webブラウザの開発者ツール

デバッガのもうひとつの例で、こちらはグラフィカルなインターフェースを備えています。

# strace

プログラムが発行するシステムコールを観察します: `strace {program}`。

# プロファイリング

プロファイリングの種類: CPU、メモリなど。

もっとも単純なプロファイラ: `time`。

## Go

CPUプロファイラを有効にしてテストコードを実行する: `go test -cpuprofile=cpu.out`

プロファイルを分析する: `go tool pprof -web cpu.out`

メモリプロファイラを有効にしてテストコードを実行する: `go test -memprofile=mem.out`

プロファイルを分析する: `go tool pprof -web mem.out`

## Perf

基本的なパフォーマンス統計: `perf stat {command}`

プロファイラを有効にしてプログラムを実行する: `perf record {command}`

プロファイルを分析する: `perf report`
