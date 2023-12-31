# quickstarts

Репозиторий содержит готовые конфигурации запуска всей платформы.

## docker compose (local)

### Для запуска в докере необходимо:
1. Убедиться, что порты `20000—20100` свободны
1. Перейти в директорию `00-docker-local`
1. Опционально. Указать в `.env` теги образов приложений (`DENJI_TAG`, `POWER_TAG`, ...)
1. Указать в `.env` абсолютный путь до ssl-сертификата для домена `localhost` (`LOCALHOST_PFX_PATH`) и пароль (`LOCALHOST_PFX_PASSWORD`)
1. Запустить compose `docker compose --env-file .env.local up -d`

### Для проверки работоспособности (oauth 2.0 implicit flow):
1. Создать тестовые данные: `curl -IX POST https://localhost:20001/api/private/configuration/seed`
1. Запустить в [инкогнито-]браузере процесс авторизации, открыв ссылку: `https://localhost:20000/connect/authorize?client_id=implicit&redirect_uri=http%3A%2F%2Flocalhost%3A10000&response_type=token&scope=beam.read&state=abc&nonce=xyz`
1. После регистрации и согласия на доступ к данным получаем в адресной строке access_token

Лучше использовать режим инкогнито, так как приложения создают cookie, которые привязаны к домену localhost и их нужно периодически чистить.

Остановить и удалить всё: `docker compose down -v`

### Как сгенерировать ssl-сертификат для localhost

Один из вариантов — использовать dotnet sdk:
1. Создать сертификат и экспортировать его: `dotnet dev-certs https -ep /Users/dk/.aspnet/https/localhost.pfx -p localhost`
1. Сделать сертификат доверенным: `dotnet dev-certs https --trust`
1. Достать приватный ключ из pfx: `openssl pkcs12 -in [yourfile.pfx] -nocerts -out [yourfile.key]`
1. Расшифровать приватный ключ: `openssl rsa -in [yourfile.key] -out [yourfile-decrypted.key]`
1. Достать сертификат из pfx: `openssl pkcs12 -in [yourfile.pfx] -clcerts -nokeys -out [yourfile.crt]`

В случае проблем, можно удалить сертификаты и затем выполнить команды выше заново: `dotnet dev-certs https --clean`

Так же, можно использовать любой другой вариант генерации сертификатов, например, утилиту `openssl`.

## helm (local)

### Dependencies

1. `docker`
1. `kubernetes`
1. `helm`
1. `nginx-ingress`
1. `cert-manager`

Если вы планируете запускать с предложенными конфигами, то нужно внести изменения в /etc/hosts:

```
# yaiam
127.0.0.1       denji.local
127.0.0.1       power.local
127.0.0.1       cosmo.local
127.0.0.1       beam.local
```

ssl-сертификаты для доменов перечисленных выше будут сгенерированы автоматически, их нужно добавить в список доверенных.

Также необходимо перейти в директорию `01-helm-local` прежде, чем исполнять команды.

### Yaiam

Это мета-чарт, который позволяет сразу развернуть всю систему, в нём указаны только зависимости

1. Установить: `helm install -f yaiam-overrides.yaml yaiam /Users/dk/Repos/yaiam/charts/charts/yaiam`
1. Обновить: `helm upgrade -f yaiam-overrides.yaml yaiam /Users/dk/Repos/yaiam/charts/charts/yaiam`
1. Удалить: `helm uninstall yaiam`

### Denji

1. Установить: `helm install -f denji-overrides.yaml denji /Users/dk/Repos/yaiam/charts/charts/denji`
1. Обновить: `helm upgrade -f denji-overrides.yaml denji /Users/dk/Repos/yaiam/charts/charts/denji`
1. Удалить: `helm uninstall denji`

### Power

1. Установить: `helm install -f power-overrides.yaml power /Users/dk/Repos/yaiam/charts/charts/power`
1. Обновить: `helm upgrade -f power-overrides.yaml power /Users/dk/Repos/yaiam/charts/charts/power`
1. Удалить: `helm uninstall power`

### Cosmo

1. Установить: `helm install -f cosmo-overrides.yaml cosmo /Users/dk/Repos/yaiam/charts/charts/cosmo`
1. Обновить: `helm upgrade -f cosmo-overrides.yaml cosmo /Users/dk/Repos/yaiam/charts/charts/cosmo`
1. Удалить: `helm uninstall cosmo`

### Beam

1. Установить: `helm install -f beam-overrides.yaml beam /Users/dk/Repos/yaiam/charts/charts/beam`
1. Обновить: `helm upgrade -f beam-overrides.yaml beam /Users/dk/Repos/yaiam/charts/charts/beam`
1. Удалить: `helm uninstall beam`
1. Удалить: `helm uninstall cosmo`

### Для проверки работоспособности (oauth 2.0 implicit flow):
1. `kubectl port-forward pod/yaiam-denji-648c96649b-rfhwd 20001:10000`
1. Создать тестовые данные: `curl -IX POST http://localhost:20001/api/private/configuration/seed`
1. Запустить в [инкогнито-]браузере процесс авторизации, открыв ссылку: `https://denji.local/connect/authorize?client_id=implicit&redirect_uri=http%3A%2F%2Flocalhost%3A10000&response_type=token&scope=beam.read&state=abc&nonce=xyz`
1. После регистрации и согласия на доступ к данным получаем в адресной строке access_token

## Contributing

Изменения в проекте приветствуются в соответствии с [правилами](https://github.com/yaiam/.github/blob/main/CONTRIBUTING.md).