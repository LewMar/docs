# HTTP Responses

- [Creating Responses - Tworzenie odpowiedzi](#creating-responses)
    - [Attaching Headers To Responses - Dołączanie nagłówków do odpowiedzi](#attaching-headers-to-responses)
    - [Attaching Cookies To Responses - Dołączanie plików cookie do odpowiedzi](#attaching-cookies-to-responses)
    - [Cookies & Encryption - Pliki cookie i szyfrowanie](#cookies-and-encryption)
- [Redirects - Przekierowania](#redirects)
    - [Redirecting To Named Routes - Przekierowanie do nazwanych tras](#redirecting-named-routes)
    - [Redirecting To Controller Actions - Przekierowanie do akcji kontrolera](#redirecting-controller-actions)
    - [Redirecting To External Domains - Przekierowanie do domen zewnętrznych](#redirecting-external-domains)
    - [Redirecting With Flashed Session Data - Przekierowanie z migawką danych sesji](#redirecting-with-flashed-session-data)
- [Other Response Types - Inne typy odpowiedzi](#other-response-types)
    - [View Responses - Wyświetl odpowiedzi](#view-responses)
    - [JSON Responses - Odpowiedzi JSON](#json-responses)
    - [File Downloads - Pobieranie plików](#file-downloads)
    - [File Responses - Odpowiedzi plików](#file-responses)
- [Response Macros - Makra odpowiedzi](#response-macros)

<a name="creating-responses"></a>
## Creating Responses - Tworzenie odpowiedzi

#### Strings & Arrays - Ciągi i tablice

Wszystkie trasy i kontrolery powinny zwrócić odpowiedź, która zostanie odesłana do przeglądarki użytkownika. Laravel oferuje kilka różnych sposobów zwracania odpowiedzi. Najbardziej podstawową odpowiedzią jest zwrócenie ciągu znaków z trasy lub kontrolera. Struktura automatycznie przekształci ciąg znaków w pełną odpowiedź HTTP:

    Route::get('/', function () {
        return 'Hello World';
    });

Oprócz zwracania ciągów z twoich tras i kontrolerów, możesz także zwrócić tablice. Framework automatycznie przekształci tablicę w odpowiedź JSON:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} Czy wiesz, że możesz również zwrócić [Eloquent collections](/docs/{{version}}/eloquent-collections) ze swoich tras lub kontrolerów? Będą one automatycznie konwertowane na JSON. Spróbuj!

#### Response Objects - Obiekty odpowiedzi

Zazwyczaj nie będziesz zwracać prostych ciągów lub tablic z działań na trasie. Zamiast tego będziesz zwracać pełne instancje `Illuminate\Http\Response` lub [widoki](/docs/{{version}}/views).

Zwrócenie pełnej instancji `Response` umożliwia dostosowanie kodu statusu HTTP i nagłówków odpowiedzi. Instancja `Response` dziedziczy po klasie `Symfony\Component\HttpFoundation\Response`, która zapewnia różne metody budowania odpowiedzi HTTP:

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>
#### Attaching Headers To Responses - Dołączanie nagłówków do odpowiedzi

Należy pamiętać, że większość metod odpowiedzi można przenosić, co pozwala na płynną budowę instancji odpowiedzi. Na przykład możesz użyć metody `header`, aby dodać serię nagłówków do odpowiedzi przed wysłaniem jej do użytkownika:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

Lub możesz użyć metody `withHeaders` do określenia tablicy nagłówków, które zostaną dodane do odpowiedzi:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="attaching-cookies-to-responses"></a>
#### Attaching Cookies To Responses - Dołączanie plików cookie do odpowiedzi

Metoda `cookie` w instancjach odpowiedzi umożliwia łatwe dołączanie plików cookie do odpowiedzi. Na przykład możesz użyć metody `cookie`, aby wygenerować plik cookie i płynnie dołączyć go do instancji odpowiedzi, jak na przykład:

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

Metoda `cookie` przyjmuje również kilka argumentów, które są rzadziej używane. Ogólnie rzecz biorąc, te argumenty mają ten sam cel i znaczenie, co argumenty, które zostaną podane do natywnej metody [setcookie](https://secure.php.net/manual/en/function.setcookie.php) PHP:

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

Możesz też użyć fasoli `Cookie` do" umieszczenia w kolejce "plików cookie w celu dołączenia do odpowiedzi wychodzącej z aplikacji. Metoda `queue` akceptuje instancję` Cookie` lub argumenty potrzebne do utworzenia instancji `Cookie`. Te ciasteczka zostaną dołączone do odpowiedzi wychodzącej, zanim zostaną przesłane do przeglądarki:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

<a name="cookies-and-encryption"></a>
#### Cookies & Encryption - Pliki cookie i szyfrowanie

Domyślnie wszystkie pliki cookie generowane przez Laravel są szyfrowane i podpisywane, więc nie mogą być modyfikowane ani czytane przez klienta. Jeśli chcesz wyłączyć szyfrowanie dla podzbioru plików cookie generowanych przez aplikację, możesz użyć właściwości `$except` oprogramowania pośredniego `App\Http\Middleware\EncryptCookies`, które znajduje się w `app/Http/Middleware` katalog:

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## Redirects - Przekierowania

Odpowiedzi przekierowania są instancjami klasy `Illuminate\Http\RedirectResponse` i zawierają odpowiednie nagłówki potrzebne do przekierowania użytkownika do innego adresu URL. Istnieje kilka sposobów generowania instancji `RedirectResponse`. Najprostszą metodą jest użycie globalnego helpera `redirect`:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

Czasami możesz przekierować użytkownika do poprzedniej lokalizacji, na przykład gdy przesłany formularz jest nieprawidłowy. Możesz to zrobić, używając globalnej funkcji `back` pomocnika. Ponieważ ta funkcja korzysta z [sesji](/docs/{{version}}/session), upewnij się, że trasa wywołująca funkcję `back` używa grupy oprogramowania pośredniego `web` lub ma wszystkie zastosowane oprogramowania pośrednie sesji:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### Redirecting To Named Routes - Przekierowanie do nazwanych tras

Gdy wywołujesz helper `redirect` bez parametrów, zwracana jest instancja `Illuminate\Routing\Redirector`, która pozwala wywołać dowolną metodę z instancji `Redirector`. Na przykład, aby wygenerować `RedirectResponse` na nazwaną trasę, możesz użyć metody `route`:

    return redirect()->route('login');

Jeśli twoja trasa ma parametry, możesz przekazać je jako drugi argument metody `route`:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### Populating Parameters Via Eloquent Models - Wypełnianie parametrów przez modele wymowne

Jeśli przekierowujesz do trasy z parametrem "ID", który jest zapełniany z modelu Eloquent, możesz przekazać sam model. Identyfikator zostanie wyodrębniony automatycznie:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Aby dostosować wartość umieszczoną w parametrze route, należy przesłonić metodę `getRouteKey` w modelu Eloquent:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### Redirecting To Controller Actions - Przekierowanie do akcji kontrolera

Możesz również generować przekierowania do [akcji kontrolera](/docs/{{version}}/controllers). Aby to zrobić, należy przekazać kontroler i nazwę akcji do metody `action`. Pamiętaj, że nie musisz określać pełnej przestrzeni nazw do kontrolera, ponieważ `RouteServiceProvider` Laravel automatycznie ustawi obszar nazw kontrolera podstawowego:

    return redirect()->action('HomeController@index');

Jeśli twoja trasa kontrolera wymaga parametrów, możesz przekazać je jako drugi argument do metody `action`:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### Redirecting To External Domains - Przekierowanie do domen zewnętrznych

Czasami może zajść konieczność przekierowania do domeny spoza Twojej aplikacji. Możesz to zrobić, wywołując metodę `away`, która tworzy` RedirectResponse` bez dodatkowego kodowania, sprawdzania poprawności lub weryfikacji:

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### Redirecting With Flashed Session Data - Przekierowanie z migawką danych sesji

Przekierowanie na nowy adres URL i [flashowanie danych do sesji](/docs/{{version}}/session#flash-data) są zwykle wykonywane w tym samym czasie. Zwykle odbywa się to po pomyślnym wykonaniu akcji po wysłaniu wiadomości powitalnej do sesji. Dla wygody możesz utworzyć instancję `RedirectResponse` i dane flash do sesji w jednym, płynnym łańcuchu metod:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Po przekierowaniu użytkownika możesz wyświetlić komunikat wyświetlony w [sesja](/docs/{{version}}/session). Na przykład, używając [składni Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## Other Response Types - Inne typy odpowiedzi

Pomocnik `response` może być używany do generowania innych typów instancji odpowiedzi. Kiedy wywoływacz `response` jest wywoływany bez argumentów, zwracana jest implementacja `Illuminate\Contracts\Routing\ResponseFactory` [kontrakt](/docs/{{version}}/contracts) . Ta umowa zawiera kilka pomocnych metod generowania odpowiedzi.

<a name="view-responses"></a>
### View Responses - Wyświetl odpowiedzi

Jeśli potrzebujesz kontroli nad stanem odpowiedzi i nagłówkami, ale także musisz zwrócić [widok](/docs/{{version}}/views) jako treść odpowiedzi, powinieneś użyć metody `view`:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

Oczywiście, jeśli nie musisz przekazywać niestandardowego kodu statusu HTTP lub niestandardowych nagłówków, powinieneś użyć globalnej funkcji helpera `view`.

<a name="json-responses"></a>
### JSON Responses - Odpowiedzi JSON

Metoda `json` automatycznie ustawi nagłówek` Content-Type` na `application/json`, a także skonwertuje daną tablicę na JSON używając funkcji PHP `json_encode`:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

Jeśli chcesz utworzyć odpowiedź JSONP, możesz użyć metody `json` w połączeniu z metodą` withCallback`:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### File Downloads - Pobieranie plików

Metodę `download` można wykorzystać do wygenerowania odpowiedzi, która zmusza przeglądarkę użytkownika do pobrania pliku w podanej ścieżce. Metoda `download` akceptuje nazwę pliku jako drugi argument metody, który określi nazwę pliku widoczną dla użytkownika pobierającego plik. Na koniec możesz przekazać tablicę nagłówków HTTP jako trzeci argument metody:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

    return response()->download($pathToFile)->deleteFileAfterSend(true);

> {note} Symfony HttpFoundation, która zarządza pobieraniem plików, wymaga, aby pobrany plik miał nazwę pliku ASCII.

#### Streamed Downloads - Pobieranie strumieniowe

Czasami możesz zamienić odpowiedź łańcuchową danej operacji na odpowiedź do pobrania bez konieczności zapisywania zawartości operacji na dysk. W tym scenariuszu możesz użyć metody `streamDownload`. Ta metoda akceptuje wywołanie zwrotne, nazwę pliku i opcjonalną tablicę nagłówków jako argumenty:

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents']
    }, 'laravel-readme.md');

<a name="file-responses"></a>
### File Responses - Odpowiedzi plików

Metoda `file` może być używana do wyświetlania pliku, takiego jak obraz lub plik PDF, bezpośrednio w przeglądarce użytkownika, zamiast inicjować pobieranie. Ta metoda akceptuje ścieżkę do pliku jako swój pierwszy argument, a tablica nagłówków jako drugi argument:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## Response Macros - Makra odpowiedzi

Jeśli chcesz zdefiniować niestandardową odpowiedź, którą możesz ponownie wykorzystać w różnych trasach i kontrolerach, możesz użyć metody `macro` na elewacji `Response`. Na przykład z metody `boot` w [usługodawcy](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's response macros.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

Funkcja `macro` przyjmuje nazwę jako pierwszy argument, a Closure jako drugie. Closure makr zostanie wykonane po wywołaniu nazwy makra z implementacji `ResponseFactory` lub helpera` response`:

    return response()->caps('foo');
