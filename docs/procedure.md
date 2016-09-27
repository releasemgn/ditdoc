# 4. <a name="procedure">Процедура Настройки</a>

## 4.1 <a name="install_puppetmaster">Конфигурирование сервера puppet master</a>

Начиная с версии ElderFlower 2016.02, необходимо устанавливать рекомендованную OC "Scientific linux 7" самостоятельно или её аналог. Рекомендуемый вариант инсталляции "minimal". Образ установочного диска можно получить по ссылке: http://ftp1.scientificlinux.org/linux/scientific/7x/x86_64/iso/SL-7-DVD-x86_64.iso .

Разбеение дисков выполните по инструкции [Предварительные действия](preliminary). После этого на серверах роли st должны быть смонтированы разделы /node/srv/. 

Перед началом установки убедитесь, что DNS работает корректно и система может разрешить необходимые имена ([пример настройки](myoffice_infrastructure)).

Обращаем ваше внимание, что не допускается совмещать сервера **dd** и **lb**, так как это повлечёт за собой проблемы с синхронизацией сообщений для почтовых клиентов.

### 4.1.1 Запуск установки

Очистите содержимое директории `/etc/yum.repos.d/`:
```
mkdir /root/yum.repos.d
mv /etc/yum.repos.d/* /root/yum.repos.d/
```

Примонтируйте диск с дистрибутивом Мой Офис® `mount /dev/sr0 /mnt`

Запустите установку `/mnt/scripts/install_puppetmaster.sh`

Если Вы устанавливаете односерверную конфигурацию (на одну ноду) то используйте ключ `--single`

### 4.1.2 Название окружения

В предыдущих версиях название окружения вычислялось из доменного имени "<окружение>.nct". Например если серверы установлены с использованием внутреннего домена myenv.nct, то название окружения будет "myenv".

Начиная с релиза Elderflower название окружения по умолчанию равно "box". Чтобы переопределить его необходимо в файл /etc/puppet/manifests/custom.pp добавить строчку: 

```
$env_custom = 'myenv'
```

Где *myenv* - это название окружения соответствующее регулярному выражению /[a-z0-9_-]+/

От названия окружения зависят названия баз данных, именование путей к файлам конфигурации и значения прочих переменных.

### 4.1.3 <a name="auto_hiera">Установка конфига HIERA с помощью конфигуратора AUTO HIERA</a>

Перейдите в папку конфигуратора `cd /opt/auto_hiera_config/`

Перед запуском конфигуратора необходимо указать какие роли будут настроены на хостах.

Используйте конфиги-образцы для создания своего файла:

```
configs_box_single.yaml - образец конфига для установки на один узел
configs_box_legacy.yaml - образец конфига для установки на десять узлов
```

Не вносите изменения в образцы, т.к. они будут стёрты при обновлении конфигуратора пакетом rpm.

Формат файла (YAML):

```
"название_окружения":
  <hostname group>:
    options:
      standalone: true
    roles:
      - <role>
      - <role>
  <hostname group>:
    roles:
      - <role>
      - <role>
```

В случае использования одного сервера в составе группы, необходимо указать опцию `standalone: true`. Например если у вас 2 ноды db1.box.nct и db2.box.nct, то в группе db указывать опцию standalone не нужно. А если у вас только одна нода st1 в группе st, то добавьте опцию standalone. 

Запустите `bundle exec bin/construct_config.rb -c <имя конфигурационного файла yaml>`

Если ошибок нет, вы получите ответ 'Done'.

В директории `/etc/puppet/environment/production/hieradata/название_окружения/` появится сконфигурированая иерархия конфигураций hiera.

### 4.1.4 Файл настроек

Скопируйте hieracustom `cp /etc/puppet/environments/production/hieradata/SAMPLES/hieracustom/* /etc/puppet/hieracustom/`

Отредактируйте параметры в файлах `/etc/puppet/hieracustom/*`. Кроме параметров, указаных в примере, вы можете добавлять другие. 

Далее перечислены основные разделы:

#### 4.1.4.1 [Domains]

| Переменная  | Описание  |
|---|---|
|`ext_domain` | Домен по которому кластер будет доступен извне. Используется в шаблонах для вирутал-хостов nginx |
|`vhost_suffix` | Суффик для виртуал-хостов nginx. По умолчанию применяется при создании всех виртуал-хостов |
|`mail_domain`|Основной почтовый домен для почтовой подсистемы|

#### 4.1.4.2 [Networks]

Параметры сетей. Используются в модулях corosync, postgres, nctmail:fe.

| Параметр  | Описание  |
|---|---|
|`lan_network`            |Адрес локальной сети с маской вида IP-адрес/XX, где XX — маска. Используется в модулях nctmail, postgres# для определения доверенной сети|
|`data_network_address`   |IP-адрес локальной сети кластера. Используется в corosync для определения интерфейса, к которому должна быть выполнена привязка. Пример: если адрес интерфейса 10.1.2.3, то data_network_address = '10.1.2.0'|
|`data_network`           |Внутренняя сеть ячейки IP-адрес/XX, где XX — маска. Отличие от lan_network состоит в том, что у нас сеть для серверов использует 16-битную маску, а сегмент для ячейки – 24-битную   |
|`trusted_networks`       |Перечисление сетей для acl-ек dovecot (nctmail:be, nctmail:fe). Значение имеет тип “массив”   |

#### 4.1.4.3 [Nodes]

| Параметр  | Описание  |
|---|---|
| `*_nodes`  | IP-адреса нод кластера перечисленных ролей. Разделяются запятыми  |
| `auth_nodes`  | IP-адреса нод SSO. Разделяются запятыми  |

#### 4.1.4.4 [Virtual IPs]

Описание кластерных IP. Все IP, которые начинаются на vip, являются внутренними для ячейки адресами из data_network. Они используются главным образом в настройках corosync / ipvs через hiera.

| Параметр  | Описание  |
|---|---|
|`vip_auth`   |Адрес хоста SSO. Должно соответствовать DNS-имени auth.domain.domain|
|`vip_web`   |Адрес хоста веб-сервера. Должно соответствовать DNS-имени be.domain.domain|
|`vip_imap`   |Адрес хоста IMAP-сервера. Должно соответствовать DNS-имени для be.domain.domain|
|`vip_mail`   |Адрес хоста почтового-сервера. Должно соответствовать DNS-имени для be.domain.domain|
|`vip_gw`   |Адрес хоста GW-сервера. Должно соответствовать DNS-имени gw.domain.domain|
|`vip_db`   |Адрес хоста DB-сервера. Должно соответствовать DNS-имени для db.domain.domain|
|`vip_srch_db`   |Адрес хоста search-сервера (indexer). Должно быть равно `vip_db` |
|`vip_mail_db`   | Должно быть равно для быть равно `vip_db`   |
|`vip_redis`   | Адрес хоста redis-сервера. Должно соответствовать DNS-имени redis.domain.domain. Не должно быть одинаковым с vip_db|
|`vip_swift`   |  Адрес хоста для соединения с swift-proxy. Должно соответствовать DNS-имени swift.domain.domain  |
|`real_fsapi`  | Внешний адрес FSAPI |
|`real_auth`   | Внешний адрес SSO |

#### 4.1.4.5 [Cluster]

| Параметр  | Описание  |
|---|---|
|`mail_db_host` |Адрес хоста базы данных для почты. Должно соответствовать DNS-имени для vip_mail_db|
|`db_node_list` |Ноды кластера postgres, перечисленные через запятую. Эти значения необходимы для pacemaker – ресурса postgres'а   |
|`ms_node_list` |Аналогично `db_node_list`, но для базы ms (равно db_node_list)   |
|`logs_node`    | Массив Хостов или IP на которые будут собираться логи. На коллекторах логи собираются в папке /var/log/rsyslog|

#### 4.1.4.6 [Mail]

Конфигурация почты.

| Параметр  | Описание  |
|---|---|
|`alert_emails` | Список email адресов разделенных запятой, на которые отсылаются уведомления.|
|`doveadm_pwd`|Пароль для доступа в админку dovecot directors|
| `install_ews` | Поставить true для установки EWS библиотеки |
|`aliases` | Переменная указывающая на почтовые алиасы для окружения. | 
| `sa_tag_level_deflt` | Оценка выше которой добавляются X-Spam заголовки в письмо |
| `sa_tag2_level_deflt` | Оценка выше которой добавляется X-Spam-Flag заголовок, указывающий почтовому клиенту что письмо считается спамом |
| `sa_kill_level_deflt` | Оценка по достижении которой, к письму применятся действие опеределенное в final_spam_destiny |
| `final_spam_destiny` | Действие с письмом которое набрало оценку не меньше чем `sa_kill_level_deflt`. По умолчанию применяется действие D_PASS |

#### 4.1.4.7 [GoogleAccount]

Настройки google аккаунта для возможности добавления аккаунтов пользователями google. Подробнее смотри  

| Параметр  | Описание  |
|---|---|
|`google_redirecturi_nul`   | Authorized redirect URI. Настраивается в аккаунте google. По умолчанию `https://mail-%{::env}.%{::ext_domain}/?from=google` |
|`google_client_id`   | Соответствующие данные из аккаунта google  |
|`google_client_secret`   | Соответствующие данные из аккаунта google  |

#### 4.1.4.8 [Databases]

Описание пользователей БД. Переменная с окончанием `_user` – имя соответствующего пользователя, `_pwd` – пароль, `_name` – имя БД.

| Параметр  | Описание  |
|---|---|
|`db_mon_user`|Пользователь для мониторинга, имеет права на LOGIN + connect  |
|`db_mon_user_pwd`| |
|`postgres_pwd`| |
|`fs_db_name`|Основная база данных fsapi |
|`fs_db_user`|Пользователь основной базы данных fsapi |
|`fs_db_pwd`| |
|`srch_db_name`|База данных индексатора |
|`srch_db_user`| |
|`srch_db_pwd`| |
|`vmail_db_name`|База данных почтовой MTA |
|`vmail_db_ro_user`|Пользователь основной базы данных почтовой системы (read only) |
|`vmail_db_ro_user_pwd`| |
|`vmail_db_admin_user`|Пользователь основной базы данных почтовой системы |
|`vmail_db_admin_user_pwd`| |
|`amavis_db_name`|База данных антиспам/антивирус демона amavis |
|`amavis_db_user`|Пользователь базы антиспам/антивирус демона amavis |
|`amavis_db_user_pwd`| |
|`cluebringer_db_name`| База данных демона политик почты|
|`cluebringer_db_user`| Пользователь базы демона политик почты|
|`cluebringer_db_user_pwd`| |

#### 4.1.4.9 [Swift]

Настройки объектного хранилища swift.

| Параметр  | Описание  |
|---|---|
|`swift_apply_sysctl_rules = true`|Сообщает модулю о необходимости применить настройки ядра для улучшения работы |
|`swift_device_path`| Путь к корню каталогов swift|
|`swift_ring_device`|Список смонтированных фс которые использует swift (через запятую) |
|`swift_proxy_port`|Порт для swift-proxy. По умолчанию порт 8080, но это значение часто используется другими сервисами и скорее всего требует переопределения |


#### 4.1.4.10 [Security]

Секция для настроек безопасности стенда

| Переменная  | Описание  |
|---|---|
|`allow_ssh_root` | разрешать доступ на все ноды по ssh пользователю root. Значение по умолчанию - `'no'`|
|`ssl_crt_path`  | Путь к файлу сертификата для работы https (nginx) и tls (postfix, dovecot)   |
|`ssl_key_path`  | Путь к файлу секретного ключа  |
|`krb_realms`| реалмы kerberos для работы с AD. Более подробно можно прочитать в [6.4] |
|`nctsecurity::users:`| Хэш. Определяет пользователей, их группы и пароли. Пример ниже.|
|`nctsecurity::ssh_keys:`| Хэш. Определяет ssh ключи пользователей. Пример ниже.|

Обратите внимание, что у пользователя root пароль **не сменится**.

```
nctsecurity::users:
  root:
    password: '$4$RootPasswordHash'
  sample_username:
    ensure: present
    comment: 'Surname Name'
    groups: ['wheel', 'ssh_users', 'power_users']
    shell: '/bin/bash'
    home: '/home/sample_usersname'
    managehome: true
    password: '$6$SampleUserNamePasswordHash'
  sample_username_2:
    ensure: present
    comment: 'Surname Name 2'
    groups: ['wheel', 'ssh_users', 'power_users']
    shell: '/bin/bash'
    home: '/home/sample_usersname_2'
    managehome: true
    password: '$6$SampleUser2NamePasswordHash'

nctsecurity::ssh_keys:
  sample_username_ssh_key:
    name: 'sample_username@qnx'
    user: 'sample_username'
    key: 'AAAAB/SSHPublicKey'
    type: 'ssh-rsa'
  sample_username_2_ssh_key:
    name: 'sample_username_2@qnx'
    user: 'sample_username_@'
    key: 'AAAAB/SSHPublicKey2'
    type: 'ssh-rsa'
```        

#### 4.1.4.11 [Dnsqmasq]

Если у вас используется dnsmasq то рекомендуем отказаться от этой практики в пользу более надёжных DNS. dnsmasq поставляется вместе с продуктом с целью облегчения тестирования.

Все виртуальные адреса будут настроены в нём автоматически на основе секции `[Virtual IPs]` из файла конфигурации common.yaml (см. п. 4.1.4.4).

По умолчанию dnsmasq выключен. Чтобы включить dnsmasq добавьте в кастомную часть Хиеры для роли, накоторой расположен `puppetmaster` например в `/etc/puppet/hieracustom/db/common.yaml` следующие параметры:

```
apipkg::dnsmasq::srv_ensure: running
apipkg::dnsmasq::srv_enable: true
apipkg::dnsmasq::external_dns:
  - "8.8.8.8"
  - "8.8.4.4"
apipkg::dnsmasq::custom_records:
  puppet.box.nct: "10.0.25.50"
  db1.box.nct:    "10.0.25.50"
  db2.box.nct:    "10.0.25.51"
  db3.box.nct:    "10.0.25.52"
  be1.box.nct:    "10.0.25.61"
  be2.box.nct:    "10.0.25.62"
  be3.box.nct:    "10.0.25.63"
  be4.box.nct:    "10.0.25.64"
  st1.box.nct:    "10.0.25.71"
  st2.box.nct:    "10.0.25.72"
  st3.box.nct:    "10.0.25.73"
  st4.box.nct:    "10.0.25.74"
  gw1.box.nct:    "10.0.25.11"
  gw1.box.nct:    "10.0.25.12"
  lb1.box.nct:    "10.0.25.21"
  lb2.box.nct:    "10.0.25.22" 
```

Замените значения по умолчанию в списке external_dns на свои DNS, а в custom_records пропишите ваши хосты.

Чтобы подключить все серверы к этим DNS добавьте в `/etc/puppet/hieracustom/common.yaml` следующие параметры:
```
apipkg::dnsmasq::resolv_conf::enable: True
apipkg::dnsmasq::resolv_conf::nameservers:
  - "10.0.25.50"
  - "10.0.25.51"
  - "10.0.25.52"
```

Замените IP адреса в `nameservers` чтобы они соответствовали puppetmaster и одноименным ноды той-же группы.

Перед первым прогоном `puppet agent -t` на остальных серверах, отредактируйте /etc/resolv.conf вручную и установите `nameserver: <puppetmaster ip>`  

#### 4.1.4.12 `fail_ip_skip`

Параметр`fsapi::options:fail_ip_skip` содержит пречисление сетей, при соединении из которых не используется блокировка IP после превышения порога ошибок авторизации.
Значение параметра должно содержать как минимум loopback-сеть (127.0.0.0/8) и внутреннюю сеть кластера ($lan_network) разделенных запятыми.
Задание сетей в виде подстроки, например '192.168.' для сети '192.168.0.0/16' также поддерживается, но является устаревшим и будет отключено в последующих версиях.

#### 4.1.4.13 Брендирование

В дистрибутиве присутствуют пакеты содержащих файлы брендирования. Чтобы определить какие версии пакетов есть в вашей системе выполните команду на сервере puppetmaster (db1): `find /var/repo_nct/ -name 'fsapi-template*'`
Первый из них устанавливается по-умолчанию и содержит файлы для установки под брендом myoffice.ru. Второй предназначен для установки под брендом collabio.com.
Для смены устанавливаемого пакета необходимо добавить параметр `nct_fsapi_templates_ver` в значении которого указать версию пакета fsapi-templates:

```
nct_fsapi_templates_ver: 1.8col-1.sl7
```

#### 4.1.4.14 Количество потоков FSAPI

Значения по умолчанию указаны в таблице ниже. Вы можете переопределить их в hieracustom/common.yaml

```
# [NCT soft pcount]
nct_mailapi_pcount:         "30"
nct_websmtpapi_pcount:      "10"
nct_webapi_pcount:          "30"
nct_webapi_pcount_users:    "10"
nct_webapi_pcount_fs_imap:  "10"
nct_webapi_pcount_fs_slow:  "10"
nct_webapi_pcount_fs_fast:  "10"
nct_webapi_pcount_others:   "10"
nct_webdav_pcount:          "30"
nct_carddav_pcount:         "30"
nct_caldav_pcount:          "30"
nct_webadminapi_pcount:     "30"
nct_webbackupapi_pcount:    "10"
nct_adminapi_ui_pcount:     "10"
```

### 4.1.5 Действия на сервере конфигурации

#### 4.1.5.1 EWS

Скачайте и установите EWS библиотеку. Работа системы без библиотеки EWS невозможна! [См Установка EWS](enable_ews_support)

#### 4.1.5.2 Запустите puppetmaster:

```
systemctl start puppetmaster
```

#### 4.1.5.3 Запуск puppet agent: 

```
puppet agent -t
```

Этот запуск должен завершиться без ошибок. Если ошибки есть, то запустите повторно. 

### 4.2 Конфигурирование серверов системы

#### 4.2.1 На всех остальных нодах настройте сеть

Необходимо чтобы по команде `hostname` выводилось имя полностью вместе с доменом, для этого выполните команду `hostname $FQDN`, заменяя $FQDN на имя полностью вместе с доменом.

Пропишите в /etc/hosts строку, заменя переменные на свои значения

```
$IPADDR $FQDN $HOST
```

Пропишите в /etc/resolv.conf, заменя переменные на свои значения
```
cat > /etc/resolv.conf <<EOF
search $DOMAIN
nameserver $DNSIP
EOF
```


#### 4.2.2 Выполните перечисленные ниже команды на всех остальных нодах кроме серверов роли st и db:

```
curl puppet:3128/misc/install_agent.sh | bash

puppet agent -t
```
   
Если есть ошибки, то запустите команду `puppet agent -t` еще раз. 

#### 4.2.3 Запуск puppet agent на серверах роли db

1) Выполните пункт 4.2.1 на всех серверах роли db.

2) Выполните на одной из нод роли db команду `pcs resource cleanup pgsql_res`

#### 4.2.4 Запуск puppet agent на серверах роли st

Процедура на серверах роли st должна выполняться пользователем root в следующем порядке:

1) Выполните пункт 4.2.1 на всех серверах, кроме генератора кольца (т.е. кроме сервера st1).

2) Выполните пункт 4.2.1 на генераторе кольца.

3) Выполните `puppet agent -t` на всех серверах, кроме генератора кольца, повторно, для того чтобы кольцо было распространено между всеми экземплярами swift.

Если есть ошибки, то запустите команду `puppet agent -t` еще раз.

4) Убедитесь, что сервис swift-proxy запущен. Для этого выполните команду `systemctl status openstack-swift-proxy`

#### 4.2.5 Удаление неиспользуемых ресурсов на балансировщике

Если вы не используете vip_auth, real_fsapi, или real_auth, то необходимо данные ресурсы удалить с роли lb.

Выполните команду на одной из нод роли lb:

```
pcs resource delete vip_auth
pcs resource cleanup keepalived
```

### 4.3  Заключительный этап конфигурации

4.3.1 Выполните от имени root после конфигурирования всех нод кластера, на любом из серверов FS Backend (be), следующие команды:

```
export LINES=24
export COLUMNS=80
cd /usr/local/lib64/perl5/PSync/API
./t/postgresql.t --recreate
./t/postgresql.t --references
/usr/fsapi/bin/fix_script.pl --fix_maillists
/usr/fsapi/bin/fix_script.pl --fix_domain
./t/postgresql.t --force_delete_user1_user2
cd /usr/fsapi/
./bin/fix_script.pl --create_application='login=app-msg@$mail_domain'
./bin/fix_script.pl --create_application='login=app-co@$mail_domain'
```

где `$mail_domain` - почтовый домен

4.3.2 Выполните на puppet master пользователем root следующие команды:

```
/root/bin/indexer_update_fs_db.sh
/root/bin/indexer_update_srch_db.sh
pcs resource cleanup nct_indexer
```

Проигнорируйте ошибку `ERROR:  function fts(text, text, text, text, text, integer, integer, text) does not exist`

***

#### Важно

Для работы почты необходимы корректные MX записи в DNS а так же рекомендуется настроить прямую и обратные записи для SMTP серверов, `myhostname` которых формируется как `mail.$mail_domain` где `$mail_domain` - почтовый домен

***

### 4.4 Проверка корректности установки

Выполните перечисленные ниже команды на одном из бэкенд-серверов (т.е. be):

```
export LINES=24
export COLUMNS=80
cd /usr/local/lib64/perl5/PSync/API
prove -p -v --timer
cd /usr/local/lib64/perl5/NCT/WebAdminAPI/
prove -p -v --timer
```

Критерием корректности установки является сообщение All tests successful после завершения работы команды.
 
### 4.5 Алгоритм установки FS на одну ноду:

Прочитайте как проводиться процедура установки на полноценный кластер и после этого воспользуйтесь краткой инструкцией:

* Имя ноды установить `fedbst`

* Настроить сеть, подключить xfs раздел для swift как в общей инструкции

* Начальный скрипт установки запустить с параметром "--single": `/mnt/scripts/install_puppetmaster.sh --single`

* [4.1.2 Установка конфига HIERA с помощью конфигуратора AUTO HIERA](#auto_hiera)

* Заполнить переменную `$fe_nodes` и секцию `Networks` корректными данными, а также `$swift_ring_device`.

* Переменные gw_nodes, lb_nodes и dd_nodes оставить пустыми

* Значение переменной vip_* сделать "%{::ipaddress}"

* Настроить dns (Все хосты для окружения резолвятся в эту, единственную ноду). 

* Скачайте и установите EWS библиотеку. [См Установка EWS](enable_ews_support)

* Запустить puppet-master `systemctl start puppetmaster`

* Запустить 2 раза `puppet agent -t`.

* Запустить скрипты:

```
cd /usr/local/lib64/perl5/PSync/API/
./t/postgresql.t --recreate
./t/postgresql.t --references
/usr/fsapi/bin/fix_script.pl --fix_maillists
/usr/fsapi/bin/fix_script.pl --fix_domain
./t/postgresql.t --force_delete_user1_user2
/root/bin/indexer_update_fs_db.sh
/root/bin/indexer_update_srch_db.sh
```

* Запустить еще раз `puppet agent -t`.

* Запустить indexer `systemctl stop indexer; sleep 5; systemctl start indexer`

* Запустить тесты:

```
export LINES=24
export COLUMNS=80
cd /usr/local/lib64/perl5/PSync/API
prove -p -v --timer
cd /usr/local/lib64/perl5/NCT/WebAdminAPI/
prove -p -v --timer
```
