# Born2beroot

## 概要
サーバーを構築し、マシンの中にマシンを構築する課題。

---

## 質問 (Question)
- **Q1. なぜ Debian を選んだのでしょうか?**
- **Q2. Debian と CentOS の違いは何ですか?**
- **仮想マシンとは何ですか?**
- **aptitude と APT (Advanced Packaging Tool) の違いは何ですか?**
- **AppArmor とは何ですか?**
- **LVMとは?**
- **UFW (シンプルなファイアウォール)とは?**
- **SSHとは何ですか?**
- **Cronとは何ですか?**
- **aptitudeとaptの違いや、SELinuxやAppArmorが何なのか?**

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

#!/bin/bash

# システムアーキテクチャ情報の取得
arch=$(uname -a)
# uname: システムに関する情報を取得するコマンド
# -a: カーネル名（OSの種類）、ホスト名、カーネルリリース、カーネルバージョン、
#     マシンのハードウェア名、プロセッサの種類、ハードウェアプラットフォーム、OSの名前
# $(): コマンド置換

# CPU情報の取得
cpuf=$(grep "physical id" /proc/cpuinfo | wc -l)
# grep: 指定ファイルから抽出
# physical id: 物理CPU（ソケット）の識別番号
# /proc/cpuinfo: CPUに関する情報を含む仮想ファイル
# wc: カウント
# -l: 行数

cpuv=$(grep "processor" /proc/cpuinfo | wc -l)
# processor: 論理CPU（スレッド）の識別番号

# メモリ使用状況の取得
ram_total=$(free --mega | awk '$1 == "Mem:" {print $2}')
ram_use=$(free --mega | awk '$1 == "Mem:" {print $3}')
ram_percent=$(free --mega | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
# free: システムのメモリ使用状況を表示
# --mega: 単位をMB（メガバイト）で表示
# awk: テキスト処理を行うツール

# free --megaの例
#               total        used        free      shared  buff/cache   available
# Mem:          16000        8000        2000         500        6000        7000
# Swap:          8192        1000        7192

# $1 == "Mem:": 1列目（最初のフィールド）が"Mem:"である行を抽出
# {print $2}: 2列目（= totalの値）を出力
# {print $3}: 3列目（= usedの値）を出力
# {printf("%.2f"), $3/$2*100}: 小数点2桁までのフォーマットで表示
#                              (used / total) * 100でメモリ使用率を計算

# ディスク使用状況の取得
disk_total=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_t += $2} END {printf ("%.1fGb\n"), disk_t/1024}')
disk_use=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}')
disk_percent=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} {disk_t+= $2} END {printf("%d"), disk_u/disk_t*100}')
# df: ディスクの使用状況を表示
# -m: 単位がMB（メガバイト）

# df -mの例
# Filesystem     1M-blocks  Used Available Use% Mounted on
# udev               7999      0      7999   0% /dev
# tmpfs              1600      2      1598   1% /run
# /dev/sda1        512000 200000    312000  39% /
# /dev/sdb1       1024000 500000    524000  49% /mnt/data
# /dev/sdc1        256000  80000    176000  31% /boot

# -v: 除外（逆マッチ）
# {disk_t += $2}: 2列目（総サイズ）をdisk_tに加算していく
# disk_t: 総ディスク容量
# {disk_u += $3}: 3列目（使用済み容量）をdisk_uに加算していく
# disk_u: 使用中のディスク容量
# {printf("%d"), disk_u/disk_t*100}: 使用容量を総容量で割り使用率を計算
# END: すべての行の処理が終わった後に実行されるブロック

# CPU負荷の計算
cpul=$(vmstat 1 2 | tail -1 | awk '{printf $15}')
cpu_op=$(expr 100 - $cpul)
cpu_fin=$(printf "%.1f" $cpu_op)
# vmstat: システムのメモリ、プロセス、I/O、CPU使用率などを表示
# 1: 1秒間隔で情報を表示
# 2: 2回のデータ取得
# tail: 後ろから抽出
# -1: 最後の1行
# expr: 数値計算

# LVM確認
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)
# lsblk: ブロックデバイス（ディスクやパーティション）の情報を表示
# lvm: LVM（論理ボリューム）
# if [ $(...) -gt 0 ]: LVM関連の行が1行以上あるかを確認
# -gt 0: 0より大きい
# echo yes: 真（LVMが存在する）の場合
# echo no: 偽（LVMが存在しない）の場合

# TCP接続数の取得
tcpc=$(ss -ta | grep ESTAB | wc -l)
# ss: ソケット統計を表示、ネットワーク接続の状態を確認
# -t: TCP接続だけを表示
# -a: すべてのソケット（リスニング、確立済みなど）を表示
# ESTAB:（確立状態）"Established"（確立）の略

# ログインユーザー数の取得
ulog=$(users | wc -w)
# users: 現在システムにログインしているユーザーを表示
# -w: 単語数

# ネットワークアドレス情報の取得
ip=$(hostname -I)
mac=$(ip link | grep "link/ether" | awk '{print $2}')
# hostname: システムのホスト名やネットワーク情報を表示
# -I: システムに割り当てられたIPアドレスを表示
# ip link: ネットワークインターフェースの状態を表示

# sudo使用回数の取得
cmnd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
# journalctl: systemdのログ管理ツール
# _COMM=sudo: sudoコマンドで実行されたログエントリのみをフィルタリング
# COMMAND: sudoを使って実行された実際のコマンドがログに記録される際の文字列

# 情報の表示
wall "	Architecture: $arch
	CPU physical: $cpuf
	vCPU: $cpuv
	Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
	Disk Usage: $disk_use/${disk_total} ($disk_percent%)
	CPU load: $cpu_fin%
	Last boot: $lb
	LVM use: $lvmu
	Connections TCP: $tcpc ESTABLISHED
	User log: $ulog
	Network: IP $ip ($mac)
	Sudo: $cmnd cmd"
# wall: メッセージをすべてのユーザーに表示