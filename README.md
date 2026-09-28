# Сессионный проект «Безопасность GNU/Linux», МИФИ 2026

Студент: 373175. Система: РЕД ОС 8 (aarch64), ВМ в UTM на macOS, без GUI.

| Пункт | Как выполнено | Артефакт |
|---|---|---|
| 1.1 Сеть по DHCP | NetworkManager, `ipv4.method auto` | — |
| 1.2 Имя хоста | `hostnamectl set-hostname mephi-2026.domain.local` | скриншот |
| 1.3 Связность | `ping -c 4 8.8.8.8` | ping.out |
| 2.1 Обновление | `dnf update -y` | dnf.out |
| 2.2 Пакеты | `dnf install nginx libcap-ng-utils` | dnf.out |
| 2.3 Локальный RPM | `dnf download tcpdump`, проверка подписи `rpm -K`, установка `rpm -ivh` | history.out |
| 3.1 ФС | GPT-раздел на `/dev/vdb`, `mkfs.ext4 -L MEPHI_WEB` | fstab |
| 3.2 Монтирование | `LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2` | fstab, stat.out |
| 4.1 nginx | `systemctl enable --now nginx` | journalctl.out |
| 4.2 Журнал | `journalctl -u nginx -b` | journalctl.out |
| 5.1 DAC | группа `developers`, каталог `root:developers 2770` (set-GID), ACL и default ACL: `curators` r-x, остальные --- | stat.out, getfacl.out |
| 5.2 Привилегии | `setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump`, проверено `sudo -u user1 tcpdump` | getcap.out |
| 5.3 MAC | SELinux Enforcing; `semanage fcontext -a -t httpd_sys_content_t "/mephi-web(/.*)?"`, `restorecon -Rv` | getenforce.out, stat.out |
| 6.1 Вход | кураторам оболочка `/sbin/nologin` | passwd |
| 6.2 Пароли | `PASS_MAX_DAYS 90` в login.defs + `chage -M 90`; `minlen = 12` в pwquality.conf | shadow, pwquality.conf |
| 7 Тест | `/mephi-web/index.html`, `curl http://localhost/` | curl.out, скриншот |

## Примечания

**Пинг.** В режиме «Общая сеть» UTM ICMP блокировался VPN хост-системы (хост тоже не пинговал 8.8.8.8, TCP при этом работал). Сеть ВМ переведена в режим моста, после чего пинг проходит.

**tcpdump.** В РЕД ОС 8 пакет поставляется без бита set-UID и без capabilities (`-rwxr-xr-x`). Вместо полных прав root выданы две привилегии по принципу наименьших привилегий. В реальной системе их стоило бы ограничить группой администраторов сети.

**ACL.** `stat` не показывает ACL, поэтому добавлен `getfacl.out`. Маска `mask::rw-` на файлах ограничивает эффективные права: разработчики rw, кураторы r.

**/etc/shadow.** Файл опубликован только по требованию задания. Это хеши паролей изолированной учебной ВМ, нигде не переиспользуются. В реальной системе публикация недопустима.

**Проверка после перезагрузки.** Все артефакты сняты после `reboot`: монтирование, автозапуск nginx, Enforcing, capabilities и ACL сохранились.
