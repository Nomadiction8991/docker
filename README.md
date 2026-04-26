# 🐳 Docker + PHP 8.3 + Apache + Laravel

Ambiente completo em Docker para desenvolvimento PHP com Apache e MySQL. No primeiro boot, cria uma aplicação Laravel nova em `/${APP_NAME}` usando a versão estável mais recente.

## ⚡ Quick Start

```bash
# 1. Clonar e entrar no diretório
git clone <repo-url> && cd docker

# 2. Inicializar (cria .env, sobe containers e gera o Laravel em /${APP_NAME})
make up

# 3. Acessar
open http://localhost:<porta-mostrada-no-terminal>

# 4. Parar tudo
make down
```

## 📋 Requisitos

- Docker & Docker Compose (20.10+)
- ~500MB de espaço em disco
- Nenhuma outra aplicação usando as portas base definidas em `.env.example`

## 🏗️ Stack

| Componente  | Versão           |
| ----------- | ---------------- |
| PHP         | 8.3              |
| Apache      | 2.4              |
| MySQL       | 8.0+ / MariaDB   |
| Composer    | 2.x              |

## 🎯 Arquitetura

```plaintext
┌─────────────────────────────────────────┐
│      localhost:HOST_PORT (Host)         │
└──────────────┬──────────────────────────┘
               │
      ┌────────┴────────┐
      │   Docker        │
      │   Network       │
      │                 │
  ┌───▼─────┐    ┌──────▼──────┐
  │   web   │    │     db      │
  │ (PHP/   │───→│   (MySQL/   │
  │ Apache) │    │  MariaDB)   │
  └─────────┘    └─────────────┘
   :80→HOST_PORT :DB_PORT→DB_PORT
   /var/www/     /var/lib/
   html→.        mysql→db/
```

## 🚀 Comandos

### Com Makefile (recomendado)

```bash
make up       # Cria .env e inicia containers em background
make down     # Para, remove containers/volumes e limpa .env
make help     # Lista comandos disponíveis
```

### Docker Compose (direto)

```bash
# Iniciar
docker compose up --build              # foreground com rebuild
docker compose up -d --build            # background com rebuild

# Parar
docker compose stop                     # para containers
docker compose down                     # para e remove tudo

# Logs e shell
docker compose logs -f web              # logs em tempo real
docker compose exec web bash            # shell do container PHP
docker compose exec db mysql -u root -p # shell do MySQL
```

## 📁 Estrutura

```plaintext
docker/
├── Dockerfiles/
│   ├── Dockerfile-web          # Imagem PHP/Apache
│   ├── Dockerfile-db           # Imagem MySQL
│   ├── php.ini                 # Configurações PHP
│   ├── apache.conf             # VirtualHost Apache
│   ├── boot.sh                 # Script de inicialização
│   └── .dockerignore
├── ${APP_NAME}/                # Aplicação Laravel gerada no primeiro boot
├── docker-compose.yml          # Orquestração
├── composer.json               # Dependências PHP
├── .env.example                # Template de variáveis
├── Makefile                    # Atalhos
└── README.md
```

## ⚙️ Configuração

### Variáveis de Ambiente (.env)

O arquivo `.env` é criado automaticamente por `make up`. Edite para customizar:

```env
APP_NAME=docker              # Nome da pasta do Laravel e prefixo das imagens. Minusculo, sem espacos, use "-" como separador
LOCAL_UID=                  # Vazio = usa UID do host automaticamente
LOCAL_GID=                  # Vazio = usa GID do host automaticamente
HOST_PORT=8080              # Base HTTP; make up sobe para a primeira livre
DB_PORT=3306                # Base MySQL; make up sobe para a primeira livre
DB_DATABASE=docker          # Nome do banco
DB_ROOT_PASSWORD=root       # Senha root MySQL
```

### PHP Customizado

Edite `Dockerfiles/php.ini` e faça rebuild da imagem para aplicar mudanças.

### Apache/VirtualHost

Apache aponta para `/${APP_NAME}/public`. Rebuild necessário se mexer na imagem ou no boot.

## 💻 Desenvolvimento

### Adicionar Dependências

```bash
docker compose exec web composer require package/name
docker compose exec web composer install
```

### Acessar Banco de Dados

```bash
# Terminal MySQL
docker compose exec db mysql -uroot -p

# Queries diretas
docker compose exec db mysql -uroot -p -e "SELECT * FROM table;"
```

### Editar Código

- Arquivos em `/${APP_NAME}` são sincronizados em tempo real
- Não precisa restartear container para mudanças em `.php`
- Para mudanças em `Dockerfiles/`, faça rebuild: `docker compose up --build`

## 🔧 Troubleshooting

### Porta já em uso

```bash
# make up já pula para a primeira porta livre
```

### Permissões em volumes

```bash
# Se quiser forçar manualmente:
# preencha LOCAL_UID/LOCAL_GID em .env
```

### Erro de conexão MySQL

```bash
# Verifique credenciais em .env
docker compose exec db mysql -u root -p
# Ou reinicie: make down && make up
```

## 📚 Próximos Passos

- Explore `Dockerfiles/` para customizações
- Adicione migrations, modelos e controladores em `/${APP_NAME}`

## 📝 Licença

Este repositório é um **modelo pessoal de estudo** — use livremente como base para seus projetos.
