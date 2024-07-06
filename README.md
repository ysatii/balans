# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»` - Мельник Юрий Александрович


## Задание 1

### ` `
## ` `

1. ` `
 

 
## Решение 1
1. ` `
 
## Задание 2

 


1. `Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример` конфигурационного  [файла](https://github.com/netology-code/sflt-homeworks/blob/main/1/keepalived-simple.conf) .
2. `Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах`
3. `Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.`
4. `Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script`
5. `Па проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html.`

 



## Решение 2

1. `Создадим Две виртуальные машины и проверим с помощью команды ping видят ли они друг друга по сети`   
 Машина 1  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2.jpg)  
 Машина 2  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_1.jpg)

2. `Установим сервис на обе машины`
 ```
 sudo apt-get install keepalived
 ```  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_2.jpg)

3. `Произвдем настройку конфигурационных файлов`  
 Сервер1 , мастер к виду   
  ```
  vrrp_instance VI_1 {
    	state MASTER
    	interface enp0s3
    	virtual_router_id 15
    	priority 255
    	advert_int 1

    	virtual_ipaddress {
          	10.0.2.100/24
    	}
 }
 
 ```  
  ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_3.jpg)  
 Сервер 2, бэпап к виду   
 ```
 
 vrrp_instance VI_1 {
    	state BACKUP
    	interface enp0s3
    	virtual_router_id 15
    	priority 200
    	advert_int 1

    	virtual_ipaddress {
          	10.0.2.100/24
    	}

 }
 
 ```
 
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_4.jpg)  
 
4. `Произведем запуск сервисов`  
 Сервер 1, мастер  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_5.jpg)  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_6.jpg)  
 
 Сервер 2, бэкап   
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_7.jpg)  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_8.jpg)  
 Заметим что плаващий  10.0.2.100 только на сервер1, но с сервер 2 пинг проходит на 10.0.2.100 
 
5. `Произведем проверку сервиса Keepalived`  
 Остановим сервис на сервер1 и проверим адреса  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_9.jpg)  
 
 Произведем пинг с сервер 1 на плавающий 10.0.2.100  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_10.jpg)  
 
 Сервис на второй машине перешол в состояние мастер
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_11.jpg)  
 
 Запустим сервис снова на первой машине  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_12.jpg) 
 сервер 1 Мастер восстановил свою работоспособность! 
 
6. `установим nginx  на оба сервера!`  
 ```
 sudo apt-get install nginx
 ```
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_13.jpg)  
 
7. `произведем проверку работоспособности nginx, проверим работу сервиса nginx на плавающем ip `
 Изменим страницу по умолчанию для первого сервера, ip 10.0.2.15  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_14.jpg)  
 
 Для второго сервера сервера, ip 10.0.2.4  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_15.jpg)  
 
 Проверим доступность на всех ip адресах
 на сервер 1  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_16.jpg)  
 
 На сервер 2  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_17.jpg)  

 и на плавающем ip  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_18.jpg)  
 Как и ожидалось плавающий ip это сервер 1
 
8. `отслеживание работоспособности проверка состояния nginx.` 
 Приведем конфигурационный файл сервера 1 к виду
 ```
 vrrp_track_process check_nginx {
   	process "nginx"}

 vrrp_instance VI_1 {
    	state MASTER
    	interface enp0s3
    	virtual_router_id 15
    	priority 255
    	advert_int 1

    	virtual_ipaddress {
            	10.0.2.100/24
    	}
    	track_process {
            	check_nginx
    	}
 
 }
 ```
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_19.jpg)  
 
 Приведем конфигурационный файл  сервера 2 к виду
 ```
 vrrp_track_process check_nginx {
   	process "nginx" }

 vrrp_instance VI_1 {
    	state BACKUP
    	interface enp0s3
    	virtual_router_id 15
    	priority 200
    	advert_int 1

    	virtual_ipaddress {
            	10.0.2.100/24
    	}
    	track_process {
               	check_nginx
    	}

 }
 ```
 
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_20.jpg)  
 
 Перезапустим сервисы
 
 ```
 sudo systemctl restart keepalived
 ```
 Сервер 1  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_21.jpg)  

 Сервер 2  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_22.jpg)  
 
 Принудительно остановим nginx на первом сервере 1 и проверим работоспособность
 
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_23.jpg)  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_24.jpg)  
 
 Работоспособность на сервере2 

 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_25.jpg)  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_26.jpg)  
 
 Плавающий IP переместился на сервер 2  
 
9. `отслеживание файла index.html, в случаи его отсутствия осуществляем перенос плающего IP`  
  
 Листинг /home/lamer/check_nginx.sh"
 ```
 #!/bin/bash
 if [[ $(netstat -tuln | grep LISTEN | grep :80) ]] && [[ -f /var/www/html/index.html ]]; then
        exit 0
 else
        exit 1
 fi
 ```
 Приведен файл настроек сервер 1 к виду   
 
 ```
  vrrp_script check_script {
      script "/home/lamer/check_nginx.sh"
      interval 3
  }

#vrrp_track_process check_nginx {
#       process "nginx"}

 vrrp_instance VI_1 {
    	state MASTER
    	interface enp0s3
    	virtual_router_id 15
    	priority 255
    	advert_int 1

    	virtual_ipaddress {
          	10.0.2.100/24
    	}
#        track_process {
#	        check_nginx
        
#        }       
        track_script {
                   check_script
        }
 
 }

 ```  
 Перезапустим сервис   
 
 ```
 sudo systemctl status keepalived
 ```  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_27.jpg)  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_28.jpg)  
 
 Принудительно выключаем ngnix делая недоступным порт 80 на сервер 1  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_29.jpg)  
 
 Запустим nginx, видим что сервис вернул   keepalived свою работоспособность не сервер 1  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_30.jpg)  
 
 Просмиотрим лог  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_31.jpg)  
 
 Удалим файл /var/www/html/index.html   
 Сервер 1, nginx в работе  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_32.jpg)  
 
 Сервер 2 в работе  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_33.jpg)  
 
 При этом плавающий ип стал работать на сервер2  
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_34.jpg)  
 
 Создадим файл index.html и проверим сотояние Keepalived на сервере 1
 ![alt text](https://github.com/ysatii/Keepalived/blob/main/img/image2_35.jpg)  
 
 Работа сервиса возвращена!
 9. `Листинги `  
  -  конфигурационный файл сервер1  [файла](https://github.com/ysatii/Keepalived/blob/main/keepalived.conf_server1) 
  -  конфигурационный файл сервер2  [файла](https://github.com/ysatii/Keepalived/blob/main/keepalived.conf_server2) 
  - скрипт для проверки состояния порта 80, и файла index.html [файла](https://github.com/ysatii/Keepalived/blob/main/check_nginx.sh) 
