# Imagem base do Ubuntu 22.04
FROM ubuntu:22.04

# Maintainer do Docker
LABEL maintainer="seu-email@example.com"

# Atualizando o sistema e instalando as dependências básicas
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y \
    software-properties-common \
    curl \
    unzip \
    git \
    gnupg \
    build-essential && \
    apt-get clean

# Instalando o PHP 8.1.3 e extensões necessárias
RUN add-apt-repository ppa:ondrej/php -y && \
    apt-get update && \
    apt-get install -y \
    php8.1 \
    php8.1-cli \
    php8.1-mbstring \
    php8.1-xml \
    php8.1-curl \
    php8.1-zip \
    php8.1-bcmath \
    php8.1-tokenizer \
    php8.1-fpm && \
    apt-get clean

# Verificando a versão do PHP
RUN php -v

# Instalando o Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Instalando o Node.js na versão 18.16.0
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g n && \
    n 18.16.0 && \
    apt-get remove -y nodejs && \
    apt-get autoremove -y

# Verificando a versão do Node.js
RUN node -v && npm -v

# Configurando o diretório de trabalho
WORKDIR /var/www/laravel

# Copiando arquivos do projeto para o container
COPY . .

# Instalando dependências do Composer
RUN composer install --no-dev --optimize-autoloader

# Gerando a chave do Laravel
RUN php artisan key:generate

# Configurando permissões
RUN chmod -R 775 storage bootstrap/cache && \
    chown -R www-data:www-data /var/www/laravel

# Configurando o Laravel para usar a porta 5000
ENV APP_PORT=5000

# Expondo a porta 5000
EXPOSE 5000

# Instalando as dependências de frontend do Laravel (opcional, para Laravel Mix ou Vite)
RUN npm install && npm run build

# Configurando o servidor web (Nginx)
RUN apt-get install -y nginx && \
    echo "server { \
    listen 5000; \
    root /var/www/laravel/public; \
    index index.php index.html; \
    server_name localhost; \
    location / { \
        try_files \$uri \$uri/ /index.php?\$query_string; \
    } \
    location ~ \.php$ { \
        include snippets/fastcgi-php.conf; \
        fastcgi_pass 127.0.0.1:9000; \
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name; \
        include fastcgi_params; \
    } \
}" > /etc/nginx/sites-available/laravel

RUN ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
RUN rm /etc/nginx/sites-enabled/default

# Comando para iniciar o servidor
CMD service php8.1-fpm start && nginx -g 'daemon off;'
