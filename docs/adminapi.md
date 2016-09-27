# AdminAPI reference

**Рассматриваемая версия AdminAPI**: 1.34

## Содержание

- [Термины](#термины)
- [Общая информация](#общая-информация)
- [О запросах к AdminAPI](#о-запросах-к-adminapi)
- [Начало работы](#начало-работы)
  - [Общая информация](#общая-информация)
  - [Создание администратора](#создание-администратора)
  - [`auth`](#auth)
- [Операции над пользователями](#операции-над-пользователями)
  - [Общая информация об операциях над пользователями](#общая-информация-об-операциях-над-пользователями) 
  - [Управление пользователями](#управление-пользователями)
    - [`add_user`](#add_user)
    - [`update_user`](#update_user)
    - [`set_user_status`](#set_user_status)
    - [`add_user_to_group`](#add_user_to_group)
    - [`revoke_user_from_group`](#revoke_user_from_group)
    - [`set_group_user_roles`](#set_group_user_roles)
    - [`set_disk_quota`](#set_disk_quota)
    - [`add_features_to_user`](#add_features_to_user)
    - [`revoke_features_from_user`](#revoke_features_from_user)
    - [`reset_password`](#reset_password)
  - [Вывод данных о пользователе](#вывод-данных-о-пользователе)
    - [`corp_groups`](#corp_groups)
    - [`get_personal_data`](#get_personal_data)
- [Операции над группами](#операции-над-группами)
  - [Общая информация об операциях над группами](#общая-информация-об-операциях-над-группами)
  - [Управление группами](#управление-группами)
    - [`add_group`](#add_group)
    - [`set_group_status`](#set_group_status)
    - [`update_group`](#update_group)
  - [Вывод данных о группе](#вывод-данных-о-группе)
    - [`list_group_features`](#list_group_features)
    - [`corp_groups`](#corp_groups)
    - [`list_group_tariffs`](#list_group_tariffs)
    - [`list_users`](#list_users)
- [Операции на списком рассылок](#термины)
  - [Общая информация об операциях над списками рассылок](#общая-информация-об-операциях-над-списками-рассылок)
  - [Управление списком рассылки](#управление-списком-рассылки)
    - [`add_maillist`](#add_maillist)
    - [`update_maillist`](#update_maillist)
    - [`delete_maillist`](#delete_maillist)
  - [`Вывод данных о списках рассылки`](#вывод-данных-о-списках-рассылки)
    - [`list_maillists`](#list_maillists)
    - [`get_maillist`](#get_maillist)
- [Операции над доменами](#операции-над-доменами)
  - [Общая информация об операциях над доменами](#общая-информация-об-операциях-над-доменами)
  - [Управление доменами](#управление-доменами)
    - [`add_domain`](#add_domain)
    - [`delete_domain`](#delete_domain)
  - [Вывод информации о доменах](#вывод-информации-о-доменах)
    - [`list_domains`](#list_domains)
- [Операции над псевдопользователями](#операции-над-псевдопользователями)
  - [Общая информация об операциях над псевдопользователями](#общая-информация-об-операциях-над-псевдопользователями)
  - [Управление псевдопользователями](#управление-псевдопользователями)
    - [`add_resource`](#add_resource)
    - [`update_resource`](#update_resource)
    - [`set_resource_status`](#set_resource_status)
  - [Вывод данных псевдопользователей](#вывод-данных-псевдопользователей)
    - [`list_resources`](#list_resources)
- [Восстановление файлов](#восстановление-файлов)
  - [Общая информация об операциях по восстановлению файлов](#общая-информация-об-операциях-по-восстановлению-файлов)
  - [`search_objects_to_restore`](#search_objects_to_restore)
  - [`restore_file`](#restore_file)

## Термины

|Термин   | Определение  |
|---|---|
| Компания, корпорация, корповая группа  |  Компания, управление которой выполняется через AdminAPI. Компания также является группой. Такая группа отмечена флагом is_corp |
| Корпоративные пользователи, корповые пользователи  |  Пользователи этой компании |
|  Админ корповой группы | Пользователь компании, который имеет права управлять ею через AdminAPI  |
|  Фичи |  Возможности, которые можно выделить компании или её пользователям |
| Корпоративные тарифы  | Тарифы, содержащие фичи, которые будут выделены компании. Компании может быть назначено более одного тарифа  |
|  Покупка тарифа | Прикрепление к компании одного или более тарифов. При этом формируется счёт (инвойс)  |
| Активация инвойса  |  При активации инвойса фичи, содержащиеся в тарифах инвойса, выделяются компании. Далее администратор может раздать эти фичи пользователям компании |

## Общая информация

AdminAPI реализует возможность управлять компаниями. Эта возможность предоставляется пользователю, который является их администратором. Раздел [Начало работы](#начало-работы) содержит инструкции для быстрого старта работы с AdminAPI.

Отметим, что некоторые из команд могут быть выполнены только в отношении определённых тарифов.
Так, команда `add_features_to_user` позволяет добавить на пользователя фичу только из тарифов типа "MosReg", включающих пакеты дискового пространства фиксированного размера (2 либо 4 Гбайт). Иными словами, при помощи `add_features_to_user` пользователю можно предоставить либо 2 Гбайт (однократно), либо 4 Гбайт (более одного раза). Cоответственно, команда `revoke_features_from_user` позволяет аннулировать добавленную при помощи `add_features_to_user` фичу.

В свою очередь, команда `set_disk_quota` позволяет установить квоту произвольного размера на дисковое пространство (в байтах); использовать её можно только в отношении тарифов типа "default".


## О запросах к AdminAPI

- Метод: GET
- Хост, используемый для примеров: `https://admin-42-1.myoffice.ru/api/`
- Префикс команды: ?cmd
- Пример запроса: `https://admin-42-1.myoffice.ru/api/?cmd=auth&login=adminka@42- 1.myoffice.ru&password=123456789`
- Формат ответа: JSON

## Начало работы
### Общая информация

Рекомендуемым способом получить возможность делать запросы к AdminAPI является обращение в службу эксплуатации с просьбой завести корпоративного пользователя. Служба эксплуатации выполняет это действие при помощи скрипта `adminapi_script.pl` (называемого также "фикс-скрипт", подробнее см. https://confluence.nct:8444/display/DOC/Admin+API+Documentation). После того как реквизиты (логин, пароль) корпоративного пользователя предоставлены, остаётся лишь получить токен авторизации (см. Получение токена).

Альтернативой является cамостоятельное [создание администратора](#создание-администратора) при помощи команды `add_login`. Этот способ не рекомендуется, поскольку набор доступных запросов в отношении созданной группы будет ограниченным. В частности, добавлять в группу возможно только тарифы типа "default".

### Создание администратора

Чтобы иметь возможность создать администратора корпоративной группы, необходимо предварительно сгенерировать промокод `IS_CORP` с типом `2`. Данное действие выполняется при помощи скрипта `generate_promo_codes.pl`. Вызов справки по использованию скрипта: `./PSync/API/generate_promo_codes.pl -h` 

Напомним, что самостоятельно создавать администратора [не рекомендуется](#общая-информация).

**Пример запроса, ответа**

```
curl -k 'add_login&email=test123@42-1.myoffice.ru&is_corp=1&login=test1-vige7f8c743a32c4@email.tests&password=test999test&personal={"first_name":"Тест", "middle_name":"Тестович","last_name":"Тестов","lang":"ru"}&promocode=dgsvwnhqq9huwduf' | python -m json.tool

{  
  "response":{  
    "id":"Udev24hjaR6tzChSsKr8",
    "root":"Odev24hjaR6tARXfSasU",
    "success":"true"
  },
  "success":"true"
}
```

**Примечание**. Параметр `is_corp=1` указывает, что в запросе передаётся промокод IS_CORP с типом 2.

### `auth`

**Действие**: получение OAuth-токена для авторизации. Токен используется в большинстве команд AdminAPI.

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=auth&login=tw.ad@42-2-t23.myoffice.ru&password=12345678' | python -m json.tool

{
    "response": {
        "client_id": "8BCF9D600AE611E696D2C7B88C04CD5C",
        "id": "U4221TsgdGrWQh0hW1",
        "success": "true",
        "token": "04221808aa8c87e3d1e34119b1655229560eb"
    },
    "success": "true"
}
```
---

## Операции над пользователями

### Общая информация об операциях над пользователями

Пользователь AdminAPI может выполнять следующие действия в отношении пользователей:

- добавлять пользователей: `add_user`
- изменять параметры пользователя: `update_user`
- блокировать/активировать/удалять пользователя: `set_user_status`
- добавлять пользователя в группу: `add_user_to_group`
- удалять пользователя из группы: `revoke_user_from_group`
- изменять роль пользователя в компании: `set_group_user_roles`
- выделять дисковое пространство (произвольного размера) пользователю: `set_disk_quota`
- добавлять фичи пользователю (т.е. дисковое пространство фиксированного размера): `add_features_to_user`
- аннулировать фичи у пользователя: `revoke_features_from_user`
- выполнять сброс пароля пользователя: `reset_password`
- получать данные о группах, в которых состоит пользователь: `corp_groups`
- получать информацию о компаниях/группах, в которых состоит пользователь: `get_personal_data`
- выводить список пользователей: `list_users`

### Управление пользователями

#### `add_user`

**Действие**: создание нового пользователя. Отметим, что, несмотря на то что компания является группой, создать пользователя командой [`add_user_to_group`](#add_user_to_group) невозможно.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `email` | Email нового пользователя | email-адрес | + |
| `group_id` | Идентификатор группы | Строка | + |
| `login` | Имя учётной записи пользователя | email-адрес | + |
| `password` | Пароль пользователя | Строка | + |
| `personal` | Данные пользователя | JSON-массив. Включает следующие поля: `first_name`: имя, `middle_name`: отчество, `last_name`: фамилия, `recovery_email`: e-mail для восстановления пароля, `lang`: язык | - |
| `quota` | Объем дискового пространства для пользователя | Целое число | - |

**Примечание к параметру `email`**: домен поля email должен совпадать с одним из доменов компании. При попытке добавить пользователя с доменом в поле email, который не совпадает ни с одним из доменов компании, будет выдана ошибка "Domain of email must be in domains of corporation". Для добавления нового домена в систему необходимо выполнить команду [`add_domain`](#add-domain).

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_user&email=newuser@4042-2-t23.myoffice.ru&login=newuser@4042-2-t23.myoffice.ru&password=12345678&group_id=G4221TsgdGteeUWhcj&personal=%7B%22first_name%22%3A%22John%22%2C%22last_name%22%3A%22Doe%22%2C%22middle_name%22%3A%22%22%2C%22recovery_email%22%3A%22email%40mail.ru%22%7D&lang=ru-RU&token=04221f50d67998bcc24c256acfff898ed251e&quota=100000000000000' | python -m json.tool

{  
   "response":{  
      "is_resource":0,
      "is_user":1,
      "root_id":"O4222huorJAlRp1DUZX",
      "success":"true",
      "user_id":"U4222huorJAkSDz90GZ"
   },
   "success":"true"
}
```

**Попытка выполнить команду `add_user` с превышением дисковой квоты**:

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_user&email=zab%4042-2-t23.myoffice.ru&login=zab%4042-2-t23.myoffice.ru&password=12345678&group_id=G4221TsgdGteeUWhcj&personal=%7B%22first_name%22%3A%22Zap%22%2C%22last_name%22%3A%22Brannigan%22%2C%22middle_name%22%3A%22%22%2C%22recovery_email%22%3A%22a%40a.ru%22%7D&lang=ru-RU&token=04221f50d67998bcc24c256acfff898ed251e&quota=1000000000000' | python -m json.tool

{
    "error_code": "025",
    "error_msg": "Excess of quota",
    "ext_msg": "10996929098383 > 10000000000000",
    "success": "false"
}
```

---

#### `update_user`
 
**Действие**: изменение параметров пользователя.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `personal` | Данные пользователя | JSON-массив. Включает следующие поля: `first_name`: имя, `middle_name`: отчество, `last_name`: фамилия, `recovery_email`: e-mail для восстановления пароля, `lang`: язык | - |
| `props` | Произвольные поля, которые, помимо предустановленных, могут быть добавлены к группе | JSON-массив | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=update_user&user_id=U4222huorJAkSDz90GZ&group_id=G4221TsgdGteeUWhcj&personal=%7B%22first_name%22%3A%22Johny%22%2C%22last_name%22%3A%22Doe%22%2C%22middle_name%22%3A%22%22%2C%22recovery_email%22%3A%22email%40mail.ru%22%2C%22email%22%3A%22newuser%4042-2-t23.myoffice.ru%22%7D&token=04221f50d67998bcc24c256acfff898ed251e' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}
```

**Примечание**: Метаданные (`personal`) владельца компании может изменять только владелец компании.

---

#### `set_user_status`

**Действие**: Блокировать/активировать/удалять пользователя.

**Примечание**: при выполнении блокировки пользователя все сессии последнего завершаются. В результате пользователь не сможет авторизоваться. Если блокировка произошла после того как пользователь был авторизован, то любое его действие приведёт к принудительной деавторизации.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `status` | Блокировать/разблокировать/удалить | `1`: разблокировать пользователя, `2`: заблокировать пользователя, `4` – удалить пользователя | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_user_status&user_id=U4221hup8Mxhzs6JTz4&token=04221f50d67998bcc24c256acfff898ed251e&status=1' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}
```

**Статусы пользователя, примечание**:

- Пользователь активен (`isActive = 1`): стандартное штатное состояния. Пользователь работает в системе без ограничений, пользователь не блокирован.
- Пользователь блокирован (`isActive = 0`): пользователь не может залогиниться в системе, но отображается в списке пользователей.
- Пользователь  удален (`isDeleted = 1`): пользователь блокирован, а его профиль и данные помечены на удаление сборщиком мусора. Данные не удаляются, и могут сменить владельца. В этом случае пользователь не отображается в списке пользователей.

**Стандартные ошибки**


```
Не указан токен

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_user_status&user_id=U4221hup8Mxhzs6JTz4&status=1' | python -m json.tool

{
    "error_code": "001",
    "error_msg": "Not Authorised",
    "ext_msg": "",
    "success": "false"
}
```

```
Пользователь с указанным user_id не найден

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_user_status&user_id=U1221hup8Mxhzs6JTz4&token=04221f50d67998bcc24c256acfff898ed251e&status=1' | python -m json.tool

{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "(User is not found)",
    "success": "false"
}
```

```
Указан несуществующий status

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_user_status&user_id=U4221hup8Mxhzs6JTz4&token=04221f50d67998bcc24c256acfff898ed251e&status=666' | python -m json.tool

{
    "error_code": "132",
    "error_msg": "Parameter is not correct",
    "ext_msg": "(status)",
    "success": "false"
}
```

```
Невозможно выполнить команду в отношении владельца компании

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_user_status&user_id=U4221TsgdGrWQh0hW1&token=04221f50d67998bcc24c256acfff898ed251e&status=1' | python -m json.tool

{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "user is company owner",
    "success": "false"
}
```

```
Попытка заблокировать уже заблокированного пользователя

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_user_status&user_id=U4221hup8Mxhzs6JTz4&token=04221f50d67998bcc24c256acfff898ed251e&status=2' | python -m json.tool

{
    "error_code": "302",
    "error_msg": "OPERATION_FAILED",
    "ext_msg": "You may deactivate only active user",
    "success": "false"
}
```

```
Попытка активировать уже активного пользователя

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_user_status&user_id=U4221hup8Mxhzs6JTz4&token=04221f50d67998bcc24c256acfff898ed251e&status=1' | python -m json.tool

{
    "error_code": "302",
    "error_msg": "OPERATION_FAILED",
    "ext_msg": "You may activate only blocked user",
    "success": "false"
}
```
---

#### `add_user_to_group`

**Действие**: добавить пользователя в группу.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор компании | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_user_to_group&group_id=G4221hteXXqUtqz43Xc&user_id=U4222htdEBQlhKqbM4C&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}

Повторное выполнение вернёт следующую ошибку:

{
    "error_code": "302",
    "error_msg": "OPERATION_FAILED",
    "ext_msg": "(User is already linked to group)",
    "success": "false"
}
```

---

#### `revoke_user_from_group`

**Действие**: удалить пользователя из группы. Отметим, что, хотя компания является группой, удалить его при помощи данной команды нельзя - следует использовать команду [`set_user_status`](#set_user_status).

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор компании | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=revoke_user_from_group&group_id=G4221hteXXqUtqz43Xc&user_id=U4222htdEBQlhKqbM4C&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}

Повторное выполнение вернёт следующую ошибку:

{
    "error_code": "302",
    "error_msg": "OPERATION_FAILED",
    "ext_msg": "(User is not linked to group)",
    "success": "false"
}
```

---

#### `set_group_user_roles`

**Действие**: изменить роль пользователя в компании.

**Примечание**: компания может иметь произвольное количество администраторов.

**Примечание**: администратор может применить команду с параметром `unset_admin` к себе самому со всеми вытекающими последствиями. При этом владелец компании не может быть лишён прав администрировать компанию.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор компании | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `set_admin` | Назначить ли пользователя администратором группы | `1`: назначить | - |
| `unset_admin` | Назначить ли пользователя администратором группы | `1`: назначить | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_user_roles&user_id=U4221hup8Mxhzs6JTz4&group_id=G4221TsgdGteeUWhcj&set_admin=1&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_user_roles&user_id=U4221hup8Mxhzs6JTz4&group_id=G4221TsgdGteeUWhcj&unset_admin=1&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

Повторное выполнение команды не возвращает ошибку.

```

**Стандартные ошибки**

```
Указаны не все обязательные параметры

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_user_roles&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqAraD9tLIrOah&user_id=Udev24hqy4M5fl0dfI2b' | python -m json.tool
{
    "error_code": "122",
    "error_msg": "NO_ONE_OF_NECESSARY_PARAMS_IS_FILLED",
    "ext_msg": "(set_admin, unset_admin)",
    "success": "false"
}
```
```
Пользователь "tests-ivas8@email.tests" не имеет права администрировать группу

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_user_roles&token=dev24339df68e60a1831acee83071aa21d5d8&group_id=Gdev24hqAraD9tLIrOah&user_id=Udev24hqy4M5fl0dfI2b&unset_admin=1' | python -m json.tool
{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "",
    "success": "false"
}
```
```
Ошибка авторизации

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_user_roles&token=WRONG&group_id=Gdev24hqAraD9tLIrOah&user_id=Udev24hqy4M5fl0dfI2b&unset_admin=1' | python -m json.tool
{
    "error_code": "015",
    "error_msg": "Invalid Token",
    "ext_msg": "",
    "success": "false"
}
```
```
Неправильный user_id

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_user_roles&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqAraD9tLIrOah&user_id=WRONG&unset_admin=1' | python -m json.tool
{
    "error_code": "054",
    "error_msg": "User not Found",
    "ext_msg": "1866",
    "success": "false"
}
```

---

#### `set_disk_quota`

**Действие**: Выделить квоту на дисковое пространство произвольного размера (в байтах) для пользователя. Пользователь, чей токен дан команде, должен быть администратором группы с указанным `group_id`.

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |
| `granted` | Объем дискового пространства, выделяемого пользователю | Количество Кбайт | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_disk_quota&user_id=U4221TwpNdJYkf8qsl&group_id=G4221TsgdGteeUWhcj&granted=1073741824&token=04221f50d67998bcc24c256acfff898ed251e' | python -m json.tool

{  
   "response":{  
      "set_disk_quota":{  
         "undistributed":9986929098383
      },
      "success":"true"
   },
   "success":"true"
}
```

**Пояснение**: `"undistributed": 5368708120` - нераспределённая квота группы. Для тарифов MosReg эта функция не применима.

---

#### `add_features_to_user`

**Действие**: Добавить фичу пользователю группы, т.е. добавить 2 Гбайт (не более одного раза) либо 4 Гбайт (допускается более одной фичи на пользователя). Данная команда применима исключительно для тарифов блока MosReg.

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |
| `feature_id` | Пакет дискового пространства фиксированного размера (2 или 4 Гбайт); количество пользователей, для которого выделяется указанный пакет | `2`: количество пользователей, `5`: пакет 2 Гбайт (не более одного пакета на пользователя), `6`: пакет 4 Гбайт (допускается более одного пакета на пользователя) | + |
| `features` | Массив значений feature_id, см. выше | Пример: `["5", "6"]` | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_features_to_user&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqyT7mrlZzy4eX&user_id=Udev24hqy4M5fl0dfI2b&features=["5"]' | python -m json.tool

{
    "response": {
        "add_features_to_user": {
            "granted_to_user": ​2147483648,
            "group_distributed_features": {
                "5": ​1
            },
            "group_features": {
                "1": ​5368709120,
                "2": ​10010,
                "5": ​10000,
                "6": ​100
            },
            "group_id": "Gdev24hqyT7mrlZzy4eX",
            "user_features": {
                "5": ​1
            },
            "user_id": "Udev24hqy4M5fl0dfI2b"
        },
        "success": "true"
    },
    "success": "true"
}
```

**Пояснения**:

- `user_features`: в нашем примере у пользователя осталась две фичи (5 и 6).
- `granted_to_user`: по ней он получает 2GB + 4GB дискового пространства.

Повторная попытка вернет ошибку:

```
{
    "error_code": "302",
    "error_msg": "OPERATION_FAILED",
    "ext_msg": "(can't add feature to user twice: feature=5)",
    "success": "false"
}
```

---

#### `revoke_features_from_user`

**Действие**: Аннулировать фичу пользователю группы (исключительно для тарифов блока MosReg).

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |
| `feature_id` | Пакет дискового пространства фиксированного размера (2 или 4 Гбайт); количество пользователей, для которого отзывается указанный пакет | `2`: количество пользователей, `5`: пакет 2 Гбайт, `6`: пакет 4 Гбайт | + |
| `features` | Массив значений feature_id, см. выше | Пример: `["5", "6"]` | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=revoke_features_from_user&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqyT7mrlZzy4eX&user_id=Udev24hqy4M5fl0dfI2b&features=["5"]' | python -m json.tool

{
    "response": {
        "revoke_features_from_user": {
            "granted_to_user": ​0,
            "group_distributed_features": {
                "5": ​0
            },
            "group_features": {
                "1": ​5368709120,
                "2": ​10010,
                "5": ​10000,
                "6": ​100
            },
            "group_id": "Gdev24hqyT7mrlZzy4eX",
            "user_features": { },
            "user_id": "Udev24hqy4M5fl0dfI2b"
        },
        "success": "true"
    },
    "success": "true"
}
```

**Примечания**:

- `user_features`: у пользователя осталась одна фича, т.е. `6`
- `granted_to_user`: по ней он получает 4 Гбайт дискового пространства.
- `group_features`: количество фич компании.

Повторная попытка выполнить аналогичный запрос вернет ошибку

```
{
    "error_code": "302",
    "error_msg": "OPERATION_FAILED",
    "ext_msg": "(can't revoke not added feature: feature=5. 269)",
    "success": "false"
}
```

---

#### `reset_password`

**Действие**: сброс пароля пользователя.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |

При выполнении команды...

1. Производится проверка, является ли пользователь `user_id` членом хотя бы одной корп группы, которую вы (`token`) администрируете.
2. Производится проверка, есть ли у пользователя `user_id` `recovery_email`.
3. Блокируется пароль пользователя и на его `recovery_email` высылается ссылка, по которой в течение некоторого времени можно поменять пароль.

После сброса пароля все сессии пользователя будут завершены. Пользователь сможет продолжить работу после того как сформирует новый пароль. Для этого на резервный email ему будет отправлено письмо с ссылкой на форму ввода нового пароля.

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=reset_password&token=04221808aa8c87e3d1e34119b1655229560eb&user_id=U4221TsgdGrWQh0hW1' | python -m json.tool

{
    "response": {
        "success": "true"
    },
    "success": "true"
}
```

**Дополнительная информация**

- Администратор может поменять recovery_email командой [update_user](#обновить-данные-пользователя).
- Пользователь может поменять пароль перейдя по ссылке из письма командой `reset_password`.
- Восстановление пароля выполняется через WebAPI.

---

### Вывод данных о пользователе

#### `corp_groups`
 
**Действие**: вывести данные о группах, в которых состоит пользователь, а также их свойства и фичи.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=corp_groups&token=04221f50d67998bcc24c256acfff898ed251e' | python -m json.tool

{  
   "response":{  
      "corp_groups":[  
         {  
            "granted":"10000000000000",
            "group_id":"G4221TsgdGteeUWhcj",
            "is_corp":1,
            "is_tariffed":1,
            "name":"",
            "props":{  
               "features":{  
                  "1":10000000000000,
                  "2":5000
               }
            }
         }
      ],
      "success":"true"
   },
   "success":"true"
}
```

---

#### `get_personal_data`
 
**Действие**: получить информацию о компаниях/группах, в которых состоит пользователь, а также данные пользователя.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `show_corp_groups` | Флаг отображения компаний, в которых состоит пользователь | `0`: не отображать, `1`: отображать | - |
| `show_all_groups` | Флаг отображения групп, в которых состоит пользователь | `0`: не отображать, `1`: отображать | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=get_personal_data&user_id=U4221TsgdGrWQh0hW1&show_corp_groups=1&token=04221f50d67998bcc24c256acfff898ed251e' | python -m json.tool

{  
   "response":{  
      "corp_groups":[  
         {  
            "granted":10000000000000,
            "group_id":"G4221TsgdGteeUWhcj",
            "is_admin":1,
            "name":""
         }
      ],
      "is_corp_admin":1,
      "personal":{  
         "email":"tw.ad@42-2-t23.myoffice.ru",
         "lang":"ru-RU",
         "user_id":"U4221TsgdGrWQh0hW1"
      },
      "success":"true"
   },
   "success":"true"
}
```

**Примечание**: При одновременнои использовании параметров `show_all_groups` и `show_corp_groups` будет использован только `show_corp_groups`.

**Примечание**: параметр `is_corp_admin` указывает является ли пользователь администратором хотя бы в одной группе.

---

#### `list_users`

**Действие**:

- Вывести список пользователей.
- Вывести список пользователей (для тарифов блока MosReg).

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |
| `limit` | Ограничение на количество выводимых данных | Целое число | - |
| `offset` | Начальное значение диапазона выводимых данных | Целое число | - |
| `sort_field` | Критерий сортировки данных | Строка | - |
| `sort_type` | Способ сортировки данных | `asc`: восходящая сортировка, `desc`: нисходящая сортировка | - |
| `with_files_count` | Флаг отображения количества файлов | `0`: не отображать, `1`: отобразить | - |
| `with_registration_time` | Флаг отображения даты регистрации | `0`: не отображать, `1`: отобразить | - |
| `with_used` | Флаг отображения объёма использованного дискового пространства | `0`: не отображать, `1`: отобразить | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=list_users&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d&group_id=G4221TsgdGteeUWhcj&with_files_count=1&with_registration_time&with_used=1' | python -m json.tool

{
    "response": {
        "list_users": [
            {
                "email": "test-group@42-2-t23.myoffice.ru",
                "files_count": 1,
                "first_name": "\u041c\u0438\u0448\u0430",
                "granted": 2000000000,
                "group_id": "G4221TsgdGteeUWhcj",
                "is_admin": 0,
                "is_app": 0,
                "is_blocked": 0,
                "last_activity": 1463397538,
                "last_name": "\u041f\u0435\u0442\u0440\u043e\u0432\u0438\u0447",
                "login": "test-group@42-2-t23.myoffice.ru",
                "middle_name": "\u0428\u0442\u0440\u044b\u043a\u043e\u0432",
                "personal": {
                    "first_name": "\u041c\u0438\u0448\u0430",
                    "lang": "ru-RU",
                    "last_name": "\u041f\u0435\u0442\u0440\u043e\u0432\u0438\u0447",
                    "middle_name": "\u0428\u0442\u0440\u044b\u043a\u043e\u0432",
                    "recovery_email": "project666@yandex.ru"
                },
                "recovery_email": "project666@yandex.ru",
                "used": 56332,
                "user_id": "U4221hteYHlZ8fZBd1l"
            },
            {
                "email": "tw1@42-2-t23.myoffice.ru",
                "files_count": 2,
                "first_name": "tw1",
                "granted": 2000000000,
                "group_id": "G4221TsgdGteeUWhcj",
                "is_admin": 0,
                "is_app": 0,
                "is_blocked": 0,
                "last_activity": 1461721885,
                "last_name": "tw1",
                "login": "tw1@42-2-t23.myoffice.ru",
                "middle_name": "tw1",
                "personal": {
                    "first_name": "tw1",
                    "lang": "ru-RU",
                    "last_name": "tw1",
                    "middle_name": "tw1",
                    "recovery_email": "zapzarap@yandex.ru"
                },
                "recovery_email": "zapzarap@yandex.ru",
                "used": 62025,
                "user_id": "U4222htbUC57mbO3ibS"
            }
        ],
        "success": "true"
    },
    "success": "true"
}
```
---

## Операции над группами

### Общая информация об операциях над группами

Пользователь AdminAPI может выполнять следующие действия в отношении групп:

- создавать группы: `add_group`
- блокировать/разблокировать/удалять группы: `set_group_status`
- задавать новое имя группы, добавлять дополнительные поля: `update_group`
- выводить список активных фич группы: `list_group_features`
- выводить список активных фич для всех групп администратора: `corp_groups`
- выводить список приобретённых тарифов: `list_group_tariffs` 

### Управление группами

#### `add_group`

**Действие**: создать группу.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор компании | Строка | + |
| `name` | Название группы | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_group&group_id=G4221TsgdGteeUWhcj&name=%D0%9D%D0%BE%D0%B2%D0%B0%D1%8F%20%D0%B3%D1%80%D1%83%D0%BF%D0%BF%D0%B0&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "group_id":"G4222TBoc0mcNLzvJF",
      "success":"true"
   },
   "success":"true"
}
```

**Ошибки**

```
Неверный токен

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_group&token=WRONG&group_id=Gdev24hqyT7mrlZzy4eX&name=SubGroup1' | python -m json.tool
{
    "error_code": "015",
    "error_msg": "Invalid Token",
    "ext_msg": "",
    "success": "false"
}
```

```
Отсутствует обязательный параметр group_id

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_group&token=dev24226639602bdb20b883eefad78940d895&group_id=&name=SubGroup1' | python -m json.tool
{
    "error_code": "031",
    "error_msg": "Parameter is not defined",
    "ext_msg": "(group_id)",
    "success": "false"
}
```

```
У пользователя отсутствует доступ к редактированию группы group_id

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_group&token=dev24339df68e60a1831acee83071aa21d5d8&group_id=Gdev24hqyT7mrlZzy4eX&name=SubGroup1' | python -m json.tool
{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "",
    "success": "false"
}
```

```
Указанная родительская группа не существует

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_group&token=dev24226639602bdb20b883eefad78940d895&group_id=WRONG&name=SubGroup1' | python -m json.tool
{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "",
    "success": "false"
}
```

---

#### `set_group_status`

**Действие**: блокировать/разблокировать/удалить группу.

**Примечание**: если требуется удалить группу, которая содержит подгруппы, то следует использовать параметр force.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |
| `status` | Команда на выполнение одного из действий, см. значение | `1`: разблокировать группу, `2`: заблокировать группу, `3`: не определено, `4` - удалить группу | + |
| `force` | Флаг рекурсивного удаления всех подгрупп текущей группы | `0`: не удалять, `1`: удалить | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_status&group_id=G4222TBoc0mcNLzvJF&status=4&force=1&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "group_id":"G4222TBoc0mcNLzvJF",
      "success":"true"
   },
   "success":"true"
}
```

**Стандартные ошибки**

```
Недействительный токен

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_status&token=WRONG&group_id=Gdev24ho7fYZByzDLT51&status=4' | python -m json.tool

{
    "error_code": "015",
    "error_msg": "Invalid Token",
    "ext_msg": "",
    "success": "false"
}
```

```
Нет прав администрировать группу

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_status&token=dev24339df68e60a1831acee83071aa21d5d8&group_id=Gdev24ho7fYZByzDLT51&status=4' | python -m json.tool

{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "",
    "success": "false"
}
```

```
Некорректный group_id

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_status&token=dev24226639602bdb20b883eefad78940d895&group_id=WRONG&status=4' | python -m json.tool

{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "",
    "success": "false"
}
```

```
Некорректный параметр status

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_group_status&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24ho7fYZByzDLT51&status=9' | python -m json.tool

{
    "error_code": "132",
    "error_msg": "Parameter is not correct",
    "ext_msg": "(status)",
    "success": "false"
}
```

---

#### `update_group`

**Действие**: Задать новое имя группы и/или добавить дополнительные поля.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |
| `name` | Имя группы | Строка | - |
| `props` | Произвольные поля, которые, помимо предустановленных, могут быть добавлены к группе | JSON-массив | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=update_group&group_id=G4221hteXXqUtqz43Xc&name=%D0%93%D1%80%D1%83%D0%BF%D0%BF%D0%B0%202&token=04221f50d67998bcc24c256acfff898ed251e' | python -m json.tool

{  
   "response":{  
      "success":"true",
      "update_group":{  
         "group_id":"G4221hteXXqUtqz43Xc",
         "name":"Группа 2"
      }
   },
   "success":"true"
}
```

---

### Вывод данных о группе

#### `list_group_features`

**Действие**: Вывести список активных фич для группы.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=list_group_features&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d&group_id=G4221TsgdGteeUWhcj' | python -m json.tool

{
    "response": {
        "list_group_features": {
            "1": 10000000000000,
            "2": 5000
        },
        "success": "true"
    },
    "success": "true"
}
```

---

#### `corp_groups`

**Действие**:

- Получить список активных фич для всех групп администратора.
- Получить информацию о фичах, предоставленных компании (для тарифов блока MosReg).

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=corp_groups&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "corp_groups":[  
         {  
            "granted":"10000000000000",
            "group_id":"G4221TsgdGteeUWhcj",
            "is_corp":1,
            "is_tariffed":1,
            "name":"",
            "props":{  
               "features":{  
                  "1":10000000000000,
                  "2":5000
               }
            }
         }
      ],
      "success":"true"
   },
   "success":"true"
}
```

---

#### `list_group_tariffs`

**Действие**: Вывести список приобретённых тарифов.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `group_id` | Идентификатор группы | Строка | + |
| `limit` | Ограничение на количество выводимых данных | Целое число | - |
| `offset` | Начальное значение диапазона выводимых данных | Целое число | - |
| `sort_field` | Критерий сортировки данных | Строка | - |
| `sort_type` | Способ сортировки данных | `asc`: восходящая сортировка, `desc`: нисходящая сортировка | - |
| `with_files_count` | Флаг отображения количества файлов | `0`: не отображать, `1`: отобразить | - |
| `with_registration_time` | Флаг отображения даты регистрации | `0`: не отображать, `1`: отобразить | - |
| `with_used` | Флаг отображения объёма использованного дискового пространства | `0`: не отображать, `1`: отобразить | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=list_group_tariffs&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d&group_id=G4221TsgdGteeUWhcj&with_files_count=1&with_registration_time&with_used=1' | python -m json.tool

{
    "response": {
        "list_group_tariffs": [
            {
                "end_time": 32503669200,
                "group_id": "G4221TsgdGteeUWhcj",
                "invoice_id": "G4221TsgdGteeUWhcj",
                "is_active": 1,
                "mnem": "corp_3",
                "name": "Professional corp tariff. 10 TB. 5000 users",
                "props": {
                    "features": {
                        "1": 10000000000000,
                        "2": 5000
                    }
                },
                "purchase_time": 1461237458,
                "start_time": 1461237458,
                "tariff_id": "corp_3"
            }
        ],
        "success": "true"
    },
    "success": "true"
}
```

---


## Операции над списками рассылок

### Общая информация об операциях над списками рассылок

Пользователь AdminAPI может выполнять следующие действия в отношении списков рассылок:

- создавать списки рассылки: `add_maillist`
- изменять списки рассылки: `update_maillist`
- удалять списки рассылки: `delete_maillist`
- выводить список всех списков рассылки: `get_maillist`
- выводить данные списка рассылки: `list_maillists`

### Управление списком рассылки

#### `add_maillist`

**Действие**: создать список рассылки.

**Примечание**: возможна ситуация, когда список рассылки состоит в собственном списке рассылки, т.е. в списке А состоит список В, в списке В состоит список С, а в списке С состоит список А. Такой список рассылки будет работать вполне корректно 

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `active` | Флаг активации списка рассылки | `0`: не активировать, `1`: активировать | + |
| `address` | E-mail адрес списка рассылки | email-адрес | + |
| `isVisibleFromOutside` | Флаг доступности списка рассылки снаружи | `0`: недоступен, `1`: доступен | + |
| `mailboxes` | Адреса электронной почты адресатов списка рассылки (может включать другие списки рассылок) | JSON-массив | + |
| `title` | Название списка рассылки | Строка | + |
| `description` | Описание списка рассылки | Строка | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_maillist&address=666%4042-2.myoffice.ru&title=New+nice+mail+list&active=1&mailboxes=%5B%22%22%5D&isVisibleFromOutside=1&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "add_maillist":"M4221TBff8wcRmIv2h",
      "success":"true"
   },
   "success":"true"
}
```

---

#### `update_maillist`

**Действие**: изменить список рассылки.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `id` | Идентификатор списка рассылки | Строка | + |
| `active` | Флаг активации списка рассылки | `0`: не активировать, `1`: активировать | - |
| `address` | E-mail адрес списка рассылки | email-адрес | - |
| `isVisibleFromOutside` | Флаг доступности списка рассылки снаружи | `0`: недоступен, `1`: доступен | - |
| `mailboxes` | Адреса электронной почты адресатов списка рассылки (может включать другие списки рассылок) | JSON-массив | - |
| `title` | Название списка рассылки | Строка | - |
| `description` | Описание списка рассылки | Строка | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=update_maillist&id=M4221TBff8wcRmIv2h&mailboxes=%5B%22zeppy%4042-2-t23.myoffice.ru%22%2C%22zapp%4042-2-t23.myoffice.ru%22%2C%22test-group%4042-2-t23.myoffice.ru%22%5D&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "success":"true",
      "update_maillist":1
   },
   "success":"true"
}
```

---

#### `delete_maillist`

**Действие**: удалить список рассылки.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `id` | Идентификатор списка рассылки | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=delete_maillist&id=M4221TBff8wcRmIv2h&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "delete_maillist":1,
      "success":"true"
   },
   "success":"true"
}
```

---

### Вывести данные о списках рассылки

#### `list_maillists`

**Действие**: вывести список списков рассылки.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=list_maillists&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "list_maillists":[  
         {  
            "address":"aza2@42-2.myoffice.ru",
            "id":"M4222hsU4E7B9matMqK",
            "title":"Test- 02"
         },
         {  
            "address":"msk@pgfsapi.test.domain",
            "id":"M4221hsTKNHtaIcOJ7i",
            "title":"Столица"
         },
         {  
            "address":"emaillist@42-2.myoffice.ru",
            "id":"M4221hsU21D4mtJ7MlK",
            "title":"Проверка_доставки_почты_всем_участникам_рассылки"
         },
         {  
            "address":"aza3@42-2.myoffice.ru",
            "id":"M4222hsU4Xa7wSe7mO7",
            "title":"test-3"
         }
      ],
      "success":"true"
   },
   "success":"true"
}
```

---

#### `get_maillist`

**Действие**: вывести данные о списке рассылки.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `id` | Идентификатор списка рассылки | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=get_maillist&id=M4221TflFpEX3NV7jC&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "get_maillist":{  
         "active":1,
         "address":"deletemaillist@42-2.myoffice.ru",
         "description":"",
         "id":"M4221TflFpEX3NV7jC",
         "isVisibleFromOutside":"1",
         "mailboxes":[  

         ],
         "title":"Check ability to delete mail list"
      },
      "success":"true"
   },
   "success":"true"
}
```

---

## Операции над доменами

### Общая информация об операциях над доменами

Пользователь AdminAPI может выполнять следующие действия в отношении доменов:

- создавать новый домен: `add_domain`
- удалять домен: `delete_domain`
- выводить список доменов: `list_domains`

### Управление доменами

#### `add_domain`

**Действие**: создать новый домен.

Возможность добавить домен позволяет назначать пользователям основные email с доменом, отличающимся от домена по умолчанию. Эта возможность называется мультидоменностью.

**Внимание**: добавлять следует только те домены, которые принадлежат вам. Попытка добавить «чужой» домен, например, mail.ru/yandex.ru/gmail.com, приведёт к тому, что пользователи компании перестанут получать почту из этих доменов

**Дополнительная информация**: чтобы добавляемый домен стал доступен «снаружи», необходимо во внешнем DNS добавить MX-запись, указывающую на внешний IP-адрес вашей инсталляции. Проверить корректность MX-записи для домена можно командой unix-консоли

`nslookup –type=mx <имя проверяемого домена> 8.8.8.8`

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `domain` | Имя домена | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_domain&domain=anotherdomain.ru&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "add_domain":"anotherdomain.ru",
      "success":"true"
   },
   "success":"true"
}
```

---

#### `delete_domain`

**Действие**: удалить домен.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `domain` | Имя домена | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=delete_domain&domain=anotherdomain.ru&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}

Повторное выполнение не вызовет ошибку.
```

---

### Вывести информацию о доменах

#### `list_domains`

**Действие**: вывести список доменов.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=list_domains&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' --compressed

{  
   "response":{  
      "list_domains":{  
         "default_domain":"42-2-t23.myoffice.ru",
         "domains":[  
            {  
               "addresses_count":0,
               "domain":"delete.ru",
               "group_id":"G4221TsgdGteeUWhcj",
               "is_default":0
            },
            {  
               "addresses_count":0,
               "domain":"newdomain.ru",
               "group_id":"G4221TsgdGteeUWhcj",
               "is_default":0
            }
         ]
      },
      "success":"true"
   },
   "success":"true"
}
```

---

## Операции над псевдопользователями

### Общая информация об операциях над псевдопользователями

Пользователь AdminAPI может выполнять следующие действия в отношении псевдопользователей:

- создавать псевдопользователя: `add_resource`
- изменять параметры псевдопользователя: `update_resource`
- блокировать/разблокировать/удалять псевдопользователей: `set_resource_status`
- выводить список псевдопользователей: `list_resources`
- выводить данные псевдопользователя: `get_resource`

### Управление псевдопользователями

#### `add_resource`

**Действие**: создать нового псевдопользователя.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `email` | Почтовый адрес | email адрес | + |
| `name` | Наименование псевдопользователя | Строка | - |
| `type` | Тип псевдопользователя | `room`, `inventory`, `other` | - |
| `group_id` | Идентификатор группы | Строка | + |
| `description` | Описание псевдопользователя | Строка | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_resource&group_id=G4221TsgdGteeUWhcj&email=salvador%4042-2-t23.myoffice.ru&name=Salvador&type=room&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "id":"U4221TBpLfwqeWJSEc",
      "is_resource":1,
      "is_user":0,
      "root_id":"O4221TBpLfwuSrPQgk",
      "success":"true"
   },
   "success":"true"
}
```

**Стандартные ошибки**

```

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_resource&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqyT7mrlZzy4eX&login=resource1@email.tests&email=resource1@email.tests&name=room1&type=room&description=Petrovka38' | python -m json.tool
{
    "error_code": "013",
    "error_msg": "Email is not available",
    "ext_msg": "",
    "success": "false"
}
```
```

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_resource&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqyT7mrlZzy4eX&login=resource2@email.tests&email=resource1@email.tests&name=1&type=room&description=Petrovka38' | python -m json.tool
{
    "error_code": "046",
    "error_msg": "Invalid name",
    "ext_msg": "",
    "success": "false"
}
```
```

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_resource&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqyT7mrlZzy4eX&login=resource1@email.tests&email=resource1@email.tests&name=room1&type=&description=Petrovka38' | python -m json.tool
{
    "error_code": "031",
    "error_msg": "Parameter is not defined",
    "ext_msg": "(type)",
    "success": "false"
}
```
```

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=add_resource&token=dev24226639602bdb20b883eefad78940d895&group_id=Gdev24hqyT7mrlZzy4eX&login=resource1@email.tests&email=resource1@email.tests&name=room1&type=WRONG&description=Petrovka38' | python -m json.tool
{
    "error_code": "124",
    "error_msg": "Invalid parameter (see ext_info)",
    "ext_msg": "(type)",
    "success": "false"
}
```

---

#### `update_resource`

**Действие**: изменить псевдопользователя.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `resource_id` | Идентификатор псевдопользователя | Строка | + |
| `group_id` | Идентификатор компании | Строка | + |
| `name` | Наименование псевдопользователя | Строка | + |
| `type` | Тип псевдопользователя | `room`, `inventory`, `other` | + |
| `description` | Описание псевдопользователя | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=update_resource&resource_id=U4221TBpLfwqeWJSEc&type=room&name=Gonduras&email=salvador%4042-2-t23.myoffice.ru&description=&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}
```

**Стандартные ошибки**

```
Не указан обязательный параметр resource_id

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=update_resource&token=dev24226639602bdb20b883eefad78940d895&type=other' |  python -m json.tool

{
    "error_code": "031",
    "error_msg": "Parameter is not defined",
    "ext_msg": "(resource_id)",
    "success": "false"
}
```

```
Некорректный параметр type

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=update_resource&token=dev24226639602bdb20b883eefad78940d895&resource_id=Udev24Tuz4UDceqXzWL&type=WRONG'  |  python -m json.tool
{
    "error_code": "124",
    "error_msg": "Invalid parameter (see ext_info)",
    "ext_msg": "(type)",
    "success": "false"
}
```

---

#### `set_resource_status`

**Действие**: блокировать/разблокировать/удалить псевдопользователя.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `resource_id` | Идентификатор псевдопользователя | Строка | + |
| `status` | Команда, см. значение | `1`: разблокировать псевдопользователя, `2`: заблокировать псевдопользователя, `4` - удалить псевдопользователя | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_resource_status&resource_id=U4221TBpLfwqeWJSEc&status=4&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "success":"true"
   },
   "success":"true"
}
```

**Стандартные ошибки**

```
Псевдопользователь с указанным resource_id не существует

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=set_resource_status&token=dev24226639602bdb20b883eefad78940d895&resource_id=Udev24Tuz4UDceqXzWL&status=1' |  python -m json.tool

{
    "error_code": "045",
    "error_msg": "Object not found",
    "ext_msg": "(resource=Udev24Tuz4UDceqXzWL)",
    "success": "false"
}
```

---

### Вывод данных псевдопользователей

#### `list_resources`

**Действие**: вывести список псевдопользователей и их данные.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `limit` | Ограничение на количество выводимых данных | Целое число | - |
| `offset` | Начальное значение диапазона выводимых данных | Целое число | - |
| `sort_field` | Критерий сортировки данных | Строка | - |
| `sort_type` | Способ сортировки данных | `asc`: восходящая сортировка, `desc`: нисходящая сортировка | - |
| `filter` | Критерий отбора псевдопользователей, по которым должны быть предоставлены данные | Строка | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=list_resources&sort_field=name&sort_type=asc&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "list_resources":[  
         {  
            "description":"Welcome",
            "email":"gonduras@42-2-t23.myoffice.ru",
            "granted":0,
            "id":"U4222huz2j6Y53170D0",
            "is_blocked":0,
            "last_activity":0,
            "login":"gonduras@42-2-t23.myoffice.ru",
            "name":"Gonduras",
            "type":"room"
         }
      ],
      "success":"true"
   },
   "success":"true"
}

```

---

#### `get_resource`

**Действие**: вывести данные псевдопользователя.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `resource_id` | Идентификатор ресурса | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=get_resource&resource_id=U4222huz2j6Y53170D0&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{  
   "response":{  
      "resource":{  
         "description":"Welcome",
         "email":"gonduras@42-2-t23.myoffice.ru",
         "name":"Gonduras",
         "type":"room"
      },
      "success":"true"
   },
   "success":"true"
}
```
---

## Восстановление файлов

### Общая информация об операциях по восстановлению файлов

Пользователь AdminAPI может выполнять следующие действия по восстановлению файлов:

- вывести данные о доступных для восстановления файлах: `search_objects_to_restore`
- восстановить файл: `restore_file`

### `search_objects_to_restore`

**Действие**: вывести данные о доступных для восстановления файлах.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `user_id` | Идентификатор пользователя | Строка | + |
| `query` | Начальное значение диапазона выводимых данных | Целое число | - |
| `min_mtime` | Наиболее ранняя дата удаления (Unix timestamp) | Целое число | - |
| `max_mtime` | Наиболее поздняя дата удаления (Unix timestamp) | Целое число | - |
| `min_size` | Минимальный размер файла в байтах | Целое число | - |
| `max_size` | Максимальный размер файла в байтах | Целое число | - |
| `limit` | Ограничение на количество выводимых данных | Целое число | - |
| `offset` | Начальное значение диапазона выводимых данных | Целое число | - |
| `sort_field` | Критерий сортировки данных | Строка | - |
| `sort_type` | Способ сортировки данных | `asc`: восходящая сортировка, `desc`: нисходящая сортировка | - |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=search_objects_to_restore&user_id=U4221hteYHlZ8fZBd1l&token=0422257c5ad90e488eaf4ed69c4d5ed72db9d' | python -m json.tool

{
    "response": {
        "list_objects": [
            {
                "chksum": "a957a3153eb7126b1c5f8b6aac35de53",
                "ctime": 1457532251,
                "current_version": "Odev24eGgdPTBSJytq3",
                "id": "Odev24eGgdPTxcfQWqJ",
                "mtime": 1457532251,
                "name": "restore_files:warm",
                "owner_id": "Udev24eGgdPJp7R0WLn",
                "owner_login": "test3_login-ecbda479405d9ae@a0eb74da191808fa.domain.tests",
                "size": 12,
                "total_size": 12,
                "version_count": 1,
                "versioned": 0
            },
            {
                "chksum": "4a6086a4632358cdc06c48b11179a4ef",
                "ctime": 1457532249,
                "current_version": "Odev24eGgdPOjh0J83X",
                "id": "Odev24eGgdPOc4kKn0h",
                "mtime": 1457532252,
                "name": "restore_files:boy",
                "owner_id": "Udev24eGgdPJp7R0WLn",
                "owner_login": "test3_login-ecbda479405d9ae@a0eb74da191808fa.domain.tests",
                "sha256": "1aef0412032ca12d9537a9ae3b3619b53c02246a0726ad31305943a87a528706",
                "size": 8,
                "total_size": 8,
                "version_count": 1,
                "versioned": 0
            },
            {
                "chksum": "b99a88e441ae7a269ccc8e69e34b9965",
                "ctime": 1457532250,
                "current_version": "Odev24eGgdPRh3L3ue7",
                "id": "Odev24eGgdPQEa6j9ex",
                "mtime": 1457532253,
                "name": "restore_files:girl",
                "owner_id": "Udev24eGgdPJp7R0WLn",
                "owner_login": "test3_login-ecbda479405d9ae@a0eb74da191808fa.domain.tests",
                "sha256": "178ee5a00441a3ad9eb1b2e51d7964879a61d8ceefe8e0c422ab0eca525e4d7a",
                "size": 10,
                "total_size": 19,
                "version_count": 2,
                "versioned": 0
            }
        ],
        "success": "true"
    },
    "success": "true"
}
```

**Стандартные ошибки**

```
Не указан токен

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=search_objects_to_restore' | python -m json.tool
{
    "error_code": "001",
    "error_msg": "Not Authorised",
    "ext_msg": "",
    "success": "false"
}
```
```
Не указан параметр user_id

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=search_objects_to_restore&token=dev24c827038a06049b822cdffba61751e3b6' | python -m json.tool

{
    "error_code": "031",
    "error_msg": "Parameter is not defined",
    "ext_msg": "(user_id)",
    "success": "false"
}
```

---

### `restore_file`

**Действие**: восстановить файл.

***Параметры:***

| Параметр | Описание | Значение | Обязательный |
|---|---|---|---|
| `token` | Токен авторизации | Строка | + |
| `id` | ID файла | Строка | + |

**Пример запроса, ответа**

```
curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=restore_file&token=dev24c827038a06049b822cdffba61751e3b6&id=Odev24eGgdPQEa6j9ex' | python -m json.tool

{
    "response": {
        "success": "true"
    },
    "success": "true"
}
```

**Стандартные ошибки**

```
Попытка восстановить файл, который не был удалён

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=restore_file&token=dev24c827038a06049b822cdffba61751e3b6&id=Odev24eGgdPQEa6j9ex' | python -m json.tool
{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "Object is not deleted",
    "success": "false"
}
```

```

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=restore_file&token=dev24f244e1ca307769ecca4b7b0b3c07a9ef&id=Odev24eGgdPQEa6j9ex' | python -m json.tool
{
    "error_code": "024",
    "error_msg": "Permissions Denied",
    "ext_msg": "",
    "success": "false"
}
```

```
Попытка восстановить несуществующий файл

curl -k 'https://admin-42-2.myoffice.ru/api/?cmd=restore_file&token=dev24c827038a06049b822cdffba61751e3b6&id=WRONG' | python -m json.tool
{
    "error_code": "045",
    "error_msg": "Object not found",
    "ext_msg": "1593",
    "success": "false"
}
```

---