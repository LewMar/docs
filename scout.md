# Laravel Scout

- [Introduction - Wprowadzenie](#introduction)
- [Installation - Instalacja](#installation)
    - [Queueing - Kolejkowanie](#queueing)
    - [Driver Prerequisites - Wymagania wstępne sterownika](#driver-prerequisites)
- [Configuration - Konfiguracja](#configuration)
    - [Configuring Model Indexes - Konfigurowanie indeksów modeli](#configuring-model-indexes)
    - [Configuring Searchable Data - Konfigurowanie przeszukiwalnych danych](#configuring-searchable-data)
- [Indexing - Indeksowanie](#indexing)
    - [Batch Import - Import partii](#batch-import)
    - [Adding Records - Dodawanie rekordów](#adding-records)
    - [Updating Records - Aktualizowanie rekordów](#updating-records)
    - [Removing Records - Usuwanie rekordów](#removing-records)
    - [Pausing Indexing - Wstrzymywanie indeksowania](#pausing-indexing)
    - [Conditionally Searchable Model Instances - Instancja modelu warunkowego wyszukiwania](#conditionally-searchable-model-instances)
- [Searching - Wyszukiwanie](#searching)
    - [Where Clauses - Klauzura Gdzie](#where-clauses)
    - [Pagination - Paginacja](#pagination)
- [Custom Engines - Niestandardowe silniki](#custom-engines)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel Scout dostarcza proste, oparte na sterownikach rozwiązanie do dodawania wyszukiwania pełnotekstowego do [Eloquent models](/docs/{{version}}/eloquent). Korzystając z obserwatorów modeli, Scout automatycznie zsynchronizuje indeksy wyszukiwania z Twoimi Eloquent rekordami.

Obecnie statki poszukiwawcze ze sterownikiem [Algolia](https://www.algolia.com/); jednak pisanie niestandardowych sterowników jest proste i możesz rozszerzyć Scout z własnymi implementacjami wyszukiwania.

<a name="installation"></a>
## Installation - Instalacja

Najpierw zainstaluj Scouta za pomocą menedżera pakietów Composer:

    composer require laravel/scout

Po zainstalowaniu Scout, powinieneś opublikować konfigurację Scout używając polecenia `vendor:publish` Artisan. To polecenie opublikuje plik konfiguracyjny `scout.php` w twoim katalogu` config`:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

Na koniec dodaj cechę `Laravel\Scout\Searchable` do modelu, który chcesz przeszukiwać. Ta cecha spowoduje zarejestrowanie modelu obserwatora, aby zachować synchronizację modelu ze sterownikiem wyszukiwania:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### Queueing - Kolejkowanie

Chociaż nie jest to bezwzględnie wymagane do korzystania z Scout, powinieneś zdecydowanie rozważyć skonfigurowanie [sterownika kolejki](/docs/{{version}}/queues) przed użyciem biblioteki. Uruchomienie modułu kolejki pozwoli Scoutowi na umieszczenie w kolejce wszystkich operacji synchronizujących informacje o twoim modelu z indeksami wyszukiwania, zapewniając znacznie lepsze czasy odpowiedzi dla interfejsu WWW aplikacji.

Po skonfigurowaniu sterownika kolejki ustaw wartość opcji `queue` w pliku konfiguracyjnym `config/scout.php` na `true`:

    'queue' => true,

<a name="driver-prerequisites"></a>
### Driver Prerequisites - Wymagania wstępne sterownika

#### Algolia

Korzystając ze sterownika Algolia, należy skonfigurować swoje poświadczenia `id` i `secret` Algolii w pliku konfiguracyjnym `config/scout.php`. Po skonfigurowaniu poświadczeń będziesz również musiał zainstalować pakiet SDK Algolia PHP za pośrednictwem menedżera pakietów Composer:

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## Configuration - Konfiguracja

<a name="configuring-model-indexes"></a>
### Configuring Model Indexes - Konfigurowanie indeksów modeli

Każdy model Eloquent jest zsynchronizowany z danym "indeksem wyszukiwania", który zawiera wszystkie możliwe do przeszukiwania rekordy dla tego modelu. Innymi słowy, możesz myśleć o każdym indeksie jak o tabeli MySQL. Domyślnie każdy model będzie trwał do indeksu pasującego do typowej nazwy "tabeli" modelu. Zazwyczaj jest to liczba mnoga nazwy modelu; można jednak dostosować indeks modelu, zastępując metodę `searchableAs` na modelu:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### Configuring Searchable Data - Konfigurowanie przeszukiwalnych danych

Domyślnie cała forma `toArray` danego modelu będzie zachowana do indeksu wyszukiwania. Jeśli chcesz dostosować dane zsynchronizowane z indeksem wyszukiwania, możesz zastąpić metodę `toSearchableArray` na modelu:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="indexing"></a>
## Indexing - Indeksowanie

<a name="batch-import"></a>
### Batch Import - Import partii

Jeśli instalujesz Scouta w istniejącym projekcie, możesz już mieć rekordy bazy danych, które musisz zaimportować do sterownika wyszukiwania. Scout dostarcza polecenie `import` Artisan, którego możesz użyć do zaimportowania wszystkich twoich istniejących rekordów do indeksów wyszukiwania:

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### Adding Records - Dodawanie rekordów

Po dodaniu cechy `Laravel\Scout\Searchable` do modelu, wszystko, co musisz zrobić, to `save` instancję modelu i zostanie automatycznie dodana do twojego indeksu wyszukiwania. Jeśli skonfigurowałeś Scouta na [używaj kolejek](#queueing), operacja ta będzie wykonywana w tle przez pracownika kolejki:

    $order = new App\Order;

    // ...

    $order->save();

#### Adding Via Query - Dodawanie przez zapytania

Jeśli chcesz dodać kolekcję modeli do indeksu wyszukiwania za pomocą kwerendy Eloquent, możesz powiązać metodę `searchable` z kwerendą Eloquent. Metoda `searchable` spowoduje [podzielenie wyników](/docs/{{version}}/eloquent#chunking-results) zapytania i dodanie rekordów do twojego indeksu wyszukiwania. Ponownie, jeśli skonfigurowałeś Scout do używania kolejek, wszystkie porcje zostaną dodane w tle przez pracowników kolejki:

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also add records via relationships...
    $user->orders()->searchable();

    // You may also add records via collections...
    $orders->searchable();

Metoda `searchable` może być uważana za operację "upsert". Innymi słowy, jeśli rekord modelu jest już w twoim indeksie, zostanie zaktualizowany. Jeśli nie istnieje w indeksie wyszukiwania, zostanie dodany do indeksu.

<a name="updating-records"></a>
### Updating Records - Aktualizowanie rekordów

Aby zaktualizować model z możliwością wyszukiwania, wystarczy zaktualizować właściwości instancji modelu i "zapisać" model w bazie danych. Scout automatycznie utrzyma zmiany w twoim indeksie wyszukiwania:

    $order = App\Order::find(1);

    // Update the order...

    $order->save();

Możesz także użyć metody `searchable` w zapytaniu Eloquent, aby zaktualizować kolekcję modeli. Jeśli modele nie istnieją w indeksie wyszukiwania, zostaną utworzone:

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also update via relationships...
    $user->orders()->searchable();

    // You may also update via collections...
    $orders->searchable();

<a name="removing-records"></a>
### Removing Records - Usuwanie rekordów

Aby usunąć rekord z indeksu, `delete` model z bazy danych. Ta forma usuwania jest nawet zgodna z modelami [miękkiego usunięcia](/docs/{{version}}/eloquent#soft-deleting):

    $order = App\Order::find(1);

    $order->delete();

Jeśli nie chcesz pobrać modelu przed usunięciem rekordu, możesz użyć metody `unsearchable` w instancji lub kolekcji Eloquent query:

    // Removing via Eloquent query...
    App\Order::where('price', '>', 100)->unsearchable();

    // You may also remove via relationships...
    $user->orders()->unsearchable();

    // You may also remove via collections...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Pausing Indexing - Wstrzymywanie indeksowania

Czasami może zajść potrzeba wykonania partii Wymownych operacji na modelu bez synchronizacji danych modelu z indeksem wyszukiwania. Możesz to zrobić za pomocą metody `withoutSyncingToSearch`. Ta metoda akceptuje pojedyncze wywołanie zwrotne, które zostanie natychmiast wykonane. Wszystkie operacje modelu, które występują w ramach wywołania zwrotnego, nie zostaną zsynchronizowane z indeksem modelu:

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="conditionally-searchable-model-instances"></a>
### Conditionally Searchable Model Instances - Instancja modelu warunkowego wyszukiwania

Czasami może zajść potrzeba tylko przeszukiwania modelu pod pewnymi warunkami. Na przykład wyobraź sobie, że masz model `App\Post`, który może znajdować się w jednym z dwóch stanów: "draft" i "published". Możesz chcieć wyszukiwać tylko "opublikowane" posty. Aby to osiągnąć, możesz zdefiniować metodę `shouldBeSearchable` na swoim modelu:

    public function shouldBeSearchable()
    {
        return $this->isPublished();
    }

<a name="searching"></a>
## Searching - Wyszukiwanie

Możesz rozpocząć wyszukiwanie modelu za pomocą metody `search`. Metoda wyszukiwania akceptuje pojedynczy ciąg, który będzie używany do wyszukiwania modeli. Następnie powiąż metodę `get` z zapytaniem, aby pobrać modele wymowne, które pasują do podanego zapytania:

    $orders = App\Order::search('Star Trek')->get();

Ponieważ wyszukiwania Scout zwracają kolekcję modeli wymownych, możesz nawet zwrócić wyniki bezpośrednio z trasy lub kontrolera, a następnie zostaną one automatycznie przekonwertowane na JSON:

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

Jeśli chcesz uzyskać nieprzetworzone wyniki, zanim zostaną przekonwertowane do modeli Eloquent, powinieneś użyć metody `raw`:

    $orders = App\Order::search('Star Trek')->raw();

Zapytania wyszukiwania są zwykle wykonywane na indeksie określonym przez metodę [`searchableAs`](#configuring-model-indexes) modelu. Możesz jednak użyć metody `within` do określenia niestandardowego indeksu, który powinien zostać przeszukany:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Where Clauses - Klauzura Gdzie

Scout pozwala na dodanie prostych zapytań "where" do wyszukiwanych haseł. Obecnie klauzule te obsługują jedynie podstawowe kontrole równości i są przede wszystkim przydatne przy wyszukiwaniu według dzierżawy zapytania. Ponieważ indeks wyszukiwania nie jest relacyjną bazą danych, bardziej zaawansowane klauzule "where" nie są obecnie obsługiwane:

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Pagination - Paginacja

Poza pobieraniem kolekcji modeli możesz również paginować swoje wyniki wyszukiwania za pomocą metody `paginate`. Ta metoda zwróci instancję `Paginator` tak samo, jakbyś [utworzył tradycyjne zapytanie Eloquent](/docs/{{version}}/pagination):

    $orders = App\Order::search('Star Trek')->paginate();

Możesz określić liczbę modeli do pobrania na stronę, przekazując ilość jako pierwszy argument do metody `paginate`:

    $orders = App\Order::search('Star Trek')->paginate(15);

Po pobraniu wyników możesz wyświetlić wyniki i wyrenderować łącza do stron za pomocą [Blade](/docs/{{version}}/blade) tak, jakbyś napisał stronę z tradycyjnym zapytanie Eloquent:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="custom-engines"></a>
## Custom Engines - Niestandardowe silniki

#### Writing The Engine - Pisanie silnika

Jeśli któryś z wbudowanych wyszukiwarek Scout nie spełnia Twoich wymagań, możesz napisać własny, niestandardowy silnik i zarejestrować go w Scout. Twój silnik powinien rozszerzyć klasę abstrakcyjną `Laravel\Scout\Engines\Engine`. Ta abstrakcyjna klasa zawiera siedem metod, które Twój niestandardowy silnik musi implementować:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map($results, $model);
    abstract public function getTotalCount($results);

Pomocne może być przejrzenie implementacji tych metod w klasie `Laravel\Scout\Engines\AlgoliaEngine`. Ta klasa zapewni ci dobry punkt wyjścia do nauki wdrażania każdej z tych metod we własnym silniku.

#### Registering The Engine - Rejestracja silnika

Po napisaniu własnego silnika możesz go zarejestrować w Scout używając metody `extend` menedżera wyszukiwarki Scout. Powinieneś wywołać metodę `extend` z metody `boot` swojego `AppServiceProvider` lub dowolnego innego dostawcy usług używanego przez twoją aplikację. Na przykład, jeśli napisałeś `MySqlSearchEngine`, możesz zarejestrować go tak:

    use Laravel\Scout\EngineManager;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

Po zarejestrowaniu silnika możesz określić go jako domyślny `driver` Scouta w pliku konfiguracyjnym `config/scout.php`:

    'driver' => 'mysql',
