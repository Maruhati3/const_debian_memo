# Debianのセットアップ備忘録 (By HitakaK)
PCにDebianをセットアップした際の記録を残しておきます。
今回は、もともとWindowsが入っていたPCのデータをバックアップした上でDebianで完全に上書きする方針で進めました。
デュアルブートを目的とはしていません。

---
## 1-Debianのインストール
- WindowsのPCで、[Debian公式サイト](https://www.debian.org)からDebian本体をダウンロード
- [Rufus](https://rufus.ie/ja/)を使ってDebianインストーラ用USBドライブを作成
	  結構時間かかった記憶
- USBドライブをインストールするPCに挿す
- BIOSからインストーラを選択
	  ここは担当したメンバーに記事お願いしたい

---
## 2-初期設定
今回はグラフィカルインストーラを使用し、対話的に初期設定を行いました。
テキストベースのインストーラと設定項目は同じようです[(参照)](https://www.debian.org/releases/stable/amd64/ch06s01.ja.html#gtk-using)。

- 使用するメモリ、ディスクの設定
- 言語や地域の設定
- キーボードの設定
- ネットワークの設定
	  ここが第一関門でした。詳細を[後ほど](#2.1-ネットワーク設定)説明します。
- ユーザとパスワードの設定
	- rootパスワードの設定
		  rootユーザのパスワードを指定しない場合、rootユーザは作成されません。作成する場合、デフォルトではroot以外のユーザにsudo権限が与えられません。
	- 一般ユーザの設定
		  一般ユーザを1人分作成できます。新しくユーザを追加する場合は設定完了後に`adduser`コマンドを使用します。**システム管理以外の作業は一般ユーザで行いましょう。**
- タイムゾーンの設定
- パーティションの作成
	  ここで最大のディスク容量をDebianに割り当てる設定をしたので既存のシステム (Windows)は上書きされたんだと思います。また、今回はDebianシステム内のファイルはすべて1つのパーティションに保存するように設定しました。
- 基本システムのインストール
	  実行するかどうか選択できましたが、今回は実行しました。
- 追加ソフトウェアのインストール
	  第二関門です。[後ほど](#2.2-aptの設定)説明しますが、今回はスキップしました。設定するには http://ftp.jp.debian.org/debian/ などを選択してください。
- 起動ディスクを用いて起動するようにする設定
	  今回はPCに直接インストールするため、ここはスキップしました。
- USBドライブを抜いて再起動
	  初期設定が完了してターミナル画面が表示されます。

### 2.1-ネットワーク設定
シンプルな設定項目なのですが、今回ここで苦戦したので記録しておきます。
Windowsなどでネットワーク設定をするときはWi-fiルータの名前とPINなどを入力したり、有線で繋ぐことで接続できると思います。
今回は有線接続をしたのですが、DebianではLANケーブルで繋いだあとに**このPCに割り当てるローカルIPアドレス**を入力する必要があります。
例えばNECのAtermルータの場合、デフォルトゲートウェイが`192.168.10.1`、ネットマスクが`255.255.255.0`なので、IPアドレス`192.168.10.0`から`192.168.10.255`が使用できます。
一番大きい値や一番小さい値はなんかに使われてた気がするので、PCにはいい感じの値`192.168.10.100`などを割り当てます（）

#### 2.1.1-ネットワークインターフェースの設定

僕たちのように初期設定のときに間違ったIPアドレスを入力してしまった場合や、後から別のネットワークを追加する場合、手動で設定を行います。
この設定はrootユーザで行います。
まず、使用できるネットワークインターフェースを確認します。
`$ ip a`
次のような出力があることを確認してください。
`2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000`
最初の`enp3s0`のような部分がインターフェース名です[(参照)](https://zenn.dev/eiken/articles/5de36353fc3d61)。`UP`と出ていたら起動中のようです。
次に、設定ファイル`/etc/network/interfaces`を編集します。
```interfaces
allow-hotplug enp3s0
iface enp3s0 inet static
	address 192.168.1.100 # ここと
	netmask 255.255.255.0
	gateway 192.168.1.1 # ここが間違えている！
```
`auto`ディレクティブと正しいIPアドレスの設定を行います。
```interfaces
auto enp3s0
allow-hotplug enp3s0
iface enp3s0 inet static
	address 192.168.10.100 # 修正
	netmask 255.255.255.0
	gateway 192.168.10.1 # 修正
```
このように書いたら保存してください。それからネットワークの再起動を行います。
`$ ifdown enp3s0 && ifup enp3s0`
これで設定を反映できます。
`$ ping 8.8.8.8` (GoogleのDNSサーバ)
を実行し、ネットワークに接続できたことを確認してください。

#### 2.1.2-DNS設定
IPアドレスとドメインを対応づけることで、ドメインをもとに外部のサーバにアクセスできるようにすることを**名前解決**と言います。
また、名前解決してくれる対応づけ屋さんサーバを**DNS**サーバと言います。
DNSサーバはさまざまなものがあるのですが、有名なGoogleのDNSサーバを使用する設定を行います。
この設定はrootユーザで行います。
設定ファイル`/etc/resolv.conf`を編集します。
```resolv.conf
nameserver 8.8.8.8
```
保存すると次回以降のドメインへのアクセスの際にこのDNSサーバが参照されます。
`$ ping google.com`
で動作を確認しましょう。

### 2.2-aptの設定
Debianで使用できるパッケージ管理システムである`Apt`を利用したい！
そのためには最新のパッケージなどを参照するために**ミラーサーバ**を設定する必要があります。
僕たちは初期設定を行った時点で上のネットワーク設定が完了していなかったため、初期設定 (+ネットワーク設定) 完了後に手動でミラーサーバの設定を行いました。
この設定はrootユーザで行います。
設定ファイル`/etc/apt/source.list`を編集します。
```source.list
deb http://ftp.jp.debian.org/debian/ bookworm main contrib non-free-firmware
deb-src http://ftp.jp.debian.org/debian/ bookworm main contrib non-free-firmware

deb http://ftp.jp.debian.org/debian/ bookworm-updates main contrib non-free-firmware
deb-src http://ftp.jp.debian.org/debian/ bookworm-updates main contrib non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware
```
この例は間違えです。Geminiくんに聞きながら教えてくれたものをそのまま使用したので[このあと](#3.1-SSHサーバのインストール)でとんでもなく面倒なことになってしまいました。
上で`bookworm`となっている部分はDebianのバージョンのコードネームを設定する項目で、今回使用したDebian13は`bookworm` (= Debian12)ではなく、`trixie`です。なので、下が正しい例です。
```source.list
deb http://ftp.jp.debian.org/debian/ trixie main contrib non-free-firmware
deb-src http://ftp.jp.debian.org/debian/ trixie main contrib non-free-firmware

deb http://ftp.jp.debian.org/debian/ trixie-updates main contrib non-free-firmware
deb-src http://ftp.jp.debian.org/debian/ trixie-updates main contrib non-free-firmware

deb http://security.debian.org/debian-security trixie-security main contrib non-free-firmware
deb-src http://security.debian.org/debian-security trixie-security main contrib non-free-firmware
```
保存したら設定が反映されます。
`$ apt update`
を実行して、正しくアップデートが実行されたらミラーサーバが設定できていることを確認できます。

---
## 3-使用するソフトウェアのインストール (apt)

### 3.1-SSHサーバのインストール
外部からリモートログインできるようにするため、SSH (Secure Shell)サーバをインストールします。
この設定はrootユーザで行います。
まず、パッケージ管理システム`Apt`のアップデートを行います。
`$ apt update`
`$ apt upgrade`
続いて`openssh-server`をインストールします。
良い子の皆さんは依存パッケージを手動でインストールしたりせず、`Apt`におまかせしましょう。
`$ apt install openssh-server`
インストールが完了したら自動的にSSHサーバが起動します。
`$ systemctl status sshd`
で動作を確認できます。
ここまでできたらリモートログインできるようになっています。
同じLAN内にあるPCで試してみましょう。
`$ ssh <<user名>>@192.168.1.100`

*備考*
1. `openssh-client`が必要だと言われた場合、`Apt`でインストールしてください。
	`$ apt install openssh-client`
	普通は`openssh-server`のインストール時に一緒に入れてくれるんじゃないかと思います。

2. UbuntuからSSHした人と、WindowsからSSHした人はログイン時からBashに入れていたのに、僕だけなぜかshが起動するいじめを受けていたのでログインシェルの設定を行いました。
	`$ chsh`
	ユーザのパスワードとログインシェル (`/bin/bash`)を入力します。
	また、`.bashrc`と`.profile`を作成します。
	`$ cat /etc/skel/.bashrc > ~/.bashrc`
	`$ cat /etc/skel/.profile > ~/.profile`
	これで次回以降のログインでは快適なBashで遊べます👍
	
### 3.2-その他のツール
僕 (HitakaK) が普段よく使うツールを紹介します。
1. `build-essential` (https://packages.debian.org/ja/sid/build-essential)
   C言語やC++での開発によく使うパッケージをまとめてインストールしてくれるメタパッケージです。
   `$ apt install build-essential`
2. `man`
   コマンドのマニュアル (manual) を見ることができるコマンドです。
   `$ apt install man-db`
   コマンドのオプションなど、細かい内容を毎回調べる手間が省けるのでオススメ
   例) `$ man gcc`
3. Git
   バージョン管理システムです、さすがにいります。
   `$ apt install git`
4. GNU Awk
   行、列形式のテキストファイル (CSVなど)の処理にめちゃくちゃ便利です[(参照)](https://www.tohoho-web.com/ex/awk.html)。
   `$ apt install gawk`

---
#### 参考記事
- Debianインストールガイド (https://www.debian.org/releases/stable/amd64/index.ja.html)
- Debianリファレンス (https://qref.sourceforge.net/Debian/reference/index.ja.html)