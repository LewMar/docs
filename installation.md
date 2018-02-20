# Installation

- [Installation - Instalacja](#installation)
    - [Server Requirements - Wymagania serwera](#server-requirements)
    - [Installing Laravel - Instalowanie Laravel-a](#installing-laravel)
    - [Configuration - Konfiguracja](#configuration)
- [Web Server Configuration - Konfiguracja serwera internetowego](#web-server-configuration)
    - [Pretty URLs - Ładne adresy URL](#pretty-urls)

<a name="installation"></a>
## Installation - Instalacja

> {video} Czy jesteś wzrokowcem? Laracast zapewnia [darmowy, dokładny wstęp do Laravel](http://laravelfromscratch.com) dla nowo przybyłych do frameworka. To świetne miejsce na rozpoczęcie podróży.

<a name="server-requirements"></a>
### Server Requirements - Wymagania serwera

Framework Laravel ma kilka wymagań systemowych. Oczywiście wszystkie te wymagania są spełnione przez maszynę wirtualną [Laravel Homestead](/docs/{{version}}/homestead), dlatego zdecydowanie zaleca się używanie Homestead jako lokalnego środowiska programistycznego Laravel.

Jeśli jednak nie korzystasz z Homestead, musisz upewnić się, że serwer spełnia następujące wymagania:

<div class="content-list" markdown="1">
- PHP >= 7.0.0
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
</div>

<a name="installing-laravel"></a>
### Installing Laravel - Instalowanie Laravel-a

Laravel wykorzystuje [Composer](https://getcomposer.org) do zarządzania swoimi zależnościami. Przed użyciem Laravel upewnij się, że masz zainstalowany Composer na twoim komputerze.

#### Via Laravel Installer - Przez instalację Laravel-a

Najpierw pobierz instalator Laravel za pomocą Composer:

    composer global require "laravel/installer"

Upewnij się, że umieszczasz ogólnoukładowy katalog bin dostawcy w katalogu `$PATH`, dzięki czemu plik wykonywalny laravel może znajdować się w twoim systemie. Ten katalog istnieje w różnych lokalizacjach w zależności od systemu operacyjnego; jednak niektóre typowe lokalizacje obejmują:

<div class="content-list" markdown="1">
- macOS: `$HOME/.composer/vendor/bin`
- GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin`
</div>

Po zainstalowaniu, komenda `laravel new` utworzy świeżą instalację Laravel w określonym katalogu. Na przykład `laravel new blog` utworzy katalog o nazwie `blog` zawierający świeżą instalację Laravel z zainstalowanymi już wszystkimi zależnościami Laravel:

    laravel new blog

#### Via Composer Create-Project - Przez stwórz-Projekt Composera

Możesz także zainstalować Laravel, wydając komendę `create-project` programu Composer w swoim terminalu:

    composer create-project --prefer-dist laravel/laravel blog "5.5.*"

#### Local Development Server - Lokalny serwer rozwoju

Jeśli masz PHP zainstalowany lokalnie i chciałbyś użyć wbudowanego serwera programowania PHP do obsługi aplikacji, możesz użyć polecenia `serve` Artisan. To polecenie uruchomi serwer programistyczny w `http://localhost:8000`:

    php artisan serve

Oczywiście bardziej zaawansowane opcje rozwoju lokalnego są dostępne przez [Homestead](/docs/{{version}}/homestead) i [Valet](/docs/{{version}}/valet)).

<a name="configuration"></a>
### Configuration - Konfiguracja

#### Public Directory - Katalog publiczny

Po zainstalowaniu Laravel, powinieneś skonfigurować katalog / serwer internetowy swojego serwera WWW jako katalog `public`. `Index.php` w tym katalogu służy jako front controller dla wszystkich żądań HTTP wprowadzanych do twojej aplikacji.

#### Configuration Files - Pliki konfiguracyjne

Wszystkie pliki konfiguracyjne dla frameworka Laravel są przechowywane w katalogu `config`. Każda opcja jest udokumentowana, więc możesz przejrzeć pliki i zapoznać się z dostępnymi opcjami.

#### Directory Permissions - Uprawnienia do katalogu

Po zainstalowaniu Laravel może być konieczne skonfigurowanie niektórych uprawnień. Katalogi w katalogach `storage` i `bootstrap/cache` powinny być zapisywane przez twój serwer sieciowy lub Laravel nie będzie działać. Jeśli korzystasz z maszyny wirtualnej [Homestead](/docs/{{version}}/homestead), uprawnienia te powinny już być ustawione.

#### Application Key - Klucz aplikacji

Następną rzeczą, którą powinieneś zrobić po zainstalowaniu Laravel, jest ustawienie klucza aplikacji na losowy ciąg znaków. Jeśli zainstalowałeś Laravel za pomocą programu Composer lub instalatora Laravel, klucz ten został już ustawiony dla ciebie za pomocą polecenia `php artisan key:generate`.

Zazwyczaj ten ciąg powinien mieć długość 32 znaków. Klucz można ustawić w pliku środowiska `.env`. Jeśli nie zmieniłeś nazwy pliku `.env.example` na `.env`, powinieneś to zrobić teraz. **Jeśli klucz aplikacji nie jest ustawiony, sesje użytkownika i inne zaszyfrowane dane nie będą bezpieczne!**

#### Additional Configuration - Dodatkowa konfiguracja

Laravel nie potrzebuje prawie żadnej innej konfiguracji po wyjęciu z pudełka. Możesz zacząć rozwijać! Możesz jednak przejrzeć plik `config/app.php` i jego dokumentację. Zawiera kilka opcji, takich jak `timezone` i `locale`, które możesz zmienić w zależności od aplikacji.

Możesz także skonfigurować kilka dodatkowych składników Laravel, takich jak:

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>
## Web Server Configuration - Konfiguracja serwera internetowego

<a name="pretty-urls"></a>
### Pretty URLs - Ładne adresy URL

#### Apache

Laravel zawiera plik `public/.htaccess`, który służy do podawania adresów URL bez kontrolera frontowego `index.php` w ścieżce. Przed serwerem Laravel z Apache, należy włączyć moduł `mod_rewrite`, aby plik `.htaccess` był honorowany przez serwer.

Jeśli plik `.htaccess` dostarczany z Laravel nie działa z instalacją Apache, wypróbuj tę opcję:

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

Jeśli korzystasz z Nginx, poniższa dyrektywa w konfiguracji witryny przekieruje wszystkie żądania do frontowego kontrolera `index.php`:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

Oczywiście przy korzystaniu z [Homestead](/docs/{{version}}/homestead) lub [Valet](/docs/{{version}}/valet) ładne adresy URL zostaną automatycznie skonfigurowane.
