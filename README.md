# Born2Beroot
A schoolwork on linux at 42.
- [ ]  暗号化LVMを使用してpdfにあるようなパーティションを作成

- virtual boxでのセットアップ時に勝手にやってくれる。
    
    

- [ ]  sudoの設定

- sudoインストール
    
    ```bash
    apt-get update
    apt-get upgrade
    apt-get install sudo
    ```
    
- sudoの設定
    
    ```bash
    visudo
    ```
    
    ```bash
    #User privilege specificationのrootの記述の下
    user ALL=(ALL:ALL) ALL
    ```
    

- [ ]  SSHのポート番号を4242に変更

- opensshのインストール
    
    ```jsx
    apt-get install openssh-server
    ```
    
- sshd_configの中身を編集
    
    > ssh：他のサーバーにsshで接続する設定
    sshd：他のサーバーからsshで接続される時の設定
    sshdは他のサーバーのsshクライアントから実行されたコマンドを実行するため、セキュリティのためsshd_configの内容を変更する。
    > 
    
    ```bash
    vi /etc/ssh/sshd_config
    ```
    
    ```bash
    #Port 4242
    ```
    

- [ ]  rootでSSH接続できないようにする

- sshd_configの中身を編集
    
    ```bash
    vi /etc/ssh/sshd_config
    ```
    
    ```bash
    #PermitRootLogin no
    ```
    
- 設定の反映
    
    ```bash
    service ssh restart
    ```
    
    「設定」→「ネットワーク」→「advanced」→「ポートフォワーディング」→ホストポートとゲストポートを4242,ホストIPを127.0.0.1にして追加
    

![スクリーンショット 2023-03-25 21.50.48.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/38eddb78-0f1b-4a4c-86bf-8db60d3e9f5e/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2023-03-25_21.50.48.png)

- [ ]  UFWをインストールし、ポート4242だけオープンにする

- ufwのインストール
    
    ```bash
    apt-get install ufw
    ufw enable
    ufw status #check whether it’s active
    ufw allow ssh
    ufw allow 4242
    ```
    
- ufwでポートの削除をする（for defense）
    
    ```bash
    ufw status numbered
    ufw delete (num)
    ```
    

- [ ]  ホスト名を`ログイン名`+`42`にする

- ホスト名を変更する（for defense）
    
    ```bash
    hostnamectl #to see the current host name
    hostnamectl set-hostname (name) #to change host name(temporarily)
    ```
    
    上と下の設定どっちもやらないとだめみたい。上の変更はすぐ内部で変更される。
    
    ```bash
    vi /etc/hosts
    ```
    
    ```bash
    127.0.0.1       localhost
    127.0.0.1       (new hostname)
    ```
    

- [ ]  自分のログイン名とするユーザーを作成し、`user42`と`sudo`グループに所属させる

- ユーザーの作成（for defense）
    
    ```bash
    useradd -m (user)
    ```
    
    ```bash
    cut -d: -f1 /etc/passwd #to see the list of users
    ```
    
    ```bash
    userdel -r (user)
    ```
    
- グループの作成（for defense）
    
    ```bash
    groupadd (group)
    ```
    
    ```bash
    cat /etc/group #shows the list of groups that exist
    ```
    
    ```bash
    groups (user) #shows the list of groups that belongs the user
    ```
    
    ```bash
    getent group (group) #shows the list of users that belong to the group
    ```
    
- ユーザーをグループに追加する
    
    ```bash
    gpasswd -a (user) (group)
    ```
    

## パスワードポリシー

- パスワード品質チェックライブラリのインストール
    
    ```bash
    apt-get install libpam-pwquality
    ```
    

```bash
vi /etc/login.defs
```

- [ ]  有効期限は30日

```bash
PASS_MAX_DAYS   30   #l.160
```

- [ ]  最短利用日数は2日

```bash
PASS_MIN_DAYS    2
```

- [ ]  有効期限の7日前に警告メッセージを発する

```bash
PASS_WARN_AGE   7
```

```bash
vi /etc/security/pwquality.conf
```

- [ ]  10文字以上

```bash
minilen=10   #l.11
```

- [ ]  大文字と小文字と数字を含む
- [ ]  最大連続文字数は3
- [ ]  ユーザ名を含まない
- [ ]  変更前に含まれる文字が変更後のパスワードに7文字以上含まれてはならない
- [ ]  rootにも同じポリシーを適用する

```bash
password    requisite         pam_pwquality.so retry=3 **lcredit =-1　ucredit=-1 dcredit=-1 maxrepeat=3 usercheck=0 difok=7 enforce_for_root　　　　　#l.25**
```

```bash
chage -l (user) #to see if the change was affected.(affects only to users           
                #created after the change)
```

## sudoの設定

- `/etc/sudoers`を直接編集してミスると大変らしいので`visudo`を使う
- [ ]  sudo を使用した認証は、パスワードが正しくない場合、試行回数を 3 回に制限する。

```bash
Defaults passwd_tries=3
```

- [ ]  sudoの使用時にパスワードの間違いによるエラーが発生した場合、任意のカスタムメッセージを表示する。

```bash
Defaults badpass_message=”<message>”
```

- [ ]  sudo を使用する各アクションを、入力と出力の両方をアーカイブする。ログファイルは /var/log/sudo/ フォルダに保存。
- ログファイルの作成
    
    ```bash
    root@~:~/ mkdir /var/log/sudo
    root@~:~/ touch /var/log/sudo/sudo.log
    ```
    

```bash
Defaults logfile=”/var/log/sudo/sudo.log”
```

- [ ]  セキュリティ上、TTYモードを有効にする。

```bash
Defaults requiretty
```

- [ ]  また、セキュリティ上の問題から、sudoで使用できるパスを制限する。
    
    ex.)
    
    ```bash
    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
    ```
    

```bash
Defaults secure_path=”/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin”
```

## スクリプト作成

```bash
apt-get install net-tools
```

- monitoring.shファイルを作成
    
    ```bash
    touch /usr/local/bin/monitoring.sh
    ```
    
    ```bash
    chmod 777 /usr/local/bin/monitoring.sh
    ```
    

10分ごとにターミナルに以下の情報を表示する

```bash
nano /usr/local/bin/monitoring.sh
```

- [ ]  オペレーティングシステムのアーキテクチャとそのカーネルバージョン = arc
- [ ]  物理プロセッサの数 = pcpu
- [ ]  仮想プロセッサの数 = vcpu
- [ ]  サーバー上の使用可能なRAMとその使用率 uram / fram = pram(%)
- [ ]  サーバー上の使用可能なメモリとその使用率  udisk / fdisk = pdisk(%)
- [ ]  プロセッサの使用率（パーセント表示）= cpul
- [ ]  最後に再起動した日時が表示されます = lb
- [ ]  LVMがアクティブかどうか = lvmu
- [ ]  アクティブな接続の数 = ctcp
- [ ]  サーバーを使用しているユーザーの数 = ulog
- [ ]  サーバーのIPv4アドレス( = ip)とそのMACアドレス( = mac)
- [ ]  sudoプログラムで実行されたコマンドの数 = cmds

```bash
#!/bin/bash
arc=$(uname -a)
pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l)
vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
fram=$(free -m | awk '$1 == "Mem:" {print $2}')
uram=$(free -m | awk '$1 == "Mem:" {print $3}')
pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
fdisk=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
udisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
pdisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')
cpul=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi)
ctcp=$(ss -neopt state established | wc -l)
ulog=$(users | wc -w)
ip=$(hostname -I)
mac=$(ip link show | grep "ether" | awk '{print $2}')
cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
wall "	#Architecture: $arc
#CPU physical: $pcpu
#vCPU: $vcpu
#Memory Usage: $uram/${fram}MB ($pram%)
#Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
#CPU load: $cpul
#Last boot: $lb
#LVM use: $lvmu
#Connections TCP: $ctcp ESTABLISHED
#User log: $ulog
#Network: IP $ip ($mac)
#Sudo: $cmds cmd"
```
