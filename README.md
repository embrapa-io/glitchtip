# GlitchTip for Embrapa I/O

Configuração de deploy do [GlitchTip](https://glitchtip.com) (_error tracking_) no ecossistema do Embrapa I/O.

Baseado na [configuração de _deploy_ do GlitchTip usando Docker Compose](https://glitchtip.com/documentation/install#docker-compose).

## Deploy

```
docker volume create glitchtip_db_data
docker volume create glitchtip_up_data

cp .env.example .env

docker-compose up --force-recreate --build --remove-orphans --wait
```

## Update

```
docker-compose pull && docker-compose stop && docker-compose up -d
```
