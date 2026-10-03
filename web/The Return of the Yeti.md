Всем привет, сегодня мы решаем вот такую вот интересную задачку

<img width="1474" height="1001" alt="{C0D5CE65-42E6-4D2E-B9E5-FFC7EDF3DA4D}" src="https://github.com/user-attachments/assets/b973204a-d442-4e1c-82b5-9db2b6c693f7" />

### Как называется сеть Wi-Fi в PCAP?

первое что нас встречает это pcap файл, с которым мы будем работать
<img width="2145" height="63" alt="{12AE69BF-5224-4BC0-8640-DFC06E3ADC55}" src="https://github.com/user-attachments/assets/e135c4d9-666d-4534-8975-3c372f6d3d4c" />

ОТВЕТ - FreeWifiBFC

### Какой пароль для доступа к сети Wi-Fi?

Проанализируем дальше наш pcap
<img width="2557" height="228" alt="image" src="https://github.com/user-attachments/assets/788c70fd-db6b-4105-93b5-3cbd17b5ee00" />

наблюдаем ключи, мы сможем взломать их используя утилиту hcxpcapngtool

```
hcxpcapngtool -o hash.hc22000 '/home/lsd/Desktop/VanSpy.pcapng' 

cat hash.hc22000
WPA*02*c10a70d965945b57f2988ae0fcfd2b22*22c712c7e235*2a8449acf9d8*4672656557696669424643*74c790c8298bbf043890464c140f0ab2b824e65ca79810f5ac80fafe11bb9683*0103007502010a0000000000000000000195fe9674fc28a5911c78e951df9462a44042036211dbc8e1029cf3dd1d103788000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac040100000fac040100000fac020000*82
WPA*02*af804935ab322d4b0bbd84a711c36f01*22c712c7e235*d267d13f36ec*4672656557696669424643*d1744febb5c51179be135c004ef20b75966d8b599f42152c3892d5eaa93cdd0a*0103007502010a00000000000000000001e90fb51bc1b5480c37380d19d7c83c82dbc782923d28eeb93dfc4e5089cd5732000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac040100000fac040100000fac020000*82

```

как мы видим, хэш мы получили

#### Взлом хэша

└─$ hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt

af804935ab322d4b0bbd84a711c36f01:22c712c7e235:d267d13f36ec:FreeWifiBFC:Christmas
c10a70d965945b57f2988ae0fcfd2b22:22c712c7e235:2a8449acf9d8:FreeWifiBFC:Christmas

Ответ - Christmas 

### Какой подозрительный инструмент использует злоумышленник, чтобы извлечь важный файл с сервера?

мы расшифровываем трафик, теперь мы можем читать его


<img width="1431" height="1029" alt="{0BBC402A-DC5D-4172-8E0D-DAED6FFEABA4}" src="https://github.com/user-attachments/assets/9e2d77fd-54a5-43f8-abfb-e14e30afb208" />

Ответ - mimikatz


### Какой номер дела присвоила Киберполиция для решения проблем, о которых сообщил McSkidy?
для этого нам нужно найти ключ что бы убрать защиту tls, мы это сделаем через это

<img width="2560" height="1270" alt="{1FBF31A9-F463-4E81-BA85-F7DC006AD08F}" src="https://github.com/user-attachments/assets/8f1fad19-5d7c-4222-a5e2-8fc0745b2344" />


<img width="1366" height="598" alt="{EF7061EE-6682-41C0-AD09-526821079B3B}" src="https://github.com/user-attachments/assets/1ca5d552-a085-4141-9a1f-648d7c156131" />

```
┌──(lsd㉿lsd)-[~]
└─$ openssl pkcs12 -in rdp_privkey.pfx -nocerts -out rdp_privkey.pem -passin pass:mimikatz

Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
Error outputting keys and certificates
4077606D0E7F0000:error:14000065:UI routines:UI_set_result_ex:result too small:../crypto/ui/ui_lib.c:898:You must type in 4 to 1024 characters
4077606D0E7F0000:error:1400006B:UI routines:UI_process:processing error:../crypto/ui/ui_lib.c:553:while reading strings
4077606D0E7F0000:error:0480006D:PEM routines:PEM_def_callback:problems getting password:../crypto/pem/pem_lib.c:62:
4077606D0E7F0000:error:07880109:common libcrypto routines:do_ui_passphrase:interrupted or cancelled:../crypto/passphrase.c:178:
4077606D0E7F0000:error:1C80009F:Provider routines:p8info_to_encp8:unable to get passphrase:../providers/implementations/encode_decode/encode_key2any.c:122:
                                                                                                                                                                                        
┌──(lsd㉿lsd)-[~]
└─$ 
                                                                                                                                                                                        
┌──(lsd㉿lsd)-[~]
└─$ openssl rsa -in rdp_privkey.pem -out rdp_privkey_nopass.pem
Could not find private key from rdp_privkey.pem
40578BF5227F0000:error:1608010C:STORE routines:ossl_store_handle_load_result:unsupported:../crypto/store/store_result.c:160:provider=default
40578BF5227F0000:error:1608010C:STORE routines:ossl_store_handle_load_result:unsupported:../crypto/store/store_result.c:160:provider=default
40578BF5227F0000:error:1E08010C:DECODER routines:OSSL_DECODER_from_bio:unsupported:../crypto/encode_decode/decoder_lib.c:104:No supported data to decode. Input structure: EncryptedPrivateKeyInfo
                                                                                                                                                                                        
┌──(lsd㉿lsd)-[~]
└─$ openssl pkcs12 -in rdp_privkey.pfx -nocerts -out rdp_privkey.pem -passin pass:mimikatz

Enter PEM pass phrase:
Error outputting keys and certificates
4047C7D9627F0000:error:14000065:UI routines:UI_set_result_ex:result too small:../crypto/ui/ui_lib.c:898:You must type in 4 to 1024 characters
4047C7D9627F0000:error:1400006B:UI routines:UI_process:processing error:../crypto/ui/ui_lib.c:553:while reading strings
4047C7D9627F0000:error:0480006D:PEM routines:PEM_def_callback:problems getting password:../crypto/pem/pem_lib.c:62:
4047C7D9627F0000:error:07880109:common libcrypto routines:do_ui_passphrase:interrupted or cancelled:../crypto/passphrase.c:178:
4047C7D9627F0000:error:1C80009F:Provider routines:p8info_to_encp8:unable to get passphrase:../providers/implementations/encode_decode/encode_key2any.c:122:
                                                                                                                                                                                        
┌──(lsd㉿lsd)-[~]
└─$ openssl pkcs12 -in rdp_privkey.pfx -nocerts -out rdp_privkey.pem -passin pass:mimikatz
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
                                                                                                                                                                                        
┌──(lsd㉿lsd)-[~]
└─$ openssl rsa -in rdp_privkey.pem -out rdp_privkey_nopass.pem
Enter pass phrase for rdp_privkey.pem:
writing RSA key
                                                                                                                                                                                        
```




