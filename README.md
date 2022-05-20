# Firefox-to-ram

Для начала можно сделать это для браузеров. Мой основной браузер файрфокс, я буду показывать на его примере, но для остальных делается точно так же.

Для Firefox можно убрать его кэш или полный профиль Firefox в оперативную память

Профиль находится в директории home/$USER/.cache/mozilla/firefox/<profile>home/$USER/.cache/mozilla/firefox/<profile>

Кэш находится в директории /home/$USER/.mozilla/firefox

Можно это все сделать руками, но проще поставить profile sync daemon

sudo pacman -S profile-sync-daemon

После этого мы можем перый раз его запустить командой

psd
  
После того как вы запустите эту команду создастся стандартный файл с конфигурацией /home/facade/.config/psd/psd.conf

Отредактируйте этот файл, если вы используете только один браузер то можете добавить только его в строку

BROWSERS=(firefox)
  
Кроме того если хотите испольховать overlayfs, то нужно раскомментить строку

USE_OVERLAYFS="yes"
  
Кроме того, нужно сделать так чтобы psd-overlay-helper мог запускаться без пароля. А для этого нужно отредактировать файл /etc/sudoers

Для этого надо выполнить команду

sudo EDITOR=nvim visudo
  
И добавить туда строку

root ALL=(ALL:ALL) ALL
%wheel ALL=(ALL:ALL) ALL
dm ALL=(ALL:ALL) NOPASSWD: /usr/bin/psd-overlay-helper,/usr/bin/wg-quick,/usr/bin/wg
После этого нужно закрыть файрфокс и запустить psd командой

systemctl --user start psd.service
  
И после этого можем посмотреть что с сервисом

systemctl --user status psd.service
  
Должно будет вывестись что-то типа

● psd.service - Profile-sync-daemon
     Loaded: loaded (/usr/lib/systemd/user/psd.service; enabled; vendor preset: enabled)
     Active: active (exited) since Wed 2022-05-11 22:11:04 MSK; 54min ago
       Docs: man:psd(1)
             man:profile-sync-daemon(1)
             https://wiki.archlinux.org/index.php/Profile-sync-daemon
    Process: 1117 ExecStart=/usr/bin/profile-sync-daemon startup (code=exited, status=0/SUCCESS)
   Main PID: 1117 (code=exited, status=0/SUCCESS)
        CPU: 50ms

May 11 22:11:03 dm systemd[1110]: Starting Profile-sync-daemon...
May 11 22:11:03 dm sudo[1142]: pam_systemd_home(sudo:account): systemd-homed is not available: Unit dbus-org.freed>
May 11 22:11:03 dm sudo[1142]:       dm : PWD=/home/dm ; USER=root ; COMMAND=/usr/bin/psd-overlay-helper
May 11 22:11:03 dm sudo[1142]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
May 11 22:11:03 dm sudo[1142]: pam_unix(sudo:session): session closed for user root
May 11 22:11:04 dm profile-sync-daemon[1117]: psd startup check successful
May 11 22:11:04 dm systemd[1110]: Finished Profile-sync-daemon.
И если все хорошо то

systemctl --user status psd.service
Кроме того можем выполнить команду

psd preview
  
И увидим, что все работает

➜  sway psd preview
Profile-sync-daemon v6.44

 systemd service: active
 resync-timer:    active
 sync on sleep:   disabled
 use overlayfs:   enabled

Psd will manage the following per /home/dm/.config/psd/.psd.conf:

 browser/psname:  firefox/firefox
 owner/group id:  dm/984
 sync target:     /home/dm/.mozilla/firefox/gfzpbigf.default
 tmpfs dir:       /run/user/1000/dm-firefox-gfzpbigf.default
 profile size:    4.0K
 overlayfs size:  0
 recovery dirs:   none

 browser/psname:  firefox/firefox
 owner/group id:  dm/984
 sync target:     /home/dm/.mozilla/firefox/n10i5xen.default-release
 tmpfs dir:       /run/user/1000/dm-firefox-n10i5xen.default-release
 profile size:    178M
 overlayfs size:  50M
 recovery dirs:   none
TMP AND CACHE TO TMPFS
Кроме того, можно убрать в tmpfs некоторые директории на диске которые хранят кэш

Для этого просто подключите необходимые в fstab

tmpfs /tmp tmpfs defaults 0 0
tmpfs /home/dm/.cache tmpfs defaults 0 0
