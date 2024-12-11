# GlitchTip for Embrapa I/O

Configuração de deploy do [GlitchTip](https://glitchtip.com) (_error tracking_) no ecossistema do Embrapa I/O.

Baseado na [configuração de _deploy_ do GlitchTip usando Docker Compose](https://glitchtip.com/documentation/install#docker-compose).

## Deploy

```
docker volume create glitchtip_database
docker volume create --driver local --opt type=none --opt device=$(pwd)/upload --opt o=bind glitchtip_upload
docker volume create --driver local --opt type=none --opt device=$(pwd)/backup --opt o=bind glitchtip_backup

cp .env.example .env

docker-compose up --force-recreate --build --remove-orphans --wait
```

## Configuração

É necessário [criar um _superuser_](https://glitchtip.com/documentation/install#django-admin) com o e-mail `io@embrapa.br`:

```
# docker-compose exec web bash

$ ./manage.py createsuperuser
```

Em seguida, logue com este usuário e senha e, uma vez logado, vá em `/admin`.

## Update

```
docker-compose pull && docker-compose stop && docker-compose up -d
```
