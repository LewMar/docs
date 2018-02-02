# Service Container - Kontener usług

- [Introduction - Wprowadzenie](#introduction)
- [Binding - Powiązanie](#binding)
    - [Binding Basics - Podstawowe powiązania](#binding-basics)
    - [Binding Interfaces To Implementations - Powiązanie interfejsów z implementacjami](#binding-interfaces-to-implementations)
    - [Contextual Binding - Powiązanie kontekstowe](#contextual-binding)
    - [Tagging -  Tagowanie](#tagging)
    - [Extending Bindings - Rozszerzanie powiązań](#extending-bindings)
- [Resolving -  Rozwiązywanie](#resolving)
    - [The Make Method - Metoda Make](#the-make-method)
    - [Automatic Injection - Automatyczne wstrzykiwanie](#automatic-injection)
- [Container Events - Zdarzenia kontenera](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Kontener usług Laravel to zaawansowane narzędzie do zarządzania zależnościami klas i wykonywania iniekcji zależności. Wstrzykiwanie zależności to fikcyjne wyrażenie, które zasadniczo oznacza to: zależności klas są "injected"(wstrzykiwane) do klasy za pośrednictwem konstruktora lub, w niektórych przypadkach, metod  "setter"(ustawiania).

Spójrzmy na prosty przykład:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

W tym przykładzie `UserController` potrzebuje pobrać użytkowników ze źródła danych. Dlatego **inject(wstrzykniemy)** usługę, która będzie w stanie pobrać użytkowników. W tym kontekście nasz `UserRepository` najprawdopodobniej używa [Eloquent](/docs/{{version}}/eloquent) do pobierania informacji o użytkowniku z bazy danych. Ponieważ jednak repozytorium jest wstrzykiwane, możemy je łatwo zamienić na inną implementację.Możemy również łatwo "mock"(symulować), lub stworzyć fałszywą implementację `UserRepository` podczas testowania naszej aplikacji.

Dogłębne zrozumienie kontenera usług Laravel jest niezbędne do zbudowania potężnej, dużej aplikacji, a także do wniesienia wkładu w samo jądro Laravel.

<a name="binding"></a>
## Binding - Powiązanie

<a name="binding-basics"></a>
### Binding Basics - Podstawowe powiązania

Prawie wszystkie twoje powiązania kontenerów usługowych będą rejestrowane w usługodawcach [service providers (dostawców usługi)](/docs/{{version}}/providers), więc większość z tych przykładów pokaże użycie kontenera w tym kontekście.

> {tip} Nie ma potrzeby wiązania klas do kontenera, jeśli nie zależą one od żadnego interfejsu. Kontener nie musi być instruowany, jak budować te obiekty, ponieważ może automatycznie rozwiązać te obiekty za pomocą odbicia.

#### Simple Bindings - Proste powiązania.

W usługodawcy zawsze masz dostęp do kontenera za pośrednictwem właściwości `$this->app`. Możemy zarejestrować wiązanie za pomocą metody `bind`, które chcemy zarejestrować wraz z `Closure (zamknięciem)`, które zwraca instancję klasy:

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

Zwróć uwagę, że otrzymujemy sam kontener jako argument dla resolvera. Następnie możemy użyć kontenera do rozwiązania zależnych od siebie obiektów, które budujemy.

#### Binding A Singleton - Powiązanie Singleton-a (Klasa która ma tylko jedną instancję)

Metoda `singleton` wiąże klasę lub interfejs z kontenerem, który powinien zostać rozwiązany tylko raz. Po rozwiązaniu pojedynczego powiązania ta sama instancja obiektu zostanie zwrócona podczas kolejnych wywołań do kontenera:

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### Binding Instances - Powiązanie instancji

Możesz również powiązać istniejącą instancję obiektu z kontenerem za pomocą metody `instance (instancji)`. Podana instancja zawsze będzie zwracana podczas kolejnych wywołań do kontenera:

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### Binding Primitives - Powiązania surowe

Czasami możesz mieć klasę, która otrzymuje kilka wstrzykniętych klas, ale także potrzebuje wstrzykniętej wartości surowej, takiej jak liczba całkowita. Możesz łatwo użyć powiązania kontekstowego do wstrzyknięcia dowolnej wartości, której może potrzebować twoja Klasa:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### Binding Interfaces To Implementations - Powiązanie interfejsów z implementacjami

Bardzo ważną cechą kontenera usług jest możliwość powiązania interfejsu z daną implementacją. Na przykład załóżmy, że mamy interfejs `EventPusher` i `RedisEventPusher`. Po zakodowaniu naszej implementacji `RedisEventPusher` tego interfejsu, możemy zarejestrować go w kontenerze usług, tak jak to:

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

Ta instrukcja mówi kontenerowi, że powinien wstrzyknąć `RedisEventPusher`, gdy klasa potrzebuje implementacji` EventPusher`. Teraz możemy wpisać-podpowiedź do interfejsu `EventPusher` w konstruktorze lub dowolnej innej lokalizacji, w której zależności są wstrzykiwane przez kontener usługi:

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Contextual Binding - Powiązanie kontekstowe

Czasami możesz mieć dwie klasy, które wykorzystują ten sam interfejs, ale chcesz wprowadzić różne implementacje do każdej klasy. Na przykład dwóch kontrolerów może zależeć od różnych implementacji `Illuminate\Contracts\Filesystem\Filesystem` [contract (kontrakt)](/docs/{{version}}/contracts). Laravel zapewnia prosty, płynny interfejs do definiowania tego zachowania:

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### Tagging - Tagowanie

Czasami może zajść potrzeba rozwiązania wszystkich określonych "category" (kategorii) powiązań. Na przykład, być może budujesz agregator raportów, który otrzymuje tablicę wielu różnych implementacji interfejsu `Report`. Po zarejestrowaniu implementacji `Report` można przypisać im znacznik za pomocą metody `tag`:

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Po oznaczeniu usług możesz je łatwo rozwiązać za pomocą metody `tagged`:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="extending-bindings"></a>
### Extending Bindings - Rozszerzanie powiązań

Metoda `extend` pozwala na modyfikację rozwiązanych usług. Na przykład, gdy usługa zostanie rozwiązana, możesz uruchomić dodatkowy kod do dekorowania lub konfiguracji usługi. Metoda `extend` akceptuje closure, które powinno zwrócić zmodyfikowaną usługę, jako jedyny argument:

    $this->app->extend(Service::class, function($service) {
        return new DecoratedService($service);
    });

<a name="resolving"></a>
## Resolving - Rozwiązywanie

<a name="the-make-method"></a>
#### The `make` method - Metoda `make`

Możesz użyć metody `make` do rozwiązania instancji klasy poza kontenerem. Metoda `make` akceptuje nazwę klasy lub interfejsu, które chcesz rozwiązać:

    $api = $this->app->make('HelpSpot\API');

If you are in a location of your code that does not have access to the `$app` variable, you may use the global `resolve` helper:

    $api = resolve('HelpSpot\API');

Jeśli w swoim kodzie jesteś w miejscu, które nie ma dostępu do zmiennej `$app`, możesz użyć globalnego helpera` resolve`:

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>
#### Automatic Injection - Automatyczne wstrzykiwanie

Alternatywnie i co ważniejsze, możesz "wpisać wskazówkę" zależności w konstruktorze klasy, która jest rozwiązana przez kontener, w tym [kontrolery](/docs/{{version}}/controllers), [detektory zdarzeń](/docs/{{version}}/events), [zadania kolejki](/docs/{{version}}/queues), [oprogramowanie pośrednie](/docs/{{version}}/middleware) i inne. W praktyce w ten sposób większość obiektów powinna zostać rozwiązana przez kontener.

Na przykład możesz wpisać wskazówkę do repozytorium zdefiniowanego przez twoją aplikację w konstruktorze kontrolera. Repozytorium zostanie automatycznie rozpoznane i wstrzyknięte do klasy:

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## Container Events - Zdarzenia kontenera

Kontener usług uruchamia zdarzenie za każdym razem, gdy rozpatruje obiekt. Możesz odsłuchać to wydarzenie, używając metody `resolving`:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });

Jak widać, rozpatrywany obiekt zostanie przekazany do wywołania zwrotnego, umożliwiając ustawienie dowolnych dodatkowych właściwości obiektu przed jego przekazaniem konsumentowi.

<a name="psr-11"></a>
## PSR-11

Kontener usługowy Laravel implementuje interfejs [PSR-11] (https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Dlatego możesz wpisać - wskazówkę interfejsu kontenera PSR-11, aby uzyskać instancję kontenera Laravel:

    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');

        //
    });

> {note} Wywołanie metody `get` spowoduje zgłoszenie wyjątku, jeśli identyfikator nie został jawnie powiązany z kontenerem.
