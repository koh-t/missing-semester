---
layout: lecture
title: "コードのパッケージングと配布"
description: >
  プロジェクトのパッケージング、環境、バージョニング、そしてライブラリ・アプリケーション・サービスのデプロイについて学びます。
thumbnail: /static/assets/thumbnails/2026/lec6.png
date: 2026-01-20
ready: true
video:
  aspect: 56.25
  id: KBMiB-8P4Ns
---

コードを意図したとおりに動かすのは難しいことですが、その同じコードを自分のマシンとは別のマシンで動かすのは、往々にしてもっと難しいものです。

コードを配布する(shipする)とは、自分が書いたコードを、あなたのコンピュータとまったく同じセットアップがなくても他の人が実行できる、利用可能な形に変換することを意味します。
コードの配布にはさまざまな形態があり、プログラミング言語、システムライブラリ、オペレーティングシステムの選択など、多くの要因に依存します。
また、何を作っているかにも依存します。ソフトウェアライブラリ、コマンドラインツール、Webサービスでは、それぞれ要件やデプロイの手順が異なります。
とはいえ、これらすべてのシナリオには共通のパターンがあります。それは、成果物 --- すなわち _アーティファクト_ --- が何であるかを定義し、それが周囲の環境についてどのような前提を置いているかを明確にする必要がある、ということです。

この講義では、以下の内容を扱います。

- [依存関係と環境](#dependencies--environments)
- [アーティファクトとパッケージング](#artifacts--packaging)
- [リリースとバージョニング](#releases--versioning)
- [再現性](#reproducibility)
- [VMとコンテナ](#vms--containers)
- [設定](#configuration)
- [サービスとオーケストレーション](#services--orchestration)
- [公開](#publishing)

これらの概念は、Pythonエコシステムの例を通して説明していきます。具体例があるほうが理解しやすいからです。他のプログラミング言語のエコシステムではツールこそ異なりますが、概念はおおむね共通です。

# 依存関係と環境 {#dependencies--environments}

現代のソフトウェア開発では、抽象化のレイヤーがいたるところに存在します。
プログラムは当然のように、処理のロジックを他のライブラリやサービスに委ねます。
しかし、これによってあなたのプログラムと、その動作に必要なライブラリとの間に _依存関係_ が生まれます。
たとえばPythonでWebサイトのコンテンツを取得するには、よく次のように書きます。

```python
import requests

response = requests.get("https://missing.csail.mit.edu")
```

しかし `requests` ライブラリはPythonランタイムに同梱されていないため、`requests` をインストールせずにこのコードを実行しようとすると、Pythonはエラーを出します。

```console
$ python fetch.py
Traceback (most recent call last):
  File "fetch.py", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
```

このライブラリを利用可能にするには、まず `pip install requests` を実行してインストールする必要があります。
`pip` は、Pythonというプログラミング言語がパッケージのインストール用に提供しているコマンドラインツールです。
`pip install requests` を実行すると、次の一連の処理が行われます。

1. Python Package Index([PyPI](https://pypi.org/))で requests を検索する
1. 実行中のプラットフォームに適したアーティファクトを探す
1. 依存関係を解決する --- `requests` ライブラリ自体も他のパッケージに依存しているため、インストーラーはすべての推移的依存関係について互換性のあるバージョンを見つけ、先にそれらをインストールする必要があります
1. アーティファクトをダウンロードし、展開して、ファイルシステム内の適切な場所にファイルをコピーする

```console
$ pip install requests
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl (64 kB)
Collecting charset-normalizer<4,>=2
  Downloading charset_normalizer-3.4.0-cp311-cp311-manylinux_x86_64.whl (142 kB)
Collecting idna<4,>=2.5
  Downloading idna-3.10-py3-none-any.whl (70 kB)
Collecting urllib3<3,>=1.21.1
  Downloading urllib3-2.2.3-py3-none-any.whl (126 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2024.8.30-py3-none-any.whl (167 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2024.8.30 charset-normalizer-3.4.0 idna-3.10 requests-2.32.3 urllib3-2.2.3
```

ここで、`requests` には `certifi` や `charset-normalizer` といった独自の依存関係があり、それらを先にインストールしないと `requests` をインストールできないことがわかります。
いったんインストールされれば、Pythonランタイムはインポート時にこのライブラリを見つけられるようになります。

```console
$ python -c 'import requests; print(requests.__path__)'
['/usr/local/lib/python3.11/dist-packages/requests']

$ pip list | grep requests
requests        2.32.3
```

ライブラリのインストールと公開のためのツールや慣習、プラクティスは、プログラミング言語ごとに異なります。
Rustのような一部の言語ではツールチェーンが統一されており、`cargo` がビルド、テスト、依存関係の管理、公開までを担います。
一方、Pythonのような言語では、統一は仕様のレベルで行われています。単一のツールではなく、パッケージングの仕組みを定義した標準仕様があり、それぞれのタスクについて複数の競合するツールが存在できるのです(`pip` 対 [`uv`](https://docs.astral.sh/uv/)、`setuptools` 対 [`hatch`](https://hatch.pypa.io/) 対 [`poetry`](https://python-poetry.org/) など)。
また、LaTeXのようなエコシステムでは、TeX LiveやMacTeXといったディストリビューションに何千ものパッケージがあらかじめ同梱されています。

依存関係を導入すると、依存関係の競合(コンフリクト)も持ち込むことになります。
競合は、複数のプログラムが同じ依存関係の互換性のないバージョンを要求するときに発生します。
たとえば、`tensorflow==2.3.0` が `numpy>=1.16.0,<1.19.0` を要求し、`pandas==1.2.0` が `numpy>=1.16.5` を要求する場合、`numpy>=1.16.5,<1.19.0` を満たすバージョンであれば有効です。
しかし、プロジェクト内の別のパッケージが `numpy>=1.19` を要求すると、すべての制約を満たす有効なバージョンが存在しない競合状態になります。

このように、複数のパッケージが共有する依存関係について互いに相容れないバージョンを要求する状況は、一般に _依存関係地獄(dependency hell)_ と呼ばれます。
競合に対処する方法のひとつは、各プログラムの依存関係をそれぞれ独立した _環境_ に分離することです。
Pythonでは、次のように実行して仮想環境を作成します。

```console
$ which python
/usr/bin/python
$ pwd
/home/missingsemester
$ python -m venv venv
$ source venv/bin/activate
$ which python
/home/missingsemester/venv/bin/python
$ which pip
/home/missingsemester/venv/bin/pip
$ python -c 'import requests; print(requests.__path__)'
['/home/missingsemester/venv/lib/python3.11/site-packages/requests']

$ pip list
Package Version
------- -------
pip     24.0
```

環境とは、独自のインストール済みパッケージ一式を持つ、言語ランタイムの完全に独立したコピーのようなものだと考えることができます。
この仮想環境(venv)は、インストールされた依存関係をグローバルなPythonインストールから分離します。
プロジェクトごとに、必要な依存関係を含む仮想環境を用意するのが良いプラクティスです。

> 多くの現代的なオペレーティングシステムにはPythonのようなプログラミング言語ランタイムがあらかじめインストールされていますが、OSが自身の機能のためにそれらに依存している可能性があるため、これらのインストールを変更するのは賢明ではありません。代わりに別の環境を使うようにしましょう。

一部の言語では、インストールのプロトコルはツールではなく仕様として定義されています。
Pythonでは、[PEP 517](https://peps.python.org/pep-0517/) がビルドシステムのインターフェースを定義し、[PEP 621](https://peps.python.org/pep-0621/) がプロジェクトのメタデータを `pyproject.toml` にどう格納するかを規定しています。
これにより、開発者は `pip` を改良して `uv` のようなより最適化されたツールを生み出せるようになりました。`uv` のインストールは `pip install uv` を実行するだけです。

`pip` の代わりに `uv` を使っても同じインターフェースに従いますが、大幅に高速です。

```console
$ uv pip install requests
Resolved 5 packages in 12ms
Prepared 5 packages in 0.45ms
Installed 5 packages in 8ms
 + certifi==2024.8.30
 + charset-normalizer==3.4.0
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

> インストール時間が劇的に短縮されるため、可能な限り `pip` ではなく `uv pip` を使うことを強くおすすめします。

依存関係の分離だけでなく、環境を使うことで、プログラミング言語ランタイム自体の異なるバージョンを持つこともできます。

```console
$ uv venv --python 3.12 venv312
Using CPython 3.12.7
Creating virtual environment at: venv312

$ source venv312/bin/activate && python --version
Python 3.12.7

$ uv venv --python 3.11 venv311
Using CPython 3.11.10
Creating virtual environment at: venv311

$ source venv311/bin/activate && python --version
Python 3.11.10
```

これは、複数のPythonバージョンでコードをテストする必要があるときや、プロジェクトが特定のバージョンを要求するときに役立ちます。

> プログラミング言語によっては、手動で作成しなくても各プロジェクトが自動的に依存関係用の独自の環境を持つものもありますが、原理は同じです。また、最近のほとんどの言語には、1つのシステム上で言語の複数バージョンを管理し、プロジェクトごとに使用するバージョンを指定する仕組みがあります。

# アーティファクトとパッケージング {#artifacts--packaging}

ソフトウェア開発では、ソースコードとアーティファクトを区別します。開発者が書いたり読んだりするのがソースコードであるのに対し、アーティファクトはそのソースコードから生成される、パッケージ化された配布可能な出力物であり、そのままインストールやデプロイができる状態のものです。
アーティファクトは、実行するコードファイル1つのような単純なものから、アプリケーションに必要なあらゆるものを含んだ仮想マシン(VM)全体のような複雑なものまでありえます。
カレントディレクトリにPythonファイル `greet.py` がある、次の例を考えてみましょう。

```console
$ cat greet.py
def greet(name):
    print(f"Hello, {name}!")

$ python -c "from greet import greet; greet('World')"
Hello, World!

$ cd /tmp
$ python -c "from greet import greet; greet('World')"
ModuleNotFoundError: No module named 'greet'
```

別のディレクトリに移動するとインポートが失敗するのは、Pythonが特定の場所(カレントディレクトリ、インストール済みパッケージ、`PYTHONPATH` に含まれるパス)しかモジュールを探さないからです。パッケージングは、コードを既知の場所にインストールすることでこの問題を解決します。

Pythonでは、ライブラリのパッケージングとは、`pip` や `uv` のようなパッケージインストーラーが必要なファイルをインストールするために利用できるアーティファクトを生成することを意味します。
Pythonのアーティファクトは _wheel_ と呼ばれ、パッケージのインストールに必要なすべての情報 --- コードファイル、パッケージに関するメタデータ(名前、バージョン、依存関係)、環境内のどこにファイルを配置するかの指示 --- を含んでいます。
アーティファクトをビルドするには、プロジェクトの詳細、必要な依存関係、パッケージのバージョンなどの情報を記述したプロジェクトファイル(マニフェストとも呼ばれます)を書く必要があります。Pythonでは、この目的に `pyproject.toml` を使います。

> `pyproject.toml` が現代的で推奨される方法です。`requirements.txt` や `setup.py` といった以前のパッケージング方法もまだサポートされていますが、可能な限り `pyproject.toml` を使うべきです。

コマンドラインツールも提供するライブラリのための、最小限の `pyproject.toml` の例を示します。

```toml
[project]
name = "greeting"
version = "0.1.0"
description = "A simple greeting library"
dependencies = ["typer>=0.9"]

[project.scripts]
greet = "greeting:cli"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

`typer` ライブラリは、最小限の定型コードでコマンドラインインターフェースを作れる人気のPythonパッケージです。

対応する `greeting.py` は次のとおりです。

```python
import typer


def greet(name: str) -> None:
    print(f"Hello, {name}!")


def cli() -> None:
    typer.run(greet)


if __name__ == "__main__":
    cli()
```

このファイルがあれば、wheelをビルドできます。

```console
$ uv build
Building source distribution...
Building wheel from source distribution...
Successfully built dist/greeting-0.1.0.tar.gz
Successfully built dist/greeting-0.1.0-py3-none-any.whl

$ ls dist/
greeting-0.1.0-py3-none-any.whl
greeting-0.1.0.tar.gz
```

`.whl` ファイルがwheel(特定の構造を持つzipアーカイブ)で、`.tar.gz` はソースからビルドする必要があるシステム向けのソース配布物(source distribution)です。

wheelの中身を調べて、何がパッケージングされているかを確認することもできます。

```console
$ unzip -l dist/greeting-0.1.0-py3-none-any.whl
Archive:  dist/greeting-0.1.0-py3-none-any.whl
  Length      Date    Time    Name
---------  ---------- -----   ----
      150  2024-01-15 10:30   greeting.py
      312  2024-01-15 10:30   greeting-0.1.0.dist-info/METADATA
       92  2024-01-15 10:30   greeting-0.1.0.dist-info/WHEEL
        9  2024-01-15 10:30   greeting-0.1.0.dist-info/top_level.txt
      435  2024-01-15 10:30   greeting-0.1.0.dist-info/RECORD
---------                     -------
      998                     5 files
```

このwheelを他の人に渡せば、その人は次のように実行してインストールできます。

```console
$ uv pip install ./greeting-0.1.0-py3-none-any.whl
$ greet Alice
Hello, Alice!
```

これにより、先ほどビルドしたライブラリが、`greet` というCLIツールも含めて相手の環境にインストールされます。

この方法には限界もあります。特に、ライブラリがプラットフォーム固有のライブラリ(たとえばGPUアクセラレーション用のCUDA)に依存している場合、アーティファクトはそれらの特定のライブラリがインストールされたシステムでしか動作せず、異なるプラットフォーム(Linux、macOS、Windows)やアーキテクチャ(x86、ARM)向けに別々のwheelをビルドする必要が出てくるかもしれません。


ソフトウェアをインストールする際には、ソースからのインストールとビルド済みバイナリのインストールという重要な区別があります。ソースからのインストールとは、元のコードをダウンロードして自分のマシンでコンパイルすることを意味します。これにはコンパイラとビルドツールがインストールされている必要があり、大規模なプロジェクトではかなりの時間がかかることがあります。

ビルド済みバイナリのインストールとは、すでに誰かがコンパイルしたアーティファクトをダウンロードすることです。より高速で簡単ですが、バイナリが自分のプラットフォームとアーキテクチャに一致している必要があります。
たとえば、[ripgrepのリリースページ](https://github.com/BurntSushi/ripgrep/releases)には、Linux(x86_64、ARM)、macOS(Intel、Apple Silicon)、Windows向けのビルド済みバイナリが公開されています。


# リリースとバージョニング {#releases--versioning}

コードは継続的なプロセスで作られますが、リリースは離散的な単位で行われます。
ソフトウェア開発では、開発(dev)環境と本番(production)環境が明確に区別されます。
コードは、本番環境に _配布(ship)_ される前に、開発環境で動作することが実証されている必要があります。
リリースプロセスには、テスト、依存関係の管理、バージョニング、設定、デプロイ、公開など、多くのステップが含まれます。


ソフトウェアライブラリは静的なものではなく、修正や新機能を取り込みながら時間とともに進化していきます。
この進化は、ある時点でのライブラリの状態に対応する離散的なバージョン識別子によって追跡されます。
ライブラリの挙動の変更には、重要度の低い機能を修正するパッチから、機能を拡張する新機能、後方互換性を破壊する変更まで、さまざまなものがあります。
あるバージョンがどのような変更をもたらすかを記録するのがチェンジログ(changelog)です。これは、ソフトウェア開発者が新しいリリースに伴う変更を伝えるために使う文書です。

しかし、ひとつひとつの依存関係の変更を逐一追いかけるのは現実的ではありません。推移的依存関係 --- すなわち依存関係の依存関係 --- まで考えればなおさらです。

> `uv tree` を使うと、プロジェクトの依存関係ツリー全体を可視化できます。すべてのパッケージとその推移的依存関係がツリー形式で表示されます。

この問題を単純化するために、ソフトウェアのバージョン付けに関する慣習が存在します。最も普及しているもののひとつが[セマンティックバージョニング](https://semver.org/)(Semantic Versioning、SemVer)です。
セマンティックバージョニングでは、バージョンは MAJOR.MINOR.PATCH という形式の識別子を持ち、それぞれの値は整数をとります。要点をまとめると、アップグレードのルールは次のとおりです。

- PATCH(例: 1.2.3 → 1.2.4)はバグ修正のみを含み、完全に後方互換であるべき
- MINOR(例: 1.2.3 → 1.3.0)は後方互換な形で新機能を追加する
- MAJOR(例: 1.2.3 → 2.0.0)はコードの修正が必要になりうる破壊的変更を示す

> これは簡略化した説明です。たとえば 0.1.3 から 0.2.0 への更新がなぜ破壊的変更を引き起こしうるのか、1.0.0-rc.1 が何を意味するのかを理解するには、SemVerの仕様全文を読むことをおすすめします。
Pythonのパッケージングはセマンティックバージョニングをネイティブにサポートしているため、依存関係のバージョンを指定する際にさまざまな指定子(specifier)を使えます。

`pyproject.toml` では、依存関係の互換バージョンの範囲を制約する方法がいくつかあります。

```toml
[project]
dependencies = [
    "requests==2.32.3",  # Exact version - only this specific version
    "click>=8.0",        # Minimum version - 8.0 or newer
    "numpy>=1.24,<2.0",  # Range - at least 1.24 but less than 2.0
    "pandas~=2.1.0",     # Compatible release - >=2.1.0 and <2.2.0
]
```

バージョン指定子は多くのパッケージマネージャー(npm、cargoなど)に存在しますが、正確なセマンティクスはそれぞれ異なります。`~=` 演算子はPythonの「互換リリース(compatible release)」演算子で、`~=2.1.0` は「2.1.0と互換性のある任意のバージョン」を意味し、`>=2.1.0` かつ `<2.2.0` に相当します。これは、SemVerの互換性の概念に従うnpmやcargoのキャレット(`^`)演算子とおおよそ同等です。

すべてのソフトウェアがセマンティックバージョニングを使っているわけではありません。よくある代替がカレンダーバージョニング(Calendar Versioning、CalVer)で、バージョンは意味論的な意味ではなくリリース日に基づきます。たとえばUbuntuは `24.04`(2024年4月)や `24.10`(2024年10月)といったバージョンを使います。CalVerはリリースがどれくらい古いかをひと目で把握できますが、互換性については何も伝えません。最後に、セマンティックバージョニングも万能ではなく、メンテナが意図せずマイナーリリースやパッチリリースに破壊的変更を持ち込んでしまうこともあります。


# 再現性 {#reproducibility}

現代のソフトウェア開発では、あなたが書くコードは膨大な数の抽象化レイヤーの上に載っています。
これには、プログラミング言語ランタイム、サードパーティライブラリ、オペレーティングシステム、さらにはハードウェアそのものまで含まれます。
これらのレイヤーのいずれかに違いがあれば、コードの挙動が変わったり、意図したとおりに動かなくなったりする可能性があります。
さらに言えば、基盤となるハードウェアの違いでさえ、ソフトウェアを配布する能力に影響します。

ライブラリをピン留め(pinning)するとは、範囲ではなく正確なバージョンを指定すること、たとえば `requests>=2.0` ではなく `requests==2.32.3` と指定することを指します。

パッケージマネージャーの仕事のひとつは、依存関係 --- そして推移的依存関係 --- が提示するすべての制約を考慮し、すべての制約を満たす有効なバージョンのリストを生成することです。
この具体的なバージョンのリストは、再現性のためにファイルに保存できます。こうしたファイルは _ロックファイル(lock file)_ と呼ばれます。

```console
$ uv lock
Resolved 12 packages in 45ms

$ cat uv.lock | head -20
version = 1
requires-python = ">=3.11"

[[package]]
name = "certifi"
version = "2024.8.30"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/...", hash = "sha256:..." }
wheels = [
    { url = "https://files.pythonhosted.org/...", hash = "sha256:..." },
]
...
```

依存関係のバージョニングと再現性を扱ううえで決定的に重要な区別のひとつが、ライブラリとアプリケーション/サービスの違いです。
ライブラリは、独自の依存関係を持ちうる他のコードからインポートされて使われることを意図しているため、過度に厳しいバージョン制約を指定すると、利用者の他の依存関係と競合を起こす可能性があります。
対照的に、アプリケーションやサービスはソフトウェアの最終消費者であり、通常はプログラミングインターフェースではなく、ユーザーインターフェースやAPIを通じて機能を公開します。
ライブラリでは、パッケージエコシステム全体との互換性を最大化するためにバージョン範囲を指定するのが良いプラクティスです。アプリケーションでは、正確なバージョンをピン留めすることで再現性が保証されます --- アプリケーションを実行する全員が、まったく同じ依存関係を使うことになります。


最大限の再現性が求められるプロジェクトのために、[Nix](https://nixos.org/) や [Bazel](https://bazel.build/) のようなツールは _ハーメチック(hermetic)_ ビルドを提供します。これは、コンパイラ、システムライブラリ、さらにはビルド環境そのものを含むすべての入力がピン留めされ、コンテンツアドレス化されているビルドです。これにより、いつどこでビルドを実行しても、ビットレベルで同一の出力が保証されます。

> NixOSを使えばコンピュータのインストール全体を管理することもでき、自分のコンピュータのセットアップの新しいコピーを簡単に立ち上げ、その完全な構成をバージョン管理された設定ファイルで管理できます。

ソフトウェア開発における終わりのない緊張関係として、新しいソフトウェアバージョンは意図的か否かにかかわらず破壊をもたらす一方で、古いソフトウェアバージョンは時間とともにセキュリティ脆弱性で危険にさらされていく、というものがあります。
これに対処するには、新しいソフトウェアバージョンに対してアプリケーションをテストする継続的インテグレーション(CI)パイプラインを使い([コード品質とCI](/2026/code-quality/) の講義で詳しく見ます)、[Dependabot](https://github.com/dependabot) のような、依存関係の新バージョンのリリースを検出する自動化の仕組みを整えておくことが有効です。

CIによるテストがあっても、ソフトウェアバージョンのアップグレード時には問題が起こります。多くの場合、開発環境と本番環境の避けがたい不一致が原因です。
そのような状況で最善の策は、_ロールバック_ 計画を用意しておくことです。つまり、バージョンアップグレードを取り消し、動作が確認されているバージョンを再デプロイするのです。

# VMとコンテナ {#vms--containers}

より複雑な依存関係に頼るようになると、コードの依存関係がパッケージマネージャーの扱える範囲を超えていく可能性が高くなります。
よくある理由のひとつは、特定のシステムライブラリやハードウェアドライバとやり取りする必要があることです。
たとえば科学計算やAIの分野では、プログラムがGPUハードウェアを利用するために専用のライブラリやドライバを必要とすることがよくあります。
多くのシステムレベルの依存関係(GPUドライバ、特定バージョンのコンパイラ、OpenSSLのような共有ライブラリ)は、依然としてシステム全体へのインストールを必要とします。

伝統的に、このより広い依存関係の問題は仮想マシン(VM)で解決されてきました。
VMはコンピュータ全体を抽象化し、専用のオペレーティングシステムを持つ完全に隔離された環境を提供します。
より現代的なアプローチがコンテナです。コンテナはアプリケーションをその依存関係、ライブラリ、ファイルシステムとともにパッケージ化しますが、コンピュータ全体を仮想化するのではなく、ホストのオペレーティングシステムのカーネルを共有します。
コンテナはカーネルを共有するためVMより軽量で、起動が速く、実行効率も高くなります。

最も人気のあるコンテナプラットフォームは [Docker](https://www.docker.com/) です。Dockerは、コンテナのビルド、配布、実行の標準化された方法を導入しました。内部的には、Dockerはコンテナランタイムとしてcontainerdを使用しています。これは業界標準であり、Kubernetesなどの他のツールも使用しています。

コンテナの実行は簡単です。たとえば、コンテナ内でPythonインタープリタを実行するには `docker run` を使います(`-it` フラグはコンテナをターミナル付きの対話モードにします。終了するとコンテナは停止します)。

```console
$ docker run -it python:3.12 python
Python 3.12.7 (main, Nov  5 2024, 02:53:25) [GCC 12.2.0] on linux
>>> print("Hello from inside a container!")
Hello from inside a container!
```

実際には、プログラムがファイルシステム全体に依存していることもあります。
これを乗り越えるために、アプリケーションのファイルシステム全体をアーティファクトとして持ち運ぶコンテナイメージを使うことができます。
コンテナイメージはプログラム的に作成されます。Dockerでは、Dockerfileという構文を使って、イメージの依存関係、システムライブラリ、設定を正確に指定します。

```dockerfile
FROM python:3.12
RUN apt-get update
RUN apt-get install -y gcc
RUN apt-get install -y libpq-dev
RUN pip install numpy
RUN pip install pandas
COPY . /app
WORKDIR /app
RUN pip install .
```

重要な区別として、Dockerの**イメージ**はパッケージ化されたアーティファクト(いわばテンプレート)であり、**コンテナ**はそのイメージの実行中のインスタンスです。同じイメージから複数のコンテナを実行できます。イメージはレイヤー構造でビルドされ、Dockerfile内の各命令(`FROM`、`RUN`、`COPY` など)が新しいレイヤーを作ります。Dockerはこれらのレイヤーをキャッシュするため、Dockerfileの1行を変更した場合、そのレイヤーとそれ以降のレイヤーだけを再ビルドすれば済みます。

先ほどのDockerfileにはいくつか問題があります。slim版ではなくフルサイズのPythonイメージを使っている、別々の `RUN` コマンドを実行して不要なレイヤーを作っている、バージョンがピン留めされていない、パッケージマネージャーのキャッシュを削除しておらず不要なファイルを含めてしまっている、という点です。他のよくある間違いとしては、コンテナをrootで実行してしまう危険な運用や、レイヤーにシークレットをうっかり埋め込んでしまうことが挙げられます。

改善版は次のとおりです。

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev && \
    rm -rf /var/lib/apt/lists/*
WORKDIR /app
ENV PATH="/app/.venv/bin:$PATH"
COPY pyproject.toml uv.lock ./
RUN uv sync --locked --no-dev --no-install-project
COPY . .
RUN uv sync --locked --no-dev
```

この例では、`uv` をソースからインストールする代わりに、`ghcr.io/astral-sh/uv:latest` イメージからビルド済みバイナリをコピーしています。これは _ビルダー(builder)_ パターンとして知られています。このパターンを使えば、コードのコンパイルに必要なツール一式を持ち運ぶ必要はなく、アプリケーションの実行に必要な最終的なバイナリ(この場合は `uv`)だけを含めれば済みます。

Dockerには知っておくべき重要な制限があります。第一に、コンテナイメージは多くの場合プラットフォーム固有です。`linux/amd64` 向けにビルドされたイメージは、`linux/arm64`(Apple SiliconのMac)ではエミュレーションなしにネイティブ実行できず、エミュレーションは低速です。第二に、DockerコンテナはLinuxカーネルを必要とするため、macOSやWindowsでは、Dockerは実際には内部で軽量のLinux VMを動かしており、オーバーヘッドが生じます。第三に、Dockerの分離はVMより弱いものです。コンテナはホストのカーネルを共有するため、マルチテナント環境ではセキュリティ上の懸念になります。

> 最近では、[nix flakes](https://serokell.io/blog/practical-nix-flakes) を通じて、「システム全体」のライブラリやアプリケーションでさえプロジェクトごとにnixで管理するプロジェクトが増えています。

# 設定 {#configuration}

ソフトウェアは本質的に設定可能なものです。[コマンドライン環境](/2026/command-line-environment/)の講義では、プログラムがフラグ、環境変数、あるいは設定ファイル(いわゆるdotfiles)を通じてオプションを受け取るのを見ました。これはより複雑なアプリケーションでも同様で、大規模に設定を管理するための確立されたパターンが存在します。
ソフトウェアの設定はコードに埋め込むのではなく、実行時に与えるべきです。
よく使われる方法として、環境変数と設定ファイルがあります。

環境変数で設定されるアプリケーションの例を示します。

```python
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///local.db")
DEBUG = os.environ.get("DEBUG", "false").lower() == "true"
API_KEY = os.environ["API_KEY"]  # Required - will raise if not set
```

アプリケーションは設定ファイルを通じて設定することもできます(たとえば `yaml.load` で設定を読み込むPythonプログラム)。`config.yaml` の例:

```yaml
database:
  url: "postgresql://localhost/myapp"
  pool_size: 5
server:
  host: "0.0.0.0"
  port: 8080
  debug: false
```

設定について考える際の良い経験則は、同じコードベースを、設定の変更だけで --- コードの変更は一切なしに --- 異なる環境(開発、ステージング、本番)にデプロイできるべきだ、というものです。

数ある設定項目の中には、APIキーのような機密データが含まれることがよくあります。
シークレットは誤って漏らさないよう慎重に扱う必要があり、バージョン管理に含めてはいけません。


# サービスとオーケストレーション {#services--orchestration}

現代のアプリケーションが単独で存在することはめったにありません。典型的なWebアプリケーションには、永続的なストレージのためのデータベース、パフォーマンスのためのキャッシュ、バックグラウンドタスクのためのメッセージキュー、その他さまざまな支援サービスが必要になるでしょう。すべてを単一のモノリシックなアプリケーションに詰め込むのではなく、現代的なアーキテクチャでは、機能を個別に開発・デプロイ・スケールできる別々のサービスに分解することがよくあります。

例として、アプリケーションにキャッシュを導入すると有益だと判断した場合、自前で実装する代わりに、[Redis](https://redis.io/) や [Memcached](https://memcached.org/) のような実績のある既存のソリューションを活用できます。
Redisをコンテナの一部としてビルドし、アプリケーションの依存関係に埋め込むこともできますが、それはRedisとアプリケーションのすべての依存関係を整合させることを意味し、困難、あるいは不可能でさえあるかもしれません。
その代わりにできるのは、各アプリケーションをそれぞれ独自のコンテナとして別々にデプロイすることです。
これは一般にマイクロサービスアーキテクチャと呼ばれ、各コンポーネントが独立したサービスとして動作し、ネットワーク越しに(通常はHTTP APIを介して)通信します。

[Docker Compose](https://docs.docker.com/compose/) は、マルチコンテナアプリケーションを定義・実行するためのツールです。コンテナを個別に管理する代わりに、すべてのサービスを1つのYAMLファイルで宣言し、まとめてオーケストレーションします。これで、アプリケーション全体は複数のコンテナで構成されるようになります。

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - REDIS_URL=redis://cache:6379
    depends_on:
      - cache

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

`docker compose up` を実行すると両方のサービスが一緒に起動し、Webアプリケーションはホスト名 `cache` を使ってRedisに接続できます(Dockerの内部DNSがサービス名を自動的に解決します)。
Docker Composeを使えば、1つ以上のサービスをどうデプロイしたいかを宣言でき、それらをまとめて起動し、サービス間のネットワークを設定し、データ永続化のための共有ボリュームを管理するといったオーケストレーションを担ってくれます。

本番デプロイでは、docker composeのサービスをブート時に自動起動させ、障害時には再起動させたいことがよくあります。よくあるアプローチは、systemdを使ってdocker composeのデプロイを管理することです。

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target
```

このsystemdユニットファイルにより、システムのブート時に(Dockerの準備が整った後で)アプリケーションが起動し、`systemctl start myapp`、`systemctl stop myapp`、`systemctl status myapp` といった標準的な操作が使えるようになります。

デプロイ要件がより複雑になるにつれ --- 複数マシンにまたがるスケーラビリティ、サービスがクラッシュしたときの耐障害性、高可用性の保証が必要になると --- 組織はKubernetes(k8s)のような高度なコンテナオーケストレーションプラットフォームに移行します。Kubernetesは、マシンのクラスタ全体で何千ものコンテナを管理できます。とはいえ、Kubernetesは学習コストが高く、運用上のオーバーヘッドも大きいため、小規模なプロジェクトには過剰であることが多いです。

このようなマルチコンテナ構成が可能なのは、現代のサービスがHTTP REST APIをはじめとする標準化されたAPIで互いに通信していることが一因です。たとえば、プログラムがOpenAIやAnthropicのようなLLMプロバイダーとやり取りするときは、内部的には彼らのサーバーにHTTPリクエストを送信し、レスポンスをパースしているのです。

```console
$ curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2023-06-01" \
    -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 256,
         "messages": [{"role": "user", "content": "Explain containers vs VMs in one sentence."}]}'
```

# 公開 {#publishing}

コードが動くことを示せたら、他の人がダウンロードしてインストールできるように配布したくなるかもしれません。
配布にはさまざまな形態があり、扱っているプログラミング言語や環境と本質的に結びついています。

最も単純な配布の形態は、アーティファクトをアップロードして、人々がダウンロードしてローカルにインストールできるようにすることです。
これは今でも一般的で、たとえば [Ubuntuのパッケージアーカイブ](http://archive.ubuntu.com/ubuntu/pool/main/) のような場所で見られます。これは本質的に `.deb` ファイルのHTTPディレクトリリスティングです。

今日では、GitHubがソースコードとアーティファクトを公開するためのデファクトのプラットフォームになっています。
ソースコードが公開されていることが多いのに加えて、GitHub Releasesを使えば、メンテナはタグ付けされたバージョンにビルド済みバイナリやその他のアーティファクトを添付できます。


パッケージマネージャーには、GitHubから直接インストールできるものもあります。ソースからでも、ビルド済みwheelからでも可能です。

```console
# Install from source (will clone and build)
$ pip install git+https://github.com/psf/requests.git

# Install from a specific tag/branch
$ pip install git+https://github.com/psf/requests.git@v2.32.3

# Install a wheel directly from a GitHub release
$ pip install https://github.com/user/repo/releases/download/v1.0/package-1.0-py3-none-any.whl
```

実際、Goのような一部の言語は分散型の配布モデルを採用しています。中央のパッケージリポジトリではなく、Goモジュールはソースコードリポジトリから直接配布されます。
`github.com/gorilla/mux` のようなモジュールパスはコードの所在を示し、`go get` はそこから直接取得します。しかし、`pip`、`cargo`、`brew` のようなほとんどのパッケージマネージャーは、配布とインストールを容易にするため、パッケージ化済みプロジェクトの中央インデックスを持っています。次を実行すると

```console
$ uv pip install requests --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: requests==2.32.5 [compatible] (requests-2.32.5-py3-none-any.whl)
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl.metadata
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl
```

`requests` のwheelをどこから取得しているかがわかります。ファイル名の `py3-none-any` に注目してください。これは、このwheelがあらゆるPython 3バージョン、あらゆるOS、あらゆるアーキテクチャで動作することを意味します。コンパイル済みコードを含むパッケージでは、wheelはプラットフォーム固有になります。

```console
$ uv pip install numpy --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: numpy==2.2.1 [compatible] (numpy-2.2.1-cp312-cp312-macosx_14_0_arm64.whl)
```

ここで `cp312-cp312-macosx_14_0_arm64` は、このwheelがARM64(Apple Silicon)のmacOS 14以降上のCPython 3.12専用であることを示しています。別のプラットフォームを使っている場合、`pip` は別のwheelをダウンロードするか、ソースからビルドします。

逆に、私たちが作成したパッケージを人々に見つけてもらうには、こうしたレジストリのいずれかに公開する必要があります。
Pythonの主要なレジストリは [Python Package Index (PyPI)](https://pypi.org) です。
インストールと同様に、パッケージの公開にも複数の方法があります。`uv publish` コマンドは、パッケージをPyPIにアップロードするための現代的なインターフェースを提供します。

```console
$ uv publish --publish-url https://test.pypi.org/legacy/
Publishing greeting-0.1.0.tar.gz
Publishing greeting-0.1.0-py3-none-any.whl
```

ここでは [TestPyPI](https://test.pypi.org) を使っています。これは、本物のPyPIを汚染することなく公開ワークフローをテストするための、独立したパッケージレジストリです。アップロードしたら、TestPyPIからインストールできます。

```console
$ uv pip install --index-url https://test.pypi.org/simple/ greeting
```

ソフトウェアを公開する際の重要な考慮事項が信頼です。ダウンロードしたパッケージが本当にあなたのものであり、改ざんされていないことを、ユーザーはどうやって検証すればよいのでしょうか。パッケージレジストリはチェックサムを使って完全性を検証し、一部のエコシステムでは、作者であることの暗号学的な証明を提供するパッケージ署名をサポートしています。

言語ごとに独自のパッケージレジストリがあります。Rustの [crates.io](https://crates.io)、JavaScriptの [npm](https://www.npmjs.com)、Rubyの [RubyGems](https://rubygems.org)、そしてコンテナイメージの [Docker Hub](https://hub.docker.com) などです。一方、プライベートまたは社内向けのパッケージについては、組織が独自のパッケージリポジトリ(プライベートPyPIサーバーやプライベートDockerレジストリなど)をデプロイしたり、クラウドプロバイダーのマネージドソリューションを使ったりすることがよくあります。

Webサービスをインターネットにデプロイするには、追加のインフラが必要です。ドメイン名の登録、ドメインをサーバーに向けるDNS設定、そして多くの場合、HTTPSの処理とトラフィックのルーティングを行うnginxのようなリバースプロキシです。ドキュメントや静的サイトのようなシンプルな用途には、[GitHub Pages](https://pages.github.com/) がリポジトリから直接無料ホスティングを提供しています。

<!--
## Documentation

So far we have emphasized the deliverable _artifact_ as the main output of packaging and shipping code.
In addition to the artifact, we need to document for users the code's functionality, installation instructions, and usage examples.

Tools like [Sphinx](https://www.sphinx-doc.org/) (Python) and [MkDocs](https://www.mkdocs.org/) can automatically generate browsable documentation from docstrings and markdown files, often hosted on services like [Read the Docs](https://readthedocs.org/).
For HTTP-based APIs, the [OpenAPI specification](https://www.openapis.org/) (formerly Swagger) provides a standard format for describing API endpoints, which tools can use to generate interactive documentation and client libraries automatically. -->


# 演習

1. `printenv` で環境をファイルに保存し、venvを作成して有効化(activate)し、もう一度 `printenv` を別のファイルに保存して `diff before.txt after.txt` を実行してみましょう。環境の何が変わったでしょうか? なぜシェルはvenvのほうを優先するのでしょうか?(ヒント: 有効化の前後で `$PATH` を見比べてみましょう。)`which deactivate` を実行し、deactivateというbash関数が何をしているのか考えてみましょう。
1. `pyproject.toml` を持つPythonパッケージを作成し、仮想環境にインストールしてみましょう。ロックファイルを作成し、その中身を調べてみましょう。
1. Dockerをインストールし、docker composeを使ってMissing Semesterの講義Webサイトをローカルでビルドしてみましょう。
1. シンプルなPythonアプリケーションのためのDockerfileを書いてみましょう。次に、そのアプリケーションをRedisキャッシュと一緒に実行する `docker-compose.yml` を書いてみましょう。
1. PythonパッケージをTestPyPIに公開してみましょう(共有する価値があるものでない限り、本物のPyPIには公開しないでください!)。次に、そのパッケージを含むDockerイメージをビルドし、`ghcr.io` にプッシュしてみましょう。
1. [GitHub Pages](https://docs.github.com/en/pages/quickstart) を使ってWebサイトを作ってみましょう。追加の(単位にはならない)課題: カスタムドメインを設定してみましょう。
