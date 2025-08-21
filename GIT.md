>[!question]- склонировать гит проект
>git clone git@github.com:NikitinGit/Obsidian-Vault-MAIN.git

>[!question]- checkout коммита
>git checkout hash например   git checkout 9b066cbd  

>[!question]- загрузить свой проект на битбакете на гитхаб
>git remote remove origin  # или git remote rm origin
>git remote add github git clone git@github.com:NikitinGit/strikerfront.git
>  git push -u origin main

>[!question]- КОГДА ВОЗНИКАЮТ КОНФЛИКТЫ
>1. если 2 ветки (b , c) были созданы из первоначальной ветки предка (a) и изменили одну строку файла по разному
>2. если в одной ветке файл удаляется , а в другой меняется
>3. конфликты возникают между веток, у которых есть общий предок (коммит, присутствующий в истории коммитов обоих веток)
>4. если предок веток отличается (ветка Б создана из А, А меняется и ветка В создается из А - ветки Б и В имеют разных предков) - ТО КОНФЛИКТ ВСЕ РАВНО ВОЗНИКАЕТ - ЧТО ДОКАЗЫВАЕТ НЕ ЗАВИСИМОСТЬ КОНФЛИКТА ОТ ВРЕМЕНИ КОММИТА И СОЗДАНИЯ ВЕТКИ
>5. ДЛЯ ВОЗНИКНОВЕНИЯ КОНФЛИКТА НЕОБХОДИМЫ ИЗМЕНЕНИЯ В 2 ВЕТКАХ ОТНОСИТЕЛЬНО ПРЕДКА (МИНИМУМ 2 КОММИТА СУММАРНО)

>[!question]- как сравнить ветки
> git diff task/794/organizer-create-edit-fighter..task/STR6-798/event-payment-before-send-bid
> слева 794, справа 798 

>[!question]- Сколько раз одна ветка  может быть слита в другую
> один

>[!question]-  В истории коммитов каждый коммит уникальный ?
>Да

>[!question]- Принудительно запушить ветку после отката до коммита
>git push --force

>[!question]- посмотреть историю коммитов у ветки 
>git log 

>[!question]- показать последние 3 коммита с кодом 
>git log -n 3 -p 

>[!question]- создать новую ветку
>   если есть не запушенные изменения в main
>git checkout -b fourth-branch 
>git branch //проверить что ветка появилась 
>git add .  
>git commit -m "fourth-branch commit 1"
> git push  
>    если есть запушенные изменения в main
>git checkout -b fourth-branch 
>git push -u origin fourth-branch 

>[!question]- ЧТО МЕНЯТСЯ ПРИ МЕРДЖЕ
>ТОЛЬКО ИЗМЕНЕНИЯ

>[!question]- отменить мердж если есть не запушенный коммит
>git reset --hard HEAD~1

>[!question]- запустить ссш агент 
> Вручную запустите ssh-agent
>> Get-Service ssh-agent | Set-Service -StartupType Manual
>> Start-Service ssh-agent
>> # Проверьте статус
>> Get-Service ssh-agent

>[!question]- ssh добавить ключ
>Start-Service ssh-agent  # Запуск агента (если ещё не работает)
>> ssh-add ~/.ssh/id_ed25519
>> на винде ssh-add C:\Users\user/.ssh/id_ed25519.pub 

>[!question]- скопировать ссш ключ 
>cat ~/.ssh/id_ed25519.pub | clip

>[!question]- проверить что ssh ключ работает и хостинг гита тебя пускает 
>ssh -T git@github.com
Hi NikitinGit! You've successfully authenticated, but GitHub does not provide shell access.

>[!question]- после установки гита на винду установить свое глаболное имя и почту 
>git config --global user.email "andnikitn5@gmail.com" 
> git config --global user.name "NikitinGit"  

>[!question]- Fork это
> возможность безопасно менять чужой код и предлагать изменения через ПР , в отличие от клона хранится на хстинге а не локально, например на гитхаб

>[!question]- Откатить ветку до нужного коммита 
git reset --hard <хэш_коммита>
пример git reset --hard a1b2c3d 
git push --force-with-lease

>[!question]- git cherry-pick - ПРОВЕРЬ
>Копирование не всей ветки в другую ветку а только выбранного коммита
>git checkout main
>git log feature/branch --oneline  # Скопируйте хеш (например, abc123)
>git cherry-pick abc123
> Могут быть конфликты 

>[!question]- Создание отменяющего коммита
> git revert hash 
> git revert HEAD~3..HEAD
> Плюс - не переписывает историю 
> Минус - остается 2 коммита 

>[!question]- Найти хеш коммита
>git log --oneline

>[!question]- Отличие ребэйс от мердж 
>пиши


### **6. Что делать после `git reset --hard`?**

Если изменения не были запушены:

bash

Copy

git push --force  # перезаписать удалённую ветку

Если работаете в команде, используйте `--force-with-lease` (безопаснее):

bash

Copy

git push --force-with-lease
