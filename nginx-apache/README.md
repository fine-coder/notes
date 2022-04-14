# Nginx proxy & Apache


## Настройка виртуальных хостов

Создаем директории

```
mkdir /var/www/example.loc /var/www/example.loc/html /var/www/example.loc/log
```

или

```
mkdir -p /var/www/example.loc/html /var/www/example.loc/log
```

### Apache

Создаем директории

```
mkdir /etc/httpd/sites-available /etc/httpd/sites-enabled
```

Откроем файл

```
nano /etc/httpd/conf/httpd.conf
```

Добавим в конец

```
IncludeOptional sites-enabled/*.conf
```

Создадим файл

```
nano /etc/httpd/sites-available/example.loc.conf
```

Содержимое файла [httpd/sites-available/example.loc.conf](httpd/sites-available/example.loc.conf)

Активируем виртуальный хост, для этого создим символьную ссылку в директории sites-enabled

```
ln -s /etc/httpd/sites-available/example.loc.conf /etc/httpd/sites-enabled/example.loc.conf
```

Затем

```
systemctl restart httpd
```

### Nginx

Создаем директории

```
mkdir /etc/nginx/sites-available /etc/nginx/sites-enabled
```

Откроем файл

```
nano /etc/nginx/nginx.conf
```

Добавим в конец

```
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
```

Вторая строка увеличивает объем памяти, выделенный для обработки доменов

Создадим файл

```
nano /etc/nginx/sites-available/example.loc.conf
```

Содержимое файла [nginx/sites-available/example.loc.conf](nginx/sites-available/example.loc.conf)

Активируем виртуальный хост, для этого создим символьную ссылку в директории sites-enabled

```
ln -s /etc/nginx/sites-available/example.loc.conf /etc/nginx/sites-enabled/example.loc.conf
```

Затем

```
systemctl restart nginx
```


## Nginx пользователь и права доступа

### Пользователь

В файле `/etc/nginx/nginx.conf` смотрим

```
user apache;
```

ВАЖНО! Этого же пользователя ставим на директорию `/var/lib/nginx`

```
chown -R apache:apache /var/lib/nginx
```

Иначе, возникнут проблемы, например, с загрузкой файлов на сайте  
Будет ошибка 500 Internal Server Error, а в логе ошибок nginx

```
open() "/var/lib/nginx/tmp/client_body/0000000005" failed (13: Permission denied)
```

На директорию сайта `/var/www/example.loc` ставим этого же пользователя

```
chown -R apache:apache /var/www/example.loc
```

### Права доступа к директориям и файлам

* Директории: 755
* Файлы: 644

Изменить права доступа для файлов

```
find /var/www/example.loc/ -type f -exec chmod 644 {} \;
```

Изменить права доступа для директорий

```
find /var/www/example.loc/ -type d -exec chmod 755 {} \;
```

Ссылки:  
https://firstvds.ru/technology/linux-permissions  
https://ittricks.ru/administrirovanie/linux/896/nastrojka-prav-dostupa-dlya-veb-servera-apache-httpd


## Apache конфиг

### Options, AllowOverride, Require

Options -ExecCGI -Indexes -Includes +FollowSymLinks  
-ExecCGI — не выполнять CGI скрипты  
-Indexes — не выводить содержимое каталога (не показывать посетителю список файлов) при отсутствии index.php или index.html  
-Includes — не разрешать Server-Side Includes, которая представляет собой файлы .shtml, собираемые из других html файлов и cgi-скриптов  
+FollowSymLinks — как следует из названия директивы, Apache сможет переходить по символьным ссылкам

AllowOverride All — указывает, что для корневого каталога виртуального хоста и всех вложенных нужно использовать .htaccess. 
Если с сервера работает одно приложение — один сайт, AllowOverride  лучше выставлять в None и задавать все настройки в конфигурационном файле. 
Это означает несколько большую трудоемкость процесса разворачивания сайта, но позволяет добиться лучшей производительности. Особенно при высоких нагрузках.

Require all granted - разрешение доступа всем без ограничений  
Раньше использовали

```
Order deny,allow
Allow from all
#Deny from all
```

Директивы Allow, Deny, Order модуля mod_access_compat нежелательны к использованию и считаются устаревшими, хотя и поддерживаются еще в версиях Apache 2.3 и 2.4, но в следующих версиях они будут удалены. 
Теперь, вместо устаревших директив Allow, Deny, Order, начиная с версии Apache 2.3, этот функционал реализуется директивой Require из модуля mod_authz_core.

### CustomLog

```
CustomLog /var/www/example.loc/log/httpd-access.log combined
```

combined добавит дополнительные поля
* "http://www.example.com/start.html" ( \"%{Referer}i\") Заголовок HTTP-запроса Referer
* "Mozilla/4.08 [en] (Win98; I ;Nav)" ( \"%{User-agent}i\") Заголовок HTTP-запроса User-Agent

### https

Возможно потребуется в файле `/etc/httpd/conf.d/ssl.conf` закомментировать

```
#Listen 443 https
```


## Nginx конфиг

### https

После добавления https изображения перестали отображаться на сайте, добавил

```
sub_filter http:// https://;
```

### Дополнительно

Конструкции if лучше не использовать

```
if ($host ~ "^www\.(.*)$") {
    return 301 $scheme://$1$request_uri;
}
```

^ - искать с начала строки, а не с любого символа. Выражение `/box/` будет соответствовать и `box-web` и `home-box-web`, а `/^box/` только первому варианту

$ - привязка к концу строки, `/index\.php/` будет соответствовать и `site.ru/index.php` и `site.ru/index.php?v=3`, выражение `/index\.php$/` будет соответствовать только первому варианту

.* - ноль и более любых символов

.+ - один и более любых символов

\. - так как точка специальный символ, то для того, чтобы обозначить точку, ее нужно экранировать


## SELinux

SELinux настраивать не будем, вводим

```
sestatus
```

Если включен, то

```
nano /etc/selinux/config
```

ставим

```
SELINUX=permissive
```

или

```
SELINUX=disabled
```

SELinux имеет три режима работы, по умолчанию установлен режим Enforcing.
* Enforcing: Все действия, которые каким-то образом нарушают текущую политику безопасности, будут блокироваться, а попытка нарушения будет зафиксирована в журнале.
* Permissive: Информация о всех действиях, которые нарушают текущую политику безопасности, будут зафиксированы в журнале, но сами действия не будут заблокированы.
* Disabled: Полное отключение системы принудительного контроля доступа.

Ссылки:  
https://redos.red-soft.ru/base/arm/arm-other/disable-selinux/


## FirewallD

До настройки сертификата в nginx нужно открыть 443 порт  
iptables не нужно настраивать

Вводим

```
firewall-cmd --zone=public --add-port=443/tcp --permanent
```

или

```
firewall-cmd --zone=public --add-service=https --permanent
```

Затем

```
firewall-cmd --reload
```

Проверить изменения

```
firewall-cmd --list-all
```

Получим информацию по зоне public (она по умолчанию)  
По другой зоне

```
firewall-cmd --list-all --zone=home
```

Подробнее  
https://netpoint-dc.com/blog/centos-7-firewalld/


## SSL CentOS 7

Получим обычный сертификат от Let's Encrypt, не WildCard

Установим snapd

```
yum install snapd
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap
```

Ссылки:  
https://snapcraft.io/docs/installing-snapd

Установим certbot

```
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
```

Получим сертификат

Для того, чтобы certbot автоматически отредактировал конфигурацию nginx для обслуживания сертификата, введем

```
certbot --nginx -d example.loc -d www.example.loc
```

Иначе введем

```
certbot certonly --nginx -d example.loc -d www.example.loc
```

Не забываем

```
systemctl restart nginx
```

Сертификаты будут автоматически обновляться, проверить это можно командой

```
systemctl list-timers
```

Сертификат Let's Encrypt можно обновить, если его срок действия истекает менее, чем через 30 дней

!Проверить автоматическое обновление!

Протестировать автоматическое обновление сертификатов

```
certbot renew --dry-run
```

Ссылки:  
https://certbot.eff.org/instructions?ws=nginx&os=centosrhel7

Посмотреть изменения в конфигурации

```
nano /etc/nginx/sites-available/example.loc.conf
```

Отменить изменения в конфигурации

```
certbot --nginx rollback
```

!`rollback` протестировать для конкретного домена!

Ссылки:  
https://eff-certbot.readthedocs.io/en/stable/using.html#nginx

Проверить SSL  
https://www.sslshopper.com/ssl-checker.html

Посмотреть какие порты прослушиваются

```
netstat -plant
```

или конкретный порт

```
netstat -plant | grep --color :443
```
