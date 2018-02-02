# Events

- [Introduction - Wprowadzenie](#introduction)
- [Registering Events & Listeners - Rejestrowanie zdarzeń i słuchaczy](#registering-events-and-listeners)
    - [Generating Events & Listeners - Generowanie zdarzeń i słuchaczy](#generating-events-and-listeners)
    - [Manually Registering Events - Ręczne rejestrowanie zdarzeń](#manually-registering-events)
- [Defining Events - Definiowanie zdarzeń](#defining-events)
- [Defining Listeners - Definiowanie słuchaczy](#defining-listeners)
- [Queued Event Listeners - Obserwatorzy kolejkowania zdarzeń](#queued-event-listeners)
    - [Manually Accessing The Queue - Ręczny dostęp do kolejki](#manually-accessing-the-queue)
    - [Handling Failed Jobs - Obsługa nieudanych zadań](#handling-failed-jobs)
- [Dispatching Events - Wysyłanie zdarzeń](#dispatching-events)
- [Event Subscribers - Subskrybenci zdarzeń](#event-subscribers)
    - [Writing Event Subscribers - Pisanie subskrybentów zdarzeń](#writing-event-subscribers)
    - [Registering Event Subscribers - Pisanie subskrybentów zdarzeń](#registering-event-subscribers)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Wydarzenia Laravel zapewniają prostą implementację obserwatora, umożliwiającą subskrybowanie i słuchanie różnych zdarzeń występujących w aplikacji. Klasy zdarzeń są zwykle przechowywane w katalogu `app/Events`, a ich detektory są przechowywane w `app/Listeners`. Nie martw się, jeśli nie widzisz tych katalogów w aplikacji, ponieważ zostaną one utworzone dla Ciebie podczas generowania zdarzeń i słuchaczy przy użyciu poleceń konsoli Artisan.

Zdarzenia to świetny sposób na rozłączenie różnych aspektów aplikacji, ponieważ jedno wydarzenie może mieć wielu słuchaczy, którzy nie są od siebie zależni. Na przykład możesz wysłać do użytkownika powiadomienie o zaleganiu za każdym razem, gdy zamówienie zostanie wysłane. Zamiast łączenia kodu przetwarzania zamówienia z kodem powiadomień Slack, możesz wywołać zdarzenie `OrderShipped`, które odbiorca może otrzymać i przekształcić w powiadomienie Slack.

<a name="registering-events-and-listeners"></a>
## Registering Events & Listeners - Rejestrowanie zdarzeń i słuchaczy

`EventServiceProvider` dołączony do aplikacji Laravel zapewnia wygodne miejsce do rejestrowania wszystkich detektorów zdarzeń aplikacji. Właściwość `listen` zawiera tablicę wszystkich zdarzeń (kluczy) i ich detektorów (wartości). Oczywiście możesz dodać tyle wydarzeń do tej tablicy, ile wymaga twoja aplikacja. Na przykład dodajmy wydarzenie `OrderShipped`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### Generating Events & Listeners - Generowanie zdarzeń i słuchaczy

Oczywiście ręczne tworzenie plików dla każdego zdarzenia i słuchacza jest uciążliwe. Zamiast tego dodaj detektory i zdarzenia do `EventServiceProvider` i użyj polecenia `event:generate`. To polecenie wygeneruje wszelkie zdarzenia lub detektory, które są wymienione w `EventServiceProvider`. Oczywiście wydarzenia i słuchacze, które już istnieją, pozostaną nietknięte:

    php artisan event:generate

<a name="manually-registering-events"></a>
### Manually Registering Events - Ręczne rejestrowanie zdarzeń

Zazwyczaj zdarzenia powinny być rejestrowane za pomocą tablicy `EventServiceProvider` `$listen`; jednak można również zarejestrować zdarzenia oparte na Closure ręcznie w metodzie `boot` swojego `EventServiceProvider`:

    /**
     * Register any other events for your application.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### Wildcard Event Listeners -Wieloznaczne zdarzenia słuchaczy

Możesz nawet zarejestrować słuchaczy, używając parametru `*` jako parametru wieloznacznego, umożliwiającego przechwytywanie wielu zdarzeń na tym samym odbiorniku. Symbole wieloznaczne odbierają nazwę zdarzenia jako swój pierwszy argument, a cała tablica danych zdarzenia jest ich drugim argumentem:

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="defining-events"></a>
## Defining Events - Definiowanie zdarzeń

Klasa zdarzenia jest kontenerem danych, który przechowuje informacje związane ze zdarzeniem. Na przykład załóżmy, że nasze wygenerowane zdarzenie `OrderShipped` otrzymuje obiekt [Eloquent ORM](/docs/{{version}}/eloquent):

    <?php

    namespace App\Events;

    use App\Order;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use SerializesModels;

        public $order;

        /**
         * Create a new event instance.
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

Jak widać, ta klasa zdarzeń nie zawiera żadnej logiki. Jest to kontener dla zakupionej instancji `Order`. Cecha `SerializesModels` używana przez zdarzenie będzie z gracją serializować dowolne modele Eloquent, jeśli obiekt zdarzenia jest serializowany za pomocą funkcji PHP  `serialize`.

<a name="defining-listeners"></a>
## Defining Listeners - Definiowanie słuchaczy

Następnie przyjrzyjmy się słuchaczowi naszego przykładowego wydarzenia. Detektory zdarzeń odbierają instancję zdarzenia w swojej metodzie `handle`. Komenda `event: generate` automatycznie zaimportuje odpowiednią klasę zdarzenia i poda wpisać wskazówkę zdarzeniu w metodzie `handle`. W ramach metody `handle` możesz wykonać wszystkie działania niezbędne do udzielenia odpowiedzi na zdarzenie:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }

> {tip} Twoje detektory zdarzeń mogą również wpisywać wskazówki, których potrzebują na swoich konstruktorach. Wszystkie detektory zdarzeń są rozwiązywane za pośrednictwem Laravel [kontener usługi](/docs/{{version}}/container), więc zależności będą automatycznie wprowadzane.

#### Stopping The Propagation Of An Event - Zatrzymianie rozprzestrzeniania się zdarzenia

Czasami możesz chcieć zatrzymać rozprzestrzeniania się wydarzenia innym słuchaczom. Możesz to zrobić, zwracając `false` z metody `handle` twojego słuchacza.

<a name="queued-event-listeners"></a>
## Queued Event Listeners - Obserwatorzy kolejkowania zdarzeń

Kolejkowanie słuchaczy może być korzystne, jeśli słuchacz wykonuje powolne zadanie, takie jak wysłanie wiadomości e-mail lub żądanie HTTP. Przed rozpoczęciem korzystania z oczekujących w kolejce programów nasłuchujących, upewnij się, że [skonfigurowałeś swoją kolejkę](/docs/{{version}}/queues) i uruchom program nasłuchujący kolejki na serwerze lub lokalnym środowisku programistycznym.

Aby określić, że detektor powinien być w kolejce, dodaj interfejs `ShouldQueue` do klasy listener. Słuchacze generowani przez komendę `event: generate` Artisan mają już ten interfejs zaimportowany do bieżącego obszaru nazw, więc możesz go użyć natychmiast:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

I to wszystko! Teraz, gdy ten detektor zostanie wezwany do zdarzenia, zostanie automatycznie umieszczony w kolejce przez dyspozytor zdarzeń za pomocą [systemu kolejki](/docs/{{version}}/queues) Laravel. Jeśli nie są zgłaszane żadne wyjątki, gdy detektor jest wykonywany przez kolejkę, kolejkowane zadanie zostanie automatycznie usunięte po zakończeniu przetwarzania.

#### Customizing The Queue Connection & Queue Name - Dostosowywanie połączenia kolejki i nazwy kolejki

Jeśli chcesz dostosować połączenie kolejki i nazwę kolejki używane przez detektor zdarzeń, możesz zdefiniować właściwości `$connection` i `$queue` na swojej klasie listener:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * The name of the connection the job should be sent to.
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * The name of the queue the job should be sent to.
         *
         * @var string|null
         */
        public $queue = 'listeners';
    }

<a name="manually-accessing-the-queue"></a>
### Manually Accessing The Queue - Ręczny dostęp do kolejki

If you need to manually access the listener's underlying queue job's `delete` and `release` methods, you may do so using the `Illuminate\Queue\InteractsWithQueue` trait. This trait is imported by default on generated listeners and provides access to these methods:
Jeśli potrzebujesz ręcznie uzyskać dostęp do metod `delete` i `release` bazowego instrumentu wsadowego odbiornika, możesz to zrobić, używając metody `Illuminate\Queue\InteractsWithQueue`. Ta metoda jest domyślnie importowana do generowanych detektorów i zapewnia dostęp do tych metod:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="handling-failed-jobs"></a>
### Handling Failed Jobs - Obsługa nieudanych zadań

Czasami oczekujące detektory zdarzeń mogą się nie powieść. Jeśli kolejkowany program nasłuchujący przekroczy maksymalną liczbę prób zdefiniowaną przez moduł kolejki, na detektorze zostanie wywołana metoda `failed`. Metoda `failed` odbiera instancję zdarzenia i wyjątek, który spowodował niepowodzenie:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }

        /**
         * Handle a job failure.
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Exception  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>
## Dispatching Events - Wysyłanie zdarzeń

Aby wysłać zdarzenie, możesz przekazać wystąpienie zdarzenia do helpera `event`. Pomocnik wyśle wydarzenie wszystkim zarejestrowanym słuchaczom. Ponieważ helper `event` jest dostępny globalnie, możesz go wywołać z dowolnego miejsca w aplikacji:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // Order shipment logic...

            event(new OrderShipped($order));
        }
    }

> {tip} When testing, it can be helpful to assert that certain events were dispatched without actually triggering their listeners. Laravel's [built-in testing helpers](/docs/{{version}}/mocking#event-fake) makes it a cinch.
Podczas testowania może być pomocne twierdzenie, że pewne zdarzenia zostały wysłane bez faktycznego wyzwalania ich słuchaczy.[Wbudowane narzędzia testujące](/docs/{{version}}/mocking#event-fake) Laravel-a sprawia, że jest to bardzo proste.

<a name="event-subscribers"></a>
## Event Subscribers - Subskrybenci zdarzeń

<a name="writing-event-subscribers"></a>
### Writing Event Subscribers - Pisanie subskrybentów zdarzeń

Subskrybenci zdarzeń to klasy, które mogą subskrybować wiele zdarzeń z samej klasy, co pozwala zdefiniować kilka procedur obsługi zdarzeń w ramach jednej klasy. Subskrybenci powinni zdefiniować metodę `subscribe`, która zostanie przekazana do instancji dyspozytora zdarzeń. Możesz wywołać metodę `listen` na danym dyspozytorze, aby zarejestrować detektory zdarzeń:

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### Registering Event Subscribers - Rejestrowanie subskrybentów zdarzeń

Po zapisaniu subskrybenta możesz go zarejestrować w dyspozytorium zdarzeń. Możesz zarejestrować subskrybentów za pomocą właściwości `$subscribe` na `EventServiceProvider`. Na przykład dodajmy do listy `UserEventSubscriber`:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
