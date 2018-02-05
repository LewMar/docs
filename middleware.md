# Middleware

- [Introduction - Wprowadzenie](#introduction)
- [Defining Middleware - Definiowanie oprogramowania pośredniego](#defining-middleware)
- [Registering Middleware - Rejestrowanie oprogramowania pośredniego](#registering-middleware)
    - [Global Middleware - Globalne oprogramowanie pośredniczące](#global-middleware)
    - [Assigning Middleware To Routes - Przypisywanie oprogramowania pośredniego do routingu (trasowania)](#assigning-middleware-to-routes)
    - [Middleware Groups - Grupy oprogramowania pośredniego](#middleware-groups)
- [Middleware Parameters - Parametry oprogramowania pośredniego](#middleware-parameters)
- [Terminable Middleware - Oprogramowanie Posrednie po zakończeniu](#terminable-middleware)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Oprogramowanie pośredniczące zapewnia wygodny mechanizm filtrowania żądań HTTP wprowadzanych do aplikacji. Na przykład Laravel zawiera oprogramowanie pośrednie, które sprawdza, czy użytkownik Twojej aplikacji jest uwierzytelniony. Jeśli użytkownik nie zostanie uwierzytelniony, oprogramowanie pośrednie przekieruje użytkownika na ekran logowania. Jeśli jednak użytkownik zostanie uwierzytelniony, oprogramowanie pośredniczące pozwoli mu przejść dalej do aplikacji.

Oczywiście dodatkowe oprogramowanie pośrednie można dopisać, aby wykonać wiele zadań poza uwierzytelnianiem. Warstwa pośrednia CORS może być odpowiedzialna za dodanie odpowiednich nagłówków do wszystkich odpowiedzi opuszczających twoją aplikację. Oprogramowanie pośredniczące do rejestrowania może rejestrować wszystkie przychodzące żądania do aplikacji.

Oprogramowanie Laravel zawiera kilka programów pośredniczących, w tym oprogramowanie pośrednie do uwierzytelniania i ochrony CSRF. Wszystkie te oprogramowanie pośrednie znajdują się w katalogu`app/Http/Middleware`.

<a name="defining-middleware"></a>
## Defining Middleware - Definiowanie oprogramowania pośredniego

Aby utworzyć nowe oprogramowanie pośrednie, użyj polecenia `make: middleware` Artisan:

    php artisan make:middleware CheckAge

To polecenie umieści nową klasę `CheckAge` w twoim katalogu  `app/Http/Middleware`. W tym oprogramowaniu pośredniczącym umożliwiamy dostęp do trasy tylko wtedy, gdy podany `age` jest większy niż 200. W przeciwnym razie przekierowujemy użytkowników z powrotem na `home` URI.

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

            return $next($request);
        }
    }

Jak widać, jeśli podany `age` jest mniejszy lub równy `200`, oprogramowanie pośredniczące zwróci przekierowanie HTTP do klienta; w przeciwnym razie wniosek zostanie przekazany dalej do aplikacji. Aby przekazać żądanie głębiej do aplikacji (pozwalając oprogramowaniu pośredniczącemu "przejść"), wywołaj funkcję zwrotną `$next` za pomocą `$request`.

Najlepiej wyobrazić sobie oprogramowanie pośredniczące, ponieważ serie "warstw" żądań HTTP muszą zaliczyć, zanim trafią do aplikacji. Każda warstwa może sprawdzić żądanie, a nawet całkowicie go odrzucić.

### Before & After Middleware - Przed i po oprogramowaniu pośrednim

To, czy oprogramowanie pośrednie działa przed czy po żądaniu, zależy od samego oprogramowania pośredniego. Na przykład poniższe oprogramowanie pośredniczące wykona jakieś zadanie **zanim (befor)** żądanie zostanie obsłużone przez aplikację:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

Jednak to oprogramowanie pośrednie wykona swoje zadanie **po (after)** żądaniu obsłużenia przez aplikację:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>
## Registering Middleware - Rejestrowanie oprogramowania pośredniego

<a name="global-middleware"></a>
### Global Middleware - Globalne oprogramowanie pośredniczące

Jeśli chcesz, aby oprogramowanie pośrednie działało podczas każdego żądania HTTP do twojej aplikacji, wyświetl klasę oprogramowania pośredniego we właściwości `$middleware` twojej klasy `app/Http/Kernel.php`.

<a name="assigning-middleware-to-routes"></a>
### Assigning Middleware To Routes - Przypisywanie oprogramowania pośredniego do routingu (trasowania)

Aby przypisać oprogramowanie pośrednie do określonych tras, należy najpierw przypisać oprogramowanie pośrednie do klucza w pliku `app/Http/Kernel.php`. Domyślnie właściwość `$routeMiddleware` tej klasy zawiera wpisy dla oprogramowania pośredniego dołączonego do Laravel. Aby dodać własny, dodaj go do tej listy i przypisz mu klucz do wyboru. Na przykład:

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

Po zdefiniowaniu oprogramowania pośredniego w kernelu HTTP możesz użyć metody "middleware" do przydzielenia oprogramowania pośredniego do trasy:

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

Możesz także przypisać wiele programów pośredniczących do trasy:

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

Podczas przypisywania oprogramowania pośredniego można również przekazać pełną nazwę klasy:

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### Middleware Groups - Grupy oprogramowania pośredniego

Czasami możesz zgrupować kilka programów pośredniczących pod jednym kluczem, aby ułatwić ich przypisywanie do tras. Możesz to zrobić za pomocą właściwości `$middlewareGroups` swojego jądra HTTP.

Po wyjęciu z pudełka, Laravel jest dostarczany z grupami oprogramowania pośredniczącego `web` i `api`, które zawierają typowe oprogramowanie pośrednie, które możesz zastosować do tras interfejsu użytkownika i interfejsu API:

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

Grupy oprogramowania pośredniego mogą być przypisane do tras i działań kontrolerów przy użyciu tej samej składni, co indywidualne oprogramowanie pośrednie. Ponownie, grupy oprogramowania pośredniego ułatwiają przypisanie wielu programów pośredniczących do trasy naraz:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} Po wyjęciu z pudełka grupa `web` oprogramowania pośredniego jest automatycznie stosowana do pliku `routes/web.php` przez `RouteServiceProvider`.

<a name="middleware-parameters"></a>
## Middleware Parameters - Parametry oprogramowania pośredniego

Oprogramowanie pośrednie może również otrzymywać dodatkowe parametry. Na przykład, jeśli aplikacja musi zweryfikować, czy uwierzytelniony użytkownik ma określoną rolę przed wykonaniem danej czynności, można utworzyć oprogramowanie pośrednie `CheckRole`, które odbiera nazwę roli jako dodatkowy argument.

Dodatkowe parametry oprogramowania pośredniego zostaną przekazane do oprogramowania pośredniego po argumencie `$next`:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

Parametry oprogramowania pośredniego można określić podczas definiowania trasy, oddzielając nazwę oprogramowania pośredniego od parametrów za pomocą `:`. Wiele parametrów należy rozdzielać przecinkami:

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## Terminable Middleware - Oprogramowanie Posrednie po zakończeniu

Czasami oprogramowanie pośrednie może wymagać trochę pracy po wysłaniu odpowiedzi HTTP do przeglądarki. Na przykład oprogramowanie pośredniczące "session" dołączone do Laravel zapisuje dane sesji do pamięci po wysłaniu odpowiedzi do przeglądarki. Jeśli zdefiniujesz metodę `terminate` w oprogramowaniu pośredniczącym, zostanie ona automatycznie wywołana po wysłaniu odpowiedzi do przeglądarki.

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

Metoda `terminate` powinna otrzymać zarówno żądanie, jak i odpowiedź. Po zdefiniowaniu terminatora oprogramowania pośredniego należy go dodać do listy routingu lub globalnego oprogramowania pośredniego w pliku `app/Http/Kernel.php`.

Wywołując metodę `terminate` w oprogramowaniu pośredniczącym, Laravel rozwiąże nową instancję oprogramowania pośredniego z [service container (kontenera usług)](/docs/{{version}}/container). Jeśli chcesz użyć tej samej instancji oprogramowania pośredniego, gdy wywoływane są metody `handle` i` terminate`, zarejestruj oprogramowanie pośredniczące z kontenerem, używając metody `singleton` kontenera.
