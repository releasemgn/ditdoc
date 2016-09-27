### Бэкапирование FS

На клиентском узле с установленной SL6.

1. Установить необходимые зависимости:

```
yum install epel-release
yum install fuse-libs jansson
```

2. Установить драйвер бэкапа из пакета rpm:

```
rpm -ihv backupfs-1.0.0-el6.x86_64.rpm
```
 
3. Убедиться, что хост-клиент backupapi правильно разрешает доменное имя из переменной `"webbackupapi%{::vhost_suffix}"`, указанной в файле конфигурации `web.yaml`, например, `host.domain`:
```
# ping webbackupapi.host.domain -c 1
>>
PING webbackupapi.host.domain (10.0.0.100) 56(84) bytes of data.
64 bytes from webbackupapi.host.domain (10.0.0.100): icmp_seq=1 ttl=64 time=0.470 ms
```
4. Создать точку монтирования файловой системы:
```
mkdir /mnt/fsapi-backup-host.domain
```
 
5. Смонтировать файловую систему:
```
/usr/local/bin/backupfs /mnt/fsapi-backup-host.domain -o key=HqPo8_Ma -o host="webbackupapi.host.domain"
```
 
Утилита backupfs принимает следующие аргументы:
```
-o key=STRING ключ авторизации для backupapi
-o host=HOST:PORT сервер backupapi
-o timeout=NUM длительность кеширования директорий, секунды
-o limit=NUM количество одновременных запросов, по умолчанию 1 (больше делать не рекомендуется)
```

```
# egrep fsapi-backup-host.domain /proc/mounts                                                            
backupfs /mnt/fsapi-backup-host.domain fuse.backupfs rw,nosuid,nodev,relatime,user_id=0,group_id=0 0 0
```

6. Теперь данные из директории /mnt/fsapi-backup-host.domain можно копировать любым удобным способом.


