# Database: Pagination

- [Introduction - Wprowadzenie](#introduction)
- [Basic Usage - Proste użycie](#basic-usage)
    - [Paginating Query Builder Results - Paginujące wyniki konstruktora kwerend](#paginating-query-builder-results)
    - [Paginating Eloquent Results - Paginowanie wymownych wyników](#paginating-eloquent-results)
    - [Manually Creating A Paginator - Ręczne tworzenie paginatora](#manually-creating-a-paginator)
- [Displaying Pagination Results - Wyświetlanie wyników paginacji](#displaying-pagination-results)
    - [Converting Results To JSON - Konwertowanie wyników na JSON](#converting-results-to-json)
- [Customizing The Pagination View- Dostosowywanie widoku paginacji](#customizing-the-pagination-view)
- [Paginator Instance Methods - Metody instancji Paginatora](#paginator-instance-methods)

<a name="introduction"></a>
## Introduction - Wprowadzenie

W innych framework-ach paginacja może być bardzo bolesna. Paginator Laravel jest zintegrowany z [konstruktor zapytań](/docs/{{version}}/queries), [Eloquent ORM](/docs/{{version}}/eloquent) i zapewnia wygodną, łatwą w użyciu paginację wynikiów bazy danych po wyjęciu z pudełka. HTML wygenerowany przez paginator jest zgodny z [Bootstrap CSS framework](https://getbootstrap.com/).

<a name="basic-usage"></a>
## Basic Usage - Proste użycie

<a name="paginating-query-builder-results"></a>
### Paginating Query Builder Results - Paginujące wyniki konstruktora kwerend

Istnieje kilka sposobów na stronicowanie pozycji. Najprostszym jest użycie metody `paginate` w [konstruktorze zapytań](/docs/{{version}}/queries) lub [Eloquent query](/docs/{{version}}/eloquent). Metoda `paginate` automatycznie zajmuje się ustawianiem odpowiedniego limitu i przesunięcia na podstawie bieżącej strony oglądanej przez użytkownika. Domyślnie bieżąca strona jest wykrywana przez wartość argumentu `page` w zapytaniu HTTP. Oczywiście wartość ta jest automatycznie wykrywana przez Laravel, a także automatycznie wstawiana do linków generowanych przez paginator.

W tym przykładzie jedynym argumentem przekazanym do metody `paginate` jest liczba elementów, które chcesz wyświetlić "na stronie". W tym przypadku określmy, że chcielibyśmy wyświetlić "15" elementów na stronie:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show all of the users for the application.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} Obecnie operacje stronicowania, które używają instrukcji `groupBy`, nie mogą być wydajnie wykonywane przez Laravel. Jeśli musisz użyć `groupBy` z paginowanym zestawem wyników, zalecane jest zapytanie do bazy danych i ręczne utworzenie paginatora.

#### "Simple Pagination" - Proste Paginowanie

Jeśli w widoku paginacji musisz wyświetlać tylko proste linki "Następny" i "Poprzedni", możesz użyć metody `simplePaginate`, aby wykonać bardziej efektywne zapytanie. Jest to bardzo przydatne w przypadku dużych zestawów danych, gdy nie trzeba wyświetlać linku dla każdego numeru strony podczas renderowania widoku:

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### Paginating Eloquent Results - Paginowanie wymownych wyników

Możesz również podać w paginacji zapytania [Eloquent](/docs/{{version}}/eloquent). W tym przykładzie będziemy paginować model `User` z `15` elementami na stronie. Jak widać, składnia jest prawie identyczna z wynikami paginacji generatora zapytań:

    $users = App\User::paginate(15);

Oczywiście możesz nazwać `paginate` po ustawieniu innych ograniczeń w zapytaniu, na przykład w klauzulach `where`:

    $users = User::where('votes', '>', 100)->paginate(15);

Możesz także użyć metody `simplePaginate` podczas dzielenia na strony Wymowne modele:

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### Manually Creating A Paginator - Ręczne tworzenie paginatora

Czasami możesz ręcznie utworzyć instancję stronicowania, przekazując jej tablicę elementów. Możesz to zrobić, tworząc instancję `Illuminate\Pagination\Paginator` lub `Illuminate\Pagination\LengthAwarePaginator`, w zależności od potrzeb.

Klasa `Paginator` nie musi znać całkowitej liczby elementów w zestawie wyników; jednak z tego powodu klasa nie ma metod pobierania indeksu ostatniej strony. `LengthAwarePaginator` akceptuje prawie te same argumenty co `Paginator`; wymaga to jednak zliczenia całkowitej liczby elementów w zestawie wyników.

Innymi słowy, `Paginator` odpowiada metodzie `simplePaginate` na konstruktorze zapytań i Eloquent, natomiast `LengthAwarePaginator` odpowiada metodzie `paginate`.

> {note} Podczas ręcznego tworzenia instancji paginatora należy ręcznie "pokroić" tablicę wyników przekazywanych do paginatora. Jeśli nie masz pewności, jak to zrobić, zapoznaj się z funkcją PHP [array_slice](https://secure.php.net/manual/en/function.array-slice.php).

<a name="displaying-pagination-results"></a>
## Displaying Pagination Results - Wyświetlanie wyników paginacji

Wywołując metodę `paginate`, otrzymasz instancję `Illuminate\Pagination\LengthAwarePaginator`. Wywołując metodę `simplePaginate`, otrzymasz instancję `Illuminate\Pagination\Paginator`. Te obiekty udostępniają kilka metod opisujących zestaw wyników. Oprócz tych metod pomocników instancje paginatora są iteratorami i mogą być zapętlone jako tablica. Po uzyskaniu wyników możesz wyświetlić wyniki i wyświetlić łącza do stron za pomocą [Blade](/docs/{{version}}/blade):

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

Metoda `links` wyrenderuje łącza do pozostałych stron w zestawie wyników. Każdy z tych linków będzie już zawierać poprawną zmienną łańcuchową zapytania `page`. Pamiętaj, że HTML wygenerowany przez metodę `links` jest zgodny z [Bootstrap CSS framework](https://getbootstrap.com).

#### Customizing The Paginator URI - Dostosowywanie identyfikatora URI Paginatora

Metoda `withPath` umożliwia dostosowanie identyfikatora URI używanego przez paginator podczas generowania linków. Na przykład, jeśli chcesz, aby paginator generował łącza, takie jak `http://example.com/custom/url?page=N`, powinieneś przekazać `custom/url` do metody `withPath`:

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

#### Appending To Pagination Links - Dołączanie do łączy paginacji

Możesz dołączyć do łańcucha zapytania linków paginacji za pomocą metody `appends`. Na przykład, aby dołączyć `sort=votes` do każdego linku do stronicowania, należy wykonać następujące wywołanie do `appends`:

    {{ $users->appends(['sort' => 'votes'])->links() }}

Jeśli chcesz dodać "fragment skrótu" do adresów URL paginatora, możesz użyć metody `fragment`. Na przykład, aby dołączyć `#foo` na końcu każdego linku do paginacji, wykonaj następujące wywołanie metody `fragment`:

    {{ $users->fragment('foo')->links() }}

<a name="converting-results-to-json"></a>
### Converting Results To JSON - Konwertowanie wyników na JSON

Klasy wyników Laravel paginator implementują umowę `Illuminate\Contracts\Support\Jsonable` i eksponują metodę `toJson`, więc bardzo łatwo jest przekonwertować wyniki paginacji na JSON. Możesz również przekonwertować instancję paginatora na JSON, zwracając ją z trasy lub akcji kontrolera:

    Route::get('users', function () {
        return App\User::paginate();
    });

JSON z paginatora będzie zawierał metadane, takie jak `total`, `current_page`, `last_page` i więcej. Rzeczywiste obiekty wynikowe będą dostępne za pośrednictwem klucza `data` w tablicy JSON. Oto przykład JSON utworzonego przez zwrócenie instancji paginatora z trasy:

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## Customizing The Pagination View - Dostosowywanie widoku paginacji

Domyślnie widoki renderowane w celu wyświetlenia linków do stronicowania są zgodne ze strukturą Bootstrap CSS. Jeśli jednak nie używasz Bootstrap, możesz zdefiniować własne widoki, aby renderować te linki. Wywołując metodę `links` na instancji paginatora, przekaż nazwę widoku jako pierwszy argument metody:

    {{ $paginator->links('view.name') }}

    // Passing data to the view...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

Najłatwiejszym sposobem dostosowania widoków paginacji jest wyeksportowanie ich do katalogu `resources/views/vendor` przy użyciu polecenia `vendor:publish`:

    php artisan vendor:publish --tag=laravel-pagination

To polecenie umieści widoki w katalogu `resources/views/vendor/pagination`. Plik `default.blade.php` w tym katalogu odpowiada domyślnemu widokowi paginacji. Edytuj ten plik, aby zmodyfikować HTML strony.

<a name="paginator-instance-methods"></a>
## Paginator Instance Methods - Metody instancji Paginatora

Każda instancja paginatora udostępnia dodatkowe informacje na temat stronicowania za pomocą następujących metod:

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (Niedostępny podczas korzystania simplePaginate)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (Not available when using simplePaginate)`
- `$results->url($page)`
