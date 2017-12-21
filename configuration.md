# Configuration

- [Introduction - Wprowadzenie](#introduction)
- [Environment Configuration - Konfiguracja środowiska](#environment-configuration)
    - [Retrieving Environment Configuration - Pobieranie konfiguracji środowiska](#retrieving-environment-configuration)
    - [Determining The Current Environment - Określanie obecnego środowiska](#determining-the-current-environment)
- [Accessing Configuration Values - Dostęp do wartości konfiguracyjnych](#accessing-configuration-values)
- [Configuration Caching - Buforowanie konfiguracji](#configuration-caching)
- [Maintenance Mode - Tryb konserwacji](#maintenance-mode)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Wszystkie pliki konfiguracyjne dla frameworka Laravel są przechowywane w katalogu `config`. Każda opcja jest udokumentowana, więc możesz przejrzeć pliki i zapoznać się z dostępnymi opcjami.

<a name="environment-configuration"></a>
## Environment Configuration - Konfiguracja środowiska

Często pomocne jest posiadanie różnych wartości konfiguracyjnych w zależności od środowiska, w którym działa aplikacja. Na przykład możesz chcieć używać innego sterownika cache lokalnie niż na twoim serwerze produkcyjnym.

Aby to zrobić, Laravel wykorzystuje bibliotekę PHP [DotEnv](https://github.com/vlucas/phpdotenv) autorstwa Vance Lucas. W świeżej instalacji Laravel, katalog główny twojej aplikacji będzie zawierał plik `.env.example`. Jeśli zainstalujesz Laravel przez Composer, ten plik zostanie automatycznie zmieniony na ".env". W przeciwnym razie należy ręcznie zmienić nazwę pliku.

Twój plik `.env` nie powinien być dołaczony do kontroli versji twojej aplikacji, ponieważ każdy programista / serwer korzystający z aplikacji może wymagać innej konfiguracji środowiska. Ponadto może to stanowić zagrożenie bezpieczeństwa w przypadku, gdy intruz uzyska dostęp do repozytorium kontroli versji, ponieważ wszelkie poufne dane uwierzytelniające zostaną ujawnione.

Jeśli pracujesz w zespole, możesz kontynuować dołączanie pliku `.env.example` do swojej aplikacji. Umieszczając wartości spersonalizowane w przykładowym pliku konfiguracyjnym, inni programiści w Twoim zespole mogą wyraźnie zobaczyć, które zmienne środowiskowe są potrzebne do uruchomienia aplikacji. Możesz także utworzyć plik `.env.testing`. Ten plik zastąpi plik `.env` podczas uruchamiania testów PHPUnit lub wykonywania poleceń Artisan za pomocą opcji `--env=testing`.

> {tip} Każda zmienna w pliku `.env` może zostać nadpisana przez zewnętrzne zmienne środowiskowe, takie jak zmienne środowiskowe na poziomie serwera lub systemu.

<a name="retrieving-environment-configuration"></a>
### Retrieving Environment Configuration - Pobieranie konfiguracji środowiska

Wszystkie zmienne wymienione w tym pliku zostaną załadowane do zmiennej super globalnej PHP `$ _ENV`, gdy aplikacja otrzyma żądanie. Możesz jednak użyć helpera `env` do pobrania wartości z tych zmiennych w swoich plikach konfiguracyjnych. W rzeczywistości, jeśli przejrzysz pliki konfiguracyjne Laravel, zauważysz kilka opcji, które już używają tego pomocnika:

    'debug' => env('APP_DEBUG', false),

Drugą wartością przekazaną do funkcji `env` jest" wartość domyślna ". Ta wartość zostanie użyta, jeśli dla danego klucza nie istnieje zmienna środowiskowa.

<a name="determining-the-current-environment"></a>
### Determining The Current Environment - Określanie obecnego środowiska 

Aktualne środowisko aplikacji jest określane za pomocą zmiennej `APP_ENV` z pliku` .env`. Możesz uzyskać dostęp do tej wartości za pomocą metody `environment` [facade (fasady)](/docs/{{version}}/facades) `App`:

    $environment = App::environment();

Możesz także przekazać argumenty do metody `environment`, aby sprawdzić, czy środowisko pasuje do danej wartości. Metoda zwróci wartość `true`, jeśli środowisko pasuje do którejkolwiek z podanych wartości:

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment(['local', 'staging'])) {
        // The environment is either local OR staging...
    }

> {tip} Obecne wykrywanie środowiska aplikacji może zostać przesłonięte przez zmienną środowiskową `APP_ENV` na poziomie serwera. Może to być przydatne, gdy trzeba udostępnić tę samą aplikację dla różnych konfiguracji środowiska, aby można było skonfigurować dany host tak, aby pasował do danego środowiska w konfiguracjach serwera.

<a name="accessing-configuration-values"></a>
## Accessing Configuration Values - Dostęp do wartości konfiguracyjnych

Możesz łatwo uzyskać dostęp do wartości konfiguracyjnych za pomocą globalnej funkcji `config` helper z dowolnego miejsca w aplikacji. Dostęp do wartości konfiguracyjnych można uzyskać za pomocą składni "kropka", która zawiera nazwę pliku i opcję, do której chcesz uzyskać dostęp. Wartość domyślna może być również określona i zostanie zwrócona, jeśli nie istnieje opcja konfiguracji:

    $value = config('app.timezone');

Aby ustawić wartości konfiguracyjne w środowisku wykonawczym, przekaż tablicę do helpera `config`:

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## Configuration Caching - Buforowanie konfiguracji

Aby przyspieszyć działanie aplikacji, wszystkie pliki konfiguracyjne należy przechowywać w pamięci podręcznej w jednym pliku przy użyciu polecenia `config: cache` Artisan. Spowoduje to połączenie wszystkich opcji konfiguracji aplikacji w jeden plik, który zostanie szybko załadowany przez framework-a.

Zwykle należy uruchomić polecenie `php artisan config: cache` jako część procedury wdrażania produkcyjnego. Polecenie nie powinno być uruchamiane podczas rozwoju lokalnego, ponieważ opcje konfiguracji często muszą być zmieniane w trakcie rozwoju aplikacji.

> {note} Jeśli wykonasz komendę `config: cache` podczas procesu wdrażania, powinieneś być pewien, że wywołujesz funkcję` env` z plików konfiguracyjnych.

<a name="maintenance-mode"></a>
## Maintenance Mode - Tryb konserwacji

Gdy aplikacja jest w trybie konserwacji, niestandardowy widok będzie wyświetlany dla wszystkich żądań do aplikacji. Ułatwia to "wyłączenie" aplikacji podczas jej aktualizacji lub podczas wykonywania konserwacji. Sprawdzanie trybu konserwacji znajduje się w domyślnym stosie oprogramowania pośredniego dla aplikacji. Jeśli aplikacja jest w trybie konserwacji, zostanie zgłoszony wyjątek `MaintenanceModeException` z kodem statusu 503.

Aby włączyć tryb konserwacji, po prostu wykonaj polecenie `down` Artisan:

    php artisan down

Możesz również podać opcje `message` i` retry` dla komendy `down`. Wartość `message` może być użyta do wyświetlenia lub zalogowania niestandardowej wiadomości, natomiast wartość` retry` zostanie ustawiona jako wartość nagłówka HTTP `Retry-After`:

    php artisan down --message="Upgrading Database" --retry=60

Aby wyłączyć tryb konserwacji, użyj polecenia `up`:

    php artisan up

> {tip} Możesz dostosować domyślny szablon trybu konserwacji, definiując własny szablon w `resources/views/errors/503.blade.php`.

#### Maintenance Mode & Queues - Tryb konserwacji i kolejki

Podczas gdy twoja aplikacja znajduje się w trybie konserwacji, żadne [queued jobs (zadania w kolejce)](/docs/{{version}}/queues) nie będą obsługiwane. Zadania będą nadal obsługiwane jak zwykle, gdy aplikacja znajdzie się poza trybem konserwacji.

#### Alternatives To Maintenance Mode - Alternatywy do trybu konserwacji

Ponieważ tryb konserwacji wymaga, aby aplikacja miała kilka sekund przestoju, rozważ rozwiązania alternatywne, takie jak [Envoyer](https://envoyer.io), aby wdrożyć zerową liczbę przestojów za pomocą Laravel.
