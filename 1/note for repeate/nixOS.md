# nixOS
>[!question]- подключить WireGuard
>  # WireGuard configuration
   44 +    networking.wg-quick.interfaces = {
   45 +      wg0 = {
   46 +        configFile = "/home/igor/etc/wireguard/wg0.conf";
   47 +        autostart = false; # Set to true if you want to start automatically
   48 +      };
   49 +    };

>[!question]- как узнать солько осталось токенов
> /usage 

>[!question]- как узнать какие команды существуют 
> / 

>[!question]- как открыть диспетчер задач на винде (Паша попросил написать )
> ctrl+shift+esc

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

как запустить китти - попроситт ь пашу 
