# Домашнее задание к занятию «Кластеризация и балансировка нагрузки» - 'Пшённов Николай'

## Задание 1
`Запустите два simple python сервера на своей виртуальной машине на разных портах`
`Установите и настройте HAProxy, воспользуйтесь материалами к лекции`
`Настройте балансировку Round-robin на 4 уровне.`
`На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.`

### Решение

* `Запускаю два сервера`
```
mkdir ~/http1
cd ~/http1
nano index.html -> Server 1 :8888
python3 -m http.server 8888 --bind 0.0.0.0
```
`открываю еще одно окно терминала`
```
mkdir ~/http2
cd ~/http2
nano index.html -> Server 2 :9999
python3 -m http.server 9999 --bind 0.0.0.0
```

* `Устанавливаю и конфигурирую HAproxy`
```
sudo apt install haproxy
sudo nano /etc/haproxy/haproxy.cfg
```

* `В файл конфигурации добавляю:`
```
listen stats 
        bind :888
        mode http 
        stats enable 
        stats uri /stats 
        stats refresh 5s 
        stats realm Haproxy\ Statistics

frontend example
        mode tcp
        bind :8088
        default_backend web_servers

backend web_servers
        mode tcp
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check
```
![Конфигурация](https://github.com/pshennov/sflt-haproxy/blob/main/images/conf1.png)
[Файл](https://github.com/pshennov/sflt-haproxy/blob/main/file/haproxy1.cfg)

* `Перезапускаю HAproxy, проверяю результат`
```
sudo systemctl reload haproxy
curl http://127.0.0.1:8088
```
![Результат](https://github.com/pshennov/sflt-haproxy/blob/main/images/zadanie1.png)

* `Дополнительно проверяю статистику`
![Статистика](https://github.com/pshennov/sflt-haproxy/blob/main/images/stat1.png)

## Задание 2
`Запустите три simple python сервера на своей виртуальной машине на разных портах`
`Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4`
`HAproxy должен балансировать только тот http-трафик, который адресован домену example.local`
`На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.`

### Решение

* `Добавляю еще один сервер к двум имеющимся`
```
mkdir ~/http3
cd ~/http3
nano index.html -> Server 3 :7777
python3 -m http.server 7777 --bind 0.0.0.0
```

* `Изменяю файл конфигурации:`
```
listen stats
        bind :888
        mode http
        stats enable
        stats uri /stats
        stats refresh 5s
        stats realm Haproxy\ Statistics

frontend example
        mode http
        bind :8088
        acl ACL_example.local hdr(host) -i example.local # фильтр по хосту
        use_backend web_servers if ACL_example.local

backend web_servers
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check weight 2
        server s2 127.0.0.1:9999 check weight 3
        server s3 127.0.0.1:7777 check weight 4
```

![Конфигурация](https://github.com/pshennov/sflt-haproxy/blob/main/images/conf2.png)
[Файл](https://github.com/pshennov/sflt-haproxy/blob/main/file/haproxy.cfg)

* `Перезапускаю HAproxy, проверяю результат`
```
sudo systemctl reload haproxy
curl -H 'Host:example.local' http://127.0.0.1:8088
```
![Результат](https://github.com/pshennov/sflt-haproxy/blob/main/images/zadanie2.png)

* `Проверяю без использования домена example.local`
![Результат](https://github.com/pshennov/sflt-haproxy/blob/main/images/error.png)

* `Дополнительно проверяю статистику`
![Статистика](https://github.com/pshennov/sflt-haproxy/blob/main/images/stat2.png)

---
