### Запуск

Чтобы сервис elasticsearch мог работать, перед стартом контейнеров следует убедиться, что директория, куда будет монтироваться его volume с данными, имеет необходимые атрибуты доступа:

```
$ sudo mkdir /opt/esdata
$ sudo chown 1000:1000 /opt/esdata
```

Кроме того, часто бывает необходимо для обеспечения возможности запуска elasticsearch установить следующую настройку ядра Linux:

```
$ sudo sysctl -w vm.max_map_count=262144
$ sudo echo -e "\nvm.max_map_count=262144\n" >> /etc/sysctl.conf
```

После выполнения следующей команды весь стэк PricePlan должен прийти в рабочее состояние:

```
$ docker-compose -p pp -f docker-compose.yaml up -d
```

Остановить \(и удалить контейнеры\), если нужно, можно командой:

```
$ docker-compose -p pp -f docker-compose.yaml down -v
```



