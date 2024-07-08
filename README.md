# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»` - Мельник Юрий Александрович


## Задание 1


1. `Запустите два simple python сервера на своей виртуальной машине на разных портах`
2. `Установите и настройте HAProxy, воспользуйтесь материалами к лекции  `
по ссылке -  конфигурационный файл [файла](https://github.com/netology-code/sflt-homeworks/blob/main/2) 
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
 дает ошибку в стоке 55 на моей версии операционной истсемы  

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
        balance roundrobin
        server s1 127.0.0.1:8888 check inter 3s weight 1
	    server s2 127.0.0.1:9999 check inter 3s weight 2
```
проверим работу haproxy
```
sudo systemctl reload haproxy.service
sudo systemctl status haproxy.service
```
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_5.jpg)   
 
## Задание 2
1. `Запустите три simple python сервера на своей виртуальной машине на разных портах` 
2. `Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4`
3. `HAproxy должен балансировать только тот http-трафик, который адресован домену example.local`
4. `На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.`
 
 



## Решение 2

1. `Создадим Две виртуальные машины и проверим с помощью команды ping видят ли они друг друга по сети`   
 Машина 1  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2.jpg)  
 Машина 2  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_1.jpg)


