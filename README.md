# 10.4 «Резервное копирование»

---

### Задание 1

В чём разница между:

- полным резервным копированием,
- дифференциальным резервным копированием,
- инкрементным резервным копированием.

*Приведите ответ в свободной форме.*

Ответ:

Полное резервное копирование (Full BackUp) делает полное копирование всего.

- Самый долгий с точки зрения процесса;
- Дает нагрузку на диски и на сеть, если она сетевая;
- Самый надежный и быстрый с точки зрения восстановления данных.

Дифференциальное резервное копирование

Принцип работы:

- сначала делается полное резервное копирование,
- затем при каждом запуске процесса резервируются только измененные данные, но точкой отсчета является состояние времени полного бэкапа.

Резервное копирование быстрее, чем полное, но медленнее, чем инкрементное. Восстановление наоборот. Памяти на определенный период меньше, чем у полного, но больше, чем у инкрементного.

Инкрементное копирование работает как дифференцированное копирование, но в отличии от него бэкапятся данные, которые были изменены из последнего слепка, то есть отправная точка каждого нового бэкапа это бэкап n-1.


---

### Задание 2

Установите программное обеспечении Bacula, настройте bacula-dir, bacula-sd,  bacula-fd. Протестируйте работу сервисов.

*Пришлите:*   
*- конфигурационные файлы для bacula-dir, bacula-sd,  bacula-fd,*   
*- скриншот, подтверждающий успешное прохождение резервного копирования.*

Ответ:

Конфигурацтонные файлы лежат в репозитории.

![screen1](https://github.com/KorolkovDenis/10.4-bacula-rsync/blob/main/screenshots/screen1.jpg)
![screen2](https://github.com/KorolkovDenis/10.4-bacula-rsync/blob/main/screenshots/screen2.jpg)

В директории /backup появился наш бэкап-файл

![screen3](https://github.com/KorolkovDenis/10.4-bacula-rsync/blob/main/screenshots/screen3.jpg)

---

### Задание 3

Установите программное обеспечении Rsync. Настройте синхронизацию на двух нодах. Протестируйте работу сервиса.

*Пришлите рабочую конфигурацию сервера и клиента Rsync блоком кода в вашем md-файле.*

Ответ:

сервер: /etc/rsyncd.conf

```
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
transfer logging = true
munge symlinks = yes
[data]
path = /root/
uid = root
read only =yes
list = yes
comment = Data backup Dir ][a][a][a
auth users = backup
secrets file = /etc/rsyncd.scrt
[new]
path = /etc/
uid = root
read only = yes
list = yes
comment = System fileset
auth users = backup
secrets file = /etc/rsyncd.scrt
```

/root/scripts/backup-node1.sh
```
#!/bin/bash
date
# Папка, куда будем складывать архивы — ее либо сразу создать либо не создавать а положить в уже существующие
syst_dir=/backup/
                                        # Имя сервера, который архивируем
srv_name=node2                   #из тестовой конфигурации
                                        # Адрес сервера, который архивируем
srv_ip=192.168.1.216
# Пользователь rsync на сервере, который архивируем
srv_user=backup
# Ресурс на сервере для бэкапа
srv_dir=data
echo "Start backup ${srv_name}"
# Создаем папку для инкрементных бэкапов
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir}
${syst_dir}${srv_name}/current/ --backup --backup-dir=${syst_dir}${srv_name}/increment/`date +%Y-%m-%d`/
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d
-mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"
```

клиент: /etc/rsyncd.conf

```
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
transfer logging = true
munge symlinks = yes
[data]
path = /etc/
uid = root
read only =yes
list = yes
comment = Data backup Dir ][a][a][a
auth users = backup
secrets file = /etc/rsyncd.scrt
[new]
path = /etc/
uid = root
read only = yes
list = yes
comment = System fileset
auth users = backup
secrets file = /etc/rsyncd.scrt
```

/root/scripts/backup-node1.sh

```
#!/bin/bash
date
# Папка, куда будем складывать архивы — ее либо сразу создать либо не создавать а положить в уже существующие
syst_dir=/backup/
                                        # Имя сервера, который архивируем
srv_name=clear                   #из тестовой конфигурации
                                        # Адрес сервера, который архивируем
srv_ip=192.168.1.100
# Пользователь rsync на сервере, который архивируем
srv_user=backup
# Ресурс на сервере для бэкапа
srv_dir=data
echo "Start backup ${srv_name}"
# Создаем папку для инкрементных бэкапов
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir}
${syst_dir}${srv_name}/current/ --backup --backup-dir=${syst_dir}${srv_name}/increment/`date +%Y-%m-%d`/
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d
-mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"
```

Итог запуска с сервера:

![screen4](https://github.com/KorolkovDenis/10.4-bacula-rsync/blob/main/screenshots/screen4.jpg)
![screen5](https://github.com/KorolkovDenis/10.4-bacula-rsync/blob/main/screenshots/screen5.jpg)


---

### Задание со звёздочкой*
Это задание дополнительное. Его можно не выполнять. На зачёт это не повлияет. Вы можете его выполнить, если хотите глубже разобраться в материале.

---

### Задание 4*

Настройте резервное копирование двумя или более методами, используя одну из рассмотренных команд для папки /etc/default. Проверьте резервное копирование.

*Пришлите рабочую конфигурацию выбранного сервиса по поставленной задаче.*


## Моя работа в Google:

[Моя работа по Резервному копированию. Bacula-Rsync.](https://docs.google.com/document/d/1z8p_pjVZVZo2qWvjZlIl89ogZE8y2k6c/edit?usp=share_link&ouid=104113173630640462528&rtpof=true&sd=true)