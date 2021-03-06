## Список изменений в релизах ПК "МойОфис"

### Изменения в версии ElderFlower 2016.02.p3
* исправлен баг с некорректной работой DNS почтовой службы
* убраны повторяющиеся упоминания об изменении ресурсов corosync
* настройка логов PostgreSQL для работы с syslog
* исправлена проверка состояния и перезапуск Indexer

### Изменения в версии ElderFlower 2016.02.p2
* Новая балансировка для сервера авторизации. Порт 8888
* Удалена привязка к $env в параметре google_redirecturi_nul
* Добавлено описание конфигурирования кол-ва процессов ФС
* Доработан процесс установки

### Изменения в версии ElderFlower 2016.02
* В качестве операционной системы используется Scientific Linux 7
* Обновлениа БД Postgresql до версии 9.5
* Обновлен Openstack Swift до версии 2.6.0
* Изменена структура конфигурации Puppet. Конфигурация разделена на неизменяемую часть, входящую в состав дистрибутива и конфигурируемую при инсталляции, уникальныю для каждой инсталляции.
* Обновление EWS Для корректной работы требуется библиотека EWS версии не ниже 1.1.12a

### Изменения в версии 4.4.7
* Исправлена уязвимость ImageMagic (CVE-2016-3714)
* Исправлена возможность кражи сессии

### Изменения в версии 4.4 (Dill Seed Update)
* Обновление EWS
  Для корректной работы требуется библиотека EWS версии не ниже 1.1.11
* Изменнена логика обработки параметра `logs_node` в `manifests/globals.pp` -- теперь требуется указывать полное имя сервера (FQDN) на который требуется отправлять логи.
  Например, `logs_node = "logs.mydomain.tld,logs.otherdomain.ltd"`
* Изменены рекомендации по разбивке диска для серверов роли `st`. Также изменено значение по умолчанию параметра `swift_ring_device` в `manifests/globals.pp`
  Рекоменуется использование нескольких дисков под данные swift, перечисленных в параметре `swift_ring_device` через запятую.
* Добавлен новый сервис `poolmon` на ноды кластера с ролью `lb`. Сервис предназначен для отслеживания состояния сервиса `dovecot` в кластере.
* Данные по часовым поясам актуализированы по состоянию на март 2016 года.
* Обновлены файлы брендирования: актуальные версии - '1.8-1.el6' для бренда 'myoffice.ru' и '1.8col-1.el6' для 'collabio.com'
* Добавлена [документация](adminapi_script_guide) по административным операциям при работе с установленным ПО "МойОфис"
* Добавлена [информация](preliminary#dns_setup) по конфигурированию внешней DNS-зоны сервиса.
  При обновлении уже развернутых сервисов на базе ПК "МойОфис" требуется обратить внимание на появившееся требование по SRV-записям для сервиса caldav

