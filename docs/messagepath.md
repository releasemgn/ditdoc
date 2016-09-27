# Логика работы почтовой системы

## Общая информация

Под почтовой системой подразумевается система доставки сообщений эликтронной почты (SMTP - simple mail transfer protocol) от отправителя (sender) в почтоый ящик получателя (recipient).
Системы получения доступа пользователя к сообщениям, содержащимся в его почтовом ящике не являются компонентами системы доставки сообщений.
Ниже рассмотрена реализация системы, используемая в ПО МойОфис.

## Понятие сеанса SMTP.

Обмен инфрмацией в SMTP-сесии происходит так называемыми сеансами - в ходе каждого сеанса происходит обработка одного (!) сообщения.
Подробное описание сеанса SMTP описано в соответствующих RFC, основной из которых - `RFC 822`; в данном документе сеансы рассмотрены лишь в той мере, в которой это необходимо для понимания работы системы в рамках ПО МойОфис и связей между БД файлового хранилища и почтовой системы.
Таким образом условно можно разделить три типа сенсов:

1. [Передача сообщения от клиента на сервер для последующей отправки получателю/получателям](#%D0%A1%D0%B5%D0%B0%D0%BD%D1%81-%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80)
2. [Передача сообщения сервером клиента на сервер(а) получателя/получателей](#%D0%A1%D0%B5%D0%B0%D0%BD%D1%81-%D0%BD%D0%B0%D1%88_%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80-%D0%B2%D0%BD%D0%B5%D1%88%D0%BD%D0%B8%D0%B9_%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80)
3. [Приём сообщения от внешнего сервера и доставка его в почтовый(е) ящик(и) получателя/получателей обслуживаемых данным сервером](#%D0%A1%D0%B5%D0%B0%D0%BD%D1%81-%D0%B2%D0%BD%D0%B5%D1%88%D0%BD%D0%B8%D0%B9_%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80-%D0%BD%D0%B0%D1%88_%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80)

### Сеанс клиент-сервер

При отправке сообщения со стороны клиента на сервер выполняются следующие стадии:

1. Соединение, при необходимости - переход в зашифрованный режим (StartTLS)
2. Авторизация клиента. В случае неудачной авторизации сервер разрывает соединение и завершает сеанс.
3. Отправка команды **MAIL FROM**: \<sender_address\>
4. Отправка команды **RCPT TO**: \<recipient(s)\_address\>. После извлечения всех адресов получателей сервер выполняет проверку возможности отправки пользователем, авторизованным в п.2. сообщения с адреса, указанного в п.3. (sender_login_maps), в случае отрицательного результата в качестве ответа формируется соответствующее уведомление.
5. Отправка команды **DATA**
6. После команды **DATA** - отправка остальных полей заголовка и тела письма в соответствии с `RFC 822`. После завершения обработки команды **DATA** сервер:
 * проверяет письмо на вирусы, по резудьтатам проверки добавляется соответствющая информация в заголовок
 * добавляет DKIM запись
 * помещает письмо в очередь отправки.

### Сеанс наш\_сервер - внешний\_сервер

Обрабочик очереди, получив письмо выполняет следующую последовательность действий:
1. Группирует всех получателей, указанных в полях **TO**, **CC** и **BCC** по доменам.
2. Для каждого домена:
 * Проверяется, не является ли этот домен локальным (обслуживаемым) для этого сервера (alias_domain & domain). При положительном результате запускается обработка локальной доставки письма - сеанс 3-го типа (внешний\_сервер - наш\_сервер) с шага 4.
 * Запрашивает у DNS информацию об MX-сервере для домена-получателя. При отсутствии информации формируется NDR (Non-Deliverable Report) отправителю исходного письма.
 * Проверяется количество получателей в этом домене, и при превышении максимально допустимого количества получателей в одном домене (конфигурируемый параметр) разбивает сессию на требуемуе число сеансов.
 * Соединяется с MX-сервером получателя и передаёт письмо, добавляя в заголовок поле **Recieved by**: с указанием своего имени и даты/времени пересылки сообщения.
 * Обрабатывает ответы от сервера-получателя, при необходимости откладывая сеанс для повторения позже или формируя NDR отправителю (подробнее - `RFC 822`).
3. В зависимости от результатов обработки ответов всех серверов назначения п.2. письмо либо удаляется из очереди, либо перемещается в очередь на повторную попытку отправки через определённое время (`RFC 822`).

### Сеанс внешний\_сервер - наш\_сервер

При получении запроса на соединение от внешнего сервра выполняются следующие операции:

1. Проверки внешнего сервера (helo\_check) - конфигурируемая последовательность проверок IP, имени сервера в DNS, имени, указанного сервером в команде **HELO** или **EHLO**. По результатам проверки сеанс может быть завершён с сообщением об отказе обработки запроса, либо продолжен (возможно - с изменением SPAM-score письма).
2. Проверка адреса отправителя:
 * Соответствие MX-записи для домена отправителя подключившемуся внешнему серверу.
 * Поиск адреса/домена отправителя по чёрным/белым спискам - состав списков конфигурируется в широких пределах.
 * Наличие общих/корпоративных политик, регламентирующих получение сообщений от этого адреса/домена (sender_bcc_maps_domain/user и других) - конфигурируется в широких пределах администратором организации.
 * Возможно добавление дополнительных, не указанных выше, проверок, конфигурируемых администратором организации.
Действия по результатам проверок аналогичны действиям в п.1.
3. Из полей **TO**, **CC** и **BCC** Составляется список получателей сообщения и список доменов получателей.
4. Для каждого домена выполняются следующие действия:
 * Проверяется, обслуживается ли этот домен нашим сервером (domain & alias_domain), при отрицательном результате получатели, адреса которых находятся в этом домене удаляются из списка обработки получателей и выполняется проверка следующего домена.
 * Проверяется, не является ли этот домен alias-ом другого домена (alias_domain), при положительном результате в адресах всех получателей в этом домене доменная часть заменяется на домен-получатель.
 * Проверяется наличие настроек "получения скрытой копии" для этого домена (recipient_bcc_maps_domain) - конфигурируется в широких пределах администратором организации.
5. Для каждого адреса в оставшемся после п.4. списке получателей производится поиск конечных адресов доставки (virtual_alias_maps). Если по результатам проверки получился пустой список, не содержащий ни одного адреса, формируется NDR отправител исходного письма; в противоположном случае адрес получателя заменяется на список, полученный в результате проверки.
6. Из получившегося списка получателей удаляются дубликаты.
7. Для каждого из оставшихся в итоговом списке адресов выполняются действия:
 * Проверяется наличие настроек "получения скрытой копии" для этого адреса (recipient_bcc_maps_user) - конфигурируется в широких пределах администратором организации.
 * Дополнительные проверки, конфигурируемые администратором организации (корпоративные/групповые чёрные/белые списки; корпоративные/групповые/пользовательские политики)
 * Дополниельная проверка на пользовательские чёрные/белые списки, управляемая самим пользователем
 * Проверка письма на спам, после которой изменяется финальный SPAM-score письма, и, в соответствии с конфигурацией, заданной администратором организации выполняются результирующие действия (пометка письма, как спам, игнорирование письма или продолжение обработки)
 * Передача письма на финальную доставку в соттвествующий доставщик (transport_maps_*)
 * В зависимости от ответа финального доставщика экземпляр письма удяляется из очереди на обработку, либо перемещается в очередь на повторную обработку через некоторое время. Так же, в некоторых случаях, формируется NDR отправителю исходного письма.

## Описание таблиц почтовой БД, упомянутых в документе:

### alias

        Column        |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
----------------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 address              | character varying(255)      | not null default ''::character varying                              | extended |              | 
 name                 | character varying(255)      | not null default ''::character varying                              | extended |              | 
 moderators           | text                        | not null default ''::text                                           | extended |              | 
 accesspolicy         | character varying(30)       | not null default ''::character varying                              | extended |              | 
 domain               | character varying(255)      | not null default ''::character varying                              | extended |              | 
 islist               | smallint                    | not null default 0                                                  | plain    |              | 
 created              | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified             | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 expired              | timestamp without time zone | not null default '9999-12-31 00:00:00'::timestamp without time zone | plain    |              | 
 active               | smallint                    | not null default 1                                                  | plain    |              | 
 id                   | character varying(32)       | not null default NULL::character varying                            | extended |              | 
 title                | character varying(255)      | not null default ''::character varying                              | extended |              | 
 search_title         | tsvector                    | not null default ''::tsvector                                       | extended |              | 
 description          | text                        | not null default ''::text                                           | extended |              | 
 mailboxes            | tsvector                    | not null default ''::tsvector                                       | extended |              | 
 mailboxes_unwrapped  | tsvector                    | not null default ''::tsvector                                       | extended |              | 
 isVisibleFromOutside | boolean                     | not null default true                                               | plain    |              | 

### alias_domain

    Column     |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
---------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 alias_domain  | character varying(255)      | not null                                                            | extended |              | 
 target_domain | character varying(255)      | not null                                                            | extended |              | 
 created       | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified      | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 active        | smallint                    | not null default 1                                                  | plain    |              | 

### domain

   Column    |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
-------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 domain      | character varying(255)      | not null default ''::character varying                              | extended |              | 
 description | text                        | not null default ''::text                                           | extended |              | 
 disclaimer  | text                        | not null default ''::text                                           | extended |              | 
 aliases     | bigint                      | not null default 0                                                  | plain    |              | 
 mailboxes   | bigint                      | not null default 0                                                  | plain    |              | 
 maxquota    | bigint                      | not null default 0                                                  | plain    |              | 
 quota       | bigint                      | not null default 0                                                  | plain    |              | 
 transport   | character varying(255)      | not null default 'dovecot'::character varying                       | extended |              | 
 settings    | text                        | not null default ''::text                                           | extended |              | 
 backupmx    | smallint                    | not null default 0                                                  | plain    |              | 
 created     | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified    | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 expired     | timestamp without time zone | not null default '9999-12-31 00:00:00'::timestamp without time zone | plain    |              | 
 active      | smallint                    | not null default 1                                                  | plain    |              | 
 group_id    | character varying(32)       | default NULL::character varying                                     | extended |              | 
 is_default  | boolean                     | not null default false                                              | plain    |              | 

### mailbox

          Column          |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
--------------------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 username                 | character varying(255)      | not null                                                            | extended |              | 
 password                 | character varying(255)      | not null default ''::character varying                              | extended |              | 
 name                     | character varying(255)      | not null default ''::character varying                              | extended |              | 
 language                 | character varying(5)        | not null default 'en_US'::character varying                         | extended |              | 
 storagebasedirectory     | character varying(255)      | not null default '/var/vmail'::character varying                    | extended |              | 
 storagenode              | character varying(255)      | not null default 'vmail1'::character varying                        | extended |              | 
 maildir                  | character varying(255)      | not null default ''::character varying                              | extended |              | 
 quota                    | bigint                      | not null default 0                                                  | plain    |              | 
 domain                   | character varying(255)      | not null default ''::character varying                              | extended |              | 
 transport                | character varying(255)      | not null default ''::character varying                              | extended |              | 
 department               | character varying(255)      | not null default ''::character varying                              | extended |              | 
 rank                     | character varying(255)      | not null default 'normal'::character varying                        | extended |              | 
 employeeid               | character varying(255)      | default ''::character varying                                       | extended |              | 
 isadmin                  | smallint                    | not null default 0                                                  | plain    |              | 
 isglobaladmin            | smallint                    | not null default 0                                                  | plain    |              | 
 enablesmtp               | smallint                    | not null default 1                                                  | plain    |              | 
 enablesmtpsecured        | smallint                    | not null default 1                                                  | plain    |              | 
 enablepop3               | smallint                    | not null default 1                                                  | plain    |              | 
 enablepop3secured        | smallint                    | not null default 1                                                  | plain    |              | 
 enableimap               | smallint                    | not null default 1                                                  | plain    |              | 
 enableimapsecured        | smallint                    | not null default 1                                                  | plain    |              | 
 enabledeliver            | smallint                    | not null default 1                                                  | plain    |              | 
 enablelda                | smallint                    | not null default 1                                                  | plain    |              | 
 enablemanagesieve        | smallint                    | not null default 1                                                  | plain    |              | 
 enablemanagesievesecured | smallint                    | not null default 1                                                  | plain    |              | 
 enablesieve              | smallint                    | not null default 1                                                  | plain    |              | 
 enablesievesecured       | smallint                    | not null default 1                                                  | plain    |              | 
 enableinternal           | smallint                    | not null default 1                                                  | plain    |              | 
 enabledoveadm            | smallint                    | not null default 1                                                  | plain    |              | 
 enablelib-storage        | smallint                    | not null default 1                                                  | plain    |              | 
 enablelmtp               | smallint                    | not null default 1                                                  | plain    |              | 
 lastlogindate            | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 lastloginipv4            | inet                        | not null default '0.0.0.0'::inet                                    | main     |              | 
 lastloginprotocol        | character(255)              | not null default ''::bpchar                                         | extended |              | 
 disclaimer               | text                        | not null default ''::text                                           | extended |              | 
 allowedsenders           | text                        | not null default ''::text                                           | extended |              | 
 rejectedsenders          | text                        | not null default ''::text                                           | extended |              | 
 allowedrecipients        | text                        | not null default ''::text                                           | extended |              | 
 rejectedrecipients       | text                        | not null default ''::text                                           | extended |              | 
 settings                 | text                        | not null default ''::text                                           | extended |              | 
 passwordlastchange       | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 created                  | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified                 | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 expired                  | timestamp without time zone | not null default '9999-12-31 00:00:00'::timestamp without time zone | plain    |              | 
 active                   | smallint                    | not null default 1                                                  | plain    |              | 
 local_part               | character varying(255)      | not null default ''::character varying                              | extended |              | 

### recipient_bcc_domain

   Column    |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
-------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 domain      | character varying(255)      | not null default ''::character varying                              | extended |              | 
 bcc_address | character varying(255)      | not null default ''::character varying                              | extended |              | 
 created     | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified    | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 expired     | timestamp without time zone | not null default '9999-12-31 00:00:00'::timestamp without time zone | plain    |              | 
 active      | smallint                    | not null default 1                                                  | plain    |              | 

### recipient_bcc_user

   Column    |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
-------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 username    | character varying(255)      | not null default ''::character varying                              | extended |              | 
 bcc_address | character varying(255)      | not null default ''::character varying                              | extended |              | 
 domain      | character varying(255)      | not null default ''::character varying                              | extended |              | 
 created     | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified    | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 expired     | timestamp without time zone | not null default '9999-12-31 00:00:00'::timestamp without time zone | plain    |              | 
 active      | smallint                    | not null default 1                                                  | plain    |              | 

### sender_bcc_domain

   Column    |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
-------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 domain      | character varying(255)      | not null default ''::character varying                              | extended |              | 
 bcc_address | character varying(255)      | not null default ''::character varying                              | extended |              | 
 created     | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified    | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 expired     | timestamp without time zone | not null default '9999-12-31 00:00:00'::timestamp without time zone | plain    |              | 
 active      | smallint                    | not null default 1                                                  | plain    |              | 

### sender_bcc_user

   Column    |            Type             |                              Modifiers                              | Storage  | Stats target | Description 
-------------+-----------------------------+---------------------------------------------------------------------+----------+--------------+-------------
 username    | character varying(255)      | not null default ''::character varying                              | extended |              | 
 bcc_address | character varying(255)      | not null default ''::character varying                              | extended |              | 
 domain      | character varying(255)      | not null default ''::character varying                              | extended |              | 
 created     | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 modified    | timestamp without time zone | not null default '1970-01-01 00:00:00'::timestamp without time zone | plain    |              | 
 expired     | timestamp without time zone | not null default '9999-12-31 00:00:00'::timestamp without time zone | plain    |              | 
 active      | smallint                    | not null default 1                                                  | plain    |              | 
