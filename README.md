# To-Do List com Docker - Arquitetura de Software

Projeto de API de gerenciamento de tarefas (To-Do List) desenvolvido com **FastAPI**, **MySQL** e **Adminer**, containerizado com **Docker** e orquestrado via **Docker Compose**.

---

## Estrutura do Repositório

- `aula_19-10_docker/api_mysql/`: **(Projeto Principal)** API com persistência no MySQL e interface web Adminer.
- `aula_19-10_docker/pratica1_api_simples/`: **(Prática 1)** API simples em memória usando apenas Dockerfile (sem banco de dados).
- `aula_19-10_docker/postman/`: Coleção e variáveis de ambiente prontas para testes via Postman.

---

## Como Rodar o Projeto Principal (`api_mysql`)

### 1. Pré-requisitos
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado e em execução no computador.

### 2. Configurar as Variáveis de Ambiente
O arquivo `compose.yaml` requer 4 variáveis de ambiente para configurar o banco de dados MySQL:
- `MYSQL_ROOT_PASSWORD`
- `MYSQL_DATABASE`
- `MYSQL_USER`
- `MYSQL_PASSWORD`

Crie um arquivo chamado `.env` dentro da pasta `aula_19-10_docker/api_mysql/` (você pode simplesmente copiar o modelo `.env.example` existente):

```powershell
cd "aula_19-10_docker/api_mysql"
Copy-Item .env.example .env
```

Conteúdo esperado do `.env`:

```env
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=tarefas_db
MYSQL_USER=appuser
MYSQL_PASSWORD=userpassword
```

> **Dica (PowerShell):** Se preferir não criar o arquivo `.env`, você pode definir as variáveis temporariamente no terminal antes de subir os containers:
> ```powershell
> $env:MYSQL_ROOT_PASSWORD="rootpassword"
> $env:MYSQL_DATABASE="tarefas_db"
> $env:MYSQL_USER="appuser"
> $env:MYSQL_PASSWORD="userpassword"
> ```

### 3. Iniciar a Aplicação
No terminal, navegue até a pasta do projeto e inicie os containers:

```powershell
cd "aula_19-10_docker/api_mysql"
docker compose up --build
```
*(Para rodar em segundo plano, adicione a flag `-d`: `docker compose up --build -d`)*

### 4. Acessar a Aplicação
- **Documentação Interativa (Swagger UI):** [http://localhost:8000/docs](http://localhost:8000/docs)
- **Verificação de Saúde (API + Banco):** [http://localhost:8000/saude](http://localhost:8000/saude)
- **Adminer (Interface do Banco de Dados):** [http://localhost:8080](http://localhost:8080)
  - **Sistema:** `MySQL`
  - **Servidor:** `db` *(atenção: usar `db`, e não `localhost`)*
  - **Usuário:** `appuser` (ou `root`)
  - **Senha:** `userpassword` (ou a senha definida no root)
  - **Base de dados:** `tarefas_db`

### 5. Parar a Aplicação
```powershell
docker compose down
```

---

## Guia de Solução de Problemas (Troubleshooting)

Aqui estão os erros mais comuns ao executar este projeto e o que fazer para resolver:

### 1. Erro: `Database is uninitialized and password option is not specified`
- **Sintoma:** O container `api_mysql-db-1` fecha logo no início com código de erro `Exited (1)`, e os containers da API e do Adminer ficam parados no status `Created`.
- **Causa:** O comando `docker compose up` foi executado sem declarar as variáveis de ambiente necessárias para o MySQL. A imagem oficial não permite iniciar sem uma senha de root definida.
- **Como resolver:**
  1. Remova os containers travados:
     ```powershell
     docker compose down
     ```
  2. Crie o arquivo `.env` dentro de `aula_19-10_docker/api_mysql/` (conforme o Passo 2) ou defina as variáveis no terminal PowerShell (`$env:MYSQL_ROOT_PASSWORD=...`).
  3. Execute novamente:
     ```powershell
     docker compose up --build
     ```

### 2. O script `init.sql` não roda ou precisa recriar o banco do zero
- **Sintoma:** O banco de dados inicia sem as tabelas criadas ou com dados antigos/corrompidos.
- **Causa:** O script `init.sql` só é executado pelo container do MySQL na **primeira vez** em que o volume de dados (`mysql_data`) é inicializado. Se o volume já existir (inclusive de uma tentativa com erro anterior), o MySQL pula a execução do `init.sql`.
- **Como resolver:**
  Derrube os containers apagando o volume antigo com o parâmetro `-v`:
  ```powershell
  docker compose down -v
  docker compose up --build
  ```

### 3. Erro de porta ocupada (`bind: address already in use`)
- **Sintoma:** O Docker avisa que a porta `8000` (API) ou `8080` (Adminer) já está alocada por outro processo.
- **Causa:** Outro aplicativo ou container antigo já está escutando na mesma porta no Windows.
- **Como resolver:**
  - Descubra se há outros containers ativos usando `docker ps` e encerre-os com `docker stop <id>`.
  - Ou altere a porta externa no [compose.yaml](file:///C:/Users/debor/OneDrive/Documentos/UP/ARQUITETURA%20DE%20SOFTWARE/pedidos/aula_19-10_docker/to-do-list-docker/aula_19-10_docker/api_mysql/compose.yaml) (por exemplo, alterar `"8000:8000"` para `"8001:8000"`).

### 4. Falha de conexão no Adminer (`Connection refused` ou erro de acesso)
- **Sintoma:** Não é possível conectar ao MySQL através do Adminer em `http://localhost:8080`.
- **Causa comum:** Digitar `localhost` ou `127.0.0.1` no campo **Servidor**.
- **Como resolver:**
  - Preencha o campo **Servidor** com o nome **`db`** (que é o nome do serviço na rede interna do Docker).
  - Use o usuário e senha definidos nas variáveis de ambiente (`appuser` / `userpassword` ou `root` / `rootpassword`).

### 5. Erro: Docker Desktop não está rodando (`cannot connect to docker daemon`)
- **Sintoma:** Mensagens como `open //./pipe/docker_engine: The system cannot find the file specified`.
- **Causa:** O aplicativo Docker Desktop está fechado ou ainda inicializando.
- **Como resolver:** Abra o **Docker Desktop** no Windows, aguarde o ícone da baleia ficar verde indicando *Engine running*, e tente novamente.

---

## Como Visualizar e Alterar Tarefas

### Pela Documentação Swagger UI (Navegador)
Acesse [http://localhost:8000/docs](http://localhost:8000/docs):
- **Ver tarefas:** Clique em `GET /tarefas` > **Try it out** > **Execute**.
- **Criar tarefa:** Clique em `POST /tarefas` > **Try it out** > informe o JSON `{"titulo": "Nova tarefa"}` > **Execute**.
- **Alterar tarefa:** Clique em `PATCH /tarefas/{tarefa_id}` > informe o `id` da tarefa e o JSON de alteração (ex: `{"concluida": true}` ou `{"titulo": "Novo texto"}`) > **Execute**.

### Pelo Postman
Importe a coleção e o ambiente disponíveis na pasta `aula_19-10_docker/postman`:
- `AS_aula-19-10_tarefas.postman_collection.json`
- `AS_aula-19-10_local.postman_environment.json`
