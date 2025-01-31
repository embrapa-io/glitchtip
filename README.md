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

## Backup e Restore

O _backup_ é feito de forma automática pelo serviço `backup` da _stack_ de containers. Para isso, é utilizado a imagem [prodrigestivill/postgres-backup-local](https://hub.docker.com/r/prodrigestivill/postgres-backup-local).

Para o _restore_ é recomendado executar o seguinte procedimento:

1. Desligue todos os containers:

```
docker-compose stop
```

2. Crie um novo volume para o DB (sem apagar o anterior):

```
docker volume create glitchtip_database2
```

**Atenção!** Não se esqueça de alterar o nome na variável `VOLUME_DB` do `.env`.

3. Agora suba apenas o container do banco de dados:

```
docker-compose up --force-recreate --build --remove-orphans --wait db
```

4. Por fim, execute a restauração utilizando um container independente do PostgreSQL:

```
docker run --rm --tty --interactive -v $BACKUPFILE:/tmp/backupfile.sql.gz --network=$NETWORK postgres:$VERSION /bin/sh -c "zcat /tmp/backupfile.sql.gz | psql --host=db --port=5432 --username=$DB_USER --dbname=glitchtip -W"
```

O `$BACKUPFILE` será o nome do arquivo de _backup_ a ser restaurado. Veja dentro do diretório `backup/` qual é o mais apropriado (p.e., `./backup/last/glitchtip-latest.sql.gz`).

É necessário que `$NETWORK` seja a rede da _stack_ de containers (normalmente será `glitchtip_default`). É possível verificar com o comando `docker network ls`.

A variável `$VERSION` deve refletir a mesma versão utilizada pela imagem `postgres` no `docker-compose.yaml`.

O valor da variável `DB_USER` está no `.env` e, ao executar, será pedida a senha do DB, que está na variável `DB_PASSWORD` também no `.env`.
