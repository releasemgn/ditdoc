# Примеры сессий для тестирования почтовой системы на нижнем уровне

## IMAP

### Порт 143 (STARTTLS)

1. Поключаемся к IMAP
```
openssl s_client -starttls imap -connect imap.22-9.nct:143
```
Ответ сервера:
```
CONNECTED(00000003)
depth=1 C = US, O = GeoTrust Inc., CN = GeoTrust SHA256 SSL CA
verify error:num=20:unable to get local issuer certificate
verify return:0
---
Certificate chain
 0 s:/C=RU/ST=Moscow/L=Moscow/O=New Cloud Technologies, Ltd./OU=IT/CN=*.myoffice.ru
   i:/C=US/O=GeoTrust Inc./CN=GeoTrust SHA256 SSL CA
 1 s:/C=US/O=GeoTrust Inc./CN=GeoTrust SHA256 SSL CA
   i:/C=US/O=GeoTrust Inc./OU=(c) 2008 GeoTrust Inc. - For authorized use only/CN=GeoTrust Primary Certification Authority - G3
 2 s:/C=US/O=GeoTrust Inc./CN=GeoTrust SHA256 SSL CA
   i:/C=US/O=GeoTrust Inc./OU=(c) 2008 GeoTrust Inc. - For authorized use only/CN=GeoTrust Primary Certification Authority - G3
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIE+TCCA+GgAwIBAgIQOEQOKX6thboKHnym4KtHRDANBgkqhkiG9w0BAQsFADBG
MQswCQYDVQQGEwJVUzEWMBQGA1UEChMNR2VvVHJ1c3QgSW5jLjEfMB0GA1UEAxMW
R2VvVHJ1c3QgU0hBMjU2IFNTTCBDQTAeFw0xNTA5MjgwMDAwMDBaFw0xNjEyMjQy
MzU5NTlaMHsxCzAJBgNVBAYTAlJVMQ8wDQYDVQQIEwZNb3Njb3cxDzANBgNVBAcU
Bk1vc2NvdzElMCMGA1UEChQcTmV3IENsb3VkIFRlY2hub2xvZ2llcywgTHRkLjEL
MAkGA1UECxQCSVQxFjAUBgNVBAMUDSoubXlvZmZpY2UucnUwggEiMA0GCSqGSIb3
DQEBAQUAA4IBDwAwggEKAoIBAQDFw9F/NwfB0te8U8oEeklutTUu/qMo2zJTrEBG
uRkCffMXnpE2ekQi2pP8xNDjqxvam6HjKX5hP5gfogIs9PjZc4MWGaOPaNmyp7/Q
uUJLAYdZYudZEobQaoYNcVRidfYuA+PHCkApCGqn8qHnub23H8AADOOMhmWsYrIK
157iKBcgbdkUt/zS6Fmgb4QEarDiwM2lhx7htejn41mBI8+nK8ZRTSJehEvT16ZM
izxcOTQhwhuW2UvfF/fyv+sHqW9Xk6iBEh+1zFzz82sez4lChaOlNTfuB4PB2fk/
vobLSIBkpRPm62W8nYXRUh9Rbj1DniIxEFKPjLUa3Gqpe23RAgMBAAGjggGsMIIB
qDAlBgNVHREEHjAcgg0qLm15b2ZmaWNlLnJ1ggtteW9mZmljZS5ydTAJBgNVHRME
AjAAMA4GA1UdDwEB/wQEAwIFoDArBgNVHR8EJDAiMCCgHqAchhpodHRwOi8vZ2ou
c3ltY2IuY29tL2dqLmNybDCBnQYDVR0gBIGVMIGSMIGPBgZngQwBAgIwgYQwPwYI
KwYBBQUHAgEWM2h0dHBzOi8vd3d3Lmdlb3RydXN0LmNvbS9yZXNvdXJjZXMvcmVw
b3NpdG9yeS9sZWdhbDBBBggrBgEFBQcCAjA1DDNodHRwczovL3d3dy5nZW90cnVz
dC5jb20vcmVzb3VyY2VzL3JlcG9zaXRvcnkvbGVnYWwwHQYDVR0lBBYwFAYIKwYB
BQUHAwEGCCsGAQUFBwMCMB8GA1UdIwQYMBaAFBRnju2DT9YenUAEDARGoXA0sg9y
MFcGCCsGAQUFBwEBBEswSTAfBggrBgEFBQcwAYYTaHR0cDovL2dqLnN5bWNkLmNv
bTAmBggrBgEFBQcwAoYaaHR0cDovL2dqLnN5bWNiLmNvbS9nai5jcnQwDQYJKoZI
hvcNAQELBQADggEBABI+VUsD8Gd/T620PfYiDDI1coMVcWrJFeYk8GeHuWpAZobB
JgREqrke1VJG2dJQMern91SXu9Q7ht77S+i5wTzNs9Jh+oGZudmeej8INJW7OWWs
vaVo7ixzE5R+FshAW0izdQXcWSjRV1YjVAlgx/N3MMLHNxnIm3ntbrQ9RbdOc7Nh
fe0okLmiY1H51ZaO/+rzrJBRqZYtvTK/dI6Um6/wg4Ib8raCv1wbYfowCJiY7bCv
+lLWfmyC4nEj6PyWwNAkAmE+JTgBIEXx8jvAvEl1POGOBKZmJ16y9pKW6jA+TLi6
NZF53Ukpb8Ovp9tkeOcDlri3np2CVccOBXmUcEA=
-----END CERTIFICATE-----
subject=/C=RU/ST=Moscow/L=Moscow/O=New Cloud Technologies, Ltd./OU=IT/CN=*.myoffice.ru
issuer=/C=US/O=GeoTrust Inc./CN=GeoTrust SHA256 SSL CA
---
No client certificate CA names sent
---
SSL handshake has read 4891 bytes and written 511 bytes
---
New, TLSv1/SSLv3, Cipher is DHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : DHE-RSA-AES256-GCM-SHA384
    Session-ID: D97424FFF3B5620AFC4FD11E9ABC51CA120F9FF0747870CD29ED26A07E749062
    Session-ID-ctx: 
    Master-Key: FD145C44CFBB4DBE8A1AE6378DD8515EC36E91009BC672E754B46DA3D63706680B0F00D9A91919DF6397E8C54B2A4FE1
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - fd 1a 0b 1b 53 47 9e e7-ca 78 79 81 17 e2 84 cd   ....SG...xy.....
    0010 - 96 73 d3 5e 2c 61 3c fb-6b b3 f0 0a f5 1d 9b ac   .s.^,a<.k.......
    0020 - e9 f0 6c 83 67 e3 a7 e3-f4 d1 b1 7f 37 6e 79 e0   ..l.g.......7ny.
    0030 - 71 be c4 e4 49 aa d7 08-de b9 8a d3 9c 1a 2f b9   q...I........./.
    0040 - d7 06 32 c9 33 d3 a5 cb-0b b6 2d d0 db eb 0a 8c   ..2.3.....-.....
    0050 - 58 dc 3d 7c 02 5c 73 c0-3c 5e c0 aa 70 20 4d 53   X.=|.\s.<^..p MS
    0060 - 1d a3 eb 4a 93 ff 95 be-79 a2 18 de 15 1d a2 39   ...J....y......9
    0070 - c9 b1 68 9a e5 40 e6 46-f8 ad 55 a1 60 3e b6 1c   ..h..@.F..U.`>..
    0080 - d3 82 f6 94 26 67 97 5d-a8 2e e3 32 69 7d f0 61   ....&g.]...2i}.a
    0090 - 82 5f 6d 67 fd a0 a0 a3-27 5f 2c ea df 1c 6c 2e   ._mg....'_,...l.

    Start Time: 1467201318
    Timeout   : 300 (sec)
    Verify return code: 20 (unable to get local issuer certificate)
---
. OK Pre-login capabilities listed, post-login capabilities have more.
```
2. Авторизуемся
```
a001 login qa11@22-9.myoffice.ru "12345678"
```
Ответ сервера:
```
* CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 ESEARCH SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS SPECIAL-USE
a001 OK Logged in
```
3. Переходим на папку "Входящие" (INBOX)
```
a002 select inbox
```
Ответ сервера:
```
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft)] Flags permitted.
* 0 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1467203906] UIDs valid
* OK [UIDNEXT 1] Predicted next UID
* OK [NOMODSEQ] No permanent modsequences
a002 OK [READ-WRITE] Select completed.
```
4. Создаем тестовую папку
```
a003 create "Test_Folder_001"
```
Ответ сервера:
```
a003 OK Create completed.
```
5. Запрашиваем список папок
```
a005 list "" "*"
```
Ответ сервера:
```
* LIST (\HasNoChildren \Drafts) "/" "Drafts"
* LIST (\HasNoChildren) "/" "INBOX"
* LIST (\HasNoChildren \Junk) "/" "Junk"
* LIST (\HasNoChildren \Sent) "/" "Sent"
* LIST (\HasNoChildren) "/" "Test_Folder_001"
* LIST (\HasNoChildren \Trash) "/" "Trash"
a005 OK List completed.
```
6. Отключаемся
```
a006 logout
```
Ответ сервера
```
* BYE Logging out
a006 OK Logout completed.
closed
```

### Порт 993 (SSL)

Отличается только командой на подключение: "openssl s_client -connect imap.22-9.nct:993"; остальное аналогично предыдущему пункту.

