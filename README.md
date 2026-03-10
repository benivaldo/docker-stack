# Docker Stack - Ambiente de Desenvolvimento Multi-Aplicação

Stack Docker completo para desenvolvimento com PHP 5.6, PHP 8.4, Python, Redis, RabbitMQ, Elasticsearch e Kibana.

## 🚀 Quick Start

```bash
# Iniciar todos os serviços
docker-compose up -d

# Ver logs
docker-compose logs -f

# Parar serviços
docker-compose down
```

## 📦 Serviços Disponíveis

| Serviço | Porta | Descrição |
|---------|-------|-----------|
| Nginx (Monolítico) | 80 | PHP 5.6 Application |
| Nginx (Laravel) | 9000 | PHP 8.4 Laravel API |
| Nginx (Python) | 5000 | Python Application |
| Redis | 6379 | Cache & Data Store |
| RabbitMQ | 5672, 15672 | Message Broker |
| Elasticsearch | 9200 | Search Engine |
| Kibana | 5601 | Data Visualization |
| Nginx (Node) | 5000 | Node Application |

## 📚 Documentação

Para documentação completa de configuração, explicação de cada campo e uso detalhado, consulte:

**[📖 Documentação de Configuração](docs/CONFIGURACAO.md)**

## 🔧 Estrutura do Projeto

```
docker-stack/
├── docker-compose.yml          # Orquestração de containers
├── nginx/                      # Configurações Nginx
│   ├── default.conf           # PHP 5.6 (Monolítico)
│   ├── laravel.conf           # PHP 8.4 (Laravel)
│   └── python.conf            # Python (Proxy)
├── php/                       # Configurações PHP
│   ├── opcache.ini           # PHP 5.6 OPcache + Logs
│   └── php.ini               # PHP 8.4 Custom
├── logs/                      # Logs das aplicações
│   └── php56/                # Logs PHP 5.6 Monolítico
├── php56.Dockerfile          # Dockerfile PHP 5.6 customizado
└── docs/                     # Documentação
    └── CONFIGURACAO.md       # Documentação completa
```

## 🎯 Acesso Rápido

- **Monolítico PHP 5.6**: http://localhost:80
- **Laravel API**: http://localhost:9000
- **Python App**: http://localhost:5000
- **RabbitMQ Management**: http://localhost:15672 (admin/admin)
- **Kibana**: http://localhost:5601
- **Elasticsearch**: http://localhost:9200
- **Node API**: http://localhost:3000

## 📝 Comandos Úteis

```bash
# Composer (PHP 5.6)
docker-compose exec php56 composer install

# Artisan (Laravel)
docker-compose exec php84 php artisan migrate

# Ver logs do Monolítico
tail -f logs/php56/monolitico-error.log

# Acessar container
docker-compose exec [serviço] bash

# Reconstruir
docker-compose up -d --build
```

## ⚙️ Requisitos

- Docker Engine 19.03.0+
- Docker Compose 1.27.0+

## 📄 Licença

Este projeto é de uso interno para desenvolvimento.
