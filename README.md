# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»` - Мельник Юрий Александрович


## Задание 1


1. `Запустите два simple python сервера на своей виртуальной машине на разных портах`
2. `Установите и настройте HAProxy, воспользуйтесь материалами к лекции  `
по ссылке -  конфигурационный файл [файла](https://github.com/netology-code/sflt-homeworks/blob/main/2) 
3. `Настройте балансировку Round-robin на 4 уровне`
4. `На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.`
 

 
## Решение 1
1. `Запустим сервера на портах 8888 и 9999`

создадим в папке проэтка папку http1 и скопируем в нее файл
[файл](https://github.com/netology-code/sflt-homeworks/tree/main/2/http1)
перейдем в нее и запусим сервер
```
python3 -m http.server 8888 --bind 0.0.0.0
```

 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1.jpg)  
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_1.jpg)  

создадим в папке проэтка папку http2 и скопируем в нее файл
[файл](https://github.com/netology-code/sflt-homeworks/tree/main/2/http2)
перейдем в нее и запусим сервер
```
python3 -m http.server 9999 --bind 0.0.0.0
```
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_2.jpg)  
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_3.jpg)  

 проверим работоспособность используя curl
 ![alt text](https://github.com/ysatii/balans/blob/main/img/image1_4.jpg)  
 
 
 
 
 
 
 
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


