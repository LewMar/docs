# Routing

- [Basic Routing - Podstawowy routing](#basic-routing)
    - [Redirect Routes - Przekierowania tras](#redirect-routes)
    - [View Routes - Widok trasy](#view-routes)
- [Route Parameters - Parametry trasy](#route-parameters)
    - [Required Parameters - Wymagane parametry](#required-parameters)
    - [Optional Parameters - Opcjonalne parametry](#parameters-optional-parameters)
    - [Regular Expression Constraints - Ograniczanie wyrażeniam regularnymi](#parameters-regular-expression-constraints)
- [Named Routes - Nazwane trasy](#named-routes)
- [Route Groups - Grupy tras](#route-groups)
    - [Middleware - Oprogramowanie pośrednie](#route-group-middleware)
    - [Namespaces - Przestrzenie nazw](#route-group-namespaces)
    - [Sub-Domain Routing - Ruting poddomeny](#route-group-sub-domain-routing)
    - [Route Prefixes - Prefiksy tras](#route-group-prefixes)
    - [Route Name Prefixes - Prefiksy nazw tras](#route-group-name-prefixes)
- [Route Model Binding - Wiązanie modelu trasy](#route-model-binding)
    - [Implicit Binding - Niejawne powiązania](#implicit-binding)
    - [Explicit Binding - Jawne powiązania](#explicit-binding)
- [Rate Limiting - Ograniczenie tempa](#rate-limiting)
- [Form Method Spoofing - Podszywanie się pod Metody Form-a](#form-method-spoofing)
- [Accessing The Current Route - Dostęp do aktualnej trasy](#accessing-the-current-route)

<a name="basic-routing"></a>
## Basic Routing - Podstawowy routing

Najbardziej podstawowe trasy Laravel akceptują URI i `Closure`, zapewniając bardzo prostą i ekspresywną metodę definiowania tras:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### The Default Route Files - Domyślne pliki trasy

Wszystkie trasy Laravel są zdefiniowane w twoich plikach tras, które znajdują się w katalogu `routes`. Te pliki są automatycznie ładowane przez framework. Plik `routes/web.php` określa trasy dla twojego interfejsu sieciowego. Te trasy mają przypisaną grupę oprogramowania `web`, która zapewnia funkcje takie jak stan sesji i ochrona CSRF. Trasy w `routes/api.php` są bezstanowe i mają przypisaną grupę oprogramowania pośredniego `api`.

W przypadku większości aplikacji zaczniesz od zdefiniowania tras w pliku `routes/web.php`. Trasy zdefiniowane w `routes/web.php` mogą być dostępne po wprowadzeniu adresu URL zdefiniowanej trasy w przeglądarce. Na przykład możesz uzyskać dostęp do poniższej trasy, przechodząc do `http://twoja-aplikacja.dev/user` w swojej przeglądarce:

    Route::get('/user', 'UserController@index');

Trasy zdefiniowane w pliku `routes/api.php` są zagnieżdżone w grupie tras przez `RouteServiceProvider`. W tej grupie prefiks URI `/api` jest automatycznie stosowany, więc nie trzeba ręcznie stosować go do każdej trasy w pliku. Możesz zmodyfikować prefiks i inne opcje grupy tras, modyfikując klasę `RouteServiceProvider`.

#### Available Router Methods - Dostępne metody routera

Router umożliwia rejestrowanie tras odpowiadających na dowolny czyności HTTP:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Czasami może zajść potrzeba zarejestrowania trasy, która odpowiada na wiele czynności HTTP. Możesz to zrobić za pomocą metody `any`. Lub możesz nawet zarejestrować trasę, która odpowiada na wszystkie czasowniki HTTP za pomocą metody `any`:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF Protection - Ochrona CSRF

Wszelkie formularze HTML wskazujące na trasy `POST`, `PUT`, lub `DELETE` zdefiniowane w pliku tras `web` powinny zawierać pole tokenu CSRF. W przeciwnym razie prośba zostanie odrzucona. Więcej informacji na temat ochrony CSRF można znaleźć w [dokumentacji CSRF](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### Redirect Routes - Przekierowania tras

Jeśli definiujesz trasę, która przekierowuje do innego URI, możesz użyć metody `Route::redirect`. Ta metoda zapewnia wygodny skrót, dzięki czemu nie trzeba definiować pełnej trasy ani kontrolera do wykonywania prostego przekierowania:

    Route::redirect('/to', '/tam', 301);

<a name="view-routes"></a>
### View Routes - Widok trasy

Jeśli twoja trasa musi tylko zwrócić widok, możesz użyć metody `Route::view`. Podobnie jak w przypadku metody `redirect`, ta metoda zapewnia prosty skrót, dzięki czemu nie trzeba definiować pełnej trasy ani kontrolera. Metoda `view` akceptuje URI jako swój pierwszy argument, a nazwę widoku jako drugi argument. Ponadto możesz podać tablicę danych do przekazania do widoku jako opcjonalnego trzeciego argumentu:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## Route Parameters - Parametry trasy

<a name="required-parameters"></a>
### Required Parameters - Wymagane parametry

Oczywiście czasami będziesz musiał przechwycić segmenty URI na swojej trasie. Na przykład może być konieczne przechwycenie identyfikatora (ID) użytkownika z adresu URL. Możesz to zrobić, definiując parametry trasy:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

Możesz zdefiniować tyle parametrów trasy, ile wymaga twoja trasa:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Parametry trasy są zawsze zamknięte w nawiasach klamrowych `{}` i powinny składać się ze znaków alfabetu i niemogą zawierać znaku `-`. Zamiast używać znaku `-`, użyj znaku podkreślenia (`_`). Parametry trasy są wprowadzane do wywołań zwrotnych / kontrolerów trasy w zależności od ich kolejności - nazwy argumentów wywołania zwrotnego / kontrolera nie mają znaczenia.

<a name="parameters-optional-parameters"></a>
### Optional Parameters - Opcjonalne parametry

Czasami konieczne może być określenie parametru trasy, ale obcionalnie dla tej trasy. Możesz to zrobić umieszczając znak `?` Po nazwie parametru. Upewnij się, że przypisana zmienna trasy jest wartością domyślną:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Regular Expression Constraints - Ograniczanie wyrażeniam regularnymi

Możesz ograniczyć format parametrów trasy, używając metody `where` na instancji trasy. Metoda `where` akceptuje nazwę parametru i wyrażenie regularne definiujące sposób ograniczenia parametru:

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### Global Constraints - Ograniczanie globalnie

Jeśli chcesz, aby parametr trasy był zawsze ograniczony przez dane wyrażenie regularne, możesz użyć metody `pattern`. Powinieneś zdefiniować te wzorce w metodzie `boot` twojego` RouteServiceProvider`:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

Po zdefiniowaniu wzoru jest on automatycznie stosowany do wszystkich tras z użyciem tej nazwy parametru:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="named-routes"></a>
## Named Routes - Nazwane trasy

Nazwane trasy umożliwiają wygodne generowanie adresów URL lub przekierowań dla określonych tras. Możesz określić nazwę trasy, łącząc metodę `name` z definicją trasy:

    Route::get('user/profile', function () {
        //
    })->name('profile');

Możesz również określić nazwy tras dla działań kontrolera:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### Generating URLs To Named Routes - Generowanie adresów URL dla nazwanych tras

Po przypisaniu nazwy do danej trasy możesz użyć nazwy trasy podczas generowania adresów URL lub przekierowań za pośrednictwem globalnej funkcji `route`:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

Jeśli nazwana trasa definiuje parametry, możesz przekazać parametry jako drugi argument funkcji `route`. Podane parametry zostaną automatycznie wstawione do adresu URL na ich właściwych pozycjach:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### Inspecting The Current Route - Sprawdzanie aktualnej trasy

Jeśli chcesz sprawdzić, czy bieżące żądanie zostało skierowane do podanej trasy, możesz użyć metody `named` na instancji Route. Na przykład możesz sprawdzić nazwę aktualnej trasy z oprogramowania pośredniego trasy:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## Route Groups - Grupy tras

Grupy tras umożliwiają udostępnianie atrybutów trasy, takich jak oprogramowanie pośrednie lub przestrzenie nazw, na wielu trasach bez konieczności definiowania tych atrybutów na każdej pojedynczej trasie. Wspólne atrybuty są określane w formacie tablicowym jako pierwszy parametr metody `Route::group`.

<a name="route-group-middleware"></a>
### Middleware - Oprogramowanie pośrednie

Aby przypisać oprogramowanie pośrednie do wszystkich tras w grupie, przed zdefiniowaniem grupy można użyć metody `middleware`. Oprogramowanie pośrednie są wykonywane w kolejności, w jakiej są wymienione w tablicy:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });

        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Namespaces - Przestrzenie nazw

Innym częstym przypadkiem użycia dla grup tras jest przypisanie tej samej przestrzeni nazw PHP do grupy kontrolerów używających metody `namespace`:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Pamiętaj, domyślnie, `RouteServiceProvider` zawiera pliki tras w grupie przestrzeni nazw, co pozwala ci rejestrować trasy kontrolerów bez określania pełnego przedrostka przestrzeni nazw `App\Http\Controllers`. Musisz tylko określić część przestrzeni nazw, która pochodzi od podstawowej przestrzeni nazw `App\Http\Controllers`.

<a name="route-group-sub-domain-routing"></a>
### Sub-Domain Routing - Ruting poddomeny

Grupy tras mogą być również używane do obsługi routingu poddomeny. Subdomeny mogą mieć przypisane parametry trasy, podobnie jak identyfikatory URI trasy, pozwalające na przechwycenie części subdomen do wykorzystania na trasie lub kontrolerze. Poddomena może być określona przez wywołanie metody `domain` przed zdefiniowaniem grupy:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Route Prefixes - Prefiksy tras

Metodę `prefix` można użyć do prefiksowania każdej trasy w grupie z danym URI. Na przykład możesz dodać przedrostek wszystkich identyfikatorów URI trasy w grupie za pomocą `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Route Name Prefixes - Prefiksy nazw tras

Metodę `name` można użyć do prefiksowania każdej nazwy trasy w grupie podanym łańcuchem. Na przykład możesz przedefiniować wszystkie nazwy zgrupowanych tras za pomocą `admin`. Podany ciąg jest poprzedzany nazwą trasy dokładnie tak, jak jest określona, więc na pewno będziemy dostarczać końcowy znak `.` w przedrostku:

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        });
    });

<a name="route-model-binding"></a>
## Route Model Binding - Wiązanie modelu trasy

Po wprowadzeniu ID modelu do trasy lub działania kontrolera, często będziesz wyszukiwał model, który odpowiada temu ID. Wiązanie modelu trasy Laravel zapewnia wygodny sposób automatycznego wstrzykiwania instancji modelu bezpośrednio do twoich tras. Na przykład, zamiast wstrzykiwać identyfikator użytkownika, można wstrzyknąć całą instancję modelu `User`, która pasuje do podanego ID.

<a name="implicit-binding"></a>
### Implicit Binding - Niejawne powiązania

Laravel automatycznie rozpoznaje Wymowne modele zdefiniowane w trasach lub akcjach kontrolerów, których nazwy zmiennych wskazane przez typ pasują do nazwy segmentu trasy. Na przykład:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Ponieważ zmienna `$user` ma typ wskazujący na Wymowny model `App\User` i nazwa zmiennej pasuje do segmentu URI `{user}`, Laravel automatycznie wstrzykuje instancję modelu, która ma ID pasujące do odpowiadającej wartości z identyfikatora URI danego żądania. Jeśli odpowiednia instancja modelu nie zostanie znaleziona w bazie danych, automatycznie wygenerowana zostanie odpowiedź HTTP 404.

#### Customizing The Key Name - Dostosowywanie nazwy klucza

Jeśli chcesz, aby powiązanie modelu korzystało z kolumny bazy danych innej niż `id` podczas pobierania danej klasy modelu, możesz zastąpić metodę `getRouteKeyName` w modelu Eloquent:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Explicit Binding - Jawne powiązania

Aby zarejestrować jawne powiązanie, użyj metody `model` routera, aby określić klasę dla danego parametru. Powinieneś zdefiniować swoje jawne powiązania modelu w metodzie `boot` klasy` RouteServiceProvider`:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Następnie zdefiniuj trasę zawierającą parametr `{user}`:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Ponieważ przywiązaliśmy wszystkie parametry `{user}` do modelu `App\User`, do trasy zostanie wprowadzona instancja `User`. Tak więc, na przykład, żądanie do `profile/1` wstrzyknie instancję `User` z bazy danych o identyfikatorze `1`.

Jeśli odpowiednia instancja modelu nie zostanie znaleziona w bazie danych, automatycznie wygenerowana zostanie odpowiedź HTTP 404.

#### Customizing The Resolution Logic - Dostosowywanie logiki decyzyjnej

Jeśli chcesz użyć własnej logiki decyzyjnej, możesz użyć metody `Route::bind`. `Closure`, które przekazujesz do metody `bind`, otrzyma wartość segmentu URI i powinno zwrócić instancję klasy, która ma zostać wstrzyknięta do trasy:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="rate-limiting"></a>
## Rate Limiting - Ograniczenie tempa

Laravel zawiera [oprogramowanie pośrednie](/docs/{{version}}/middleware), aby ocenić limit dostępu do tras w aplikacji. Aby rozpocząć, przypisz oprogramowanie pośredniczące `throttle` do trasy lub grupy tras. Program pośredniczący `throttle` akceptuje dwa parametry określające maksymalną liczbę żądań, które można wykonać w danej liczbie minut. Na przykład, określmy, że uwierzytelniony użytkownik może uzyskać dostęp do następującej grupy tras 60 razy na minutę:

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Dynamic Rate Limiting - Dynamiczne ograniczenie tempa

Możesz określić maksymalną wartość żądania dynamicznego na podstawie atrybutu uwierzytelnionego modelu `User`. Na przykład, jeśli twój model `User` zawiera atrybut `rate_limit`, możesz przekazać nazwę tego atrybutu do oprogramowania pośredniczącego `throttle`, aby użyć go do obliczenia maksymalnej liczby zapytań:

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

<a name="form-method-spoofing"></a>
## Form Method Spoofing - Podszywanie się pod Metody Form-a

Formularze HTML nie obsługują akcji `PUT`, ` PATCH` lub `DELETE`. Tak więc, definiując trasy `PUT`, ` PATCH` lub `DELETE`, które są wywoływane z formularza HTML, musisz dodać ukryte pole `_method` do formularza. Wartość przesłana za pomocą pola `_method` będzie używana jako metoda żądania HTTP:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Możesz użyć dyrektywy Blade `@method` do wygenerowania wejścia `_method`:

    <form action="/foo/bar" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Accessing The Current Route - Dostęp do aktualnej trasy

Możesz użyć metod `current`, `currentRouteName` oraz `currentRouteAction` na elewacji `Route`, aby uzyskać dostęp do informacji o trasie obsługującej przychodzące żądanie:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Zapoznaj się z dokumentacją interfejsu API dla [podstawowej klasy fasady trasy](https://laravel.com/api/5.5/Illuminate/Routing/Router.html) i [instancja trasy](https://laravel.com/api/5.5/Illuminate/Routing/Route.html), aby przejrzeć wszystkie dostępne metody.
