# Controllers

- [Introduction - Wprowadzenie](#introduction)
- [Basic Controllers - Podstawowe kontrolery](#basic-controllers)
    - [Defining Controllers - Definiowanie kontrolerów](#defining-controllers)
    - [Controllers & Namespaces - Kontrolery i przestrzenie nazw](#controllers-and-namespaces)
    - [Single Action Controllers - Kontrolery pojedynczej akcji](#single-action-controllers)
- [Controller Middleware - Oprogramowanie pośredniczące kontrolera](#controller-middleware)
- [Resource Controllers - Kontrolery zasobów](#resource-controllers)
    - [Partial Resource Routes - Trasy częściowych zasobów](#restful-partial-resource-routes)
    - [Naming Resource Routes - Nazewnictwo tras zasobów](#restful-naming-resource-routes)
    - [Naming Resource Route Parameters - Nazewnictwo parametrów trasy zasobów](#restful-naming-resource-route-parameters)
    - [Localizing Resource URIs - Lokalizowanie identyfikatorów URI zasobów](#restful-localizing-resource-uris)
    - [Supplementing Resource Controllers - Uzupełnianie kontrolerów zasobów](#restful-supplementing-resource-controllers)
- [Dependency Injection & Controllers - Wsztrzykiwanie zależności i kontrolery](#dependency-injection-and-controllers)
- [Route Caching - Buforowanie tras](#route-caching)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Zamiast definiować całą logikę obsługi żądań jako Closures w plikach tras, możesz zorganizować to zachowanie za pomocą klas Controller. Kontrolery mogą grupować powiązaną logikę obsługi żądań w jedną klasę. Kontrolery są przechowywane w katalogu `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Basic Controllers - Podstawowe kontrolery

<a name="defining-controllers"></a>
### Defining Controllers - Definiowanie kontrolerów

Poniżej znajduje się przykład podstawowej klasy kontrolera. Zauważ, że kontroler rozszerza klasę kontrolera podstawowego zawartą w Laravel. Klasa bazowa udostępnia kilka wygodnych metod, takich jak metoda `middleware`, która może być używana do dołączania oprogramowania pośredniego do działań kontrolera:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Możesz zdefiniować trasę do akcji tego kontrolera w następujący sposób:

    Route::get('user/{id}', 'UserController@show');

Teraz, gdy żądanie pasuje do określonego identyfikatora URI trasy, zostanie uruchomiona metoda `show` w klasie `UserController`. Oczywiście parametry trasy również zostaną przekazane do metody.

> {tip} Kontrolery nie są **wymagane** do rozszerzenia klasy bazowej. Jednak nie będziesz mieć dostępu do wygodnych funkcji, takich jak metody `middleware`, `validate`, i `dispatch`.

<a name="controllers-and-namespaces"></a>
### Controllers & Namespaces - Kontrolery i przestrzenie nazw

Bardzo ważne jest, aby pamiętać, że nie trzeba było określać pełnej przestrzeni nazw kontrolera podczas definiowania trasy kontrolera. Ponieważ `RouteServiceProvider` ładuje twoje pliki trasy w grupie tras, która zawiera przestrzeń nazw, my tylko określiliśmy część nazwy klasy, która pochodzi po części aplikacji `App\Http\Controllers`.

Jeśli zdecydujesz się zagnieździć kontrolery głębiej w katalogu `App\Http\Controllers`, po prostu użyj konkretnej nazwy klasy względem obszaru nazw root `App\Http\Controllers`. Tak więc, jeśli twoją pełną klasą kontrolerów jest `App\Http\Controllers\Photos\AdminController`, powinieneś zarejestrować trasy do kontrolera tak:

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### Single Action Controllers - Kontrolery pojedynczej akcji

Jeśli chciałbyś zdefiniować kontroler, który obsługuje tylko jedną akcję, możesz umieścić na kontrolerze pojedynczą metodę `__invoke`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Podczas rejestrowania tras dla kontrolerów pojedynczego działania nie trzeba określać metody:

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## Controller Middleware - Oprogramowanie pośredniczące kontrolera

[Middleware](/docs/{{version}}/middleware) może być przypisany do tras kontrolera w twoich plikach trasy:

    Route::get('profile', 'UserController@show')->middleware('auth');

Jednak wygodniej jest określić oprogramowanie pośrednie w konstruktorze kontrolera. Korzystając z metody  `middleware` z konstruktora kontrolera, możesz łatwo przypisać oprogramowanie pośrednie do działania kontrolera. Możesz nawet ograniczyć oprogramowanie pośrednie tylko do niektórych metod w klasie kontrolera:

    class UserController extends Controller
    {
        /**
         * Instantiate a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

Kontrolery umożliwiają również rejestrację oprogramowania pośredniego za pomocą Closure. Zapewnia to wygodny sposób definiowania oprogramowania pośredniego dla pojedynczego kontrolera bez definiowania całej klasy oprogramowania pośredniego:

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} Możesz przypisać oprogramowanie pośrednie do podzbioru akcji kontrolera; może to jednak oznaczać, że kontroler jest zbyt duży. Zamiast tego rozważ rozdzielenie kontrolera na wiele mniejszych kontrolerów.

<a name="resource-controllers"></a>
## Resource Controllers - Kontrolery zasobów

Routing zasobów Laravel przypisuje typowe trasy "CRUD" do kontrolera za pomocą pojedynczej linii kodu. Na przykład możesz chcieć utworzyć kontroler, który obsługuje wszystkie żądania HTTP dla "photos" przechowywanych przez twoją aplikację. Używając polecenia `make:controller` Artisan, możemy szybko stworzyć taki kontroler:

    php artisan make:controller PhotoController --resource

To polecenie wygeneruje kontroler w `app/Http/Controllers/PhotoController.php`. Kontroler będzie zawierał metodę dla każdej dostępnej operacji zasobowej.

Następnie możesz zarejestrować żródłową trasę do kontrolera:

    Route::resource('photos', 'PhotoController');

Ta pojedyncza deklaracja trasy tworzy wiele tras, aby obsłużyć różne akcje w zasobach. Wygenerowany kontroler będzie już dysponował metodami skrótowymi dla każdego z tych działań, w tym uwagami informującymi o czasownikach HTTP i obsługiwanych identyfikatorach URI.

Możesz zarejestrować wiele kontrolerów zasobów jednocześnie, przekazując tablicę do metody `resources`:

    Route::resources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

#### Actions Handled By Resource Controller - Akcje obsługiwane przez kontroler zasobów

Verb      | URI                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### Specifying The Resource Model - Określanie modelu zasobów

Jeśli używasz wiązania modelu trasy i chcesz, aby metody kontrolera zasobów wskazowki instancji modelu, możesz użyć opcji `--model` podczas generowania kontrolera:

    php artisan make:controller PhotoController --resource --model=Photo

#### Spoofing Form Methods - Okroić metody Form

Ponieważ formularze HTML nie mogą tworzyć żądań typu `PUT`, `PATCH`, or `DELETE`, konieczne będzie dodanie ukrytego pola `_method` w celu sfałszowania tych czasowników HTTP. Pomocnik `method_field` może utworzyć to pole dla ciebie:

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### Partial Resource Routes - Trasy częściowych zasobów

Podczas deklarowania trasy zasobu możesz określić podzestaw działań, które kontroler ma obsłużyć zamiast pełnego zestawu domyślnych działań:

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

#### API Resource Routes - Trasy zasobów API

Podczas deklarowania tras zasobów, które będą używane przez interfejsy API, zwykle chcesz wykluczyć trasy, które przedstawiają szablony HTML, takie jak `create` i `edit`. Dla wygody możesz użyć metody `apiResource`, aby automatycznie wykluczyć te dwie trasy:

    Route::apiResource('photo', 'PhotoController');
    
Możesz zarejestrować wiele kontrolerów zasobów API jednocześnie, przekazując tablicę do metody `apiResources`:

    Route::apiResources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

<a name="restful-naming-resource-routes"></a>
### Naming Resource Routes - Nazewnictwo tras zasobów

Domyślnie wszystkie akcje kontrolera zasobów mają nazwę trasy; możesz jednak zastąpić te nazwy, przekazując tablicę `names` z opcjami:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### Naming Resource Route Parameters - Nazewnictwo parametrów trasy zasobów

Domyślnie `Route::resource` utworzy parametry trasy dla twoich tras zasobów na podstawie "zindywidualizowanej" wersji nazwy zasobu. Możesz łatwo zmienić to na podstawie zasobów, przekazując `parameters` w tablicy opcji. Tablica `parameters` powinna być asocjacyjną tablicą nazw zasobów i nazw parametrów:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

 Powyższy przykład generuje następujące identyfikatory URI dla trasy `show` zasobu:

    /user/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### Localizing Resource URIs - Lokalizowanie identyfikatorów URI zasobów

Domyślnie `Route::resource` utworzy identyfikatory URI zasobów używając angielskich czasowników. Jeśli chcesz zlokalizować czasowniki akcji `create` i` edit`, możesz użyć metody `Route::resourceVerbs`. Można to zrobić za pomocą metody `boot` twojego` AppServiceProvider`:

    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }

Po dostosowaniu czasowników rejestracja trasy zasobów, np. `Route::resource('fotos', 'PhotoController')` spowoduje utworzenie następujących identyfikatorów URI:

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### Supplementing Resource Controllers - Uzupełnianie kontrolerów zasobów

Jeśli chcesz dodać dodatkowe trasy do kontrolera zasobów poza domyślny zestaw tras zasobów, powinieneś zdefiniować te trasy przed wywołaniem `Route::resource`; w przeciwnym razie trasy zdefiniowane za pomocą metody `resource` mogą nieumyślnie mieć pierwszeństwo przed Twoimi dodatkowymi trasami:

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} Pamiętaj, aby skupić się na kontrolerach. Jeśli rutynowo potrzebujesz metod wykraczających poza typowy zestaw akcji zasobów, rozważ podzielenie kontrolera na dwa mniejsze kontrolery

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers - Wsztrzykiwanie zależności i kontrolery

#### Constructor Injection - Wstrzykiwanie konstruktora

Laravel [kontener usługi](/docs/{{version}}/container) służy do rozwiązywania wszystkich kontrolerów Laravel. W rezultacie możesz wpisać wskazówkę o zależnościach, których twój kontroler może potrzebować w swoim konstruktorze. Zadeklarowane zależności zostaną automatycznie rozwiązane i wstrzyknięte do instancji kontrolera:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

Oczywiście możesz również wpisać wskazówkę dowolnej [umowy Laravella](/docs/{{version}}/contracts). Jeśli kontener może go rozwiązać, możesz wpisać wskazówkę do niego. W zależności od aplikacji wstrzyknięcie twoich zależności do kontrolera może zapewnić lepszą testowalność.

#### Method Injection - Wstrzyknięcie metody

Oprócz iniekcji konstruktora można również wpisywać zależności do swoich metod kontrolera. Typowym przypadkiem użycia metody iniekcji jest wstrzykiwanie instancji `Illuminate\Http\Request` do metod kontrolera:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

Jeśli twoja metoda kontrolera oczekuje również wprowadzenia z parametru trasy, po prostu wypisz argumenty trasy po innych zależnościach. Na przykład, jeśli twoja trasa jest zdefiniowana w następujący sposób:

    Route::put('user/{id}', 'UserController@update');

Nadal możesz wpisać wskazówkę  `Illuminate\Http\Request` i uzyskać dostęp do parametru` id`, definiując swoją metodę kontrolera w następujący sposób:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the given user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## Route Caching - Buforowanie tras

> {note} Trasy oparte na Closure nie mogą być buforowane. Aby korzystać z buforowania tras, musisz przekonwertować wszystkie trasy Closure na klasy kontrolerów.

Jeśli twoja aplikacja używa wyłącznie tras opartych na kontrolerze, powinieneś skorzystać z pamięci podręcznej tras Laravel. Korzystanie z pamięci podręcznej tras drastycznie skraca czas potrzebny na zarejestrowanie wszystkich tras aplikacji. W niektórych przypadkach rejestracja trasy może być nawet do 100 razy szybsza. Aby wygenerować pamięć podręczną trasy, po prostu wykonaj polecenie `route:cache` Artisan:

    php artisan route:cache

Po uruchomieniu tego polecenia, buforowany plik tras zostanie załadowany na każde żądanie. Pamiętaj, że jeśli dodasz nowe trasy, będziesz musiał wygenerować nową pamięć podręczną tras. Z tego powodu podczas uruchamiania projektu należy uruchamiać tylko polecenie `route:cache`.

Możesz użyć polecenia `route:clear`, aby wyczyścić pamięć podręczną trasy:

    php artisan route:clear
