# PHP

## php.ini

```
nano /etc/php.ini
```

```
systemctl restart httpd
```

### limits

```
memory_limit = 2048M
post_max_size = 1024M
upload_max_filesize = 1024M
max_execution_time = 120
max_input_time = 120
```

### timezone

```
date.timezone = Asia/Irkutsk
```

### short_open_tag

```
short_open_tag = On
```

Определяет, разрешается ли короткая форма записи (<? ?>) тегов PHP. 
Если же эта опция отключена, мы должны использовать длинную форму открывающего тега PHP (<?php ?>). 
Эта директива не влияет на сокращение <?=, которое всегда доступно.

### mysql.connect_timeout

Время ожидания ответа до разрыва соединения в секундах. 
Linux также использует это значение при ожидании первого ответа от сервера.

```
mysql.connect_timeout = 60
```

`mysql.connect_timeout` сообщает PHP, как долго он должен ждать ответа от сервера MySQL при попытке подключения. 
`connect_timeout` в конфигурации MySQL сообщает серверу MySQL, как долго ждать пакета подключения от клиента, прежде чем ответить ошибкой `Bad handshake`. 
Apache не участвует ни в одном из этих тайм-аутов, они просто между PHP и MySQL. 

Ссылки:  
https://stackoverflow.com/questions/25378533/mysql-connect-timeout-in-php-vs-connect-timeout-in-mysql
