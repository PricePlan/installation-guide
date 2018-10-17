## Запуск 

### Генерация секретного ключа

Сгенерируйте случайную строку, которая будет выступать в качестве секретного ключа, следующей командой:

```bash
$ python -c "import string, random; uni=string.ascii_letters+string.digits+string.punctuation; print repr(''.join([random.SystemRandom().choice(uni) for i in range(random.randint(45,50))]))"
```

### Редактирование docker-compose.yaml

Полученное значение рекомендуется экспортировать в переменную окружения:

```bash
$ export PRICEPLAN_SECRET_KEY=<your_value>
```

Далее следует в домашней директории пользователя, который входит в группу docker, создать файл

`docker-compose.yaml`:

```yaml
#
# /home/<your_user_name>/docker-compose.yaml
#
version: "3"
services:

  redis:
    image: redis:4-alpine
    expose:
      - 6379
    networks:
      - priceplan

  postgres:
    image: postgres:10-alpine
    expose:
      - 5432
    volumes:
      - /opt/PG/10/data/:/var/lib/postgresql/data/
      - /tmp:/tmp
    networks:
      - priceplan
    environment:
      POSTGRES_USER: priceplan
      POSTGRES_PASSWORD: priceplan

  rabbitmq:
    image: rabbitmq:3.7-alpine
    expose:
      - 5672
    networks:
      - priceplan
    volumes:
      - /var/lib/rabbitmq/:/var/lib/rabbitmq/
    environment:
      RABBITMQ_DEFAULT_USER: priceplan
      RABBITMQ_DEFAULT_PASS: priceplan

  elasticsearch2:
    image: elasticsearch:2.4-alpine
    expose:
      - 9200
    networks:
      - priceplan
    volumes:
      - /opt/esdata2/:/usr/share/elasticsearch/data/

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:${ELK_VERSION:-6.4.2}
    expose:
      - 9200
    networks:
      - priceplan
    volumes:
      - /opt/esdata/:/usr/share/elasticsearch/data/:rw
    environment:
      "discovery.type": single-node

  logstash:
    image: docker.elastic.co/logstash/logstash:${ELK_VERSION:-6.4.2}
    depends_on:
      - elasticsearch
    expose:
      - '5045/udp'
    networks:
      - priceplan
    volumes:
      - /etc/logstash/pipeline:/usr/share/logstash/pipeline:ro

  kibana:
    image: docker.elastic.co/kibana/kibana:${ELK_VERSION:-6.4.2}
    depends_on:
      - elasticsearch
    expose:
      - 5601
    networks:
      - priceplan

  uwsgi:
    image: registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1}
    depends_on:
      - postgres
      - redis
      - rabbitm bash
      - elastic bashearch2
      - elasticsearch
      - logstash
    expose:
      - 8001
    networks:
      - priceplan
    environment:
      SECRET_KEY: ${PRICEPLAN_SECRET_KEY:-dummy_key}
    volumes:
      - static:/static
    command: ["uwsgi", ]

  nginx:
    image: nginx:1.15-alpine
    depends_on:
      - uwsgi
      - kibana
      - flower
    ports:
      - 80:80
    networks:
      - priceplan
    volumes:
      - /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - /etc/nginx/sites-available/:/etc/nginx/sites-available/:ro
      - /etc/nginx/sites-enabled/:/etc/nginx/sites-enabled/:ro
      - /var/cache/nginx/:/var/cache/nginx/:rw
      - /var/log/nginx/:/var/log/nginx/:rw
      - static:/static

  cworker:
    image: registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1}
    depends_on:
      - postgres
      - redis
      - rabbitmq
      - elasticsearch2
      - logstash
    networks:
      - priceplan
    environment:
      SECRET_KEY: ${PRICEPLAN_SECRET_KEY:-dummy_key}
    command: ["celery", ]

  billing:
    image: registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1}
    depends_on:
      - postgres
      - redis
      - rabbitmq
      - elasticsearch2
      - logstash
    networks:
      - priceplan
    environment:
      SECRET_KEY: ${PRICEPLAN_SECRET_KEY:-dummy_key}
    command: ["billing", ]

  ordinal:
    image: registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1}
    depends_on:
      - postgres
      - redis
      - rabbitmq
      - elasticsearch2
      - logstash
    networks:
      - priceplan
    environment:
      SECRET_KEY: ${PRICEPLAN_SECRET_KEY:-dummy_key}
    command: ["ordinal", ]

  haystack:
    image: registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1}
    depends_on:
      - postgres
      - redis
      - rabbitmq
      - elasticsearch2
      - logstash
    networks:
      - priceplan
    environment:
      SECRET_KEY: ${PRICEPLAN_SECRET_KEY:-dummy_key}
    command: ["haystack", ]

  master:
    image: registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1}
    depends_on:
      - postgres
      - redis
      - rabbitmq
      - elasticsearch2
      - logstash
    networks:
      - priceplan
    environment:
      SECRET_KEY: ${PRICEPLAN_SECRET_KEY:-dummy_key}
    command: ["master", ]

  flower:
    image: registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1}
    depends_on:
      - postgres
      - redis
      - rabbitmq
      - elasticsearch2
      - logstash
    expose:
      - 5555
    networks:
      - priceplan
    environment:
      SECRET_KEY: ${PRICEPLAN_SECRET_KEY:-dummy_key}
    command: ["flower", ]

networks:
  priceplan:

volumes:
  static:
```

### Запуск

Чтобы сервис elasticsearch мог работать, перед стартом контейнеров следует убедиться, что директория, куда будет монтироваться его volume с данными, имеет необходимые атрибуты доступа:

```bash
$ sudo mkdir /opt/esdata
$ sudo chown 1000:1000 /opt/esdata
```

Кроме того, часто бывает необходимо для обеспечения возможности запуска elasticsearch установить следующую настройку ядра Linux:

```bash
$ sudo sysctl -w vm.max_map_count=262144
$ sudo echo -e "\nvm.max_map_count=262144\n" >> /etc/sysctl.conf
```

После выполнения следующей команды весь стэк PricePlan должен прийти в рабочее состояние:

```bash
$ docker-compose -p pp -f docker-compose.yaml up -d
```

Остановить \(и удалить контейнеры\), если нужно, можно командой:

```bash
$ docker-compose -p pp -f docker-compose.yaml down -v
```



