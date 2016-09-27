# Миграция одного из обслуживаемых Exchange-сервером почтовых доменов на MyOffice.

## Общая информация

Имеется работающая инфраструктура Active Directory + почтовая система Exchange, управляемая через AD.

Требуется один из доменов, обслуживаемых в данный момент Exchange-сервером перевести на обслуживание в систему MyOffice.

Процесс миграции условно делится на 3 фазы:

1. начальное состояние: монопольное существование Echange-сервера.

2. процесс миграции: гибридное сосуществование MyOffice и Exchange, при котором часть мигрирующих пользователей уже в MyOffice, а часть — ещё на Exchange-сервере; именно в этой фазе производится непосредственно миграция пользователей (и их данных).

3. финальное состояние: параллельное существование MyOffice и Exchange, при котором каждая из структур обслуживает свой почтовый домен и является «внешней» почтовой организацией для другой структуры.

## Имена и термины
- `migrated.domain` — почтовый домен домен, пользователи из которого переносятся в систему MyOffice.
- `tmp-migrated.domain` — временный домен, сужествующий только между фазами 1 → 3 внутри AD.
- `exchange.domain` — почтовый домен, обслуживание которого остаётся на Exchange-сервере.
- `domain.local` — домен Active Directory, в котором авторизуются все пользователи.
- `dc.domain.local` — Контроллер домена Active Directory и DNS-сервер для AD.
- `ex.domain.local` — доменное имя Exchange-сервера.

Примеры приведены для случая, когда:

- окружение MyOffice развёрнуто в box-варианте;

- почтовый домен при разворачивании окружения тот же, что и у мигрирующих пользователей;

- служебные логины и пароли на базы данных соответствуют используемым по умолчанию при разворачивании окружения;

## Подготовка к переходу из фазы 1 в фазу 2

1. На DNS-сервере AD должен быть сконфигурирован домен, для мигрирующих пользователей с полным комплектом необходимых записей.
    ```
	DOMAIN migrated.domain
	@	IN	MX	10 ex.domain.local
    ```

2. Разворачивается окружение MyOffice с учётом следующих параметров:

  При конфигурировании NIC в качестве основных DNS-серверов используются сервера AD DNS.

  В файле globals.pp (см. инструкцию по установке) задаются параметры Kerberos realm. Для нашего примера будет так:
    ```
	$krb_realms {
		`domain.local` = ['`dc.domain.local`']
	}
    ```
  После разворачивания MyOffice создаются связанные с AD корпоративные пользователь и группа.

    ```
	cd /usr/local/lib64/perl5/NCT/WebAdminAPI
	./adminapi_script.pl --create_corp_user='login=migration_corp_user@migrated.domain&password=Migr@ti0nP@s$w0rd&tariff=MosReg&ad_url=ldap://ad.domain.local'
	./adminapi_script.pl --add_corp_alias='login=migration_corp_user@migrated.domain&host=migrated.domain&realm=DOMAIN.LOCAL&alias=migrated.domain&base_dn=DC=MIGRATED,DC=LOCAL'
    ```

3. Для инфраструктуры MyOffice создаются соответствующие A-записи в домене `domain.local` на DNS-серверах AD.
    ```
	DOMAIN domain.local
	fedbst	IN	A	внутренний IP-адрес сервера, на котором развёрнуто окружение MyOffice
	be	IN	CNAME	fedbst
	db	IN	CNAME	fedbst
    ```

4. На Echange-сервере проверяется, входят ли IP-адреса BE-серверов в список доверенных front-end-ов (очень сильно зависит от версии Exchange и политик безопасности в компании).
    ```
	Для Exchange 2013 список доверенных сетей и серверов находится на странице 
	Exchange Control Panel (`https://ex.domain.local/ecp/`) 
	"Поток обработки почты" -\> "соединители получения" -\> "Default Frontend EXCHANGE" -\> "определение области".
    ```

5. Из AD выгружается список e-mail-ов мигрирующих пользователей.

6. На AD DNS-сервере создаётся временный домен `tmp-migrated.domain` с единственной записью: `@ IN MX 10 mail.migrated.domain`.

7. В AD для всех пользователей п.5. создаются контакты с почтовыми адресами в домене `tmp-migrated.domain`.

8. В почтовую БД добавляются:

  В таблицу alias_domain - доменный альяс `tmp-migrated.domain` → `migrated.domain`.
    ```
	psql -U vmailadmin -W vmail_box -c "insert into domain_alias values('tmp-migrated.domain','migrated.domain',now(),now(),1)"
    ```

  В таблицу mailbox, для всех почтовых адресов из п.5 - соответствующие записи со значением transport = smtp:`ex.domain.local`
    ```
	psql -U vmailadmin -W vmail_box -c "insert into mailbox set (username,name,domain,transport,active) values('user1@migrated.domain','user1','migrated.domain','smtp:ex.domain.local',1)"
	. . . . .
	psql -U vmailadmin -W vmail_box -c "insert into mailbox set (username,name,domain,transport,active) values('userN@migrated.domain','userN','migrated.domain','smtp:ex.domain.local',1)"
    ```

## Переключение в фазу 2

Переключение производится сменой точки назначения SMTP траффика для внешнего почтового домена `migrated.domain` на систему MyOrffice. В зависимости от имеющейся сетевой конфигурации это может быть проделано либо изменением MX-записи на внешних DNS-серверах для домена `migrated.domain` (самый простой вариант), либо перенаправлением port-mapping-а на коммутаторах силами сетевыми инженерами инфраструктуры ad.domain.local.



## Фаза 2. Миграция.

Процесс миграции одного пользователя запускается (администратором или автоматически) после первого входа мигрирующего пользователя в систему MyOffice со своими авторизационными данными для AD, и состоит из 3-х шагов:

1. В почтовой БД MyOffice из таблицы mailbox удаляется запись для этого пользователя.
    ```
	psql -U vmailadmin -W vmail_box -c "delete from mailbox where username='user1@migrated.domain'"
    ```

2. В AD для этого пользователя в поле altRecipients указывается его контакт в домене `tmp-migrated.domain`.

3. Выполняется перенос данных (письма, контакты, задачи, события и т. д.) этого пользователя из Exchange в MyOffice.

## Переключение на фазу 3

1. В AD DNS для домена migrated.domain MX-запись переназначается на MyOffice.

2. Домен `migrated.domain` удаляется из списка доменов, обслуживаемых Exchange-сервером.

3. У всех контактов мигрировавших пользователей доменная часть адреса меняется на `migrated.domain`.

4. С AD DNS удаляется домен `tmp-migrated.domain`.

5. Из почтовой БД MyOffice удаляется альяс домена `tmp-migrated.domain`.
    ```
	psql -U vmailadmin -W vmail_box -c "delete from alias_domain where domain='tmp-migrated.domain'"
    ```

### Примечание

Поскольку все действия по управлению конфигурацие AD, exchange и AD DNS выполняются через GUI и весьма громоздки в описании, в этом документе они подробно не описаны. Кроме того, предполагается, что специалисты на стороне `ad.domain.local`, администрирующие уже имеющуюся AD + Exchange инфраструктуру обладют достаточной квалификацией, чтобы для описанных операций им требовалась подробная "инструкция для хомячка" с поэкранными снимками.
