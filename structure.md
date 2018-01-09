# Directory Structure

- [Introduction - Wprowadzenie](#introduction)
- [The Root Directory - Katalog główny](#the-root-directory)
    - [The `app` Directory - Katalog aplikacji](#the-root-app-directory)
    - [The `bootstrap` Directory - Katalog Bootstrap](#the-bootstrap-directory)
    - [The `config` Directory - Katalog konfiguracji](#the-config-directory)
    - [The `database` Directory - Katalog bazy danych](#the-database-directory)
    - [The `public` Directory - Publiczny katalog](#the-public-directory)
    - [The `resources` Directory - Katalog zasobów](#the-resources-directory)
    - [The `routes` Directory - Katalog tras](#the-routes-directory)
    - [The `storage` Directory - Katalog pamięci masowej](#the-storage-directory)
    - [The `tests` Directory  - Katalog testów](#the-tests-directory)
    - [The `vendor` Directory - Katalog dostawców](#the-vendor-directory)
- [The App Directory - Katalog aplikacji](#the-app-directory)
    - [The `Console` Directory - Katalog konsoli](#the-console-directory)
    - [The `Events` Directory - Katalog zdarzeń](#the-events-directory)
    - [The `Exceptions` Directory - Katalog wyjątków](#the-exceptions-directory)
    - [The `Http` Directory - Katalog Http](#the-http-directory)
    - [The `Jobs` Directory - Katalog zadań](#the-jobs-directory)
    - [The `Listeners` Directory - Katalog słuchaczy](#the-listeners-directory)
    - [The `Mail` Directory - Katalog mailowy](#the-mail-directory)
    - [The `Notifications` Directory - Katalog powiadomień](#the-notifications-directory)
    - [The `Policies` Directory - Katalog polityk](#the-policies-directory)
    - [The `Providers` Directory - Katalog dostawców](#the-providers-directory)
    - [The `Rules` Directory - Katalog ról](#the-rules-directory)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Domyślna struktura aplikacji Laravel ma zapewnić dobry punkt wyjścia zarówno dla dużych, jak i małych aplikacji. Oczywiście możesz dowolnie organizować swoją aplikację. Laravel nie nakłada prawie żadnych ograniczeń na to, gdzie znajduje się dana klasa - o ile Composer może automatycznie ładować klasę.

#### Where Is The Models Directory? - Gdzie jest katalog modeli?

Kiedy zaczynamy pracę z Laravel, wielu programistów jest zdezorientowanych brakiem katalogu `models`. Jednak brak takiego katalogu jest zamierzony. Słowo "modele" jest niejednoznaczne, ponieważ oznacza wiele różnych rzeczy dla wielu różnych osób. Niektórzy programiści określają "model" aplikacji jako całość całej logiki biznesowej, podczas gdy inni nazywają "modele" jako klasy interakcyjne z relacyjną bazą danych.

Z tego powodu, domyślnie umieszczamy Wymowne modele w katalogu `app` i zezwalamy programistom na umieszczanie ich gdzie indziej, jeśli zdecydują.

<a name="the-root-directory"></a>
## The Root Directory - Katalog główny

<a name="the-root-app-directory"></a>
#### The App Directory - Katalog aplikacji

Katalog `app`, jak można się spodziewać, zawiera podstawowy kod aplikacji. Wkrótce zapoznamy się z tym katalogiem; jednak prawie wszystkie klasy w twojej aplikacji będą w tym katalogu.

<a name="the-bootstrap-directory"></a>
#### The Bootstrap Directory - Katalog Bootstrap

Katalog `bootstrap` zawiera plik `app.php`, który ładuje system. Katalog ten zawiera także katalog `cache` zawierający pliki generowane przez framework w celu optymalizacji wydajności, takie jak pliki pamięci podręcznej tras i usług.

<a name="the-config-directory"></a>
#### The Config Directory - Katalog konfiguracji

Katalog `config`, jak sama nazwa wskazuje, zawiera wszystkie pliki konfiguracyjne twojej aplikacji. To świetny pomysł, aby przeczytać wszystkie te pliki i zapoznać się ze wszystkimi dostępnymi opcjami

<a name="the-database-directory"></a>
#### The Database Directory - Katalog bazy danych

Katalog `database` zawiera migrację bazy danych i nasiona. Jeśli chcesz, możesz również użyć tego katalogu do przechowywania bazy danych SQLite.

<a name="the-public-directory"></a>
#### The Public Directory - Publiczny katalog

Katalog `public` zawiera plik` index.php`, który jest punktem wejścia dla wszystkich żądań wprowadzanych do aplikacji i konfiguruje automatyczne ładowanie. W tym katalogu znajdują się również twoje zasoby, takie jak obrazy, JavaScript i CSS.

<a name="the-resources-directory"></a>
#### The Resources Directory - Katalog zasobów

Katalog `resources` zawiera twoje widoki, a także twoje surowe, nie skompilowane zasoby, takie jak LESS, SASS lub JavaScript. Ten katalog zawiera także wszystkie pliki językowe.

<a name="the-routes-directory"></a>
#### The Routes Directory - Katalog tras

Katalog `routes` zawiera wszystkie definicje tras dla twojej aplikacji. Domyślnie kilka plików trasy jest dołączonych do Laravel: `web.php`, `api.php`, `console.php` i `channels.php`.

Plik `web.php` zawiera trasy, które` RouteServiceProvider` umieszcza w grupie oprogramowania `web`, która zapewnia stan sesji, ochronę CSRF i szyfrowanie plików cookie. Jeśli twoja aplikacja nie oferuje bezstanowego, RESTful API, wszystkie twoje trasy najprawdopodobniej będą zdefiniowane w pliku `web.php`.

Plik `api.php` zawiera trasy, które `RouteServiceProvider` umieszcza w grupie oprogramowania pośredniczącego `api`, która zapewnia ograniczenie szybkości. Te trasy mają być statem bezstanowym, więc żądania wprowadzenia aplikacji za pośrednictwem tych tras mają być uwierzytelniane za pomocą tokenów i nie będą miały dostępu do stanu sesji.

Plik `console.php` to miejsce, gdzie możesz zdefiniować wszystkie polecenia konsoli oparte na Closure. Każde Closure jest powiązane z instancją polecenia pozwalającą na proste podejście do interakcji z metodami IO każdego polecenia. Mimo że ten plik nie definiuje tras HTTP, definiuje punkty wejścia (trasy) oparte na konsoli do twojej aplikacji.

Plik `channels.php` to miejsce, w którym możesz zarejestrować wszystkie kanały rozgłaszania zdarzeń, które obsługuje twoja aplikacja.

<a name="the-storage-directory"></a>
#### The Storage Directory - Katalog pamięci masowej

Katalog `storage` zawiera skompilowane szablony Blade, sesje oparte na plikach, pamięci podręczne plików i inne pliki generowane przez framework. Ten katalog jest podzielony na katalogi `app`, `framework` oraz `logs`. Katalog `app` może być używany do przechowywania dowolnych plików wygenerowanych przez twoją aplikację. Katalog `framework` służy do przechowywania plików generowanych przez framework i pamięci podręcznych. Na koniec katalog `logs` zawiera pliki dziennika twojej aplikacji.

Katalog `storage/app/public` może być używany do przechowywania plików generowanych przez użytkowników, takich jak awatary profilowe, które powinny być publicznie dostępne. Powinieneś utworzyć dowiązanie symboliczne w `public/storage`, które wskazuje na ten katalog. Możesz utworzyć łącze za pomocą polecenia `php artisan storage:link`.

<a name="the-tests-directory"></a>
#### The Tests Directory - Katalog testów

Katalog `tests` zawiera twoje automatyczne testy. Przykład [PHPUnit](https://phpunit.de/) jest dostarczany po wyjęciu z pudełka. Każda klasa testowa powinna być uzupełniona o słowo `Test`. Możesz uruchomić testy za pomocą komend `phpunit` lub `php vendor/bin/phpunit`.

<a name="the-vendor-directory"></a>
#### The Vendor Directory - Katalog dostawców

Katalog `vendor` zawiera zależności użytkownika [Composer](https://getcomposer.org).

<a name="the-app-directory"></a>
## The App Directory - Katalog aplikacji

Większość aplikacji znajduje się w katalogu `app`. Domyślnie katalog ten jest wyświetlany pod nazwą `App` i jest ładowany automatycznie przez Composer za pomocą [standardu automatycznego ładowania PSR-4](http://www.php-fig.org/psr/psr-4/).

Katalog `app` zawiera wiele dodatkowych katalogów, takich jak `Console`, `Http` i `Providers`. Pomyśl o katalogach `Console` i `Http` jako o API w rdzeniu twojej aplikacji. Protokół HTTP i CLI są zarówno mechanizmami do interakcji z aplikacją, ale tak naprawdę nie zawierają logiki aplikacji. Innymi słowy, są to po prostu dwa sposoby wydawania poleceń do aplikacji. Katalog `Console` zawiera wszystkie twoje polecenia Artisan, podczas gdy katalog `Http` zawiera twoje kontrolery, middleware i żądania.

W katalogu `app` będzie generowanych wiele innych katalogów, ponieważ do generowania klas używane są polecenia `make` Artisan. Na przykład katalog `app/Jobs` nie będzie istnieć, dopóki nie wykonasz polecenia `make:job` Artisan, aby wygenerować klasę zadań.

> {tip} Wiele klas w katalogu `app` może być generowanych przez Artisan za pomocą poleceń. Aby przejrzeć dostępne polecenia, uruchom polecenie `php artisan list make` w twoim terminalu.

<a name="the-console-directory"></a>
#### The Console Directory - Katalog konsoli

Katalog `Console` zawiera wszystkie niestandardowe polecenia Artisan dla twojej aplikacji. Te polecenia mogą być generowane za pomocą komendy `make:command`. Ten katalog zawiera także jądro konsoli, w którym są zarejestrowane twoje niestandardowe polecenia Artisan, a twoje [zaplanowane zadania](/docs/{{version}}/scheduling) są zdefiniowane.

<a name="the-events-directory"></a>
#### The Events Directory - Katalog zdarzeń

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla ciebie przez polecenia `event:generate` i `make:event` Artisan. Katalog `Events`, jak można się spodziewać, zawiera domy [klasy zdarzeń](/docs/{{version}}/events). Zdarzenia mogą być wykorzystywane do ostrzegania innych części aplikacji o wystąpieniu danej akcji, zapewniając dużą elastyczność i oddzielenie.

<a name="the-exceptions-directory"></a>
#### The Exceptions Directory - Katalog wyjątków

Katalog `Exceptions` zawiera aplikację obsługi wyjątków aplikacji i jest również dobrym miejscem do umieszczenia wyjątków zgłaszanych przez aplikację. Jeśli chcesz dostosować sposób rejestrowania lub renderowania wyjątków, powinieneś zmodyfikować klasę `Handler` w tym katalogu.

<a name="the-http-directory"></a>
#### The Http Directory - Katalog Http

Katalog `Http` zawiera twoje kontrolery, oprogramowanie pośrednie i żądania formularzy. Prawie cała logika obsługi zgłoszeń wprowadzanych do aplikacji zostanie umieszczona w tym katalogu.

<a name="the-jobs-directory"></a>
#### The Jobs Directory - Katalog zadań

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla ciebie, jeśli wykonasz polecenie `make:job` Artisan. Katalog `Jobs` zawiera [zadania do wykonania](/docs/{{version}}/queues) dla twojej aplikacji. Zadania mogą być kolejkowane w aplikacji lub działać synchronicznie w ramach bieżącego cyklu życia żądania. Zadania, które działają synchronicznie podczas bieżącego żądania, są czasami określane jako "komendy", ponieważ są implementacją [wzorca poleceń](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### The Listeners Directory - Katalog słuchaczy

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla ciebie, jeśli wykonasz komendy artisan `event:generate` lub `make:listener`. Katalog `Listeners` zawiera klasy, które obsługują twoje [zdarzenia](/docs/{{version}}/events). Detektory zdarzeń otrzymują wystąpienie zdarzenia i wykonują logikę w odpowiedzi na wystrzeliwane zdarzenie. Na przykład zdarzenie `UserRegistered` może być obsługiwane przez detektor` SendWelcomeEmail`.

<a name="the-mail-directory"></a>
#### The Mail Directory - Katalog mailowy

Katalog ten nie istnieje domyślnie, ale zostanie utworzony dla ciebie, jeśli wykonasz polecenie `make:mail` Artisan. Katalog `Mail` zawiera wszystkie twoje klasy reprezentujące wiadomości e-mail wysyłane przez twoją aplikację. Obiekty poczty umożliwiają enkapsulację całej logiki budowania wiadomości e-mail w pojedynczej, prostej klasie, która może być wysłana za pomocą metody `Mail::send`.

<a name="the-notifications-directory"></a>
#### The Notifications Directory - Katalog powiadomień

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla ciebie, jeśli wykonasz polecenie `make:notification` Artisan. Katalog `Notifications` zawiera wszystkie "transakcyjne" powiadomienia wysyłane przez Twoją aplikację, takie jak proste powiadomienia o zdarzeniach mających miejsce w Twojej aplikacji. Powiadomienia Laravel zawierają streszczenia wysyłania powiadomień przez różne sterowniki, takie jak e-mail, Slack, SMS lub przechowywane w bazie danych.

<a name="the-policies-directory"></a>
#### The Policies Directory - Katalog polityk

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla ciebie, jeśli wykonasz polecenie `make:policy` Artisan. Katalog `Policies` zawiera klasy zasad autoryzacji dla aplikacji. Zasady są używane do określenia, czy użytkownik może wykonać daną akcję względem zasobu. Aby uzyskać więcej informacji, zapoznaj się z [dokumentacją autoryzacyjną](/docs/{{version}}/authorization).

<a name="the-providers-directory"></a>
#### The Providers Directory - Katalog dostawców

Katalog `Providers` zawiera wszystkich [usługodawców](/docs/{{version}}/providers) dla twojej aplikacji. Usługodawcy uruchamiają aplikację przez wiązanie usług w kontenerze usług, rejestrowanie zdarzeń lub wykonywanie innych zadań w celu przygotowania wniosku o przychodzące żądania.

W świeżej aplikacji Laravel ten katalog będzie już zawierać kilku dostawców. W razie potrzeby możesz dodać własnych dostawców do tego katalogu.

<a name="the-rules-directory"></a>
#### The Rules Directory - Katalog ról

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla ciebie, jeśli wykonasz polecenie `make:rule` Artisan. Katalog `Rules` zawiera niestandardowe obiekty reguł sprawdzania poprawności dla aplikacji. Reguły są używane do enkapsulacji skomplikowanej logiki walidacji w prostym obiekcie. Aby uzyskać więcej informacji, zapoznaj się z [dokumentacją walidacyjną](/docs/{{version}}/validation).
