# Upgrade Guide

- [Upgrading To 5.6.0 From 5.5 - Aktualizacja do wersji 5.6.0 z 5.5](#upgrade-5.6.0)

<a name="upgrade-5.6.0"></a>
## Upgrading To 5.6.0 From 5.5 - Aktualizacja do wersji 5.6.0 z 5.5

#### Szacowany czas aktualizacji: 10 - 30 Minut

> {note} Staramy się udokumentować każdą możliwą zmianę przełomową. Ponieważ niektóre z tych przełomowych zmian są w mrocznych częściach struktury, tylko część tych zmian może wpłynąć na twoją aplikację.

### PHP

Laravel 5.6 wymaga PHP 7.1.3 lub wyszej.

### Updating Dependencies - Aktualizacja zależności

Zaktualizuj zależność `laravel/framework` do `5.6.*` i twoją zależność `fideloper/proxy` od `~ 4.0` w twoim pliku `composer.json`.

Ponadto, jeśli używasz następujących pakietów Laravel, należy uaktualnić je do najnowszej wersji:

<div class="content-list" markdown="1">
- Dusk (Upgrade To `~3.0`)
- Passport (Upgrade To `~5.0`)
- Scout (Upgrade To `~4.0`)
</div>

Oczywiście, nie zapomnij sprawdzić żadnych pakietów zewnętrznych używanych przez twoją aplikację i sprawdź, czy korzystasz z odpowiedniej wersji dla wsparcia Laravel 5.6.

### Symfony 4

Wszystkie podstawowe składniki Symfony używane przez Laravel zostały uaktualnione do serii wydań Symfony `~ 4.0`. Jeśli bezpośrednio wchodzisz w interakcję z komponentami Symfony w swojej aplikacji, powinieneś przejrzeć [Dziennik zmian w Symfony](https://github.com/symfony/symfony/blob/master/UPGRADE-4.0.md).

#### PHPUnit

Powinieneś zaktualizować zależność `phpunit/phpunit` twojej aplikacji na `~ 7.0`.

### Arrays

#### The `Arr::wrap` Method - Metoda `Arr::wrap`

Przekazanie `null` do metody `Arr::wrap` zwróci teraz pustą tablicę.

### Artisan

#### The `optimize` Command - Komenda `optimize`

Poprzednio wycofane polecenie `optimize` Artisan zostało usunięte. Dzięki ostatnim ulepszeniom samego PHP, w tym OPcache, komenda `optimize` nie zapewnia już żadnych istotnych korzyści związanych z wydajnością. Dlatego możesz usunąć `php artisan optimize` z `scripts` w swoim pliku `composer.json`.

### Blade

#### HTML Entity Encoding - Kodowanie encji HTML

W poprzednich wersjach Laravel, Blade (i helper `e`) nie będą podwójnie kodować encji HTML. Nie było to domyślne zachowanie ukrytej funkcji `htmlspecialchars` i mogłoby prowadzić do nieoczekiwanego zachowania podczas renderowania treści lub przekazywania treści JSON w linii do frameworków JavaScript.

W Laravel 5.6, Blade i pomocnik `e` domyślnie podwójnie kodują znaki specjalne. Dzięki temu te funkcje są zgodne z domyślnym zachowaniem funkcji PHP `htmlspecialchars`. Jeśli chcesz zachować poprzednie zachowanie zapobiegające podwójnemu kodowaniu, możesz użyć metody `Blade::withoutDoubleEncoding`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::withoutDoubleEncoding();
        }
    }

### Cache - Pamięć podręczna

#### The Rate Limiter `tooManyAttempts` Method - Metoda ograniczenia tempa `tooManyAttempts`

Nieużywany parametr `$decayMinutes` został usunięty z podpisu tej metody. Jeśli nadpisałeś tę metodę własną implementacją, powinieneś również usunąć argument z podpisu swojej metody.

### Database - Baza danych

#### Index Order Of Morph Columns - Porządek indeksu kolumn przemiany

Indeksowanie kolumn zbudowanych metodą migracji `morphs` zostało odwrócone dla lepszej wydajności. Jeśli używasz metody `morphs` w jednej z twoich migracji, możesz otrzymać komunikat o błędzie podczas próby uruchomienia metody `down` migracji. Jeśli aplikacja jest jeszcze w fazie rozwoju, możesz użyć polecenia `migrate:fresh`, aby odbudować bazę danych od podstaw. Jeśli aplikacja jest w produkcji, należy przekazać jawną nazwę indeksu do metody `morphs`.

#### `MigrationRepositoryInterface` Method Addition - Dodanie metody `MigrationRepositoryInterface`

Nowa metoda `getMigrationsBatches` została dodana do `MigrationRepositoryInterface`. W bardzo mało prawdopodobnym przypadku, gdy definiujesz własną implementację tej klasy, powinieneś dodać tę metodę do swojej implementacji. Możesz zobaczyć domyślną implementację w strukturze jako przykład.

### Eloquent

#### The `getDateFormat` Method - Metoda `getDateFormat`

Ta metoda `getDateFormat` jest teraz `public` zamiast `protected`.

### Hashing - Haszowanie

#### New Configuration File - Nowy plik konfiguracyjny

Cała konfiguracja hashowania znajduje się teraz we własnym pliku konfiguracyjnym `config/hashing.php`. Powinieneś umieścić kopię [domyślnego pliku konfiguracyjnego](https://github.com/laravel/laravel/blob/develop/config/hashing.php) we własnej aplikacji. Najprawdopodobniej powinieneś utrzymywać sterownik `bcrypt` jako domyślny sterownik. Jednak obsługiwany jest również `argon`.

### Helpers - Pomocnicy

#### The `e` Helper - Pomocnik `e`

W poprzednich wersjach Laravel, Blade (i helper `e`) nie będą podwójnie kodować encji HTML. Nie było to domyślne zachowanie ukrytej funkcji `htmlspecialchars` i mogłoby prowadzić do nieoczekiwanego zachowania podczas renderowania treści lub przekazywania treści JSON w linii do frameworków JavaScript.

W Laravel 5.6, Blade i pomocnik `e` domyślnie podwójnie kodują znaki specjalne. Dzięki temu te funkcje są zgodne z domyślnym zachowaniem funkcji PHP `htmlspecialchars`. Jeśli chcesz zachować poprzednie zachowanie w zapobieganiu podwójnemu kodowaniu, możesz przekazać `false` jako drugi argument pomocnika `e`:

    <?php echo e($string, false); ?>

### Logging - Logowanie

#### New Configuration File - Nowy plik konfiguracyjny

Cała konfiguracja rejestrowania znajduje się teraz we własnym pliku konfiguracyjnym `config/logging.php`. Powinieneś umieścić kopię [domyślnego pliku konfiguracyjnego](https://github.com/laravel/laravel/blob/develop/config/logging.php) we własnej aplikacji i dostosować ustawienia w zależności od potrzeb aplikacji.

Opcje konfiguracyjne `log` i `log_level` mogą zostać usunięte z pliku konfiguracyjnego `config/app.php`.

#### The `configureMonologUsing` Method - Metoda `configureMonologUsing`

Jeśli używasz metody `configureMonologUs` w celu dostosowania instancji Monolog do swojej aplikacji, powinieneś teraz utworzyć kanał logowania `custom`. Aby uzyskać więcej informacji na temat tworzenia niestandardowych kanałów, sprawdź [pełną dokumentację dotyczącą rejestrowania](/docs/5.6/logging#creating-custom-channels).

#### The Log `Writer` Class - Log klasa `Writer`

Klasa `Illuminate\Log\Writer` została zmieniona na `Illuminate\Log\Logger`. Jeśli jawnie wpisujesz tę klasę jako zależność jednej z klas aplikacji, powinieneś zaktualizować odwołanie do klasy do nowej nazwy. Lub, alternatywnie, powinieneś zdecydowanie rozważyć wprowadzenie podpowiedzi typu znormalizowanego interfejsu `Psr\Log\LoggerInterface`.

#### The `Illuminate\Contracts\Logging\Log` Interface - Interfejs `Illuminate\Contracts\Logging\Log`

Ten interfejs został usunięty, ponieważ interfejs ten był całkowitym powieleniem interfejsu `Psr\Log\LoggerInterface`. Zamiast tego wpisz-podpowiedz interfejs `Psr\Log\LoggerInterface`.

### Mail - Poczta

#### `withSwiftMessage` Callbacks

W poprzednich wersjach Laravel, wywołania zwrotne Swift Messages, zarejestrowane za pomocą `withSwiftMessage`, były nazywane _after_ treść była już zakodowana i dodana do wiadomości. Te wywołania zwrotne są teraz nazywane _before_ zawartość jest dodawana, co pozwala w razie potrzeby dostosować kodowanie lub inne opcje wiadomości.

### Pagination - Paginacja

#### Bootstrap 4

Łącza do paginacji generowane przez paginator są teraz domyślnie ustawione na Bootstrap 4. Aby polecić paginatorowi generowanie linków Bootstrap 3, wywołaj metodę `Paginator::useBootstrapThree` z metody `boot` twojego `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use Illuminate\Pagination\Paginator;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Paginator::useBootstrapThree();
        }
    }

### Resources - Zasoby

#### The `original` Property - Właściwość `original`

Właściwość `original` [odpowiedzi zasobów](/docs/5.6/eloquent-resources) jest teraz ustawiona na oryginalny model zamiast łańcucha / tablicy JSON. Pozwala to na łatwiejszą kontrolę modelu odpowiedzi podczas testowania.

### Routing

#### Returning Newly Created Models - Powracanie nowo utworzonych modeli

Podczas zwracania nowo utworzonego modelu Eloquent bezpośrednio z trasy, status odpowiedzi zostanie automatycznie ustawiony na `201`, zamiast `200`. Jeśli któryś z testów twojej aplikacji wyraźnie oczekiwał odpowiedzi `200`, testy te powinny zostać zaktualizowane tak, aby oczekiwały `201`.

### Trusted Proxies - Zaufani pośrednicy

Ze względu na zmiany w zaufanej funkcjonalności serwera proxy Symfony HttpFoundation, należy wprowadzić niewielkie zmiany w oprogramowaniu pośredniczącym `App\Http\Middleware\TrustProxies` aplikacji.

Właściwość `$headers`, która poprzednio była tablicą, jest obecnie właściwością bitową, która przyjmuje kilka różnych wartości. Na przykład, aby ufać wszystkim przekazywanym nagłówkom, możesz zaktualizować właściwość `$headers` do następującej wartości:

    use Illuminate\Http\Request;

    /**
     * The headers that should be used to detect proxies.
     *
     * @var string
     */
    protected $headers = Request::HEADER_X_FORWARDED_ALL;

Aby uzyskać więcej informacji na temat dostępnych wartości `$headers`, przejrzyj pełną dokumentację [trusting proxy](/docs/5.6/requests#configuring-trusted-proxies).

### Validation - Walidacja

#### The `ValidatesWhenResolved` Interface - Interfejs `ValidatesWhenResolved`

Metoda `validate` interfejsu / atrybutów `ValidatesWhenResolved` została zmieniona na `validateResolved` w celu uniknięcia konfliktów z metodą `$request->validate()`.

### Miscellaneous - Różne

Zachęcamy również do przeglądania zmian w `laravel/laravel` [repozytorium GitHub](https://github.com/laravel/laravel). Chociaż wiele z tych zmian nie jest wymaganych, możesz chcieć synchronizować te pliki z aplikacją. Niektóre z tych zmian zostaną omówione w tym przewodniku aktualizacji, ale inne, takie jak zmiany w plikach konfiguracyjnych lub komentarzach, nie będą. Możesz łatwo przeglądać zmiany za pomocą [narzędzia porównywania GitHub](https://github.com/laravel/laravel/compare/5.5...master) i wybrać, które aktualizacje są dla Ciebie ważne.