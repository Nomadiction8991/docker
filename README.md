# 🐳 Docker + PHP 8.3 + Apache

Ambiente completo em Docker para desenvolvimento PHP com Apache e MySQL. Criado como **modelo reutilizável** para servir de base em novos projetos.

## ⚡ Quick Start

```bash
# 1. Clonar e entrar no diretório
git clone <repo-url> && cd docker

# 2. Inicializar (cria .env e sobe containers)
make up

# 3. Acessar
open http://localhost:8080

# 4. Parar tudo
make down
```

## 📋 Requisitos

- Docker & Docker Compose (20.10+)
- ~500MB de espaço em disco
- Nenhuma outra aplicação usando portas 8080 ou 3306

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
│          localhost:8080 (Host)          │
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
   :80→8080      :3306→3306
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
├── public/                     # Document Root (Apache)
│   └── index.php               # Ponto de entrada
├── src/                        # Código PHP (PSR-4)
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
APP_NAME=docker              # Nome da aplicação
LOCAL_UID=1000              # UID do usuário (corrija permissões em volumes)
LOCAL_GID=1000              # GID do usuário
HOST_PORT=8080              # Porta HTTP
DB_PORT=3306                # Porta MySQL
DB_DATABASE=docker          # Nome do banco
DB_USER=docker              # Usuário MySQL
DB_PASSWORD=app123          # Senha MySQL
DB_ROOT_PASSWORD=root       # Senha root MySQL
```

### PHP Customizado

Edite `Dockerfiles/php.ini` para mudar configurações PHP (ex: upload_max_filesize, memory_limit).

### Apache/VirtualHost

Configure routes em `Dockerfiles/apache.conf` (requer rebuild: `docker compose up --build`).

## 💻 Desenvolvimento

### Adicionar Dependências

```bash
docker compose exec web composer require package/name
docker compose exec web composer install
```

### Acessar Banco de Dados

```bash
# Terminal MySQL
docker compose exec db mysql -u docker -p

# Queries diretas
docker compose exec db mysql -u docker -p -e "SELECT * FROM table;"
```

### Editar Código

- Arquivos em `public/` e `src/` são sincronizados em tempo real
- Não precisa restartear container para mudanças em `.php`
- Para mudanças em `Dockerfiles/`, faça rebuild: `docker compose up --build`

## 🔧 Troubleshooting

### Porta já em uso

```bash
# Mude HOST_PORT em .env
# Ou mate o processo:
lsof -ti:8080 | xargs kill -9
```

### Permissões em volumes

```bash
# Rebuild com UID/GID corretos:
grep -E "^(UID|GID)" /etc/login.defs | awk '{print $NF}'
# Atualize LOCAL_UID/LOCAL_GID em .env
```

### Erro de conexão MySQL

```bash
# Verifique credenciais em .env
docker compose exec db mysql -u root -p
# Ou reinicie: make down && make up
```

## 📚 Próximos Passos

- Leia [INSTRUCTIONS.md](INSTRUCTIONS.md) para detalhes técnicos
- Explore `Dockerfiles/` para customizações
- Adicione migrations, modelos e controladores em `src/App/`

## 📝 Licença

Este repositório é um **modelo pessoal de estudo** — use livremente como base para seus projetos.
