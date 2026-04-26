# Docker + PHP 8.3 + Apache + Laravel

Template Docker para desenvolvimento e deploy de projetos PHP/Laravel. Inclui ambiente local com Xdebug e pipeline de deploy automatizado via GitHub Actions (FTP + SSH).

## Quick Start

```bash
git clone <repo-url> meu-projeto && cd meu-projeto
make up
```

Acesse em `http://localhost:<porta-exibida-no-terminal>`.

## Comandos

```bash
make up       # Cria .env, encontra portas livres e sobe os containers
make down     # Para containers, remove imagens/volumes e apaga .env
make help     # Lista todos os comandos disponíveis
```

Acesso direto ao container:

```bash
docker compose exec web bash
docker compose exec web php artisan <comando>
docker compose ps   # status e portas publicadas
```

## Estrutura

```text
.
├── Dockerfiles/
│   ├── web/Dockerfile        # PHP 8.3 + Apache (multi-stage)
│   ├── apache.conf           # VirtualHost — serve APP_NAME/public
│   └── boot.sh               # Inicialização: composer install + migrate
├── start-project/            # Projeto Laravel base (ponto de partida)
├── scripts/
│   └── update_env_value.php  # Edita variáveis no .env sem quebrar comentários
├── .github/workflows/
│   ├── build-image.yml       # Monta pacote Laravel para deploy
│   ├── ftp-deploy.yml        # Envia arquivos via FTP (delta)
│   ├── ssh-finalize.yml      # composer install + key:generate + migrate via SSH
│   └── secrets-reference.md  # Guia de configuração dos secrets
├── docker-compose.yml
├── .env.example
└── Makefile
```

## Configuração (.env)

O `.env` é criado automaticamente pelo `make up` a partir do `.env.example`.

| Variável | Descrição |
| --- | --- |
| `APP_NAME` | Nome do app. Minúsculo, sem espaços, use `-` como separador |
| `APP_ENV` | Estágio Docker: `local` (Xdebug, bind-mount) ou `production` |
| `HOST_PORT` | Porta base HTTP — `make up` sobe para a primeira livre |
| `DB_HOST` | Host do banco (padrão: `db`) |
| `DB_PORT` | Porta base MySQL — `make up` sobe para a primeira livre |
| `DB_DATABASE` | Nome do banco |
| `DB_ROOT_PASSWORD` | Senha root MySQL |
| `LOCAL_UID/GID` | UID/GID do host; vazio = detectado automaticamente |

## Arquitetura Docker

Dois serviços, ambos com **build multi-stage**:

| Serviço | Imagem base        | Porta              |
| ------- | ------------------ | ------------------ |
| `web`   | PHP 8.3 + Apache   | `HOST_PORT` → 80   |
| `db`    | MySQL 8.4          | `DB_PORT` → 3306   |

Estágios da imagem `web`:

- `base` — PHP, Apache, Composer
- `production` — copia o código; sem Xdebug
- `local` — adiciona Xdebug; usa bind-mount (sem cópia)

O estágio ativo é controlado por `APP_ENV` no `.env`.

### Boot do container web

`Dockerfiles/boot.sh` executa na inicialização:

1. `composer install` — se `vendor/` ausente ou desatualizado
2. `php artisan migrate` — se `artisan` existir
3. Entrypoint padrão PHP/Apache

Apache serve a partir de `APP_NAME/public` com front controller via `.htaccess`.

## Pipeline de Deploy (GitHub Actions)

Ativado por push de tag no formato `AA.MM.DD` (ex: `26.04.25`) apontando para um commit em `main`.

```text
push de tag
    └─► build-image       → monta pacote Laravel, publica artefato
            └─► ftp-deploy    → envia arquivos via FTP (só o delta)
                    └─► ssh-finalize → composer install + key:generate + migrate
```

### Configurar deploy

Cadastre os secrets em `Settings > Secrets and variables > Actions`:

| Secret                                                              | Descrição                                  |
| ------------------------------------------------------------------- | ------------------------------------------ |
| `DOMINIO`                                                           | Domínio base do servidor (FTP, SSH, URL)   |
| `FTP_USERNAME` / `FTP_PASSWORD`                                     | Credenciais FTP                            |
| `DB_HOST` / `DB_PORT` / `DB_DATABASE` / `DB_USERNAME` / `DB_PASSWORD` | Banco de dados                          |
| `SSH_USERNAME` / `SSH_PORT` / `SSH_KEY`                             | Acesso SSH                                 |
| `SSH_PATH`                                                          | Pasta raiz no servidor (ex: `/domains`)    |

Veja `.github/workflows/secrets-reference.md` para detalhes completos.

### Como o deploy funciona

- `APP_NAME=start-project` → pasta remota: `startproject.dominio.com`
- `APP_NAME` no `.env` de produção vira título: `"Start Project"`
- FTP sincroniza apenas arquivos novos ou alterados (state persistido em cache)
- SSH finaliza: `composer install --no-scripts` → `package:discover` → `key:generate` → `migrate`

### Publicar uma versão

```bash
git tag 26.04.25
git push origin main 26.04.25
```

## Usando como base para novos projetos

1. Renomeie a pasta `start-project/` para o nome do seu projeto
2. Atualize `APP_NAME` no `.env.example`
3. Configure os secrets no GitHub
4. Faça push e crie uma tag para disparar o deploy
