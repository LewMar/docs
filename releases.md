# Release Notes

- [Versioning Scheme - Schemat wersjonowania](#versioning-scheme)
- [Support Policy - Zasady pomocy technicznej](#support-policy)
- [Laravel 5.6](#laravel-5.6)

<a name="versioning-scheme"></a>
## Versioning Scheme - Schemat wersjonowania

Schemat wersjonowania Laravel utrzymuje następującą konwencję: `paradigm.major.minor`. Ważniejsze (major) wydania framework-a są wydawane co sześć miesięcy (luty i sierpień), podczas gdy niewielkie wydania mogą być publikowane tak często, jak co tydzień. Drobne wydania powinny **nigdy** nie zawierać zmian łamania.

Podczas odwoływania się do frameworka Laravel lub jego komponentów z aplikacji lub pakietu, zawsze powinieneś używać ograniczeń wersji, takich jak `5.5.*`, Ponieważ główne wersje Laravel zawierają zmiany zrywające. Staramy się jednak zawsze zapewniać aktualizację do nowej wersji głównej w ciągu jednego dnia lub mniej.

Przesunięcia paradygmatu są rozdzielane przez wiele lat i stanowią fundamentalne przesunięcia w architekturze i konwencjach architektury. Obecnie nie ma opracowywanej zmiany paradygmatu.

<a name="support-policy"></a>
## Support Policy - Zasady pomocy technicznej

W przypadku wydań LTS, takich jak Laravel 5.5, poprawki są dostarczane przez 2 lata, a poprawki bezpieczeństwa są dostarczane przez 3 lata. Te wydania zapewniają najdłuższe okno obsługi i konserwacji. W przypadku wersji ogólnych poprawki są dostarczane przez 6 miesięcy, a poprawki bezpieczeństwa są dostarczane przez 1 rok.

<a name="laravel-5.6"></a>
## Laravel 5.6

Laravel 5.6 kontynuuje ulepszenia wprowadzone w Laravel 5.5, dodając ulepszony system rejestrowania, planowanie zadań na pojedynczym serwerze, ulepszenia serializacji modeli, dynamiczne ograniczanie szybkości, klasy kanałów rozgłaszania, generowanie kontrolera zasobów API, wymyślne ulepszanie formatowania daty, aliasy komponentów Blade, Argon2 obsługa hashowania hasła, włączenie pakietu Collision i wiele więcej. Ponadto wszystkie rusztowania frontowe zostały zaktualizowane do Bootstrap 4.

Wszystkie podstawowe składniki Symfony używane przez Laravel zostały uaktualnione do serii wydań Symfony `~ 4.0`.

Wydanie Laravel 5.6 zbiega się z wydaniem [Spark 6.0](https://spark.laravel.com), pierwszego dużego uaktualnienia Laravel Spark od czasu jego wydania. Spark 6.0 wprowadza ceny poszczególnych pasowań dla obsługi Stripe i Braintree, lokalizacji, Bootstrap 4, ulepszonego interfejsu użytkownika i elementów paskowych.

> {tip} W tej dokumentacji podsumowano najważniejsze ulepszenia w framework-u; jednak bardziej szczegółowe dzienniki zmian są zawsze dostępne [na GitHubie](https://github.com/laravel/framework/blob/5.6/CHANGELOG-5.6.md).

### Logging Improvements - Udoskonalenia w dziennikach

Laravel 5.6 wprowadza ogromne ulepszenia w systemie logowania Laravel. Cała konfiguracja rejestrowania znajduje się w nowym pliku konfiguracyjnym `config/logging.php`. Możesz teraz łatwo budować "stosy" dziennika, które wysyłają wiadomości do wielu programów obsługi. Na przykład możesz wysyłać wszystkie komunikaty poziomu `debug` do dziennika systemu, wysyłając komunikaty o poziomie `error` do Slacka, aby twój zespół mógł szybko reagować na błędy:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],
    ],

Ponadto teraz łatwiej jest dostosować istniejące kanały dziennika przy użyciu nowej funkcjonalności systemu "tap". Aby uzyskać więcej informacji, zapoznaj się z [pełną dokumentacją dotyczącą rejestrowania](/docs/{{version}}/logging).

### Single Server Task Scheduling - Harmonogram zadań pojedynczego serwera

> {note} Aby móc korzystać z tej funkcji, aplikacja musi używać sterownika pamięci podręcznej `memcached` lub `redis` jako domyślnego sterownika pamięci podręcznej aplikacji. Ponadto wszystkie serwery muszą komunikować się z tym samym centralnym serwerem pamięci podręcznej.

Jeśli twoja aplikacja działa na wielu serwerach, możesz teraz ograniczyć zaplanowane zadanie do wykonania tylko na jednym serwerze. Załóżmy na przykład, że masz zaplanowane zadanie, które generuje nowy raport w każdy piątek wieczorem. Jeśli program do planowania zadań działa na trzech serwerach roboczych, zaplanowane zadanie zostanie uruchomione na wszystkich trzech serwerach i wygeneruje raport trzy razy. Niedobrze!

Aby wskazać, że zadanie powinno działać tylko na jednym serwerze, podczas definiowania zaplanowanego zadania można użyć metody `onOneServer`. Pierwszy serwer, który uzyska zadanie, zabezpieczy blokadę atomową przed zadaniem, aby uniemożliwić innym serwerom wykonywanie tego samego zadania w tym samym cyklu Cron:

    $schedule->command('report:generate')
             ->fridays()
             ->at('17:00')
             ->onOneServer();

### Dynamic Rate Limiting - Ograniczanie szybkości dynamicznej

Podczas określania [ograniczenia tempa](/docs/{{version}}/routing#rate-limiting) na grupie tras w poprzednich wersjach Laravel, byłeś zmuszony do podania zakodowanej liczby maksymalnych żądań:

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

W Laravel 5.6 możesz określić maksimum dynamiczne na podstawie atrybutu uwierzytelnionego modelu `User`. Na przykład, jeśli twój model `User` zawiera atrybut `rate_limit`, możesz przekazać nazwę tego atrybutu do oprogramowania pośredniczącego `throttle`, aby użyć go do obliczenia maksymalnej liczby zapytań:

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

### Broadcast Channel Classes - Nadawane klasy kanałów

Jeśli twoja aplikacja zużywa wiele różnych kanałów, twój plik `routes/channels.php` może stać się nieporęczny. Zamiast korzystać z opcji Closure, aby autoryzować kanały, możesz teraz używać klas kanałów. Aby wygenerować klasę kanału, użyj polecenia `make:channel` Artisan. To polecenie umieści nową klasę kanałów w katalogu `App/Broadcasting`.

    php artisan make:channel OrderChannel

Następnie zarejestruj swój kanał w pliku `routes/channels.php`:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

Na koniec możesz umieścić logikę autoryzacji dla swojego kanału w metodzie kanału `join`. Ta metoda `join` będzie zawierała tę samą logikę, którą zwykle umieszczasz w swoim closure autoryzacji kanału. Oczywiście możesz również skorzystać z wiązania modelu kanału:

    <?php

    namespace App\Broadcasting;

    use App\User;
    use App\Order;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Authenticate the user's access to the channel.
         *
         * @param  \App\User  $user
         * @param  \App\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

### API Controller Generation - Generowanie kontrolera API

Podczas deklarowania tras zasobów, które będą używane przez interfejsy API, zwykle chcesz wykluczyć trasy, które przedstawiają szablony HTML, takie jak `create` i `edit`. Aby wygenerować kontroler zasobów, który nie zawiera tych metod, możesz teraz użyć przełącznika `--api` podczas wykonywania polecenia `make:controller`:

    php artisan make:controller API/PhotoController --api

### Model Serialization Improvements - Ulepszenie seriaizowanego modelu

W poprzednich wersjach Laravel, kolejkowane modele nie byłyby przywracane z nienaruszonymi obciążonymi relacjami. W Laravel 5.6 relacje, które zostały załadowane do modelu podczas jego kolejkowania, są automatycznie ponownie ładowane, gdy zadanie jest przetwarzane przez kolejkę.

### Eloquent Date Casting - Wymowne odlewanie daty

Możesz teraz indywidualnie dostosować format kolumn Eroquent Date Cast. Aby rozpocząć, określ żądany format daty w deklaracji dotyczącej przesyłania. Po określeniu ten format będzie używany podczas serializacji modelu do tablicy / JSON:

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];

### Blade Component Aliases

Jeśli komponenty Blade są przechowywane w podkatalogu, możesz teraz je aliasować w celu łatwiejszego dostępu. Na przykład wyobraź sobie komponent Blade przechowywany w `resources/views/components/alert.blade.php`. Możesz użyć metody `component` do aliasowania komponentu z `components.alert` do `alert`:

    Blade::component('components.alert', 'alert');

Po aliasowaniu komponentu możesz renderować go za pomocą dyrektywy:

    @alert('alert', ['type' => 'danger'])
        You are not allowed to access this resource!
    @endalert

Możesz pominąć parametry komponentu, jeśli nie ma żadnych dodatkowych gniazd:

    @alert
        You are not allowed to access this resource!
    @endalert

### Argon2 Password Hashing

Jeśli budujesz aplikację na PHP 7.2.0 lub nowszym, Laravel obsługuje teraz haszowanie hasła za pomocą algorytmu Argon2. Domyślny sterownik skrótu dla twojej aplikacji jest kontrolowany przez nowy plik konfiguracyjny `config/hashing.php`.

### UUID Methods - Metody UUID

Laravel 5.6 wprowadza dwie nowe metody generowania UUID: `Str::uuid` i `Str::orderedUuid`. Metoda `orderedUuid` wygeneruje pierwszy znacznik czasu UUID, który jest łatwiej i wydajniej indeksowany przez bazy danych, takie jak MySQL. Każda z tych metod zwraca obiekt `Ramsey\Uuid`:

    use Illuminate\Support\Str;

    return (string) Str::uuid();

    return (string) Str::orderedUuid();

### Collision - Kolizja

Domyślna aplikacja `laravel/laravel` zawiera teraz zależność `dev` Composer dla pakietu [Collision](https://github.com/nunomaduro/collision) obsługiwanego przez Nuno Maduro. Pakiety te zapewniają piękne zgłaszanie błędów podczas interakcji z aplikacją Laravel w wierszu poleceń:

<img src="https://raw.githubusercontent.com/nunomaduro/collision/stable/docs/example.png" width="600" height="388">

### Bootstrap 4

Wszystkie rusztowania frontowe, takie jak zestaw do uwierzytelniania i przykładowy komponent Vue, zostały uaktualnione do [Bootstrap 4](https://blog.getbootstrap.com/2018/01/18/bootstrap-4/). Domyślnie generowanie linków stronicowania domyślnie przyjmuje teraz wartość Bootstrap 4.