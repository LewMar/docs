# Database: Getting Started

- [Introduction - Wprowadzenie](#introduction)
    - [Configuration - Konfiguracja](#configuration)
    - [Read & Write Connections - Czytaj i zapisuj połączenia](#read-and-write-connections)
    - [Using Multiple Database Connections - Korzystanie z wielu połączeń z bazą danych](#using-multiple-database-connections)
- [Running Raw SQL Queries - Uruchamianie surowych zapytań SQL](#running-queries)
    - [Listening For Query Events - Słuchanie zdarzeń zapytań](#listening-for-query-events)
- [Database Transactions - Transakcje bazy danych](#database-transactions)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel ułatwia interakcję z bazami danych w różnych bazach danych przy użyciu albo surowego SQL-a, [płynnego konstruktora zapytań](/docs/{{version}}/queries) i [Eloquent ORM](/docs/{{version}}/eloquent). Obecnie Laravel obsługuje cztery bazy danych:

<div class="content-list" markdown="1">
- MySQL
- PostgreSQL
- SQLite
- SQL Server
</div>

<a name="configuration"></a>
### Configuration - Konfiguracja

Konfiguracja bazy danych dla twojej aplikacji znajduje się w `config/database.php`. W tym pliku możesz zdefiniować wszystkie połączenia z bazą danych, a także określić, które połączenie powinno być używane domyślnie. W tym pliku znajdują się przykłady większości obsługiwanych systemów baz danych.

Domyślnie prosta [konfiguracja środowiska](/docs/{{version}}/configuration#environment-configuration) Laravel jest gotowa do użycia z [Laravel Homestead](/docs/{{version}}/homestead), która jest wygodna maszyna wirtualna do tworzenia aplikacji Laravel na twoim lokalnym komputerze. Oczywiście, możesz dowolnie modyfikować tę konfigurację dla lokalnej bazy danych.

#### SQLite Configuration - Onfiguracja SQLite

Po utworzeniu nowej bazy danych SQLite za pomocą komendy, takiej jak `touch database/database.sqlite`, można łatwo skonfigurować zmienne środowiskowe tak, aby wskazywały na nowo utworzoną bazę danych za pomocą bezwzględnej ścieżki do bazy danych:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

<a name="read-and-write-connections"></a>
### Read & Write Connections - Czytaj i zapisuj połączenia

Czasami możesz chcieć użyć jednego połączenia z bazą danych dla instrukcji SELECT, a innego dla instrukcji INSERT, UPDATE i DELETE. Laravel sprawia, że jest to proste, a odpowiednie połączenia będą zawsze używane, niezależnie od tego, czy używasz zapytań nieprzetworzonych, konstruktora zapytań, czy też Eloquent ORM.

To see how read / write connections should be configured, let's look at this example:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

Zauważ, że do tablicy konfiguracji dodano trzy klucze: `read`, `write` oraz `sticky`. Klucze `read` i `write` mają wartości tablicowe zawierające pojedynczy klucz: `host`. Pozostałe opcje bazy danych dla połączeń `read` i` write` zostaną połączone z głównej tablicy `mysql`.

Musisz tylko umieszczać elementy w tablicach `read` i `write`, jeśli chcesz przesłonić wartości z głównej tablicy. Tak więc w tym przypadku "192.168.1.1" będzie używane jako host dla połączenia "read", a "192.168.1.2" będzie używane dla połączenia "write". Poświadczenia bazy danych, prefiks, zestaw znaków i wszystkie inne opcje w głównej tablicy `mysql` będą udostępniane obu połączeniom.

#### The `sticky` Option - Opcja `sticky`

Opcja `sticky` jest *opcjonalną* wartością, która może być użyta do natychmiastowego odczytu zapisów, które zostały zapisane w bazie danych podczas bieżącego cyklu żądania. Jeśli włączona jest opcja `sticky` i operacja " write" została przeprowadzona względem bazy danych podczas bieżącego cyklu żądania, wszelkie dalsze operacje "read" będą korzystać z połączenia "write". Zapewnia to, że wszelkie dane zapisane podczas cyklu żądania mogą być natychmiast odczytywane z bazy danych podczas tego samego żądania. To ty decydujesz, czy jest to pożądane zachowanie dla twojej aplikacji.

<a name="using-multiple-database-connections"></a>
### Using Multiple Database Connections - Korzystanie z wielu połączeń z bazą danych

Korzystając z wielu połączeń, możesz uzyskać dostęp do każdego połączenia za pomocą metody `connection` na elewacji `DB`. `name` przekazana do metody `connection` powinna odpowiadać jednemu z połączeń wymienionych w pliku konfiguracyjnym `config/database.php`:

    $users = DB::connection('foo')->select(...);

Możesz także uzyskać dostęp do surowej, bazowej instancji PDO za pomocą metody `getPdo` na instancji połączenia:

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>
## Running Raw SQL Queries - Uruchamianie surowych zapytań SQL

Po skonfigurowaniu połączenia z bazą danych można uruchamiać zapytania za pomocą elewacji `DB`. Fasada `DB` zapewnia metody dla każdego typu zapytania:`select`, `update`, `insert`, `delete`, i `statement`.

#### Running A Select Query - Uruchamianie zapytania Select

Aby uruchomić podstawowe zapytanie, możesz użyć metody `select` na elewacji `DB`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

Pierwszym argumentem przekazanym do metody `select` jest surowe zapytanie SQL, natomiast drugim argumentem są wszelkie powiązania parametrów, które muszą być powiązane z zapytaniem. Zazwyczaj są to wartości ograniczeń klauzuli `where`. Powiązanie parametrów zapewnia ochronę przed iniekcją SQL.

Metoda `select` zawsze zwróci `array` wyników. Każdy wynik w tablicy będzie obiektem PHP `StdClass`, umożliwiającym dostęp do wartości wyników:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Using Named Bindings - Używanie nazwanych powiązań

Zamiast używać `?` Do reprezentowania powiązań parametrów, możesz wykonać zapytanie używając nazwanych wiązań:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### Running An Insert Statement - Uruchamianie instrukcji Insert

Aby wykonać instrukcję `insert`, możesz użyć metody `insert` na elewacji `DB`. Podobnie jak `select`, ta metoda pobiera surowe zapytanie SQL jako swój pierwszy argument i powiązania jako swój drugi argument:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### Running An Update Statement - Uruchamianie instrukcji Update

Do aktualizacji istniejących rekordów w bazie danych należy użyć metody `update`. Liczba wierszy, których dotyczy instrukcja, zostanie zwrócona:

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### Running A Delete Statement - Uruchamianie instrukcji Delete

Aby usunąć rekordy z bazy danych, należy użyć metody `delete`. Podobnie jak w `update`, liczba zwracanych wierszy zostanie zwrócona:

    $deleted = DB::delete('delete from users');

#### Running A General Statement - Uruchamianie ogólnych instrukcji

Niektóre instrukcje bazy danych nie zwracają żadnej wartości. W przypadku tego typu operacji można użyć metody `statement` na elewacji` DB`:

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### Listening For Query Events - Słuchanie zdarzeń zapytań

Jeśli chcesz otrzymać każde zapytanie SQL wykonane przez twoją aplikację, możesz użyć metody `listen`. Ta metoda jest przydatna do rejestrowania zapytań lub debugowania. Możesz zarejestrować swój program nasłuchujący zapytania w usłudze [usługodawca](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\DB;
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
            DB::listen(function ($query) {
                // $query->sql
                // $query->bindings
                // $query->time
            });
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

<a name="database-transactions"></a>
## Database Transactions - Transakcje bazy danych

Możesz użyć metody `transaction` na elewacji `DB`, aby uruchomić zestaw operacji w ramach transakcji bazy danych. Jeśli w ramach transakcji `Closure` zostanie zgłoszony wyjątek, transakcja zostanie automatycznie wycofana. Jeśli `closure` zakończy się pomyślnie, transakcja zostanie automatycznie zatwierdzona. Nie musisz martwić się ręcznym wycofywaniem lub zatwierdzaniem podczas używania metody `transaction`:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### Handling Deadlocks - Obsługa zakleszczeń

Metoda `transaction` przyjmuje opcjonalny drugi argument, który określa, ile razy dana transakcja powinna zostać ponownie sprawdzona, gdy wystąpi zakleszczenie. Gdy te próby zostaną wyczerpane, zostanie rzucony wyjątek:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    }, 5);

#### Manually Using Transactions - Ręcznie za pomocą transakcji

Jeśli chcesz ręcznie rozpocząć transakcję i mieć pełną kontrolę nad wycofywaniem i zatwierdzaniem, możesz użyć metody `beginTransaction` na elewacji `DB`:

    DB::beginTransaction();

Możesz wycofać transakcję za pomocą metody `rollBack`:

    DB::rollBack();

Wreszcie, możesz zatwierdzić transakcję za pomocą metody `commit`:

    DB::commit();

> {tip} Metody transakcyjne `DB` kontrolują transakcje zarówno dla [konstruktora zapytań](/docs/{{version}}/queries), jak i [Eloquent ORM](/docs/{{version}}/eloquent).
