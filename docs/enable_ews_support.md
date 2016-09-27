# Поддержка EWS

Для включения в сервисе Myoffice поддержки протокола EWS необходимо

* Скачать файлы Lite.pm Lite.so7 из библиотеки libews-1.1.12a
* Положить эти файлы в папку `/etc/puppet/custom_files`
* Установить в файле `/etc/puppet/hieracustom/common.yaml` в секции `[Mail]` значение переменной `install_ews: true`