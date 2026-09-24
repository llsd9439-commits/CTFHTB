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


