# Дополнительная информация

## Описание HIERA

Большинство параметров конфигурации передаются модулям через переменные hiera. Для hiera существует конфигурационный файл `/etc/puppet/hiera.yaml` в котором описана существующая иерархия. Верхние уровни иерархии имеют больший приоритет. Все настройки модулей puppet находятся внутри каталогов hieradata соответсвующего окружения и в hieracustom.

Текущая иерархия выглядит так (heira.yaml):

```
   ---
   :hierarchy:
   # Камтомизация
   # Специфично для ноды
     - "hieracustom/%{::env}/%{::hosttype}/node/%{::hostname}"
   # Специфично для $hosttype
     - "hieracustom/%{::env}/%{::hosttype}/common"
   # Глобально
     - "hieracustom/%{::env}/common"
     - "hieracustom/common"
   # Настройка "МОЙ ОФИС"
   # Настройки специфичные для определенной ноды
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/nodes/%{::fqdn}"
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/nodes/%{::hostname}"
   # Настройки не подпадающие не под одну из групп ниже
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/common"
   # Настройки firewall(Пока не реализовано)
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/firewall"
   # Настройки ipvs балансировщика
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/ipvs"
   # Настройки для web фронтендов(nginx)
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/web"
   # Настройки баз данных postgresql
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/databases"
   # Настройки объектного хранилища
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/swift"
   # Настроки для Redis
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/redis"
   # Настройки фирменных модулей FSAPI и NCTMAIL
     - "environments/%{::environment}/hieradata/%{::env}/%{::hosttype}/nct"
   # Описание групп и пользователей
     - "environments/%{::environment}/hieradata/%{::env}/users"
   # Общие настройки и настройки по-умолчанию
     - "environments/%{::environment}/hieradata/%{::env}/common"
   #  Содержимое settings.yaml администратор забирает в Файл hieracustom/common    чтобы не терялись настройки при обновлениях
   #  - "environments/%{::environment}/hieradata/%{::env}/settings"
   :backend:
     - yaml
   :yaml:
     :datadir: "/etc/puppet"
   :merge_behavior: deeper
   :deep_merge_options:
     :knockout_prefix: '--'
```

## Настройки общие для всех нод

Находятся в файлах `/etc/puppet/environments/box/hieradata/common.yaml` и `/etc/puppet/environments/box/hieradata/users.yaml`.

В `common.yaml` собрано большинство общих настроек.

| Класс  |  Описание |
|---|---|
|`ntp ` | Стандартный модуль для настройки ntp сервера |
|`stdlib ` | Стандартный модуль добавляющий множество полезных функций в puppet |
|`apipkg::common ` | Устанавливает кастомные пакеты, и собирает информацию о нодах|
|`selinux ` | Стандартный модуль для настройи selinux |
|`sysctl ` | Управляет переменными окружения ядра |
|`shinken::nrpe_client ` | Устанавливает nrpe демон для работы сервиса мониторинга[*] (не входит в обычную поставку) |
| `rsyslog` | Устанавливает и настраивает rsyslog демон.|
| `limits` | Управляет настройкой ulimit.|

В `users.yaml` собраны пользователи, групп, а так же правила sudoers. Используется модуль `nctsecurity`.

В большинстве случаев настройки `common.yaml`, `user.yaml` менять нет необходимости.

### Необходимые настройки

* `limits::use_hiera_hash: true` Устанавливает режим работы класса limits при котором все включения limits::fragments 'склеиваются' при помощи hiera merge lookup
* `sysctl::use_hiera_hash: true` Аналогично параметру `limits::use_hiera_hash`
* `sysctl::variables:` 
    `net.ipv4.tcp_fin_timeout` устанавливается значание 10
    `net.ipv4.ip_local_port_range` Установить '15000 65000'
* `ntp` Настраивает ntpd демон для всего кластера
    `ntp::servers` Устанавливает адреса ntp серверов для всего кластера. При необходимости можно поменять, указав вместо дефолтного gw.%{::domain} желаемое значение
* `common_repos` добавляет репозитории в систему. В релизной версии по умолчанию добавляется адрес сервера puppet-master на котором настроен yum репозиторий.
* `apipkg::common::packagelist_common` Список дополнительных пактов устанавливаемых не сервера. При необходимости можно добавлять желаемые пакеты сюда и они будут установлены на все сервера при запуске puppet-agent. Стоит помнить, что для установки будет использоваться единственный репозиторий указанный в `common_repos` и нужного пакета может там не быть.

## gw nodes

### Файлы настроек:

`gw/comon.yaml(sysctl, corosync, ntp)`

### Роли

* Шлюз для доступа в интернет всей ячейки.
* Сервер ntp для кластера

### Классы

`corosync` — cтандартный модуль менеджера кластера. Нужен для создания отказоустойчивых ресурсов.

### Необходимые настройки

* `sysctl::variables:`
   `net.ipv4.ip_forward` Включить.
   `net.ipv4.conf.all.rp_filter` Отключить для всех интерфейсов. Возможно стоит указать интерфейсы поименно.
   `net.nf_conntrack_max` Увеличивает размер таблицы conntrack до 1048576

* `corosync`

Создать ресурс `IPaddr2` с `ip = $vip_gw`. Этот ip будет шлюзом по умолчанию для всех нод кластера.

* `ntp::servers:`

Список внешних ntp серверов. Этои значения переопределяют соответсвующие из `common.yaml`, так как в дефолтной конфигурации сервер роли gw будет ntp сервером для всех серверов кластера.

## lb nodes

### Файлы настроек:

* `dd/common.yaml` (corosync)


### Роли

Балансировщик трафика для остальных сервисов

### Классы

* `corosync`
* `keepalived` — Управляет настройками ipvs и keepalived
* `ntp` — Управляет списком серверов времени
* 
### Необходимые настройки

* corosync:

Ресурсы:

  * `heartbeat:IPaddr2`(int_IP =`$vip_web`, mail_IP = `$vip_mail, real_ips`)
  * `systemd`(keepalived)
  * `heartbeat:Route`(default provider route) только если не хватает белых IP и существует необходимость маршрутизировать трафик.

Группы:

Все ресурсы надо сосредоточить в одной группе.

* keepalived:

Описываются все необходимые ipvs ресурсы. Примерный список ниже.

| IP  | Ноды  |
|---|---|
|`real_fsapi:80,443 `| fe_nodes|
|`vip_web:80,443,8183,8080 `| fe_nodes|
|`real_mail_smtp:25, 587`| ds_nodes|
|`vip_mail:25, 587 `| ds_nodes|
|`real_mail_imap:143,993 `| dd_nodes|
|`vip_imap:143,993 `| dd_nodes|
|`real_auth:80,443 `| auth_nodes|

## fe nodes

### Файлы настроек:

* `fe/common.yaml`(corosync, sysctl)
* `fe/nct.yaml`(fsapi — веб статика)
* `fe/web.yaml`(nginx)

### Роли

Web фронтенд. Запущен nginx распределяющий дальше нагрузку на fastcgi демоны бэкэндов

Тут же находится веб статика.

### Классы

* `corosync`
* `fsapi` — Устанавливает и настраивает fsapi
* `nginx` — Стандартный модуль для настройки nginx

### Необходимые настройки

* corosync:

на loopback должны быть подняты ip адреса `$vip_web`,  `$real_web` (при наличии) для обработки пакетов от балансировщика. Реализуется созданием ресурсов-клонов.

* sysctl:

Настройки для предотвращения ответов на arp запросы о адресах находящихся на lo интерфейсе.

  * `net.ipv4.conf.all.arp_ignore = 1`
  * `net.ipv4.conf.all.arp_announce = 2`
  * `net_ipv4_tcp_synack_retries = 2`

Отключить rp_filter. Лучше всего поинтерфейсно. В примере ниже приведена конфигурация для всех интерфейсв

```
net.ipv4.conf.all.rp_filter = 0
fsapi:
fsapi::install_web_static: true # Установить веб статику.
fsapi::install_fsapi: false # Важный параметр, отключающий всю установку fsapi.
```

Версии пакетов для web статики а так же их названия если нужно. По умолчанию ставятся пакеты версии develop (web-dev-$pkg_name), так что на продакшен окружении эти строки надо раскомментировать

```
#fsapi::pkgname_web_mail: 'web-mail-prod'
#fsapi::pkgname_web_admin: 'web-admin-prod'
#fsapi::pkgname_calendar: 'web-calendar-prod'
#:fsapi::pkgname_contacts: 'web-contacts-prod'
fsapi::pkgver_web_mail: "%{::nct_web_mail_ver}"
fsapi::pkgver_calendar: "%{::nct_web_calendar_ver}"
fsapi::pkgver_contacts: "%{::nct_web_contacts_ver}"
fsapi::pkgver_web_admin: "%{::nct_web_admin_ver}"
```

* nginx:

Настраиваются типы логов, количество воркеров и др параметры.

Описание всех вирт-хостов и их параметров. Сам конфиг очень объемный, подробнее про свойства и параметры в офф руководстве.

Для создания upstreams используются строки `*_backends определенные в manifests/globals.pp`

Для персонфикации имен хостов, переменная ::env

## dd nodes

### Файлы настроек:

* `dd/common.yaml`(corosync, sysctl)
* `dd/nct.yaml`(nctmail::fe)

### Роли

IMAP фронтенд. Запущен dovecot в режиме dovecot-director, который транслирует входящие imap запросы на dovecot сервера (ds)

### Классы

* `corosync`
* `sysctl` — Управляет переменными sysctl
* `nctmail::fe` - dovecot director

### Необходимые настройки:

* corosync:

на loopback должны быть подняты ip адреса `$vip_imap`, `$real_imap` (при наличии) для обработки пакетов от балансировщика. Реализуется созданием ресурсов-клонов.

* sysctl:

Настройки для предотвращения ответов на arp запросы о адресах находящихся на lo интерфейсе.

`net.ipv4.conf.all.arp_ignore = 1`

`net.ipv4.conf.all.arp_announce = 2`

Отключить rp_filter. Лучше всего поинтерфейсно. В примере ниже приведена конфигурация для всех интерфейсв

`net.ipv4.conf.all.rp_filter = 0`

* nctmail::fe

Обращаем ваше внимание, что не допускается совмещать сервера **dd** и **lb**, так как это повлечёт за собой проблемы с синхронизацией сообщений для почтовых клиентов.

## be nodes и ds nodes

### Файлы настроек

* `be/common.yaml`(corosync, sysctl)
* `be/nct.yaml`(fsapi, nctmail*, nctfiles)

### Роли

* fsapi — fastcgi-демоны
* dovecot — роль `ds_node`
* postfix — роль `ds_node`

### Классы

* `corosync`
*  `fsapi` — Устанавливает и настраивает fsapi(webapi,webdav,caldav,carddav,webadinapi,noted,exnoted)
*  `nctmail::be` — Устанавливает почтовую подсистему ролей ds
*  `nctmail::ma` — Устанавливает mailapi
*  `nctmail::websmtpapi` — Устанваливает websmtpapi
*  `nctmail::cp` — Настройка сервера политик Cluebringer
*  `nctmail::am` — Настройка antispam-antivirus amavis-clamd
*  `nctfiles` — Лежат необходимые для работы файлы, серификаты итд.

### Необходимые настройки

* corosync:

* sysctl:

аналогично fe(dd)

* fsapi[Подробнее о модуле](fsapi.md)

Устанавливаются и конфигурируются пакеты fsapi (`fsapi-bin`, `fsapi-swift`, `fsapi-utils`, `fsapi-calapi`, `fsapi-webapi`, `fsapi-webdav`, `fsapi-templates`, `fsapi-webadminapi`)

`fsapi::pkgver_*` Устанавливает соответствующий пакет. Этот блок можно взять целиком из Samples. Настройки берутся из `globals.pp` (`default = absent`)

`fsapi::**_processcount:` Количество воркеров для соответствующего Fastcgi демона. Так же берется из Samples (`defaut = 30`)

`fsapi::**_port:` Порт соответвующего сервиса. Дефолтное знаечение см в коде. Берется из Samples.

Нужно взять из сэмплов. Общие настройки fsapi там тоже перечислены. Настройки для noted(exnoted) так же берутся из сэмплов

`fsapi::fsapi_pg_port:` Порт для доступа к postgres через pgbouncer установить  6432 (`default =5432`)

`fsapi::fsapi_pg_searCh_port:` аналогично `fsapi::fsapi_pg_port`

`fsapi::options:` Хэш `fsapi::options` принимает кастомные параметры для конфигурирования файла `.config.inc`. Ниже приведены несколько необходимых его ключей

 * `pg_server_prepare:` Если используется pgbouncer необходимо выставить 0
 * `sync_host:`  Путь к базе redis. Добавлен в fsapi v4.18 пока необходимо указывать хост  единственного редис сервера в ячейке(`%{::vip_redis}`), в будущем возможно другое значение.
 * `nctmail::be:`

Настройка postfix + dovecot  для роли ds

`nctmail::be::web_api_url:` Путь к REST интерфейсу fsapi(`default = "http://fsapi.${::domain}" `)

`nctmail::be::dovecot_package_provider:` repo или file(default = repo)

`nctmail::be::dovecot_package_release:` Релиз Dovecot-a (default = $::nct_dovecot_release)

`nctmail::be::dovecot_fsapi_plugin_package_version:` (`default = $::nct_dovecot_plugin_ver`)

## db nodes

### Файлы настроек

* `db/common.yaml`(corosync)
* `db/databases.yaml` (postgresql)
* `db/nodes/db_fqdn.yaml` (postgresql)
* `db/redis.yaml` (redis)

### Роли

Сервер базы данных fsapi, postgresql и redis

### Классы

* `corosync`
* `postgresql`
* `redis`

### Необходимые настройки

* corosync:

Создаются ресурсы:

* `db_IP` — общий адрес для доступа к кластеру postgres
* `redis_IP` — общий адрес для доступа к кластеру redis
* `pgbouncer` — ресурс клон, запущен на всех нодах. Аггрегатор соединений для постреса
* `redis` — ресурс создает master/slave сущность для redis (пока не в продакшен)

Создается группа `redis_master` + `redis_IP` (пока не в продакшен)

* postgresql:

Конфигурация для сервера баз данных postgresql разбита на две части. Первая в файле `databases.yaml`, здесь представлены общие настройки для всех инстансов postgres.
* `postgresql::server::listen_addresses:` Адрес который будет слушать постгрес(`default = 127.0.0.1`)
* `postgresql::server::standby_create_recovery: false` Отключает создание файла `recovery.conf` для слейв серверов. Созданием этого файла займется pacemaker
* `postgresql::server::server_mode: 'standby-sync'` Режим настройки postgres. Может быть (master,  standby-sync, standalone) В этом режиме паппет запускает процедуру копирования базы с сервера имеющего роль master
* `postgresql::server::needs_initdb: false` Отключает инициализацию базы данных. Для режима  standby-sync это вредно.
* `postgresql::server::service_manage: false` Отключает управление состоянием сервиса postgres. Этим занимается pacemaker.
* `postgresql::server::postgresql_config_entries:` Хэш параметров для конфиг файла postgres.  При необходимости сюда можно добавлять желаемые настройки.
* `postgresql::server::postgresql_pg_hba_rules:` Хэш параметров для файла контроля доступа к базе данных postgres (pg_hba)
* `postgresql::server::pgbouncer_enable: true` Устанавливает pgbouncer
* `postgresql::pgbouncer::pgbouncer_service_ensure: (default = running)` Запускает сервис pgbouncer
* `postgresql::pgbouncer::pgbouncer_service_enable:` Включает автозапуск при старте системы для pgbouncer
* `postgresql::pgbouncer::default_pool_size: 1000` Размер пула
* `postgresql::pgbouncer::max_client_conn: 10000` Количество разрешенных одновременных подключаний.
* `postgresql::pgbouncer::databases:` Хэш путей к базам данных. В нашем случае все находится в одном инстансе postgres на 127.0.0.1
* `postgresql::pgbouncer::pgbouncer_users:` Хэш пользователей для доступа к базам данных. Генерится на основании паролей из `globals.pp`

Вторая часть настроек postgresql вынесена в отделный фаил вида `db/nodes/db_fqdn.yaml. df_fqdn` — это полное имя db ноды на которой будет создаваться кластер postgres. Так как сам кластер баз данных будет создаваться именно тут, а затем реплицироваться по всем нодам находящимся в режиме standby-sync то тут создаются все необходимые базы данных и настройки доступа к ним.

* `postgresql::server::node_list: "%{::db_node_list}"` Список нод для репликации. Необходим для создания ресурса corosync
* `postgresql::server::server_mode: 'master'` Роль master. По иерархии перезаписывает роль  `standby-sync` из конфиг файла `databases.yaml`
* `postgresql::server::needs_initdb: true` Тут как уже говорилось выше необходимо инициализировать кластер базы данных
* `postgresql::server::service_manage: true` Запускает сервис postgres
* `postgresql::postgresql_databases:` Хэш баз данных для создания. На db нодах создается только база `::fs_db_name` Тут же создается пользователь-владелец базы данных
* `postgresql::postgresql_roles:` Хэш для создания пользователей, в дополнение к владельцам.
* `postgresql::postgresql_database_grants:` Хэш для назначения прав доступа для пользователей.

* redis:
 `redis::redis_instances:` Хэш содержащий список инстансов для создания redis баз данных. В нашем случае создается 1-н инстанс работающий на дефолтном порту 6379. (см описание класса redis)
 `service_ensure: 'running'` - Запускает сервис
 `service_enable: true` — Включает автозагрузку

**ВАЖНО!** Настройка нод `db_nodes` происходит в 2-а этапа. Для начала, на выбраной ноде  `df_fqdn` (см. выше) запускается puppet agent. После окончания работы необходимо создать кластерный ресурс pacemaker для базы данных postgresql. Для этого рекомендуется воспользоваться скриптом `/root/bin/create_postgresql_cluster.cib` (генерится автоматически при помощи puppet) Запуск скрипта `sh /root/bin/create_postgresql_cluster.cib` создаст master/slave ресурс `pgsql_res`, и добавит зависимости для ресурсов `pgbouncer` и `db_IP`. (`pgbouncer` →  `ms_pgsql_res`(роль мастер) → `db_IP`). Таким образом `db_IP` не появится до тех пор пока не запущены два предыдущих ресурса, а мастером postgresql не станет никогда если не запущен pgbouncer. После запуска скрипта необходимо подождать 1 минуту, после чего проверить статус ресурса `pgsql_res`.

Если ресурс остановлен, выполнить команду `pcs resource cleanup  pgsql_res` и проверить что в течение 1 минуты статус ресурсов `pgsql_res` , а также `db_IP` (по зависимости) изменились на 'Started'

В том случае если автоматически не поднимается Slave postgres либо если в процессе эксплуатации потребуется восстановить кластер после failover рекомендуется воспользоваться скриптом `/root/bin/restore_script.sh`

После настройки ноды `db_fqdn` можно запускать puppet agent на всех нодах типа db.

## ms nodes

### Файлы настроек

* `ms/common.yaml`(corosync)
* `ms/databases.yaml`(postgresql)
* `ms/nodes/ms_fqdn.yaml`(postgres, puppetdb)
* `ms/nct.yaml` (nctindexer)

### Роли

Сервер базы данных postgresql для службы индексатора и почтовой системы

Сам индексатор так же запущен на одной из нод `ms_nodes`

На одной из нод этого типа рекомендуется устанавливать Puppetmaster, хотя подойдет и любая db нода.

**ВАЖНО!** С настройки puppetmaster начинается вся настройка окружения.

### Классы

* `corosync`
*  `postgresql`
* `puppetdb` - Сервер puppetdb необходимый для хранения данных puppet.

### Необходимые настройки

* `corosync`

Создаются ресурсы:

 `ms_IP` — общий адрес для доступа к кластеру postgres для нод типа ms;

 `nct_indexer` — Простой ресурс, запускающий ровно 1-у копию индексатора.

 `pgbouncer` — ресурс-клон, запущен на всех нодах. Аггрегатор соединений для постреса

* `postgresql`

Настройки полностью аналогичны db_nodes

* `nctindexer`

`nctindexer::package_version: "%{::nct_indexer_ver}"` Устанавливает пакет индексатора

* `puppetdb`

Настройки для этого класса находятся исключительно в `ms/nodes/ms_fqdn.yaml`. Сервер требует для своей работы postgres, поэтому при не работающем кластере postgresql ничего работать не будет и запуск puppet agent-а будет выдавать ошибку.

 * `puppetdb::server::database_host: "%{::vip_ms}" `Путь к базе данных postgresql. указывает на кластерный ip для ms нод. (`default = localhost`)
 * `puppetdb::server::listen_port: '8180'` Порт который слушет puppetdb (`default = 8080`)
 * `puppetdb::database::postgresql::manage_server: false` Отключаем контроль за запуском сервера postgresql из класса puppetdb, этот параметр контролируется из класса postgresql::server

**ВАЖНО!** Действия по раздельной настройке нод типа ms аналогичны действиям для нод типа db. Только в этом случае, необходимо после настройки кластера postgresql запустить  puppet agent еще раз, чтобы все необходимые данные попали в puppetdb, которая была установлена на предыдущем этапе

## st nodes

### Файлы настроек

* `st/common.yaml`(corosync)
* `st/nodes/st_fqdn.yaml`(swift::ringbuilder_init)
* `st/swift.yam`(swift, memcached, mlocate)

### Роли

Сервера объектного хранилища swift

### Классы

`corosync`

`memcached` — Устанавливает на всех st нодах memcached сервера.

`mlocate` — Управляет работой mlocate демона.

`swift` — Базовый класс для настройки swift

`swift::storage::all` — Cоздает устройства swift (account/container/object-s)

`swift::proxy` — Настраивает swift-proxy

`swift::proxy::cache` — Настройки кэширования swift proxy

`swift::proxy::healthcheck` — Настройки healthcheck.

`swift::proxy::tempauth` — Настройки авторизации.

`swift::deploy` — Экспортирует информацию о вновь созданных объектах swift.


### Необходимые настройки

* corosync:

Создаются ресурсы:

`swift_IP` — Общий ip для доступа к кластеру swift

* mlocate:

`mlocate::prunepaths: "%{::swift_device_path}"` Удаляет из базы поиска mlocate диски на которых хранятся объекты swift

* memcached:

 Все настройки берутся по умолчанию.

Использует переменную `swift_memcached_servers` из блока #[Swift] файла настроек `globals.pp`

* swift:

 * `swift::apply_sysctl_rules: "%{::swift_apply_sysctl_rules}"` Применяет дополнительные настройки для ядра linux
 * `swift::package_ensure: '1.13.1-1.el6'` — Версия для установки
 * `swift::proxy::port: "%{::swift_proxy_port}"` Порт swift-proxy
 * `swift::swift_hash_suffix: "%{::env}_swift_hash"` — Специальный ключ для хэширования объектов в swift.
 * `swift::storage::all::storage_local_net_ip: '0.0.0.0'` — К какому адресу биндить устройства.
 * `swift::storage::all::devices: "%{::swift_device_path}"` — Корень для смонтированных файловых систем swift-а
 * `swift::proxy::proxy_local_net_ip: '0.0.0.0'` — Адрес который слушает swift-proxy
 * `swift::proxy::cache::memcache_servers: "%{::swift_memcached_servers}"` Список mamcached серверов.
 * `swift::proxy::cache::has_memcached: true` — Устанавливает mamcached на сервера
 * `swift::deploy::ring_device: "%{::swift_ring_device}"` — Список смонтироанных файловых систем (xfs) в папке `swift::storage::all::devices:` В этих файловых системах будут размещаться объекты swift.
 * `swift::ringbuilder_init`:

Этот класс запускается с одной ноды, заранее выбранной `st_fqdn`, на которой генерится кольцо swift. Настройки находятся в файле `st/nodes/st_fqdn.yaml`

 * `swift::ringserver::local_net_ip`:  '0.0.0.0' Адрес который будет слушать ring_server
 * `swift::ringbuilder_init::ring_server: "%{::ipaddress}"` Адрес, который узнают остальные st ноды и с которго скачают файлы кольца.

**ВАЖНО!**

Файловые системы xfs для работы swift необходимо монтировать с опциями: 

```
noatime,nodiratime,nobarrier,logbufs=8
```

Порядок запуска puppet agent рекомендуется следующий. Сначала на всех нодах кроме той, на которой генерится кольцо (эта нода которой соответствует файл конфигурации `hieradata/st/nodes/st_fqdn.yaml`). Затем агент запускается на ноде, на которой генерится кольцо, а затем его необходимо запустить ещё раз, на всех нодах, для того чтобы кольцо было распространено между всеми экземплярами swift.

Настоятельно рекомендуется,  до дальнейших указаний отключить работы служб `swift-object-auditor`, `swift-container-auditor`, `swift-account-auditor`, сняв с соответсвующих бинарников бит исполнения, и остановив сервисы командой

```
systemctl stop openstack-$SERVICE_NAME
```
