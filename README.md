# to-do-list-docker
API de tarefas com FastAPI e MySQL rodando no Docker, feita na aula de 19/10 de Arquitetura de Software.
A pasta pratica1_api_simples é a primeira prática (sem banco). A pasta api_mysql é o projeto com MySQL e Adminer.

## Como rodar

Dentro da pasta aula_19-10_docker/api_mysql:
    copy .env.example .env
    docker compose up -d --build
Na primeira vez, criar a coluna concluida:

    docker compose exec db mysql -uappuser -puserpassword tarefas_db -e "ALTER TABLE tarefas ADD COLUMN concluida BOOLEAN NOT NULL DEFAULT FALSE;"

A API fica em http://localhost:8000/docs e o Adminer em http://localhost:8080 (servidor db, usuário appuser, senha userpassword, base tarefas_db).

Para parar: docker compose down
