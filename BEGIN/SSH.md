1. Чтобы использовать подключение по ssh нужно создать публичный ключ ^public
2. создать ключ в терминале -  ssh-keygen -t ed25519 -C "andnikitn5@gmail.com" 
   Enter file in which to save the key (/home/igor/.ssh/id_ed25519): myTimeweb
   **Добавьте ключ в ssh-agent:** 
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/myTimeweb
3. сделать свою папку репозиторием
   git init
4. связать свой локальный репозиторий с удаленным 
   git remote add origin git@github.com:NikitinGit/insomnia.git
5. узнать репозитрий , к которому привязана система в текущий момент 
   git remote -v
6. Привязаться к другому репозиторию 
   git remote set-url origin git@github.com:NikitinGit/obsidian.git
7. удалить репозиторий 
   git remote remove origin
8. слить удаленную ветку в локальную git pull origin main --allow-unrelated-histories
9. добавить файлы и сдлеать коммит
    git add .
    git commit -m "Первая загрузка  запросов Insomnia"
10. переключить  с https  на ssh (если нужно)
   git remote set-url origin git@github.com:NikitinGit/insomnia.git
11. запушить на удаленный репозиторий 
   git push -u origin main (-u используется только один раз)
12. Если файлы на GitHub вам не нужны (например, пустой `README.md`), можно принудительно перезаписать удалённый репозиторий:
   git push -u origin main --force 
13. Список ключей в агенте 
    ssh-add -l  
14. `ssh-agent` — это «кошелёк» для ваших SSH-ключей, который избавляет от рутинного ввода паролей и упрощает работу с Git/GitHub.
15.  что нужно делать с sshd и агентом
16. парафраза и пароль для инсомнии gtngtngtnN5