## 6. Начало администрирования МойОфис® Почта

### 6.1 Общая информация

Администрирование МойОфис® Почта следует начать с создания корпоративного пользователя при помощи запуска скрипта `adminapi_script.pl`. Пример:

    ./adminapi_script.pl --create_corp_user='login=test@test.ru&password=Test_123&tariff=MosReg'

В результате запуска скрипта будет создана корпоративная группа, на которую автоматически будет добавлен тариф, указанный в параметре tariff (в нашем случае это “Mosreg”).

Настоятельно рекомендуется использовать параметр tariff. В случае отсутствия этого параметра на группу автоматически будет добавлен тариф default, который впоследствии придётся удалить


Чтобы получить подсказку по использованию скрипта, запустите его с ключом --help или -h:

    ./adminapi_script.pl --help

Чтобы включить в подсказку примеры запуска скрипта, используйте в сочетании с ключом `--help`/`-h` ключ `--samples`:
    
   ` ./adminapi_script.pl --help --samples`
   
### 6.2 Добавление дополнительных тарифов на группу

Чтобы добавить дополнительные тарифы на группу, используйте команду `add_and_activate_corp_tariffs`:

`./adminapi_script.pl --add_and_activate_corp_tariffs=<параметры скрипта>`

Параметрами скрипта являются

- идентификатор группы, на которую должен быть назначен тариф.

- Тариф, который должен быть назначен группе.

Пример:

    ./adminapi_script.pl --add_and_activate_corp_tariffs='group_id=Gdev24hnzhgEbkI6eZF8&tariffs=MosReg'

Чтобы указать на группу, которой должен быть назначен тариф, можно, помимо `group_id`, использовать логин администатора этой группы либо его идентификатор. Примеры:

```
./adminapi_script.pl --add_and_activate_corp_tariffs='user_id=Udev24hnzhgD34YRzGQ2&tariffs=MosReg'

./adminapi_script.pl --add_and_activate_corp_tariffs='login=test_mosreg_99@devmsk.ncloudtech.ru&tariffs=MosReg'
```
### Замеченные недостатки
**Внимание!**
```
В следующей версии скрипта будут выложены правки, чтобы он выполнял необходимую актуализацию тарифов после операций добавления компании или модификации тарифов компании.
Пока не были внесены эти правки, нужно после операций
     --deactivate_group_tariffs, --add_and_activate_corp_tariffs, --create_corp_user
Обязательно делать операцию
    ./adminapi_script.pl --cron1
```

### 6.2 Получение информации о системе по SNMP

В релизе версии 3 и выше появилась возможность собирать данные о системе по SNMP. Метрики с бэкенд серверов(роли be) можно получить обратившись к службе SNMP на них используя OID .1.3.6.1.4.1.46958
База управляющей информации (MIB) находится в файле NCT-MIB.txt (прилагается к документации)

### 6.3 Бэкапирование FS

[Инструкция по бэкапированию](backup)

### 6.4 Работа с Microsoft Active directory

В релизе версии 3 и выше появилась возможность использовать MS Active directory в качестве бэкэнда аутентификации. 

Для этого необходимо создать привязку корп группы к AD. На первом этапе создается корп пользователь как в пункте 6.1 только с указанием на ldap каталог

```
./adminapi_script.pl --create_corp_user='login=test@test.ru&password=Test_123&tariff=MosReg&ad_url=ldap://ldap_host'
```

Далее создается привязка созданной корпоративной группы к AD

```
/adminapi_script.pl --add_corp_alias='login=test@test.ru&host=ad.domain.com&realm=REALM.RU&alias=ad.domain.com&base_dn=DC=REALM,DC=RU'
```

host = alias - почтовый домен который обслуживате компания
realm - kerberos реалм

Для работы с ad необходимо заполнить переменную $krb_realms в globals.pp корректным(и) реалмами с которыми предстои работать. Переменная предаставляет собой хэш вида

```
 {realm1 => [kdc1, kdc2, kdc3],
  realm2 => [kdc1, kdc2 ...] 
  ...}
```

где kdcN - керберос сервера в виде либо IP либо DNS имен.

**ВАЖНО**
Сервера рекомендуется указывать используя DNS имена, а даже если через IP адрес, то в hosts или DNS должна быть прописана связка ip -> hostname для KDC. Дело в том, что ключ TGS(Ticket Granting Service) Выдается для определенного сервиса с фиксированным именем в DNS и если име не разрезолвится, то получится ошибка вида: 

```
Unspecified GSS failure.  Minor code may provide more information (Server not found in Kerberos database)
```

Проверить работоспособность связки: 

```
kinit ad_user@REALM.RU  (получаем TGT тикет для пользователя ad_user)
ldapsearch -R REALM.RU -H 'ldap://ldap_host' -b 'REALM,DC=RU' -LLL '....' (ldap запрос используя kerberos ticket.)
```
Если предполагается использовать существующие в ActiveDirectory учётные записи, воспользуйтесь разделом [Миграция одного из обслуживаемых Exchange-сервером почтовых доменов на MyOffice](take_domain_from_exchange).

### 6.5 Изменение брендинга веб-статики

В релизе версии 3 и выше появилась возможность изменения настроек страниц web-статики. Для этого нужно установить соответствующие значения для переменных (роль fe):

```
fsapi::webadmin_settings_params
fsapi::webadmin_settings_params
fsapi::webcalendar_settings_params
fsapi::webcontacts_settings_params
```

Переменные являются хэшами вида param_name => value. Подробнее о возможных пареметрах см:

* для mail [web-mail](web-static/mail.settings.environment.js)
* для admin [web-admin](web-static/admin.settings.environment.js)
* для calendar [web-calendar](web-static/calendar.settings.environment.js)
* для contacts [web-contacts](web-static/contacts.settings.environment.js)

#### Пример конфигурации.

Eсли в приложении webmail требуется в списке сервисов оставить только mail,calendar,contacts то необходимо определить переменную:

```
fsapi::webmail_settings_params:
  'webmail.settings.services':
    - 'mail'
    - 'contacts'
    - 'calendar'
```

Для изменения логотипа приложения, /img/branding/[currentBranding]/logo.png

```
fsapi::webmail_settings_params:
  'webmail.settings.currentBranding': 'collabio'
```

### 6.6 Замена вышедшего из строя узла системы

Предполагается что до выхода узла из строя все было в порядке (Все кластерные ресурсы функционировали без ошибок). Так же предполагается что puppet-master из строя не вышел.

* Создается новая нода с тем же именем, что и вышедшая из строя.
* на puppet-master: `puppet cert clean fqdn_of_failed_node`

Для ролей gw, lb, be, fe, dd, ds :

* запускается агент: `puppet agent -t`
* Подписывается сертификат: `puppet cert sign fqdn_of_failed_node`
* запускается агент еще раз: `puppet agent -t`
* на этом восстановление ноды можно считать законченным

* Для ролей db, ms, st(кроме нод которые использовались для создания postgres-master - db1,ms1):

Действия аналогичные приведенным выше. Ноды типа st должны иметь такую же разбивку по дискам как и вышедшая из строя.

После окончания работы агента, необходимо проверить что:
  для нод типа db,ms:
    кластер postgres работает в номальном режиме. puppet agent в процессе настройки попробует скопировать данные с postgres-master и ввести ноду в кластер. Если это не удастся, необходимо сделать это вручную, запустив скрипт `/root/bin/restore_script.sh`
  для нод типа st:
     `systemctl --all | grep swift` показывает что все процессы кроме аудиторов запущены.

* Для нод ms1, db1:

Все как для других нод, но после завершения работы puppet-agent-а надо запутить `/root/bin/restore_script.sh` в любом случае.

### 6.7 Проверка состояния кластера PostgreSQL

Для просмотра состояния клаcтера PostGreSQL можно воспользоваться командой `crm_mon -Afr -1`, в выводе следует обратить внимание на раздел `Node Attributes`
Ниже пример интересующего нас фрагмена вывода:

```
Node Attributes:
* Node db1.box.nct:
    + master-pgsql_res                  : 100       
    + pgsql_res-data-status             : STREAMING|SYNC
    + pgsql_res-status                  : HS:sync   
* Node db2.box.nct:
    + master-pgsql_res                  : -INFINITY 
    + pgsql_res-data-status             : DISCONNECT
    + pgsql_res-status                  : STOP      
* Node db3.box.nct:
    + master-pgsql_res                  : 1000      
    + pgsql_res-data-status             : LATEST    
    + pgsql_res-master-baseline         : 000000000A000090
    + pgsql_res-status                  : PRI       
* Node db4.box.nct:
    + master-pgsql_res                  : -INFINITY
    + pgsql_res-data-status             : STREAMING|ASYNC
    + pgsql_res-status                  : HS:async   
```

В данном примере узел db3 является master-сервером, узлы db1 и db4 - являются активными slave-серверами. Причем db1 использует синхронную репликацию и станет следующим мастером в случае отказа.
Узел db2 выключен либо отказал.

При проверке состояния кластера, необходимо чтобы slave-ноды находились в режиме HS:sync либо HS:async.
Подробную информацию можно получить в сети интернет по адресу http://clusterlabs.org/wiki/PgSQL_Replicated_Cluster

### 6.8 Статистика

На всех нодах роли BE добавьте в крон следующие строки:

```
18 3 * * * cd /usr/fsapi && cronlock ./bin/fix_script.pl --log=fix_script.log --send_statistics=<ваш e-mail>
36 4 * * * cd /usr/fsapi && cronlock ./bin/fix_script.pl --log=fix_script.log --send_statistics_ad43=<ваш e-mail>
```

```
        --send_statistics                    - collecting statistics information and sending mail (AD-27)
        --send_statistics_ad43               - collecting statistics information and sending mail (AD-43)
```
