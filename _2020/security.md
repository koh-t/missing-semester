---
layout: lecture
title: "セキュリティと暗号"
description: >
  ハッシュや鍵導出関数といった暗号プリミティブについて学び、GitやSSHのようなツールがそれらをどう使っているかを理解します。
thumbnail: /static/assets/thumbnails/2020/lec9.png
date: 2020-01-28
ready: true
video:
  aspect: 56.25
  id: tjwobAmnKTo
special: true
---

昨年の[セキュリティとプライバシーの講義](/2019/security/)では、コンピュータの _ユーザー_ としてどうすればより安全でいられるかに焦点を当てました。今年は、Gitでのハッシュ関数の利用や、SSHでの鍵導出関数と対称/非対称暗号方式の利用など、この授業でこれまでに扱ったツールを理解するうえで関係のある、セキュリティと暗号の概念に焦点を当てます。

この講義は、コンピュータシステムのセキュリティ([6.858](https://css.csail.mit.edu/6.858/))や暗号([6.857](https://courses.csail.mit.edu/6.857/)と6.875)についての、より厳密で網羅的な授業の代わりになるものではありません。セキュリティに関する正式な訓練を受けずに、セキュリティの仕事をしてはいけません。専門家でない限り、[暗号を自作](https://www.schneier.com/blog/archives/2015/05/amateurs_produc.html)してはいけません。同じ原則はシステムのセキュリティにも当てはまります。

この講義では、暗号の基本的な概念を、非常にくだけた形で(ただし実用的だと私たちは考えています)扱います。この講義だけでは、安全なシステムや暗号プロトコルを _設計_ する方法を身につけるには足りませんが、すでに使っているプログラムやプロトコルについて大まかに理解するには十分であることを願っています。

# エントロピー

[エントロピー](https://en.wikipedia.org/wiki/Entropy_(information_theory))は、ランダムさの尺度です。これは、たとえばパスワードの強度を見積もるときに役立ちます。

![XKCD 936: パスワードの強度](https://imgs.xkcd.com/comics/password_strength.png)

上の[XKCDのコミック](https://xkcd.com/936/)が示すように、「correcthorsebatterystaple」のようなパスワードは、「Tr0ub4dor&3」のようなパスワードよりも安全です。しかし、こうしたことをどうやって定量化すればよいのでしょうか?

エントロピーは _ビット_ で測られ、起こりうる結果の集合から一様ランダムに選ぶ場合、エントロピーは `log_2(# of possibilities)` に等しくなります。公正なコイン投げは1ビットのエントロピーを与えます。(6面の)サイコロを1回振ると、約2.58ビットのエントロピーがあります。

攻撃者はパスワードの _モデル_ は知っているが、特定のパスワードを選ぶのに使われたランダムさ(たとえば[サイコロを振った結果](https://en.wikipedia.org/wiki/Diceware))は知らない、と考えるべきです。

何ビットのエントロピーがあれば十分なのでしょうか?それは脅威モデルによります。XKCDのコミックが指摘するように、オンラインでの推測に対しては、約40ビットのエントロピーがあればかなり良好です。オフラインでの推測に耐えるには、より強いパスワード(たとえば80ビット、あるいはそれ以上)が必要になるでしょう。

# ハッシュ関数

[暗号学的ハッシュ関数](https://en.wikipedia.org/wiki/Cryptographic_hash_function)は、任意のサイズのデータを固定のサイズに写像するもので、いくつかの特別な性質を持ちます。ハッシュ関数の大まかな仕様は次のとおりです。

```
hash(value: array<byte>) -> vector<byte, N>  (for some fixed N)
```

ハッシュ関数の一例が、Gitで使われている[SHA1](https://en.wikipedia.org/wiki/SHA-1)です。これは任意のサイズの入力を160ビットの出力(16進数40文字で表せます)に写像します。`sha1sum` コマンドを使って、ある入力に対するSHA1ハッシュを試してみることができます。

```console
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'Hello' | sha1sum
f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0
```

大まかに言えば、ハッシュ関数は、逆算が難しく、ランダムに見える(しかし決定的な)関数だと考えることができます(そしてこれが[ハッシュ関数の理想的なモデル](https://en.wikipedia.org/wiki/Random_oracle)です)。ハッシュ関数には次のような性質があります。

- 決定的: 同じ入力は常に同じ出力を生成します。
- 非可逆: 望む出力 `h` に対して、`hash(m) = h` となるような入力 `m` を見つけるのは困難です。
- 標的衝突耐性: 入力 `m_1` が与えられたとき、`hash(m_1) = hash(m_2)` となるような別の入力 `m_2` を見つけるのは困難です。
- 衝突耐性: `hash(m_1) = hash(m_2)` となるような2つの入力 `m_1` と `m_2` を見つけるのは困難です(これは標的衝突耐性よりも真に強い性質であることに注意してください)。

注: 用途によっては問題なく使えるかもしれませんが、SHA-1は[もはや](https://web.archive.org/web/20260207211148/https://shattered.io/)強力な暗号学的ハッシュ関数とは見なされていません。この[暗号学的ハッシュ関数の寿命](https://valerieaurora.org/hash.html)の表は興味深いかもしれません。ただし、特定のハッシュ関数を推奨することは、この講義の範囲を超えていることに注意してください。これが重要になるような仕事をしているのであれば、セキュリティや暗号の正式な訓練が必要です。

## 応用例

- Git。コンテンツアドレス指定ストレージのために使われます。[ハッシュ関数](https://en.wikipedia.org/wiki/Hash_function)という考え方は、より一般的な概念です(暗号学的ではないハッシュ関数もあります)。Gitはなぜ暗号学的ハッシュ関数を使うのでしょうか?
- ファイルの内容の短い要約。ソフトウェアは、Linux ISOのように、(信頼性が低いかもしれない)ミラーからダウンロードできることがよくありますが、それらのミラーを信頼しなくて済むならそのほうがよいでしょう。公式サイトはたいてい、(サードパーティのミラーを指す)ダウンロードリンクと並べてハッシュを掲載しているので、ファイルをダウンロードしたあとでハッシュを確認できます。
- [コミットメントスキーム](https://en.wikipedia.org/wiki/Commitment_scheme)。ある特定の値にコミットしておきたいが、値そのものはあとで明かしたいとします。たとえば、双方が見られる信頼できる共有のコインなしに、「頭の中で」公正なコイントスをしたいとしましょう。私は値 `r = random()` を選び、`h = sha256(r)` を共有します。それからあなたが表か裏かを宣言します(`r` が偶数なら表、奇数なら裏だと取り決めておきます)。あなたが宣言したあとで、私は自分の値 `r` を明かし、あなたは `sha256(r)` が先ほど共有されたハッシュと一致することを確かめることで、私がごまかしていないことを確認できます。

# 鍵導出関数

暗号学的ハッシュに関連する概念である[鍵導出関数](https://en.wikipedia.org/wiki/Key_derivation_function)(KDF)は、他の暗号アルゴリズムで鍵として使うための固定長の出力を生成することをはじめ、さまざまな用途に使われます。通常、KDFはオフラインの総当たり攻撃を遅くするために、意図的に低速になっています。

## 応用例

- 他の暗号アルゴリズム(たとえば対称暗号。後述)で使うための鍵を、パスフレーズから生成する。
- ログインの認証情報を保存する。平文のパスワードを保存するのは良くありません。正しいやり方は、ユーザーごとにランダムな[ソルト](https://en.wikipedia.org/wiki/Salt_(cryptography)) `salt = random()` を生成して保存し、`KDF(password + salt)` を保存しておき、入力されたパスワードと保存されたソルトからKDFを再計算することでログインの試行を検証することです。

# 対称暗号

メッセージの内容を隠すことは、暗号について考えるとき、おそらく最初に思い浮かぶ概念でしょう。対称暗号は、次のような一連の機能によってこれを実現します。

```
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```

encrypt関数には、出力(暗号文)が与えられても、鍵なしでは入力(平文)を特定するのが困難である、という性質があります。decrypt関数には、`decrypt(encrypt(m, k), k) = m` という、当然の正当性の性質があります。

現在広く使われている対称暗号方式の一例が、[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)です。

## 応用例

- 信頼できないクラウドサービスに保存するためにファイルを暗号化する。これはKDFと組み合わせることができ、パスフレーズでファイルを暗号化できます。`key = KDF(passphrase)` を生成し、`encrypt(file, key)` を保存します。

# 非対称暗号

「非対称」という用語は、異なる役割を持つ2つの鍵があることを指しています。秘密鍵は、その名のとおり秘密にしておくためのものです。一方、公開鍵は公に共有することができ、そうしてもセキュリティには影響しません(対称暗号方式で鍵を共有する場合とは異なります)。非対称暗号方式は、暗号化/復号と、署名/検証のために、次のような一連の機能を提供します。

```
keygen() -> (public key, private key)  (this function is randomized)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)

sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```

encrypt/decrypt関数は、対称暗号方式における同名の関数と似た性質を持ちます。メッセージは _公開_ 鍵を使って暗号化できます。出力(暗号文)が与えられても、_秘密_ 鍵なしでは入力(平文)を特定するのは困難です。decrypt関数には、`decrypt(encrypt(m, public key), private key) = m` という、当然の正当性の性質があります。

対称暗号と非対称暗号は、物理的な錠にたとえることができます。対称暗号方式はドアの錠のようなもので、鍵を持っている人なら誰でも施錠と解錠ができます。非対称暗号は、鍵付きの南京錠のようなものです。開いた状態の南京錠(公開鍵)を誰かに渡すと、その人はメッセージを箱に入れて南京錠をかけることができます。そのあとは、鍵(秘密鍵)を手元に残しておいたあなただけが、その錠を開けられます。

sign/verify関数は、物理的な署名に期待するのと同じ性質、つまり署名の偽造が困難であるという性質を持ちます。どんなメッセージであっても、_秘密_ 鍵なしでは、`verify(message, signature, public key)` がtrueを返すような署名を作るのは困難です。そしてもちろん、verify関数には、`verify(message, sign(message, private key), public key) = true` という、当然の正当性の性質があります。

## 応用例

- [PGPによるメールの暗号化](https://en.wikipedia.org/wiki/Pretty_Good_Privacy)。人々は自分の公開鍵をオンライン(たとえばPGPの鍵サーバーや[Keybase](https://keybase.io/))に掲載しておけます。誰でもその人に暗号化されたメールを送れます。
- プライベートメッセージング。[Signal](https://signal.org/)や[Keybase](https://keybase.io/)のようなアプリは、非対称鍵を使ってプライベートな通信チャネルを確立します。
- ソフトウェアへの署名。Gitでは、コミットやタグにGPGで署名できます。公開鍵が掲載されていれば、誰でもダウンロードしたソフトウェアの真正性を検証できます。

## 鍵配送

非対称鍵暗号は素晴らしいものですが、公開鍵を配送すること、すなわち公開鍵を現実世界のアイデンティティに対応づけることという、大きな課題があります。この問題には多くの解決策があります。Signalの解決策は単純で、初回利用時に信頼し(trust on first use)、帯域外での公開鍵の交換をサポートする(友人の「安全番号」を対面で確認する)というものです。PGPには別の解決策があり、それが[信頼の輪(web of trust)](https://en.wikipedia.org/wiki/Web_of_trust)です。Keybaseにはさらに別の解決策があり、[ソーシャルプルーフ](https://keybase.io/blog/chat-apps-softer-than-tofu)を(他の巧妙なアイデアとともに)使います。どのモデルにもそれぞれの利点があります。私たち(講師陣)はKeybaseのモデルが気に入っています。

# ケーススタディ

## パスワードマネージャ

これは、誰もが使うよう心がけるべき必須のツールです(たとえば[KeePassXC](https://keepassxc.org/)、[pass](https://git.zx2c4.com/password-store/about/)、[1Password](https://1password.com)など)。パスワードマネージャを使えば、すべてのログインに、ランダムに生成された高エントロピーの固有のパスワードを手軽に使えるようになります。また、すべてのパスワードを1か所に保存し、KDFを使ってパスフレーズから生成した鍵を用いて、対称暗号で暗号化してくれます。

パスワードマネージャを使うことで、パスワードの使い回しを避けられ(そのため、Webサイトが侵害されたときの影響が小さくなります)、高エントロピーのパスワードを使え(そのため、自分が侵害される可能性が低くなります)、しかも覚えておく必要があるのは高エントロピーのパスワードひとつだけになります。

## 二要素認証

[二要素認証](https://en.wikipedia.org/wiki/Multi-factor_authentication)(2FA)は、パスフレーズ(「あなたが知っているもの」)に加えて、2FA認証器([YubiKey](https://www.yubico.com/)のような「あなたが持っているもの」)を使うことを要求します。これは、パスワードの盗難や[フィッシング](https://en.wikipedia.org/wiki/Phishing)攻撃から身を守るためのものです。

## フルディスク暗号化

ノートパソコンのディスク全体を暗号化しておくことは、ノートパソコンが盗まれた場合にデータを守るための簡単な方法です。Linuxでは[cryptsetup + LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system)、Windowsでは[BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/)、macOSでは[FileVault](https://support.apple.com/en-us/HT204837)が使えます。これは、パスフレーズで保護された鍵を使い、対称暗号でディスク全体を暗号化します。

## プライベートメッセージング

[Signal](https://signal.org/)か[Keybase](https://keybase.io/)を使いましょう。エンドツーエンドのセキュリティは、非対称鍵暗号を出発点として確立されます。ここで決定的に重要なステップは、連絡先の公開鍵を入手することです。高いセキュリティが欲しいなら、(SignalまたはKeybaseで)帯域外で公開鍵を認証するか、(Keybaseで)ソーシャルプルーフを信頼する必要があります。

## SSH

SSHとSSH鍵の使い方は、[以前の講義](/2020/command-line/#remote-machines)で扱いました。ここでは、その暗号に関わる側面を見てみましょう。

`ssh-keygen` を実行すると、非対称の鍵ペア `public_key, private_key` が生成されます。これは、オペレーティングシステムが提供するエントロピー(ハードウェアのイベントなどから収集されたもの)を使って、ランダムに生成されます。公開鍵はそのまま保存されますが(公開されるものなので、秘密にしておくことは重要ではありません)、秘密鍵は保存時にディスク上で暗号化されているべきです。`ssh-keygen` プログラムはユーザーにパスフレーズの入力を求め、これが鍵導出関数に通されて鍵が生成され、その鍵を使って秘密鍵が対称暗号で暗号化されます。

利用時には、サーバーがクライアントの公開鍵(`.ssh/authorized_keys` ファイルに保存されています)を知っていれば、接続してくるクライアントは非対称署名を使って自分の身元を証明できます。これは[チャレンジ・レスポンス](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication)によって行われます。大まかに言えば、サーバーが乱数を選んでクライアントに送ります。クライアントはこのメッセージに署名して署名をサーバーに送り返し、サーバーは記録にある公開鍵と照らし合わせてその署名を確認します。これにより、サーバーの `.ssh/authorized_keys` ファイルにある公開鍵に対応する秘密鍵をクライアントが持っていることが事実上証明されるので、サーバーはクライアントのログインを許可できます。

{% comment %}
時間があれば扱う追加トピック

セキュリティの概念、コツ
- 生体認証
- HTTPS
{% endcomment %}

# 参考資料

- [昨年のノート](/2019/security/): この講義が、コンピュータのユーザーとしてのセキュリティとプライバシーにより重点を置いていた頃のものです
- [Cryptographic Right Answers](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html): よくあるさまざまなXについて、「Xにはどの暗号を使うべきか?」に答えています。

# 演習

1. **エントロピー。**
    1. パスワードが、小文字の辞書単語4つを連結したものとして選ばれるとします。各単語は、100,000語からなる辞書から一様ランダムに選ばれます。そのようなパスワードの例が `correcthorsebatterystaple` です。これは何ビットのエントロピーを持つでしょうか?
    1. 別の方式として、パスワードがランダムな英数字(小文字と大文字の両方を含む)8文字の列として選ばれる場合を考えます。例は `rg8Ql34g` です。これは何ビットのエントロピーを持つでしょうか?
    1. どちらがより強いパスワードでしょうか?
    1. 攻撃者が1秒間に10,000個のパスワードを推測できるとします。それぞれのパスワードを破るのに、平均でどれくらいの時間がかかるでしょうか?
1. **暗号学的ハッシュ関数。** [ミラー](https://www.debian.org/CD/http-ftp/)から(たとえば[このアルゼンチンのミラーから](http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/))Debianのイメージをダウンロードしてください。そのハッシュを(たとえば `sha256sum` コマンドを使って)、Debianの公式サイトから取得したハッシュ(リンク先のファイルをアルゼンチンのミラーからダウンロードした場合は、たとえば `debian.org` でホストされている[このファイル](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS))と照合してください。
1. **対称暗号。** [OpenSSL](https://www.openssl.org/)を使って、ファイルをAESで暗号化してください: `openssl aes-256-cbc -salt -in {input filename} -out {output filename}`。`cat` または `hexdump` を使って中身を見てみましょう。`openssl aes-256-cbc -d -in {input filename} -out {output filename}` で復号し、`cmp` を使って内容が元のファイルと一致することを確認してください。
1. **非対称暗号。**
    1. 自分がアクセスできるコンピュータで[SSH鍵](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)をセットアップしてください(Athenaではなく。KerberosはSSH鍵と奇妙な相互作用をするためです)。秘密鍵がパスフレーズで暗号化されていて、保存時に保護されていることを確認してください。
    1. [GPGをセットアップしてください](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)
    1. Anishに暗号化したメールを送ってください([公開鍵](https://keybase.io/anish))。
    1. `git commit -S` でGitのコミットに署名するか、`git tag -s` で署名付きのGitタグを作成してください。`git show --show-signature` でコミットの署名を、または `git tag -v` でタグの署名を検証してください。
