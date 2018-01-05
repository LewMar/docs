# Eloquent: Getting Started

- [Introduction - Wprowadzenie](#introduction)
- [Defining Models - Definiowanie modeli](#defining-models)
    - [Eloquent Model Conventions - Konwencje elokwentnych modeli](#eloquent-model-conventions)
- [Retrieving Models - Pobieranie modeli](#retrieving-models)
    - [Collections - Kolekcje](#collections)
    - [Chunking Results - Porcjowanie wyników](#chunking-results)
- [Retrieving Single Models / Aggregates - Pobieranie pojedynczych modeli / agregatów](#retrieving-single-models)
    - [Retrieving Aggregates - Pobieranie agregatów](#retrieving-aggregates)
- [Inserting & Updating Models - Wstawianie i aktualizowanie modeli](#inserting-and-updating-models)
    - [Inserts - Wstawki](#inserts)
    - [Updates - Aktualizacje](#updates)
    - [Mass Assignment - Masowe przypisania](#mass-assignment)
    - [Other Creation Methods - Inne metody tworzące](#other-creation-methods)
- [Deleting Models - Usuwanie modeli](#deleting-models)
    - [Soft Deleting - Miękkie usuwanie](#soft-deleting)
    - [Querying Soft Deleted Models - Zapytanie na miękko usuniętych modelach](#querying-soft-deleted-models)
- [Query Scopes - Zakres zapytań](#query-scopes)
    - [Global Scopes - Zakres globaln](#global-scopes)
    - [Local Scopes - Lokalne zakresy](#local-scopes)
- [Events - Wydarzenia](#events)
    - [Observers - Obserwatorzy](#observers)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Eloquent ORM dołączony do Laravel zapewnia piękną, prostą implementację ActiveRecord do pracy z bazą danych. Każda tabela bazy danych ma odpowiedni "Model", który służy do interakcji z tą tabelą. Modele umożliwiają wyszukiwanie danych w tabelach, a także wstawianie nowych rekordów do tabeli.

Przed rozpoczęciem należy skonfigurować połączenie z bazą danych w `config/database.php`. Aby uzyskać więcej informacji na temat konfigurowania bazy danych, sprawdź [dokumentację](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>
## Defining Models - Definiowanie modeli

Aby rozpocząć, stwórzmy model elokwentny (Eloquent). Modele zazwyczaj znajdują się w katalogu `app`, ale możesz umieścić je w dowolnym miejscu, które mogą być automatycznie ładowane zgodnie z plikiem `composer.json`. Wszystkie modele Eloquent rozszerzają klasę `Illuminate\Database\Eloquent\Model`.

Najłatwiejszym sposobem utworzenia instancji modelu jest użycie `make:model` [polecenie Artisan](/docs/{{version}}/artisan):

    php artisan make:model User

Jeśli chcesz wygenerować [migracja bazy danych](/docs/{{version}}/migrations) podczas generowania modelu, możesz użyć opcji `--migration` lub` -m`:

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquent Model Conventions - Konwencje elokwentnych modeli

Teraz spójrzmy na przykładowy model `Flight`, który użyjemy do pobierania i przechowywania informacji z naszej tabeli bazy danych `flights`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }

#### Table Names

Zwróć uwagę, że nie powiedzieliśmy Eloquent-owi, która tabela ma być używana w naszym modelu `Flight`. Zgodnie z konwencją "snake case", nazwa w liczbie mnogiej klasy, będzie używana jako nazwa tabeli, chyba że inna nazwa zostanie jawnie określona. Tak więc w tym przypadku, Eloquent weźmie model `Flight` i przechowa zapisy w tabeli `flights`. Możesz określić niestandardową tabelę, definiując właściwość `table` w swoim modelu:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### Primary Keys - Klucze podstawowe

Eloquent zakłada również, że każda tabela ma kolumnę klucza podstawowego o nazwie `id`. Możesz zdefiniować chronioną własność  `$primaryKey`, aby zastąpić tę konwencję.

Ponadto, Eolquent zakłada, że klucz podstawowy jest zwiększającą się liczbą całkowitą, co oznacza, że domyślnie klucz podstawowy zostanie automatycznie przeniesiony do `int`. Jeśli chcesz użyć nie inkrementujacego lub nieliczbowego klucza podstawowego, musisz ustawić publiczną właściwość `$incrementing` w swoim modelu na `false`. Jeśli twój klucz podstawowy nie jest liczbą całkowitą, powinieneś ustawić chronioną właściwość `$keyType` w twoim modelu na `string`.

#### Timestamps - Znaczniki czasu

Domyślnie, Eloquent oczekuje, że kolumny `created_at` i `updated_at` będą istniały w twoich tabelach. Jeśli nie chcesz automatycznie zarządzać tymi kolumnami przez Eloquent, ustaw właściwość `$timestamps` w swoim modelu na` false`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Jeśli chcesz dostosować format znaczników czasu, ustaw właściwość `$dateFormat` w swoim modelu. Ta właściwość określa sposób przechowywania atrybutów daty w bazie danych, a także ich format, gdy model jest serializowany do tablicy lub JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Jeśli chcesz dostosować nazwy kolumn używanych do przechowywania znaczników czasu, możesz ustawić stałe `CREATED_AT` i `UPDATED_AT` w swoim modelu:

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### Database Connection - Połączenie z bazą danych

Domyślnie wszystkie modele Eloquent będą używać domyślnego połączenia z bazą danych skonfigurowanego dla aplikacji. Jeśli chcesz określić inne połączenie dla modelu, użyj właściwości `$connection`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The connection name for the model.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="retrieving-models"></a>
## Retrieving Models - Pobieranie modeli

Po utworzeniu modelu i [powiązanej z nim tabeli bazy danych](/docs/{{version}}/migrations#writing-migrations) można rozpocząć pobieranie danych z bazy danych. Pomyśl o każdym elokwentnym modelu jako potężnym [konstruktorze zapytań](/docs/{{version}}/queries), dzięki czemu możesz płynnie wyszukiwać tabelę bazy danych powiązaną z modelem. Na przykład:

    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### Adding Additional Constraints - Dodawanie dodatkowych ograniczeń

Metoda Eloquent `all` zwróci wszystkie wyniki w tabeli modelu. Ponieważ każdy model Eloquent służy jako [konstruktor zapytań](/docs/{{version}}/queries), możesz także dodawać ograniczenia do zapytań, a następnie użyć metody `get` w celu pobrania wyników:

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Ponieważ modele Eloquent to generatory zapytań, powinieneś przejrzeć wszystkie metody dostępne w [konstruktorze zapytań](/docs/{{version}}/queries). Możesz użyć dowolnej z tych metod w swoich elokwentnych zapytaniach.

<a name="collections"></a>
### Collections - Kolekcje

Dla metod Eloquent, takich jak `all` i` get`, które pobierają wiele wyników, zostanie zwrócona instancja `Illuminate\Database\Eloquent\Collection`. Klasa `Collection` dostarcza [różnorodne pomocne metody](/docs/{{version}}/eloquent-collections#available-methods) do pracy z Twoimi elokwentnymi wynikami:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

Oczywiście możesz po prostu zapętlić kolekcję jak tablicę:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### Chunking Results - Porcjowanie wyników

Jeśli chcesz przetworzyć tysiące wymownych rekordów, użyj polecenia `chunk`. Metoda `chunk` pobiera "porcję" modeli Eloquent, podając je do danego `Closure` do przetworzenia. Użycie metody `chunk` pozwoli zaoszczędzić pamięć podczas pracy z dużymi zestawami wyników:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

Pierwszym argumentem przekazanym do tej metody jest liczba rekordów, które chcesz otrzymać na "porcję". Zamknięcie przekazane jako drugi argument zostanie wywołany dla każdego fragmentu, który jest pobierany z bazy danych. Zapytanie do bazy danych zostanie wykonane w celu pobrania każdego fragmentu rekordów przekazanych do zamknięcia (Closure).

#### Using Cursors - Używanie kursorów

Metoda `cursor` pozwala na iterowanie rekordów bazy danych za pomocą kursora, który wykonuje tylko jedno zapytanie. Podczas przetwarzania dużych ilości danych można użyć metody `cursor` w celu znacznego zmniejszenia wykorzystania pamięci:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## Retrieving Single Models / Aggregates - Pobieranie pojedynczych modeli / agregatów

Oczywiście, oprócz pobierania wszystkich rekordów dla danej tabeli, możesz także pobierać pojedyncze rekordy używając `find` lub` first`. Zamiast zwracania kolekcji modeli, metody te zwracają pojedynczą instancję modelu:

    // Retrieve a model by its primary key...
    $flight = App\Flight::find(1);

    // Retrieve the first model matching the query constraints...
    $flight = App\Flight::where('active', 1)->first();

Możesz również wywołać metodę `find` z tablicą kluczy podstawowych, która zwróci kolekcję pasujących rekordów:

    $flights = App\Flight::find([1, 2, 3]);

#### Not Found Exceptions - Nie znaleziono wyjątków

rzydatne w przypadku tras lub kontrolerów. Metody `findOrFail` i `firstOrFail` będą pobierać pierwszy wynik zapytania; jeśli jednak nie zostanie znaleziony żaden wynik, zostanie zgłoszony wyjątek `Illuminate\Database\Eloquent\ModelNotFoundException`:

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

Jeśli wyjątek nie zostanie przechwycony, odpowiedź HTTP `404` jest automatycznie wysyłana do użytkownika. Nie jest konieczne pisanie jawnych sprawdzeń, aby zwrócić odpowiedzi `404` podczas używania tych metod:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### Retrieving Aggregates - Pobieranie agregatów

Możesz także użyć `count`,` sum`, `max` i innych [metod agregacji](/docs/{{version}}/queries#aggregates) dostarczonych przez [konstruktora zapytań](/docs/{{version}}/queries). Te metody zwracają odpowiednią wartość skalarną zamiast pełnej instancji modelu:

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Inserting & Updating Models - Wstawianie i aktualizowanie modeli

<a name="inserts"></a>
### Inserts - Wstawki

Aby utworzyć nowy rekord w bazie danych, wystarczy utworzyć nową instancję modelu, ustawić atrybuty w modelu, a następnie wywołać metodę `save`:

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

In this example, we simply assign the `name` parameter from the incoming HTTP request to the `name` attribute of the `App\Flight` model instance. When we call the `save` method, a record will be inserted into the database. The `created_at` and `updated_at` timestamps will automatically be set when the `save` method is called, so there is no need to set them manually.

<a name="updates"></a>
### Updates - Aaktualizacje

Metodę `save` można również użyć do aktualizacji modeli, które już istnieją w bazie danych. Aby zaktualizować model, powinieneś go pobrać, ustawić wszystkie atrybuty, które chcesz zaktualizować, a następnie wywołać metodę `save`. Ponownie, znacznik czasu `updated_at` zostanie automatycznie zaktualizowany, więc nie ma potrzeby ręcznego ustawiania jego wartości:

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### Mass Updates - Masowe aktualizacje

Aktualizacje można również wykonać w stosunku do dowolnej liczby modeli pasujących do danego zapytania. W tym przykładzie wszystkie loty, które są `active` i mają `destination` do `San Diego` będą oznaczone jako opóźnione:

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

Metoda `update` oczekuje tablicy kolumn i wartości reprezentujących kolumny, które powinny zostać zaktualizowane.

> {note} Podczas wydawania aktualizacji masowej za pośrednictwem Eloquent zdarzenia modelu `saved` i `updated` nie będą uruchamiane w przypadku zaktualizowanych modeli. Wynika to z tego, że modele nigdy nie są pobierane podczas wydawania aktualizacji masymasowych.

<a name="mass-assignment"></a>
### Mass Assignment - Masowe przypisania

Możesz także użyć metody `create`, aby zapisać nowy model w jednym wierszu. Wstawiona instancja modelu zostanie zwrócona do ciebie z metody. Jednak zanim to zrobisz, będziesz musiał określić atrybut `fillable` lub` guarded` na modelu, ponieważ wszystkie modele Eloquent domyślnie chronione przed przenoszeniem masowym.


Luka polegająca na masowym przypisywaniu występuje, gdy użytkownik przekazuje nieoczekiwany parametr HTTP przez żądanie, a ten parametr zmienia kolumnę w bazie danych, której się nie spodziewałeś. Na przykład złośliwy użytkownik może wysłać parametr `is_admin` przez żądanie HTTP, które następnie zostanie przekazane do metody `create` używanej przez twój model, pozwalając użytkownikowi na przełaczenie sie na administratora.

Aby rozpocząć, należy zdefiniować atrybuty modelu, które mają zostać przypisane masowo. Możesz to zrobić za pomocą właściwości `$fillable` na modelu. Na przykład, zróbmy ten `name` atrybut naszego modelu` Flight` do masowego przypisania:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Po dodaniu masy atrybutów możemy użyć metody `create` do wstawienia nowego rekordu do bazy danych. Metoda `create` zwraca zapisaną instancję modelu:

    $flight = App\Flight::create(['name' => 'Flight 10']);

Jeśli masz już instancję modelu, możesz użyć metody `fill`, aby wypełnić ją tablicą atrybutów:

    $flight->fill(['name' => 'Flight 22']);

#### Guarding Attributes - Ochrona atrybutów

While `$fillable` serves as a "white list" of attributes that should be mass assignable, you may also choose to use `$guarded`. The `$guarded` property should contain an array of attributes that you do not want to be mass assignable. All other attributes not in the array will be mass assignable. So, `$guarded` functions like a "black list". Of course, you should use either `$fillable` or `$guarded` - not both. In the example below, all attributes **except for `price`** will be mass assignable:
Podczas gdy `$fillable` służy jako "biała lista" atrybutów, które powinny być przypisywane masowo, możesz również użyć `$guarded`. Właściwość `$guarded` powinna zawierać tablicę atrybutów, których nie chcesz przypisywać masowoy. Wszystkie pozostałe atrybuty spoza tablicy będą masowo przypisywane. Tak więc `$guarded` działa jak " zarna lista". Oczywiście powinieneś użyć `$fillable` lub `$guarded` - nie obu. W poniższym przykładzie wszystkie atrybuty **z wyjątkiem `price`** będą masowo przypisywane:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

Jeśli chcesz ustawić przypisanie wszystkich atrybutów mas, możesz zdefiniować własność `$guarded` jako pustą tablicę:

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### Other Creation Methods - Inne metody tworzące

#### `firstOrCreate`/ `firstOrNew`

Istnieją dwie inne metody, których możesz użyć do tworzenia modeli poprzez masowe przypisywanie atrybutów: `firstOrCreate` i `firstOrNew`. Metoda `firstOrCreate` spróbuje zlokalizować rekord bazy danych przy użyciu podanych par kolumn / wartości. Jeśli nie można znaleźć modelu w bazie danych, zostanie wstawiony rekord z atrybutami z pierwszego parametru, a także z opcjonalnego drugiego parametru.

Metoda `firstOrNew`, np. `FirstOrCreate`, spróbuje zlokalizować rekord w bazie danych pasujący do podanych atrybutów. Jeśli jednak nie zostanie znaleziony model, zostanie zwrócona nowa instancja modelu. Zwróć uwagę, że model zwrócony przez `firstOrNew` nie został jeszcze utrwalony w bazie danych. Będziesz musiał wywołać `save` ręcznie, aby go zachować:

    // Retrieve flight by name, or create it if it doesn't exist...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Retrieve flight by name, or create it with the name and delayed attributes...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

    // Retrieve by name, or instantiate...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

    // Retrieve by name, or instantiate with the name and delayed attributes...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

#### `updateOrCreate`

Możesz również natknąć się na sytuacje, w których chcesz zaktualizować istniejący model lub utworzyć nowy model, jeśli nie istnieje. Laravel dostarcza metodę `updateOrCreate`, aby wykonać to w jednym kroku. Podobnie jak metoda `firstOrCreate`, `updateOrCreate` utrzymuje model, więc nie ma potrzeby wywoływania `save()`:

    // If there's a flight from Oakland to San Diego, set the price to $99.
    // If no matching model exists, create one.
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );

<a name="deleting-models"></a>
## Deleting Models - Usuwanie modeli

To delete a model, call the `delete` method on a model instance:

    $flight = App\Flight::find(1);

    $flight->delete();

#### Deleting An Existing Model By Key - Usuwanie istniejącego modelu według klucza

W powyższym przykładzie pobieramy model z bazy danych przed wywołaniem metody `delete`. Jednakże, jeśli znasz klucz podstawowy modelu, możesz usunąć model bez pobierania go. Aby to zrobić, wywołaj metodę `destroy`:

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### Deleting Models By Query - Usuwanie modeli przez zapytanie

Oczywiście możesz również uruchomić instrukcję usuwania na zestawie modeli. W tym przykładzie usuniemy wszystkie loty oznaczone jako nieaktywne. Podobnie jak w przypadku aktualizacji masowych, usuwanie masowe nie wywoła żadnych zdarzeń modelu dla modeli, które zostały usunięte:

    $deletedRows = App\Flight::where('active', 0)->delete();

> {note} Podczas wykonywania instrukcji usuwania za pomocą metody Eloquent zdarzenia typu `deleting` i `deleted` nie będą uruchamiane dla usuniętych modeli. Wynika to z faktu, że modele nigdy nie są pobierane podczas wykonywania instrukcji delete.

<a name="soft-deleting"></a>
### Soft Deleting - Miękkie usuwanie

Oprócz faktycznego usuwania rekordów z bazy danych, Eloquent może również "miękko usuwać" modele. Gdy modele są usuwane miękko, nie są one faktycznie usuwane z bazy danych. Zamiast tego atrybut `deleted_at` jest ustawiony na modelu i wstawiony do bazy danych. Jeśli model ma zerową wartość `deleted_at`, model został delikatnie usunięty. Aby włączyć miękkie usuwanie dla modelu, użyj cechy `Illuminate\Database\Eloquent\SoftDeletes` na modelu i dodaj kolumnę` deleted_at` do właściwości `$dates`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;

        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

Oczywiście powinieneś dodać kolumnę `deleted_at` do swojej tabeli bazy danych. Laravel [program do tworzenia schematów (schema builder)](/docs/{{version}}/migrations) zawiera metodę pomocniczą do utworzenia tej kolumny:

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

Teraz, gdy wywołasz metodę `delete` w modelu, kolumna `deleted_at` zostanie ustawiona na bieżącą datę i godzinę. Ponadto podczas wysyłania zapytania do modelu korzystającego z miękkich usunięć, modele z miękkim usunięciem zostaną automatycznie wykluczone ze wszystkich wyników zapytania.

Aby ustalić, czy dana instancja modelu została delikatnie usunięta, użyj metody `trashed`:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### Querying Soft Deleted Models - Zapytanie na miękko usuniętych modelach

#### Including Soft Deleted Models - W tym miękkie Usunięte Modele

Jak wspomniano powyżej, modele z miękkim usuwaniem zostaną automatycznie wykluczone z wyników zapytania. Można jednak wymusić pojawienie się miękkich usuniętych modeli w zestawie wyników za pomocą metody `withTrashed` w zapytaniu:

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

Metodę `withTrashed` można również użyć w zapytaniu [relationship (związek)](/docs/{{version}}/eloquent-relationships):

    $flight->history()->withTrashed()->get();

#### Retrieving Only Soft Deleted Models - Pobieranie tylko miękkich usuniętych modeli

Metoda `onlyTrashed` pobiera **tylko** modele z miękkim usunięciem:

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### Restoring Soft Deleted Models - Przywracanie usuniętych modeli

Czasami możesz chcieć "przywrucić" miękki usunięty model. Aby przywrócić miękki usunięty model do stanu aktywnego, użyj metody `restore` na instancji modelu:

    $flight->restore();

Możesz także użyć metody `restore` w zapytaniu, aby szybko przywrócić wiele modeli. Ponownie, podobnie jak w przypadku innych operacji "masowych", nie uruchomi się żadnych zdarzeń modelu dla przywróconych modeli:

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Podobnie jak w przypadku metody `withTrashed`, metoda` restore` może być również używana w [relacjach](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

#### Permanently Deleting Models -Trwale usuwa modele

Czasami trzeba naprawdę usunąć model z bazy danych. Aby trwale usunąć miękki usunięty model z bazy danych, użyj metody `forceDelete`:

    // Force deleting a single model instance...
    $flight->forceDelete();

    // Force deleting all related models...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## Query Scopes - Zakres zapytań

<a name="global-scopes"></a>
### Global Scopes - Zakres globalny

Globalne zakresy pozwalają dodawać ograniczenia do wszystkich zapytań dla danego modelu. Własna funkcja Laravel [miękkie usuwanie](#soft-deleting) wykorzystuje globalne zakresy, aby pobierać tylko "przywrucone" modele z bazy danych. Pisanie własnych globalnych zakresów może zapewnić wygodny i łatwy sposób na upewnienie się, że każde zapytanie dotyczące danego modelu otrzymuje pewne ograniczenia.

#### Writing Global Scopes - Pisanie globalnych zakresów

Pisanie zakresu globalnego jest proste. Zdefiniuj klasę implementującą interfejs `Illuminate\Database\Eloquent\Scope`. Ten interfejs wymaga zastosowania jednej metody: `apply`. Metoda `apply` może w razie potrzeby dodać ograniczenie `where` do zapytania w razie potrzeby:

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} Jeśli twój globalny zasięg dodaje kolumny do klauzuli select zapytania, powinieneś użyć metody `addSelect` zamiast` select`. Zapobiegnie to niezamierzonemu zastąpieniu istniejącej klauzuli select zapytania.

#### Applying Global Scopes - Stosowanie globalnych zakresów

Aby przypisać globalny zasięg do modelu, należy przesłonić metodę `boot` danego modelu i użyć metody` addGlobalScope`:

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

Po dodaniu zakresu, zapytanie `User::all()` wygeneruje następujący kod SQL:

    select * from `users` where `age` > 200

#### Anonymous Global Scopes - Anonimowe globalne zakresy

Eloquent pozwala również definiować globalne zakresy przy użyciu Closures, co jest szczególnie przydatne w przypadku prostych zakresów, które nie gwarantują oddzielnej klasy:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

#### Removing Global Scopes - Usuwanie globalnych zakresów

Jeśli chcesz usunąć zasięg globalny dla danego zapytania, możesz użyć metody `withoutGlobalScope`. Metoda przyjmuje nazwę klasy globalnego zakresu jako jedyny argument:

    User::withoutGlobalScope(AgeScope::class)->get();

Jeśli chcesz usunąć kilka lub nawet wszystkie globalne zakresy, możesz użyć metody `withoutGlobalScopes`:

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();

    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### Local Scopes - Lokalne zakresy

Lokalne zakresy umożliwiają definiowanie wspólnych zestawów ograniczeń, które można łatwo ponownie wykorzystać w aplikacji. Na przykład może być konieczne częste pobieranie wszystkich użytkowników, którzy są "popular". Aby zdefiniować zakres, po prostu prefiksuj metodę modelu wymownego z `scope`.

Scopes should always return a query builder instance:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### Utilizing A Local Scope - Korzystanie z lokalnego zakresu

Po zdefiniowaniu zakresu można wywoływać metody zasięgu podczas wysyłania zapytania do modelu. Nie powinieneś jednak włączać prefiksu `scope` podczas wywoływania metody. Można nawet łączyć wywołania do różnych zakresów, na przykład:

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### Dynamic Scopes - Dynamiczne zakresy

Czasami możesz chcieć zdefiniować zasięg akceptujący parametry. Aby rozpocząć, wystarczy dodać dodatkowe parametry do swojego zakresu. Parametry zakresu powinny być zdefiniowane po parametrze `$query`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @param mixed $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

Teraz możesz przekazać parametry podczas wywoływania zasięgu:

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## Events - Wydarzenia

Modele elokwentne uruchamiają kilka zdarzeń, umożliwiając przechwycenie następujących punktów cyklu życia modelu: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Zdarzenia umożliwiają łatwe uruchamianie kodu za każdym razem, gdy konkretna klasa modelu jest zapisywana lub aktualizowana w bazie danych.

Zdarzenie `retrieved` zostanie wywołane, gdy istniejący model zostanie pobrany z bazy danych. Gdy nowy model zostanie zapisany po raz pierwszy, zostaną wywołane zdarzenia `creation` i` created`. Jeśli model już istnieje w bazie danych i wywoływana jest metoda `save`, zdarzenia` updating` / `updated` będą uruchamiane. Jednak w obu przypadkach zostaną wywołane zdarzenia `saving` / `saved`.

Aby rozpocząć, zdefiniuj właściwość `$dispatchesEvents` w modelu Eloquent, który mapuje różne punkty cyklu życia modelu do twojej własnej [klasy zdarzeń](/docs/{{version}}/events):

    <?php

    namespace App;

    use App\Events\UserSaved;
    use App\Events\UserDeleted;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

<a name="observers"></a>
### Observers - Obserwatorzy

Jeśli słuchasz wielu zdarzeń na danym modelu, możesz użyć obserwatorów, aby zgrupować wszystkich swoich słuchaczy w jedną klasę. Klasy obserwatorów mają nazwy metod, które odzwierciedlają Elokwentne wydarzenia, których chcesz słuchać. Każda z tych metod otrzymuje model jako jedyny argument. Laravel nie zawiera domyślnego katalogu dla obserwatorów, więc możesz utworzyć dowolny katalog, w którym chcesz umieścić swoje klasy obserwatorów:

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * Listen to the User created event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * Listen to the User deleting event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function deleting(User $user)
        {
            //
        }
    }

Aby zarejestrować obserwatora, użyj metody `observe` na modelu, który chcesz obserwować. Możesz zarejestrować obserwatorów w metodzie `boot` jednego z twoich dostawców usług. W tym przykładzie zarejestrujemy obserwatora w `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use App\User;
    use App\Observers\UserObserver;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            User::observe(UserObserver::class);
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
