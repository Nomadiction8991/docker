# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão Geral

Template Docker para ambientes PHP 8.3 com Apache e MySQL 8.4. Projetado como ponto de partida para novos projetos PHP, com separação clara entre ambientes de desenvolvimento e produção.

## Comandos Principais

```bash
make up       # Cria .env a partir de .env.example (se não existir) e sobe os containers
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
| `web`   | `Dockerfiles/web/`       | 8080 → 80    |
| `db`    | `Dockerfiles/db/`        | 3306 → 3306  |

Ambos os serviços usam **builds multi-stage**:

- `base` → dependências comuns (PHP, Apache, Composer)
- `prod` → copia o código completo para a imagem
- `dev`  → adiciona Xdebug; não copia o código (usa bind-mount)

O estágio é controlado pela variável `HAMBIENTE` no `.env` (ex.: `prod` ou `dev`).

### Sequência de inicialização (web container)

`Dockerfiles/boot.sh` executa na inicialização:

1. `composer install` — se `vendor/` estiver ausente ou desatualizado
2. `php artisan migrate` — apenas se o arquivo `artisan` existir
3. Entrypoint padrão do PHP/Apache

### Roteamento

- Apache serve a partir de `/var/www/html/public`
- `public/.htaccess` redireciona todas as requisições para `index.php` (front controller)
- Namespace PSR-4: `App\` → `src/`

### Variáveis de Ambiente

Copiar `.env.example` para `.env` antes de iniciar. Variáveis importantes:

| Variável        | Descrição                                          |
|-----------------|----------------------------------------------------|
| `HAMBIENTE`     | Estágio Docker (`prod` ou `dev`)                   |
| `APP_NAME`      | Prefixo das imagens geradas                        |
| `HOST_PORT`     | Porta do host para o Apache                        |
| `DB_USERNAME`   | Usuário do banco (usar este, não DB_USER)          |
| `LOCAL_UID/GID` | UID/GID do host para evitar problemas de permissão |

## Convenções do Projeto

- Usar `DB_USERNAME` (e não `DB_USER`) nas configurações de banco.
- Validar portas publicadas com `docker compose ps` após alterações no compose.
- Expor portas de banco ao host apenas em desenvolvimento; em produção, manter `db` apenas na rede interna `app_network`.
- Documentação e mensagens em PT-BR; código em inglês.
