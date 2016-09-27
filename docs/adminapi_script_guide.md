
# Инструкция по обслуживанию тенантов при помощи adminapi_script.pl

## Версия скрипта: *

## Содержание

- [Создание компании](#создание-компании)
- [Установка домена по умолчанию](#установка-домена-по-умолчанию)
- [Смена тарифа компании](#смена-тарифа-компании)
 - [Просмотр списка тарифов компании](#шаг-1-просмотр-списка-тарифов-компании) 
 - [Деактивация инвойса](#шаг-2-деактивация-инвойса)
 - [Создание нового инвойса с новым тарифом](#шаг-3-cоздание-нового-инвойса-с-новым-тарифом)
- [Справочная информация](#cправочная-информация)
 - [Термины](#термины)
 - [Тарифы](#тарифы)
 - [Фичи тарифов](#фичи-тарифов)
 - [Использование идентификатора при выполнении команд скрипта](#использование-идентификатора-при-выполнении-команд-скрипта)
 - [Получение помощи по скрипту](#получение-помощи-по-скрипту)

***

скрипт запускается после перехода в директорию `/usr/local/lib64/perl5/NCT/WebAdminAPI`.

***

Если для функциональности необходима привязка к корповой группе, ее можно 
реализовать перечисленными ниже способами:

- `login=test_mosreg_99@devmsk.ncloudtech.ru`
- `group_id=Gdev24hnzhgEbkI6eZF8`
- `user_id=Udev24hnzhgD34YRzGQ2`

***

## Создание компании

`./adminapi_script.pl --create_corp_user='login=project666@yandex.org&domain=yandex.org&password=12345678&tariff=corp3'`

|Обязательные параметры|Описание|Тип значения|
|---|---|---|
|`login`|Логин владельца компании|Адрес электронной почты|
|`password`|Пароль владельца компании|string|
|`domain`|При создании компании вместе с ней создаётся домен, который будет являться дефолтным для этой компании и, соответственно, для её будущих пользователей. Очевидно, что домен из параметра `login` должен совпадать со значением параметра `domain`. Значением параметра `domain` может быть любое доменное имя|string|

|Опциональные параметры|Описание|Тип значения|
|---|---|---|
|`tariff`|Тариф компании. Если этот параметр не указан, будет выведено предупреждение, а значением по умолчанию будет `corp_1`|[Возможные варианты](tariffs_info.md)|

**Вывод**:

```
$> --log=/home/maxim.maslin/log/adminapi_script.log
create_corp_user(login=project666@yandex.org&domain=yandex.org&password=12345678&tariff=corp_1)
$VAR1 = {
          'domain' => 'yandex.org',
          'login' => 'project666@yandex.org',
          'password' => '12345678',
          'tariff' => 'corp_1'
        };
```

**Внимание**: при попытке создать две компании на одном домене будет выведено сообщение об ошибке.

```
$> --log=/home/maxim.maslin/log/adminapi_script.log
create_corp_user(login=project667@yandex.org&domain=yandex.org&password=12345678&tariff=corp_1)
$VAR1 = {
          'res' => {
                   'error_code' => '144',
                   'error_msg' => 'Domain is not available at (eval 921) line 263
',
                   'ext_msg' => '',
                   'ref' => 'PSync::API::Exception',
                   'success' => 'false'
                 }
        };
```

**Внимание**: существуют компании, которые были созданы до появления обязательного параметра `domain`. Одна из таких компаний могла быть создана командой

`./adminapi_script.pl --create_corp_user='login=project667@yandex.org&password=12345678'`

Впоследствии может быть осуществлена попытка создать компанию командой

`./adminapi_script.pl --create_corp_user='login=project667@yandex.org&domain=yandex.org&password=12345678'`

В этом случае будет выведено сообщение об ошибке.

## Установка домена по умолчанию

Функциональность компаний, которые были созданы до появления обязательного параметра `domain`, может быть нарушена. Теперь в такие компании стало невозможно добавлять пользователей. Также, все операции, задействующие проверку домена, будут возвращать ошибку.

Для решения этой проблемы необходимо установить для компании домен по умолчанию. Это делается при помощи команды AdminAPI.

```
curl '{url}?cmd=set_default_domain&token={token}&domain=new.com&group_id={group_id}' 2>/dev/null | python -m json.tool

{
    "response": {
        "success": "true"
    },
    "success": "true"
}
```

**Внимание**: Значение параметра `domain` должно совпадать со значением с домена в логине пользователя.

**Внимание**: Домен, который устанавливается по умолчанию, должен быть предварительно добавлен в компанию. Это действие выполняется при помощи команды AdminAPI `add_domain` (см. adminapi.md). В ином случае будет выведено сообщение об ошибке, приведено ниже:

```
{
"error_code": "024",
"error_msg": "Permissions Denied",
"ext_msg": "(3205)",
"success": "false"
}
```

## Смена тарифа компании

### Шаг 1 Просмотр списка тарифов компании

`./adminapi_script.pl --list_group_tariffs='login=project666@yandex.org'`

|Обязательные параметры|Описание|Тип значения|
|---|---|---|
|`group_id` либо `login` либо `user_id`|Идентификатор компании либо логин любого её пользователя (можно владельца) либо идентификатор этого пользователя||

**Дополнительная информация**: [использование идентификатора при выполнении команд скрипта](#использование-идентификатора-при-выполнении-команд-скрипта)

**Вывод**:

```
$> --log=/home/maxim.maslin/log/adminapi_script.log
list_group_tariffs(login=project666@yandex.org)
$VAR1 = {
          'group_id' => 'G3211hsHOkaswSgLkNE',
          'login' => 'project666@yandex.org',
          'user_id' => 'U3211hsHOkahUbn67oG'
        };

$VAR1 = [
          {
            'id: invoice' => 'G3211hsHOkaswSgLkNE',
            'id: tariff' => 'corp_1',
            'is_active' => 1,
            'time: purchase/start/end' => '1460023491 / 1460023491 / 32503669200'
          }
        ];
```

В выводе интересует `'id: invoice'` у которого поле `'is_active'` равно `1`. Его надо деактивировать.

### Шаг 2 Деактивация инвойса

`./adminapi_script.pl --deactivate_group_tariffs='invoice_id=G3211hsHOkaswSgLkNE'`

|Обязательные параметры|Описание|Тип значения|
|---|---|---|
|`invoice_id`|Идентификатор инвойса, предназначенного для удаления|string|

**Вывод**:

```
$> --log=/home/maxim.maslin/log/adminapi_script.log
deactivate_group_tariffs(invoice_id=G3211hsHOkaswSgLkNE)
invoice_id: G3211hsHOkaswSgLkNE => group_id: G3211hsHOkaswSgLkNE
$VAR1 = {
          'group_id' => 'G3211hsHOkaswSgLkNE',
          'now_active_count' => 0,
          'now_count' => 1,
          'was_active_count' => 1,
          'was_count' => 1
        };
```

### Шаг 3 Создание нового инвойса с новым тарифом


`./adminapi_script.pl --add_and_activate_corp_tariffs='login=project666@yandex.org&tariffs=corp_3'`

|Обязательные параметры|Описание|Тип значения|
|---|---|---|
|`group_id` либо `login` либо `user_id`|Идентификатор компании либо логин любого её пользователя (можно владельца) либо идентификатор этого пользователя||
|`tariffs`|Наименование [тарифа](#тарифы)||

**Дополнительная информация**: [использование идентификатора при выполнении команд скрипта](#использование-идентификатора-при-выполнении-команд-скрипта)

**Вывод**:

```
$> --log=/home/maxim.maslin/log/adminapi_script.log
add_and_activate_corp_tariffs(login=project666@yandex.org&tariffs=MosReg)
$VAR1 = {
          'added_invoice' => [
                               {
                                 'end_time' => '32503669200',
                                 'group_id' => 'G3211hsHOkaswSgLkNE',
                                 'invoice_id' => 'I3211hsHUl7BWdCbcdM',
                                 'is_active' => 1,
                                 'purchase_time' => '1460028552',
                                 'start_time' => '1460028552',
                                 'tariff_id' => 'MosReg'
                               }
                             ],
          'group_id' => 'G3211hsHOkaswSgLkNE',
          'login' => 'project666@yandex.org',
          'now_active_count' => 1,
          'now_count' => 2,
          'user_id' => 'U3211hsHOkahUbn67oG',
          'was_active_count' => 0,
          'was_count' => 1
        };
```

**Внимание**: при попытке указать несуществующее значение параметра `tariffs` будет выведено сообщение об ошибке.

```
./adminapi_script.pl --add_and_activate_corp_tariffs='login=project666@yandex.org&tariffs=corp_666'

$VAR1 = {
          'error_code' => '302',
          'error_msg' => 'OPERATION_FAILED at (eval 941) line 3897
',
          'ext_msg' => 'tariff is not available (\'corp_666\')',
          'ref' => 'PSync::API::Exception',
          'success' => 'false'
        };
```

## Справочная информация

### Термины

|Термин|Определение|
|---|---|
|[Фича](#фичи-тарифов)|Функциональность, которая может быть предоставлена компании или её пользователям|
|[Тариф](#тарифы)|Набор фич, который может быть прикреплён к компании. К компании может быть прикреплено более одного тарифа. После того как тариф прикреплён к компании, формируется инвойс|
|[Активация инвойса](#активация-инвойса)|Выделение компании фич, содержащихся в прикреплённом к ней тарифе. После того как фичи выделены, администратор может раздавать их пользователям компании|

###  Тарифы. Операции с тарифами

#### Тарифы

| Идентификатор тарифа  | Описание  | Фичи тарифа  | Корпоративный  |
|---|---|---|---|
| corp_1  | Default (Basic) corp tariff. 1 TB. 500 users|"1":1000000000000, "2":500| Да|
| corp_2  | Advanced corp tariff. 2 TB. 1000 users  | "1":2000000000000, "2":1000  |  Да |
| corp_3  | Professional corp tariff. 10 TB. 5000 users  | "1":10000000000000, "2":5000  |  Да |
| corp_4  | Unlimited corp tariff. 1 EB. 1 billion users  | "1":1000000000000000000,"2":1000000000  |  Да |
| MosReg  | MosReg corp tariff. Main  | "5":10000, "2":10000 | Да |
| MosReg2  | MosReg corp tariff. Addition  | "6":100  | Да |
| user_1  | Default user tariff  | "1":5368709120  | Нет |

**Примечание**: каждый тариф имеет набор тех или иных фич, т.е. параметров. Например, в приведённой выше таблице фичи тарифа corp_1 обозначены как "1":1000000000000, "2":500.  Это означает, что компания, которая приобрела этот тариф, может создать до 500 пользователей, и "раздать" им до 1000000000000 байт дискового пространства. Более подробная информация о фичах приведена ниже.

#### Фичи тарифов

| ID фичи  | Описание  |
|---|---|
| 1  | Дисковое пространство в байтах  |
| 2  | Количество пользователей   |
| 5  | Пакет 2 GB на пользователя. Пользователю не более одного |
| 6  | Пакет 4 GB на пользователя. Пользователю не более одного  |

### Использование идентификатора при выполнении команд скрипта

Рассмотрим команды

- [`list_group_tariffs`](#просмотр-списка-тарифов-компании)
- [`add_and_activate_corp_tariffs`](#создание-нового-инвойса-с-новым-тарифом)

В этих командах для идентификации компании пользователь может использовать один из перечисленных ниже параметров: 

- логин пользователя, например, `login=project666@yandex.org`
- идентификатор пользователя, например, `user_id=U3211hsHOkahUbn67oG`
- идентификатор компании, например, `group_id=G3211hsHOkaswSgLkNE`

Соответственно, команду [`list_group_tariffs`](#просмотр-списка-тарифов-компании) можно вызвать несколькими способами:

- `./adminapi_script.pl --list_group_tariffs='login=project666@yandex.org'`
- `./adminapi_script.pl --list_group_tariffs='user_id=U3211hsHOkahUbn67oG'`
- `./adminapi_script.pl --list_group_tariffs='group_id=G3211hsHOkaswSgLkNE'`
 
Результат будет одинаковым: 

```
$> --log=/home/undefer/log/adminapi_script.log
list_group_tariffs(login=test_mosreg_99@devmsk.ncloudtech.ru)
$VAR1 = {
          'group_id' => 'Gdev24S3Hqud5avPddW',
          'login' => 'test_mosreg_99@devmsk.ncloudtech.ru',
          'user_id' => 'Udev24S3HqucRW9oiok'
        };

$VAR1 = [
          {
            'id: invoice' => 'Gdev24S3Hqud5avPddW',
            'id: tariff' => 'MosReg',
            'is_active' => 1,
            'time: purchase/start/end' => '1448456662 / 1448456662 / 32503669200'
          }
        ];
```

То же самое верно и в отношении команды [`add_and_activate_corp_tariffs`](#создание-нового-инвойса-с-новым-тарифом):

- `./adminapi_script.pl --add_and_activate_corp_tariffs='login=project666@yandex.org&tariffs=MosReg'`
- `./adminapi_script.pl --add_and_activate_corp_tariffs='user_id=U3211hsHOkahUbn67oG&tariffs=MosReg'`
- `./adminapi_script.pl --add_and_activate_corp_tariffs='group_id=G3211hsHOkaswSgLkNE&tariffs=MosReg'`

Результат будет одинаковым:

```
$> --log=/home/maxim.maslin/log/adminapi_script.log
add_and_activate_corp_tariffs(group_id=G3211hsHOkaswSgLkNE&tariffs=MosReg)
$VAR1 = {
          'added_invoice' => [
                               {
                                 'end_time' => '32503669200',
                                 'group_id' => 'G3211hsHOkaswSgLkNE',
                                 'invoice_id' => 'I3211hsH3e1G3m5zvkR',
                                 'is_active' => 1,
                                 'purchase_time' => '1460036010',
                                 'start_time' => '1460036010',
                                 'tariff_id' => 'MosReg'
                               }
                             ],
          'group_id' => 'G3211hsHOkaswSgLkNE',
          'now_active_count' => 7,
          'now_count' => 8,
          'was_active_count' => 6,
          'was_count' => 7
        };
```

### Получение помощи по скрипту

Помощь по скрипту вызывается командами

- `./adminapi_script.pl -h`
- `./adminapi_script.pl -h -sample`

Ниже приведен вывод команды `./adminapi_script.pl -h -sample`

```
NCT WebAdminAPI fixing script (v *):

./adminapi_script.pl --< parameter > --< parameter >...

Parameters:
        --help  - this page
             possible with help:
                --samples   - extend samples help
        --log=s - tee STDERR to log     (/root/log/adminapi_script.log)

        --activate_invoice=?                 - activate_invoice
        --cron1                              - cron operations: actualize_features etc

        see './adminapi_script.pl -h -sample' to understand format of below commands
        --create_corp_user=?                 - create corp user
        --add_and_activate_corp_tariffs=?    - add and activate corp tariffs to user or group.
                                        Be careful! Some tariffs are not combined with each other.
        --list_group_tariffs=?               - list group tariffs
        --deactivate_group_tariffs=?         - revoke tariffs from corp user or group

        --syslog_facility=?                  - syslog facility (current: local3)
        --add_corp_alias=?                   - add host, realm and alias to corp AD group
        --delete_corp_alias=?                - delete alias record from corp AD group
        --list_corp_aliases=?                - list all aliases for corp AD group
        --get_domain_info=?                  - get information about domain
        --get_corp_accounts                  - get all corporate accounts on installation

        --delete_public_domains              - deletion of the public domains from vmail.domain and fs.company_domains [AD-119]

        --sync_vmail_domains_into_fs         - [AD-104] synchronize data from vmail::domain into fs::company_domains

        --fix_resource_empty_quota=?         - [AD-138] set quota for resource with empty quota to 1GB for resource_id
        --fix_all_resources_empty_quota      - [AD-138] set quota for resources with empty quota to 1GB

Samples:
 ./adminapi_script.pl --help --samples
          --samples - use this option with -h to see other -samples


Samples (2):
To see your choosed parameters:
./adminapi_script.pl -h

To activate invoice
./adminapi_script.pl --activate_invoice=Idev24hjP7N232aBaV9D

Sample for creation of corp group and modification of their tariffs for set_disk_quota functionality
./adminapi_script.pl --create_corp_user='domain=devmsk.ncloudtech.ru&login=test_mosreg_99@devmsk.ncloudtech.ru&password=12345678&tariff=corp_1'
./adminapi_script.pl --add_and_activate_corp_tariffs='login=test_mosreg_99@devmsk.ncloudtech.ru&tariffs=corp1'
./adminapi_script.pl --list_group_tariffs='login=test_mosreg_99@devmsk.ncloudtech.ru&tariffs=corp1'
./adminapi_script.pl --deactivate_group_tariffs='invoice_id=Idev24hp1K9n68Q1Jatc'

Sample for creation of corp group and modification of their tariffs for add_features_to_user functionality
./adminapi_script.pl --create_corp_user='domain=devmsk.ncloudtech.ru&login=test_mosreg_99@devmsk.ncloudtech.ru&password=12345678&tariff=MosReg'
./adminapi_script.pl --create_corp_user='domain=devmsk.ncloudtech.ru&login=test_mosreg_99@devmsk.ncloudtech.ru&password=12345678&tariff=MosReg&ad_url=ldap://10.110.1.2'
./adminapi_script.pl --add_and_activate_corp_tariffs='login=test_mosreg_99@devmsk.ncloudtech.ru&tariffs=MosReg,MosReg2'
./adminapi_script.pl --list_group_tariffs='login=test_mosreg_99@devmsk.ncloudtech.ru'
./adminapi_script.pl --deactivate_group_tariffs='invoice_id=Gdev24hnzhgEbkI6eZF8'
--to deactivate only one tariff by invoice_id and tariffs ids
./adminapi_script.pl --deactivate_group_tariffs='invoice_id=Idev24hnzhlofYrTgqTK&tariffs=MosReg2'
./adminapi_script.pl --list_group_tariffs='group_id=Gdev24hnzhgEbkI6eZF8'
./adminapi_script.pl --get_corp_accounts

Sample for creation deletion and get list for aliases to AD realms
./adminapi_script.pl --add_corp_alias='login=test_mosreg_99@devmsk.ncloudtech.ru&host=ncloudtech.ru&realm=CO.COM&alias=ncloudtech.ru&base_dn=DC=CO,DC=COM'
./adminapi_script.pl --delete_corp_alias='login=test_mosreg_99@devmsk.ncloudtech.ru&host=ncloudtech.ru&realm=CO.COM&alias=CO'
./adminapi_script.pl --list_corp_aliases='login=test_mosreg_99@devmsk.ncloudtech.ru&host=ncloudtech.ru&realm=CO.COM'
./adminapi_script.pl --list_corp_aliases='login=test_mosreg_99@devmsk.ncloudtech.ru&host=ncloudtech.ru'
./adminapi_script.pl --list_corp_aliases='login=test_mosreg_99@devmsk.ncloudtech.ru'

Sample for domain information
./adminapi_script.pl --get_domain_info='domain=domain5.com'

Sample for companies corporate accounts information
./adminapi_script.pl --get_corp_accounts

--You may use next beside of 'to_login=test_mosreg_99@devmsk.ncloudtech.ru':
   'group_id=Gdev24hnzhgEbkI6eZF8'
or 'user_id=Udev24hnzhgD34YRzGQ2'

Syncronize data from vmail::domain into fs::company_domains
./adminapi_script.pl --sync_vmail_domains_into_fs
```