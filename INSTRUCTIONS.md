# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão Geral do Projeto

Este é um projeto Docker + PHP 8.3 + Apache criado como **modelo reutilizável** para aprendizagem e base em novos projetos PHP. É um ambiente de estudo pessoal, não um produto final.

**Stack**: Docker, PHP 8.3, Apache, MySQL/MariaDB, Composer.

## Arquitetura e Estrutura

```plaintext
docker/
├── Dockerfiles/          # Configurações de containerização
│   ├── Dockerfile-web    # Imagem PHP/Apache
│   ├── Dockerfile-db     # Imagem MySQL/MariaDB
│   ├── php.ini           # Configurações PHP para desenvolvimento
│   ├── apache.conf       # Configurações Apache/VirtualHost
│   ├── boot.sh           # Script de inicialização
│   └── .dockerignore
├── public/               # Raiz do servidor web (DocumentRoot)
│   └── index.php         # Ponto de entrada da aplicação
├── src/                  # Código PHP da aplicação (PSR-4 autoload)
├── docker-compose.yml    # Orquestração de containers (web + db)
├── composer.json         # Dependências PHP (PSR-4 autoload: App\\)
├── .env.example          # Variáveis de ambiente padrão
├── Makefile              # Atalhos para comandos comuns
└── .gitignore           # Arquivos ignorados pelo Git
```

## Configuração e Variáveis de Ambiente

O projeto usa um arquivo `.env` para configurar:

- `APP_NAME`: nome da aplicação (padrão: `docker`)
- `LOCAL_UID`, `LOCAL_GID`: UID/GID do usuário local (evita permissões incorretas em volumes)
- `HOST_PORT`: porta HTTP do host (padrão: `8080`)
- `DB_PORT`, `DB_DATABASE`, `DB_USER`, `DB_PASSWORD`, `DB_ROOT_PASSWORD`: configuração MySQL

**Gerar `.env`**: O arquivo é criado automaticamente pelo `make up` copiando `.env.example`. Não commitar `.env`.

## Comandos Principais

### Com Makefile (recomendado)

```bash
make up       # Cria .env (se não existir) e inicia containers em background
make down     # Para containers, remove imagens/volumes e limpa .env/vendor
make help     # Lista comandos disponíveis
```

### Com Docker Compose (direto)

```bash
docker compose up --build              # Rebuild e inicia em foreground
docker compose up -d --build            # Rebuild e inicia em background
docker compose down                     # Para e remove containers
docker compose logs -f [service]        # Mostra logs (web ou db)
docker compose exec web bash            # Acessa shell do container web
docker compose exec db mysql -u root -p # Acessa shell MySQL
```

## Fluxo de Desenvolvimento

1. **Inicializar**: `make up` — inicia web (PHP/Apache) e db (MySQL)
2. **Acessar**: <http://localhost:8080> (ou porta configurada em `.env`)
3. **Editar código**: Volumes montados em tempo real (hotreload)
4. **Dentro do container web**:
   - Composer: `docker compose exec web composer install`
   - PHP: `docker compose exec web php script.php`
5. **Parar**: `make down` — limpa containers, volumes e `.env`

## PSR-4 Autoload

Código PHP em `src/App/...` é carregado automaticamente via Composer (configurado em `composer.json`). Exemplo:

```php
// src/App/MyClass.php
namespace App;

class MyClass { ... }

// public/index.php
require __DIR__ . '/../vendor/autoload.php';
use App\MyClass;
```

## Container Web (PHP/Apache)

- **Imagem**: Dockerfile-web
- **PHP**: 8.3 (com extensões úteis)
- **Servidor**: Apache 2.4 com `mod_rewrite`
- **Volumes**: Raiz do projeto montada em `/var/www/html`
- **Arquivo de configuração**: `Dockerfiles/php.ini` (settings de desenvolvimento)
- **VirtualHost**: Configurado em `Dockerfiles/apache.conf`

## Container DB (MySQL)

- **Imagem**: Dockerfile-db (baseado em MySQL/MariaDB)
- **Credenciais**: Definidas em `.env` (padrão: user `docker`, password `app123`)
- **Volume persistente**: `db_volumes-db` — dados sobrevivem a `docker compose down`
- **Porta**: `DB_PORT` (padrão: 3306)

## Networking

- Rede bridge customizada: `${APP_NAME}-network`
- Containers podem se comunicar pelo hostname (ex: web acessa db via `db:3306`)
- Host pode acessar containers pelas portas mapeadas

## Recomendações para Contribuições

1. **Editar código PHP**: Coloque em `src/App/...` e importe via namespace
2. **Adicionar dependências**: Use `docker compose exec web composer require package`
3. **Modificar configurações PHP**: Edite `Dockerfiles/php.ini` e rebuild
4. **Modificar Apache**: Edite `Dockerfiles/apache.conf` e rebuild
5. **Limpar ambiente**: `make down` remove tudo; `make up` reconstrói from scratch
