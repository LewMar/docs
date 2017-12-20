# Contracts

- [Introduction - Wprowadzienie](#introduction)
    - [Contracts Vs. Facades - Kontrakty kontra Fasady](#contracts-vs-facades)
- [When To Use Contracts - Kiedy używac kontraktów](#when-to-use-contracts)
    - [Loose Coupling - Luźne połączenie](#loose-coupling)
    - [Simplicity - Prostota](#simplicity)
- [How To Use Contracts - Jak używać kontraktów](#how-to-use-contracts)
- [Contract Reference - Referencje kontraktów](#contract-reference)

<a name="introduction"></a>
## Introduction - Wprowadzienie

Kontrakty Laravel to zestaw interfejsów definiujących podstawowe usługi świadczone przez framework. Na przykład umowa `Illuminate\Contracts\Queue\Queue` definiuje metody potrzebne do kolejkowania zadań, a umowa`Illuminate\Contracts\Mail\Mailer` określa metody potrzebne do wysyłania wiadomości e-mail.

Każda umowa ma odpowiednią implementację zapewnioną przez framework. Na przykład Laravel zapewnia implementację kolejki z różnymi sterownikami oraz implementację programu pocztowego obsługiwaną przez [SwiftMailer](https://swiftmailer.symfony.com/).

Wszystkie kontrakty Laravel mieszkają w [ich własnym repozytorium GitHub](https://github.com/illuminate/contracts). Zapewnia to szybki punkt odniesienia dla wszystkich dostępnych umów, a także pojedynczy, oddzielony od produkcji pakiet, który może być wykorzystywany przez twórców pakietów.

<a name="contracts-vs-facades"></a>
### Contracts Vs. Facades - Kontrakty kontra Fasady

[Fasady](/docs/{{version}}/facades) Laravel-a i funkcje pomocnicze zapewniają prosty sposób korzystania z usług Laravel-a bez konieczności wpisywania wskazówki  i rozwiązywania kontraktów z kontenera usług. W większości przypadków każda fasada ma równoważną umowę.

W odróżnieniu od fasad, które nie wymagają od ciebie aby wymagać w konstruktorze klasy, kontrakty pozwalają ci definiować jawne zależności dla twoich klas. Niektórzy programiści wolą wyraźnie określać swoje zależności w ten sposób i dlatego wolą korzystać z kontraktów, podczas gdy inni deweloperzy cieszą się wygodą fasad.

> {tip} Większość aplikacji będzie wysokiej jakości, bez względu na to, czy preferujesz fasady czy kontrakty. Jednakże, jeśli budujesz pakiet, powinieneś zdecydowanie rozważyć użycie kontraktów, ponieważ łatwiej będzie je przetestować w kontekście pakietu.

<a name="when-to-use-contracts"></a>
## When To Use Contracts - Kiedy używac kontraktów

Jak zostało to omówione gdzie indziej, znaczna część decyzji o korzystaniu z kontraktów lub fasad sprowadza się do osobistego gustu i gustu zespołu programistów. Zarówno kontrakty, jak i fasady można wykorzystywać do tworzenia solidnych, dobrze przetestowanych aplikacji Laravel. Tak długo, jak długo będziesz koncentrował się na swoich zadaniach, zauważysz bardzo niewiele praktycznych różnic między korzystaniem z kontraktów i fasad.

Jednak wciąż możesz mieć kilka pytań dotyczących umów. Na przykład po co używać interfejsów? Czy korzystanie z interfejsów nie jest bardziej skomplikowane? Wyjaśnijmy powody używania interfejsów do następujących pozycji: luźne połączenie(loose coupling) i prostota(simplicity).

<a name="loose-coupling"></a>
### Loose Coupling - Luźne połączenie

Najpierw przejrzyjmy kod, który jest ściśle powiązany z implementacją pamięci podręcznej. Rozważ następujące:

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

W tej klasie kod jest ściśle powiązany z daną implementacją pamięci podręcznej. Jest ściśle sprzężony, ponieważ zależymy od konkretnej klasy pamięci podręcznej od dostawcy pakietu. Jeśli API tego pakietu zmieni się, nasz kod również musi się zmienić.

Podobnie, jeśli chcemy zastąpić naszą podstawową technologię pamięci podręcznej (Memcached) inną technologią (Redis), ponownie będziemy musieli zmodyfikować nasze repozytorium. Nasze repozytorium nie powinno posiadać tak dużej wiedzy na temat tego, kto dostarcza dane lub jak je udostępnia.

**Zamiast tego podejścia możemy ulepszyć nasz kod, w zależności od prostego, agnostycznego interfejsu dostawcy:**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

Teraz kod nie jest powiązany z żadnym konkretnym dostawcą, a nawet z Laravel. Ponieważ pakiet umów nie zawiera implementacji i nie ma zależności, możesz łatwo napisać alternatywną implementację dowolnej umowy, umożliwiając zastąpienie implementacji pamięci podręcznej bez modyfikowania kodu klasy kożystajacej z tej pamieći podręcznej.

<a name="simplicity"></a>
### Simplicity - Prostota

Gdy wszystkie usługi Laravel są starannie zdefiniowane w ramach prostych interfejsów, bardzo łatwo jest określić funkcjonalność oferowaną przez daną usługę. **Kontrakty służą jako zwięzła dokumentacja cech framework-a**

Ponadto, kiedy zależy Ci na prostych interfejsach, Twój kod jest łatwiejszy do zrozumienia i utrzymania. Zamiast wybierać, które metody są dostępne w dużej, skomplikowanej klasie, możesz odwołać się do prostego, przejrzystego interfejsu.

<a name="how-to-use-contracts"></a>
## How To Use Contracts - Jak używać kontraktów

Jak zatem uzyskać implementacje kontraktu? T całkiem proste.

Wiele typów klas w Laravel jest rozwiązywanych za pośrednictwem [kontenera usług (service container)](/docs/{{version}}/container) / container), w tym kontrolerów(controllers), detektorów zdarzeń(event listeners), oprogramowania pośredniego(middleware), zadań w kolejce(queued jobs), a nawet zamknięć trasy(route Closures). Tak więc, aby uzyskać implementację umowy, można po prostu "wpisać wskazówkę" interfejsu w konstruktorze rozwiązywanej klasy.

Na przykład spójrz na ten detektor zdarzeń:

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Database;

    class CacheOrderInformation
    {
        /**
         * The Redis database implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

Po rozpoznaniu detektora zdarzeń (event listener) kontener usług odczytuje wskazówki typu konstruktora klasy i wstrzykuje odpowiednią wartość. Aby dowiedzieć się więcej na temat rejestrowania rzeczy w kontenerze usług, sprawdź [jego dokumentację](/docs/{{version}}/container).

<a name="contract-reference"></a>
## Contract Reference - Referencje kontraktów

Poniższa tabela zawiera krótkie odniesienia do wszystkich umów Laravel i ich równoważnych fasad:

Contract  |  References Facade
------------- | -------------
[Illuminate\Contracts\Auth\Access\Authorizable](https://github.com/illuminate/contracts/blob/5.5/Auth/Access/Authorizable.php) | &nbsp;
[Illuminate\Contracts\Auth\Access\Gate](https://github.com/illuminate/contracts/blob/5.5/Auth/Access/Gate.php) | `Gate`
[Illuminate\Contracts\Auth\Authenticatable](https://github.com/illuminate/contracts/blob/5.5/Auth/Authenticatable.php) | &nbsp;
[Illuminate\Contracts\Auth\CanResetPassword](https://github.com/illuminate/contracts/blob/5.5/Auth/CanResetPassword.php) | &nbsp;
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/5.5/Auth/Factory.php) | `Auth`
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/5.5/Auth/Guard.php) | `Auth::guard()`
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/5.5/Auth/PasswordBroker.php) | `Password::broker()`
[Illuminate\Contracts\Auth\PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/5.5/Auth/PasswordBrokerFactory.php) | `Password`
[Illuminate\Contracts\Auth\StatefulGuard](https://github.com/illuminate/contracts/blob/5.5/Auth/StatefulGuard.php) | &nbsp;
[Illuminate\Contracts\Auth\SupportsBasicAuth](https://github.com/illuminate/contracts/blob/5.5/Auth/SupportsBasicAuth.php) | &nbsp;
[Illuminate\Contracts\Auth\UserProvider](https://github.com/illuminate/contracts/blob/5.5/Auth/UserProvider.php) | &nbsp;
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/5.5/Bus/Dispatcher.php) | `Bus`
[Illuminate\Contracts\Bus\QueueingDispatcher](https://github.com/illuminate/contracts/blob/5.5/Bus/QueueingDispatcher.php) | `Bus::dispatchToQueue()`
[Illuminate\Contracts\Broadcasting\Factory](https://github.com/illuminate/contracts/blob/5.5/Broadcasting/Factory.php) | `Broadcast`
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/5.5/Broadcasting/Broadcaster.php)  | `Broadcast::connection()`
[Illuminate\Contracts\Broadcasting\ShouldBroadcast](https://github.com/illuminate/contracts/blob/5.5/Broadcasting/ShouldBroadcast.php) | &nbsp;
[Illuminate\Contracts\Broadcasting\ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/5.5/Broadcasting/ShouldBroadcastNow.php) | &nbsp;
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/5.5/Cache/Factory.php) | `Cache`
[Illuminate\Contracts\Cache\Lock](https://github.com/illuminate/contracts/blob/5.5/Cache/Lock.php) | &nbsp;
[Illuminate\Contracts\Cache\LockProvider](https://github.com/illuminate/contracts/blob/5.5/Cache/LockProvider.php) | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/5.5/Cache/Repository.php) | `Cache::driver()`
[Illuminate\Contracts\Cache\Store](https://github.com/illuminate/contracts/blob/5.5/Cache/Store.php) | &nbsp;
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/5.5/Config/Repository.php) | `Config`
[Illuminate\Contracts\Console\Application](https://github.com/illuminate/contracts/blob/5.5/Console/Application.php) | &nbsp;
[Illuminate\Contracts\Console\Kernel](https://github.com/illuminate/contracts/blob/5.5/Console/Kernel.php) | `Artisan`
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/5.5/Container/Container.php) | `App`
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/5.5/Cookie/Factory.php) | `Cookie`
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/5.5/Cookie/QueueingFactory.php) | `Cookie::queue()`
[Illuminate\Contracts\Database\ModelIdentifier](https://github.com/illuminate/contracts/blob/5.5/Database/ModelIdentifier.php) | &nbsp;
[Illuminate\Contracts\Debug\ExceptionHandler](https://github.com/illuminate/contracts/blob/5.5/Debug/ExceptionHandler.php) | &nbsp;
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/5.5/Encryption/Encrypter.php) | `Crypt`
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/5.5/Events/Dispatcher.php) | `Event`
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/5.5/Filesystem/Cloud.php) | `Storage::cloud()`
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/5.5/Filesystem/Factory.php) | `Storage`
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/5.5/Filesystem/Filesystem.php) | `Storage::disk()`
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/5.5/Foundation/Application.php) | `App`
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/5.5/Hashing/Hasher.php) | `Hash`
[Illuminate\Contracts\Http\Kernel](https://github.com/illuminate/contracts/blob/5.5/Http/Kernel.php) | &nbsp;
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/5.5/Logging/Log.php) | `Log`
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/5.5/Mail/MailQueue.php) | `Mail::queue()`
[Illuminate\Contracts\Mail\Mailable](https://github.com/illuminate/contracts/blob/5.5/Mail/Mailable.php) | &nbsp;
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/5.5/Mail/Mailer.php) | `Mail`
[Illuminate\Contracts\Notifications\Dispatcher](https://github.com/illuminate/contracts/blob/5.5/Notifications/Dispatcher.php) | `Notification`
[Illuminate\Contracts\Notifications\Factory](https://github.com/illuminate/contracts/blob/5.5/Notifications/Factory.php) | `Notification`
[Illuminate\Contracts\Pagination\LengthAwarePaginator](https://github.com/illuminate/contracts/blob/5.5/Pagination/LengthAwarePaginator.php) | &nbsp;
[Illuminate\Contracts\Pagination\Paginator](https://github.com/illuminate/contracts/blob/5.5/Pagination/Paginator.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Hub](https://github.com/illuminate/contracts/blob/5.5/Pipeline/Hub.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Pipeline](https://github.com/illuminate/contracts/blob/5.5/Pipeline/Pipeline.php) | &nbsp;
[Illuminate\Contracts\Queue\EntityResolver](https://github.com/illuminate/contracts/blob/5.5/Queue/EntityResolver.php) | &nbsp;
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/5.5/Queue/Factory.php) | `Queue`
[Illuminate\Contracts\Queue\Job](https://github.com/illuminate/contracts/blob/5.5/Queue/Job.php) | &nbsp;
[Illuminate\Contracts\Queue\Monitor](https://github.com/illuminate/contracts/blob/5.5/Queue/Monitor.php) | `Queue`
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/5.5/Queue/Queue.php) | `Queue::connection()`
[Illuminate\Contracts\Queue\QueueableCollection](https://github.com/illuminate/contracts/blob/5.5/Queue/QueueableCollection.php) | &nbsp;
[Illuminate\Contracts\Queue\QueueableEntity](https://github.com/illuminate/contracts/blob/5.5/Queue/QueueableEntity.php) | &nbsp;
[Illuminate\Contracts\Queue\ShouldQueue](https://github.com/illuminate/contracts/blob/5.5/Queue/ShouldQueue.php) | &nbsp;
[Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/5.5/Redis/Factory.php) | `Redis`
[Illuminate\Contracts\Routing\BindingRegistrar](https://github.com/illuminate/contracts/blob/5.5/Routing/BindingRegistrar.php) | `Route`
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/5.5/Routing/Registrar.php) | `Route`
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/5.5/Routing/ResponseFactory.php) | `Response`
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/5.5/Routing/UrlGenerator.php) | `URL`
[Illuminate\Contracts\Routing\UrlRoutable](https://github.com/illuminate/contracts/blob/5.5/Routing/UrlRoutable.php) | &nbsp;
[Illuminate\Contracts\Session\Session](https://github.com/illuminate/contracts/blob/5.5/Session/Session.php) | `Session::driver()`
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/5.5/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Htmlable](https://github.com/illuminate/contracts/blob/5.5/Support/Htmlable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/5.5/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\MessageBag](https://github.com/illuminate/contracts/blob/5.5/Support/MessageBag.php) | &nbsp;
[Illuminate\Contracts\Support\MessageProvider](https://github.com/illuminate/contracts/blob/5.5/Support/MessageProvider.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/5.5/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Support\Responsable](https://github.com/illuminate/contracts/blob/5.5/Support/Responsable.php) | &nbsp;
[Illuminate\Contracts\Translation\Loader](https://github.com/illuminate/contracts/blob/5.5/Translation/Loader.php) | &nbsp;
[Illuminate\Contracts\Translation\Translator](https://github.com/illuminate/contracts/blob/5.5/Translation/Translator.php) | `Lang`
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/5.5/Validation/Factory.php) | `Validator`
[Illuminate\Contracts\Validation\ImplicitRule](https://github.com/illuminate/contracts/blob/5.5/Validation/ImplicitRule.php) | &nbsp;
[Illuminate\Contracts\Validation\Rule](https://github.com/illuminate/contracts/blob/5.5/Validation/Rule.php) | &nbsp;
[Illuminate\Contracts\Validation\ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/5.5/Validation/ValidatesWhenResolved.php) | &nbsp;
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/5.5/Validation/Validator.php) | `Validator::make()`
[Illuminate\Contracts\View\Engine](https://github.com/illuminate/contracts/blob/5.5/View/Engine.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/5.5/View/Factory.php) | `View`
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/5.5/View/View.php) | `View::make()`
