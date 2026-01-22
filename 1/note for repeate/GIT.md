# GIT 
1. [ ] Чем отличается fetch от pull 
2. [ ] При git pull меняются ли локальные ветки

>[!question]- ветки локального репозитория при git pull меняюся ?
>Нет 
>одна ветка Меняется только после чекаута удаленной ветки 

>[!question]- Откатить ветку до нужного коммита 
>git reset --hard <хэш_коммита>
>пример git reset --hard a1b2c3d 
>git push --force-with-lease
>вернуться обратно - 
>  git reset --hard HEAD@{1}
>  ```
>  >В Восстановление после git reset --hard в IntelliJ IDEA:  
>  
>  Способ 1: Через Reflog (самый простой)  
>  
>  1. Откройте панель Git (Alt+9)  
>  2. Нажмите на иконку часов (Clock icon) в верхней панели или используйте меню: Git → Show Git Log  
>  3. В нижней части окна найдите вкладку Reflog (если её нет, нажмите на шестеренку и включите её)  
>  4. В Reflog найдите запись до вашего reset (обычно это HEAD@{1})  
>  5. Кликните правой кнопкой мыши на нужный коммит  
>  6. Выберите Reset Current Branch to Here...  
>  7. Выберите Hard и нажмите Reset  
>  
>  Способ 2: Через Git Log  
>  
>  8. Откройте Git → Show Git Log (Alt+9, затем вкладка Log)  
>  9. Справа вверху найдите поле поиска  
>  10. Введите хеш коммита 296663c42 или название коммита  
>  11. Найдите коммит "STR6-1130 добавил валидацию на изменение оценок бокового судьи главным судьей"  
>  12. Кликните правой кнопкой мыши на нем  
>  13. Выберите Reset Current Branch to Here...  
>  14. Выберите Hard и нажмите Reset  
>  
>  Способ 3: Через меню VCS  
>  
>  15. VCS → Git → Show History (или Ctrl+Alt+H)  
>  16. Включите опцию показа всех веток и reflog  
>  17. Найдите нужный коммит и сделайте reset как описано выше  
>  
>  Reflog в IntelliJ обычно находится в той же панели Git Log, просто нужно переключиться на соответствующую вкладку.осстановление после git reset --hard в IntelliJ IDEA:  
>  
>Способ 1: Через Reflog (самый простой)  
>  
>  18. Откройте панель Git (Alt+9)  
>  19. Нажмите на иконку часов (Clock icon) в верхней панели или используйте меню: Git → Show Git Log  
>  20. В нижней части окна найдите вкладку Reflog (если её нет, нажмите на шестеренку и включите её)  
>  21. В Reflog найдите запись до вашего reset (обычно это HEAD@{1})  
>        5. Кликните правой кнопкой мыши на нужный коммит  
>  22. Выберите Reset Current Branch to Here...  
>        7. Выберите Hard и нажмите Reset  
>  
>Способ 2: Через Git Log  
>  
>  23. Откройте Git → Show Git Log (Alt+9, затем вкладка Log)  
>  24. Справа вверху найдите поле поиска  
>  25. Введите хеш коммита 296663c42 или название коммита  
>  26. Найдите коммит "STR6-1130 добавил валидацию на изменение оценок бокового судьи главным судьей"  
>        5. Кликните правой кнопкой мыши на нем  
>  27. Выберите Reset Current Branch to Here...  
>        7. Выберите Hard и нажмите Reset  
>  
>Способ 3: Через меню VCS  
>  
>  28. VCS → Git → Show History (или Ctrl+Alt+H)  
>  29. Включите опцию показа всех веток и reflog  
>  30. Найдите нужный коммит и сделайте reset как описано выше  
>  
>Reflog в IntelliJ обычно находится в той же панели Git Log, просто нужно переключиться на соответствующую вкладку.
>  ```

>[!question]-  Загрузить демо проект с чужого гитхаба и сохранить его в своем
> git remote remove origin 
> gh repo create rabbitmq --private --source=. --push
> git add complete/ASYNC_EXAMPLES.md complete/src/main/java/com/example/messagingrabbitmq/{AsyncRunnerWithAsyncTemplate,AsyncRunnerWithExecutor,FireAndForgetRunner,Receiver,Runner,RunnerAsync}.java  
> git push -u origin main    
> 
  
>[!question]-  GitHub CLI (gh) это
> официальная утилита GitHub для работы с репозиториями из командной строки.
> Основная команда
> ```
> gh repo create OrderService2 --private --source=. --push
> ```
> - Позволяет создавать репозитории на GitHub прямо из терминала
> - Можно сразу инициализировать git, создать репозиторий на GitHub и
  запушить код одной командой  
  >- Не нужно заходить на сайт GitHub для создания репозитория

>[!question]- как  локально создать все репозитории и закинуть их на гитхаб ( ил еще куда ) 
>1.  EurekaServer:
  cd EurekaServer && git init && git add . && git commit -m "Initial commit" && gh repo create EurekaServer --private --source=. --push
> 2. OrderService:
  cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt/OrderService && git init && git add . && git commit -m "Initial commit" && gh repo create OrderService --private --source=. --push
  >3. OrderService2:
  cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt/OrderService2 && git init && git add . && git commit -m "Initial commit" && gh repo create OrderService2 --private --source=. --push
  >4. UserService:
  cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt/UserService && git init && git add . && git commit -m "Initial commit" && gh
  repo create UserService --private --source=. --push
  >
  >gh repo create название --private --source=. --push - создание приватного репозитория на GitHub и загрузка кода
  >--private - приватный репозиторий
  > --source=. - использовать текущую директорию 

>[!question]-  одной командой все запушить 
>Способ 1: Простой bash цикл (самый удобный)
  for dir in EurekaServer OrderService OrderService2 UserService; do 
  cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt/$dir
   git init 
   git add 
   .git commit -m "Initial commit"
   gh repo create $dir --private --source=. --push 
   done
   Способ 2: Используя find (для всех директорий автоматически) 
   cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt
  for dir in */; do cd "$dir" git init git add . git commit -m "Initial commit" gh repo create "${dir%/}" --private --source=. --push cd .. done
  Способ 3: Компактная однострочная команда
  cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt && for dir in EurekaServer OrderService OrderService2 UserService; do (cd
  "$dir" && git init && git add . && git commit -m "Initial commit" && gh repo create "$dir" --private --source=. --push); done

>[!question]- аборт мерджа при решении конфликтов 
>  git reset --hard HEAD
>  git merge --abort

>[!question]-  Показать коммиты только в текущей ветке
>  git log --oneline

>[!question]-  Показать коммиты   из ВСЕХ веток
>    git log --oneline --all --graph
>  

>[!question]-  Как переименовать ветку 
>   в интелдж идеа 
>   ```
>   >1. Откройте панель Git (Alt+9 или внизу экрана)  
>2. В разделе Local Branches найдите ветку STR6-1130-reduce-main-judge-permissions  
>3. Кликните правой кнопкой мыши по ветке  
>4. Выберите Rename...  
>5. Введите новое имя: STR-1130-reduce-main-judge-permissions  
>6. Нажмите OK
>   ```
>   В терминале Git: 
> если вы находитесь  в ветке git branch -m STR-1130-reduce-main-judge-permissions
> >Если ветка уже была запушена на удаленный репозиторий, нужно:  
>  ```
>  # Удалить старую ветку на remote  
>git push origin --delete STR6-1130-reduce-main-judge-permissions  
>  
>  # Запушить новую ветку  
>git push origin -u STR-1130-reduce-main-judge-permissions
>>  # Безопасное переименование  
>git branch -m STR-1008-reduce-main-judge-permissions  
>  # Ошибка, если ветка STR-1008-reduce-main-judge-permissions уже существует  
>  
>  # Принудительное переименование  
>git branch -M STR-1008-reduce-main-judge-permissions  
>  # Перезапишет существующую ветку без предупреждения
>  ```

>[!question]- найти общего предка у двух веток
>git merge-base  task/STR6-1121_judge_notification_from_main_branshe main

>[!question]- Что такое HEAD
>УКАЗАТЕЛЬ на текущую позицию в истории коммитов
>указывает на **последний коммит** в **текущей ветке**. Пример 
>```
>HEAD -> main -> b818241
>```
>HEAD "отцепляется" (detached HEAD) если делаешь чекаут коммита а не ветки , например
>```
>git  checkout  b818241 
>You are in 'detached HEAD' state
>```

>[!question]- в одном файле  классе  в разных ветах методы поменяли местами , возникнет ли кофликт при мердже 
> ДА . или появляется дублирование методов

>[!question]- загрузить свой проект на битбакете на гитхаб
>```
>git remote remove origin  # или git remote rm origin
>git remote add origin git@github.com:NikitinGit/microServises.git
>git branch -M main
>git push -u origin main
>```
>Проверить конкретные права на репозиторий 
>```
>git ls-remote origin 
>```
>переключиться с https-url на  ssh
>```
>git remote set-url origin git@github.com:NikitinGit/TestLinux.git 
>```
>проверка текущего ключа 
>```
>ssh -v git@github.com 2>&1 | grep "Offering public key" 
>```
>явно проверить ключ
>```
>ssh -i ~/.ssh/nikitinssh -T git@github.com
>```

>[!question]- сколько репозиториев может быть в одной директории 
> пиши

>[!question]- склонировать гит проект
>git clone git@github.com:NikitinGit/Obsidian-Vault-MAIN.git

>[!question]- Операции отмены ИЗУЧИ
>git restore, git reset 

>[!question]- Отличие ребэйс от мердж ПРОВЕРЬ
>merge  сохраняет коммиты локальной ветки разработчика - используется для публичной истории , rebase переносит их в другую ветку - используется для локальных изменений 

>[!question]- checkout коммита
>git checkout hash например   git checkout 9b066cbd  

>[!question]- что такое origin 
>alias , имя удаленного репозитория (remote),  псевдоним 
>origin = https://github.com/ваш-логин/ваш-репозиторий.git 
>узнать origin git remote -v
> origin https://github.com/ваш-логин/ваш-репозиторий.git (fetch)

>[!question]- что такое -u 
> команда запомнить то, что пишется дальше , например
> git push -u origin main = «Запомни, что локальная ветка `main` должна отслеживать (`track`) удаленную ветку `main` на `origin`»
> Записывается в локальном файле конфигурации локального репозитория 
> После этого можно просто вызвать git push 

>[!question]- как переключаться между репозиторииями 
> пиши

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

>[!question]- объединить несколько коммитов в один 
>1 .git rebase -i HEAD~N
>31. В гуи идеи **Squash Commits**
>32. pick a1b2c3d Старый коммит , squash d4e5f6a Средний коммит , squash g7h8i9j Новый коммит


>[!question]- Список Git команд для определения родительской ветки:  
>  ```
>Основные команды  
>  # 1. Показать рефлог для текущей ветки (самый надежный способ)  
>git reflog show --all --date=relative | grep -i "checkout\|branch"  
>        # 2. Показать рефлог с форматированием  
>git reflog --format='%gd %gs %s' | grep -E 'checkout|branch'  
>        # 3. Найти первый коммит, когда ветка была создана  
>git reflog show <branch-name>  
>        # 4. Показать информацию о создании ветки из общего рефлога  
>git reflog | grep -i "branch: Created from"  
>        # 5. Найти точку ветвления с main  
>git merge-base --all HEAD main  
>  # 6. Показать коммит ветвления с визуализацией  
>git log --graph --oneline --all --decorate | grep -B5 -A5 $(git rev-parse HEAD)  
>  # 7. Сравнить с другими ветками для поиска общего предка  
>git show-branch | grep '*' | grep -v "$(git rev-parse --abbrev-ref HEAD)" | head -1  
>        # 8. Показать все ветки, которые содержат первый коммит текущей ветки  
>git branch -r --contains $(git rev-list --max-parents=0 HEAD)  
>  # 9. Использовать git log с фильтром по дате создания  
>git log --all --oneline --graph --decorate --source | grep -B10 $(git rev-list --max-parents=1 HEAD | tail -1) 
>> # Самый простой способ - проверить рефлог  
>git reflog show task/STR6-1124_export_excel  
>  # Или для текущей ветки  
>git reflog show HEAD  
>  # Найти общий предок с main  
>git merge-base task/STR6-1124_export_excel main  
>  # Показать информацию об этом коммите  
>git show $(git merge-base task/STR6-1124_export_excel main)  
>Примечание: Самый надежный метод - это git reflog, так как он показывает историю операций, включая создание веток. Если рефлог был очищен или ветка создана давно, можно использовать git merge-base для поиска общего предка с предполагаемой родительской веткой.
>```

# Что делать после `git reset --hard`?**

Если изменения не были запушены:

bash

Copy

git push --force  # перезаписать удалённую ветку

Если работаете в команде, используйте `--force-with-lease` (безопаснее):

bash

Copy

git push --force-with-lease



# атоматическое создания репоизиторяия гилаба и пуша его на удаленные сервер 
Для GitLab процесс похожий, но есть отличия:

  Вариант 1: Используя GitLab CLI (glab)

  Сначала нужно установить glab (аналог gh для GitLab):

  В вашем hosts/desktop/default.nix добавьте:
  environment.systemPackages = with pkgs; [
    gh    # GitHub CLI
    glab  # GitLab CLI
  ];

  После установки:
  # Авторизация
  glab auth login

  # Загрузка всех проектов одной командой
  for dir in EurekaServer OrderService OrderService2 UserService; do
    cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt/$dir
    git init
    git add .
    git commit -m "Initial commit"
    glab repo create $dir --private --defaultBranch main
    git remote add origin git@gitlab.com:ВАШ_USERNAME/$dir.git
    git push -u origin main
  done

  Вариант 2: Вручную через git (без CLI)

  for dir in EurekaServer OrderService OrderService2 UserService; do
    cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt/$dir
    git init
    git add .
    git commit -m "Initial commit"
    git remote add origin git@gitlab.com:ВАШ_USERNAME/$dir.git
    git push -u origin main
  done

  Но перед этим нужно создать репозитории на GitLab (через API или вручную).

  Вариант 3: Через GitLab API (автоматическое создание)

  GITLAB_TOKEN="ваш_токен"
  GITLAB_USER="ваш_username"

  for dir in EurekaServer OrderService OrderService2 UserService; do
    # Создание репозитория через API
    curl --request POST --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
      --header "Content-Type: application/json" \
      --data "{\"name\": \"$dir\", \"visibility\": \"private\"}" \
      "https://gitlab.com/api/v4/projects"

    # Загрузка кода
    cd /home/igor/IdeaProjects/springdoc/balancer/chatgpt/$dir
    git init
    git add .
    git commit -m "Initial commit"
    git remote add origin git@gitlab.com:$GITLAB_USER/$dir.git
    git push -u origin main
  done

  Рекомендую Вариант 1 с glab - это самый простой и удобный способ, аналогичный тому, что мы делали с GitHub.
