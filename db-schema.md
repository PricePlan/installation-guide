### Создание схемы данных

В завершение остаётся создать объекты данных специальными командами:

```bash
$ docker run -it --rm --name ppshell --network pp_priceplan \
    -e SECRET_KEY="${PRICEPLAN_SECRET_KEY:-dummy_key}" \
    registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION:-1.6.1} \
    shell
```

и после вывода приглашения shell контейнера:

```
(pve) /pp $ PRICEPLAN_ADMIN_PASSWORD=<your_admin_password> \
    ./manage.py create_schema main
```

С помощью переменных окружения` PRICEPLAN_ADMIN_USERNAME` \(по умолчанию priceplan\) и `PRICEPLAN_ADMIN_EMAIL` можно задать логин и адрес электронной почты администратора данной инсталляции АСР. В дальнейшем планируется включить в руководство отдельный раздел, где будут описаны все переменные окружения, определённые для образа. В настоящее время за помощью в данном вопросе следует обращаться к специалистам PricePlan.

Для выхода из командной оболочки контейнера следует использовать стандартную комбинацию «Ctrl + D».

