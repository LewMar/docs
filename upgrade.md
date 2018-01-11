# Upgrade Guide

- [Upgrading To 5.5.0 From 5.4 - Aktualizacja do wersji 5.5.0 Od 5.4](#upgrade-5.5.0)

<a name="upgrade-5.5.0"></a>
## Upgrading To 5.5.0 From 5.4 - Aktualizacja do wersji 5.5.0 Od 5.4

#### Estimated Upgrade Time: 1 Hour - Szacowany czas aktualizacji: 1 godzina

> {note} Staramy się udokumentować każdą możliwą zmianę przełomową. Ponieważ niektóre z tych przełomowych zmian są w mrocznych częściach struktury, tylko część tych zmian może wpłynąć na twoją aplikację.

### PHP

Laravel 5.5 wymaga PHP 7.0.0 lub nowszego.

### Updating Dependencies - Aktualizacja zależności

Zaktualizuj zależność `laravel/framework` do `5.5.*`. W pliku `composer.json`. Ponadto powinieneś zaktualizować swoją zależność `phpunit/phpunit` na `~ 6.0`. Następnie dodaj pakiet `filp/whoops` z wersją `~2.0` do sekcji `require-dev` w pliku `composer.json`. Na koniec, w sekcji `scripts` pliku `composer.json` dodaj polecenie `package:discover` do zdarzenia `post-autoload-dump`:

    "scripts": {
        ...
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ],
    }

Oczywiście, nie zapomnij sprawdzić pakietów zewnętrznych używanych przez twoją aplikację i sprawdź, czy korzystasz z właściwej wersji dla obsługi Laravel 5.5.

#### Laravel Installer - Instalator Laravel

> {tip} Jeśli często używasz instalatora Laravel przez `laravel new`, powinieneś zaktualizować swój pakiet instalacyjny Laravel za pomocą komendy `composer global update`.


#### Laravel Dusk

Laravel Dusk `2.0.0` został wydany w celu zapewnienia zgodności z Laravel 5.5 i bezgłowymi testami Chrome.

#### Pusher

Sterownik rozgłaszania zdarzeń Pusher wymaga teraz wersji `~3.0` z Pusher SDK.

#### Swift Mailer

Laravel 5.5 wymaga wersji `~6.0` Swift Mailera.

### Artisan

#### Auto-Loading Commands - Automatyczne ładowanie poleceń

W Laravel 5.5, Artisan może automatycznie wykrywać polecenia, dzięki czemu nie musisz ręcznie rejestrować ich w jądrze. Aby skorzystać z tej nowej funkcji, dodaj następujący wiersz do metody `commands` klasy `App\Console\Kernel`:

    $this->load(__DIR__.'/Commands');

#### The `fire` Method - Metoda `fire`

Wszelkie metody `fire` obecne w twoich komendach Artisan powinny zostać przemianowane na `handle`.

#### The `optimize` Command - Komenda `optimize`

Dzięki ostatnim ulepszeniom w buforowaniu kodu op-code PHP, polecenie `optimize` Artisan nie jest już potrzebne. Powinieneś usunąć wszelkie odwołania do tego polecenia ze skryptów wdrażania, ponieważ zostaną one usunięte w przyszłej wersji Laravel.

### Authorization - Upoważnienie

#### The `authorizeResource` Controller Method - Metoda kontrolera `authorizeResource`

Podczas przekazywania nazwy słowa składającego się z wielu wyrazów do metody `authorizeResource` wynikowy segment trasy będzie teraz wyglądał jak "wąż ", dopasowując zachowanie kontrolerów zasobów.

#### The `before` Policy Method

Metoda `before` klasy polityki nie zostanie wywołana, jeśli klasa nie zawiera metody pasującej do nazwy sprawdzanej umiejętności.

### Cache

#### Database Driver -Sterownik Bazy danych

Jeśli używasz sterownika pamięci podręcznej bazy danych, podczas instalowania uaktualnionej aplikacji Laravel 5.5 po raz pierwszy powinieneś uruchomić `php artisan cache:clear`.

### Eloquent

#### The `belongsToMany` Method - Metoda `belongsToMany`

Jeśli nadpisujesz metodę `belongsToMany` w swoim modelu Wymownym, powinieneś zaktualizować swóje oznaczenie metody, dodając nowe argumenty:

    /**
     * Define a many-to-many relationship.
     *
     * @param  string  $related
     * @param  string  $table
     * @param  string  $foreignPivotKey
     * @param  string  $relatedPivotKey
     * @param  string  $parentKey
     * @param  string  $relatedKey
     * @param  string  $relation
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function belongsToMany($related, $table = null, $foreignPivotKey = null,
                                  $relatedPivotKey = null, $parentKey = null,
                                  $relatedKey = null, $relation = null)
    {
        //
    }

#### BelongsToMany `getQualifiedRelatedKeyName`

Nazwa metody `getQualifiedRelatedKeyName` została zmieniona na `getQualifiedRelatedPivotKeyName`.

#### BelongsToMany `getQualifiedForeignKeyName`

The `getQualifiedForeignKeyName` method has been renamed to `getQualifiedForeignPivotKeyName`.
Metoda `getQualifiedForeignKeyName` została zmieniona na `getQualifiedForeignPivotKeyName`.

#### Model `is` Method - Metoda `is` Model-u

Jeśli nadpisujesz metodę `is` swojego modelu Eloquent, powinieneś usunąć metodę typu `Model` z metody. Dzięki temu metoda `is` może odbierać `null` jako argument:

    /**
     * Determine if two models have the same ID and belong to the same table.
     *
     * @param  \Illuminate\Database\Eloquent\Model|null  $model
     * @return bool
     */
    public function is($model)
    {
        //
    }

#### Model `$events` Property - Właściwość Model-u `$events`

Właściwość `$events` w modelach powinna mieć zmienioną nazwę na `$dispatchesEvents`. Ta zmiana została dokonana ze względu na dużą liczbę użytkowników, którzy musieli zdefiniować relację `events`, która spowodowała konflikt ze starą nazwą właściwości.

#### Pivot `$parent` Property - Właściwości Pivot `$parent`

Chroniona własność `$parent` w klasie `Illuminate\Database\Eloquent\Relations\Pivot` została zmieniona na `$pivotParent`.

#### Relationship `create` Methods - Metody relacji `create`

Metody `create` klas `BelongsToMany`, `HasOneOrMany` i `MorphOneOrMany` zostały zmodyfikowane, aby zapewnić domyślną wartość argumentu `$attributes`. Jeśli nadpisujesz te metody, zaktualizuj swoje podpisy tak, aby pasowały do nowej definicji:

    public function create(array $attributes = [])
    {
        //
    }

#### Soft Deleted Models - Miękie usuwanie modeli

Podczas usuwania modelu "miękkiego usunięcia" właściwość `exists` w modelu pozostanie `true`.

#### `withCount` Column Formatting - Formatowanie kolumny `withCount`

Gdy używasz aliasu, metoda `withCount` nie będzie automatycznie dodawać `_count` do wynikowej nazwy kolumny. Na przykład w Laravel 5.4, następujące zapytanie spowoduje dodanie kolumny `bar_count` do zapytania:

    $users = User::withCount('foo as bar')->get();

Jednak w Laravel 5.5, alias będzie używany dokładnie tak, jak jest podany. Jeśli chcesz wstawić `_count` do wynikowej kolumny, musisz określić ten sufiks podczas definiowania aliasu:

    $users = User::withCount('foo as bar_count')->get();

#### Model Methods & Attribute Names - Metody modelów i nazwy atrybutów

Aby uniemożliwić dostęp do prywatnych właściwości modelu podczas korzystania z dostępu do tablicy, nie można już stosować metody modelu o tej samej nazwie, co atrybut lub właściwość. Spowoduje to wywołanie wyjątków podczas uzyskiwania dostępu do atrybutów modelu za pośrednictwem dostępu do tablicy (`$user['name']`) lub funkcji pomocniczej `data_get`.

### Exception Format - Format wyjątków

W Laravel 5.5 wszystkie wyjątki, w tym wyjątki sprawdzania poprawności, są konwertowane na odpowiedzi HTTP przez procedurę obsługi wyjątku. Skonsultować się z JSON. Nowy format jest zgodny z następującą konwencją:

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

Jeśli jednak chcesz zachować format błędu Laravel 5.4 JSON, możesz dodać następującą metodę do klasy `App\Exceptions\Handler`:

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

#### JSON Authentication Attempts - Próby uwierzytelnienia JSON

Ta zmiana wpływa również na formatowanie błędów sprawdzania poprawności dla prób uwierzytelnienia wykonanych za pomocą JSON. W Laravel 5.5, błędy uwierzytelniania JSON zwrócą komunikaty o błędach zgodnie z nową konwencją formatowania opisaną powyżej.

#### A Note On Form Requests - Uwaga dotycząca żądań formularzy

Jeśli dostosowywałeś format odpowiedzi pojedynczego żądania formularza, powinieneś teraz zastąpić metodę `failedValidation` tego żądania formularza i rzucić instancję `HttpResponseException` zawierającą twoją niestandardową odpowiedź:

    use Illuminate\Http\Exceptions\HttpResponseException;

    /**
     * Handle a failed validation attempt.
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return void
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json(..., 422));
    }

### Filesystem

#### The `files` Method

Metoda `files` klasy `Illuminate\Filesystem\Filesystem` zmieniła swoje oznaczenie , dodając argument `$hidden`, a teraz zwraca tablicę obiektów `SplFileInfo`, podobnie jak metoda `allFiles`. Poprzednio metoda `files` zwróciła tablicę nazw ścieżek tekstowych. Nowe oznaczenie jest następujące:

    public function files($directory, $hidden = false)

### Mail

#### Unused Parameters - Niewykorzystane parametry

Nieużywane argumenty `$data` i` $callback` zostały usunięte z metod `queue` i ` later` kontraktu `Illuminate\Contracts\Mail\MailQueue`:

    /**
     * Queue a new e-mail message for sending.
     *
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function queue($view, $queue = null);

    /**
     * Queue a new e-mail message for sending after (n) seconds.
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function later($delay, $view, $queue = null);

### Queues

#### The `dispatch` Helper - Pomocnik `dispatch`

Jeśli chcesz wysłać zadanie, które działa natychmiast i zwraca wartość z metody `handle`, powinieneś użyć metody `dispatch_now` lub `Bus::dispatchNow` do wysłania zadania:

    use Illuminate\Support\Facades\Bus;

    $value = dispatch_now(new Job);

    $value = Bus::dispatchNow(new Job);

### Requests

#### The `all` Method - Metoda `all`

Jeśli nadpisujesz metodę `all` klasy `Illuminate\Http\Request`, powinieneś zaktualizować swoje oznaczenie metody, aby odzwierciedlić nowy argument `$keys`:

    /**
     * Get all of the input and files for the request.
     *
     * @param  array|mixed  $keys
     * @return array
     */
    public function all($keys = null)
    {
        //
    }

#### The `has` Method - Metoda `has`

Metoda `$request->has` zwraca teraz wartość `true`, nawet jeśli wartość wejściowa jest pustym łańcuchem lub `null`. Dodano nową metodę `$request->filled`, która zapewnia poprzednie zachowanie metody `has`.

#### The `intersect` Method - Metoda `intersect`

Metoda `intersect` została usunięta. Możesz replikować to zachowanie używając `array_filter` w wywołaniu `$request->only`:


    return array_filter($request->only('foo'));

#### The `only` Method - Metoda `only`

Metoda `only` zwróci teraz tylko te atrybuty, które są rzeczywiście obecne w ładunku żądania. Jeśli chcesz zachować stare zachowanie metody `only`, możesz zamiast tego użyć metody `all`.

    return $request->all('foo');

#### The `request()` Helper - Pomocnik `request()`

Pomocnik `request` nie będzie już odzyskiwał zagnieżdżonych kluczy. W razie potrzeby możesz użyć metody `input` żądania, aby osiągnąć to zachowanie:

    return request()->input('filters.date');

### Testing - Testowanie

#### Authentication Assertions - Asercje uwierzytelniania

Niektóre asercje uwierzytelniające zostały zmienione pod kątem lepszej spójności z pozostałymi elementami asercji:

<div class="content-list" markdown="1">
- `seeIsAuthenticated` zmienił nazwę na `assertAuthenticated`.
- `dontSeeIsAuthenticated` zmienił nazwę na `assertGuest`.
- `seeIsAuthenticatedAs` zmienił nazwę na `assertAuthenticatedAs`.
- `seeCredentials` zmienił nazwę na `assertCredentials`.
- `dontSeeCredentials` zmienił nazwę na `assertInvalidCredentials`.
</div>

#### Mail Fake

Jeśli używasz fałszywki `Mail` w celu ustalenia, czy program pocztowy **został umieszczony w kolejce** podczas żądania, powinieneś teraz użyć `Mail::assertQueued` zamiast `Mail::assertSent`. To rozróżnienie pozwala wyraźnie stwierdzić, że poczta była w kolejce do wysłania w tle i nie została wysłana podczas samego żądania.

#### Tinker

Laravel Tinker obsługuje teraz pomijanie obszarów nazw podczas odwoływania się do klas aplikacji. Ta funkcja wymaga zoptymalizowanej mapy klasy Composer, więc powinieneś dodać dyrektywę `optimize-autoloader` do sekcji `config` pliku `composer.json`:

    "config": {
        ...
        "optimize-autoloader": true
    }

### Translation - Tłumaczenie

#### The `LoaderInterface`

`Illuminate\Translation\LoaderInterface` interfejs został przeniesiony do `Illuminate\Contracts\Translation\Loader`.

### Validation - Uprawomocnienie

#### Validator Methods - Sprawdzanie poprawności

Wszystkie metody sprawdzania poprawności są teraz `public` zamiast `protected`.

### Views - Widoki

#### Dynamic "With" Variable Names - Dynamiczna nazwa zmiennej "With"

When allowing the dynamic `__call` method to share variables with a view, these variables will automatically use "camel" case. For example, given the following:
Po zezwoleniu dynamicznej metodzie `__call` na współdzielenie zmiennych z widokiem, zmienne te będą automatycznie używać "camel" case. Na przykład, biorąc pod uwagę:

    return view('pool')->withMaximumVotes(100);

Zmienna `maximumVotes` może być dostępna w szablonie w następujący sposób:

    {{ $maximumVotes }}

#### `@php` Blade Directive - Dyrektywa `@php` w blade

Dyrektywa blade `@php` nie przyjmuje już tagów wbudowanych. Zamiast tego użyj pełnej wersji dyrektywy:

    @php
        $teamMember = true;
    @endphp

### Miscellaneous - Różne

Zachęcamy również do przeglądania zmian w `laravel/laravel` [repozytorium GitHub](https://github.com/laravel/laravel). Chociaż wiele z tych zmian nie jest wymaganych, możesz chcieć synchronizować te pliki z aplikacją. Niektóre z tych zmian zostaną omówione w tym przewodniku aktualizacji, ale inne, takie jak zmiany w plikach konfiguracyjnych lub komentarzach, nie będą. Możesz łatwo przeglądać zmiany za pomocą [narzędzia porównywania GitHub](https://github.com/laravel/laravel/compare/5.4...master) i wybrać, które aktualizacje są dla Ciebie ważne.
