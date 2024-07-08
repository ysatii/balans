# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»` - Мельник Юрий Александрович


## Задание 1


1. `Запустите два simple python сервера на своей виртуальной машине на разных портах`
2. `Установите и настройте HAProxy, воспользуйтесь материалами к лекции  `
по ссылке -   [конфигурационный файл](https://github.com/netology-code/sflt-homeworks/blob/main/2) 
3. `Настройте балансировку Round-robin на 4 уровне`
4. `На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.`
 

 
## Решение 1
1. `Запустим сервера на портах 8888 и 9999`

создадим в папке проэтка папку **http1** и скопируем в нее файл
[файл](https://github.com/netology-code/sflt-homeworks/tree/main/2/http1)
перейдем в нее и запусим сервер
```
python3 -m http.server 8888 --bind 0.0.0.0
```

 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1.jpg)  
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_1.jpg)  

создадим в папке проэтка папку **http2** и скопируем в нее файл
[файл](https://github.com/netology-code/sflt-homeworks/tree/main/2/http2)
перейдем в нее и запусим сервер
```
python3 -m http.server 9999 --bind 0.0.0.0
```
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_2.jpg)  
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_3.jpg)  

 проверим работоспособность используя curl
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_4.jpg)  
 
2. `установка HAProxy`  
```
sudo apt-get install haproxy
```
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_5.jpg)  
 
 Конфигурационный файл https://github.com/netology-code/sflt-homeworks/blob/main/2/haproxy/haproxy.cfg  
 выдает ошибку в стоке 55 на моей версии операционной и сервисе haproxy  

 haproxy - Version: 2.0.33-0ubuntu0.1  

 операционная система   
 No LSB modules are available.  
 Distributor ID:	Ubuntu  
 Description:	Ubuntu 20.04.6 LTS  
 Release:	20.04  
 Codename:	focal  
 
приведем конфигурационный файл /etc/haproxy/haproxy.cfg к виду  
```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
        #default_backend web_servers
	acl ACL_example.com hdr(host) -i example.com
	use_backend web_servers if ACL_example.com

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk GET /index.html  HTTP/1.1
        http-check send hdr Content-Type html/text 
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check


listen web_tcp

	bind :1325
        server s1 127.0.0.1:8888 check inter 3s weight 1
	server s2 127.0.0.1:9999 check inter 3s weight 2
```
проверим работу haproxy
```
sudo systemctl reload haproxy.service
sudo systemctl status haproxy.service
```
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_6.jpg)   
 
3. `Настройте балансировку Round-robin на 4 уровне`
в секции  **listen web_tcp** пропишем строку **balance roundrobin**
приведем конфигурационный файл /etc/haproxy/haproxy.cfg к виду 
```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
        #default_backend web_servers
	acl ACL_example.com hdr(host) -i example.com
	use_backend web_servers if ACL_example.com

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk GET /index.html  HTTP/1.1
        http-check send hdr Content-Type html/text 
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check


listen web_tcp
       balance roundrobin
	bind :1325
        server s1 127.0.0.1:8888 check inter 3s weight 1
	server s2 127.0.0.1:9999 check inter 3s weight 2
```



Перечитаем конфигурационный файл haproxy и посмотрим его состояние 
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_7.jpg)   


Балансировка на уровне 4 работает на порту 1325 , согласно установленных весов  
listen web_tcp  

        bind :1325  
        balance roundrobin  
        server s1 127.0.0.1:8888 check inter 3s weight 1  
        server s2 127.0.0.1:9999 check inter 3s weight 2  
		
127.0.0.1:8888 вес 1  
127.0.0.1:9999 вес 2  

![alt text](https://github.com/ysatii/balans/blob/main/img/image1_8.jpg)  


4. `принудительно выключим веб сервер на порту 8888`  
 и посмотрим стаистику haproxy в веб интервейсе на порту 888  
  
 видим что сервер s1 исключем из балансировки и помечен красным  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_9.jpg)  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_10.jpg)  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_11.jpg)  

 проверим командой на какие сервера осуществляеться переброс трафика, видим что остался только сервер на порту 9999  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_13.jpg)  

 востановим работоспособность s1
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_12.jpg)  


 
 
  
 
5. `Ссылка  используемоый конфигурационный файла`

[Конфигурационный файл](https://github.com/ysatii/balans/blob/main/haproxy.cfg) 

 
## Задание 2
1. `Запустите три simple python сервера на своей виртуальной машине на разных портах` 
2. `Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4`
3. `HAproxy должен балансировать только тот http-трафик, который адресован домену example.local`
4. `На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.`
 
 



## Решение 2

1. `Запустим третий сервер на python порту 7777`  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_13.jpg)  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_14.jpg)  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_15.jpg)  

Приводим конфиг к виду /etc/haproxy/haproxy.cfg

В секции frontend example прописываем 
    acl ACL_example.local hdr(host) -i example.local
	use_backend web_servers if ACL_example.local
	
В секции web_servers дописываем стороку третий сервер
    server s3 127.0.0.1:7000 chek 
```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
        #default_backend web_servers
	    acl ACL_example.local hdr(host) -i example.local
	    use_backend web_servers if ACL_example.local

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk GET /index.html  HTTP/1.1
        http-check send hdr Content-Type html/text
        server s1 127.0.0.1:8888 check inter 3s weight 2
        server s2 127.0.0.1:9999 check inter 3s weight 3
        server s3 127.0.0.1:7000 check inter 3s weight 4

listen web_tcp

	bind :1325
        balance roundrobin
        server s1 127.0.0.1:8888 check inter 3s weight 1
    	server s2 127.0.0.1:9999 check inter 3s weight 2
```

Перечитаем конфиг !  
```
curl -H 'Host:example.ru' http://127.0.0.1:8088
```
Проверим балансировку трафика  
балансировка только домена example.local  

Во всех остальных случая ошибка 503   



![alt text](https://github.com/ysatii/balans/blob/main/img/image1_16.jpg)  

Видим балансировку трафика на все три сервера согласно спрописанных весов при балансировке
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_17.jpg)  

Посмотрим симстему мониторинга HAProxy  
![alt text](https://github.com/ysatii/balans/blob/main/img/image1_18.jpg)  
Видим третий север появившийся в балансировке!


## Задание 3

1. `Настройте связку HAProxy + Nginx как было показано на лекции.`
2. `Настройте Nginx так, чтобы файлы .jpg выдавались самим Nginx (предварительно разместите несколько тестовых картинок в директории /var/www/), а остальные запросы переадресовывались на HAProxy, который в свою очередь переадресовывал их на два Simple Python server.`
3. `На проверку направьте конфигурационные файлы nginx, HAProxy, скриншоты с запросами jpg картинок и других файлов на Simple Python Server, демонстрирующие корректную настройку.`
## Решение 3
1. `Выработаем схему работы серверов`
![alt text](https://github.com/ysatii/balans/blob/main/img/image3.jpg)  

2. `установка nginx`
```
apt-get install nginx
```

![alt text](https://github.com/ysatii/balans/blob/main/img/image3_1.jpg)  

3. `Настройка nginx`
по ссылке -   [конфигурационный файл](https://github.com/netology-code/sflt-homeworks/blob/main/2)  Скачаем и установим конфигурационные файлы   

листинг /etc/nginx/nginx.conf

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}


stream {
	include /etc/nginx/include/upstream.inc;
	server	{
		listen 8080;
		
		error_log	/var/log/nginx/example-tcp-error.log;
		proxy_pass	example_app;
	}
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
```
листинг /etc/nginx/conf.d/example-http.conf
```
include 

server {
   listen	80;
   

   server_name	example-http.com;
   

   access_log	/var/log/nginx/example-http.com-acess.log;
   error_log	/var/log/nginx/example-http.com-error.log;

   location / {
		proxy_pass	http://example_app;

   }

}

```

листинг /etc/nginx/include/upstream.inc
```
upstream example_app {

	server 127.0.0.1:8888 weight=3;
        server 127.0.0.1:9999;

}
```
4. `Проверим порт 80 nginx`

```
curl -H 'Host:example-http.com' http://127.0.0.1:80
```

Видим балансировку 7 уровня  
![alt text](https://github.com/ysatii/balans/blob/main/img/image3_2.jpg)  

при использовании других адресов ngnix выдаст стандартную парковочную страницу  
![alt text](https://github.com/ysatii/balans/blob/main/img/image3_3.jpg)  

5. `запросим файл image1.jpg`  
  листинг /etc/nginx/conf.d/example-http.conf  
``` 
 include /etc/nginx/include/upstream.inc;

 server {
   listen	80;
   

   server_name	example-http.com;
   

   access_log	/var/log/nginx/example-http.com-acess.log;
   error_log	/var/log/nginx/example-http.com-error.log;

   location / {
		proxy_pass	http://example_app;

   
   
       location ~ "\.(jpg|jpeg|gif|png|ico)$" {
                root /var/www;
           }
    }
}

```

   файл находиться по пути /var/www/image1.jpg  
   
   ![alt text](https://github.com/ysatii/balans/blob/main/img/image3_4.jpg)  
   
   получим сообщение что файл бинарный!  
   
   Скачаем его и просмотрим в графической оболочке  
   ![alt text](https://github.com/ysatii/balans/blob/main/img/image3_5.jpg)  
   
настройки nginx представлены по ссылке![alt text](https://github.com/ysatii/balans/tree/main/nginx )  
   
   

## Задание 4

1. `Запустите 4 simple python сервера на разных портах.`
2. `Первые два сервера будут выдавать страницу index.html вашего сайта example1.local (в файле index.html напишите example1.local)`
3. `Вторые два сервера будут выдавать страницу index.html вашего сайта example2.local (в файле index.html напишите example2.local)`
4. `Настройте два бэкенда HAProxy`
5. `Настройте фронтенд HAProxy так, чтобы в зависимости от запрашиваемого сайта example1.local или example2.local запросы перенаправлялись на разные бэкенды HAProxy`
6. `На проверку направьте конфигурационный файл HAProxy, скриншоты, демонстрирующие запросы к разным фронтендам и ответам от разных бэкендов.`

## Решение 4

1. `example1.local и example2.local`
    example1.local сервеара 1 и 2 порты 8888 и 9999
    example2.local сервеара 3 и 4 порты 7777 и 6666
	
	
листинг /etc/haproxy/haproxy.cfg

```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
        #default_backend web_servers
	acl ACL_example1.local hdr(host) -i example1.local
	use_backend web_servers if ACL_example1.local

        acl ACL_example2.local hdr(host) -i example2.local
        use_backend web_servers2 if ACL_example2.local

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk GET /index.html  HTTP/1.1
        http-check send hdr Content-Type html/text 
        server s1 127.0.0.1:8888 check inter 3s weight 1
        server s2 127.0.0.1:9999 check inter 3s weight 1
        
backend web_servers2    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk GET /index.html  HTTP/1.1
        http-check send hdr Content-Type html/text
        server s3 127.0.0.1:6666 check inter 3s weight 1
        server s4 127.0.0.1:7777 check inter 3s weight 1



listen web_tcp

	bind :1325
        balance roundrobin
        server s1 127.0.0.1:8888 check inter 3s weight 1
	server s2 127.0.0.1:9999 check inter 3s weight 2
```


	
	
	
 перзапустим HAProxy
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image4.jpg)  
	
 посмотрим интерфейс 
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image4_1.jpg)  
	
 выполним запросы к порту 8088 с указанирем доменных имен
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image4_2.jpg)  
	
 запустим 4 web сеервер на порту 6666   
 убедимся что все сервера в работе
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image4_3.jpg) 
