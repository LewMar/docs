# Broadcasting

- [Introduction - Wprowadzenie](#introduction)
    - [Configuration - Konfiguracja](#configuration)
    - [Driver Prerequisites - Wymagania wstępne sterownika](#driver-prerequisites)
- [Concept Overview - Przegląd koncepcji](#concept-overview)
    - [Using An Example Application - Korzystanie z przykładowej aplikacji](#using-example-application)
- [Defining Broadcast Events - Definiowanie zdarzeń rozgłoszeniowych](#defining-broadcast-events)
    - [Broadcast Name - Nazwa transmisji](#broadcast-name)
    - [Broadcast Data - Dane transmisji](#broadcast-data)
    - [Broadcast Queue - Kolejka rozgłoszeniowa](#broadcast-queue)
    - [Broadcast Conditions - Warunki transmisji](#broadcast-conditions)
- [Authorizing Channels - Autoryzacja kanałów](#authorizing-channels)
    - [Defining Authorization Routes - Definiowanie tras autoryzacji](#defining-authorization-routes)
    - [Defining Authorization Callbacks - Definiowanie wywołań autoryzacji](#defining-authorization-callbacks)
- [Broadcasting Events - Rozgłaszanie zdarzeń](#broadcasting-events)
    - [Only To Others - Tylko dla innych](#only-to-others)
- [Receiving Broadcasts - Odbierane rozgłoszenia](#receiving-broadcasts)
    - [Installing Laravel Echo - Instalowanie Laravel Echo](#installing-laravel-echo)
    - [Listening For Events - Słuchanie zdarzeń](#listening-for-events)
    - [Leaving A Channel - Opuszczając kanał](#leaving-a-channel)
    - [Namespaces - Przestrzenie nazw](#namespaces)
- [Presence Channels - Kanały obecności](#presence-channels)
    - [Authorizing Presence Channels - Autoryzacja kanałów obecności](#authorizing-presence-channels)
    - [Joining Presence Channels - Dołączanie do kanałów obecności](#joining-presence-channels)
    - [Broadcasting To Presence Channels - Transmisja do kanałów obecności](#broadcasting-to-presence-channels)
- [Client Events - Zdarzenia klienta](#client-events)
- [Notifications - Powiadomienia](#notifications)

<a name="introduction"></a>
## Introduction - Wprowadzenie

W wielu nowoczesnych aplikacjach webowych WebSockets są wykorzystywane do implementowania interfejsów użytkownika w czasie rzeczywistym i aktualizacji na żywo. Gdy niektóre dane są aktualizowane na serwerze, wiadomość jest zazwyczaj wysyłana za pośrednictwem połączenia WebSocket, z którym klient ma obsługiwać. Zapewnia to bardziej niezawodną, wydajną alternatywę do ciągłego odpytywania aplikacji o zmiany.

Aby pomóc ci w budowaniu tego typu aplikacji, Laravel ułatwia "rozgłaszanie" twoich [wydarzeń](/docs/{{version}}/events) przez połączenie WebSocket. Nadawanie zdarzeń Laravel umożliwia udostępnianie tych samych nazw zdarzeń między kodem po stronie serwera a aplikacją JavaScript po stronie klienta.


> {tip} Przed rozpoczęciem nurkowania w transmisje zdarzeń upewnij się, że przeczytałeś całą dokumentację dotyczącą Laravel [wydarzenia i słuchacze](/docs/{{version}}/events).

<a name="configuration"></a>
### Configuration - Konfiguracja

Cała konfiguracja rozgłaszania zdarzeń aplikacji jest przechowywana w pliku konfiguracyjnym `config/broadcasting.php`. Laravel obsługuje kilka sterowników rozgłoszeniowych: [Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis) i sterownik `log` do lokalnego programowania i debugowania . Dodatkowo dołączony jest sterownik `null`, który pozwala całkowicie wyłączyć nadawanie. Przykład konfiguracji dla każdego z tych sterowników znajduje się w pliku konfiguracyjnym `config/broadcasting.php`.

#### Broadcast Service Provider - Dostawca usług rozgłoszeniowych

Przed transmisją jakichkolwiek wydarzeń, musisz najpierw zarejestrować `App\Providers\BroadcastServiceProvider`. W świeżych aplikacjach Laravel wystarczy odkomentować tego dostawcę w tablicy `provider` pliku konfiguracyjnego `config/app.php`. Ten dostawca umożliwi rejestrację tras autoryzacji rozgłaszania i wywołań zwrotnych.

#### CSRF Token - Żeton CSRF

[Laravel Echo](#installing-laravel-echo) będzie wymagać dostępu do tokenu CSRF bieżącej sesji. Powinieneś sprawdzić, czy element HTML `head` twojej aplikacji definiuje znacznik `meta` zawierający token CSRF:

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### Driver Prerequisites - Wymagania wstępne sterownika

#### Pusher

Jeśli transmitujesz swoje wydarzenia przez [Pusher](https://pusher.com), powinieneś zainstalować Pusher PHP SDK przy użyciu menedżera pakietów Composer:

    composer require pusher/pusher-php-server "~3.0"

Następnie powinieneś skonfigurować swoje dane Pusher w pliku konfiguracyjnym `config/broadcasting.php`. Przykładowa konfiguracja Pusher jest już zawarta w tym pliku, co pozwala szybko określić klucze Pusher, tajny i identyfikator aplikacji. Konfiguracja `pusher` pliku `config/broadcasting.php` pozwala również na określenie dodatkowych `opcji` obsługiwanych przez Pusher, takich jak klaster:

    'options' => [
        'cluster' => 'eu',
        'encrypted' => true
    ],

Używając Pushera i [Laravel Echo](#installation-laravel-echo), powinieneś określić `pusher` jako żądany nadawca podczas tworzenia wystąpienia Echo w pliku `resources/assets/js/bootstrap.js`:

    import Echo from "laravel-echo"

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

#### Redis

Jeśli korzystasz z nadawcy Redis, powinieneś zainstalować bibliotekę Predis:

    composer require predis/predis

Nadawca Redis będzie nadawać wiadomości za pomocą funkcji pub / sub Redis-a; należy jednak sparować to z serwerem WebSocket, który może odbierać wiadomości od Redis i transmitować je do swoich kanałów WebSocket.

Kiedy nadawca Redis opublikuje wydarzenie, zostanie on opublikowany w nazwach określonych kanałów zdarzenia, a ładunek będzie zakodowanym ciągiem JSON zawierającym nazwę zdarzenia, ładunek `data` i użytkownika, który wygenerował identyfikator gniazda zdarzenia (jeśli dotyczy).

#### Socket.IO

Jeśli zamierzasz sparować nadawcę Redis z serwerem Socket.IO, będziesz musiał dołączyć bibliotekę kliencką JavaScript Socket.IO do elementu HTML `head` twojej aplikacji. Po uruchomieniu serwera Socket.IO automatycznie udostępni on bibliotekę JavaScript klienta pod standardowym adresem URL. Na przykład, jeśli korzystasz z serwera Socket.IO w tej samej domenie co twoja aplikacja internetowa, możesz uzyskać dostęp do biblioteki klienta tak:

    <script src="//{{ Request::getHost() }}:6001/socket.io/socket.io.js"></script>

Następnie musisz utworzyć instancję Echo ze złączem `socket.io` i `host`.


    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

Na koniec będziesz musiał uruchomić kompatybilny serwer Socket.IO. Laravel nie zawiera implementacji serwera Socket.IO; jednak serwer Socket.IO zarządzany przez społeczność jest obecnie utrzymywany na serwerze [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) repozytorium GitHub.

#### Queue Prerequisites - Wymagania kolejkowania

Przed emisją zdarzeń należy również skonfigurować i uruchomić [moduł do odbierania kolejek](/docs/{{version}}/queues). Wszystkie transmisje zdarzeń odbywają się za pośrednictwem zadań oczekujących w kolejce, dzięki czemu czas reakcji w aplikacji nie jest poważnie zagrożony.

<a name="concept-overview"></a>
## Concept Overview - Przegląd koncepcji

Rozgłaszanie zdarzeń Laravel umożliwia rozgłaszanie zdarzeń Laravel po stronie serwera do aplikacji klienckiej JavaScript po stronie klienta przy użyciu podejścia opartego na sterownikach do WebSockets. Obecnie Laravel jest dostarczany ze sterownikami [Pusher](https://pusher.com) i Redis. Zdarzenia mogą być łatwo konsumowane po stronie klienta za pomocą pakietu Javascript [Laravel Echo](#installation-laravel-echo).

Wydarzenia są nadawane za pośrednictwem "kanałów", które można określić jako publiczne lub prywatne. Każdy odwiedzający twoją aplikację może zasubskrybować publiczny kanał bez uwierzytelniania lub autoryzacji; Jednak aby subskrybować kanał prywatny, użytkownik musi być uwierzytelniony i upoważniony do słuchania na tym kanale.

<a name="using-example-application"></a>
### Using An Example Application - Korzystanie z przykładowej aplikacji

Zanim przejdziemy do każdego komponentu emisji wydarzeń, przyjrzyjmy się przeglądowi wysokiego poziomu, na przykładzie sklepu e-commerce. Nie będziemy omawiać szczegółów konfiguracji [Pusher](https://pusher.com) ani [Laravel Echo](#installation-laravel-echo), ponieważ zostanie to szczegółowo omówione w innych sekcjach tej dokumentacji.

W naszej aplikacji przyjmijmy, że mamy stronę, która umożliwia użytkownikom przeglądanie statusu wysyłki dla swoich zamówień. Załóżmy również, że zdarzenie `ShippingStatusUpdated` jest uruchamiane, gdy aplikacja aktualizuje status wysyłki:

    event(new ShippingStatusUpdated($update));

#### The `ShouldBroadcast` Interface - Interfejs `ShouldBroadcast`

Gdy użytkownik przegląda jedno ze swoich zamówień, nie chcemy, aby musiał odświeżać stronę, aby wyświetlić aktualizacje statusu. Zamiast tego chcemy nadawać aktualizacje aplikacji, gdy są one tworzone. Musimy więc zaznaczyć zdarzenie `ShippingStatusUpdated` z interfejsem `ShouldBroadcast`. Spowoduje to, że Laravel będzie transmitować wydarzenie po jego uruchomieniu:

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * Information about the shipping status update.
         *
         * @var string
         */
        public $update;
    }

Interfejs `ShouldBroadcast` wymaga, aby nasze zdarzenie definiowało metodę `broadcastOn`. Ta metoda jest odpowiedzialna za zwrócenie kanałów, na których wydarzenie ma nadawać. Pusty odcinek tej metody jest już zdefiniowany na wygenerowanych klasach zdarzeń, więc musimy tylko wypełnić jego szczegóły. Chcemy tylko, aby twórca zamówienia mógł wyświetlać aktualizacje statusu, dlatego będziemy transmitować wydarzenie na prywatnym kanale powiązanym z zamówieniem:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### Authorizing Channels - Autoryzacja kanałów

Pamiętaj, że użytkownicy muszą mieć uprawnienia do słuchania na prywatnych kanałach. Możemy zdefiniować nasze reguły autoryzacji kanałów w pliku `routes/channels.php`. W tym przykładzie musimy sprawdzić, czy każdy użytkownik usiłujący nasłuchiwać na prywatnym kanale `order.1` jest w rzeczywistości twórcą zamówienia:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Metoda `channel` przyjmuje dwa argumenty: nazwę kanału i wywołanie zwrotne, które zwraca `true` lub `false` wskazujące, czy użytkownik jest uprawniony do nasłuchiwania na kanale.

Wszystkie wywołania zwrotne autoryzacji odbierają aktualnie uwierzytelnionego użytkownika jako swój pierwszy argument i wszelkie dodatkowe parametry wieloznaczne jako kolejne argumenty. W tym przykładzie używamy symbolu zastępczego `{orderId}`, aby wskazać, że część "ID" nazwy kanału jest symbolem wieloznacznym.
#### Listening For Event Broadcasts - Słuchanie rozgłaszanych zdarzeń

Następnie pozostaje nam tylko wysłuchać zdarzenia w naszej aplikacji JavaScript. Możemy to zrobić za pomocą Laravel Echo. Najpierw użyjemy metody "private" do subskrybowania kanału prywatnego. Następnie możemy użyć metody `listen` do nasłuchiwania zdarzenia `ShippingStatusUpdated`. Domyślnie wszystkie właściwości publiczne wydarzenia zostaną uwzględnione w wydarzeniu transmisji:

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## Defining Broadcast Events - Definiowanie zdarzeń rozgłoszeniowych

Aby poinformować Laravel, że dane wydarzenie ma być rozgłaszane, należy zaimplementować interfejs `Illuminate\Contracts\Broadcasting\ShouldBroadcast` na klasie zdarzenia. Ten interfejs jest już zaimportowany do wszystkich klas zdarzeń wygenerowanych przez framework, dzięki czemu można go łatwo dodać do dowolnych zdarzeń.

Interfejs `ShouldBroadcast` wymaga implementacji jednej metody: `broadcastOn`. Metoda `broadcastOn` powinna zwracać kanał lub tablicę kanałów, na których wydarzenie ma nadawać. Kanały powinny być instancjami `Channel`, `PrivateChannel` lub `PresenceChannel`. Instancje `Channel` reprezentują publiczne kanały, które każdy użytkownik może subskrybować, a `PrivateChannels` oraz `PresenceChannels` reprezentują prywatne kanały, które wymagają [autoryzacji kanału](#authorizing-channels):

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

Następnie musisz tylko [wywołać zdarzenie](/docs/{{version}}/events) tak jak zwykle. Po uruchomieniu zdarzenia kolejka [zadanie w kolejce](/docs/{{version}}/queues) automatycznie rozgłasza zdarzenie nad określonym sterowniku rozgłoszeniowym.

<a name="broadcast-name"></a>
### Broadcast Name - Nazwa transmisji

Domyślnie Laravel będzie rozgłaszał zdarzenie, używając nazwy klasy zdarzenia. Możesz jednak dostosować nazwę rozgłoszenia, definiując metodę `broadcastAs` dla zdarzenia:

    /**
     * The event's broadcast name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

Jeśli dostosujesz nazwę rozgłoszeniową za pomocą metody `broadcastAs`, powinieneś upewnić się, że zarejestrowałeś swojego słuchacza z wiodącym znakiem `.`. Spowoduje to, że Echo nie będzie dodawać przestrzeni nazw aplikacji do zdarzenia:

    .listen('.server.created', function (e) {
        ....
    });

<a name="broadcast-data"></a>
### Broadcast Data - Dane transmisji

When an event is broadcast, all of its `public` properties are automatically serialized and broadcast as the event's payload, allowing you to access any of its public data from your JavaScript application. So, for example, if your event has a single public `$user` property that contains an Eloquent model, the event's broadcast payload would be:
Gdy zdarzenie jest transmitowane, wszystkie jego właściwości "publiczne" są automatycznie szeregowane i emitowane jako ładunek zdarzenia, co pozwala na dostęp do dowolnych jego publicznych danych z aplikacji JavaScript. Na przykład, jeśli zdarzenie ma jedną publiczną właściwość `$user` zawierającą model Eloquent, ładunek emisji wydarzenia będzie następujący:

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

Jeśli jednak chcesz mieć większą kontrolę nad swoim nadajnikiem transmisji, możesz dodać metodę `broadcastWith` do swojego zdarzenia. Ta metoda powinna zwrócić tablicę danych, które chcesz transmitować jako ładunek zdarzenia:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### Broadcast Queue - Kolejka rozgłoszeniowa

Domyślnie każde zdarzenie emisji jest umieszczane w domyślnej kolejce dla domyślnego połączenia kolejki określonego w pliku konfiguracyjnym `queue.php`. Możesz dostosować kolejkę używaną przez nadawcę, definiując właściwość `broadcastQueue` w klasie zdarzenia. Ta właściwość powinna określać nazwę kolejki, której chcesz używać podczas rozgłaszania:

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

Jeśli chcesz transmitować swoje zdarzenia za pomocą kolejki `sync` zamiast domyślnego sterownika kolejki, możesz zaimplementować interfejs `ShouldBroadcastNow` zamiast `ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }
<a name="broadcast-conditions"></a>
### Broadcast Conditions - Warunki transmisji

Czasami chcesz transmitować swoje wydarzenie tylko wtedy, gdy dany warunek jest prawdziwy. Możesz zdefiniować te warunki, dodając metodę `broadcastWhen` do swojej klasy zdarzenia:

    /**
     * Determine if this event should broadcast.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>
## Authorizing Channels - Autoryzacja kanałów

Kanały prywatne wymagają autoryzacji, aby aktualnie uwierzytelniony użytkownik mógł rzeczywiście słuchać na kanale. Można to osiągnąć, wysyłając żądanie HTTP do aplikacji Laravel z nazwą kanału i zezwalając aplikacji na określenie, czy użytkownik może słuchać na tym kanale. Podczas korzystania z [Laravel Echo](#installation-laravel-echo) żądanie HTTP autoryzujące subskrypcje kanałów prywatnych zostanie wykonane automatycznie; jednak musisz zdefiniować właściwe trasy, aby odpowiedzieć na te żądania.

<a name="defining-authorization-routes"></a>
### Defining Authorization Routes - Definiowanie tras autoryzacji

Na szczęście Laravel ułatwia zdefiniowanie tras odpowiadających na żądania autoryzacji kanałów. W `BroadcastServiceProvider` dołączonym do twojej aplikacji Laravel zobaczysz połączenie z metodą `Broadcast::routes`. Ta metoda zarejestruje trasę `/broadcasting/auth`, aby obsłużyć żądania autoryzacji:

    Broadcast::routes();

Metoda `Broadcast::routes` automatycznie umieszcza swoje trasy w grupie `web` oprogramowania pośredniego; jednak możesz przekazać tablicę atrybutów trasy do metody, jeśli chcesz dostosować przypisane atrybuty:

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### Defining Authorization Callbacks - Definiowanie wywołań autoryzacji

Następnie musimy zdefiniować logikę, która faktycznie wykona autoryzację kanału. Odbywa się to w pliku `routes/channels.php` dołączonym do aplikacji. W tym pliku możesz użyć metody `Broadcast::channel`, aby zarejestrować wywołania zwrotne autoryzacji kanału:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

Metoda `channel` przyjmuje dwa argumenty: nazwę kanału i wywołanie zwrotne, które zwraca `true` lub `false` wskazujące, czy użytkownik jest uprawniony do nasłuchiwania na kanale.

Wszystkie wywołania zwrotne autoryzacji odbierają aktualnie uwierzytelnionego użytkownika jako swój pierwszy argument i wszelkie dodatkowe parametry wieloznaczne jako kolejne argumenty. W tym przykładzie używamy symbolu zastępczego `{orderId}`, aby wskazać, że część "ID" nazwy kanału jest symbolem wieloznacznym.

#### Authorization Callback Model Binding - Autoryzacja Callback Model Binding

Podobnie jak trasy HTTP, trasy kanałów mogą również wykorzystywać niejawne i jawne [powiązanie modelu trasy](/docs/{{version}}/routing#route-model-binding). Na przykład zamiast odbierać ciąg znaków lub numeryczny identyfikator zamówienia, możesz poprosić o rzeczywistą instancję modelu `Order`:

    use App\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="broadcasting-events"></a>
## Broadcasting Events - Rozgłaszanie zdarzeń

Po zdefiniowaniu zdarzenia i oznaczeniu go za pomocą interfejsu `ShouldBroadcast`, wystarczy wywołać zdarzenie za pomocą funkcji `event`. Program rozsyłający zdarzenia zauważy, że zdarzenie jest oznaczone interfejsem `ShouldBroadcast` i spowoduje umieszczenie zdarzenia w kolejce do nadawania:

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### Only To Others - Tylko dla innych

Podczas budowania aplikacji wykorzystującej rozgłszanie zdarzeń, możesz zastąpić funkcję `event` funkcją `broadcast`. Podobnie jak funkcja `event`, funkcja `broadcast` wywołuje zdarzenie do nasłuchujących po stronie serwera:

    broadcast(new ShippingStatusUpdated($update));

Jednak funkcja `broadcast` również eksponuje metodę `toOthers`, która pozwala ci wykluczyć bieżącego użytkownika z odbiorców transmisji:

    broadcast(new ShippingStatusUpdated($update))->toOthers();

Aby lepiej zrozumieć, kiedy chcesz użyć metody `toOthers`, wyobraźmy sobie aplikację na liście zadań, w której użytkownik może utworzyć nowe zadanie, wprowadzając nazwę zadania. Aby utworzyć zadanie, aplikacja może wysłać żądanie do punktu końcowego `/task`, który rozgłasza tworzenie zadania i zwraca reprezentację JSON nowego zadania. Gdy aplikacja JavaScript otrzyma odpowiedź od punktu końcowego, może bezpośrednio wstawić nowe zadanie do swojej listy zadań, tak jak to:

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

Pamiętaj jednak, że nadawaliśmy także tworzenie tego zadania. Jeśli twoja aplikacja JavaScript nasłuchuje tego zdarzenia w celu dodania zadań do listy zadań, będziesz mieć zduplikowane zadania na liście: jedną z punktu końcowego i jedną z transmisji.

Możesz rozwiązać ten problem, używając metody `toOthers`, aby poinstruować nadawcę, aby nie transmitował zdarzenia do bieżącego użytkownika.

#### Configuration - Konfiguracja

Podczas inicjowania instancji Laravel Echo do połączenia jest przypisywany identyfikator gniazda. Jeśli używasz [Vue](https://vuejs.org) i [Axios](https://github.com/mzabriskie/axios), identyfikator gniazda zostanie automatycznie dołączony do każdego wychodzącego żądania jako `X- Nagłówek Socket-ID`. Następnie, wywołując metodę `toOthers`, Laravel wyodrębni identyfikator gniazda z nagłówka i poinstruuje nadawcę, aby nie nadawał żadnych połączeń z tym identyfikatorem gniazda.

Jeśli nie używasz Vue i Axios, będziesz musiał ręcznie skonfigurować swoją aplikację JavaScript, aby wysłać nagłówek `X-Socket-ID`. Możesz pobrać identyfikator gniazda za pomocą metody `Echo.socketId`:

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## Receiving Broadcasts - Odbierane rozgłoszenia

<a name="installing-laravel-echo"></a>
### Installing Laravel Echo - Instalowanie Laravel Echo

Laravel Echo to biblioteka JavaScript, która sprawia, że bezboleśnie subskrybować kanały i słuchać wydarzeń emitowanych przez Laravel. Możesz zainstalować Echo za pośrednictwem menedżera pakietów NPM. W tym przykładzie zainstalujemy również pakiet `pusher-js`, ponieważ będziemy używać nadawcy Pusher:

    npm install --save laravel-echo pusher-js

Po zainstalowaniu Echo możesz przygotować nową instancję Echo w JavaScript swojej aplikacji. Doskonałym miejscem na to jest u dołu pliku `resources/assets/js/bootstrap.js`, który jest dołączony do frameworka Laravel:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

Podczas tworzenia instancji Echo, która korzysta ze złącza `pusher`, możesz także określić `cluster`, a także, czy połączenie powinno być szyfrowane:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        cluster: 'eu',
        encrypted: true
    });

<a name="listening-for-events"></a>
### Listening For Events - Słuchanie zdarzeń

Po zainstalowaniu i utworzeniu instancji Echo możesz zacząć słuchać transmisji zdarzeń. Najpierw użyj metody `channel`, aby pobrać instancję kanału, a następnie wywołaj metodę `listen`, aby nasłuchiwać określonego zdarzenia:

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

Jeśli chcesz słuchać zdarzeń na prywatnym kanale, użyj zamiast tego metody `private`. Możesz kontynuować łańcuch połączeń do metody `listen`, aby nasłuchiwać wielu zdarzeń na jednym kanale:

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>
### Leaving A Channel - Opuszczając kanał

Aby opuścić kanał, możesz wywołać metodę `leave` na instancji Echo:

    Echo.leave('orders');

<a name="namespaces"></a>
### Namespaces - Przestrzenie nazw

Być może zauważyłeś w powyższych przykładach, że nie określiliśmy pełnej przestrzeni nazw dla klas zdarzeń. Dzieje się tak, ponieważ Echo automatycznie zakłada, że zdarzenia znajdują się w przestrzeni nazw `App\Events`. Można jednak skonfigurować główną przestrzeń nazw podczas tworzenia instancji Echo, przekazując opcję konfiguracyjną `namespace`:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        namespace: 'App.Other.Namespace'
    });

Alternatywnie, możesz prefiksować klasy zdarzeń z `.` podczas subskrybowania ich za pomocą Echo. Umożliwi to zawsze określenie w pełni kwalifikowanej nazwy klasy:

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Presence Channels - Kanały obecności

Kanały obecności opierają się na bezpieczeństwie kanałów prywatnych, a jednocześnie uwidaczniają dodatkową funkcję świadomości, kto jest subskrybentem kanału. Ułatwia to tworzenie zaawansowanych, współpracujących funkcji aplikacji, takich jak powiadamianie użytkowników, gdy inny użytkownik przegląda tę samą stronę.

<a name="authorizing-presence-channels"></a>
### Authorizing Presence Channels - Autoryzacja kanałów obecności

Wszystkie kanały obecności są również kanałami prywatnymi; w związku z tym użytkownicy muszą być [uprawnieni do uzyskiwania do nich dostępu](#authorizing-channels). Jednak podczas definiowania wywołań zwrotnych autoryzacji dla kanałów obecności nie zwrócisz wartości `true`, jeśli użytkownik jest uprawniony do przyłączenia się do kanału. Zamiast tego powinieneś zwrócić tablicę danych o użytkowniku.

Dane zwrócone przez wywołanie zwrotne autoryzacji zostaną udostępnione detektorom zdarzeń kanału obecności w aplikacji JavaScript. Jeśli użytkownik nie ma uprawnień do przyłączenia się do kanału obecności, powinieneś zwrócić `false` lub `null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Joining Presence Channels - Dołączanie do kanałów obecności

Aby dołączyć do kanału obecności, możesz użyć metody `join` firmy Echo. Metoda `join` zwróci implementację `PresenceChannel`, która wraz z odsłonięciem metody `listen` umożliwia subskrybowanie zdarzeń `here`, `join` oraz` leaving`.

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

Wywołanie `here` zostanie wykonane natychmiast po pomyślnym połączeniu kanału i otrzyma tablicę zawierającą informacje o użytkowniku dla wszystkich innych użytkowników aktualnie subskrybujących kanał. Metoda `joining` zostanie wykonana, gdy nowy użytkownik dołączy do kanału, natomiast metoda `leaving` zostanie wykonana, gdy użytkownik opuści kanał.

<a name="broadcasting-to-presence-channels"></a>
### Broadcasting To Presence Channels - Transmisja do kanałów obecności

Kanały obecności mogą odbierać zdarzenia, tak jak kanały publiczne lub prywatne. Na przykładzie czatu możemy chcieć transmitować zdarzenia `NewMessage` do kanału obecności w pokoju. W tym celu zwrócimy instancję `PresenceChannel` z metody` broadcastOn` zdarzenia:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Podobnie jak zdarzenia publiczne lub prywatne, zdarzenia kanału obecności mogą być nadawane za pomocą funkcji `broadcast`. Podobnie jak w przypadku innych zdarzeń, możesz użyć metody `toOthers`, aby wykluczyć bieżącego użytkownika z odbioru transmisji:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

Możesz posłuchać zdarzenia join przez metodę `listen` Echa:

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>
## Client Events - Zdarzenia klienta

Czasami możesz chcieć nadać zdarzenie innym podłączonym klientom bez rrejestrowania w aplikację Laravel. Może to być szczególnie przydatne w przypadku takich powiadomień, jak "pisząc", w których chcesz powiadomić użytkowników swojej aplikacji, że inny użytkownik pisze wiadomość na danym ekranie. Aby transmitować zdarzenia klienta, możesz użyć metody `whisper` Echa:

    Echo.channel('chat')
        .whisper('typing', {
            name: this.user.name
        });

Aby nasłuchiwać zdarzeń klienta, możesz użyć metody `listenForWhisper`:

    Echo.channel('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>
## Notifications - Powiadomienia

Po powiązaniu emisji zdarzeń z [powiadomieniami](/docs/{{version}}/notifications), aplikacja JavaScript może otrzymywać nowe powiadomienia, które pojawiają się bez potrzeby odświeżania strony. Najpierw przeczytaj dokumentację na temat [kanału powiadomień o transmisji](/docs/{{version}}/notifications#broadcast-notifications).

Po skonfigurowaniu powiadomienia o korzystaniu z kanału transmisji możesz wysłuchać zdarzeń rozgłoszeniowych za pomocą metody `notification` Echo. Pamiętaj, że nazwa kanału powinna pasować do nazwy klasy podmiotu odbierającego powiadomienia:

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

W tym przykładzie wszystkie powiadomienia wysyłane do instancji `App\User` za pośrednictwem kanału `broadcast` będą odbierane przez wywołanie zwrotne. Wywołanie autoryzacji kanału dla kanału `App.User.{id}` jest zawarte w domyślnym `BroadcastServiceProvider` dostarczanym ze strukturą Laravel.
