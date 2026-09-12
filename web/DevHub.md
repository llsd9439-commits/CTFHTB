
<img width="1718" height="329" alt="{868FEA7D-34CB-4AA8-9C55-06D6B1946318}" src="https://github.com/user-attachments/assets/ce828c76-0149-49e5-baf8-db4783e60329" />

# NMAP 

<img width="829" height="225" alt="{8918B2BE-E950-4843-8517-B7DEECE4C22C}" src="https://github.com/user-attachments/assets/f2fb0cea-bac2-4f54-9f69-17cbf2e35bd1" />

по сканеру портов вижу что есть 80 и еще какой то странный порт, решаюсь сначала зайти на 80

<img width="1616" height="969" alt="{81897AE5-3555-4349-8A4F-F65BEF9950B6}" src="https://github.com/user-attachments/assets/9a8be2c7-eaf5-4d41-a38a-19a637c29b95" />

он отсылает на 6274 который как раз светился в сети

<img width="1643" height="1051" alt="{D004EDD8-4AF2-42F1-9748-81BD10C4C502}" src="https://github.com/user-attachments/assets/e0766f4f-acee-4475-9323-ee73ef934718" />

# RCE

гуглю cve и нахожу CVE-2026-23744 

https://github.com/suljov/CVE-2026-23744-Remote-Code-Execution-POC/blob/main/exploit.py

<img width="762" height="282" alt="{00900DF3-77E2-457A-96D2-F585DBC7E60D}" src="https://github.com/user-attachments/assets/4c8d94fc-a5b1-401f-beb5-54a7cc0d2763" />

# LPE 

Вспоминаю про 8888 порт 

<img width="855" height="251" alt="{B463141B-6A11-4A20-B8EA-1A2E0AFB6089}" src="https://github.com/user-attachments/assets/85715b3e-a36c-401a-93c2-a8120597ad25" />

в итоге вижу это

<img width="708" height="334" alt="{5BD22291-0721-4AEA-80EE-580386D8FE6B}" src="https://github.com/user-attachments/assets/c8cd7bed-3194-4921-a6e6-d33cffc66c3c" />

след действие, узнать что это и от какого юзера запущено

<img width="1459" height="334" alt="{7478D4A1-6A86-4ECD-A951-7D6834AC41EB}" src="https://github.com/user-attachments/assets/c8f4d58b-3f42-4f6e-95a3-ce017678ab14" />

видим что 8888 запущен от имени пользователя analyst, 5000 от root. скорее всего сначала до root на не дадут повыситься, так что идем к 8888 порту, для начала хочу провести соединение через chisel
так, ну я настроил chisel таким образом - машина к которой я получил доступ - мой vps - мой пк 

в итоге вижу jupyter, но к нему нужен пароль либо токен
<img width="1874" height="152" alt="{5FDA69AF-BF84-43E0-B89E-D74240C5CB50}" src="https://github.com/user-attachments/assets/0b065997-d609-439d-aea0-2cae0f94fa3b" />

ну как бы вот он и токен

вот мы и повысились 
<img width="1642" height="1049" alt="{0FBDCFBC-8B8A-4DF0-958D-08D9EEF2AAF3}" src="https://github.com/user-attachments/assets/d590983d-a413-4723-b588-62c23b6f7b12" />


# повышение до root

<img width="685" height="149" alt="{0C655FF6-C438-41D5-BA64-BD03BEF3481A}" src="https://github.com/user-attachments/assets/779e323b-9df6-48ef-bda5-28c9cc49237b" />
опа, а это что то интересноееее, ану ка прочитаем че тут 
проанализировав код мы можем составить вот такой вот пайлоуд 
curl -s -X POST http://127.0.0.1:5000/tools/call -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" -H "Content-Type: application/json" -d '{"name": "ops._admin_dump", "arguments": {"target": "ssh_keys", "confirm": true}}'


<img width="1875" height="425" alt="{FA6935C9-0DF6-45E4-889B-281C05F37C4E}" src="https://github.com/user-attachments/assets/7913f91e-b822-45ec-a5f8-04a51e1166ae" />

кидаю этот ключ к себе и получаю root

<img width="1835" height="803" alt="{44D51FC6-5F12-4EBE-96D7-1BB392C5E619}" src="https://github.com/user-attachments/assets/371a150a-ad99-45f5-b271-1697867dab74" />










