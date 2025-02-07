# Born2beroot

## 概要
サーバーを構築し、マシンの中にマシンを構築する課題。

---

## 質問 (Question)
- **なぜ Debian を選んだのでしょうか?**
- **Debian と CentOS の違いは何ですか?**
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
```
## 詳細説明 (Detail Explain)

