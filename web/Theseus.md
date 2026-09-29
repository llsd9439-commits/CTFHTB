<img width="1493" height="991" alt="{CBC06C2B-1393-47BB-937C-7D37CD1EE69B}" src="https://github.com/user-attachments/assets/1341b977-05dd-4980-986b-bf88c70788d3" />

#СКАНИРОВАНИЕ ПОРТОВ

<img width="893" height="222" alt="{1D404F36-BB46-4C74-8FA1-8A7211201382}" src="https://github.com/user-attachments/assets/63604373-3626-4d5f-82aa-d6f1488dbb33" />
на веб странице вижу это

<img width="858" height="874" alt="{E1C490BF-D752-4B86-9138-732782537CBF}" src="https://github.com/user-attachments/assets/86b99402-6755-47de-ae4d-6af6352ee099" />

#РАСШИФРОВКА ШИФРА

TGUE?O·S·K·MTUEGI·SYENFE·TOI···SRO·T·SF·OYT···O·T·KUMH·I·AE·NMK·

использовав ии я понял что это Scytale Cipher, использовал https://atbashcipher.com/tools/scytale-cipher и получил это

<img width="1143" height="857" alt="{4C15DBE0-47BF-4212-B094-8590B4B1DC28}" src="https://github.com/user-attachments/assets/fe54795a-5e97-45b9-b4b1-3697da29e5f5" />

TO·GET·TO·KING·MINOS·YOU·MUST·FIRST·MAKE·USE·OF·THE·?KEY·········

первое что приходит в голову это то что на вебе есть что то связанное с ?key
проверив я увидел это 

<img width="1933" height="396" alt="{1E67B512-2A12-4253-926D-0AB3F2DB7CFE}" src="https://github.com/user-attachments/assets/f49d0285-f1a0-42ef-a8b1-e113c83a0ad1" />

я решил попробовать ssti ввести http://target?key={{7*7}}, я получил 49. Погуглю poc

<img width="1923" height="289" alt="{B114AAC0-38FA-4FC9-860C-522A184428E7}" src="https://github.com/user-attachments/assets/764894cc-006e-49a2-b0ed-b8611705ca67" />

<img width="811" height="213" alt="{16C478BC-0829-4846-953E-14B97A6B0E1F}" src="https://github.com/user-attachments/assets/33a8fd3e-7109-4fb9-b749-1b9ebe672cc9" />
ура, мы получили rce

<img width="391" height="107" alt="{F7628BD6-C81E-4756-B6F0-823115DBB9C5}" src="https://github.com/user-attachments/assets/acecc049-a170-4adf-bccb-a49f77458653" />


Theseus insisted he knew the dangers but
would succeed in his journey to Crete. 
As the ship left the harbour wall he 
shouted to his father King Aegeus "and 
you will be proud of your son".

"Then I wish you luck, my son, I shall
watch for you every day. If you are
successful, take down these black sails
and replace them with white ones. That way 
I will know you are coming safe to me."

As the ship docked in Crete, King Minos himself
came down to inspect the prisoners from Athens.
He enjoyed the chance to taunt the Athenians
and to humiliate them even further.

As King Minos jeered as to who would enter the
labyrinth first, Theseus stepped forward.

"I will go first. I am Theseus, Prince of Athens,
and I do not fear what is within the walls of
your maze."

"Those are brave words for one so young and 
feeble, but the Minotaur will soon have you
between its horns. Guards, open the labyrinth
and let him in!"

Username: entrance
Password: Knossos

<img width="1144" height="178" alt="{925F125E-7A0C-4FC3-B5E5-BBE1B3477B0F}" src="https://github.com/user-attachments/assets/01cede5f-63ea-4482-a84e-3634cbeee097" />


так как оригинальный гитефобин вбанили а точнее там произошли какие то изменения ( из-за чего много lpe пропали ) я использовал вебархив
<img width="1071" height="735" alt="{AB44C22A-331C-4E4B-B95B-1132802253AE}" src="https://github.com/user-attachments/assets/d1169f26-f2e8-4690-be26-c3d03b131070" />

ну и я получил root
я закинул свой ключ в ssh что бы получить удобный tty, по итогу увидел что после того как я захожу по ssh у меня юзается 
root@Minos:~# cat minotaur

       -""\
    .-"  .`)     (
   j   .'_+     :[                )      .^--..
  i    -"       |l                ].    /      i
 ," .:j         `8o  _,,+.,.--,   d|   `:::;    b
 i  :'|          "88p;.  (-."_"-.oP        \.   :
 ; .  (            >,%%%   f),):8"          \:'  i
i  :: j          ,;%%%:; ; ; i:%%%.,        i.   `.
i  `: ( ____  ,-::::::' ::j  [:```          [8:   )
<  ..``'::::8888oooooo.  :(jj(,;,,,         [8::  <
`. ``:.      oo.8888888888:;%%%8o.::.+888+o.:`:'  |
 `.   `        `o`88888888b`%%%%%88< Y888P""'-    ;
   "`---`.       Y`888888888;;.,"888b."""..::::'-'
          "-....  b`8888888:::::.`8888._::-"
             `:::. `:::::O:::::::.`%%'|
              `.      "``::::::''    .'
                `.                   <
                  +:         `:   -';
                   `:         : .::/
                    ;+_  :::. :..;;;       
                    ;;;;,;;;;;;;;,;;

root@Minos:~# 

мб это что то значит (но это не точно)







