# Browser Tests (Laravel Dusk)

- [Introduction - Wprowadzenie](#introduction)
- [Installation - Instalacja](#installation)
    - [Using Other Browsers - Korzystanie z innych przeglądarek](#using-other-browsers)
- [Getting Started - Pierwsze kroki](#getting-started)
    - [Generating Tests - Generowanie testów](#generating-tests)
    - [Running Tests - Uruchamianie testów](#running-tests)
    - [Environment Handling - Obsługa środowiska](#environment-handling)
    - [Creating Browsers - Tworzenie przeglądarek](#creating-browsers)
    - [Authentication - Poświadczenie](#authentication)
    - [Database Migrations - Migracje baz danych](#migrations)
- [Interacting With Elements - Interakcja z elementami](#interacting-with-elements)
    - [Dusk Selectors - Selektory Dusk](#dusk-selectors)
    - [Clicking Links - Kliknięcie łącza](#clicking-links)
    - [Text, Values, & Attributes - Tekst, wartości i atrybuty](#text-values-and-attributes)
    - [Using Forms - Korzystanie z formularzy](#using-forms)
    - [Attaching Files - Dołączanie plików](#attaching-files)
    - [Using The Keyboard - Korzystanie z klawiatury](#using-the-keyboard)
    - [Using The Mouse - Używanie myszy](#using-the-mouse)
    - [Scoping Selectors - Selektory zakresu](#scoping-selectors)
    - [Waiting For Elements - Oczekiwanie na elementy](#waiting-for-elements)
    - [Making Vue Assertions - Tworzenie asercji Vue](#making-vue-assertions)
- [Available Assertions - Dostępne asercje](#available-assertions)
- [Pages - Strony](#pages)
    - [Generating Pages - Generowanie stron](#generating-pages)
    - [Configuring Pages - Konfigurowanie stron](#configuring-pages)
    - [Navigating To Pages - Nawigowanie do stron](#navigating-to-pages)
    - [Shorthand Selectors - Skróty selektorów](#shorthand-selectors)
    - [Page Methods - Metody strony](#page-methods)
- [Components - Komponenty](#components)
    - [Generating Components - Generowanie komponentów](#generating-components)
    - [Using Components - Używanie komponentów](#using-components)
- [Continuous Integration - Ciągła integracja](#continuous-integration)
    - [Travis CI](#running-tests-on-travis-ci)
    - [CircleCI](#running-tests-on-circle-ci)
    - [Codeship](#running-tests-on-codeship)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel Dusk zapewnia ekspresyjny, łatwy w użyciu interfejs API do automatyzacji i testowania przeglądarki. Domyślnie Dusk nie wymaga instalacji JDK ani Selenium na twoim komputerze. Zamiast tego Dusk korzysta z samodzielnej instalacji [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home). Możesz jednak korzystać z dowolnego innego zgodnego sterownika Selenium.

<a name="installation"></a>
## Installation - Instalacja

Aby rozpocząć, powinieneś dodać zależność `laravel/dusk` Composer do swojego projektu:

    composer require --dev laravel/dusk:^2.0

Po zainstalowaniu Dusk należy zarejestrować dostawcę usług `Laravel\Dusk\DuskServiceProvider`. Zazwyczaj odbywa się to automatycznie poprzez rejestrację automatycznego dostawcy usługi Laravel.

> {note} Jeśli ręcznie rejestrujesz dostawcę usług Dusk, **nigdy** nie rejestruj go w swoim środowisku produkcyjnym, ponieważ może to doprowadzić do tego, że arbitralni użytkownicy będą mogli uwierzytelnić się w Twojej aplikacji.

Po zainstalowaniu pakietu Dusk, uruchom polecenie `dusk:install` Artisan:

    php artisan dusk:install

Katalog `Browser` zostanie utworzony w twoim katalogu `tests` i będzie zawierał przykładowy test. Następnie ustaw zmienną środowiskową `APP_URL` w swoim pliku` .env`. Ta wartość powinna odpowiadać adresowi URL używanemu do uzyskania dostępu do aplikacji w przeglądarce.

Aby uruchomić testy, użyj polecenia `dusk` Artisan. Polecenie `dusk` przyjmuje każdy argument, który jest również akceptowany przez polecenie `phpunit`:

    php artisan dusk

<a name="using-other-browsers"></a>
### Using Other Browsers - Korzystanie z innych przeglądarek

Domyślnie Dusk używa przeglądarki Google Chrome i samodzielnej instalacji [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home), aby uruchomić testy przeglądarki. Możesz jednak uruchomić własny serwer Selenium i uruchomić testy w dowolnej przeglądarce.

Aby rozpocząć, otwórz plik `tests/DuskTestCase.php`, który jest podstawowym testem Dusk dla twojej aplikacji. W tym pliku możesz usunąć wywołanie metody `startChromeDriver`. Spowoduje to zatrzymanie Dusk od automatycznego uruchamiania ChromeDriver:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Następnie możesz zmodyfikować metodę `driver`, aby połączyć się z wybranym adresem URL i portem. Ponadto można zmodyfikować "pożądane możliwości", które należy przekazać do WebDriver:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## Getting Started - Pierwsze kroki

<a name="generating-tests"></a>
### Generating Tests - Generowanie testów

Aby wygenerować test Dusk, użyj polecenia `dusk:make` Artisan. Wygenerowany test zostanie umieszczony w katalogu `test/Browser`:

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### Running Tests - Uruchamianie testów

Aby uruchomić testy przeglądarki, użyj polecenia `dusk` Artisan:

    php artisan dusk

Polecenie `dusk` akceptuje każdy argument, który jest zwykle akceptowany przez uruchamiacz testu PHPUnit, co pozwala ci tylko uruchamiać testy dla danej [grupy](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group), itp .:

    php artisan dusk --group=foo

#### Manually Starting ChromeDriver - Ręczne uruchamianie ChromeDriver

Domyślnie Dusk automatycznie spróbuje uruchomić ChromeDriver. Jeśli to nie działa w twoim systemie, możesz ręcznie uruchomić ChromeDriver przed uruchomieniem polecenia `dusk`. Jeśli zdecydujesz się ręcznie uruchomić ChromeDriver, powinieneś skomentować następujący wiersz pliku `tests/DuskTestCase.php`:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Ponadto, jeśli uruchomisz ChromeDriver na porcie innym niż 9515, powinieneś zmodyfikować metodę `driver` dla tej samej klasy:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### Environment Handling - Obsługa środowiska

Aby zmusić Dusk do używania własnego pliku środowiska podczas uruchamiania testów, utwórz plik `.env.dusk.{environment}` w katalogu głównym projektu. Na przykład, jeśli będziesz inicjować komendę `dusk` ze swojego środowiska `local`, powinieneś utworzyć plik `.env.dusk.local`.

Podczas testów, Dusk wykona kopię zapasową twojego pliku `.env` i zmieni nazwę twojego środowiska Dusk na `.env`. Po zakończeniu testów plik ".env" zostanie przywrócony.

<a name="creating-browsers"></a>
### Creating Browsers - Tworzenie przeglądarek

Aby rozpocząć, napiszmy test, który sprawdzi, czy możemy zalogować się do naszej aplikacji. Po wygenerowaniu testu możemy go zmodyfikować, aby przejść do strony logowania, wprowadzić dane uwierzytelniające i kliknąć przycisk "Login". Aby utworzyć instancję przeglądarki, wywołaj metodę `browse`:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * A basic browser test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

Jak widać w powyższym przykładzie, metoda `browse` akceptuje wywołanie zwrotne. Instancja przeglądarki zostanie automatycznie przekazana do tego wywołania zwrotnego przez Dusk i jest głównym obiektem używanym do interakcji i tworzenia asercji z aplikacją.

> {tip} Ten test może być użyty do przetestowania ekranu logowania wygenerowanego przez polecenie `make:auth` Artisan.

#### Creating Multiple Browsers - Tworzenie wielu przeglądarek

Czasami możesz potrzebować wielu przeglądarek, aby prawidłowo przeprowadzić test. Na przykład do testowania ekranu czatu, który współdziała z sieciami internetowymi, może być potrzebnych wiele przeglądarek. Aby utworzyć wiele przeglądarek, "zapytaj" o więcej niż jedną przeglądarkę w oznaczeniu wywołania zwrotnego podanego do metody `browse`:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

#### Resizing Browser Windows - Zmienianie rozmiaru okien przeglądarki

Możesz użyć metody `resize`, aby dostosować rozmiar okna przeglądarki:

    $browser->resize(1920, 1080);

Do maksymalizacji okna przeglądarki można użyć metody `maximize`:

    $browser->maximize();

<a name="authentication"></a>
### Authentication - Poświadczenie

Często będziesz testować strony wymagające uwierzytelnienia. Możesz użyć metody `loginAs` Dusk, aby uniknąć interakcji z ekranem logowania podczas każdego testu. Metoda `loginAs` akceptuje identyfikator użytkownika lub instancję modelu użytkownika:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} Po użyciu metody `loginAs` sesja użytkownika zostanie zachowana dla wszystkich testów w pliku.

<a name="migrations"></a>
### Database Migrations - Migracje baz danych

Kiedy twój test wymaga migracji, takich jak powyższy przykład uwierzytelnienia, nigdy nie powinieneś używać cechy `RefreshDatabase`. Właściwość `RefreshDatabase` wykorzystuje transakcje bazy danych, które nie będą miały zastosowania we wszystkich żądaniach HTTP. Zamiast tego użyj cechy `DatabaseMigrations`:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    }

<a name="interacting-with-elements"></a>
## Interacting With Elements - Interakcja z elementami

<a name="dusk-selectors"></a>
### Dusk Selectors - Selektory Dusk

Wybór dobrych selektorów CSS do interakcji z elementami jest jedną z najtrudniejszych części pisania testów Dusk. Z czasem zmiany frontendu mogą spowodować, że selektory CSS, takie jak poniższe, będą łamać twoje testy:

    // HTML...

    <button>Login</button>

    // Test...

    $browser->click('.login-page .container div > button');

Selektory Dusk pozwalają skupić się na pisaniu efektywnych testów zamiast zapamiętywania selektorów CSS. Aby zdefiniować selektor, dodaj atrybut "dusk" do swojego elementu HTML. Następnie prefiksuj selektor za pomocą `@ ', aby manipulować załączonym elementem w teście Dusk:

    // HTML...

    <button dusk="login-button">Login</button>

    // Test...

    $browser->click('@login-button');

<a name="clicking-links"></a>
### Clicking Links - Kliknięcie łącza

Aby kliknąć łącze, możesz użyć metody `clickLink` w instancji przeglądarki. Metoda `clickLink` kliknie link z podanym tekstem wyświetlanym:

    $browser->clickLink($linkText);

> {note} Ta metoda współdziała z jQuery. Jeśli jQuery nie jest dostępny na stronie, Dusk automatycznie wstrzykuje go na stronę, aby był dostępny przez czas trwania testu.

<a name="text-values-and-attributes"></a>
### Text, Values, & Attributes - Tekst, wartości i atrybuty

#### Retrieving & Setting Values - Pobieranie i ustawianie wartości

Dusk udostępnia kilka metod interakcji z bieżącym wyświetlanym tekstem, wartością i atrybutami elementów na stronie. Na przykład, aby uzyskać "wartość" elementu, który pasuje do danego selektora, użyj metody `value`:

    // Retrieve the value...
    $value = $browser->value('selector');

    // Set the value...
    $browser->value('selector', 'value');

#### Retrieving Text - Pobieranie tekstu

Metodę `text` można użyć do wyświetlenia wyświetlanego tekstu elementu pasującego do podanego selektora:

    $text = $browser->text('selector');

#### Retrieving Attributes - Pobieranie atrybutów

Wreszcie, metoda `attribute` może być użyta do pobrania atrybutu elementu pasującego do podanego selektora:

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>
### Using Forms - Korzystanie z formularzy

#### Typing Values - Wpisywanie wartości

Dusk dostarcza różnorodnych metod interakcji z formularzami i elementami wejściowymi. Najpierw przyjrzyjmy się przykładowi wpisywania tekstu w polu wejściowym:

    $browser->type('email', 'taylor@laravel.com');

Należy zauważyć, że chociaż metoda akceptuje jedną, jeśli jest to konieczne, nie musimy przekazywać selektora CSS do metody `type`. Jeśli selektor CSS nie zostanie podany, Dusk wyszuka pole wejściowe z podanym atrybutem `name`. Wreszcie, Dusk spróbuje znaleźć `textarea` z podanym atrybutem` name`.

Aby dołączyć tekst do pola bez czyszczenia jego zawartości, możesz użyć metody `append`:

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

Możesz wyczyścić wartość wejścia za pomocą metody `clear`:

    $browser->clear('email');

#### Dropdowns - Listy rozwijane

Aby wybrać wartość z rozwijanego menu wyboru, możesz użyć metody `select`. Podobnie jak w przypadku metody `type`, metoda `select` nie wymaga pełnego selektora CSS. Podczas przekazywania wartości do metody `select` należy przekazać wartość opcji bazowej zamiast wyświetlanego tekstu:

    $browser->select('size', 'Large');

Możesz wybrać opcję losową, pomijając drugi parametr:

    $browser->select('size');

#### Checkboxes - Pola wyboru

Aby "sprawdzić" pole wyboru, możesz użyć metody `check`. Podobnie jak wiele innych metod związanych z wprowadzaniem danych, pełny selektor CSS nie jest wymagany. Jeśli nie można znaleźć dokładnego dopasowania selektora, Dusk wyszuka pole wyboru z pasującym atrybutem `name`:

    $browser->check('terms');

    $browser->uncheck('terms');

#### Radio Buttons - Pole radio

Aby "wybrać" opcję pola radio, możesz użyć metody `radio`. Podobnie jak wiele innych metod związanych z wprowadzaniem danych, pełny selektor CSS nie jest wymagany. Jeśli nie można znaleźć dokładnego dopasowania selektora, Dusk wyszuka radio z pasującymi atrybutami `name` i `value`:

    $browser->radio('version', 'php7');

<a name="attaching-files"></a>
### Attaching Files - Dołączanie plików

Do dołączenia pliku do elementu wejściowego `file` można użyć metody `attach`. Podobnie jak wiele innych metod związanych z wprowadzaniem danych, pełny selektor CSS nie jest wymagany. Jeśli nie można znaleźć dokładnego dopasowania, Dusk wyszuka dane wejściowe z pasującym atrybutem `name`:

    $browser->attach('photo', __DIR__.'/photos/me.png');

<a name="using-the-keyboard"></a>
### Using The Keyboard - Korzystanie z klawiatury

Metoda `keys` pozwala na dostarczanie bardziej złożonych sekwencji wejściowych do danego elementu niż normalnie dozwolona przez metodę `type`. Na przykład możesz trzymać klawisze modyfikujące wprowadzając wartości. W tym przykładzie klawisz `shift` zostanie przytrzymany, podczas gdy `taylor` zostanie wprowadzony do elementu pasującego do danego selektora. Po wpisaniu `taylor`,  `otwell` zostanie wpisane bez żadnych klawiszy modyfikujących:

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

Możesz nawet wysłać "klawisz skrótu" do głównego selektora CSS, który zawiera twoją aplikację:

    $browser->keys('.app', ['{command}', 'j']);

> {tip} Wszystkie klucze modyfikujące są zawijane w znaki `{}` i dopasowują stałe zdefiniowane w klasie `Facebook\WebDriver\WebDriverKeys`, które mogą być [znalezione na GitHubie](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>
### Using The Mouse - Używanie myszy

#### Clicking On Elements - Kliknięcie na elementy

Metodę typu `click` można użyć do "kliknięcia" elementu odpowiadającego danemu selektorowi:

    $browser->click('.selector');

#### Mouseover - Po najechaniu myszką

Metodę `mouseover` można zastosować, gdy trzeba przesunąć kursor myszy nad elementem pasującym do podanego selektora:

    $browser->mouseover('.selector');

#### Drag & Drop - Przeciągnij i upuść

Metodę `drag` można użyć do przeciągnięcia elementu pasującego do wybranego selektora na inny element:

    $browser->drag('.from-selector', '.to-selector');

Lub możesz przeciągnąć element w jednym kierunku:

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

<a name="scoping-selectors"></a>
### Scoping Selectors - Selektory zakresu

Czasami możesz chcieć wykonać kilka operacji podczas ustalania zakresu wszystkich operacji w obrębie danego selektora. Na przykład możesz chcieć potwierdzić, że jakiś tekst istnieje tylko w tabeli, a następnie kliknąć przycisk w tej tabeli. Do tego celu możesz użyć metody `with`. Wszystkie operacje wykonywane w ramach wywołania zwrotnego podane dla metody `with` będą miały zasięg do pierwotnego selektora:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

<a name="waiting-for-elements"></a>
### Waiting For Elements - Oczekiwanie na elementy

Podczas testowania aplikacji, które intensywnie korzystają z JavaScript, często konieczne staje się "oczekiwanie" na udostępnienie pewnych elementów lub danych przed przystąpieniem do testu. Dusk sprawia, że jest to łatwe zadanie. Korzystając z różnych metod, możesz poczekać, aż elementy będą widoczne na stronie, a nawet poczekać, aż dane wyrażenie JavaScript zwróci wartość `true`.

#### Waiting - Oczekiwanie

Jeśli chcesz wstrzymać test na określoną liczbę milisekund, użyj metody `pause`:

    $browser->pause(1000);

#### Waiting For Selectors - Oczekiwanie na selektor

Metodę `waitFor` można użyć do wstrzymania wykonywania testu, dopóki element pasujący do podanego selektora CSS nie wyświetli się na stronie. Domyślnie wstrzyma to test na maksymalnie pięć sekund przed wyrzuceniem wyjątku. Jeśli to konieczne, możesz przekazać niestandardowy próg limitu czasu jako drugi argument metody:

    // Wait a maximum of five seconds for the selector... - Zaczekaj maksymalnie pięć sekund na selektor ...
    $browser->waitFor('.selector');

    // Wait a maximum of one second for the selector... - Zaczekaj maksymalnie 1 sekundę na selektor ...
    $browser->waitFor('.selector', 1);

Możesz także poczekać, aż dany selektor zniknie ze strony:

    $browser->waitUntilMissing('.selector');

    $browser->waitUntilMissing('.selector', 1);

#### Scoping Selectors When Available - Selektory zakresu, gdy są dostępne

Czasami możesz poczekać na dany selektor, a następnie wejść w interakcję z elementem pasującym do selektora. Na przykład możesz poczekać, aż pojawi się okno modalne, a następnie nacisnąć przycisk "OK" w modalu. W tym przypadku można użyć metody `whenAvailable`. Wszystkie operacje elementów wykonywane w ramach danego wywołania zwrotnego będą miały zasięg do pierwotnego selektora:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### Waiting For Text - Oczekiwanie na tekst

Metodę `waitForText` można użyć do oczekiwania, aż dany tekst zostanie wyświetlony na stronie:

    // Wait a maximum of five seconds for the text... - Zaczekaj maksymalnie pięć sekund na tekst ...
    $browser->waitForText('Hello World');

    // Wait a maximum of one second for the text... - Zaczekaj maksymalnie 1 sekundę na tekst ...
    $browser->waitForText('Hello World', 1);

#### Waiting For Links - Oczekiwanie na link

Metodę `waitForLink` można używać do czekania, aż dany tekst łącza zostanie wyświetlony na stronie:

    // Wait a maximum of five seconds for the link... - Zaczekaj maksymalnie pięć sekund na link ...
    $browser->waitForLink('Create');

    // Wait a maximum of one second for the link... - Zaczekaj maksymalnie 1 sekundę na link ...
    $browser->waitForLink('Create', 1);

#### Waiting On The Page Location - Oczekiwanie na lokalizacja strony

Podczas tworzenia asercji ścieżki, takiej jak `$browser->assertPathIs('/home')`, asercja może się nie powieść, jeśli `window.location.pathname` jest aktualizowana asynchronicznie. Możesz użyć metody `waitForLocation`, aby poczekać, aż lokalizacja będzie miała określoną wartość:

    $browser->waitForLocation('/secret');

#### Waiting for Page Reloads - Oczekiwanie na ponowne wczytanie strony

Jeśli chcesz wykonać asercje po przeładowaniu strony, użyj metody `waitForReload`:

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

#### Waiting On JavaScript Expressions - Oczekiwanie na wyrażeń JavaScript

Czasami możesz wstrzymać wykonywanie testu, dopóki dane wyrażenie JavaScript nie zwróci wartości `true`. Możesz to łatwo osiągnąć za pomocą metody `waitUntil`. Podczas przekazywania wyrażenia do tej metody nie trzeba uwzględniać słowa kluczowego `return` ani końcowego średnika:

    // Wait a maximum of five seconds for the expression to be true... - Odczekaj maksymalnie pięć sekund, aby wyrażenie było prawdziwe ...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // Wait a maximum of one second for the expression to be true... - Zaczekaj maksymalnie jedną sekundę, aby wyrażenie było prawdziwe ...
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### Waiting With A Callback - Czekam z oddzwonieniem

Wiele metod "czekania" w Dusk opiera się na bazowej metodzie `waitUsing`. Możesz użyć tej metody bezpośrednio, aby poczekać, aż dany zwrot wywoła zwrot `true`. Metoda `waitUsing` akceptuje maksymalną liczbę sekund oczekiwania, przedział, w jakim powinna być oceniana Closure, Closure i opcjonalny komunikat o błędzie:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="making-vue-assertions"></a>
### Making Vue Assertions - Tworzenie asercji Vue

Dusk pozwala nawet na tworzenie twierdzeń dotyczących stanu danych składnika [Vue](https://vuejs.org). Na przykład wyobraź sobie, że twoja aplikacja zawiera następujący komponent Vue:

    // HTML...

    <profile dusk="profile-component"></profile>

    // Component Definition...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                  name: 'Taylor'
                }
            };
        }
    });

Możesz twierdzić na temat stanu komponentu Vue, jak na przykładzie:

    /**
     * A basic Vue test example.
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="available-assertions"></a>
## Available Assertions - Dostępne asercje

Dusk dostarcza różnorodnych twierdzeń, które możesz podjąć przeciwko swojej aplikacji. Wszystkie dostępne asercje są udokumentowane w poniższej tabeli:

Twierdzenia  | Opis
------------- | -------------
`$browser->assertTitle($title)`  |  Twierdzimy, że tytuł strony pasuje do podanego tekstu.
`$browser->assertTitleContains($title)`  |  Twierdzimy, że tytuł strony zawiera podany tekst.
`$browser->assertUrlIs($url)`  |  Twierdzimy, że bieżący URL (bez ciągu zapytania) pasuje do podanego ciągu.
`$browser->assertPathBeginsWith($path)`  |  Twierdzimy się, że bieżąca ścieżka URL zaczyna się od podanej ścieżki.
`$browser->assertPathIs('/home')`  |  Twierdzimy, że obecna ścieżka pasuje do podanej ścieżki.
`$browser->assertPathIsNot('/home')`  |  Twierdzimy, że obecna ścieżka nie pasuje do podanej ścieżki.
`$browser->assertRouteIs($name, $parameters)`  |  Twierdzimy, że bieżący adres URL pasuje do podanego adresu URL danej trasy.
`$browser->assertQueryStringHas($name, $value)`  |  Twierdzimy, że podany parametr ciągu zapytania jest obecny i ma określoną wartość.
`$browser->assertQueryStringMissing($name)`  |  Twierdzimy, że jest brak podanego parametru ciągu zapytania.
`$browser->assertHasQueryStringParameter($name)`  |  Twierdzimy, że podany parametr ciągu zapytania jest obecny.
`$browser->assertHasCookie($name)`  |  Twierdzimy, że podany plik ciasteczka jest obecny.
`$browser->assertCookieMissing($name)`  |  Twierdzimy, że podany plik ciasteczka nie jest obecne.
`$browser->assertCookieValue($name, $value)`  |  Twierdzimy, że plik ciasteczka ma określoną wartość.
`$browser->assertPlainCookieValue($name, $value)`  |  Twierdzimy, że niezaszyfrowany plik ciasteczka ma określoną wartość.
`$browser->assertSee($text)`  |  Twierdzimy, że dany tekst jest obecny na stronie.
`$browser->assertDontSee($text)`  |  Twierdzimy, że dany tekst nie jest obecny na stronie.
`$browser->assertSeeIn($selector, $text)`  |  Twierdzimy, że dany tekst jest obecny w selektorze.
`$browser->assertDontSeeIn($selector, $text)`  |  Twierdzimy, że dany tekst nie jest obecny w selektorze.
`$browser->assertSourceHas($code)`  |  Twierdzimy, że dany kod źródłowy jest obecny na stronie.
`$browser->assertSourceMissing($code)`  |  Twierdzimy, że podany kod źródłowy nie występuje na stronie.
`$browser->assertSeeLink($linkText)`  |  Twierdzimy, że dany link jest obecny na stronie.
`$browser->assertDontSeeLink($linkText)`  |  Twierdzimy, że podany link nie jest obecny na stronie.
`$browser->assertInputValue($field, $value)`  |  Twierdzimy, że dane pole wejściowe ma podaną wartość.
`$browser->assertInputValueIsNot($field, $value)`  |  Twierdzimy, że podane pole wejściowe nie ma podanej wartości.
`$browser->assertChecked($field)`  |  Twierdzimy, że zaznaczone pole wyboru jest zaznaczone.
`$browser->assertNotChecked($field)`  |  Twierdzimy, że zaznaczone pole wyboru nie jest zaznaczone.
`$browser->assertRadioSelected($field, $value)`  |  Twierdzenie, że dane pole radio jest wybrane.
`$browser->assertRadioNotSelected($field, $value)` |  Twierdzenie, że dane pole radio nie jest wybrane.
`$browser->assertSelected($field, $value)`  |  Twierdzenie, że w liście rozwijanej wybrana jest dana wartość.
`$browser->assertNotSelected($field, $value)`  |  Twierdzenie, że w liście rozwijanej nie wybrana jest dana wartość.
`$browser->assertSelectHasOptions($field, $values)`  |  Twierdzenie, że dana opcja zaznaczenia jest dostępna do wybrania.
`$browser->assertSelectMissingOptions($field, $values)`  |  Twierdzenie, że dana opcja zaznaczenia nie jest dostępna do wybrania.
`$browser->assertSelectHasOption($field, $value)`  |  Twierdzenie, że dana opcja zaznaczenia jest dostępna do wybrania w danym polu.
`$browser->assertValue($selector, $value)`  |  Twierdzenie, że element pasujący do danego selektora ma podaną wartość.
`$browser->assertVisible($selector)`  |  Twierdzenie, że element pasujący do danego selektora jest widoczny.
`$browser->assertMissing($selector)`  |  Twierdzenie, że element pasujący do danego selektora nie jest widoczny.
`$browser->assertDialogOpened($message)`  |  Twierdzenie, że zostało otwarte okno dialogowe JavaScript z podaną wiadomością.
`$browser->assertVue($property, $value, $component)`  |  Twierdzenie, że dana właściwość komponentu Vue jest zgodna z podaną wartością.
`$browser->assertVueIsNot($property, $value, $component)`  | Twierdzenie, że dana właściwość komponentu Vue nie jest zgodna z podaną wartością.

<a name="pages"></a>
## Pages - Strony

Czasami testy wymagają wykonania kilku skomplikowanych czynności w sekwencji. Może to sprawić, że twoje testy będą trudniejsze do odczytania i zrozumienia. Strony umożliwiają definiowanie działań ekspresywnych, które następnie mogą być wykonywane na danej stronie przy użyciu jednej metody. Strony umożliwiają również definiowanie skrótów do wspólnych selektorów dla aplikacji lub pojedynczej strony.

<a name="generating-pages"></a>
### Generating Pages - Generowanie stron

Aby wygenerować obiekt strony, użyj polecenia `dusk:page` Artisan. Wszystkie obiekty strony zostaną umieszczone w katalogu `tests/Browser/Pages`:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### Configuring Pages - Konfigurowanie stron

Domyślnie strony mają trzy metody: `url`, `assert`, i `elements`. Omówimy teraz metody `url` i `assert`. Metoda `elements` będzie [omówiona bardziej szczegółowo poniżej](#shorthand-selectors).

#### The `url` Method - Metoda `url`

Metoda `url` powinna zwracać ścieżkę adresu URL, który reprezentuje stronę. Dusk użyje tego adresu URL podczas nawigacji do strony w przeglądarce:

    /**
     * Get the URL for the page.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

#### The `assert` Method - Metoda `assert`

Metoda `assert` może wykonywać dowolne asercje niezbędne do zweryfikowania, czy przeglądarka faktycznie znajduje się na danej stronie. Wypełnienie tej metody nie jest konieczne; jednak możesz swobodnie dokonywać tych twierdzeń, jeśli chcesz. Te twierdzenia będą uruchamiane automatycznie podczas nawigacji na stronie:

    /**
     * Assert that the browser is on the page.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### Navigating To Pages - Nawigowanie do stron

Po skonfigurowaniu strony możesz przejść do niej za pomocą metody `visit`:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

Czasami możesz już być na danej stronie i musisz "załadować" selektory i metody strony do bieżącego kontekstu testu. Jest to powszechne po naciśnięciu przycisku i przekierowaniu na daną stronę bez bezpośredniego nawigowania do niego. W tej sytuacji możesz użyć metody `on` do załadowania strony:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### Shorthand Selectors - Skróty selektorów

Metoda `elements` stron umożliwia zdefiniowanie szybkich, łatwych do zapamiętania skrótów dla dowolnego selektora CSS na twojej stronie. Na przykład, zdefiniujmy skrót dla pola wejściowego "email" na stronie logowania aplikacji:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

Teraz możesz użyć tego skróconego selektora w dowolnym miejscu, w którym używałbyś pełnego selektora CSS:

    $browser->type('@email', 'taylor@laravel.com');

#### Global Shorthand Selectors - Globalne skróty selektorów

Po zainstalowaniu Dusk, podstawowa klasa `Page` zostanie umieszczona w twoim katalogu `tests/Browser/Pages`. Ta klasa zawiera metodę `siteElements`, która może być używana do definiowania globalnych skrótowych selektorów, które powinny być dostępne na każdej stronie w aplikacji:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### Page Methods - Metody strony

Oprócz domyślnych metod zdefiniowanych na stronach, możesz zdefiniować dodatkowe metody, które mogą być używane w twoich testach. Na przykład, wyobraźmy sobie, że tworzymy aplikację do zarządzania muzyką. Typową akcją dla jednej strony aplikacji może być utworzenie listy odtwarzania. Zamiast ponownie pisać logikę, aby utworzyć listę odtwarzania w każdym teście, możesz zdefiniować metodę `createPlaylist` na klasie strony:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // Other page methods...

        /**
         * Create a new playlist.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

Po zdefiniowaniu metody możesz jej użyć w dowolnym teście, który wykorzystuje stronę. Instancja przeglądarki zostanie automatycznie przekazana do metody strony:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## Components - Komponenty

Komponenty są podobne do "obiektów strony" Dusk, ale są przeznaczone dla części interfejsu i funkcji, które są ponownie używane w całej aplikacji, takich jak pasek nawigacji lub okno powiadomień. W związku z tym komponenty nie są powiązane z określonymi adresami URL.

<a name="generating-components"></a>
### Generating Components - Generowanie komponentów

Aby wygenerować komponent, użyj polecenia `dusk:component` Artisan. Nowe komponenty są umieszczane w katalogu `test/Browser/Components`:

    php artisan dusk:component DatePicker

Jak pokazano powyżej, "date picker" to przykład komponentu, który może istnieć w całej aplikacji na różnych stronach. Ręczne zapisywanie logiki automatyzacji przeglądarki może być kłopotliwe, aby wybrać datę w dziesiątkach testów w całym zestawie testów. Zamiast tego możemy zdefiniować komponent Dusk do reprezentowania selektora daty, co pozwala nam zamknąć tę logikę w komponencie:

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * Get the root selector for the component.
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }

        /**
         * Assert that the browser page contains the component.
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * Get the element shortcuts for the component.
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * Select the given date.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $month
         * @param  int  $year
         * @return void
         */
        public function selectDate($browser, $month, $year)
        {
            $browser->click('@date-field')
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### Using Components - Używanie komponentów

Po zdefiniowaniu komponentu możemy łatwo wybrać datę z selektora daty z dowolnego testu. A jeśli logika niezbędna do zmiany daty zmieni się, wystarczy zaktualizować komponent:

    <?php

    namespace Tests\Browser;

    use Tests\DuskTestCase;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        /**
         * A basic component test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(1, 2018);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## Continuous Integration - Ciągła integracja

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Aby uruchomić testy Dusk na Travis CI, będziemy musieli użyć środowiska "Ubuntu 14.04 (Trusty)" z obsługą sudo. Ponieważ Travis CI nie jest środowiskiem graficznym, konieczne będzie podjęcie dodatkowych kroków w celu uruchomienia przeglądarki Chrome. Ponadto użyjemy `php artisan serve` do uruchomienia wbudowanego serwera WWW PHP:

    sudo: required
    dist: trusty

    addons:
       chrome: stable

    install:
       - cp .env.testing .env
       - travis_retry composer install --no-interaction --prefer-dist --no-suggest

    before_script:
       - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
       - php artisan serve &

    script:
       - php artisan dusk

<a name="running-tests-on-circle-ci"></a>
### CircleCI

#### CircleCI 1.0

Jeśli używasz CircleCI 1.0 do uruchamiania testów Dusk, możesz użyć tego pliku konfiguracyjnego jako punktu wyjścia. Podobnie jak TravisCI, użyjemy polecenia `php artisan serve` do uruchomienia wbudowanego serwera WWW PHP:

	dependencies:
	  pre:
	      - curl -L -o google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
	      - sudo dpkg -i google-chrome.deb
	      - sudo sed -i 's|HERE/chrome\"|HERE/chrome\" --disable-setuid-sandbox|g' /opt/google/chrome/google-chrome
	      - rm google-chrome.deb

    test:
        pre:
            - "./vendor/laravel/dusk/bin/chromedriver-linux":
                background: true
            - cp .env.testing .env
            - "php artisan serve":
                background: true

        override:
            - php artisan dusk

 #### CircleCI 2.0

Jeśli używasz CircleCI 2.0 do uruchamiania testów Dusk, możesz dodać te kroki do swojej kompilacji:

     version: 2
     jobs:
         build:
             steps:
                - run: sudo apt-get install -y libsqlite3-dev
                - run: cp .env.testing .env
                - run: composer install -n --ignore-platform-reqs
                - run: npm install
                - run: npm run production
                - run: vendor/bin/phpunit

                - run:
                   name: Start Chrome Driver
                   command: ./vendor/laravel/dusk/bin/chromedriver-linux
                   background: true

                - run:
                   name: Run Laravel Server
                   command: php artisan serve
                   background: true

                - run:
                   name: Run Laravel Dusk Tests
                   command: php artisan dusk

<a name="running-tests-on-codeship"></a>
### Codeship

Aby uruchomić testy Dusk w [Codeship](https://codeship.com), dodaj następujące polecenia do swojego projektu Codeship. Oczywiście te polecenia są punktem wyjścia i możesz dodać dodatkowe polecenia w razie potrzeby:

    phpenv local 7.1
    cp .env.testing .env
    composer install --no-interaction
    nohup bash -c "./vendor/laravel/dusk/bin/chromedriver-linux 2>&1 &"
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk
