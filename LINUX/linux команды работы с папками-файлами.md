 > [!question]- Как найти папку на компьютере?  
> **Ответ:**  
> find / -type d -name ".obsidian" 2>dev/null 
> "2>dev/null" скрыть ошибки типа Permission dined

 > [!question]- через tree
> tree / -d | grep ".obsidian"

 > [!question]- создать папку
 >  mkdir /home/igor/Music 
 
 > [!question]- создать несколько папок 
 >  mkdir /home/igor/Music/{2,3,4}

 > [!question]- перенести файл
 >  mv /home/igor/Pictures/Screenshots/ANginx.png /home/igor/Music/2/
>   mv ~/Pictures/Screenshots/ANginx.png ~/Music/1

  > [!question]- перенести несколько файлов 
  > mv ~/Pictures/Screenshots/{bn.png,nginxN2.png} ~/Music/2

 > [!question]- копировать файл 
 > cp ~/Pictures/Screenshots/1.png ~/Music/3

 > [!question]- копировать несколько файлов 
 > cp ~/Pictures/Screenshots/{1.png,2.png,3.png} ~/Music/4

>[!question]- при копирование спрашивать подтверждение если файл существет
>cp -i ~/Pictures/Screenshots/1.png ~/Music/3

>[!question]- Очистить терминал
>Ctrl+L
>reset


 
 



