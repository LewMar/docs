# Cache

- [Configuration - Konfiguracja](#configuration)
    - [Driver Prerequisites - Wymagania wstępne sterownika](#driver-prerequisites)
- [Cache Usage - Wykorzystanie pamięci podręcznej](#cache-usage)
    - [Obtaining A Cache Instance - Uzyskiwanie instancji Cache](#obtaining-a-cache-instance)
    - [Retrieving Items From The Cache - Pobieranie elementów z pamięci podręcznej](#retrieving-items-from-the-cache)
    - [Storing Items In The Cache - Przechowywanie przedmiotów w pamięci podręcznej](#storing-items-in-the-cache)
    - [Removing Items From The Cache - Usuwanie elementów z pamięci podręcznej](#removing-items-from-the-cache)
    - [The Cache Helper - Pomocnik Cache](#the-cache-helper)
- [Cache Tags - Tagi pamięci podręcznej](#cache-tags)
    - [Storing Tagged Cache Items - Przechowywanie oznaczonych elementów pamięci podręcznej](#storing-tagged-cache-items)
    - [Accessing Tagged Cache Items - Dostęp do oznaczonych elementów pamięci podręcznej](#accessing-tagged-cache-items)
    - [Removing Tagged Cache Items - Usuwanie oznaczonych elementów pamięci podręcznej](#removing-tagged-cache-items)
- [Adding Custom Cache Drivers - Dodawanie niestandardowych sterowników pamięci podręcznej](#adding-custom-cache-drivers)
    - [Writing The Driver - Pisanie sterownika](#writing-the-driver)
    - [Registering The Driver - Rejestrowanie sterownika](#registering-the-driver)
- [Events - Zdarzenia](#events)

<a name="configuration"></a>
## Configuration - Konfiguracja

Laravel zapewnia ekspresyjny, zunifikowany interfejs API dla wszystkich pamięci podręcznych. Konfiguracja pamięci podręcznej znajduje się w `config/cache.php`. W tym pliku możesz określić, który sterownik będzie domyślnie używany w całej aplikacji. Laravel obsługuje popularne pamięci podręczne, takie jak [Memcached](https://memcached.org) i [Redis](https://redis.io) po wyjęciu z pudełka.

Plik konfiguracyjny pamięci podręcznej zawiera również inne opcje, które są udokumentowane w pliku, więc pamiętaj o przeczytaniu tych opcji. Domyślnie Laravel jest skonfigurowany do używania sterownika pamięci podręcznej `file`, który przechowuje zserializowane, buforowane obiekty w systemie plików. W przypadku większych aplikacji zaleca się użycie bardziej niezawodnego sterownika, takiego jak Memcached lub Redis. Możesz nawet skonfigurować wiele konfiguracji pamięci podręcznej dla tego samego sterownika.

<a name="driver-prerequisites"></a>
### Driver Prerequisites - Wymagania wstępne sterownika

#### Database

Podczas korzystania ze sterownika bazy danych `database` musisz skonfigurować tabelę, aby zawierała elementy pamięci podręcznej. Znajdziesz przykładową deklarację `Schema` dla poniższej tabeli:

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} Możesz również użyć polecenia `php artisan cache:table`, aby wygenerować migrację z odpowiednim schematem.

#### Memcached

Użycie sterownika Memcached wymaga zainstalowania [Pakiet Memcached PECL](https://pecl.php.net/package/memcached). Możesz wymienić wszystkie swoje serwery Memcached w pliku konfiguracyjnym `config/cache.php`:

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

Możesz także ustawić opcję `host` na ścieżkę gniazda UNIX. Jeśli to zrobisz, opcja `port` powinna być ustawiona na `0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

Przed użyciem pamięci podręcznej Redis z Laravel, musisz albo zainstalować pakiet `predis/predis` (~ 1.0) za pośrednictwem Composer lub zainstalować rozszerzenie PHP PhpRedis za pośrednictwem PECL.

Więcej informacji na temat konfigurowania Redis znajduje się na [stronie dokumentacji Laravel](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>
## Cache Usage - Wykorzystanie pamięci podręcznej

<a name="obtaining-a-cache-instance"></a>
### Obtaining A Cache Instance - Uzyskiwanie instancji Cache

[Kontrakty](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Factory` i `Illuminate\Contracts\Cache\Repository` zapewniają dostęp do usług pamięci podręcznej Laravel. Kontrakt `Factory` zapewnia dostęp do wszystkich sterowników pamięci podręcznej zdefiniowanych dla twojej aplikacji. Kontrakt `Repository` jest zwykle implementacją domyślnego sterownika cache dla twojej aplikacji, określonego przez twój plik konfiguracyjny `cache`.

Możesz jednak użyć elewacji `Cache`, której użyjemy w tej dokumentacji. Fasada `Cache` zapewnia wygodny, zwięzły dostęp do podstawowych implementacji kontraktów pamięci podręcznej Laravel:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### Accessing Multiple Cache Stores - Dostęp do wielu magazynów z pamięcią podręczną

Używając fasady `Cache`, możesz uzyskać dostęp do różnych magazynów pamięci podręcznej za pomocą metody `store`. Klucz przekazany do metody `store` powinien odpowiadać jednemu ze magazynów wymienionych w tablicy konfiguracji `stores` w pliku konfiguracyjnym `cache`:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### Retrieving Items From The Cache - Pobieranie elementów z pamięci podręcznej

Metoda `get` na elewacji `Cache` służy do pobierania elementów z pamięci podręcznej. Jeśli element nie istnieje w pamięci podręcznej, zwracana będzie wartość `null`. Jeśli chcesz, możesz przekazać drugi argument do metody `get` określającej domyślną wartość, która ma zostać zwrócona, jeśli element nie istnieje:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

Możesz nawet przekazać `Closure` jako wartość domyślną. Wynik `Closure` zostanie zwrócony, jeśli określony element nie istnieje w pamięci podręcznej. Przekazanie zamknięcia umożliwia odroczenie pobierania wartości domyślnych z bazy danych lub innej usługi zewnętrznej:

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### Checking For Item Existence - Sprawdzanie istnienia elementu

Metodę `has` można użyć do określenia, czy dany element istnieje w pamięci podręcznej. Ta metoda zwróci wartość `false`, jeśli wartość wynosi `null` lub `false`:

    if (Cache::has('key')) {
        //
    }

#### Incrementing / Decrementing Values - Inkrementowanie / Dekrementowanie wartości

Metody `increment` i `decrement` mogą być użyte do dostosowania wartości elementów całkowitych w pamięci podręcznej. Obie te metody akceptują opcjonalny drugi argument wskazujący kwotę, o którą należy zwiększyć lub zmniejszyć wartość elementu:

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### Retrieve & Store - Pobierz i zapisz

Czasem możesz chcieć odzyskać element z pamięci podręcznej, ale także zapisać domyślną wartość, jeśli żądany element nie istnieje. Na przykład możesz chcieć pobrać wszystkich użytkowników z pamięci podręcznej lub, jeśli nie istnieją, pobrać je z bazy danych i dodać do pamięci podręcznej. Możesz to zrobić za pomocą metody `Cache :: remember`:

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });

Jeśli pozycja nie istnieje w pamięci podręcznej, zostanie wykonane `Closure` przekazane do metody `remember`, a jej wynik zostanie umieszczony w pamięci podręcznej.

Możesz użyć metody `rememberForever`, aby pobrać element z pamięci podręcznej lub przechowywać go na zawsze:

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### Retrieve & Delete - Pobierz i usuń

Jeśli chcesz pobrać element z pamięci podręcznej, a następnie usunąć element, możesz użyć metody `pull`. Podobnie jak w przypadku metody `get`, zwracana jest wartość `null`, jeśli element nie istnieje w pamięci podręcznej:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### Storing Items In The Cache - Przechowywanie przedmiotów w pamięci podręcznej

Możesz użyć metody `put` na elewacji `Cache` do przechowywania elementów w pamięci podręcznej. Po umieszczeniu elementu w pamięci podręcznej należy określić liczbę minut, dla których wartość powinna być przechowywana w pamięci podręcznej:

    Cache::put('key', 'value', $minutes);

Zamiast podawać liczbę minut jako liczbę całkowitą, możesz również przekazać instancję `DateTime` reprezentującą czas wygaśnięcia pozycji w pamięci podręcznej:

    $expiresAt = now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### Store If Not Present - Magazynuj jeśli nie obecne

Metoda `add` doda element tylko do pamięci podręcznej, jeśli jeszcze nie istnieje w magazynie pamięci podręcznej. Metoda zwróci wartość `true`, jeśli element faktycznie zostanie dodany do pamięci podręcznej. W przeciwnym razie metoda zwróci wartość `false`:

    Cache::add('key', 'value', $minutes);

#### Storing Items Forever - Magazynuje przedmioty na zawsze

Metodę `forever` można używać do stałego przechowywania elementu w pamięci podręcznej. Ponieważ te elementy nie wygasną, należy je ręcznie usunąć z pamięci podręcznej za pomocą metody `forget`:

    Cache::forever('key', 'value');

> {tip} Jeśli używasz sterownika Memcached, elementy przechowywane "na zawsze" mogą zostać usunięte, gdy pamięć podręczna osiągnie swój limit rozmiaru.

<a name="removing-items-from-the-cache"></a>
### Removing Items From The Cache - Usuwanie elementów z pamięci podręcznej

Możesz usunąć elementy z pamięci podręcznej za pomocą metody `forget`:

    Cache::forget('key');

Możesz wyczyścić całą pamięć podręczną, używając metody `flush`:

    Cache::flush();

> {note} Płukanie pamięci podręcznej nie respektuje prefiksu pamięci podręcznej i usunie wszystkie wpisy z pamięci podręcznej. Rozważ to ostrożnie, usuwając pamięć podręczną współużytkowaną przez inne aplikacje.

<a name="the-cache-helper"></a>
### The Cache Helper - Pomocnik Cache

Oprócz używania fasady `Cache` lub [kontraktu cache](/docs/{{version}}/contracts), możesz także użyć globalnej funkcji `cache` do pobierania i przechowywania danych poprzez pamięć podręczną. Gdy funkcja `cache` zostanie wywołana z pojedynczym argumentem łańcuchowym, zwróci wartość danego klucza:

    $value = cache('key');

Jeśli udostępnisz tablicę par klucz / wartość i czas wygaśnięcia tej funkcji, będą przechowywać wartości w pamięci podręcznej przez określony czas:

    cache(['key' => 'value'], $minutes);

    cache(['key' => 'value'], now()->addSeconds(10));

> {tip} Podczas testowania wywołania globalnej funkcji `cache` możesz użyć metody `Cache::shouldReceive`, tak jakbyś [testował fasadę](/docs/{{version}}/mocking#mocking-facades).


<a name="cache-tags"></a>
## Cache Tags - Tagi pamięci podręcznej

> {note} Tagi pamięci podręcznej nie są obsługiwane podczas korzystania ze sterowników pamięci podręcznej `file` lub `database`. Ponadto przy korzystaniu z wielu znaczników z pamięcią podręczną, które są przechowywane "na zawsze", wydajność będzie najlepsza w przypadku sterownika, takiego jak "memcached", który automatycznie usuwa nieaktualne rekordy.

<a name="storing-tagged-cache-items"></a>
### Storing Tagged Cache Items - Przechowywanie oznaczonych elementów pamięci podręcznej

Tagi pamięci podręcznej umożliwiają oznaczanie powiązanych elementów w pamięci podręcznej, a następnie opróżnianie wszystkich buforowanych wartości, którym przypisano dany znacznik. Możesz uzyskać dostęp do oznaczonej pamięci podręcznej, przekazując uporządkowaną tablicę nazw znaczników. Na przykład, przejdźmy do otagowanej pamięci podręcznej i wartości `put` w pamięci podręcznej:

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### Accessing Tagged Cache Items - Dostęp do oznaczonych elementów pamięci podręcznej

Aby pobrać oznaczony element pamięci podręcznej, należy przekazać tę samą uporządkowaną listę znaczników do metody `tags`, a następnie wywołać metodę `get` z kluczem, który chcesz odzyskać:

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### Removing Tagged Cache Items - Usuwanie oznaczonych elementów pamięci podręcznej

Możesz opróżnić wszystkie elementy, którym przypisano znacznik lub listę znaczników. Na przykład, to stwierdzenie usunie wszystkie pamięci podręczne oznaczone tagiem `people`,` authors` lub oba. Tak więc, zarówno `Anne`, jak i` John` zostaną usunięte z pamięci podręcznej:

    Cache::tags(['people', 'authors'])->flush();

W przeciwieństwie do tego, to stwierdzenie usunie tylko pamięci podręczne oznaczone `authors`, więc `Anne` zostanie usunięte, ale nie `John`:

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## Adding Custom Cache Drivers - Dodawanie niestandardowych sterowników pamięci podręcznej

<a name="writing-the-driver"></a>
### Writing The Driver - Pisanie sterownika

Aby utworzyć nasz niestandardowy sterownik pamięci podręcznej, musimy najpierw wdrożyć [kontrakt](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Tak więc implementacja pamięci podręcznej MongoDB wyglądałaby tak:

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Musimy tylko wdrożyć każdą z tych metod za pomocą połączenia MongoDB. Aby zobaczyć przykład wdrożenia każdej z tych metod, spójrz na `Illuminate\Cache\MemcachedStore` w kodzie źródłowym frameworka. Po zakończeniu implementacji możemy zakończyć naszą niestandardową rejestrację sterownika.

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} Jeśli zastanawiasz się, gdzie umieścić niestandardowy kod sterownika pamięci podręcznej, możesz utworzyć przestrzeń nazw `Extensions` w swoim katalogu `app`. Pamiętaj jednak, że Laravel nie ma sztywnej struktury aplikacji i możesz dowolnie organizować swoją aplikację zgodnie z własnymi preferencjami.

<a name="registering-the-driver"></a>
### Registering The Driver - Rejestrowanie sterownika

Aby zarejestrować niestandardowy sterownik pamięci podręcznej za pomocą Laravel, użyjemy metody `extend` na elewacji `Cache`. Wywołanie `Cache::extend` może być wykonane w metodzie `boot` domyślnej `App\Providers\AppServiceProvider`, która jest dostarczana ze świeżymi aplikacjami Laravel, lub możesz stworzyć własnego dostawcę usług, aby pomieścić rozszerzenie - po prostu nie zapomnij zarejestrować dostawcę w tablicy dostawcy `config/app.php`:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Pierwszym argumentem przekazanym do metody `extend` jest nazwa sterownika. Odpowiada to opcji `driver` w pliku konfiguracyjnym `config/cache.php`. Drugi argument to Closure, które powinno zwrócić instancję  `Illuminate\Cache\Repository`. Closure zostanie przekazane instancji `$app`, która jest instancją [kontenera usług](/docs/{{version}}/container).

Po zarejestrowaniu rozszerzenia zaktualizuj opcję sterownika `config/cache.php` do nazwy swojego rozszerzenia.

<a name="events"></a>
## Events - Zdarzenia

Aby wykonać kod dla każdej operacji w pamięci podręcznej, można nasłuchiwać [zdarzeń](/docs/{{version}}/events) uruchamianych przez pamięć podręczną. Zazwyczaj powinieneś umieścić te detektory zdarzeń w swoim `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
