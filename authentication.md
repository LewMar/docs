# Authentication

- [Introduction - Wprowadzenie](#introduction)
    - [Database Considerations - Rozważania baz danych](#introduction-database-considerations)
- [Authentication Quickstart- Szybki start uwierzytelniania](#authentication-quickstart)
    - [Routing](#included-routing)
    - [Views - Widoki](#included-views)
    - [Authenticating - Uwierzytelnianie](#included-authenticating)
    - [Retrieving The Authenticated User - Pobieranie uwierzytelnionego użytkownika](#retrieving-the-authenticated-user)
    - [Protecting Routes - Chronione trasy](#protecting-routes)
    - [Login Throttling - Ograniczanie logowania](#login-throttling)
- [Manually Authenticating Users - Ręczne uwierzytelnianie użytkowników](#authenticating-users)
    - [Remembering Users - Zapamiętując użytkowników](#remembering-users)
    - [Other Authentication Methods - Inne metody uwierzytelniania](#other-authentication-methods)
- [HTTP Basic Authentication - Uwierzytelnianie podstawowe HTTP](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication - Bezstanowe uwierzytelnianie HTTP](#stateless-http-basic-authentication)
- [Social Authentication - Uwierzytelnianie społecznościowe](https://github.com/laravel/socialite)
- [Adding Custom Guards - Dodawanie niestandardowych strażników](#adding-custom-guards)
- [Adding Custom User Providers - Dodawanie niestandardowych dostawców użytkowników](#adding-custom-user-providers)
    - [The User Provider Contract - Umowa z dostawcą użytkownika](#the-user-provider-contract)
    - [The Authenticatable Contract - Autentyczny kontrakt](#the-authenticatable-contract)
- [Events - Zdarzenia](#events)

<a name="introduction"></a>
## Introduction - Wprowadzenie

> {tip} **Chcesz szybko zacząć?** Uruchom `php artisan make:auth` i `php artisan migrate` w świeżej aplikacji Laravel. Następnie przejdź do przeglądarki na adres `http://your-app.dev/register` lub dowolny inny adres URL przypisany do Twojej aplikacji. Te dwa polecenia zajmą się ruszeniem całego twojego systemu uwierzytelniania!

Laravel sprawia, że wdrażanie uwierzytelniania jest bardzo proste. W rzeczywistości prawie wszystko jest skonfigurowane dla Ciebie po wyjęciu z pudełka. Plik konfiguracji uwierzytelniania znajduje się w `config/auth.php`, który zawiera kilka dobrze udokumentowanych opcji dostosowywania zachowania usług uwierzytelniania.

Zasadniczo urządzenia uwierzytelniające Laravel składają się z "strażników" i "dostawców". Strażnicy określają sposób uwierzytelniania użytkowników dla każdego żądania. Na przykład Laravel jest dostarczany ze strażnikiem `session`, który utrzymuje stan za pomocą pamięci sesji i ciasteczek.

Dostawcy określają sposób pobierania użytkowników z magazynu trwałego. Laravel oferuje wsparcie dla pobierania użytkowników za pomocą narzędzia Eloquent i narzędzia do budowania zapytań do bazy danych. Możesz jednak zdefiniować dodatkowych dostawców, którzy będą potrzebować twojej aplikacji.

Nie przejmuj się, jeśli to wszystko teraz brzmi myląco! Wiele aplikacji nigdy nie będzie musiało modyfikować domyślnej konfiguracji uwierzytelniania.

<a name="introduction-database-considerations"></a>
### Database Considerations - Rozważania baz danych

Domyślnie Laravel zawiera `App\User` [Eloquent model](/docs/{{version}}/eloquent) w twoim katalogu `app`. Ten model może być używany z domyślnym sterownikiem uwierzytelniania Eloquent. Jeśli aplikacja nie używa Eumquent, możesz użyć sterownika uwierzytelniania `database`, który używa programu Laravel do tworzenia zapytań.

Podczas budowania schematu bazy danych dla modelu `App\User` upewnij się, że kolumna hasła ma co najmniej 60 znaków długości. Utrzymanie domyślnej długości kolumny ciąg 255 znaków byłoby dobrym wyborem.

Powinieneś również sprawdzić, czy twoja tablica `users` (lub odpowiednik) zawiera kolumnę `remember_token`z pustymi hasłami zawierającą 100 znaków. Ta kolumna służy do przechowywania tokena dla użytkowników, którzy wybiorą opcję "zapamiętaj mnie" podczas logowania do aplikacji.

<a name="authentication-quickstart"></a>
## Authentication Quickstart - Szybki start uwierzytelniania

Laravel jest dostarczany z kilkoma wbudowanymi kontrolerami uwierzytelniania, które znajdują się w przestrzeni nazw `App\Http\Controllers\Auth`. `RegisterController` obsługuje rejestrację nowego użytkownika, `LoginController` obsługuje uwierzytelnianie, `ForgotPasswordController` obsługuje łącza e-mailingowe do resetowania haseł, a `ResetPasswordController` zawiera logikę do resetowania haseł. Każdy z tych kontrolerów używa cechy (trait), aby uwzględnić ich niezbędne metody. W wielu aplikacjach w ogóle nie trzeba modyfikować tych kontrolerów.

<a name="included-routing"></a>
### Routing

Laravel zapewnia szybki sposób na skonfigurowanie wszystkich tras i widoków potrzebnych do uwierzytelnienia za pomocą jednego prostego polecenia:

    php artisan make:auth

To polecenie powinno być używane w nowych aplikacjach i będzie instalować widok układu, widoki rejestracji i logowania, a także trasy dla wszystkich punktów końcowych uwierzytelniania. Generator `HomeController` zostanie również wygenerowany w celu obsługi żądań post-login na pulpicie aplikacji.

<a name="included-views"></a>
### Views - Widoki

Jak wspomniano w poprzedniej sekcji, polecenie `php artisan make:auth` utworzy wszystkie widoki potrzebne do uwierzytelnienia i umieści je w katalogu `resources/views/auth`.

Polecenie `make:auth` utworzy również katalog `resources/views/layouts` zawierający układ podstawowy dla twojej aplikacji. Wszystkie te widoki korzystają ze schematu Bootstrap CSS, ale możesz dowolnie je dostosowywać.

<a name="included-authenticating"></a>
### Authenticating - Uwierzytelnianie

Teraz, gdy masz skonfigurowane trasy i widoki dla dołączonych kontrolerów uwierzytelniania, jesteś gotowy do zarejestrowania i uwierzytelnienia nowych użytkowników dla twojej aplikacji! Możesz uzyskać dostęp do aplikacji w przeglądarce, ponieważ kontrolery uwierzytelniania zawierają już logikę (za pośrednictwem ich cech), aby uwierzytelnić istniejących użytkowników i przechowywać nowych użytkowników w bazie danych.

#### Path Customization - Dostosowywanie ścieżki

Gdy użytkownik zostanie pomyślnie uwierzytelniony, zostanie przekierowany do identyfikatora URI `/home`. Można dostosować lokalizację przekierowania po uwierzytelnieniu, definiując właściwość `redirectTo` w `LoginController`, `RegisterController` i `ResetPasswordController`:

    protected $redirectTo = '/';

Następnie należy zmodyfikować metodę `handle` oprogramowania pośredniego `RedirectIfAuthenticated`, aby użyć nowego identyfikatora URI podczas przekierowywania użytkownika.

Jeśli ścieżka przekierowania wymaga logiki generowania niestandardowego, możesz zdefiniować metodę `redirectTo` zamiast właściwości` redirectTo`:

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} Metoda `redirectTo` będzie miała pierwszeństwo przed atrybutem `redirectTo`

#### 

By default, Laravel uses the `email` field for authentication. If you would like to customize this, you may define a `username` method on your `LoginController`:

    public function username()
    {
        return 'username';
    }

#### Guard Customization - Dostosowanie nazwy użytkownika

Możesz także dostosować "strażnika" używanego do uwierzytelniania i rejestrowania użytkowników. Aby rozpocząć, zdefiniuj metodę `guard` na swoim `LoginController`, `RegisterController` i `ResetPasswordController`. Metoda powinna zwrócić instancję strażnika:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Validation / Storage Customization - Sprawdzanie poprawności / pamięci masowej

Aby zmodyfikować pola formularza, które są wymagane, gdy nowy użytkownik rejestruje się w aplikacji lub aby dostosować sposób przechowywania nowych użytkowników do bazy danych, można zmodyfikować klasę `RegisterController`. Ta klasa jest odpowiedzialna za weryfikację i tworzenie nowych użytkowników aplikacji.

Metoda `validator` kontrolera` RegisterController` zawiera reguły sprawdzania poprawności dla nowych użytkowników aplikacji. Możesz dowolnie modyfikować tę metodę.

Metoda `create` w `RegisterController` jest odpowiedzialna za tworzenie nowych rekordów `App\User` w bazie danych za pomocą [Eloquent ORM](/docs/{{version}}/eloquent). Możesz modyfikować tę metodę zgodnie z potrzebami swojej bazy danych.

<a name="retrieving-the-authenticated-user"></a>
### Retrieving The Authenticated User - Pobieranie uwierzytelnionego użytkownika

Możesz uzyskać dostęp do uwierzytelnionego użytkownika poprzez elewację `Auth`:

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

Alternatywnie, po uwierzytelnieniu użytkownika można uzyskać dostęp do uwierzytelnionego użytkownika za pośrednictwem instancji `Illuminate\Http\Request`. Pamiętaj, że klasy z wskazówkami typu będą automatycznie wprowadzane do twoich metod kontrolera:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

#### Determining If The Current User Is Authenticated - Określanie, czy bieżący użytkownik jest uwierzytelniony

Aby ustalić, czy użytkownik jest już zalogowany w aplikacji, możesz użyć metody `check` na elewacji `Auth`, która zwróci `true`, jeśli użytkownik jest uwierzytelniony:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} Mimo że możliwe jest ustalenie, czy użytkownik jest uwierzytelniany za pomocą metody `check`, zazwyczaj używa się oprogramowania pośredniego w celu sprawdzenia, czy użytkownik jest uwierzytelniany przed zezwoleniem użytkownikowi na dostęp do określonych tras / kontrolerów. Aby dowiedzieć się więcej na ten temat, zapoznaj się z dokumentacją [chronione trasy](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Protecting Routes - Chronione trasy

[Trase oprogramowanie pośrednie](/docs/{{version}}/middleware) może być używane tylko w celu umożliwienia uwierzytelnionym użytkownikom dostępu do danej trasy. Laravel jest dostarczany z oprogramowaniem pośredniczącym `auth`, które jest zdefiniowane w `Illuminate\Auth\Middleware\Authenticate`. Ponieważ to oprogramowanie pośrednie jest już zarejestrowane w jądrze HTTP, wystarczy dołączyć oprogramowanie pośrednie do definicji trasy:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

Oczywiście, jeśli używasz [kontrolerów](/docs/{{version}}/controllers), możesz wywołać metodę `middleware` z konstruktora kontrolera, zamiast dołączać ją bezpośrednio do definicji trasy:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Redirecting Unauthenticated Users - Przekierowywanie nieuwierzytelnionych użytkowników

Gdy oprogramowanie pośredniczące `auth` wykryje nieautoryzowanego użytkownika, zwróci odpowiedź JSON `401` lub, jeśli żądanie nie jest żądaniem AJAX, przekieruje użytkownika do `login` [nazwanej trasy](/docs/{{version}}/routing#named-routes).

Możesz zmodyfikować to zachowanie, definiując funkcję `unauthenticated` w pliku `app/Exceptions/Hander.php`:

    use Illuminate\Auth\AuthenticationException;

    protected function unauthenticated($request, AuthenticationException $exception)
    {
        return $request->expectsJson()
                    ? response()->json(['message' => $exception->getMessage()], 401)
                    : redirect()->guest(route('login'));
    }

#### Specifying A Guard - Określanie Strażnika

Podczas dołączania oprogramowania pośredniego `auth` do trasy, możesz również określić, która osłona powinna być użyta do uwierzytelnienia użytkownika. Określony strażnik powinien odpowiadać jednemu z kluczy w tablicy `guards` pliku konfiguracyjnego `auth.php`:

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### Login Throttling - Ograniczanie logowania

Jeśli korzystasz z wbudowanej w Laravel klasy `LoginController`, właściwość  `Illuminate\Foundation\Auth\ThrottlesLogins` zostanie już uwzględniona w twoim kontrolerze. Domyślnie użytkownik nie będzie mógł zalogować się przez minutę, jeśli nie uda mu się podać poprawnych poświadczeń po kilku próbach. Ograniczanie jest unikalne dla nazwy użytkownika / adresu e-mail użytkownika i jego adresu IP.

<a name="authenticating-users"></a>
## Manually Authenticating Users - Ręczne uwierzytelnianie użytkowników

Oczywiście nie musisz używać kontrolerów uwierzytelniania zawartych w Laravel. Jeśli zdecydujesz się usunąć te kontrolery, będziesz musiał zarządzać uwierzytelnianiem użytkowników bezpośrednio za pomocą klas uwierzytelniania Laravel. Nie martw się, to nic nadzwyczajnego!

Uzyskamy dostęp do usług uwierzytelniania Laravel za pośrednictwem `Auth` [fasady](/docs/{{version}}/facades), więc musimy upewnić się, że zaimportujemy fasadę `Auth` na górze klasy. Następnie sprawdźmy metodę `attempt`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

Metoda `attempt` akceptuje tablicę par klucz / wartość jako swój pierwszy argument. Wartości w tablicy zostaną wykorzystane do znalezienia użytkownika w tabeli bazy danych. Tak więc w powyższym przykładzie użytkownik zostanie pobrany przez wartość kolumny `email`. Jeśli użytkownik zostanie znaleziony, hashowane hasło przechowywane w bazie danych zostanie porównane z wartością `password` przekazaną do metody za pośrednictwem tablicy. Nie powinieneś mieszać hasła określonego jako wartość `password`, ponieważ struktura automatycznie będzie mieszała wartość przed porównaniem jej z hashowanym hasłem w bazie danych. Jeśli dwa hashe pasują do uwierzytelnionej sesji, zostanie uruchomiona dla użytkownika.

The `attempt` method will return `true` if authentication was successful. Otherwise, `false` will be returned.
Metoda `attempt` zwróci `true`, jeśli uwierzytelnienie zakończyło się pomyślnie. W przeciwnym razie zwracane będzie `false`.

Metoda `intended` dla przekierowacza przekieruje użytkownika do adresu URL, do którego próbował uzyskać dostęp, zanim zostanie przechwycony przez oprogramowanie pośredniczące do uwierzytelniania. Identyfikator URI zapasowy może być nadany tej metodzie, jeśli zamierzony cel nie jest dostępny.

#### Specifying Additional Conditions - Określanie dodatkowych warunków

Jeśli chcesz, możesz dodać dodatkowe warunki do kwerendy uwierzytelniającej oprócz e-mail użytkownika i hasła. Na przykład możemy zweryfikować, że użytkownik jest oznaczony jako "aktywny":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} W tych przykładach `email` nie jest wymaganą opcją, jest jedynie używany jako przykład. Należy użyć dowolnej nazwy kolumny odpowiadającej "nazwie użytkownika" w bazie danych.

#### Accessing Specific Guard Instances - Uzyskiwanie dostępu do konkretnych przypadków instancji Strażnika

Możesz określić, którą instancję strażniczą chcesz użyć, używając metody `guard` na elewacji `Auth`. Umożliwia to zarządzanie uwierzytelnianiem dla oddzielnych części aplikacji przy użyciu całkowicie oddzielnych modeli uwierzytelniających lub tabel użytkowników.

Nazwa strażnika przekazana do metody `guard` powinna odpowiadać jednemu ze strażników skonfigurowanych w pliku konfiguracyjnym` auth.php`:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Logging Out - Wylogowywanie się

Aby wylogować użytkowników z aplikacji, możesz użyć metody `logout` na elewacji `Auth`. Spowoduje to wyczyszczenie informacji uwierzytelniających w sesji użytkownika:

    Auth::logout();

<a name="remembering-users"></a>
### Remembering Users - Zapamiętując użytkowników

Jeśli chcesz udostępnić funkcję "zapamiętaj mnie" w swojej aplikacji, możesz przekazać wartość logiczną jako drugi argument metody `attempt`, która będzie utrzymywać użytkownika w stanie uwierzytelnienia przez czas nieokreślony lub do momentu ręcznego wylogowania. Oczywiście, twoja tabela `users` musi zawierać kolumnę `remember_token`, która będzie używana do przechowywania tokena "zapamiętaj mnie".

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} Jeśli używasz wbudowanego `LoginController`, który jest dostarczany z Laravel, właściwa logika do "zapamiętywania" użytkowników jest już zaimplementowana przez cechy (traits) używane przez kontroler.

Jeśli "pamiętasz" użytkowników, możesz użyć metody `viaRemember` w celu ustalenia, czy użytkownik został uwierzytelniony za pomocą cookie "zapamiętaj mnie":

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Other Authentication Methods - Inne metody uwierzytelniania

#### Authenticate A User Instance - Uwierzytelnij instancję użytkownika

Jeśli chcesz zalogować istniejącą instancję użytkownika do swojej aplikacji, możesz wywołać metodę `login` z instancją użytkownika. Podany obiekt musi być implementacją `Illuminate\Contracts\Auth\Authenticatable` [kontrakt](/docs/{{version}}/contracts). Oczywiście model `App\User` dołączony do Laravel już implementuje ten interfejs:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

Oczywiście możesz określić instancję strażniczą, której chcesz użyć:

    Auth::guard('admin')->login($user);

#### Authenticate A User By ID - Uwierzytelnij użytkownika według identyfikatora

Aby zalogować użytkownika do aplikacji za pomocą swojego identyfikatora, możesz użyć metody `loginUsingId`. Ta metoda akceptuje klucz podstawowy użytkownika, który chcesz uwierzytelnić:

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### Authenticate A User Once - Uwierzytelnij użytkownika raz

Możesz użyć metody `once`, aby zalogować użytkownika do aplikacji dla pojedynczego żądania. Żadne sesje ani pliki cookie nie będą używane, co oznacza, że ta metoda może być pomocna przy tworzeniu bezstanowego interfejsu API:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication - Uwierzytelnianie podstawowe HTTP

[Uwierzytelnianie podstawowe HTTP](https://en.wikipedia.org/wiki/Basic_access_authentication) zapewnia szybki sposób uwierzytelnienia użytkowników aplikacji bez konieczności konfigurowania dedykowanej strony logowania. Aby rozpocząć, podłącz `auth.basic` [oprogramowanie pośrednie](/docs/{{version}}/middleware) do swojej trasy. Program pośredniczący `auth.basic` jest dołączony do frameworka Laravel, więc nie trzeba go definiować:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

Po podłączeniu oprogramowania pośredniego do trasy użytkownik zostanie automatycznie poproszony o podanie poświadczeń podczas uzyskiwania dostępu do trasy w przeglądarce. Domyślnie oprogramowanie pośredniczące `auth.basic` używa kolumny` email` w rekordzie użytkownika jako "nazwa użytkownika".

#### A Note On FastCGI - Uwaga dotycząca FastCGI

Jeśli korzystasz z PHP FastCGI, uwierzytelnianie HTTP Basic może nie działać poprawnie po wyjęciu z pudełka. W pliku `.htaccess` należy dodać następujące linie:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication - Bezstanowe uwierzytelnianie HTTP

Możesz również użyć podstawowego uwierzytelniania HTTP bez ustawiania ciasteczka identyfikującego użytkownika w sesji, co jest szczególnie przydatne w przypadku uwierzytelniania API. Aby to zrobić, [zdefiniuj oprogramowanie pośrednie](/docs/{{version}}/middleware), które wywołuje metodę `onceBasic`. Jeśli żadna odpowiedź nie zostanie zwrócona przez metodę `onceBasic`, żądanie może zostać przekazane dalej do aplikacji:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

Następnie [zarejestruj oprogramowanie pośredniczące do trasy](/docs/{{version}}/middleware#registering-middleware) i dołącz je do trasy:

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="adding-custom-guards"></a>
## Adding Custom Guards - Dodawanie niestandardowych strażników

Możesz zdefiniować własne zabezpieczenia uwierzytelniające za pomocą metody `extend` na elewacji `Auth`. Powinieneś umieścić to wywołanie na `extend` w [usługodawcy](/docs/{{version}}/providers). Ponieważ Laravel jest już dostarczany z `AuthServiceProvider`, możemy umieścić kod w tym dostawcy:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

Jak widać w powyższym przykładzie, wywołanie zwrotne przekazane do metody `extend` powinno zwrócić implementację `Illuminate\Contracts\Auth\Guard`. Ten interfejs zawiera kilka metod, które musisz zastosować, aby zdefiniować niestandardowego strażnika. Po zdefiniowaniu własnego strażnika możesz użyć tego strażnika w konfiguracji `guards` pliku konfiguracyjnego `auth.php`:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Adding Custom User Providers - Dodawanie niestandardowych dostawców użytkowników

Jeśli nie używasz tradycyjnej relacyjnej bazy danych do przechowywania użytkowników, musisz rozszerzyć Laravel o własnego dostawcę uwierzytelniania. Będziemy używać metody `provider` na elewacji `Auth`, aby zdefiniować niestandardowego użytkownika:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function ($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

Po zarejestrowaniu dostawcy przy użyciu metody `provider` możesz przełączyć się na nowego dostawcę użytkownika w pliku konfiguracyjnym `auth.php`. Najpierw zdefiniuj `provider`, który używa twojego nowego sterownika:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Wreszcie, możesz użyć tego dostawcy w swojej konfiguracji `guards`:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### The User Provider Contract - Umowa z dostawcą użytkownika

Implementacje `Illuminate\Contracts\Auth\UserProvider` są odpowiedzialne tylko za pobranie implementacji `Illuminate\Contracts\Auth\Authenticatable` z trwałego systemu pamięci masowej, takiego jak MySQL, Riak itp. Te dwa interfejsy umożliwiają mechanizm uwierzytelniania Laravel aby kontynuować działanie bez względu na to, jak przechowywane są dane użytkownika lub jaki typ klasy jest używany do jego reprezentowania.

Rzućmy okiem na kontrakt `Illuminate\Contracts\Auth\UserProvider`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

Funkcja `retrieveById` zwykle otrzymuje klucz reprezentujący użytkownika, taki jak automatycznie zwiększający się identyfikator z bazy danych MySQL. Implementacja `Authenticatable` zgodna z identyfikatorem powinna zostać pobrana i zwrócona przez metodę.

Funkcja `retrieveByToken` pobiera użytkownika przez swój unikalny `$identifier` oraz "remember me" `$token`, przechowywany w polu `remember_token`. Podobnie jak w poprzedniej metodzie, powinna zostać zwrócona implementacja `Authenticatable`.

Metoda `updateRememberToken` aktualizuje `$user` pole `remember_token` z nowym `$token`. Nowy token może być nowym tokenem, przypisanym do udanej próby logowania "zapamiętaj mnie" lub gdy użytkownik się wyloguje.

Metoda `retrieveByCredentials` odbiera tablicę poświadczeń przekazanych do metody `Auth::attempt` przy próbie zalogowania się do aplikacji. Metoda powinna następnie "zapytać" podstawowy stały magazyn dla użytkownika pasującego do tych poświadczeń. Zwykle ta metoda uruchomi zapytanie z warunkiem "where" na `$credentials['username']`. Metoda powinna następnie zwrócić implementację `Authenticatable`. **Ta metoda nie powinna podejmować prób uwierzytelniania ani uwierzytelniania haseł.**

Metoda `validateCredentials` powinna porównać podany `$user` z `$credentials` w celu uwierzytelnienia użytkownika. Na przykład ta metoda powinna prawdopodobnie używać `Hash::check` do porównywania wartości `$user->getAuthPassword()`z wartością `$credentials['password']`. Ta metoda powinna zwracać wartość `true` lub `false` , wskazując, czy hasło jest prawidłowe.

<a name="the-authenticatable-contract"></a>
### The Authenticatable Contract - Autentyczny kontrakt

Po przeanalizowaniu każdej metody na `UserProvider`, spójrzmy na umowę `Authenticatable`. Pamiętaj, że dostawca powinien zwrócić implementacje tego interfejsu z metod `retrieveById` i `retrieveByCredentials`:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

Ten interfejs jest prosty. Metoda `getAuthIdentifierName` powinna zwrócić nazwę pola "klucz podstawowy" użytkownika, a metoda `getAuthIdentifier` powinna zwrócić "klucz podstawowy" użytkownika. W back-endach MySQL ponownie będzie to automatycznie zwiększany klucz podstawowy. `GetAuthPassword` powinno zwracać hashowane hasło użytkownika. Ten interfejs umożliwia systemowi uwierzytelniania pracę z dowolną klasą użytkownika, niezależnie od tego, z której warstwy ORM lub warstwy abstrakcji magazynu korzystasz. Domyślnie Laravel zawiera klasę `User` w katalogu `app`, która implementuje ten interfejs, więc możesz zapoznać się z tą klasą dla przykładu implementacji.

<a name="events"></a>
## Events - Zdarzenia

Podczas procesu uwierzytelniania Laravel gromadzi różne [zdarzenia](/docs/{{version}}/events). Możesz dołączyć detektory do tych zdarzeń w swoim `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],

        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
