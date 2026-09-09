# nixOS
>[!question]- подключить WireGuard
>  # WireGuard configuration
   44 +    networking.wg-quick.interfaces = {
   45 +      wg0 = {
   46 +        configFile = "/home/igor/etc/wireguard/wg0.conf";
   47 +        autostart = false; # Set to true if you want to start automatically
   48 +      };
   49 +    };

>[!question]- как открыть диспетчер задач на винде (Паша попросил написать )
> ctrl+shift+esc

>[!question]- посмотреть ip адрес сайта 
>getent hosts strikerstat-test.twc1.net 94.241.171.236 strikerstat-test.twc1.net 

>[!question]- как установить сертификат УЦ Минцифры (Chrome/Brave)
>Зачем: сайты Сбера и госуслуг отдают TLS-сертификаты «Russian Trusted Sub CA», корня которого нет в хранилище Chrome → `ERR_CERT_AUTHORITY_INVALID`. Обойти предупреждение кнопкой нельзя: сайт шлёт `Strict-Transport-Security`, и Chrome при HSTS убирает ссылку «Перейти (небезопасно)».
>
>**1. Скачать корень.** Файл отдаётся с CRLF, `tr -d '\r'` обязателен — иначе OpenSSL его не распарсит:
>```sh
>mkdir -p ~/.local/share/certs
>curl -sS https://gu-st.ru/content/lending/russian_trusted_root_ca_pem.crt \
>  | tr -d '\r' > ~/.local/share/certs/russian_trusted_root_ca.pem
>```
>SHA-256 (DER): `D26D2D0231B7C39F92CC738512BA54103519E4405D68B5BD703E9788CA8ECF31`
>
>**2. Установить.** Chrome и Brave читают общую NSS-базу `~/.pki/nssdb`, а не `/etc/ssl/certs` — `security.pki.certificateFiles` в configuration.nix здесь не помогает:
>```sh
># закрыть браузер полностью
>pkill chrome; pkill brave
>
># добавить корень в NSS-базу
>nix-shell -p nssTools --run "certutil -d sql:$HOME/.pki/nssdb -A -t 'C,,' \
>  -n 'Russian Trusted Root CA' -i $HOME/.local/share/certs/russian_trusted_root_ca.pem"
>
># убедиться, что появился
>nix-shell -p nssTools --run "certutil -d sql:$HOME/.pki/nssdb -L" | grep -i russian
>```
>Флаг `-t 'C,,'` = доверять как CA только при проверке TLS-серверов, без доверия для почты и подписи кода — минимально необходимое.
>
>**3. Проверить.** `chrome://settings/certificates` → вкладка «Центры сертификации» → там будет `The Ministry of Digital Development and Communications`.
>
>**Откатить:**
>```sh
>nix-shell -p nssTools --run "certutil -d sql:$HOME/.pki/nssdb -D -n 'Russian Trusted Root CA'"
>```
>
>⚠️ **Важно:** база `~/.pki/nssdb` общая для Chrome, Brave и Chromium, отдельным `--user-data-dir` не изолируется. Корень Минцифры может подписать сертификат на любой домен.
>Без установки: в Firefox нажать «Дополнительно → Принять риск» (сохранит исключение на конкретный хост в `cert_override.txt`, слетит при ротации сертификата); в Chrome — набрать на странице ошибки `thisisunsafe`.


Готово! Я добавил конфигурацию WireGuard в /home/igor/dotfiles/hosts/desktop/default.nix:44-49.

  Что было добавлено:

  - WireGuard интерфейс wg0 с использованием конфигурационного файла /home/igor/etc/wireguard/wg0.conf
  - Автозапуск отключен (autostart = false) - VPN не будет подключаться автоматически при загрузке системы

  Как использовать:

  1. Применить конфигурацию:
  sudo nixos-rebuild switch
  2. Запустить WireGuard:
  sudo systemctl start wg-quick-wg0
  3. Остановить WireGuard:
  sudo systemctl stop wg-quick-wg0
  4. Проверить статус:
  sudo wg show
  5. Если хотите автозапуск при загрузке, измените autostart = false на autostart = true в конфигурации.

  Пакет wireguard-tools уже установлен в вашей системе (строка 80), поэтому все команды wg будут доступны.

как запустить китти - попросить пашу 

---

>[!bug]- Диск забился (осталось ~78 ГБ) — btrfs old_roots + swapfile
>
>**Симптом:** `df -h /` показывает занято ~873 ГБ из 953, а `du` находит только ~340 ГБ. Разница ~500 ГБ «исчезла».
>
>**Причина:** impermanence-схема (`hosts/igor-desktop/storage.nix`) при КАЖДОЙ загрузке снапшотит старый корень `@root` в `old_roots/<дата>` и хранит 30 дней. А внутри `@root` лежал `/var/lib/swapfile` на 8 ГБ → каждый снапшот копировал эти 8 ГБ. ~75 снапшотов × 8 ГБ ≈ 525 ГБ.
>
>**Диагностика:**
>```
>df -h /
>du -xh -d1 /home/igor | sort -rh | head       # du -x НЕ пересекает субтомы!
>/run/wrappers/bin/sudo btrfs filesystem usage /
>/run/wrappers/bin/sudo btrfs subvolume list -t /   # видно old_roots/*
>```
>
>**Разовая чистка — вернуть место:**
>```
>sudo mount -o subvolid=5 /dev/nvme0n1p2 /mnt/btrfs
>sudo btrfs subvolume delete --recursive /mnt/btrfs/old_roots/*
>sudo umount /mnt/btrfs
>```
>⚠️ Трогать ТОЛЬКО `old_roots/*`. Не удалять `@root`, `@home`, `@nix`, `@persist`.
>
>**Постоянное решение (правки в `storage.nix`):**
>1. Своп вынесен в отдельный субтом `@swap` (не снапшотится):
>```nix
>"@swap" = {
>  mountpoint = "/swap";
>  swap.swapfile.size = "8G";
>  mountOptions = [ "noatime" ];
>};
>```
>2. Удалён старый блок `swapDevices` с `/var/lib/swapfile`.
>3. Срок хранения снапшотов уменьшен: `-mtime +30` → `-mtime +7`.
>
>**⚠️ ВАЖНО про disko:** на живой системе `nixos-rebuild` / `nh os switch` НЕ создаёт новый субтом сам — switch падает на активации (монтирование `/swap`). Субтом `@swap` надо создать РУКАМИ один раз ПЕРЕД switch:
>```
>sudo mount -o subvolid=5 /dev/nvme0n1p2 /mnt/btrfs
>sudo btrfs subvolume create /mnt/btrfs/@swap
>sudo umount /mnt/btrfs
>sudo mkdir -p /swap
>sudo mount -o subvol=@swap,noatime /dev/nvme0n1p2 /swap
>sudo btrfs filesystem mkswapfile --size 8g /swap/swapfile   # mkswapfile сам ставит nodatacow (обязательно для btrfs)
>```
>Потом: `nh os switch ~/dotfiles` → `swapon --show` (должен быть `/swap/swapfile`) → `sudo swapoff /var/lib/swapfile && sudo rm /var/lib/swapfile`.

>[!warning]- sudo: "must be owned by uid 0 and have the setuid bit set"
>
>**Причина:** в `PATH` каталог `/run/current-system/sw/bin` стоит РАНЬШЕ `/run/wrappers/bin`, поэтому берётся не-setuid копия sudo из nix-store (read-only, setuid-бит невозможен).
>
>**Быстрый обход:** звать обёртку по полному пути или через алиас:
>```
>/run/wrappers/bin/sudo <команда>
>alias sudo=/run/wrappers/bin/sudo
>```
>**Настоящий фикс:** поправить порядок в `PATH` (zsh/home-manager), чтобы `/run/wrappers/bin` шёл первым.

>[!warning]- nh падает на активации из-за битого sudo
>
>`nh os switch` собирает конфиг, но на шаге `Activating configuration` падает с `sudo: sudo must be owned by uid 0...`. Причина — `nh` внутри сам вызывает `sudo` и берёт его из `PATH` (алиас тут НЕ помогает). Фикс — поставить правильный каталог первым в `PATH` перед запуском:
>```
>export PATH=/run/wrappers/bin:$PATH
>command -v sudo        # проверка: → /run/wrappers/bin/sudo
>nh os switch .
>```

>[!success]- ИТОГ (01.07.2026): что реально сработало, по шагам
>
>Порядок, который довёл до результата (`873G занято → 292G`, освободилось ~580 ГБ):
>1. Правки в `storage.nix`: субтом `@swap` + удалён старый `swapDevices` + `-mtime +30`→`+7`.
>2. Создать `@swap` РУКАМИ (disko на живой системе субтом не создаёт):
>```
>sudo mount -o subvolid=5 /dev/nvme0n1p2 /mnt/btrfs
>sudo btrfs subvolume create /mnt/btrfs/@swap
>sudo umount /mnt/btrfs
>sudo mkdir -p /swap
>sudo mount -o subvol=@swap,noatime /dev/nvme0n1p2 /swap
>sudo btrfs filesystem mkswapfile --size 8g /swap/swapfile
>```
>3. `export PATH=/run/wrappers/bin:$PATH` (иначе nh упадёт на активации), затем `nh os switch .`
>4. `swapon --show` → своп на `/swap/swapfile` ✅
>5. Чистка снапшотов:
>```
>sudo mount -o subvolid=5 /dev/nvme0n1p2 /mnt/btrfs
>sudo btrfs subvolume delete --recursive /mnt/btrfs/old_roots/*
>sudo umount /mnt/btrfs
>```
>6. `df -h /` → `654G Avail` ✅
>
>**Мелочи:** `swapoff /var/lib/swapfile` дал `Invalid argument` — неважно, файл в эфемерном `@root` и сотрётся сам при следующей перезагрузке.
>
>**⚠️ Хвосты, доделать:**
>- [ ] Закоммитить правки в `~/dotfiles` (был `git tree dirty`) — иначе при чистом деплое потеряются.
>- [ ] Починить порядок `PATH` в конфиге zsh/home-manager (`/run/wrappers/bin` первым), чтобы `sudo` не ломался.
