# N8N - Orquestração de Workflows com Docker Compose

Este projeto utiliza Docker Compose para orquestrar uma stack completa do n8n, incluindo banco de dados PostgreSQL, Redis, workers, backup automatizado e interface de administração (pgAdmin).

## Sumário
- [Arquitetura dos Serviços](#arquitetura-dos-serviços)
- [Configuração de Variáveis de Ambiente](#configuração-de-variáveis-de-ambiente)
- [Processo de Inicialização do Banco](#processo-de-inicialização-do-banco)
- [Backup Automático](#backup-automático)
- [Volumes e Persistência](#volumes-e-persistência)
- [Como subir o ambiente](#como-subir-o-ambiente)
- [Acesso aos Serviços](#acesso-aos-serviços)

---

## Arquitetura dos Serviços

- **n8n**: Orquestrador principal de workflows.
- **n8n_worker_01/02/03**: Workers para execução paralela de fluxos.
- **postgres_n8n**: Banco de dados PostgreSQL para persistência dos dados do n8n.
- **postgres_backup**: Serviço que realiza backups automáticos do banco.
- **n8n_pgadmin**: Interface web para administração do PostgreSQL.
- **redis_n8n**: Fila de tarefas para execução distribuída.

## Configuração de Variáveis de Ambiente

As variáveis de ambiente estão centralizadas no arquivo `.env` (exemplo em `.env.exemple`). Principais variáveis:

- `DOMAIN_NAME`, `SUBDOMAIN`: Domínio e subdomínio de acesso ao n8n.
- `GENERIC_TIMEZONE`: Timezone do sistema.
- `POSTGRES_IMAGE`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`: Configurações do banco.
- `POSTGRES_NON_ROOT_USER`, `POSTGRES_NON_ROOT_PASSWORD`: Usuário não-root criado pelo script `init-data.sh`.
- `ENCRYPTION_KEY`: Chave de criptografia do n8n.
- `PGADMIN_DEFAULT_EMAIL`, `PGADMIN_DEFAULT_PASSWORD`: Acesso ao pgAdmin.

## Processo de Inicialização do Banco

O script `init-data.sh` é executado automaticamente ao subir o container do PostgreSQL. Ele:
- Cria um usuário não-root com permissões seguras.
- Garante privilégios para manipulação de tabelas, sequências e funções.
- Define privilégios padrão para novos objetos criados no banco.

## Backup Automático

O serviço `postgres_backup` executa periodicamente:
- Realiza backup do banco de dados usando `pg_dump` e salva compactado em `/backups`.
- Mantém apenas os 10 backups mais recentes, removendo os antigos automaticamente.
- Os arquivos de backup ficam disponíveis na pasta `backups/` do projeto.

## Volumes e Persistência

- `n8n_db_postgres`: Dados do PostgreSQL.
- `n8n_storage`: Dados persistentes do n8n.
- `n8n_redis_storage`: Dados do Redis.
- `n8n_pgadmin`: Dados do pgAdmin.
- `backups`: Backups do banco.
- `local-files/`: Armazenamento local e logs do n8n.

## Como subir o ambiente

1. Copie o arquivo `.env.exemple` para `.env` e ajuste as variáveis conforme necessário.
2. Execute:
   ```bash
   docker compose up -d
   ```
3. Os serviços serão inicializados automaticamente.

## Acesso aos Serviços

- **n8n**: https://[SUBDOMAIN].[DOMAIN_NAME]:5678
- **pgAdmin**: http://localhost:8081 (login conforme `.env`)
- **Backups**: Pasta `backups/` no projeto

---

### Observações
- O backup é feito a cada 24 horas (ajuste o tempo no `docker-compose.yml` conforme necessidade).
- O script `init-data.sh` só executa na primeira inicialização do banco.
- Para restaurar um backup, utilize o comando `pg_restore` ou `psql` manualmente.
