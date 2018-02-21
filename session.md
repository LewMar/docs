# HTTP Session

- [Introduction - Wprowadzenie](#introduction)
    - [Configuration - Konfiguracja](#configuration)
    - [Driver Prerequisites - Wymagania wstępne sterownika](#driver-prerequisites)
- [Using The Session - Używanie sesji](#using-the-session)
    - [Retrieving Data - Pobieranie danych](#retrieving-data)
    - [Storing Data - Przechowywanie danych](#storing-data)
    - [Flash Data - Migawka danych](#flash-data)
    - [Deleting Data - Usuwanie danych](#deleting-data)
    - [Regenerating The Session ID - Ponowne generowanie identyfikatora sesji](#regenerating-the-session-id)
- [Adding Custom Session Drivers - Dodawanie niestandardowych sterowników sesji](#adding-custom-session-drivers)
    - [Implementing The Driver - Wdrażanie sterownika](#implementing-the-driver)
    - [Registering The Driver - Rejestrowanie sterownika](#registering-the-driver)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Ponieważ aplikacje sterowane przez protokół HTTP są bezstanowe, sesje umożliwiają przechowywanie informacji o użytkowniku w wielu żądaniach. Laravel jest dostarczany z różnymi mechanizmami obsługi sesji, do których dostęp uzyskuje się za pomocą ekspresywnego, zunifikowanego interfejsu API. Obsługa popularnych serwerów pocztowych, takich jak [Memcached](https://memcached.org), [Redis](https://redis.io) i baz danych jest dołączona do zestawu.

<a name="configuration"></a>
### Configuration - Konfiguracja

Plik konfiguracji sesji jest przechowywany w `config/session.php`. Zapoznaj się z dostępnymi opcjami w tym pliku. Domyślnie Laravel jest skonfigurowany do używania sterownika `file` session, który będzie działał dobrze dla wielu aplikacji. W aplikacjach produkcyjnych możesz rozważyć użycie sterowników `memcached` lub` redis` dla jeszcze szybszej wydajności sesji.

Opcja konfiguracyjna sesji `driver` określa miejsce przechowywania danych sesji dla każdego żądania. Laravel jest dostarczony z kilkoma świetnymi draiver-ami:

<div class="content-list" markdown="1">
- `file` - sesje są przechowywane w `storage/framework/sessions`.
- `cookie` - sesje są przechowywane w bezpiecznych, zaszyfrowanych plikach cookie.
- `database` - sesje są przechowywane w relacyjnej bazie danych.
- `memcached` / `redis` - sesje są przechowywane w jednym z tych szybkich magazynów z pamięcią podręczną.
- `array` - sesje są przechowywane w tablicy PHP i nie zostaną utrwalone.
</div>

> {tip} Sterownik tablicowy jest używany podczas [testowania](/docs/{{version}}/testing) i zapobiega utrwalaniu danych przechowywanych w sesji.

<a name="driver-prerequisites"></a>
### Driver Prerequisites - Wymagania wstępne sterownika

#### Database - Baza danych

Korzystając ze sterownika sesji `database`, należy utworzyć tabelę zawierającą elementy sesji. Poniżej znajduje się przykładowa deklaracja `Schema` dla tabeli:

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->unsignedInteger('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

Możesz użyć polecenia `session: table` Artisan, aby wygenerować tę migrację:

    php artisan session:table

    php artisan migrate

#### Redis

Przed rozpoczęciem korzystania z sesji Redis za pomocą Laravel, musisz zainstalować pakiet `predis/predis` (~ 1.0) za pośrednictwem Composer. Możesz skonfigurować połączenia Redis w pliku konfiguracyjnym `database`. W pliku konfiguracyjnym `session` opcja` connection` może być użyta do określenia, które połączenie Redis jest używane przez sesję.

<a name="using-the-session"></a>
## Using The Session - Używanie sesji

<a name="retrieving-data"></a>
### Retrieving Data - Pobieranie danych

Istnieją dwa główne sposoby pracy z danymi sesji w Laravel: globalny pomocnik `session` i instancja` Request`. Po pierwsze, spójrzmy na dostęp do sesji za pomocą instancji `Request`, która może byćwstrzyknięta do metody kontrolera. Pamiętaj, że zależności metody kontrolera są automatycznie wprowadzane za pośrednictwem Laravel [service container (kontener usługi)](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

Po pobraniu elementu z sesji można również przekazać wartość domyślną jako drugi argument metody `get`. Ta wartość domyślna zostanie zwrócona, jeśli określony klucz nie istnieje w sesji. Jeśli przekażesz `Closure` jako domyślną wartość metody `get`, a żądany klucz nie istnieje, `Closure` zostanie wykonane i jego wynik zostanie zwrócony:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### The Global Session Helper - Globalny pomocnik sesji

Możesz również użyć globalnej funkcji `session` PHP do pobierania i przechowywania danych w sesji. Kiedy helper `session` jest wywoływany z pojedynczym argumentem string, zwróci wartość tego klucza sesji. Gdy helper zostanie wywołany z tablicą par klucz / wartość, wartości te zostaną zapisane w sesji:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');

        // Specifying a default value...
        $value = session('key', 'default');

        // Store a piece of data in the session...
        session(['key' => 'value']);
    });

> {tip} Istnieje niewielka praktyczna różnica między używaniem sesji za pośrednictwem instancji żądania HTTP a używaniem globalnego pomocnika `session`. Obie metody są [testowalne](/ docs / {{version}} / testing) za pomocą metody `assertSessionHas`, która jest dostępna we wszystkich testowych przypadkach.

#### Retrieving All Session Data - Pobieranie wszystkich danych sesji

Jeśli chcesz odzyskać wszystkie dane z sesji, możesz użyć metody "all":

    $data = $request->session()->all();

#### Determining If An Item Exists In The Session - Ustalenie, czy dana pozycja istnieje w sesji

Aby ustalić, czy element jest obecny w sesji, możesz użyć metody `has`. Metoda `has` zwraca` true`, jeśli element jest obecny i nie ma wartości `null`:

    if ($request->session()->has('users')) {
        //
    }

Aby określić, czy element jest obecny w sesji, nawet jeśli jego wartość to `null`, możesz użyć metody `exist`. Metoda `exists` zwraca `true`, jeśli element jest obecny:

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### Storing Data - Przechowywanie danych

Aby przechowywać dane w sesji, zazwyczaj używasz metody `put` lub` session` helper-a:

    // Via a request instance...
    $request->session()->put('key', 'value');

    // Via the global helper...
    session(['key' => 'value']);

#### Pushing To Array Session Values - Pchanie do tablicy sesji wartości

Metodę `push` można użyć do przekazania nowej wartości do wartości sesji, która jest tablicą. Na przykład, jeśli klucz `user.teams` zawiera tablicę nazw drużyn, możesz wprowadzić nową wartość do tablicy:

    $request->session()->push('user.teams', 'developers');

#### Retrieving & Deleting An Item - Pobieranie i usuwanie pozycji

Metoda `pull` pobierze i usunie element z sesji w pojedynczej instrukcji:

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### Flash Data - Migawka danych

Czasami możesz chcieć przechowywać przedmioty w sesji tylko dla następnego żądania. Możesz to zrobić za pomocą metody `flash`. Dane przechowywane w sesji przy użyciu tej metody będą dostępne tylko podczas kolejnego żądania HTTP, a następnie zostaną usunięte. Dane Flash są przydatne przede wszystkim w przypadku krótkotrwałych komunikatów o stanie:

    $request->session()->flash('status', 'Task was successful!');

Jeśli chcesz przechowywać migawkę danych dla kilku żądań, możesz użyć metody `reflash`, która zachowa wszystkie dane flash dla dodatkowego żądania. Jeśli potrzebujesz tylko zachować określone dane flash, możesz użyć metody `keep`:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### Deleting Data - Usuwanie danych

Metoda `forget` usunie fragment danych z sesji. Jeśli chcesz usunąć wszystkie dane z sesji, możesz użyć metody `flush`:

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### Regenerating The Session ID - Ponowne generowanie identyfikatora sesji

Ponowne generowanie identyfikatora sesji jest często wykonywane, aby uniemożliwić złośliwym użytkownikom wykorzystywanie ataku [session fixation](https://en.wikipedia.org/wiki/Session_fixation) na twoją aplikację.

Laravel automatycznie generuje ponownie identyfikator sesji podczas uwierzytelniania, jeśli używasz wbudowanego `LoginController`; Jeśli jednak potrzebujesz ręcznie zregenerować identyfikator sesji, możesz użyć metody `regenerate`.

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## Adding Custom Session Drivers - Dodawanie niestandardowych sterowników sesji

<a name="implementing-the-driver"></a>
#### Implementing The Driver - Wdrażanie sterownika

Twój niestandardowy sterownik sesji powinien implementować `SessionHandlerInterface`. Ten interfejs zawiera tylko kilka prostych metod, które musimy wdrożyć. Wycinek implementacja MongoDB wygląda mniej więcej tak:

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} Laravel nie jest dostarczany z katalogiem zawierającym rozszerzenia. Możesz je umieścić w dowolnym miejscu. W tym przykładzie utworzyliśmy katalog `Extensions`, w którym znajduje się `MongoSessionHandler`.

Ponieważ cel tych metod nie jest łatwy do zrozumienia, szybko omówmy, co każda z metod robi:

<div class="content-list" markdown="1">
- Metoda `open` byłaby zazwyczaj używana w systemach plików sesji opartych na plikach. Ponieważ Laravel jest dostarczany ze sterownikiem sesji `file`, prawie nigdy nie będziesz musiał umieszczać czegokolwiek w tej metodzie. Możesz zostawić go jako pusty stub. Fakt, że PHP wymaga od nas wdrożenia tej metody, jest kiepskim projektem interfejsu (który omówimy później).
- Metodę `close`, podobnie jak metodę `open`, można również zignorować. Dla większości sterowników nie jest to konieczne.
- Metoda `read` powinna zwrócić wersję string danych sesji związanych z danym `$sessionId`. Podczas pobierania lub przechowywania danych sesji w sterowniku nie trzeba wykonywać żadnych serializacji ani kodowania, ponieważ Laravel wykona dla ciebie serializację.
- Metoda `write` powinna zapisać podany ciąg `$data` powiązany z `$sessionId` z jakimś trwałym systemem pamięci masowej, takim jak MongoDB, Dynamo, itp. Ponownie, nie powinieneś wykonywać żadnej serializacji - Laravel będzie już obsługiwać to za ciebie.
- Metoda `destroy` powinna usunąć dane związane z `$sessionId` z trwałej pamięci.
- Metoda `gc` powinna zniszczyć wszystkie dane sesji, które są starsze niż podane `$lifetime`, które jest znacznikiem UNIX. W przypadku systemów, które wygasają, takich jak Memcached i Redis, ta metoda może pozostać pusta.
</div>

<a name="registering-the-driver"></a>
#### Registering The Driver - Rejestrowanie sterownika

Po zaimplementowaniu sterownika jesteś gotowy do zarejestrowania go w framework-u. Aby dodać dodatkowe sterowniki do zaplecza sesji Laravel, możesz użyć metody `extend` na `Session` [elewacji(fasady)](/ docs/{{version}}/fasady). Powinieneś wywołać metodę `extend` z metody` boot` [dostawcy usług](/ docs/{{version}}/providers). Możesz to zrobić z istniejącego `AppServiceProvider` lub stworzyć całkowicie nowego dostawcę:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionHandler;
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Po zarejestrowaniu sterownika sesji możesz użyć sterownika `mongo` w swoim pliku konfiguracyjnym `config/session.php`.