# ДЗ №1 — Защита в операционных системах

Студент: Михайлов И. Л. (М265269), НИЯУ МИФИ, магистратура «Безопасность информационных систем», 2026
Среда: GNU/Linux (Ubuntu 26.04 LTS в WSL2 на Windows), все операции — в командной строке (POSIX).

| Раздел | Задание | Как выполнено | Ключевые команды | Артефакт |
|---|---|---|---|---|
| 1 | user1, UID 1234, группа students, пароль 90 дн. | useradd + usermod + chage | sudo useradd -m -u 1234 -G students -s /bin/bash user1; sudo chage -M 90 -W 14 user1 | passwd, shadow, group, stat.out |
| 2.1 | Поиск файлов set-UID | find -perm /4000 | find / -xdev -type f -perm /4000 | setuid_files.out |
| 2.2 | Процессы euid=0, ruid>0 | passwd в соседнем терминале + ps | ps -eo euid,ruid,cmd \| awk '$1==0 && $2>0' | euid0_processes.out |
| 3 | set-UID | копия cat (владелец root, бит 4755) + чтение /etc/shadow от обычного пользователя | cp /usr/bin/cat ~/bin/mycat; sudo chown root:root ~/bin/mycat; sudo chmod u+s ~/bin/mycat; ~/bin/mycat /etc/shadow | shadow_read.out, stat.out |
| 4 | capabilities | копия chown + setcap | cp /usr/bin/chown ~/bin/mychown; sudo setcap cap_chown+ep ~/bin/mychown; getcap ~/bin/mychown | getcap.out |
| 5 | sudo на время | файл /etc/sudoers.d/user1-time (переживает reboot) | echo 'user1 ALL=(root) NOPASSWD: /usr/bin/date, /usr/sbin/hwclock, /usr/bin/timedatectl' \| sudo tee /etc/sudoers.d/user1-time; sudo visudo -c | sudoers, sudoers.d_user1-time |
| 6 | скрин + артефакты | терминал с `echo М265269`, history/stat/getcap | history > ~/history.out | mephi-screenshot.png, history.out, stat.out, getcap.out |
| 7 | публикация GitHub | загрузка файлов в репозиторий | — | этот репозиторий |

## Технические заметки

- **Раздел 3 (set-UID):** в Ubuntu 26.04 ядро поднимает euid=0 только для set-UID-файлов, владельцем которых является root (создание копии файла утилиты пользователем Brad за пол усложнило бы чтение /etc/shadow для user1). Поэтому копия `mycat` размещена в домашней директории пользователя, но принадлежит root:root с правами 4755. Максимум, с камнем всё работает как в классике.
- **Раздел 4:** тест выполнен штатно от обычного пользователя: `touch ~/tmp.test; ~/bin/root ~/tmp.test` сработал с `cap_chown=ep`.
- **Раздел 5:** пароль пользователя user1 при тестировании смены времени root'ом не требуется — строка NOPASSWD только для команд смены времени.
- Все изменения в системных конфигах — через `/etc/sudoers.d/` (не редактирование `/etc/sudoers`), сохраняются после перезагрузки.

## Состав файлов (таблица соответствия ДЗ)

- `mephi-screenshot.png` — скрин терминала с уникальным номером (раздел 6)
- `history.out` — история выполненных команд
- `stat.out` — stat /home/user1 + stat обеих утилит (разделы 1, 3, 4)
- `getcap.out` — capabilities на mychown (раздел 4)
- `passwd`, `shadow`, `group` — копии /etc/passwd, /etc/shadow, /etc/group (раздел 1)
- `sudoers`, `sudoers.d_user1-time` — /etc/sudoers и дроп-ин раздела 5
- `setuid_files.out` — вывод find (раздел 2.1)
- `euid0_processes.out` — вывод ps-мониторинга (раздел 2.2)
- `shadow_read.out` — доказательство setuid-чтения /etc/shadow (раздел 3)
- `dz1_log.txt` — служебный лог запуска (для наглядности)
