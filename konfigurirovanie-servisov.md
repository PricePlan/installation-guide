## Конфигурирование сервисов

Для тестовой установки рекомендуется размещать файлы в соответствии с теми путями, которые приводятся в данном руководстве.

### Logstash

Для сбора логов в проекте применяется ELK \(Elasticsearch + Logstash + Kibana\). Как минимум, Logstash нужно сконфигурировать, создав файл:

```
#
# /etc/logstash/pipeline/logstash.conf
#
input {
  udp {
    port =
>
 5045
    codec =
>
 msgpack
  }
}

filter {
  if [type] == "python-logstash" {
    mutate {
      remove_field =
>
 "type"
      add_tag =
>
 "python-logstash"
    }
  }
}

output {
  elasticsearch {
    hosts =
>
 ["http://elasticsearch:9200"]
    user =
>
 "
<
your_elastic_login
>
"
    password =
>
 "
<
your_elastic_password
>
"
  }
}

```

Здесь и далее_переменные_в угловых скобках \("&lt;" и "&gt;"\) нужно заменять на соответствующие значения



