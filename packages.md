# Package Development

- [Introduction - Wprowadzenie](#introduction)
    - [A Note On Facades - Uwaga na fasady](#a-note-on-facades)
- [Package Discovery - Wykrywanie pakietów](#package-discovery)
- [Service Providers - Dostawcy usług](#service-providers)
- [Resources - Zasoby](#resources)
    - [Configuration - Konfiguracja](#configuration)
    - [Migrations - Migracje](#migrations)
    - [Routes - Trasy](#routes)
    - [Translations - Tłumaczenia](#translations)
    - [Views - Widoki](#views)
- [Commands - Polecenia](#commands)
- [Public Assets - Zasoby publiczne](#public-assets)
- [Publishing File Groups - Publikowanie grup plików](#publishing-file-groups)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Pakiety są podstawowym sposobem dodawania funkcjonalności do Laravel. Pakiety mogą być wszystkim, od świetnego sposobu pracy z datami takimi jak [Carbon](https://github.com/briannesbitt/Carbon) lub całą strukturą testowania BDD, np. [Behat](https://github.com/Behat/Behat).

Oczywiście istnieją różne rodzaje pakietów. Niektóre pakiety są niezależne, co oznacza, że działają z dowolnym środowiskiem PHP. Carbon i Behat są przykładami autonomicznych pakietów. Dowolny z tych pakietów może być używany z Laravel, po prostu żądając ich w pliku `composer.json`.

Z drugiej strony, inne pakiety są specjalnie przeznaczone do użytku z Laravel. Pakiety te mogą zawierać trasy, kontrolery, widoki i konfigurację specjalnie zaprojektowaną w celu ulepszenia aplikacji Laravel. Ten przewodnik opisuje przede wszystkim rozwój tych pakietów, które są specyficzne dla Laravel.

<a name="a-note-on-facades"></a>
### A Note On Facades - Uwaga na fasady

Podczas pisania aplikacji Laravel zwykle nie ma znaczenia, czy używasz kontraktów czy fasad, ponieważ oba zapewniają w zasadzie równy poziom testowalności. Jednak podczas pisania paczek, twój pakiet nie będzie zazwyczaj miał dostępu do wszystkich pomocników testujących Laravel. Jeśli chciałbyś móc pisać testy pakietów tak, jakby istniały w typowej aplikacji Laravel, możesz skorzystać z pakietu [Orchestral Testbench](https://github.com/orchestral/testbench).

<a name="package-discovery"></a>
## Package Discovery - Wykrywanie pakietów

W pliku konfiguracyjnym `config/app.php` aplikacji Laravel opcja `providers` definiuje listę dostawców usług, które powinny być ładowane przez Laravel. Kiedy ktoś instaluje twój pakiet, zazwyczaj chcesz, aby Twój usługodawca znalazł się na tej liście. Zamiast wymagać od użytkowników ręcznego dodawania usługodawcy do listy, możesz zdefiniować dostawcę w sekcji `extra` pliku `composer.json` pakietu. Oprócz usługodawców możesz również wymienić dowolne [fasady](/docs/{{version}}/facades), które chciałbyś zarejestrować:

    "extra": {
        "laravel": {
            "providers": [
                "Barryvdh\\Debugbar\\ServiceProvider"
            ],
            "aliases": {
                "Debugbar": "Barryvdh\\Debugbar\\Facade"
            }
        }
    },

Po skonfigurowaniu pakietu do wykrywania, Laravel automatycznie zarejestruje dostawców usług i fasady po zainstalowaniu, tworząc wygodną instalację dla użytkowników pakietu.

### Opting Out Of Package Discovery - Rezygnacja z odkrywania pakietów

Jeśli jesteś konsumentem pakietu i chciałbyś wyłączyć wykrywanie pakietów dla pakietu, możesz podać nazwę pakietu w sekcji `extra` pliku `composer.json` aplikacji:

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },

Możesz wyłączyć odnajdywanie pakietów dla wszystkich pakietów, używając znaku `*` wewnątrz dyrektywy `dont-discover` aplikacji:

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },

<a name="service-providers"></a>
## Service Providers - Dostawcy usług

[Dostawcy usług](/docs/{{version}}/providers) to punkty połączenia między pakietem a Laravel. Dostawca usług jest odpowiedzialny za wiązanie rzeczy w [kontenerze usługi Laravel](/docs/{{version}}/container) i informowanie Laravel, gdzie należy załadować zasoby pakietu, takie jak widoki, pliki konfiguracyjne i lokalizacyjne.

Dostawca usług rozszerza klasę `Illuminate\Support\ServiceProvider` i zawiera dwie metody: `register` i `boot`. Podstawowa klasa `ServiceProvider` znajduje się w pakiecie `illuminate/support` Composer, który należy dodać do zależności twojego pakietu. Aby dowiedzieć się więcej o strukturze i celu usługodawców, sprawdź [ich dokumentację](/docs/{{version}}/providers).

<a name="resources"></a>
## Resources - Zasoby

<a name="configuration"></a>
### Configuration - Konfiguracja

Zazwyczaj będziesz musiał opublikować plik konfiguracyjny pakietu w katalogu `config` aplikacji. Pozwoli to użytkownikom pakietu łatwo zastąpić domyślne opcje konfiguracji. Aby zezwolić na publikowanie plików konfiguracyjnych, wywołaj metodę `publishes` z metody` boot` twojego usługodawcy:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

Teraz, gdy użytkownicy twojego pakietu wykonają polecenie Laravel `vendor:publish`, twój plik zostanie skopiowany do określonej lokalizacji publikacji. Oczywiście, kiedy twoja konfiguracja zostanie opublikowana, jej wartości mogą być dostępne jak każdy inny plik konfiguracyjny:

    $value = config('courier.option');

> {note} Nie powinieneś definiować Closures w swoich plikach konfiguracyjnych. Nie można ich poprawnie serializować, gdy użytkownicy wykonują komendę `config:cache` Artisan.

#### Default Package Configuration - Domyślna konfiguracja pakietu

Możesz także scalić swój plik konfiguracyjny pakietu z opublikowaną kopią aplikacji. Umożliwi to użytkownikom zdefiniowanie tylko tych opcji, które faktycznie mają zastąpić w opublikowanej kopii konfiguracji. Aby scalić konfiguracje, użyj metody `mergeConfigFrom` w ramach metody` register` dostawcy usług:

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

> {note} Ta metoda łączy tylko pierwszy poziom tablicy konfiguracji. Jeśli Twoi użytkownicy częściowo zdefiniują wielowymiarową tablicę konfiguracji, brakujące opcje nie zostaną scalone.

<a name="routes"></a>
### Routes - Trasy

Jeśli twój pakiet zawiera trasy, możesz je załadować za pomocą metody `loadRoutesFrom`. Ta metoda automatycznie określi, czy trasy aplikacji są buforowane i nie załaduje pliku tras, jeśli trasy zostały już zapisane w pamięci podręcznej:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }

<a name="migrations"></a>
### Migrations - Migracje

Jeśli twój pakiet zawiera [migracje baz danych](/docs/{{version}}/migrations), możesz użyć metody `loadMigrationsFrom` do poinformowania Laravel, jak je załadować. Metoda `loadMigrationsFrom` akceptuje ścieżkę do migracji pakietu jako jedyny argument:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }

Po zarejestrowaniu migracji pakietu, zostaną one automatycznie uruchomione po wykonaniu polecenia `php artisan migrate`. Nie musisz eksportować ich do głównego katalogu `database/migrations` aplikacji.

<a name="translations"></a>
### Translations - Tłumaczenia

Jeśli twój pakiet zawiera [pliki tłumaczeń](/docs/{{version}}/localization), możesz użyć metody `loadTranslationsFrom`, aby poinformować Laravel, jak je załadować. Na przykład, jeśli Twój pakiet nazywa się `courier`, powinieneś dodać następujące polecenie do metody `boot` usługodawcy:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

Do tłumaczeń pakietów odwołujemy się za pomocą konwencji składniowej `package::file.line`. Tak więc możesz załadować `welcome` paczki `courier` z pliku `messages` tak jak poniżej:

    echo trans('courier::messages.welcome');

#### Publishing Translations - Publikowanie tłumaczeń

Jeśli chcesz opublikować tłumaczenia pakietu w katalogu `resources/lang/vendor` aplikacji, możesz użyć metody `publishes` dostawcy usług. Metoda `publishes` akceptuje tablice ścieżek pakietów i ich pożądane lokalizacje publikowania. Na przykład, aby opublikować pliki tłumaczeń dla pakietu `courier`, możesz wykonać następujące czynności:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }

Teraz, gdy użytkownicy twojego pakietu wykonają polecenie Laravel `vendor:publish` Artisan, tłumaczenia Twojego pakietu zostaną opublikowane w określonej lokalizacji publikacji.

<a name="views"></a>
### Views - Widoki

Aby zarejestrować pakiet [widoki](/docs/{{version}}/views) za pomocą Laravel, musisz powiedzieć Laravel-owi, gdzie znajdują się widoki. Możesz to zrobić za pomocą metody `loadViewsFrom` dostawcy usług. Metoda `loadViewsFrom` przyjmuje dwa argumenty: ścieżkę do szablonów widoku i nazwę pakietu. Na przykład, jeśli nazwa twojego pakietu to `courier`, dodaj to do metody `boot` usługodawcy:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

Widoki pakietów są przywoływane przy użyciu konwencji składni `package::view`. Tak więc, po zarejestrowaniu ścieżki widoku w usługodawcy, możesz wczytać widok `admin` z pakietu `courier`:

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### Overriding Package Views - Przesłanianie widoków pakietów

Kiedy używasz metody `loadViewsFrom`, Laravel faktycznie rejestruje dwie lokalizacje dla twoich widoków: katalog `resources/views/vendor` aplikacji oraz katalog, który określasz. Tak więc, używając przykładu `courier`, Laravel najpierw sprawdzi, czy niestandardowa wersja widoku została dostarczona przez programistę w `resources/views/vendor/courier`. Następnie, jeśli widok nie został spersonalizowany, Laravel przeszuka katalog widoku pakietów określony w twoim wywołaniu do `loadViewsFrom`. Ułatwia to użytkownikom pakietów dostosowywanie / zastępowanie widoków pakietu.

#### Publishing Views - Publikowanie widoków

Jeśli chcesz udostępnić swoje widoki do publikacji w katalogu `resources/views/vendor` aplikacji, możesz użyć metody `publishes` dostawcy usług. Metoda `publishes` akceptuje tablicę ścieżek widoków pakietów i ich pożądanych lokalizacji publikowania:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

Teraz, gdy użytkownicy twojego pakietu wykonają polecenie Laravel `vendor:publish` Artisan, widok twojego pakietu zostanie skopiowany do określonej lokalizacji publikacji.

<a name="commands"></a>
## Commands - Polecenia

Aby zarejestrować polecenia Artisan w pakiecie za pomocą Laravel, możesz użyć metody `commands`. Ta metoda oczekuje na tablicę nazw klas poleceń. Po zarejestrowaniu poleceń możesz je wykonać za pomocą [Artisan CLI](/docs/{{version}}/artisan):

    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>
## Public Assets - Zasoby publiczne

Twój pakiet może zawierać zasoby takie jak JavaScript, CSS i obrazy. Aby opublikować te zasoby w katalogu `public` aplikacji, użyj metody `publishes` aplikacji dostawcy. W tym przykładzie dodamy również `public` tag grupy zasobów, który może być używany do publikowania grup powiązanych zasobów:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

Teraz, gdy użytkownicy pakietu wykonają komendę `vendor:publish`, zasoby zostaną skopiowane do określonej lokalizacji publikacji. Ponieważ zazwyczaj będziesz musiał nadpisywać zasoby za każdym razem, gdy pakiet jest aktualizowany, możesz użyć flagi `--force`:

    php artisan vendor:publish --tag=public --force

<a name="publishing-file-groups"></a>
## Publishing File Groups - Publikowanie grup plików

Możesz opublikować grupy zasobów pakietu i zasobów oddzielnie. Na przykład możesz pozwolić użytkownikom na publikowanie plików konfiguracyjnych pakietu bez konieczności publikowania zasobów pakietu. Możesz to zrobić przez "oznaczenie" ich podczas wywoływania metody `publishing` od dostawcy usług pakietu. Na przykład, użyjmy znaczników, aby zdefiniować dwie grupy publikowania w metodzie `boot` dostawcy usług pakietowych:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }

Teraz Twoi użytkownicy mogą publikować te grupy osobno, odwołując się do ich tagów podczas wykonywania polecenia `vendor:publish`:

    php artisan vendor:publish --tag=config
