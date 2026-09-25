<img width="1930" height="424" alt="{4841F8B6-50B2-4A72-8FD5-8FAFBC4CC17C}" src="https://github.com/user-attachments/assets/e0c5d813-3568-4eec-900f-17c48f2af5f8" />

просканировав порты увидел 22 и 80 порт
на главной странице ничего не нашел, но поискал vhost и нашел http://site.wekor.thm/
<img width="1310" height="462" alt="{A76AAB99-CFD7-4832-AE2F-2801E2324E42}" src="https://github.com/user-attachments/assets/7da7454e-d3e4-4226-980f-4a134b0a1c27" />

на главной странице написано 
Hi there! Nothing here for now, but there should be an amazing website here in about 2 weeks, SO DON'T FORGET TO COME BACK IN 2 WEEKS! - Jim
есть пользователь Jim - в дальнейшем я думаю нам это пригодиться 

через fuff нахожу wordpress 
<img width="2372" height="892" alt="{532D91F8-2C68-4800-8541-62C5EF740E00}" src="https://github.com/user-attachments/assets/4d8daaeb-c6e0-447f-b5f0-2d2064a86246" />

использую утилиту wpscan для проверки уязвимостей wordpress, вручную попытался посмотреть файл index.php/wp-json/wp/v2/users/ нашел пользователя admin
---- я думаю что скорее всего в этой задачи придется брутить пароль от админа, но в нынешних реалиях забрутить пароль от какого то пользователя очень трудно ( ну если конечно вы не имеете крутой ботнет, но не об этом
мне не очень хочется составлять запрос под hydra по этому я решил забрутить через wpscan, параллельно пытаюсь использовать базовые креды пока идет брутфорс 
<img width="1473" height="610" alt="{68B294B8-7B39-4CED-B449-91016D5DF0DE}" src="https://github.com/user-attachments/assets/9a7fc4ec-ac9f-4176-9d4f-5cf984b3ded2" />

все таки это очень долго, ладно, сделаем как и всегда, через hydra

<img width="1522" height="878" alt="{E6548FCD-AA72-4B0F-B1D2-256E09DCC77D}" src="https://github.com/user-attachments/assets/592b717f-104a-4ef4-bed8-7635c793d37d" />

в итоге через многое время я ничего не нашел((

решил вернуться к wekor.thm и посмтреть что там в robots.txt
нашел /comingreallysoon/
а там же Welcome Dear Client! We've setup our latest website on /it-next, Please go check it out! If you have any comments or suggestions, please tweet them to @faketwitteraccount! Thanks a lot !

<img width="1586" height="785" alt="{D7C25C8E-15F4-4B14-B9D8-F714B385E896}" src="https://github.com/user-attachments/assets/913092af-8125-402f-9f47-a1938511b495" />

sqlmap <img width="1258" height="791" alt="{9045A1EB-D13F-4DAA-9628-83F250CE3F52}" src="https://github.com/user-attachments/assets/917bb33f-a131-4ef5-a6f3-4f38c1a6f961" />


<img width="1494" height="212" alt="{2ABA5282-ECE7-4833-B956-CA531FCCAB22}" src="https://github.com/user-attachments/assets/d494c7e9-5602-40a4-a3b0-20f6fac87830" />


нуууу кароче кидаем хэши и брутим их

┌──(root㉿lsd)-[/home/lsd]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (phpass [phpass ($P$ or $H$) 256/256 AVX2 8x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
rockyou          (?)     
xxxxxx           (?)     
2g 0:00:00:00 DONE (2026-09-24 16:43) 20.00g/s 11520p/s 17280c/s 17280C/s evelyn..summer1
Use the "--show --format=phpass" options to display all of the cracked passwords reliably
Session completed. 
<img width="1639" height="959" alt="{705BD7EA-150A-4D9B-810B-EA159D329C00}" src="https://github.com/user-attachments/assets/145e244e-a1b6-49ac-a65d-5c5e87ab9a2a" />

у него нету прав, зашел на wp_yura <img width="1589" height="1005" alt="{9F8691CC-2D8F-46A9-A419-0D41CD190463}" src="https://github.com/user-attachments/assets/20d18a53-131d-4695-b8b0-b07b5cf31f29" />

получение rce через многие пути возможны в wordpress, посмотрим что уготовили нам тут
<img width="1610" height="495" alt="{90A2075E-149A-4066-85DA-6B53728B3D45}" src="https://github.com/user-attachments/assets/71f2bf3a-16d5-460d-ae55-2dbb926b6b4b" />
я мог редачить темы, туда закинул cmd.php
по ss увидел что открыт Memcached, там же и увидел пароль от Orka

#ROOT


<img width="1088" height="231" alt="{16D29869-EA0E-4DBC-8831-8A4820524FA8}" src="https://github.com/user-attachments/assets/bc917ba7-7d45-4663-87cb-73f181fb4755" />

<img width="1031" height="853" alt="{AA71A9B8-591A-402E-A2AD-58E1EB90748D}" src="https://github.com/user-attachments/assets/429bca19-2349-4093-abd4-df860a63e334" />

перенесем этот биткоин себе на машину что бы узнать какой там пароль, ну или же посмотрим есть ли там возможность получить как то bash с этого скрипта, потому что именного его мы можем запускать от root

<img width="1162" height="924" alt="{6D551B48-BC59-4FD9-814E-6284A700A69A}" src="https://github.com/user-attachments/assets/c32583a3-bedd-412f-b8ee-c8e6544a237e" />
кароче разреверсил, увидел что там есть пароль, и то что он вызывает питон команду, через python по linpeas видел что есть доступ на запись /usr/sbin

<img width="708" height="519" alt="{69432DA5-7902-4ABE-A09D-FF687D5AC57F}" src="https://github.com/user-attachments/assets/e6e7ab3d-8bac-45d4-9ee5-dd94c55f80cf" />







                     


