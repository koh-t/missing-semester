---
layout: lecture
title: "セキュリティとプライバシー"
presenter: Jon
date: 2019-01-31
order: 2
video:
  aspect: 56.25
  id: OBx_c-i-M8s
special: true
---

世界は恐ろしい場所で、誰もがあなたを狙っています。

まあ、そこまでではないかもしれませんが、だからといって自分の秘密をすべて見せびらかしたいわけではないでしょう。セキュリティ(そしてプライバシー)とは、おおむね攻撃者にとってのハードルを上げることです。自分の脅威モデルが何であるかを見極め、それに合わせてセキュリティの仕組みを設計しましょう!脅威モデルがNSAやモサドなら、_おそらく_ ひどい目に遭うことになります。

技術的な面での自分をより安全にする方法は _たくさん_ あります。ここでは高レベルな事柄を数多く取り上げますが、これは継続的なプロセスであり、自ら学ぶことが、できる最善のことのひとつです。というわけで:

## 適切な人をフォローする

セキュリティのノウハウを高める最良の方法のひとつは、セキュリティについて積極的に発信している人をフォローすることです。いくつかおすすめを挙げます。

 - [@TroyHunt](https://twitter.com/TroyHunt)
 - [@SwiftOnSecurity](https://twitter.com/SwiftOnSecurity)
 - [@taviso](https://twitter.com/taviso)
 - [@thegrugq](https://twitter.com/thegrugq)
 - [@tqbf](https://twitter.com/tqbf)
 - [@mattblaze](https://twitter.com/mattblaze)
 - [@moxie](https://twitter.com/moxie)

さらに多くのおすすめについては、[このリスト](https://heimdalsecurity.com/blog/best-twitter-cybersec-accounts/)も参照してください。

## 一般的なセキュリティのアドバイス

Tech Solidarityには、[ジャーナリスト向けの「すべきこと・すべきでないこと」](https://web.archive.org/web/20221123204419/https://techsolidarity.org/resources/basic_security.htm)をまとめたなかなか優れたリストがあり、まっとうなアドバイスが数多く載っていて、内容もそれなりに新しいものです。[@thegrugq](https://medium.com/@thegrugq)も、[旅行時のセキュリティに関するアドバイス](https://medium.com/@thegrugq/stop-fabricating-travel-security-advice-35259bf0e869)についての良いブログ記事を書いており、読む価値があります。ここでは、これらの情報源のアドバイスの多くを繰り返し、さらにいくつか付け加えます。また、[USBデータブロッカー](https://www.amazon.com/dp/B00QRRZ2QM/)を手に入れましょう。[USBは恐ろしい](https://www.bleepingcomputer.com/news/security/heres-a-list-of-29-different-types-of-usb-attacks/)からです。

## 認証

まだやっていないなら、真っ先にすべきことはパスワードマネージャをダウンロードすることです。良いものをいくつか挙げます。

 - [1password](https://1password.com/)
 - [KeePass](https://keepass.info/)
 - [BitWarden](https://bitwarden.com/)
 - [`pass`](https://git.zx2c4.com/password-store/about/)

特に用心深い人は、パスワードをサーバーに平文で保存するのではなく、自分のコンピュータ上でローカルに暗号化するものを使いましょう。それを使って、いま自分が大事にしているすべてのWebサイトのパスワードを生成してください。次に、二要素認証を有効にします。理想的には[FIDO/U2F](https://fidoalliance.org/)ドングル(たとえば[YubiKey](https://www.yubico.com/quiz/)。[学生は20%引き](https://www.yubico.com/why-yubico/for-education/)です)を使いましょう。TOTP(Google AuthenticatorやDuoなど)も間に合わせとしては使えますが、[フィッシングは防げません](https://twitter.com/taviso/status/1082015009348104192)。SMSは、通信途中でパスワードを拾う行きずりの他人だけが脅威モデルである場合を除いて、ほとんど役に立ちません。

また、ペーパーキーについても一言。サービスはしばしば「バックアップキー」を発行してくれます。これは、本来の第二要素を失くした場合に第二要素として使えるものです(ところで、予備のドングルは必ずどこか安全な場所に保管しておきましょう!)。これらをパスワードマネージャに入れておくことも _できます_ が、そうすると、誰かにパスワードマネージャへアクセスされた時点で完全におしまいです(とはいえ、その脅威モデルで構わないという人もいるでしょう)。本当に用心深い人は、これらのペーパーキーを印刷し、決してデジタルでは保存せず、現実世界の金庫に入れておきましょう。

## プライベートな通信

[Signal](https://www.signal.org/)を使いましょう([セットアップ手順](https://medium.com/@mshelton/signal-for-beginners-c6b44f76a1f0)。[Wire](https://wire.com/en/)[でも構いません](https://www.securemessagingapps.com/)。WhatsAppもまずまずです。[Telegramは使わないでください](https://twitter.com/bascule/status/897187286554628096))。デスクトップ版のメッセンジャーはかなり問題を抱えています(通常Electronに依存していて、信頼しなければならないスタックが巨大になることが一因です)。

電子メールは、PGPで署名されていたとしても、とりわけ問題があります。一般に前方秘匿性がなく、鍵配布の問題もかなり深刻です。[keybase.io](https://keybase.io/)はその助けになりますし、ほかにもさまざまな理由で便利です。また、PGPの鍵は一般にデスクトップコンピュータ上で扱われますが、これはもっとも安全性の低いコンピューティング環境のひとつです。これに関連して、Chromebookを手に入れるか、あるいはキーボード付きのタブレットで作業することを検討してみてください。

## ファイルのセキュリティ

ファイルのセキュリティは難しく、多くのレベルで機能します。あなたは何から守ろうとしているのでしょうか?

[![$5 wrench](https://imgs.xkcd.com/comics/security.png)](https://xkcd.com/538/)

 - オフライン攻撃(電源が切れている間に誰かがノートパソコンを盗む): フルディスク暗号化を有効にしましょう。(Linuxでは[cryptsetup + LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system)、Windowsでは[BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/)、macOSでは[FileVault](https://support.apple.com/en-us/HT204837)。ただし、攻撃者があなたの身柄 _も_ 押さえていて、本気であなたの秘密を欲しがっている場合には役に立たないことに注意してください。
 - オンライン攻撃(誰かがあなたのノートパソコンを手にしていて、電源が入っている): ファイル暗号化を使いましょう。これには主に2つの仕組みがあります
    - 暗号化ファイルシステム: スタック型のファイルシステム暗号化ソフトウェアは、ブロックデバイスを暗号化するのではなく、ファイルを個別に暗号化します。復号鍵を与えることでこれらのファイルシステムを「マウント」でき、その後は中のファイルを自由に閲覧できます。アンマウントすると、それらのファイルはすべて利用できなくなります。最近のソリューションとしては[gocryptfs](https://github.com/rfjakob/gocryptfs)や[eCryptFS](https://www.ecryptfs.org/)があります。より詳しい比較は[こちら](https://nuetzlich.net/gocryptfs/comparison/)と[こちら](https://wiki.archlinux.org/index.php/disk_encryption#Comparison_table)にあります
    - 暗号化ファイル: 共通鍵暗号(`gpg -c` を参照)と秘密の鍵で個々のファイルを暗号化します。あるいは `pass` のように、その鍵も自分の公開鍵で暗号化しておき、あとで自分の秘密鍵を使って自分だけが読み戻せるようにします。暗号化の具体的な設定は非常に重要です!
 - [もっともらしい否認](https://en.wikipedia.org/wiki/Plausible_deniability)(お巡りさん、何か問題でも?): たいてい性能が低く、データも失いやすくなります。それが[否認可能な暗号化](https://en.wikipedia.org/wiki/Deniable_encryption)を実現していると実際に証明するのは困難です! [こちらの議論](https://security.stackexchange.com/questions/135846/is-plausible-deniability-actually-feasible-for-encrypted-volumes-disks)を読んだうえで、[VeraCrypt](https://www.veracrypt.fr/en/Home.html)(古き良きTrueCryptの、メンテナンスされているフォーク)を試してみたいかどうか考えてみてください。
 - 暗号化バックアップ: [Tarsnap](https://www.tarsnap.com/)または[Borgbase](https://www.borgbase.com/)を使いましょう
    - 攻撃者があなたのノートパソコンを手に入れた場合に、バックアップを削除できてしまわないか考えてみてください!

## インターネットのセキュリティとプライバシー

インターネットは _非常に_ 恐ろしい場所です。オープンなWiFiネットワーク[は](https://www.troyhunt.com/the-beginners-guide-to-breaking-website/)[恐ろしい](https://www.troyhunt.com/talking-with-scott-hanselman-on/)ものです。使ったあとは必ず削除しておきましょう。さもないと、あなたのスマートフォンはあとで同じ名前の何かに対して、喜んで自分の存在を知らせて再接続してしまいます!

信頼できないネットワークにいる場合、VPNには価値が _あるかもしれません_ が、VPNプロバイダを _大いに_ 信頼することになる点を忘れないでください。本当にISPよりもそのプロバイダを信頼できますか?どうしてもVPNが欲しいなら、確実に信頼できるプロバイダを使いましょう。そしておそらくお金を払って使うべきです。あるいは、自分で[WireGuard](https://www.wireguard.com/)をセットアップしましょう -- これは[素晴らしい](https://web.archive.org/web/20210526211307/https://latacora.micro.blog/there-will-be/)ものです!

また、インターネットに接続する多くのアプリケーション向けの安全な設定が[cipherlist.eu](https://cipherlist.eu/)にあります。特にプライバシーを重視する人には、[privacytools.io](https://privacytools.io)も良い情報源です。

[Tor](https://www.torproject.org/)はどうなのかと思う人もいるでしょう。Torは強力なグローバル攻撃者に対して特に耐性があるわけでは _なく_、トラフィック解析攻撃にも弱いということを覚えておいてください。小規模にトラフィックを隠すのには役立つかもしれませんが、プライバシーの面ではそれほど多くを得られるわけではありません。そもそも、より安全なサービス(Signal、TLS + 証明書ピンニングなど)を使うほうがよいでしょう。

## Webのセキュリティ

それで、Webにもアクセスしたいというのですか?やれやれ、ずいぶん危ない橋を渡ろうとしていますね。

[HTTPS Everywhere](https://www.eff.org/https-everywhere)をインストールしましょう。SSL/TLSは[極めて重要](https://www.troyhunt.com/ssl-is-not-about-encryption/)で、暗号化のためだけのものでは _なく_、そもそも正しいサービスと通信していることを検証できるようにするためのものでもあります!自分でWebサーバーを運用しているなら、[テストしてください](https://www.ssllabs.com/ssltest/index.html)。TLSの設定は[厄介になることがあります](https://wiki.mozilla.org/Security/Server_Side_TLS)。HTTPS Everywhereは、代替がある場合には決してHTTPのサイトに移動させないよう最善を尽くしてくれます。それで安全が保証されるわけではありませんが、助けにはなります。本当に用心深い人は、絶対に必要というわけではないSSL/TLSのCAをすべてブラックリストに入れましょう。

[uBlock Origin](https://github.com/gorhill/uBlock)をインストールしましょう。これは[広範囲を対象とするブロッカー](https://github.com/gorhill/uBlock/wiki/Blocking-mode)で、広告を止めるだけでなく、ページが行おうとするあらゆる種類のサードパーティとの通信も止めます。インラインスクリプトなども同様です。うまく動くように設定に多少の時間をかけてもよいなら、[ミディアムモード](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-medium-mode)や、さらには[ハードモード](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-hard-mode)にしましょう。これらのモードでは、設定を十分にいじるまで一部のサイトが _確実に_ 動かなくなりますが、オンラインでのセキュリティも大幅に向上します。

Firefoxを使っているなら、[Multi-Account Containers](https://support.mozilla.org/en-US/kb/containers)を有効にしましょう。ソーシャルネットワーク、銀行、ショッピングなどのために別々のコンテナを作ります。Firefoxは各コンテナのCookieやその他の状態を完全に分離して保持するので、あるコンテナで訪れたサイトが他のコンテナの機密データを盗み見ることはできません。Google Chromeでは、[Chromeプロファイル](https://support.google.com/chrome/answer/2364824)を使って同様の結果を得られます。

## 演習

1. PGPを使ってファイルを暗号化してください
1. veracryptを使って、シンプルな暗号化ボリュームを作成してください
1. もっとも機密性の高いデータを扱うアカウント(GMail、Dropbox、Githubなど)で2FAを有効にしてください
