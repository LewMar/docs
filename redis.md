# Redis

- [Introduction - Wprowadzenie(#introduction)
    - [Configuration - Konfiguracja](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Interacting With Redis - Interakcja z Redis](#interacting-with-redis)
    - [Pipelining Commands - Polecenia potokowania](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Introduction - Wprowadzenie

[Redis](https://redis.io) to open source-owy, zaawansowany magazyn kluczy i wartości. Jest często określany jako serwer struktury danych, ponieważ klucze mogą zawierać [łańcuchy znaków](https://redis.io/topics/data-types#strings), [hasze](https://redis.io/topics/data-types#hashes), [listy](https://redis.io/topics/data-types#lists), [zestawy](https://redis.io/topics/data-types#sets), and [posortowane zestawy](https://redis.io/topics/data-types#sorted-sets).

Przed użyciem Redis z Laravel, będziesz musiał zainstalować pakiet `predis/predis` przez Composer:

    composer require predis/predis

Alternatywnie możesz zainstalować rozszerzenie PHP [PhpRedis](https://github.com/phpredis/phpredis) przez PECL. Rozszerzenie jest bardziej skomplikowane w instalacji, ale może zapewnić lepszą wydajność aplikacji, które intensywnie wykorzystują Redis.

<a name="configuration"></a>
### Configuration - Konfiguracja

Konfiguracja Redis dla twojej aplikacji znajduje się w pliku konfiguracyjnym `config/database.php`. W tym pliku zobaczysz tablicę `redis` zawierającą serwery Redis wykorzystywane przez twoją aplikację:

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

Domyślna konfiguracja serwera powinna wystarczyć do rozwoju. Możesz jednak modyfikować tę tablicę w zależności od środowiska. Każdy serwer Redis zdefiniowany w pliku konfiguracyjnym musi mieć nazwę, host i port.

#### Configuring Clusters - Konfigurowanie klastrów

Jeśli twoja aplikacja korzysta z klastra serwerów Redis, powinieneś zdefiniować te klastry w kluczu `clusters` w konfiguracji Redis:

    'redis' => [

        'client' => 'predis',

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

Domyślnie klastry będą wykonywać odseparowanie po stronie klienta między węzłami, pozwalając na łączenie węzłów i tworzenie dużej ilości dostępnej pamięci RAM. Należy jednak pamiętać, że sharding po stronie klienta nie obsługuje przełączania awaryjnego; w związku z tym nadaje się przede wszystkim do przechowywania danych w pamięci podręcznej, które są dostępne z innej głównej składnicy danych. Jeśli chcesz używać natywnego klastrowania Redis, powinieneś to podać w kluczu `options` w konfiguracji Redis:

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

Oprócz domyślnych opcji konfiguracyjnych `host`, `port`, `database` i `password`, Predis obsługuje dodatkowe [parametry połączenia](https://github.com/nrk/predis/wiki/Connection-Parameters), które można zdefiniować dla każdego z serwerów Redis. Aby wykorzystać te dodatkowe opcje konfiguracji, po prostu dodaj je do konfiguracji serwera Redis w pliku konfiguracyjnym `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

> {note} Jeśli masz zainstalowane rozszerzenie PHP PhpRedis za pośrednictwem PECL, będziesz musiał zmienić nazwę aliasu Redis w pliku konfiguracyjnym `config/app.php`.

Aby móc korzystać z rozszerzenia PhpRedis, powinieneś zmienić opcję `client` swojej konfiguracji Redis na `phpredis`. Ta opcja znajduje się w pliku konfiguracyjnym `config/database.php`:

    'redis' => [

        'client' => 'phpredis',

        // Rest of Redis configuration...
    ],

Oprócz domyślnych opcji konfiguracyjnych `host`,` port`, `database` i `password`, PhpRedis obsługuje następujące dodatkowe parametry połączenia: `persistent`, `prefix`, `read_timeout` oraz `timeout`. Możesz dodać dowolną z tych opcji do konfiguracji serwera Redis w pliku konfiguracyjnym `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

<a name="interacting-with-redis"></a>
## Interacting With Redis - Interakcja z Redis

Możesz wchodzić w interakcje z Redis, wywołując różne metody na `Redis` [elewacji](/docs/{{version}}/facades)). Fasada `Redis` obsługuje metody dynamiczne, co oznacza, że możesz wywołać dowolne polecenie [Redis](https://redis.io/commands) na elewacji, a polecenie zostanie przekazane bezpośrednio do Redis. W tym przykładzie wywołajmy komendę Redis `GET`, wywołując metodę `get` na elewacji `Redis`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

Oczywiście, jak wspomniano powyżej, możesz wywołać dowolne z poleceń Redis na fasadzie `Redis`. Laravel używa magicznych metod do przekazywania poleceń do serwera Redis, więc po prostu przekazuj argumenty, których oczekuje polecenie Redis:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

Alternatywnie możesz także przekazywać polecenia do serwera, używając metody `command`, która przyjmuje nazwę polecenia jako swój pierwszy argument, oraz tablicę wartości jako jej drugi argument:

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Using Multiple Redis Connections - Korzystanie z wielu połączeń Redis

Możesz uzyskać instancję Redis, wywołując metodę `Redis::connection`:

    $redis = Redis::connection();

To da ci instancję domyślnego serwera Redis. Możesz również przekazać nazwę połączenia lub klastra do metody `connection`, aby uzyskać określony serwer lub klaster, zgodnie z definicją w konfiguracji Redis:

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### Pipelining Commands - Polecenia potokowania

Pipelining powinien być używany, gdy potrzebujesz wysłać wiele komend do serwera w jednej operacji. Metoda `pipeline` przyjmuje jeden argument: `Closure`, które otrzymuje instancję Redis. Możesz wydać wszystkie swoje polecenia do tej instancji Redis i wszystkie zostaną wykonane w ramach jednej operacji:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel zapewnia wygodny interfejs do poleceń Redis `publish` i `subscribe`. Te polecenia Redis umożliwiają słuchanie wiadomości na danym "kanale". Możesz publikować wiadomości na kanale z innej aplikacji lub nawet używać innego języka programowania, umożliwiając łatwą komunikację pomiędzy aplikacjami i procesami.

Najpierw skonfiguruj detektor kanału za pomocą metody `subscribe`. Wywołanie tej metody zostanie wykonane za pomocą polecenia [Artisan](/docs/{{version}}/artisan), ponieważ wywołanie metody `subscribe` rozpoczyna długotrwały proces:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

Teraz możemy publikować wiadomości na kanale przy użyciu metody `publish`:

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### Wildcard Subscriptions - Subskrypcje wieloznaczne

Korzystając z metody `psubscribe`, możesz zasubskrybować kanał wieloznaczny, który może być przydatny do przechwytywania wszystkich wiadomości na wszystkich kanałach. Nazwa `$channel` zostanie przekazana jako drugi argument do dostarczonego wywołania zwrotnego  `Closure`:

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
