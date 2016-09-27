## Установка сервиса Диалоги

### Совместимость

Для установки сервиса МойОфис Диалоги требуется установленный и настроенный сервис МойОфис Частное Облако версии не ниже Elder Flolwer 2016.02

### Настройка DNS

Требуется создать домен dialogues, который должен разрешаться в тот же адрес, что и fsapi, т.е на примере домена `domain.tld`:

```
$ORIGIN domain.tld
dialogues	IN 	CNAME fsapi
```

***

*Важно!*

Домен dialogues.domain.tld должен резолвиться в том числе с серверов, на которых установлен сервис Диалоги.

А также должны работать tcp-соединения на порты 80 и 443 по этому адресу.

***

### Создание базы данных

Для создания БД сервиса Диалоги требуется подключиться к серверу, являющегося PostgreSQL master и выполнить следующие команды:

```
# psql -U postgres
postgres=#  CREATE USER msg WITH password 'msg';
postgres=# CREATE DATABASE msg WITH OWNER msg;
postgres=# \c msg
msg=# CREATE SCHEMA catalog;
msg=# CREATE SCHEMA extensions;
msg=# CREATE EXTENSION pgcrypto WITH SCHEMA extensions;
msg=# CREATE EXTENSION ltree WITH SCHEMA extensions;
msg=# CREATE SCHEMA queue;
msg=# GRANT ALL PRIVILEGES ON SCHEMA catalog TO msg;
msg=# GRANT ALL PRIVILEGES ON SCHEMA queue TO msg;
msg=# GRANT ALL PRIVILEGES ON SCHEMA extensions TO msg;
```

При создании пользователя и базы можно указать и другие значения в качестве имени пользователя, пароля и названия БД. В этом случае также надо обратить внимание на примечание к следующему пункту.

### Добавление данных сервиса Диалоги

Создать rpm-репозиторий из пакетов сервиса Диалоги, находящихся в архиве `messenger-el7.repo.tar.gz`,

сделать его доступным по http и добавить его на сервера роли fe.

Либо скопировать пакеты в имеющийся rpm-репозиторий.

Примечание: если нет выделенных серверов с ролью fe, действия выполняются на серверах роли be.

### Конфигурация puppet

Распаковать содержимое архива с данными для puppet:

```
rm -rf /etc/puppet/environments/production/modules/dialogues
tar xf dialogues.tar.gz -C /etc/puppet/environments/production/modules
```

Внести в файл `/etc/puppet/hieracustom/common.yaml` следующие значения:

```
fsapi::install_dlg: true
dialogues::domain: "dialogues%{::vhost_suffix}"
dialogues::repo_baseurl: "http://EDITME_PATH_TO_DILOGUES_REPO/"
dialogues::cardapi_postgres_url: "postgres://%{::fs_db_user}:%{::fs_db_pwd}@%{::vip_db}/%{::fs_db_name}?sslmode=disable"
dialogues::schema: https
dialogues::gateway_ssl_cert: "/etc/nginx/fsapi.%{::interdomain}.crt"
dialogues::gateway_ssl_key: "/etc/nginx/fsapi.%{::interdomain}.key"
dialogues::backend_ensure: "2.0.5-44.el7.centos"
nginx::string_mappings:
  header_authorization_value:
    ensure: absent
    string: 'empty'
    mappings:
      default: ''
```

В параметре `dialogues::repo_baseurl` указать актуальный путь к репозитоию с rpm-пакетами сервиса Диалоги

Если при создании базы в качестве имени пользователя, пароля и названия БД использовались значения отличные от указанных в документации, надо дополнительно добавить в файл `/etc/puppet/hieracustom/common.yaml` следующий пареметр с использованными параметрами подключения к БД

```
dialogues::postgres_url: "postgres://msg:msg@%{::vip_db}/msg?sslmode=disable"
```

### Cоздать пользователей.
 
На одной ноде роли be выполнить.

Домен myoffice.ru заменить на актуальный для данной инсталляции.

```
source /etc/sysconfig/messenger
cd /usr/fsapi/
./bin/fix_script.pl --create_application='login=app-msg@myoffice.ru'
./bin/fix_script.pl --create_application='login=app-co@myoffice.ru'
systemctl restart capella
systemctl restart skat
systemctl restart baham
```

### Запуск puppet agent

На серверах роли fe выполниьть команду `puppet agent -t`
