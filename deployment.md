# Deployment

- [Introduction - Wprowadzenie](#introduction)
- [Server Configuration - Konfiguracja serwera](#server-configuration)
    - [Nginx](#nginx)
- [Optimization - Optymalizacja](#optimization)
    - [Autoloader Optimization - Optymalizacja autoloader](#autoloader-optimization)
    - [Optimizing Configuration Loading - Optymalizacja ładowania konfiguracji](#optimizing-configuration-loading)
    - [Optimizing Route Loading - Optymalizacja ładowania trasy](#optimizing-route-loading)
- [Deploying With Forge - Wdrażanie w Forge](#deploying-with-forge)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Kiedy jesteś gotowy do wdrożenia aplikacji Laravel do produkcji, możesz zrobić kilka ważnych rzeczy, aby upewnić się, że aplikacja działa tak wydajnie, jak to tylko możliwe. W tym dokumencie opiszemy kilka świetnych punktów początkowych, aby upewnić się, że Twoja aplikacja Laravel została poprawnie wdrożona.

<a name="server-configuration"></a>
## Server Configuration - Konfiguracja serwera

<a name="nginx"></a>
### Nginx

Jeśli wdrażasz aplikację na serwerze, na którym działa Nginx, możesz użyć poniższego pliku konfiguracyjnego jako punktu wyjścia do konfiguracji swojego serwera WWW. Najprawdopodobniej ten plik będzie musiał być dostosowany w zależności od konfiguracji twojego serwera. Jeśli potrzebujesz pomocy w zarządzaniu swoim serwerem, rozważ skorzystanie z usługi takiej jak [Laravel Forge](https://forge.laravel.com):

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## Optimization - Optymalizacja

<a name="autoloader-optimization"></a>
### Autoloader Optimization - Optymalizacja autoloader

Podczas wdrażania do produkcji upewnij się, że optymalizujesz mapę klas ładujących klasy Composer, aby Composer mógł szybko znaleźć właściwy plik do załadowania dla danej klasy:

    composer install --optimize-autoloader

> {tip} Oprócz optymalizacji autoloadera należy zawsze pamiętać o dołączeniu pliku `composer.lock` do repozytorium kontroli źródła projektu. Zależności twojego projektu można zainstalować znacznie szybciej, gdy obecny jest plik `composer.lock`.

<a name="optimizing-configuration-loading"></a>
### Optimizing Configuration Loading - Optymalizacja ładowania konfiguracji

Podczas wdrażania aplikacji do produkcji powinieneś upewnić się, że uruchomiłeś polecenie `config:cache` Artisan podczas procesu wdrażania:

    php artisan config:cache

To polecenie połączy wszystkie pliki konfiguracyjne Laravel w pojedynczy buforowany plik, co znacznie zmniejszy liczbę podróży, które framework musi wykonać w systemie plików podczas ładowania wartości konfiguracyjnych.


<a name="optimizing-route-loading"></a>
### Optimizing Route Loading - Optymalizacja ładowania trasy

Jeśli budujesz dużą aplikację z wieloma trasami, powinieneś upewnić się, że używasz polecenia `route:cache` Artisan podczas procesu wdrażania:

    php artisan route:cache

To polecenie redukuje wszystkie rejestracje trasy do pojedynczego wywołania metody w zbuforowanym pliku, poprawiając wydajność rejestracji trasy podczas rejestrowania setek tras.

> {note} Ponieważ ta funkcja korzysta z serializacji PHP, można buforować tylko trasy dla aplikacji używających wyłącznie tras opartych na sterowniku. PHP nie może serializować Closures

<a name="deploying-with-forge"></a>
## Deploying With Forge - Wdrażanie w Forge

Jeśli nie jesteś jeszcze gotowy do zarządzania własną konfiguracją serwera lub nie jesteś w stanie skonfigurować wszystkich różnych usług potrzebnych do uruchomienia solidnej aplikacji Laravel, [Laravel Forge](https://forge.laravel.com) jest cudowną alternatywą.


Laravel Forge może tworzyć serwery u różnych dostawców infrastruktury, takich jak DigitalOcean, Linode, AWS i innych. Ponadto Forge instaluje i zarządza wszystkimi narzędziami potrzebnymi do budowy solidnych aplikacji Laravel, takich jak Nginx, MySQL, Redis, Memcached, Beanstalk i innych.
