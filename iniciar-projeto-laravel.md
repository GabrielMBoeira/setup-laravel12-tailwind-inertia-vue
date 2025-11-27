? 1. Criar o projeto Laravel 12 já preparado para Sail
curl -s "https://laravel.build/meuapp?with=mysql" | bash


Depois:

cd meuapp
./vendor/bin/sail up -d

? 2. Instalar Inertia + Vue 3
Instalar o preset oficial:
./vendor/bin/sail composer require inertiajs/inertia-laravel
./vendor/bin/sail composer require laravel/breeze --dev

Gerar scaffolding Inertia + Vue:
./vendor/bin/sail php artisan breeze:install vue


?? IMPORTANTE: Breeze instala o plugin @vitejs/plugin-vue automaticamente, porém a versão padrão pode ser incompatível com Vite 7 (Laravel 12 usa Vite 7+).

Então, vamos remover e instalar a versão correta:

./vendor/bin/sail npm uninstall @vitejs/plugin-vue
./vendor/bin/sail npm install @vitejs/plugin-vue@latest --save-dev

? (Opcional) Confirme as versões no package.json

O plugin deve estar assim:

"@vitejs/plugin-vue": "^5.0.0"


(Varia conforme update, mas compatível com Vite 7.)

? 3. Instalar dependências JS e build inicial
./vendor/bin/sail npm install
./vendor/bin/sail npm run dev

? 4. Configurar o Tailwind CSS

Laravel Breeze já instala Tailwind automaticamente.
Mas, se quiser garantir a instalação manual:

./vendor/bin/sail npm install -D tailwindcss postcss autoprefixer
./vendor/bin/sail npx tailwindcss init -p


Depois, confirme que tailwind.config.js contém:

content: [
    './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
    './storage/framework/views/*.php',
    './resources/views/**/*.blade.php',
    './resources/js/**/*.vue',
],


E no resources/css/app.css:

@tailwind base;
@tailwind components;
@tailwind utilities;

? 5. Configurar MySQL no .env

Laravel Sail já cria o MySQL no docker.

Edite o .env:

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=meuapp
DB_USERNAME=sail
DB_PASSWORD=password



### Configurar compse.yaml
### Inicio
services:
    laravel.test:
        build:
            context: './vendor/laravel/sail/runtimes/8.4'
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: 'sail-8.4/app'
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
            IGNITION_LOCAL_SITES_PATH: '${PWD}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - mysql
    mysql:
        image: 'mysql/mysql-server:8.0'
        ports:
            - '${FORWARD_DB_PORT:-3306}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ALLOW_EMPTY_PASSWORD: 1
            MYSQL_EXTRA_OPTIONS: '${MYSQL_EXTRA_OPTIONS:-}'
        volumes:
            - 'sail-mysql:/var/lib/mysql'
        networks:
            - sail
        healthcheck:
            test:
                - CMD
                - mysqladmin
                - ping
                - '-p${DB_PASSWORD}'
            retries: 3
            timeout: 5s
networks:
    sail:
        driver: bridge
volumes:
    sail-mysql:
        driver: local
### fim



? 6. Levantar o ambiente
./vendor/bin/sail up -d


Testar conexão:

./vendor/bin/sail php artisan migrate

?? Pronto! Seu projeto agora tem:

? Laravel 12
? Sail gerenciando containers
? Docker com MySQL
? Vue 3 (Composition API)
? Inertia.js como ponte Laravel ? Vue
? Tailwind CSS
? Vite 7 funcionando sem conflitos

?? Comando ÚNICO ? Tudo de uma vez

Se quiser rodar tudo em sequência sem pensar:

curl -s "https://laravel.build/meuapp?with=mysql" | bash \
&& cd meuapp \
&& ./vendor/bin/sail up -d \
&& ./vendor/bin/sail composer require inertiajs/inertia-laravel \
&& ./vendor/bin/sail composer require laravel/breeze --dev \
&& ./vendor/bin/sail php artisan breeze:install vue \
&& ./vendor/bin/sail npm uninstall @vitejs/plugin-vue \
&& ./vendor/bin/sail npm install @vitejs/plugin-vue@latest --save-dev \
&& ./vendor/bin/sail npm install \
&& ./vendor/bin/sail npm run dev