# Service Providers

- [Introduction - Wprowadzenie](#introduction)
- [Writing Service Providers - Pisanie dostawców usług](#writing-service-providers)
    - [The Register Method - Metoda rejestracji](#the-register-method)
    - [The Boot Method - Metoda rozruchu](#the-boot-method)
- [Registering Providers - Rejestrowanie dostawców](#registering-providers)
- [Deferred Providers - Rejestrowanie dostawców](#deferred-providers)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Dostawcy usług są centralnym miejscem wszystkich uruchomień aplikacji Laravel. Twoja własna aplikacja, a także wszystkie podstawowe usługi Laravel są "bootstrapped" (samo załadowywane) za pośrednictwem dostawców usług.

Ale co mamy na myśli przez "bootstrapped"? Ogólnie rzecz biorąc, mamy na myśli **registering (rejestrowanie)** rzeczy, w tym rejestrowanie powiązań service container bindings (kontenerów usług), event listeners (detektorów zdarzeń), middleware (oprogramowania pośredniego), a nawet routes (tras). Service providers (Dostawcy usług) są głównym miejscem do konfiguracji aplikacji.

Jeśli otworzysz plik `config/app.php` dołączony do Laravel, zobaczysz tablicę `providers`. Są to wszystkie klasy service provider (dostawców usług), które zostaną załadowane do Twojej aplikacji. Oczywiście, wielu z nich jest "deferred ("odroczonych) providers (dostawców), co oznacza, że nie zostaną załadowane na każde żądanie, ale tylko wtedy, gdy usługi, które świadczą, są rzeczywiście potrzebne.

W tym przeglądzie nauczysz się, jak pisać własnych dostawców usług i rejestrować je za pomocą aplikacji Laravel.

<a name="writing-service-providers"></a>
## Writing Service Providers - Pisanie dostawców usług

Wszyscy service providers (dostawcy usług) rozszerzają klasę `Illuminate\Support\ServiceProvider`. Większość service providers (dostawcy usług) zawiera metodę `register` i `boot`. W ramach metody `register`, powinieneś **tylko powiązać rzeczy w [service container (kontener usługi)](/docs/{{version}}/container)**. Nigdy nie należy próbować rejestrować żadnych event listeners (etektorów zdarzeń), routes (tras), ani żadnej innej funkcjonalności w ramach metody `register`.

Artisan CLI może wygenerować nowego dostawcę za pomocą komendy `make:provider`:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### The Register Method - Metoda rejestracji

Jak wspomniano wcześniej, w ramach metody `register` powinieneś tylko powiązać rzeczy w [service container (kontener usługi)](/docs/{{version}}/container). Nigdy nie należy próbować rejestrować żadnych event listeners (etektorów zdarzeń), routes (tras), ani żadnej innej funkcjonalności w ramach metody `register`. W przeciwnym razie możesz przypadkowo skorzystać z usługi świadczonej przez usługodawcę, który jeszcze nie został załadowany.

Rzućmy okiem na podstawowego dostawcę usług. W ramach każdej z metod dostawcy usług zawsze masz dostęp do właściwości `$app`, która zapewnia dostęp do kontenera usług:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

Ten dostawca usług definiuje tylko metodę `register`,  i używa tej metody do zdefiniowania implementacji `Riak\Connection` w kontenerze usług. Jeśli nie wiesz, jak działa kontener usługi, sprawdź [jego dokumentację](/docs/{{version}}/container).

#### The `bindings` And `singletons` Properties - Właściwości `bindings` i `singletons`

Jeśli twój dostawca usług rejestruje wiele prostych powiązań, możesz chcieć użyć właściwości `bindings` i `singletons` zamiast ręcznie rejestrować każde powiązanie kontenera. Gdy dostawca usług zostanie załadowany przez strukturę, automatycznie sprawdzi te właściwości i zarejestruje ich powiązania:

    <?php

    namespace App\Providers;

    use App\Contracts\ServerProvider;
    use App\Contracts\DowntimeNotifier;
    use Illuminate\Support\ServiceProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\DigitalOceanServerProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * All of the container bindings that should be registered.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * All of the container singletons that should be registered.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier,
        ];
    }

<a name="the-boot-method"></a>
### The Boot Method - Metoda rozruchu

A więc, jeśli musimy zarejestrować view composer (kompozytora widoku) w naszym usługodawcy? Powinno to nastąpić w ramach metody `boot`. **Ta metoda jest wywoływana po zarejestrowaniu wszystkich innych dostawców usług**, co oznacza, że masz dostęp do wszystkich innych usług zarejestrowanych w frameworku (ramach):

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### Boot Method Dependency Injection - Metoda rozruch wstrzykniętych zależności

Możesz wpisać wskazówkę zależności dla metody `boot` service provider (dostawcy usług). [service container (Kontener usług)](/docs/{{version}}/container) automatycznie wstrzyknie wszelkie wymagane zależności:

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Registering Providers - Rejestrowanie dostawców

Wszyscy dostawcy usług są zarejestrowani w pliku konfiguracyjnym `config/app.php`. Plik ten zawiera tablicę `providers` , w której można wyświetlić nazwy klas service providers (dostawców usług). Domyślnie zestaw podstawowych dostawców usług Laravel jest wymienionych w tej tablicy. Dostawcy ci uruchamiają podstawowe składniki Laravel, takie jak mailer (program pocztowy), queue (kolejka), cache (pamięć podręczna) i inne.

Aby zarejestrować dostawcę, dodaj go do tablicy:

    'providers' => [
        // Other Service Providers

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Deferred Providers - Odroczeni dostawcy

Jeśli Twój dostawca **tylko** rejestruje powiązania w [service container (kontenerze usługi)](/docs/{{version}}/container), możesz odroczyć jego rejestrację, aż jedno z zarejestrowanych powiązań będzie faktycznie potrzebne. Odroczenie ładowania takiego dostawcy poprawi wydajność aplikacji, ponieważ nie jest ona ładowana z systemu plików na każde żądanie.

Laravel kompiluje i przechowuje listę wszystkich usług dostarczanych przez odroczonych dostawców usług, wraz z nazwą swojej klasy dostawcy usług. Następnie, tylko gdy spróbujesz rozwiązać jedną z tych usług, Laravel ładuje dostawcę usług.

Aby odroczyć ładowanie dostawcy, ustaw właściwość `defer` na `true` zdefiniuj metodę `provides`. Metoda `provides`  powinna zwrócić powiązania kontenera usług zarejestrowane przez dostawcę:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    }
