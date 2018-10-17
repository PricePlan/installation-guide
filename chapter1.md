## Загрузка образа PricePlan

Здя загрузки образа PricePlan вам потребуется получить у вашего менеджера логин и пароль для доступа к содержимому репозитория \([sales@priceplan.pro](mailto:sales@priceplan.pro)\). Также желательно уточнить актуальную версию PricePlan.

Используя полученные данные, следует выполнить команды:

```bash
$ docker login -u <your_login> registry.priceplan.pro
```

К сожалению, неоднократно были отмечены проблемы в работе команды`docker login`. При этом после ввода пароля отображается подобное сообщение:

    Error saving credentials: error storing credentials - err: exit status 1, out:
    `Cannot autolaunch D-Bus without X11 $DISPLAY`

Если в вашем случае события получили такое развитие, следует выполнить следующие команды:

```bash
$ mkdir ~/.docker/
$ cat <<EOT >> ~/.docker/config.json
{
  "auths": {
    "registry.priceplan.pro": {
      "auth": ""
    }
  }
}
EOT
```

Затем повторить попытку входа в репозиторий. Если она увенчалась успехом, и пользователь, под которым вы вошли, обладает необходимыми привилегиями, загрузка образа не должна вызвать затруднений.

Для интересующихся:[https://docs.docker.com/engine/reference/commandline/login/\#credentials-store](https://docs.docker.com/engine/reference/commandline/login/#credentials-store)

```bash
$ export PRICEPLAN_VERSION=<actual_version>
$ docker pull registry.priceplan.pro/priceplan/priceplan:${PRICEPLAN_VERSION}
```

По окончании загрузки можно переходить к следующему этапу - конфигурирование сервисов .

