# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão Geral

Template Docker para ambientes PHP 8.3 com Apache e MySQL 8.4. Projetado como ponto de partida para novos projetos PHP, com separação clara entre ambientes de desenvolvimento e produção.

## Comandos Principais

```bash
make up       # Cria .env, acha as portas livres a partir dos valores em .env.example/.env e sobe os containers
make down     # Remove containers, imagens, volumes e apaga .env e vendor/
make help     # Lista os comandos disponíveis
```

Acessar o container web diretamente:

```bash
docker compose exec web bash
docker compose exec web php artisan <comando>   # se o projeto usar Laravel
```

Verificar status dos serviços e portas publicadas:

```bash
docker compose ps
```

## Arquitetura

### Serviços Docker

| Serviço | Dockerfile               | Porta padrão |
|---------|--------------------------|--------------|
| `web`   | `Dockerfiles/web/`       | HOST_PORT → 80 |
| `db`    | `Dockerfiles/db/`        | DB_PORT → DB_PORT |

Ambos os serviços usam **builds multi-stage**:

- `base` → dependências comuns (PHP, Apache, Composer)
- `production` → copia o código completo para a imagem
- `local` → adiciona Xdebug; não copia o código (usa bind-mount)

O estágio é controlado pela variável `APP_ENV` no `.env` (ex.: `production` ou `local`).

### Sequência de inicialização (web container)

`Dockerfiles/boot.sh` executa na inicialização:

1. `composer create-project laravel/laravel "$APP_NAME"` — se `/$APP_NAME/artisan` não existir
2. `composer install` — se `vendor/` estiver ausente ou desatualizado
3. `php artisan migrate` — apenas se o arquivo `artisan` existir
4. Entrypoint padrão do PHP/Apache

### Roteamento

- Apache serve a partir de `/$APP_NAME/public`
- `public/.htaccess` redireciona todas as requisições para `index.php` (front controller)
- Laravel fica em `/$APP_NAME`

### Variáveis de Ambiente

Copiar `.env.example` para `.env` antes de iniciar. Variáveis importantes:

| Variável        | Descrição                                          |
|-----------------|----------------------------------------------------|
| `APP_ENV`       | Estágio Docker (`production` ou `local`)           |
| `APP_NAME`      | Prefixo das imagens geradas                        |
| `HOST_PORT`     | Porta base do host para o Apache                   |
| `DB_PORT`       | Porta base do MySQL                                |
| `DB_ROOT_PASSWORD` | Senha root do MySQL                              |
| `LOCAL_UID/GID` | UID/GID do host; se vazio, usa o do host automático |

## Convenções do Projeto

- `make up` ajusta `HOST_PORT` e `DB_PORT` para a primeira porta livre.
- Validar portas publicadas com `docker compose ps` após alterações no compose.
- Documentação e mensagens em PT-BR; código em inglês.
