# Documentação de Configuração - Docker Stack

## Índice
- [Visão Geral](#visão-geral)
- [Docker Compose](#docker-compose)
- [Nginx](#nginx)
- [PHP](#php)
- [Dockerfile PHP 5.6](#dockerfile-php-56)

---

## Visão Geral

Este stack Docker fornece um ambiente completo para desenvolvimento com múltiplas aplicações:
- **Monolítico PHP 5.6** (porta 80)
- **Laravel API PHP 8.4** (porta 9000)
- **Python** (porta 5000)
- **Redis** (porta 6379)
- **RabbitMQ** (portas 5672, 15672)
- **Elasticsearch** (porta 9200)
- **Kibana** (porta 5601)

---

## Docker Compose

### Estrutura do arquivo `docker-compose.yml`

#### **version: "3.9"**
Define a versão do Docker Compose. A versão 3.9 suporta recursos modernos e é compatível com Docker Engine 19.03.0+.

---

### Serviços

#### **nginx**
Servidor web para aplicação monolítica PHP 5.6.

```yaml
nginx:
  image: nginx:latest
  ports:
    - "80:80"
  volumes:
    - /home/Projetos/monolitico:/var/www/monolitico
    - /home/Projetos/monolitico/coverage:/var/www/monolitico/coverage
    - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
  networks:
    - app_network
```

**Campos:**
- `image`: Imagem Docker oficial do Nginx (última versão)
- `ports`: Mapeia porta 80 do host para porta 80 do container
- `volumes`:
  - Monta diretório do projeto monolítico
  - Monta diretório de coverage de testes
  - Monta arquivo de configuração customizado do Nginx
- `networks`: Conecta à rede compartilhada `app_network`

**Uso:** Acesse http://localhost:80

---

#### **nginx_laravel**
Servidor web para aplicação Laravel com PHP 8.4.

```yaml
nginx_laravel:
  image: nginx:latest
  ports:
    - "9000:80"
  volumes:
    - /home/Projetos/laravel/laravel-api:/var/www/laravel
    - ./nginx/laravel.conf:/etc/nginx/conf.d/default.conf
  networks:
    - app_network
```

**Campos:**
- `image`: Imagem Docker oficial do Nginx
- `ports`: Mapeia porta 9000 do host para porta 80 do container
- `volumes`:
  - Monta diretório do projeto Laravel
  - Monta configuração específica para Laravel
- `networks`: Conecta à rede compartilhada

**Uso:** Acesse http://localhost:9000

---

#### **nginx_python**
Servidor web como proxy reverso para aplicação Python.

```yaml
nginx_python:
  image: nginx:latest
  ports:
    - "5000:80"
  volumes:
    - ./nginx/python.conf:/etc/nginx/conf.d/default.conf
  networks:
    - app_network
```

**Campos:**
- `image`: Imagem Docker oficial do Nginx
- `ports`: Mapeia porta 5000 do host para porta 80 do container
- `volumes`: Monta configuração de proxy reverso
- `networks`: Conecta à rede compartilhada

**Uso:** Acesse http://localhost:5000

---

#### **php56**
Container Composer para gerenciar dependências PHP 5.6.

```yaml
php56:
  image: composer:1.10
  command: tail -f /dev/null
  volumes:
    - /home/Projetos/monolitico:/var/www/monolitico
  working_dir: /var/www/monolitico
  networks:
    - app_network
```

**Campos:**
- `image`: Composer versão 1.10 (compatível com PHP 5.6)
- `command`: Mantém container rodando indefinidamente
- `volumes`: Monta diretório do projeto
- `working_dir`: Define diretório de trabalho padrão
- `networks`: Conecta à rede compartilhada

**Uso:** 
```bash
docker-compose exec php56 composer install
docker-compose exec php56 composer update
```

---

#### **php56-fpm**
PHP-FPM 5.6 para processar requisições PHP do monolítico.

```yaml
php56-fpm:
  image: php:5.6-fpm
  volumes:
    - /home/Projetos/monolitico:/var/www/monolitico
    - ./php/opcache.ini:/usr/local/etc/php/conf.d/opcache.ini
    - ./logs/php56:/var/log/php
  networks:
    - app_network
```

**Campos:**
- `image`: PHP-FPM versão 5.6
- `volumes`:
  - Monta diretório do projeto
  - Monta configuração de OPcache e logs
  - Monta diretório de logs (monolitico-error.log)
- `networks`: Conecta à rede compartilhada

**Uso:** Processa arquivos PHP via FastCGI (porta 9000 interna)

**Logs:** Arquivo `logs/php56/monolitico-error.log` identifica erros do monolítico

---

#### **php84**
PHP-FPM 8.4 para processar requisições PHP do Laravel.

```yaml
php84:
  image: php:8.4-fpm
  volumes:
    - /home/Projetos/laravel/laravel-api:/var/www/laravel
    - ./php/php.ini:/usr/local/etc/php/conf.d/custom.ini
  networks:
    - app_network
```

**Campos:**
- `image`: PHP-FPM versão 8.4
- `volumes`:
  - Monta diretório do projeto Laravel
  - Monta configuração customizada do PHP
- `networks`: Conecta à rede compartilhada

**Uso:** Processa arquivos PHP via FastCGI (porta 9000 interna)

---

#### **python**
Container Python para executar aplicação Python.

```yaml
python:
  image: python:latest
  working_dir: /app
  volumes:
    - /home/Projetos/python:/app
  command: python app.py
  networks:
    - app_network
```

**Campos:**
- `image`: Python última versão
- `working_dir`: Diretório de trabalho dentro do container
- `volumes`: Monta diretório do projeto Python
- `command`: Comando para iniciar a aplicação
- `networks`: Conecta à rede compartilhada

**Uso:** Aplicação Python rodando na porta 5000 (interna)

---

#### **redis**
Cache e armazenamento de dados em memória.

```yaml
redis:
  image: redis:latest
  ports:
    - "6379:6379"
  networks:
    - app_network
```

**Campos:**
- `image`: Redis última versão
- `ports`: Mapeia porta padrão do Redis (6379)
- `networks`: Conecta à rede compartilhada

**Uso:** 
- Host: `redis` (dentro dos containers) ou `localhost` (do host)
- Porta: 6379

---

#### **rabbitmq**
Message broker para filas de mensagens.

```yaml
rabbitmq:
  image: rabbitmq:3-management
  ports:
    - "5672:5672"
    - "15672:15672"
  environment:
    - RABBITMQ_DEFAULT_USER=admin
    - RABBITMQ_DEFAULT_PASS=admin
  deploy:
    resources:
      limits:
        memory: 512M
      reservations:
        memory: 256M
  networks:
    - app_network
```

**Campos:**
- `image`: RabbitMQ versão 3 com interface de gerenciamento
- `ports`:
  - `5672`: Porta AMQP para conexões de aplicação
  - `15672`: Interface web de gerenciamento
- `environment`:
  - `RABBITMQ_DEFAULT_USER`: Usuário padrão (admin)
  - `RABBITMQ_DEFAULT_PASS`: Senha padrão (admin)
- `deploy.resources`:
  - `limits.memory`: Limite máximo de memória (512MB)
  - `reservations.memory`: Memória reservada (256MB)
- `networks`: Conecta à rede compartilhada

**Uso:**
- AMQP: `amqp://admin:admin@localhost:5672`
- Interface Web: http://localhost:15672 (admin/admin)

---

#### **elasticsearch**
Motor de busca e análise de dados.

```yaml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.0.0
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=false
  ports:
    - "9200:9200"
  networks:
    - app_network
```

**Campos:**
- `image`: Elasticsearch versão 8.0.0
- `environment`:
  - `discovery.type=single-node`: Modo single-node (desenvolvimento)
  - `xpack.security.enabled=false`: Desabilita autenticação (desenvolvimento)
- `ports`: Mapeia porta HTTP do Elasticsearch (9200)
- `networks`: Conecta à rede compartilhada

**Uso:** http://localhost:9200

---

#### **kibana**
Interface de visualização para Elasticsearch.

```yaml
kibana:
  image: docker.elastic.co/kibana/kibana:8.0.0
  ports:
    - "5601:5601"
  environment:
    - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
  networks:
    - app_network
```

**Campos:**
- `image`: Kibana versão 8.0.0
- `ports`: Mapeia porta da interface web (5601)
- `environment`:
  - `ELASTICSEARCH_HOSTS`: URL do Elasticsearch
- `networks`: Conecta à rede compartilhada

**Uso:** http://localhost:5601

---

### Networks

```yaml
networks:
  app_network:
```

**Campo:**
- `app_network`: Rede bridge customizada que permite comunicação entre todos os containers usando nomes de serviço como hostname.

**Uso:** Containers podem se comunicar usando nomes de serviço (ex: `http://redis:6379`, `http://elasticsearch:9200`)

---

## Nginx

### default.conf (Monolítico PHP 5.6)

```nginx
server {
    listen 80;
    server_name _;
    root /var/www/monolitico/public;
    index index.php;

    location ~ \.php$ {
        fastcgi_pass php56-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

**Diretivas:**
- `listen 80`: Escuta na porta 80
- `server_name _`: Aceita qualquer hostname
- `root`: Diretório raiz da aplicação
- `index`: Arquivo padrão (index.php)
- `location ~ \.php$`: Processa arquivos PHP
  - `fastcgi_pass`: Encaminha para PHP-FPM no container php56-fpm
  - `fastcgi_index`: Arquivo index padrão
  - `fastcgi_param SCRIPT_FILENAME`: Define caminho do script
  - `include fastcgi_params`: Inclui parâmetros FastCGI padrão
- `location /`: Roteamento para aplicações PHP
  - `try_files`: Tenta arquivo, diretório ou redireciona para index.php

---

### laravel.conf (Laravel PHP 8.4)

```nginx
server {
    listen 80;
    server_name _;
    root /var/www/laravel/public;
    index index.php;

    location ~ \.php$ {
        fastcgi_pass php84:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

**Diretivas:**
- Mesma estrutura do default.conf
- `fastcgi_pass php84:9000`: Encaminha para PHP-FPM 8.4
- `root /var/www/laravel/public`: Diretório público do Laravel

---

### python.conf (Proxy Reverso Python)

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://python:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Diretivas:**
- `proxy_pass`: Encaminha requisições para aplicação Python
- `proxy_set_header Host`: Preserva hostname original
- `proxy_set_header X-Real-IP`: IP real do cliente
- `proxy_set_header X-Forwarded-For`: Cadeia de proxies
- `proxy_set_header X-Forwarded-Proto`: Protocolo original (http/https)

---

## PHP

### php.ini (PHP 8.4 - Laravel)

```ini
upload_tmp_dir = /var/www/laravel/storage/temp
sys_temp_dir = /var/www/laravel/storage/temp
```

**Diretivas:**
- `upload_tmp_dir`: Diretório temporário para uploads
- `sys_temp_dir`: Diretório temporário do sistema

**Uso:** Garante que arquivos temporários sejam armazenados no storage do Laravel

---

### opcache.ini (PHP 5.6 - Monolítico)

```ini
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
memory_limit=256M
realpath_cache_size=4096K
realpath_cache_ttl=600

; Logs - Monolítico
error_log = /var/log/php/monolitico-error.log
log_errors = On
error_reporting = E_ALL
display_errors = Off
```

**Diretivas OPcache:**
- `opcache.enable=1`: Ativa OPcache
- `opcache.memory_consumption=256`: Memória para OPcache (256MB)
- `opcache.interned_strings_buffer=16`: Buffer para strings internadas (16MB)
- `opcache.max_accelerated_files=10000`: Máximo de arquivos em cache
- `opcache.revalidate_freq=2`: Frequência de revalidação (2 segundos)
- `opcache.fast_shutdown=1`: Shutdown rápido

**Diretivas PHP:**
- `memory_limit=256M`: Limite de memória por script
- `realpath_cache_size=4096K`: Cache de caminhos reais (4MB)
- `realpath_cache_ttl=600`: TTL do cache de caminhos (600 segundos)

**Diretivas de Log:**
- `error_log`: Caminho do arquivo de log (identifica como monolítico)
- `log_errors=On`: Ativa registro de erros
- `error_reporting=E_ALL`: Registra todos os tipos de erro
- `display_errors=Off`: Não exibe erros no navegador (segurança)

**Uso:** Otimiza performance do PHP 5.6 e registra erros em arquivo específico

---

## Dockerfile PHP 5.6

### php56.Dockerfile

```dockerfile
FROM php:5.6-fpm

# Instalar extensões
RUN apt-get update && apt-get install -y \
    libxml2-dev \
    && docker-php-ext-install opcache \
    && docker-php-ext-install mbstring \
    && rm -rf /var/lib/apt/lists/*

# Configurar OPcache
RUN echo "opcache.enable=1" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.memory_consumption=256" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.interned_strings_buffer=16" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.max_accelerated_files=10000" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.revalidate_freq=2" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.fast_shutdown=1" >> /usr/local/etc/php/conf.d/opcache.ini

# Configurar PHP
RUN echo "memory_limit=256M" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "realpath_cache_size=4096K" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "realpath_cache_ttl=600" >> /usr/local/etc/php/conf.d/custom.ini

WORKDIR /var/www/monolitico
```

**Instruções:**
- `FROM php:5.6-fpm`: Imagem base PHP 5.6 FPM
- `RUN apt-get update`: Atualiza repositórios
- `apt-get install -y libxml2-dev`: Instala dependências
- `docker-php-ext-install opcache`: Instala extensão OPcache
- `docker-php-ext-install mbstring`: Instala extensão mbstring
- `rm -rf /var/lib/apt/lists/*`: Remove cache do apt
- Configurações OPcache e PHP: Mesmas do opcache.ini
- `WORKDIR`: Define diretório de trabalho

**Uso:** 
```bash
docker build -f php56.Dockerfile -t custom-php56 .
```

---

## Comandos Úteis

### Iniciar stack
```bash
docker-compose up -d
```

### Parar stack
```bash
docker-compose down
```

### Ver logs
```bash
docker-compose logs -f [serviço]
```

### Executar comandos
```bash
# Composer (PHP 5.6)
docker-compose exec php56 composer install

# Artisan (Laravel)
docker-compose exec php84 php artisan migrate

# Python
docker-compose exec python python manage.py
```

### Ver logs
```bash
# Logs do container
docker-compose logs -f [serviço]

# Logs do Monolítico PHP 5.6
tail -f logs/php56/monolitico-error.log

# Logs em tempo real
watch -n 1 tail -20 logs/php56/monolitico-error.log
```

### Reconstruir containers
```bash
docker-compose up -d --build
```

---

## Personalização

### Alterar portas
Edite a seção `ports` no docker-compose.yml:
```yaml
ports:
  - "PORTA_HOST:PORTA_CONTAINER"
```

### Adicionar volumes
```yaml
volumes:
  - /caminho/host:/caminho/container
```

### Variáveis de ambiente
```yaml
environment:
  - VARIAVEL=valor
```

### Limites de recursos
```yaml
deploy:
  resources:
    limits:
      cpus: '0.5'
      memory: 512M
```

---

## Troubleshooting

### Container não inicia
```bash
docker-compose logs [serviço]
```

### Permissões de arquivo
```bash
sudo chown -R $USER:$USER /caminho/projeto
```

### Limpar volumes
```bash
docker-compose down -v
```

### Reconstruir imagens
```bash
docker-compose build --no-cache
```
