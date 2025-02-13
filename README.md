# Born2beroot

## 概要
サーバーを構築し、マシンの中にマシンを構築する課題。

---

## 目次
- 注意
- 質問
- コマンド
- sudo 設定
- パスワードポリシー
- 監視スクリプト

---

## 注意
- レビューを開始する前にまず、itermからディレクトリごとコピーし、VirtualBoxソフト内のコピー元ファイルをremoveする、この時全てのファイルを削除しないことに注意。
	`cp -r [VBのディレクトリ] [newname]`
- VirtualBoxソフト内の追加からコピーした仮想環境を選択し起動する

## 質問 (Question)

## 1. Linuxとは?
OSの一つで、無料で使えるオープンソース

## 2. ディストリビューションとは?
Linuxのアプリケーションやライブラリをひとまとめにして、PCにインストールすれば使える状態にした配布物（`Debian`,`Ubuntu`,`Rocky`,`CentOS`）など

## 3. パッケージとは?
ソフトウェア実行に必要なものをファイルにまとめたもの（`実行ファイル`,`ライブラリ`,`設定ファイル`,`リソース`）など

## 4. パッケージ管理とは?
インストール・アンインストールを管理、パッケージをリポジトリから自動で探す、依存関係の解決（`apt`,`yum`,`rpm`）など

## 5. なぜ Debian を選んだのでしょうか?
Debianは安定性が高く、オープンソースでコミュニティ主導で開発されているため、信頼性が非常に高いオペレーティングシステムです。また、パッケージ管理が非常に豊富で、セキュリティの更新が定期的に行われるため、長期運用にも適しています。これらの理由から、サーバー環境やミッション・クリティカルなシステムに適しているとされています。

## 6. Debian と CentOS の違いは何ですか?
- **パッケージ管理**: Debianは`APT`パッケージ管理システムを使用し、Debianパッケージ（.deb）を管理します。CentOSはRed Hat系列のディストリビューションであり、`YUM`や`DNF`パッケージ管理システムを使用し、RPMパッケージ（.rpm）を使用します。
- **開発のアプローチ**: Debianはコミュニティ主導で開発される一方、CentOSはRed Hat社がスポンサーとなり、Red Hat Enterprise Linux（RHEL）のクローンとして開発されます。
- **安定性**: Debianは長期間サポートされる安定性重視のリリースモデルを採用しています。CentOSは、Red Hat Enterprise Linuxの無償版として、企業向けの安定性を提供しますが、アップデートの頻度が低く、企業用途に適しています。

| 項目               | Debian                                        | CentOS                                      |
|--------------------|----------------------------------------------|---------------------------------------------|
| **パッケージ管理**   | `APT`（Advanced Packaging Tool）、`.deb` パッケージ | `YUM`（`DNF`）パッケージ管理、`.rpm` パッケージ |
| **開発のアプローチ** | コミュニティ主導                             | Red Hat社がスポンサー、RHELのクローン    |
| **安定性**          | 長期間サポートされる安定性重視               | 安定性重視、企業向けで長期サポート         |
| **更新頻度**        | 定期的にリリースされ、安定版が重視される       | 更新頻度が低く、企業用途に適している      |
| **サポート**        | 主にコミュニティベース、無償サポート            | Red Hatの商用サポートが提供される        |

## 7. 仮想マシンとは何ですか?
仮想マシン（VM）は、物理的なコンピュータのリソースを仮想的に分割して、複数の独立した仮想環境を構築する技術です。これにより、1台の物理マシン上で複数の異なるオペレーティングシステムを同時に実行することができます。仮想化は、効率的なリソース利用と柔軟性を提供します。

## 8. aptitude と APT (Advanced Packaging Tool) の違いは何ですか?
`APT`（Advanced Packaging Tool）は、Debian系のLinuxディストリビューションで使用されるパッケージ管理システムです。`aptitude`は、APTのフロントエンドであり、より豊富なインターフェースを提供します。具体的には、`aptitude`はコマンドラインインターフェースを提供し、検索やパッケージの依存関係を可視化したり、インタラクティブな操作が可能です。一方、`APT`はよりシンプルで自動化されており、通常はスクリプトで使用されます。

| 項目                      | `aptitude`                                           | `APT`（Advanced Packaging Tool）                     |
|---------------------------|-----------------------------------------------------|----------------------------------------------------|
| **インターフェース**       | よりリッチでインタラクティブなインターフェースを提供 | コマンドラインでのシンプルな操作を提供            |
| **操作の柔軟性**           | パッケージ管理を視覚的に操作でき、依存関係も表示    | スクリプトや自動化に適したツール                   |
| **依存関係の可視化**       | 依存関係のグラフィカルな表示が可能                 | 依存関係の表示はテキストで、詳細表示は少ない      |
| **用途**                   | ユーザー向け、特にパッケージの検索や依存関係管理に便利 | システム管理者向け、効率的でスクリプト利用に向いている |
| **インストールパッケージ** | 自動的にインストールされることが多い                | `apt` や `dpkg` コマンドを使ってパッケージ管理    |

## 9. SELinuxやAppArmorが何なのか?
`AppArmor`（Application Armor）は、Linuxシステムにおけるセキュリティモジュールで、アプリケーションごとにアクセス制御ポリシーを設定し、システム内で許可された操作を制限します。これにより、アプリケーションが不正にシステムリソースにアクセスするのを防ぐことができます。`SELinux`（Security-Enhanced Linux）と同様の目的を持っていますが、構成が比較的簡単で、ユーザーフレンドリーな面があります。

## 10. LVMとは?
`LVM`（Logical Volume Manager）は、Linuxでディスクのパーティション管理を抽象化し、柔軟にストレージを拡張・管理するためのツールです。物理ボリュームを論理ボリュームにまとめることで、データの移動や容量の変更を柔軟に行うことができます。これにより、ストレージの使用を効率化し、システムの管理が容易になります。

## 11. UFW (シンプルなファイアウォール)とは?
`UFW`（Uncomplicated Firewall）は、Linuxで使用される簡単なファイアウォールツールで、iptablesのフロントエンドとして動作します。初心者でも扱いやすく、ファイアウォールの設定や管理が簡単に行えます。`UFW`を使用することで、基本的なファイアウォールの設定を簡単に行い、ネットワークのセキュリティを向上させることができます。

## 12. SSHとは何ですか?
`SSH`ローカルで安全に通信する又は、インターネットを介してリモートのコンピュータと安全に通信するためのプロトコルです。
SSHの通信は暗号化されており、通信途中で盗聴されたとしても内容がバレないためセキュアです。
パスワード認識と公開鍵認証があります。（今回の課題ではパスワード認証が使用されている）
`Ssh name@localhost`の場合、localhostとは自身のマシンを指し、IPアドレスでは127.0.0.1と表記される。又このアドレスは、ループバックアドレスと呼ばれる。
`ループバックアドレス`の用途としては、サーバやアプリケーションの設定の確認や、サービスのテストなどの確認に使用される。
### 利点
- リモート接続をするプロトコルはSSHの他にも`Telnet`などがあるが、通信が暗号化されていなかったのに対し、SSHは通信が暗号化されセキュアになった。
- CUI操作が基本なので、軽量で動作できる。
- 他にもリモート接続のできるプロトコルは目的によっては**Mosh（安定性）、SFTP（安全なファイル転送）、RDP（GUI操作）、VPN（ネットワーク全体の保護）**などを使用する。
### 欠点
- GUI環境に向いていない
- SSHはTCPベースのため、ネットワークが不安定だと切断されやすい
### 結論
セキュリティが高く、軽量で、リモート管理に最適だが、特定の分野には上には上がいる。

## 13. Cronとは何ですか?
`Cron`は、LinuxおよびUnix系オペレーティングシステムで定期的に実行するタスクをスケジュールするためのツールです。ユーザーが指定した時間に指定されたコマンドやスクリプトを自動的に実行することができます。これにより、バックアップやシステムメンテナンスなどの定期的な作業を自動化することができます。

---

## コマンド一覧

### 1. `su` / `sudo`
- **`su`**: スーパーユーザーまたは他のユーザーに切り替えるためのコマンド。通常、`su`の後にユーザー名を指定します。
- **`sudo reboot`**: 管理者権限でシステムを再起動します。

### 2. 情報
- **`sudo -V`**: `sudo`コマンドのバージョン情報を表示します。
- **`lsblk`**: 接続されているブロックデバイス（ハードディスク、SSD、USBドライブなど）のリストを表示
- **`uname -a`**: システムに関する情報を取得するコマンド(選択したディストリビューションを確認できる)


### 3. `sudo apt update`
- **`sudo apt update`**: パッケージリストを更新し、最新の情報を取得します。

### 4. SSH
- **`ssh mtsubasa@localhost -p 4242`**: ローカルホストのポート4242で、`mtsubasa`ユーザーとしてSSH接続します。
- **`sudo systemctl status ssh`**: SSHサービスの状態を確認します。
- **`sudo service ssh status`**: `ssh`サービスのステータスを確認します。
- **`sudo vim /etc/ssh/sshd_config`**: SSHの設定ファイルを編集します。
- **`sudo service ssh restart`**: SSHサービスを再起動します。

### 5. UFW（Uncomplicated Firewall）
- **`sudo ufw status`**: UFW（ファイアウォール）の現在のステータスを確認します。
- **`sudo ufw allow 4242`**: ポート4242をファイアウォールで許可します。
- **`sudo ufw delete allow 4242`**: ポート4242の許可を削除し、拒否します。
- **`sudo ufw enable`**: UFWを有効にします。

### 6. ファイル操作
- **`touch /etc/sudoers.d/sudo_config`**: `/etc/sudoers.d/`ディレクトリに新しい設定ファイルを作成します。
- **`shasum [machinename].vdi`**: 指定したファイルのSHA1ハッシュを計算します。

### 7. インストール
- **`apt install sudo`**: `sudo`パッケージをインストールします。
- **`sudo apt install openssh-server`**: OpenSSHサーバーをインストールします。
- **`sudo apt install ufw`**: UFW（簡易ファイアウォール）をインストールします。
- **`sudo apt install libpam-pwquality`**: パスワード品質を管理するための`libpam-pwquality`パッケージをインストールします。

### 8. ユーザー・グループ管理
- **`sudo adduser <login>`**: 新しいユーザーを作成します。
- **`sudo addgroup user42`**: 新しいグループ`user42`を作成します。
- **`getent group <groupname>`**: 指定したグループの情報を表示します。
- **`sudo adduser <user> <groupname>`**: ユーザーを指定したグループに追加します。

### 9. ファイル編集
- **`sudo vim /etc/ssh/sshd_config`**: SSH設定ファイルを`vim`で編集します。
- **`vim /etc/sudoers.d/sudo_config`**: `sudoers`設定ファイルを編集します。
- **`vim /etc/login.defs`**: ログインに関する設定ファイルを編集します。
- **`vim /etc/pam.d/common-password`**: パスワード管理の設定ファイルを編集します。
- **`vim monitoring.sh`**: `monitoring.sh`スクリプトを編集します。

### 10. `cron`の編集
- **`sudo crontab -u root -e`**: `root`ユーザーのcrontab（定期実行タスク）を編集します。

---

## 設定 (Setting)

### sudo 設定
`vim /etc/sudoers.d/sudo_config`
```ini
passwd_tries=3                      # sudo パスワードを入力するための合計試行回数。
badpass_message="message"           # パスワードが失敗したときに表示されるメッセージ。
logfile="/var/log/sudo/sudo_config" # sudo ログが保存されるパス。
log_input、log_output               # ログに記録される内容。
iolog_dir="/var/log/sudo"           # ログに記録される内容。
requiretty                          # TTY が必須になります。
secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin" # sudo から除外されるフォルダー。
```

### パスワードポリシー設定
#### `vim /etc/login.defs`
```ini
PASS_MAX_DAYS = 最大日数 # パスワードの有効期限。
PASS_MIN_DAYS = 最小日数 # パスワード変更までの最小日数。
PASS_WARN_AGE = 日数     # パスワード警告までの日数。
```

#### `vim /etc/pam.d/common-password`
```ini
minlen=10       # パスワードの最小文字数。
ucredit=-1      # 少なくとも大文字を含む。
dcredit=-1      # 少なくとも数字を含む。
lcredit=-1      # 少なくとも小文字を含む。
maxrepeat=3     # 同じ文字を 3 回以上繰り返せない。
deny_username   # パスワードにユーザー名を含めることはできない。
difok=7         # 以前のパスワードと少なくとも 7 文字異なる必要がある。
enforce_for_root # root ユーザーにも適用。
```

---

## 監視スクリプト (Monitoring Script)
#### `monitoring.sh`
```bash
#!/bin/bash
# ==========================================
# システム情報取得スクリプト
# ==========================================
# 1. システムアーキテクチャ
# 2. CPU（物理コア数 / 仮想CPU数）
# 3. メモリ使用量
# 4. ディスク使用量
# 5. CPU負荷
# 6. 最後の起動時間
# 7. LVMの使用状況
# 8. TCP接続数
# 9. ログインユーザー数
# 10. ネットワーク情報（IP / MAC）
# 11. sudo の実行回数
# ==========================================
# アーキテクチャ情報取得
arch=$(uname -a)
# CPUの物理コア数を取得
cpuf=$(grep "physical id" /proc/cpuinfo | wc -l)
# 論理（仮想）CPU数を取得
cpuv=$(grep "processor" /proc/cpuinfo | wc -l)
# メモリ情報取得
ram_total=$(free --mega | awk '$1 == "Mem:" {print $2}') # メモリ合計（MB）
ram_use=$(free --mega | awk '$1 == "Mem:" {print $3}')   # 使用中メモリ（MB）
ram_percent=$(free --mega | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}') # メモリ使用率（%）
# ディスク使用情報取得
disk_total=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_t += $2} END {printf ("%.1fGb\n"), disk_t/1024}') # ディスク総容量
disk_use=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}') # 使用済みディスク容量
disk_percent=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} {disk_t+= $2} END {printf("%d"), disk_u/disk_t*100}') # ディスク使用率
# CPU負荷情報取得
cpul=$(vmstat 1 2 | tail -1 | awk '{printf $15}') # CPUアイドル時間取得
cpu_op=$(expr 100 - $cpul) # CPU使用率 = 100 - アイドル時間
cpu_fin=$(printf "%.1f" $cpu_op) # 小数点1桁で表示
# 最後の起動時間を取得
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
# LVM（論理ボリューム管理）の使用状況を確認
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)
# 確立済みのTCP接続数を取得
tcpc=$(ss -ta | grep ESTAB | wc -l)
# ログイン中のユーザー数を取得
ulog=$(users | wc -w)
# ネットワーク情報取得
ip=$(hostname -I) # IPアドレス
mac=$(ip link | grep "link/ether" | awk '{print $2}') # MACアドレス
# sudoコマンドの実行回数を取得
cmnd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
# システム情報をすべてのユーザーに通知
wall "
	# アーキテクチャ情報を表示
	Architecture: $arch
	# 物理CPUの数を表示
	CPU physical: $cpuf
	# 仮想CPU（vCPU）の数を表示
	vCPU: $cpuv
	# メモリ使用量を表示（使用メモリ / 総メモリ）と使用率（%）を表示
	Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
	# ディスク使用量を表示（使用ディスク容量 / 総ディスク容量）と使用率（%）を表示
	Disk Usage: $disk_use/${disk_total} ($disk_percent%)
	# CPU負荷のパーセントを表示
	CPU load: $cpu_fin%
	# 最後にシステムが起動した日時を表示
	Last boot: $lb
	# LVM（Logical Volume Management）が使用されているかどうかを表示
	LVM use: $lvmu
	# 確立されているTCP接続数を表示
	Connections TCP: $tcpc ESTABLISHED
	# 現在システムにログインしているユーザー数を表示
	User log: $ulog
	# システムのIPアドレスとMACアドレスを表示
	Network: IP $ip ($mac)
	# sudo コマンドで実行されたコマンドの数を表示
	Sudo: $cmnd cmd
"
```
## 詳細説明 (Detail Explain)

## 詳細説明

### システムアーキテクチャ関連
```bash
arch=$(uname -a)
```
- `uname`: システムに関する情報を取得するコマンド
- `-a`: 以下の情報をすべて表示
  - カーネル名（OSの種類）
  - ホスト名
  - カーネルリリース
  - カーネルバージョン
  - マシンのハードウェア名
  - プロセッサの種類
  - ハードウェアプラットフォーム
  - オペレーティングシステム名
- `$()`: コマンド置換

### CPU情報関連
```bash
cpuf=$(grep "physical id" /proc/cpuinfo | wc -l)
cpuv=$(grep "processor" /proc/cpuinfo | wc -l)
```
- `grep`: 指定ファイルから抽出
- `physical id`: 物理CPU（ソケット）の識別番号
- `/proc/cpuinfo`: CPUに関する情報を含む仮想ファイル
- `wc`: カウント
- `-l`: 行数
- `processor`: 論理CPU（スレッド）の識別番号

### メモリ使用状況関連
```bash
ram_total=$(free --mega | awk '$1 == "Mem:" {print $2}')
ram_use=$(free --mega | awk '$1 == "Mem:" {print $3}')
ram_percent=$(free --mega | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
```
- `free`: システムのメモリ使用状況を表示
- `--mega`: 単位をMB（メガバイト）で表示
- `awk`: テキスト処理を行うツール

free --megaの出力例：
```
              total        used        free      shared  buff/cache   available
Mem:           16000        8000        2000         500        6000        7000
Swap:           8192        1000        7192
```
- `$1 == "Mem:"`: 1列目が"Mem:"である行を抽出
- `{print $2}`: 2列目（totalの値）を出力
- `{print $3}`: 3列目（usedの値）を出力
- `{printf("%.2f"), $3/$2*100}`: 小数点2桁までのフォーマットで使用率を計算

### ディスク使用状況関連
```bash
disk_total=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_t += $2} END {printf ("%.1fGb\n"), disk_t/1024}')
disk_use=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}')
disk_percent=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} {disk_t+= $2} END {printf("%d"), disk_u/disk_t*100}')
```
- `df`: ディスクの使用状況を表示
- `-m`: 単位がMB（メガバイト）

df -mの出力例：
```
Filesystem     1M-blocks  Used Available Use% Mounted on
udev              7999      0      7999   0% /dev
tmpfs             1600      2      1598   1% /run
/dev/sda1       512000 200000    312000  39% /
/dev/sdb1      1024000 500000    524000  49% /mnt/data
/dev/sdc1       256000  80000    176000  31% /boot
```
- `-v`: 除外（逆マッチ）
- `{disk_t += $2}`: 2列目（総サイズ）をdisk_tに加算
- `{disk_u += $3}`: 3列目（使用済み容量）をdisk_uに加算
- `END`: すべての行の処理が終わった後に実行されるブロック

### CPU負荷関連
```bash
cpul=$(vmstat 1 2 | tail -1 | awk '{printf $15}')
cpu_op=$(expr 100 - $cpul)
cpu_fin=$(printf "%.1f" $cpu_op)
```
- `vmstat`: システムのメモリ、プロセス、I/O、CPU使用率などを表示
- `1`: 1秒間隔で情報を表示
- `2`: 2回のデータ取得
- `tail`: 後ろから抽出
- `-1`: 最後の1行
- `expr`: 数値計算

### LVM状態関連
```bash
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)
```
- `lsblk`: ブロックデバイス（ディスクやパーティション）の情報を表示
- `lvm`: LVM（論理ボリューム）
- `-gt 0`: 0より大きい
- `echo yes/no`: LVMの有無により出力を変更

### ネットワーク関連
```bash
tcpc=$(ss -ta | grep ESTAB | wc -l)
ulog=$(users | wc -w)
ip=$(hostname -I)
mac=$(ip link | grep "link/ether" | awk '{print $2}')
```
- `ss`: ソケット統計を表示
- `-t`: TCP接続のみ表示
- `-a`: すべてのソケットを表示
- `ESTAB`: "Established"（確立）の略
- `users`: ログインユーザーを表示
- `-w`: 単語数
- `hostname -I`: IPアドレスを表示
- `ip link`: ネットワークインターフェース情報を表示

### sudo使用状況関連
```bash
cmnd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
```
- `journalctl`: systemdのログ管理ツール
- `_COMM=sudo`: sudoコマンドのログのみ抽出
- `COMMAND`: 実行されたコマンドの記録

### 情報表示
```bash
wall "..."
```
- `wall`: メッセージをすべてのユーザーに表示

---
