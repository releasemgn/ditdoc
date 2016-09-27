# Замена ssl-сертификатов

При необходимости замены дефолтных сертификатов на кастомные используется FileServer встроенный в Puppet, для этого:

1) положить сертификат (С) ключ (К) в формате PEM в папку `/etc/puppet/custom_files`

2) заменить значения переменных `ssl_crt_path`, `ssl_key_path` (секция # [Security]) с дефолтных на соответсвенно `puppet:///custom_files/С` и `puppet:///custom_files/К`

3) поместите cert/key в `/etc/puppet/custom_files`

файлы при этом будут доступны в манифестах как

`puppet:///custom_files/C`

`puppet:///custom_files/K`

