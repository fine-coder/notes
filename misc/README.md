# Разное

## Установка Adminer в CentOS 7

Лучше использовать Database & Terminal в PhpStorm, а не Adminer

Создадим директорию

```
mkdir /usr/share/adminer && cd /usr/share/adminer
```

Скачаем последнюю версию Adminer  
https://www.adminer.org/#download

```
wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1.php -O index.php
```

Создадим файл конфигурации

```
nano /etc/httpd/conf.d/adminer.conf
```

Добавим

```
Alias /adminer "/usr/share/adminer"
<Directory "/usr/share/adminer">
    Options -ExecCGI -Indexes -Includes -FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

Перезапускаем Apache

```
systemctl restart httpd
```

Открываем Adminer

```
http://example.loc/adminer
```

Ссылки:  
https://www.crybit.com/install-adminer-centos-server/
