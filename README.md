# 概要
* RaspberryPi 3 Model B のセットアップ手順
* openframeworks(0.9.8)をRaspberryPi 3 Model B で動かす手順。
* [このoFフォーラム](https://forum.openframeworks.cc/t/compiling-of-in-raspbian-stretch/27562/15)を参考にしている。これとは別で検証をした人のログが[これ](https://forum.openframeworks.cc/t/compiling-of-in-raspbian-stretch/27562/32)。
* [CreativeApplicationsNetに提供されたサンプル群](https://github.com/openFrameworks-arm/RaspberryPiGuideCAN)

# イメージをSDカードに焼く
1. SDカードを準備する。[Transcend 32GB MicroSDHC Class10 UHS-1 Memory Card with Adapter 60 MB/s (TS32GUSDU1)](https://www.amazon.com/Transcend-MicroSDHC-Class10-Adapter-TS32GUSDU1/dp/B00APCMMDG) などのClass10を使用する。

2. 既にSDカードを持っていて、初期化したい場合は、[SD Card Formatter](https://www.sdcard.org/jp/downloads/formatter_4/)などを使用し、初期化する。

3. [イメージ](http://downloads.raspberrypi.org/raspbian/images/)をダウンロードする。

4. [ETCHER](https://etcher.io)を使用し、イメージをSDカードに焼く。

# 起動

1. RaspberryPiを準備。まだ電源は入れない。

2. RaspberryPiにSDカードを挿入。

3. 外付けディスプレイ(HDMI)を準備し、RaspberryPiとHDMIケーブルでつなぐ。

4. RaspberryPiに電源を共有する。

# SSHを有効にする

RaspberryPiが起動したら、まずSSHを有効にする。SSHを有効にし、他PCからリモートで操作する方が楽。以下のコマンドを実行。

```
$ sudo raspi-config
```

* `5 Interfacing Options -> P2 SSH`

で、SSHを有効にする。RaspberryPiを再起動させる。※その他オプションは後で触れる。

```
$ sudo reboot
```

ここから先の設定は、画面無しでOK。他PCとRaspberryPiを同じネットワークに入れるようにし、ssh接続する。

# パッケージを取得してインストール/アップデートする

インターネットのある環境を準備。RaspberryPiが起動したら、以下を順番に行う。

```
$ sudo apt-get clean
```

```
$ sudo apt-get update
```

```
$ sudo apt-get upgrade
```

この`upgrade`が結構時間かかる。だいたい20~30分程度。

# oF - インストール準備

再度、以下のコマンドを実行。

```
$ sudo raspi-config
```

各設定を行う。

* `3 Boot Options -> B1 Desktop / CLI -> B2 Console Autologin`
* `7 Advanced Options -> A1 Expand Filesystem`
* `7 Advanced Options -> A3 Memory Split -> 64`
* `7 Advanced Options -> A7 GL Driver -> G3 Legacy`

上記を設定し、再起動を行う。

```
$ sudo reboot
```

# oF - インストール

以下のコマンドを順に実行する。

```
$ cd ~
```

```
$ git clone --depth=1 https://github.com/openFrameworks/openFrameworks.git
```

```
$ cd openFrameworks
```

```
$ sudo /bin/bash scripts/linux/debian/install_dependencies.sh
```

```
$ sudo /bin/bash scripts/linux/debian/install_codecs.sh
```

```
$ /bin/bash scripts/linux/download_libs.sh
```

# oF - .mkファイル編集

```
$ sudo nano libs/openFrameworksCompiled/project/linuxarmv6l/config.linuxarmv6l.default.mk
```
| 変更前 | 変更後 |
|:-|:-|
| PLATFORM_LIBRARIES += GLESv2 | PLATFORM_LIBRARIES += brcmGLESv2　|
| PLATFORM_LIBRARIES += GLESv1_CM | 削除　|
| PLATFORM_LIBRARIES += EGL | PLATFORM_LIBRARIES += brcmEGL |

# oF - コンパイル

以下のコマンドを順に実行する。

```
$ cd ~/openFrameworks/
```

```
$ cp scripts/templates/linuxarmv6l/Makefile examples/templates/emptyExample/
```

```
$ cd examples/templates/emptyExample/
```

```
$ make -j2 -s
```

ここ結構時間かかる。

```
$ make run
```

コンソール上に、ofLogNoticeなどのログが表示されれば問題無し。

# oF - 画面で確認してみる。

ログイン設定を変更する。

```
$ sudo raspi-config
```

* `3 Boot Options -> B1 Desktop / CLI -> B4 Desktop Autologin`

デスクトップ表示されるように変更。再起動を行う。

```
$ sudo reboot
```

外付けディスプレイを接続し、再度、任意のファイルを`make run`する。oFアプリがデスクトップに表示されることを確認してみる。

# oF - Macで開発したプロジェクトをRaspberryPiへコピー

RaspberryPiで開発を行うのはやめた方がいい。以下は、Macで開発したプロジェクトをRaspberryPiに移動させ、コンパイルする流れ。

プロジェクトフォルダごと移動。

```
$ scp -r [自分のoFプロジェクトフォルダ] pi@192.168.xx.xxx:~/openFrameworks/apps/devApps/
```

RaspberryPi上で、以下を実行。Makefileをコピー。

```
$ cd ~/openFrameworks
$ cp scripts/templates/linuxarmv6l/Makefile apps/devApps/自分のoFプロジェクトフォルダ/
$ cd apps/devApps/自分のoFプロジェクトフォルダ/
```

コンパイルする。

```
$ make -j2 -s
$ make run
```

※プロジェクトをコピーする方法ではなく、[クロスコンパイル](https://forum.openframeworks.cc/t/cross-compiler-for-of-0-9-0-jessie-arm6-rpi1/21336)する方法もあるみたい。

# oF - Macで開発したアドオンをRaspberryPiへコピー

自作アドオンをRaspberryPiへコピーする流れ。※基本的にプロジェクトを移動させる方法と同じ。

```
$ scp -r [自分のoFアドオン] pi@192.168.xx.xxx:~/openFrameworks/addons/
```

アドオン内のexampleをコンパイルする場合は、前項の「oF - Macで開発したプロジェクトをRaspberryPiへコピー」を参照すること。

# oF - アプリ自動起動

* systemdを用いた自動起動の手順。
* シェルスクリプトでoFアプリを起動させるようにし、起動時にシェルスクリプトを実行させる方法をとっている。

#### 自動起動用のシェルスクリプト(.sh)作成。

```
$ sudo nano /opt/hello.sh
```

以下を記述。

```
#!/bin/bash

cd
cd /home/pi/openFrameworks/apps/devApps/YourProjectName/
make run

while true
do
done
```

#### シェルスクリプトに実行権限を与える。

```
$ sudo chmod 0755 /opt/hello.sh
```

#### hello.serviceファイル(Unit定義ファイル)を作成。

```
$ sudo nano /etc/systemd/system/hello.service
```

以下を記述。

```
[Unit]
Description = hello daemon

[Service]
ExecStart = /opt/hello.sh
Restart = always
Type = simple

[Install]
WantedBy = multi-user.target
```

設定ファイルの準備が整ったら、以下を順に実行する。再起動し、アプリが自動で起動したらOK。

1. UnitがServiceとして認識されたかどうかの確認
```
$ sudo systemctl list-unit-files --type=service | grep hello
```

2. サービス開始
```
$ sudo systemctl start hello
```

3. サービス停止
```
$ sudo systemctl stop hello
```

4. 自動起動 ON
```
$ sudo systemctl enable hello
```

5. 再起動し、任意のアプリが自動起動するか確認。
```
$ sudo reboot
```

※ 自動起動 OFF
```
$ sudo systemctl disable hello
```
