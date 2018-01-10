# Release Notes

- [Versioning Scheme - Schemat wersjonowania](#versioning-scheme)
- [Support Policy - Zasady pomocy technicznej](#support-policy)
- [Laravel 5.5 (LTS)](#laravel-5.5)

<a name="versioning-scheme"></a>
## Versioning Scheme - Schemat wersjonowania

Schemat wersjonowania Laravel utrzymuje następującą konwencję: `paradigm.major.minor`. Ważniejsze (major) wydania framework-a są wydawane co sześć miesięcy (luty i sierpień), podczas gdy niewielkie wydania mogą być publikowane tak często, jak co tydzień. Drobne wydania powinny **nigdy** nie zawierać zmian łamania.

Podczas odwoływania się do frameworka Laravel lub jego komponentów z aplikacji lub pakietu, zawsze powinieneś używać ograniczeń wersji, takich jak `5.5.*`, Ponieważ główne wersje Laravel zawierają zmiany zrywające. Staramy się jednak zawsze zapewniać aktualizację do nowej wersji głównej w ciągu jednego dnia lub mniej.

Przesunięcia paradygmatu są rozdzielane przez wiele lat i stanowią fundamentalne przesunięcia w architekturze i konwencjach architektury. Obecnie nie ma opracowywanej zmiany paradygmatu.

#### Why Doesn't Laravel Use Semantic Versioning? - Dlaczego Laravel nie używa wersji semantycznej?

Z jednej strony wszystkie opcjonalne komponenty Laravel (Cashier, Dusk, Valet, Socialite itp.) **wykorzystują** semantyczną wersję. Jednak sama struktura Laravel nie. Powodem tego jest to, że semantyczne wersjonowanie jest "redukcjonistycznym" sposobem określania, czy dwa kawałki kodu są kompatybilne. Nawet jeśli używasz semantycznego wersjonowania, nadal musisz zainstalować uaktualniony pakiet i uruchomić swój automatyczny zestaw testów, aby dowiedzieć się, czy coś jest **faktycznie** niezgodne z bazą kodu.

So, instead, the Laravel framework uses a versioning scheme that is more communicative of the actual scope of the release. Furthermore, since minor releases **never** contain intentional breaking changes, you should never receive a breaking change as long as your version constraints follow the `paradigm.major.*` convention.
Zamiast tego, struktura Laravel używa schematu wersjonowania, który jest bardziej komunikatywny dla faktycznego zakresu wydania. Co więcej, skoro pomniejsze wydania **nigdy** nie zawierają intencjonalnych zmian łamania, nigdy nie powinieneś otrzymywać zmian łamania, o ile ograniczenia wersji są zgodne z konwencją `paradigm.major.*`.

<a name="support-policy"></a>
## Support Policy - Zasady pomocy technicznej

W przypadku wydań LTS, takich jak Laravel 5.5, poprawki są dostarczane przez 2 lata, a poprawki bezpieczeństwa są dostarczane przez 3 lata. Te wydania zapewniają najdłuższe okno obsługi i konserwacji. W przypadku wersji ogólnych poprawki są dostarczane przez 6 miesięcy, a poprawki bezpieczeństwa są dostarczane przez 1 rok.

<a name="laravel-5.5"></a>
## Laravel 5.5 (LTS)

Laravel 5.5 kontynuuje ulepszenia wprowadzone w Laravel 5.4, dodając automatyczne wykrywanie pakietów, zasoby API / transformacje, automatyczną rejestrację poleceń konsoli, kolejkowanie zadań w kolejce, ograniczanie liczby zadań w kolejce, próby zadań oparte na czasie, renderowalne skrzynki pocztowe, możliwe do renderowania i zgłaszane wyjątki, bardziej spójna obsługa wyjątków, udoskonalenia testowania baz danych, prostsze niestandardowe reguły sprawdzania poprawności, React front-end presets, metody `Route::view` i `Route::redirect`, "blokady" dla sterowników pamięci podręcznej Memcached i Redis, powiadomienia na żądanie , bezgłowe wsparcie Chrome w Dusk, wygodne skróty Blade, ulepszone wsparcie dla zaufanego proxy i wiele więcej.

Ponadto Laravel 5.5 zbiega się z wydaniem [Laravel Horizon](https://horizon.laravel.com), pięknym nowym panelem kontrolnym kolejki i systemem konfiguracji dla twoich kolejek Laravel opartych na Redis.

> {tip} W tej dokumentacji podsumowano najważniejsze ulepszenia w framework-u; jednak bardziej szczegółowe dzienniki zmian są zawsze dostępne [na GitHubie](https://github.com/laravel/framework/blob/5.5/CHANGELOG-5.5.md).

### Laravel Horizon

Horizon zapewnia piękny pulpit nawigacyjny i konfigurację opartą na kodzie dla twoich kolejek Laravel zasilanych Redis. Horizon umożliwia łatwe monitorowanie kluczowych parametrów systemu kolejki, takich jak przepustowość pracy, środowisko wykonawcze i awarie zadań.

Cała konfiguracja pracownika jest przechowywana w jednym, prostym pliku konfiguracyjnym, dzięki czemu konfiguracja pozostaje pod kontrolą źródła, z którym cały zespół może współpracować.

For more information on Horizon, check out the [full Horizon documentation](/docs/{{version}}/horizon)
Aby uzyskać więcej informacji na temat Horizon, zapoznaj się z [dokumentacją pełnego horyzontu](/docs/{{version}}/horizon)

### Package Discovery

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/5) dla tej funkcji dostępny na Laracastach.

W poprzednich wersjach Laravel instalacja pakietu zazwyczaj wymagała kilku dodatkowych kroków, takich jak dodanie dostawcy usług do pliku konfiguracyjnego `app` i rejestracja odpowiednich fasad. Jednak począwszy od wersji Laravel 5.5, Laravel może automatycznie wykrywać i rejestrować dostawców usług i fasad.

Na przykład możesz tego doświadczyć instalując popularny pakiet `barryvdh/laravel-debugbar` w swojej aplikacji Laravel. Po zainstalowaniu pakietu przez Composer, pasek debugowania będzie dostępny dla twojej aplikacji bez dodatkowej konfiguracji:

    composer require barryvdh/laravel-debugbar

Deweloperzy pakietów muszą tylko dodawać swoich dostawców usług i fasady do pliku `composer.json` pakietu:

    "extra": {
        "laravel": {
            "providers": [
                "Laravel\\Tinker\\TinkerServiceProvider"
            ]
        }
    },

Aby uzyskać więcej informacji na temat aktualizowania pakietów w celu skorzystania z usług dostawcy i odkrywania fasad, zapoznaj się z pełną dokumentacją [package development](/docs/{{version}}/packages).

### API Resources

Podczas budowania interfejsu API może być potrzebna warstwa transformacji, która znajduje się między modelami wymownymi a odpowiedziami JSON, które są faktycznie zwracane użytkownikom aplikacji. Klasy zasobów Laravel umożliwiają ekspresyjną i łatwą transformację modeli i kolekcji modeli w JSON.

Klasa zasobów reprezentuje pojedynczy model, który musi zostać przekształcony w strukturę JSON. Na przykład tutaj jest prosta klasa `UserResource`:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Oczywiście jest to tylko najbardziej podstawowy przykład zasobu API. Laravel oferuje także wiele metod, które pomogą ci budować zasoby i kolekcje zasobów. Aby uzyskać więcej informacji, zapoznaj się z [pełną dokumentacją](/docs/{{version}}/eloquent-resources) na temat zasobów API.

### Console Command Auto-Registration

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/12) dla tej funkcji dostępnej na Laracastach.

Podczas tworzenia nowych poleceń konsoli nie trzeba już ręcznie je umieszczać we właściwości `$commands` jądra konsoli. Zamiast tego, nowa metoda `load` jest wywoływana z metody `commands` twojego jądra, która skanuje dany katalog pod kątem dowolnych poleceń konsoli i rejestruje je automatycznie:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        // ...
    }

### New Frontend Presets

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/4) dla tej funkcji dostępnej na Laracastach.

Chociaż podstawowe rusztowanie Vue jest nadal zawarte w Laravel 5.5, dostępnych jest teraz kilka nowych opcji ustawień frontendu. W świeżej aplikacji Laravel możesz zamienić rusztowanie Vue na rusztowanie React za pomocą polecenia `preset`:

    php artisan preset react

Możesz także całkowicie usunąć szkielet framework-a JavaScript i CSS, używając ustawienia "none". To ustawienie pozostawi Twoją aplikację prostym plikiem Sass i kilkoma prostymi narzędziami JavaScript:

    php artisan preset none

> {note} Te polecenia mają być uruchamiane tylko w nowych instalacjach Laravel. Nie powinny być używane w istniejących aplikacjach.

### Queued Job Chaining

Łańcuch zadań umożliwia określenie listy oczekujących zadań, które powinny być uruchamiane kolejno. Jeśli jedno zadanie w sekwencji nie powiedzie się, pozostałe zadania nie zostaną uruchomione. Aby wykonać oczekujący łańcuch zadań, możesz użyć metody `withChain` dla dowolnych swoich zadań do wysłania:

    ProvisionServer::withChain([
        new InstallNginx,
        new InstallPhp
    ])->dispatch();

### Queued Job Rate Limiting

Jeśli twoja aplikacja współdziała z Redis, możesz teraz dezaktywować swoje zadania w kolejce według czasu lub współbieżności. Ta funkcja może być pomocna, gdy zadania w kolejce wchodzą w interakcje z interfejsami API, które są również ograniczone szybkością. Na przykład możesz dławić dany typ zadania, aby uruchamiać go tylko 10 razy co 60 sekund:

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} W powyższym przykładzie `klucz` może być dowolnym ciągiem, który jednoznacznie identyfikuje typ zadania, dla którego chcesz wyznaczyć limit. Na przykład możesz chcieć skonstruować klucz na podstawie nazwy klasy zadania i identyfikatorów modeli wymownych, na których on działa.

Alternatywnie możesz określić maksymalną liczbę pracowników, którzy mogą jednocześnie przetwarzać dane zadanie. Może to być pomocne, gdy zadanie w kolejce modyfikuje zasób, który powinien być modyfikowany tylko przez jedno zadanie naraz. Na przykład możemy ograniczyć zadania danego typu, które będą przetwarzane tylko przez jednego pracownika naraz:

    Redis::funnel('key')->limit(1)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

### Time Based Job Attempts

Jako alternatywę do określenia, ile razy można wykonać zadanie, zanim się nie powiedzie, możesz teraz zdefiniować czas, w którym zadanie powinno upłynąć. Pozwala to na podjęcie próby dowolną liczbę razy w określonym przedziale czasowym. Aby zdefiniować czas, w którym zadanie powinno mieć limit czasu, dodaj do swojej klasy zadania metodę `retryUntil`:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} Możesz także zdefiniować metodę `retryUntil` na detektorach zdarzeń w kolejce.

### Validation Rule Objects

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/7) dla tej funkcji dostępnej na Laracastach.

Obiekty reguł walidacji zapewniają nowy, zwarty sposób dodawania niestandardowych reguł sprawdzania poprawności do aplikacji. W poprzednich wersjach Laravel zastosowano metodę `Validator::extend` w celu dodania niestandardowych reguł sprawdzania poprawności poprzez Closures. Jednak może to być uciążliwe. W Laravel 5.5 nowa instrukcja `make:rule` Artisan wygeneruje nową regułę walidacji w katalogu` app/Rules`:

    php artisan make:rule ValidName

Obiekt reguły ma tylko dwie metody: `passses` i `message`. Metoda `pass` przyjmuje wartość atrybutu i nazwę i powinna zwracać wartość `true` lub `false` w zależności od tego, czy wartość atrybutu jest poprawna czy nie. Metoda `message` powinna zwrócić komunikat o błędzie sprawdzania poprawności, który powinien zostać użyty podczas niepowodzenia sprawdzania poprawności:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class ValidName implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strlen($value) === 6;
        }

        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The name must be six characters long.';
        }
    }

Po zdefiniowaniu reguły możesz jej użyć, po prostu przekazując instancję obiektu reguły wraz z innymi regułami sprawdzania poprawności:

    use App\Rules\ValidName;

    $request->validate([
        'name' => ['required', new ValidName],
    ]);

### Trusted Proxy Integration

Podczas uruchamiania aplikacji za równoważnikiem obciążenia, który kończy certyfikaty TLS / SSL, możesz zauważyć, że aplikacja czasami nie generuje łączy HTTPS. Zazwyczaj dzieje się tak dlatego, że Twoja aplikacja jest przekierowywana z systemu równoważenia obciążenia na porcie 80 i nie wie, że powinna generować bezpieczne linki.

Aby rozwiązać ten problem, wielu użytkowników Laravel instaluje pakiet [Trusted Proxies](https://github.com/fideloper/TrustedProxy) autorstwa Chrisa Fidao. Ponieważ jest to tak powszechny przypadek użycia, pakiet Chrisa domyślnie jest dostarczany z Laravel 5.5.

Nowe oprogramowanie pośredniczące `App\Http\Middleware\TrustProxies` jest zawarte w domyślnej aplikacji Laravel 5.5. To oprogramowanie pośrednie pozwala szybko dostosować serwery proxy, które powinny być zaufane przez aplikację:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var array
         */
        protected $proxies;

        /**
         * The current proxy header mappings.
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }

### On-Demand Notifications

Czasami może być konieczne wysłanie powiadomienia do osoby, która nie jest zapisana jako "użytkownik" aplikacji. Korzystając z nowej metody `Notification::route`, możesz określić dane routingu powiadomienia ad-hoc przed wysłaniem powiadomienia:

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->send(new InvoicePaid($invoice));

### Renderable Mailables

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/6) dla tej funkcji dostępnej na Laracastach.

Mailables mogą teraz zostać zwrócone bezpośrednio z tras, co pozwala na szybki podgląd projektów pocztowych w przeglądarce:

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

### Renderable & Reportable Exceptions

> {video} Jest bezpłatny [samouczek wideo] https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/18) dla tej funkcji dostępnej na Laracastach.

W poprzednich wersjach Laravel, być może trzeba było użyć "sprawdzania typów" w swojej procedurze obsługi wyjątków, aby wygenerować niestandardową odpowiedź dla danego wyjątku. Na przykład, możesz napisać taki kod w swojej metodzie `render` wyjątku:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof SpecialException) {
            return response(...);
        }

        return parent::render($request, $exception);
    }

W Laravel 5.5 możesz teraz zdefiniować metodę `render` bezpośrednio na swoich wyjątkach. Umożliwia to umieszczenie logiki niestandardowego renderowania odpowiedzi bezpośrednio na wyjątku, co pomaga uniknąć warunkowego gromadzenia logiki w obsłudze wyjątków. Jeśli chcesz również dostosować logikę raportowania dla wyjątku, możesz zdefiniować metodę `report` w klasie:

    <?php

    namespace App\Exceptions;

    use Exception;

    class SpecialException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * Report the exception.
         *
         * @param  \Illuminate\Http\Request
         * @return void
         */
        public function render($request)
        {
            return response(...);
        }
    }

### Request Validation

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/2) dla tej funkcji dostępnej na Laracastach.

Obiekt `Illuminate\Http\Request` udostępnia teraz metodę `validate`, która umożliwia szybkie sprawdzenie przychodzącego żądania z Closure trasy lub kontrolera:

    use Illuminate\Http\Request;

    Route::get('/comment', function (Request $request) {
        $request->validate([
            'title' => 'required|string',
            'body' => 'required|string',
        ]);

        // ...
    });

### Consistent Exception Handling

Obsługa wyjątków walidacyjnych jest teraz spójna w całym systemie. Poprzednio w strukturze istniało wiele lokalizacji wymagających dostosowywania w celu zmiany domyślnego formatu odpowiedzi na błędy sprawdzania poprawności JSON. Ponadto domyślny format odpowiedzi sprawdzania poprawności JSON w Laravel 5.5 jest teraz zgodny z następującą konwencją:

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

Wszystkie formatowanie błędów sprawdzania poprawności JSON można kontrolować, definiując pojedynczą metodę w klasie `App\Exceptions\Handler`. Na przykład poniższa personalizacja sformatuje odpowiedzi sprawdzania poprawności JSON przy użyciu konwencji Laravel 5.5.

    use Illuminate\Validation\ValidationException;

    /**
     * Convert a validation exception into a JSON response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

### Cache Locks

Sterowniki pamięci podręcznej Redis i Memcached mają teraz obsługę uzyskiwania i zwalniania atomowych "blokad". Zapewnia to prostą metodę uzyskiwania arbitralnych blokad bez martwienia się o warunki wyścigu. Na przykład przed wykonaniem zadania możesz chcieć uzyskać blokadę, aby żaden inny proces nie próbował wykonać tego samego zadania, które jest już w toku:

    if (Cache::lock('lock-name', 60)->get()) {
        // Lock obtained for 60 seconds, continue processing...

        Cache::lock('lock-name')->release();
    } else {
        // Lock was not able to be obtained...
    }

Lub możesz przekazać Closure do metody `get`. Closure zostanie wykonane tylko wtedy, gdy można uzyskać lok, a blokada zostanie automatycznie zwolniona po wykonaniu Closure:

    Cache::lock('lock-name', 60)->get(function () {
        // Lock obtained for 60 seconds...
    });

Ponadto możesz "zablokować", aż blokada stanie się dostępna:

    if (Cache::lock('lock-name', 60)->block(10)) {
        // Wait for a maximum of 10 seconds for the lock to become available...
    }

### Blade Improvements

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/10) dla tej funkcji dostępnej na Laracastach.

Programowanie niestandardowej dyrektywy jest czasami bardziej złożone niż to konieczne podczas definiowania prostych, niestandardowych instrukcji warunkowych. Z tego powodu Blade oferuje teraz metodę `Blade::if`, która pozwala szybko zdefiniować niestandardowe dyrektywy warunkowe za pomocą Closures. Na przykład, zdefiniujmy niestandardowy warunek, który sprawdza bieżące środowisko aplikacji. Możemy to zrobić w metodzie `boot` naszego `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

Po zdefiniowaniu warunku niestandardowego możemy z łatwością użyć go w naszych szablonach:

    @env('local')
        // The application is in the local environment...
    @else
        // The application is not in the local environment...
    @endenv

Poza możliwością łatwego definiowania niestandardowych dyrektyw warunkowych Blade, dodano nowe skróty, aby szybko sprawdzić status uwierzytelniania bieżącego użytkownika:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

### New Routing Methods

> {video} Jest bezpłatny [samouczek wideo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/16) dla tej funkcji dostępnej na Laracastach.

Jeśli definiujesz trasę, która przekierowuje do innego URI, możesz teraz użyć metody `Route::redirect`. Ta metoda zapewnia wygodny skrót, dzięki czemu nie trzeba definiować pełnej trasy ani kontrolera do wykonywania prostego przekierowania:

    Route::redirect('/here', '/there', 301);

Jeśli twoja trasa musi tylko zwrócić widok, możesz teraz użyć metody `Route::view`. Podobnie jak w przypadku metody `redirect`, ta metoda zapewnia prosty skrót, dzięki czemu nie trzeba definiować pełnej trasy ani kontrolera. Metoda `view` akceptuje URI jako swój pierwszy argument, a nazwę widoku jako swój drugi argument. Ponadto możesz podać tablicę danych do przekazania do widoku jako opcjonalnego trzeciego argumentu:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

### "Sticky" Database Connections

#### The `sticky` Option

Podczas konfigurowania połączeń do odczytu i zapisu bazy danych dostępna jest nowa opcja konfiguracji `sticky`:

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

Opcja `sticky` jest *opcjonalną* wartością, która może być użyta do natychmiastowego odczytu zapisów, które zostały zapisane w bazie danych podczas bieżącego cyklu żądania. Jeśli włączona jest opcja `sticky` i operacja "write" została przeprowadzona względem bazy danych podczas bieżącego cyklu żądania, wszelkie dalsze operacje "read" będą korzystać z połączenia "write". Zapewnia to, że wszelkie dane zapisane podczas cyklu żądania mogą być natychmiast odczytywane z bazy danych podczas tego samego żądania. To ty decydujesz, czy jest to pożądane zachowanie dla twojej aplikacji.
