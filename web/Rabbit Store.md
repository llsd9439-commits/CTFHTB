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

что бы авторизоваться нам приходиться добавить storage.cloudsite.thm в /etc/hosts/

<img width="2560" height="900" alt="{B870F9F1-549E-491D-80E2-95F8564C4058}" src="https://github.com/user-attachments/assets/220aedeb-2629-4b59-b974-7b809f5aebf3" />

в бурпе нашел jwt решил посмотреть что и как

<img width="984" height="694" alt="{763D1177-61FD-4100-8E5C-029B2CDE6877}" src="https://github.com/user-attachments/assets/cd02dd10-f13a-458a-a32f-c8d2fa3fd8d4" />


при логине показывает inactive от сервера, если бы был секретный ключ то мы могли бы подделать jwt, но увы у нас его нету




так что придем к другому варианту, попробуем ввести в /api/register - subscription active 


<img width="1520" height="742" alt="{D7538C7C-DDDE-46D9-B505-82EA4C4A6EC0}" src="https://github.com/user-attachments/assets/c83935b0-57c6-4bee-b204-53ca9f467e11" />


<img width="2560" height="1285" alt="{8216FFDC-E607-4B0E-8398-ADE0855B4D02}" src="https://github.com/user-attachments/assets/4de573cb-4f0b-4455-b00c-cc51c0e33fa5" />




