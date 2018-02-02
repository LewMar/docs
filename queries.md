# Database: Query Builder

- [Introduction - Wprowadzenie](#introduction)
- [Retrieving Results - Pobieranie wyników](#retrieving-results)
    - [Chunking Results - Kawałek wyników](#chunking-results)
    - [Aggregates - Agregaty](#aggregates)
- [Selects - Selekty](#selects)
- [Raw Expressions - Surowe wyrażenia](#raw-expressions)
- [Joins - Złaczenia](#joins)
- [Unions - Unie](#unions)
- [Where Clauses - Klauzury Where](#where-clauses)
    - [Parameter Grouping - Grupowanie parametrów](#parameter-grouping)
    - [Where Exists Clauses - Klauzura Where Exist](#where-exists-clauses)
    - [JSON Where Clauses - Klauzura gdzie JSON](#json-where-clauses)
- [Ordering, Grouping, Limit, & Offset - Zamawianie, grupowanie, ograniczenie i przesunięcie](#ordering-grouping-limit-and-offset)
- [Conditional Clauses - Klauzule warunkowe](#conditional-clauses)
- [Inserts - Wstawianie](#inserts)
- [Updates - Aktualizacje](#updates)
    - [Updating JSON Columns - Aktualizacja kolumn JSON](#updating-json-columns)
    - [Increment & Decrement - Inkrementacja i Dekrementacja](#increment-and-decrement)
- [Deletes - Usunięcia](#deletes)
- [Pessimistic Locking - Pesymistyczne blokowanie](#pessimistic-locking)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Konstruktor kwerend bazodanowych Laravel zapewnia wygodny, płynny interfejs do tworzenia i uruchamiania zapytań do bazy danych. Może być używany do wykonywania większości operacji na bazach danych w aplikacji i działa we wszystkich obsługiwanych systemach baz danych.

Konstruktor kwerend Laravel używa wiązania parametru PDO do ochrony aplikacji przed atakami typu SQL injection. Nie ma potrzeby czyszczenia łańcuchów przekazywanych jako wiązania.

<a name="retrieving-results"></a>
## Retrieving Results - Pobieranie wyników

#### Retrieving All Rows From A Table - Pobieranie wszystkich wierszy z tabeli

Możesz użyć metody `table` na elewacji `DB`, aby rozpocząć zapytanie. Metoda `table` zwraca płynną instancję konstruktora kwerend dla danej tabeli, umożliwiając powiązanie większej liczby ograniczeń z zapytaniem, a następnie uzyskanie wyników za pomocą metody `get`:

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
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

Metoda `get` zwraca `Illuminate\Support\Collection` zawierające wyniki, gdzie każdy wynik jest instancją obiektu PHP `StdClass`. Możesz uzyskać dostęp do wartości każdej kolumny, uzyskując dostęp do kolumny jako właściwości obiektu:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Retrieving A Single Row / Column From A Table - Pobieranie pojedynczego wiersza / kolumny z tabeli

Jeśli potrzebujesz tylko pobrać jeden wiersz z tabeli bazy danych, możesz użyć metody `first`. Ta metoda zwróci pojedynczy obiekt `StdClass`:

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

Jeśli nawet nie potrzebujesz całego wiersza, możesz wyodrębnić pojedynczą wartość z rekordu za pomocą metody `value`. Ta metoda zwróci bezpośrednio wartość kolumny:

    $email = DB::table('users')->where('name', 'John')->value('email');

#### Retrieving A List Of Column Values - Pobieranie listy wartości kolumn

Jeśli chcesz pobrać kolekcję zawierającą wartości jednej kolumny, możesz użyć metody "pluck". W tym przykładzie otrzymamy kolekcję tytułów ról:

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

 Możesz także określić kolumnę klucza niestandardowego dla zwróconej kolekcji:

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### Chunking Results - Kawałek wyników

Jeśli chcesz pracować z tysiącami rekordów bazy danych, rozważ użycie metody `chunk`. Ta metoda pobiera niewielki fragment wyników naraz i podaje każdą porcję do `Closure` w celu przetworzenia. Ta metoda jest bardzo przydatna przy pisaniu [poleceń Artisana](/docs/{{version}}/artisan), które przetwarzają tysiące rekordów. Na przykład pracujmy z całą tabelą `users` w porcjach po 100 rekordów naraz:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

Możesz przerwać przetwarzanie kolejnych fragmentów, zwracając `false` z `Closure`:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

<a name="aggregates"></a>
### Aggregates - Agregaty

Konstruktor kwerend udostępnia także różne metody agregacji, takie jak `count`, `max`, `min`, `avg`, i `sum`. Możesz wywołać dowolną z tych metod po skonstruowaniu zapytania:

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Oczywiście możesz łączyć te metody z innymi klauzulami:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects - Selekty

#### Specifying A Select Clause - Określanie klauzuli Select

Oczywiście nie zawsze chcesz wybrać wszystkie kolumny z tabeli bazy danych. Używając metody `select`, możesz określić niestandardową klauzulę `select` dla zapytania:

    $users = DB::table('users')->select('name', 'email as user_email')->get();

Metoda `distinct` pozwala wymusić na zapytaniu zwracanie wyraźnych wyników:

    $users = DB::table('users')->distinct()->get();

Jeśli masz już instancję budującą zapytania i chcesz dodać kolumnę do istniejącej klauzuli select, możesz użyć metody `addSelect`:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## Raw Expressions - Surowe wyrażenia

Czasami może zajść potrzeba użycia surowego wyrażenia w zapytaniu. Aby utworzyć wyrażenie raw, możesz użyć metody `DB::raw`:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

> {note} Surowe instrukcje będą wstrzykiwane do zapytania jako łańcuchy, więc powinieneś być bardzo ostrożny, aby nie tworzyć luk SQL injection.

<a name="raw-methods"></a>
### Raw Methods - Surowe metody

Instead of using `DB::raw`, you may also use the following methods to insert a raw expression into various parts of your query.
Zamiast używać `DB::raw`, możesz również użyć następujących metod, aby wstawić surowe wyrażenie do różnych części zapytania.

#### `selectRaw`

Metodę `selectRaw` można użyć zamiast `select(DB::raw(...))`. Ta metoda przyjmuje opcjonalną tablicę powiązań jako swój drugi argument:

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();

#### `whereRaw / orWhereRaw`

Metody `whereRaw` i` orWhereRaw` mogą być użyte do wprowadzenia surowej klauzuli `where` do zapytania. Te metody przyjmują opcjonalną tablicę powiązań jako swój drugi argument:

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

#### `havingRaw / orHavingRaw`

Metody `havingRaw` i `orHavingRaw` mogą być użyte do ustawienia nieprzetworzonego łańcucha jako wartości klauzuli `having`:

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### `orderByRaw`

Metoda `orderByRaw` może służyć do ustawienia nieprzetworzonego łańcucha jako wartości klauzuli `order by`:

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();

<a name="joins"></a>
## Joins - Złaczenia

#### Inner Join Clause - Klauzula Inner Join

Kreator zapytań może być również używany do pisania instrukcji łączenia. Aby wykonać podstawowe "sprzężenie wewnętrzne", możesz użyć metody `join` na instancji konstruktora zapytań. Pierwszy argument przekazany do metody `join` jest nazwą tabeli, do której należy się przyłączyć, podczas gdy pozostałe argumenty określają ograniczenia kolumn dla łączenia. Oczywiście, jak widać, możesz dołączyć do wielu tabel w jednym zapytaniu:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join Clause - Klauzula Left Join

Jeśli chcesz wykonać "lewe połączenie" zamiast "wewnętrznego łączenia", użyj metody `leftJoin`. Metoda `leftJoin` ma taki sam podpis jak metoda `join`:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cross Join Clause - Klauzula Cross Join

Aby wykonać "sprzężenie krzyżowe", użyj metody `crossJoin` z nazwą tabeli, do której chcesz się połączyć. Połączenia krzyżowe generują kartezjański produkt między pierwszą tabelą a połączoną tabelą:

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### Advanced Join Clauses - Klauzula zaawansowana

Możesz także określić bardziej zaawansowane klauzule dołączania. Aby rozpocząć, podaj `Closure` jako drugi argument w metodzie`join`. `Closure` otrzyma obiekt `JoinClause`, który pozwala na określenie ograniczeń w klauzuli `join`:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

Jeśli chcesz użyć klauzuli "where" w swoich połączeniach, możesz użyć metod `where` i `orWhere` na sprzężeniu. Zamiast porównywania dwóch kolumn, metody te będą porównywać kolumnę z wartością:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions - Unie

Konstruktor kwerend zapewnia również szybki sposób łączenia dwóch zapytań razem. Na przykład możesz utworzyć zapytanie wstępne i użyć metody `union`, aby połączyć ją z drugim zapytaniem:

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} Dostępna jest także metoda `unionAll`, która ma ten sam podpis metody co `union`.

<a name="where-clauses"></a>
## Where Clauses - Klauzury Where

#### Simple Where Clauses - Proste klauzury Where

Możesz użyć metody `where` na instancji konstruktora zapytań, aby dodać klauzule `where` do zapytania. Najbardziej podstawowe wywołanie `where` wymaga trzech argumentów. Pierwszym argumentem jest nazwa kolumny. Drugi argument to operator, który może być dowolnym operatorem obsługiwanym przez bazę danych. Wreszcie, trzecim argumentem jest wartość do oceny w stosunku do kolumny.

For example, here is a query that verifies the value of the "votes" column is equal to 100:
Na przykład tutaj jest zapytanie, które weryfikuje wartość kolumny "głosy" równe 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

Dla wygody, jeśli chcesz sprawdzić, czy kolumna jest równa danej wartości, możesz przekazać wartość bezpośrednio jako drugi argument do metody `where`:

    $users = DB::table('users')->where('votes', 100)->get();

Oczywiście podczas pisania klauzuli `where` możesz używać wielu innych operatorów:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

Możesz także przekazać tablicę warunków do funkcji `where`:

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Or Statements - Deklaracja OR

Możesz łączyć ze sobą wiązania, a także dodawać klauzule `or` do zapytania. Metoda `orWhere` akceptuje te same argumenty, co metoda `where`:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### Additional Where Clauses - Dodakkowe klauzury Where

**whereBetween** - gdzie pomiędzy

Metoda `whereBetween` sprawdza, czy wartość kolumny mieści się między dwiema wartościami:

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween** - nie gdzie pomiędzy

Metoda `whereNotBetween` sprawdza, czy wartość kolumny znajduje się poza dwiema wartościami:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn** - gdzie w i gdzie nie w

Metoda `whereIn` sprawdza, czy dana kolumna zawiera się w podanej tablicy:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

Metoda `whereNotIn` sprawdza, czy dana kolumna ma wartość **nie** zawartą w podanej tablicy:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull** - gdzie puste i gdzie nie puste

Metoda `whereNull` sprawdza, czy wartość danej kolumny to `NULL`:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

Metoda `whereNotNull` sprawdza, czy wartość kolumny nie ma wartości `NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear / whereTime** - gdzie data, miesiąc, dzień, rok, czas

Metoda `whereDate` może być używana do porównywania wartości kolumny z datą:

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

Metodę `whereMonth` można użyć do porównania wartości kolumny z konkretnym miesiącem roku:

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

Metodę `whereDay` można użyć do porównania wartości kolumny z określonym dniem miesiąca:

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

Metodę `whereYear` można użyć do porównania wartości kolumny z konkretnym rokiem:

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

Metodę `whereTime` można użyć do porównania wartości kolumny z określonym czasem:

    $users = DB::table('users')
                    ->whereTime('created_at', '=', '11:20')
                    ->get();

**whereColumn** - czy dwie kolumny sa równe

Do sprawdzenia, czy dwie kolumny są równe, można użyć metody `whereColumn`:

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

Możesz także przekazać operatorowi porównania metodę:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

Metodę `whereColumn` można również przekazać tablicę wielu warunków. Warunki te zostaną połączone za pomocą operatora `and`:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### Parameter Grouping - Grupowanie parametrów

Czasami może być konieczne utworzenie bardziej zaawansowanych klauzul, w których występują klauzule "gdzie istnieje" lub zagnieżdżone grupowanie parametrów. Konstruktor kwerend Laravel może również obsługiwać te kwerendy. Aby rozpocząć, spójrzmy na przykład grupowania wiązań w nawiasach:

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

Jak widać, przekazanie `Closure` do metody `orWhere` instruuje konstruktora kwerend, aby rozpoczął grupę ograniczeń. `Closure` otrzyma instancję konstruktora zapytań, której można użyć do ustawienia wiązań, które powinny być zawarte w grupie nawiasów. Powyższy przykład wygeneruje następujący kod SQL:

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Where Exists Clauses - Klauzura Where Exist

Metoda `whereExists` pozwala na pisanie klauzul SQL `tam, gdzie istnieje`. Metoda `whereExists` akceptuje argument `Closure`, który otrzyma instancję konstruktora zapytań pozwalającą zdefiniować zapytanie, które powinno zostać umieszczone w klauzuli "istnieje":

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

Powyższe zapytanie wygeneruje następujący kod SQL:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON Where Clauses - Klauzura gdzie JSON

Laravel obsługuje również wyszukiwanie typów kolumn JSON w bazach danych, które obsługują typy kolumn JSON. Obecnie obejmuje to MySQL 5.7 i PostgreSQL. Aby wysłać zapytanie do kolumny JSON, użyj operatora `->`:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit, & Offset - Zamawianie, grupowanie, ograniczenie i przesunięcie

#### orderBy - grupuj po

Metoda `orderBy` umożliwia sortowanie wyniku zapytania przez daną kolumnę. Pierwszym argumentem metody `orderBy` powinna być kolumna, którą chcesz sortować, podczas gdy drugi argument kontroluje kierunek sortowania i może to być` asc` lub `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### latest / oldest - najnowsze / najstarsze

Metody `latest` and `oldest` pozwalają łatwo zamawiać wyniki według daty. Domyślnie wynik zostanie uporządkowany według kolumny `created_at`. Możesz też podać nazwę kolumny, którą chcesz posortować według:

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder - losowe sortowanie

Metoda `inRandomOrder` może być używana do losowego sortowania wyników zapytania. Na przykład możesz użyć tej metody, aby pobrać losowego użytkownika:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having - grupuj po / mających

Do grupowania wyników zapytania można użyć metod `groupBy` i `having`. Sygnatura metody "posiadająca" jest podobna do metody `where`:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Możesz przekazać wiele argumentów do metody `groupBy` w celu grupowania według wielu kolumn:

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

Dla bardziej zaawansowanych instrukcji `having` zobacz metodę [`havingRaw`](#raw-methods).

#### skip / take - pomiń / zabierz

Aby ograniczyć liczbę wyników zwracanych przez zapytanie lub pominąć podaną liczbę wyników w zapytaniu, możesz użyć metod `skip` and `take`:

    $users = DB::table('users')->skip(10)->take(5)->get();

Alternatywnie możesz użyć metod `limit` i `offset`:

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## Conditional Clauses - Klauzule warunkowe

Czasami możesz chcieć zastosować klauzule do zapytania tylko wtedy, gdy coś innego jest prawdą. Na przykład możesz chcieć zastosować tylko instrukcję `where`, jeśli dana wartość wejściowa jest obecna w żądaniu przychodzącym. Możesz to zrobić za pomocą metody `when`:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();

Metoda `when` wykonuje tylko daną Closure, gdy pierwszym parametrem jest `true`. Jeśli pierwszym parametrem jest `false`, Closure nie zostanie wykonane.

Możesz przekazać kolejne zamknięcie jako trzeci parametr metody  `true`. To Closure zostanie wykonane, jeśli pierwszy parametr zostanie oceniony jako `false`. Aby zilustrować, w jaki sposób można użyć tej funkcji, użyjemy jej do skonfigurowania domyślnego sortowania zapytania:

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();

<a name="inserts"></a>
## Inserts - Wstawianie

Konstruktor kwerend udostępnia również metodę `insert` do wstawiania rekordów do tabeli bazy danych. Metoda `insert` przyjmuje tablicę nazw kolumn i wartości:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

Możesz nawet wstawić kilka rekordów do tabeli za pomocą jednego wywołania `insert`, przekazując tablicę tablic. Każda tablica reprezentuje wiersz do wstawienia do tabeli:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### Auto-Incrementing IDs - Autoinkrementacja identyfikatora

Jeśli tabela ma automatycznie zwiększany identyfikator, użyj metody `insertGetId`, aby wstawić rekord, a następnie pobierz identyfikator:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} Podczas korzystania z PostgreSQL, metoda `insertGetId` oczekuje, że kolumna auto-inkrementująca ma mieć nazwę `id`. Jeśli chcesz pobrać identyfikator z innej "sekwencji", możesz przekazać nazwę kolumny jako drugi parametr do metody `insertGetId`.

<a name="updates"></a>
## Updates - Aktualizacje

Oczywiście, oprócz wstawiania rekordów do bazy danych, konstruktor kwerend może również aktualizować istniejące rekordy za pomocą metody `update`. Metoda `update`, podobnie jak metoda `insert`, akceptuje tablicę kolumn i par wartości zawierających kolumny do aktualizacji. Możesz ograniczyć zapytanie `update` używając klauzul `where`:

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### Updating JSON Columns - Aktualizacja kolumn JSON

Podczas aktualizowania kolumny JSON należy użyć składni `->`, aby uzyskać dostęp do odpowiedniego klucza w obiekcie JSON. Ta operacja jest obsługiwana tylko w bazach danych obsługujących kolumny JSON:

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### Increment & Decrement - Inkrementacja i Dekrementacja

Kreator zapytań zapewnia również wygodne metody zwiększania lub zmniejszania wartości danej kolumny. Jest to skrót, zapewniający bardziej wyrazisty i zwięzły interfejs w porównaniu do ręcznego pisania instrukcji `update`.

Obie te metody akceptują co najmniej jeden argument: kolumnę do modyfikacji. Drugi argument może być opcjonalnie przekazany w celu kontrolowania ilości, o jaką kolumna powinna być zwiększana lub zmniejszana:

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

Możesz także określić dodatkowe kolumny do aktualizacji podczas operacji:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes - Usunięcia

Konstruktora zapytań można również użyć do usunięcia rekordów z tabeli za pomocą metody `delete`. Możesz ograniczyć instrukcje `delete`, dodając klauzule` where` przed wywołaniem metody `delete`:

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

Jeśli chcesz skrócić całą tabelę, co spowoduje usunięcie wszystkich wierszy i zresetowanie automatycznie zwiększającego ID do zera, możesz użyć metody `truncate`:

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## Pessimistic Locking - Pesymistyczne blokowanie

Kreator zapytań zawiera również kilka funkcji ułatwiających "pesymistyczne blokowanie" w instrukcjach `select`. Aby uruchomić instrukcję z "udostępnionym zamkiem", możesz użyć metody `sharedLock` w zapytaniu. Wspólna blokada zapobiega modyfikowaniu wybranych wierszy do momentu zatwierdzenia transakcji:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Alternatywnie możesz użyć metody `lockForUpdate`. Blokada "dla aktualizacji" uniemożliwia modyfikację wierszy lub ich wybór przy użyciu innej blokady współdzielonej:


    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
