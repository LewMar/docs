# Queues

- [Introduction - Wprowadzenie](#introduction)
    - [Connections Vs. Queues - Połączenia kontra kolejki](#connections-vs-queues)
    - [Driver Notes & Prerequisites - Listy i wymagania wstępne sterownika](#driver-prerequisites)
- [Creating Jobs - Tworzenie zadań](#creating-jobs)
    - [Generating Job Classes - Generowanie klas zadań](#generating-job-classes)
    - [Class Structure - Struktura klasy](#class-structure)
- [Dispatching Jobs - Wysyłanie zadań](#dispatching-jobs)
    - [Delayed Dispatching - Opóźniona wysyłka](#delayed-dispatching)
    - [Job Chaining - Łańcuch zadań](#job-chaining)
    - [Customizing The Queue & Connection - Dostosowywanie kolejki i połączenia](#customizing-the-queue-and-connection)
    - [Specifying Max Job Attempts / Timeout Values - Określanie maksymalnych czasów prób / wartości limitu czasu](#max-job-attempts-and-timeout)
    - [Rate Limiting - Ograniczanie szybkości](#rate-limiting)
    - [Error Handling - Obsługa błędów](#error-handling)
- [Running The Queue Worker - Uruchamianie pracownika kolejki](#running-the-queue-worker)
    - [Queue Priorities - Priorytety kolejki](#queue-priorities)
    - [Queue Workers & Deployment - Pracownicy kolejkowi i wdrażanie](#queue-workers-and-deployment)
    - [Job Expirations & Timeouts - Wygaśnięcia i limity czasu zadań](#job-expirations-and-timeouts)
- [Supervisor Configuration - Konfiguracja Nadzorcy](#supervisor-configuration)
- [Dealing With Failed Jobs - Radzenie sobie z nieudanymi zadaniami](#dealing-with-failed-jobs)
    - [Cleaning Up After Failed Jobs - Sprzątanie po nieudanych zadaniach](#cleaning-up-after-failed-jobs)
    - [Failed Job Events - Nieudane zdarzenia zadań](#failed-job-events)
    - [Retrying Failed Jobs - Ponowne wykonywanie nieudanych zadań](#retrying-failed-jobs)
- [Job Events - Zdarzenia zadań](#job-events)

<a name="introduction"></a>
## Introduction - Wprowadzenie

> {tip} Laravel oferuje teraz Horizon, piękny pulpit nawigacyjny i system konfiguracji dla kolejki zasilanej energią Redis. Zapoznaj się z pełną [dokumentacją Horizon](/docs/{{version}}/horizon), aby uzyskać więcej informacji.

Kolejki Laravel zapewniają ujednolicony interfejs API w wielu różnych mechanizmach kolejkowania, takich jak Beanstalk, Amazon SQS, Redis lub nawet relacyjna baza danych. Kolejki pozwalają odłożyć przetwarzanie czasochłonnego zadania, takiego jak wysłanie e-maila, do późniejszego czasu. Odroczenie tych czasochłonnych zadań radykalnie przyspiesza żądania internetowe do aplikacji.

Plik konfiguracyjny kolejki jest przechowywany w `config/queue.php`. W tym pliku znajdziesz konfiguracje połączeń dla każdego sterownika kolejki dołączonego do frameworku, który zawiera bazę danych [Beanstalkd](https://kr.github.io/beanstalkd/), [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io) i sterownik synchroniczny, który natychmiast wykona zadania (do użytku lokalnego). Dołączony jest także sterownik kolejki `null`, który powoduje odrzucenie zadań oczekujących w kolejce.

<a name="connections-vs-queues"></a>
### Connections Vs. Queues - Połączenia kontra kolejki

Przed rozpoczęciem pracy z kolejkami Laravel ważne jest zrozumienie rozróżnienia między "połączeniami" i "kolejkami". W pliku konfiguracyjnym `config/queue.php` znajduje się opcja konfiguracji `connections`. Ta opcja określa konkretne połączenie z usługą backendu, taką jak Amazon SQS, Beanstalk lub Redis. Jednak każde dane połączenie kolejki może mieć wiele "kolejek", które mogą być uważane za różne stosy lub stosy oczekujących zadań.

Zwróć uwagę, że każdy przykład konfiguracji połączenia w pliku konfiguracyjnym `queue` zawiera atrybut `queue`. Jest to domyślna kolejka, do której będą wysyłane zadania, gdy zostaną wysłane do danego połączenia. Innymi słowy, jeśli zadanie zostanie wysłane bez jawnego zdefiniowania, do której kolejki ma zostać wysłane, zadanie zostanie umieszczone w kolejce zdefiniowanej w atrybucie `queue` konfiguracji połączenia

    // This job is sent to the default queue...
    Job::dispatch();

    // This job is sent to the "emails" queue...
    Job::dispatch()->onQueue('emails');

Niektóre aplikacje mogą nie potrzebować nigdy przenosić zadań do wielu kolejek, zamiast tego wolą mieć jedną prostą kolejkę. Jednak przekazywanie zadań do wielu kolejek może być szczególnie przydatne dla aplikacji, które chcą priorytetyzować lub segmentować sposób przetwarzania zadań, ponieważ moduł kolejki Laravel pozwala określić, które kolejki powinny być przetwarzane według priorytetu. Na przykład, jeśli przekazujesz zadania do kolejki `high`, możesz uruchomić proces roboczy, który zapewni im wyższy priorytet przetwarzania:

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### Driver Notes & Prerequisites - Listy i wymagania wstępne sterownika

#### Database - Bazadanych

Aby użyć sterownika kolejki `database`, będziesz potrzebował tabeli bazy danych do przechowywania zadań. Aby wygenerować migrację, która tworzy tę tabelę, uruchom polecenie `queue: table` Artisan. Po utworzeniu migracji możesz przeprowadzić migrację bazy danych za pomocą komendy `migrate`:

    php artisan queue:table

    php artisan migrate

#### Redis

Aby użyć sterownika kolejki `redis`, powinieneś skonfigurować połączenie z bazą danych Redis w pliku konfiguracyjnym `config/database.php`.

**Redis Cluster**

Jeśli połączenie kolejki Redis używa klastra Redis, nazwy kolejek muszą zawierać [etykietę klucza hash](https://redis.io/topics/cluster-spec#keys-hash-tags). Jest to wymagane, aby wszystkie klucze Redis dla danej kolejki zostały umieszczone w tym samym polu skrótu:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

**Blocking - Blokowanie**

Podczas korzystania z kolejki Redis można użyć opcji konfiguracyjnej `block_for`, aby określić, jak długo sterownik ma czekać na udostępnienie zadania przed wykonaniem iteracji w pętli roboczej i ponownym odpytaniem bazy danych Redis.

Dostosowanie tej wartości w oparciu o obciążenie kolejki może być bardziej skuteczne niż ciągłe odpytywanie bazy danych Redis dla nowych zadań. Na przykład możesz ustawić wartość na `5`, aby wskazać, że sterownik powinien blokować przez pięć sekund podczas oczekiwania na udostępnienie zadania:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],

> {note} Blokowanie popu to eksperymentalna funkcja. Istnieje niewielka szansa, że zadanie oczekujące w kolejce może zostać utracone, jeśli serwer Redis lub pracownik ulegnie awarii w tym samym czasie, gdy zadanie zostanie pobrane.

#### Other Driver Prerequisites - Inne wymagania wstępne sterownika

Dla wymienionych sterowników kolejki potrzebne są następujące zależności:

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`
</div>

<a name="creating-jobs"></a>
## Creating Jobs - Tworzenie zadań

<a name="generating-job-classes"></a>
### Generating Job Classes - Generowanie klas zadań

Domyślnie wszystkie kolejki zadań dla aplikacji są przechowywane w katalogu `app/Jobs`. Jeśli katalog `app/Jobs` nie istnieje, zostanie utworzony po uruchomieniu polecenia` make:job` Artisan. Możesz wygenerować nowe zadanie w kolejce przy użyciu interfejsu Artisan CLI:

    php artisan make:job ProcessPodcast

Wygenerowana klasa implementuje interfejs `Illuminate\Contracts\Queue\ShouldQueue`, wskazując Laravel-owi, że zadanie powinno zostać przekazane do kolejki, aby działało asynchronicznie.

<a name="class-structure"></a>
### Class Structure - Struktura klasy

Klasy zadań są bardzo proste, zwykle zawierają tylko metodę `handle`, która jest wywoływana, gdy zadanie jest przetwarzane przez kolejkę. Na początek rzućmy okiem na przykładową klasę zadań. W tym przykładzie będziemy udawać, że zarządzamy usługą publikowania podcastów i musimy przetworzyć przesłane pliki podcastów przed ich opublikowaniem:

    <?php

    namespace App\Jobs;

    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    }

W tym przykładzie zauważ, że udało nam się przekazać [Eloquent model](/docs/{{version}}/eloquent) bezpośrednio do konstruktora kolejki zadań. Ze względu na cechę `SerializesModels` używaną w danym zadaniu, modele Eloquent będą serializowane z gracją i odserializowane podczas przetwarzania zadania. Jeśli twoje zadanie w kolejce akceptuje model Eloquent w swoim konstruktorze, tylko identyfikator dla modelu będzie serializowany do kolejki. Gdy zadanie zostanie faktycznie obsłużone, system kolejki automatycznie ponownie pobierze pełną instancję modelu z bazy danych. To wszystko jest całkowicie przezroczyste dla twojej aplikacji i zapobiega problemom, które mogą powstać w wyniku serializacji pełnych instancji Eloquent-a.

Metoda `handle` jest wywoływana, gdy zadanie jest przetwarzane przez kolejkę. Zauważ, że jesteśmy w stanie wpisywać zależność wskazówki w metodzie "obsługi" zlecenia. Laravel [kontener usługi](/docs/{{version}}/container) automatycznie wstrzykuje te zależności.

> {note} Dane binarne, takie jak zawartość obrazu nieprzetworzonego, powinny zostać przekazane za pośrednictwem funkcji `base64_encode` przed przekazaniem do zadania oczekującego w kolejce. W przeciwnym razie zadanie może nie poprawnie serializować do JSON, gdy zostanie umieszczone w kolejce.

<a name="dispatching-jobs"></a>
## Dispatching Jobs - Wysyłanie zadań

Po napisaniu klasy pracy możesz ją wysłać za pomocą metody `dispatch` w samym zadaniu. Argumenty przekazane do metody `dispatch` zostaną przekazane do konstruktora zadania:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast);
        }
    }

<a name="delayed-dispatching"></a>
### Delayed Dispatching - Opóźniona wysyłka

Jeśli chcesz opóźnić wykonanie zlecenia w kolejce, możesz użyć metody `delay` podczas wysyłania zlecenia. Na przykład, określmy, że zadanie nie powinno być dostępne do przetworzenia do 10 minut po jego wysłaniu:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));
        }
    }

> {note} Usługa kolejkowania Amazon SQS ma maksymalny czas opóźnienia 15 minut.

<a name="job-chaining"></a>
### Job Chaining - Łańcuch zadań

Łańcuch zadań umożliwia określenie listy oczekujących zadań, które powinny być uruchamiane kolejno. Jeśli jedno zadanie w sekwencji nie powiedzie się, pozostałe zadania nie zostaną uruchomione. Aby wykonać oczekujący łańcuch zadań, możesz użyć metody `withChain` dla dowolnych swoich zadań do wysłania:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch();

#### Chain Connection & Queue - Łancuch połaczenia i kolejki

Jeśli chcesz określić domyślne połączenie i kolejkę, które powinny być używane dla połączonych zadań, możesz użyć metod `allOnConnection` i `allOnQueue`. Te metody określają połączenie kolejki i nazwę kolejki, które powinny być używane, chyba że kolejkowemu zadaniu jawnie przypisano inne połączenie / kolejkę:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch()->allOnConnection('redis')->allOnQueue('podcasts');

<a name="customizing-the-queue-and-connection"></a>
### Customizing The Queue & Connection - Dostosowywanie kolejki i połączenia

#### Dispatching To A Particular Queue - Wysyłanie do konkretnej kolejki

Przesyłając zadania do różnych kolejek, można "kategoryzować" swoje zadania w kolejce, a nawet priorytetywać liczbę pracowników przypisanych do różnych kolejek. Należy pamiętać, że nie przesyła ona zadań do różnych "połączeń" w kolejce zdefiniowanych w pliku konfiguracyjnym kolejki, ale tylko do określonych kolejek w ramach jednego połączenia. Aby określić kolejkę, użyj metody `onQueue` podczas wysyłania zadania:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }

#### Dispatching To A Particular Connection - Wysyłanie do określonego połączenia

Jeśli pracujesz z wieloma połączeniami kolejkowania, możesz określić, z którego połączenia chcesz przekazać zadanie. Aby określić połączenie, użyj metody `onConnection` podczas wysyłania zadania:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }

Oczywiście możesz łańcuchować metody `onConnection` i` onQueue`, aby określić połączenie i kolejkę dla zadania:

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

<a name="max-job-attempts-and-timeout"></a>
### Specifying Max Job Attempts / Timeout Values - Określanie maksymalnych czasów prób / wartości limitu czasu

#### Max Attempts - Maksymalna ilość prób

Jednym ze sposobów określania maksymalnej liczby prób wykonania zadania jest użycie przełącznika `--tries` na linii poleceń Artisan:

    php artisan queue:work --tries=3

Można jednak przyjąć bardziej szczegółowe podejście, definiując maksymalną liczbę prób dla samej klasy zadania. Jeśli maksymalna liczba prób jest określona w zadaniu, będzie miała pierwszeństwo przed wartością podaną w wierszu poleceń:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of times the job may be attempted.
         *
         * @var int
         */
        public $tries = 5;
    }

<a name="time-based-attempts"></a>
#### Time Based Attempts - Czas bazowy prób

Jako alternatywę do określenia, ile razy można wykonać zadanie, zanim się nie powiedzie, możesz określić czas, w którym zadanie powinno upłynąć. Pozwala to na podjęcie próby dowolną liczbę razy w określonym przedziale czasowym. Aby zdefiniować czas, w którym zadanie powinno mieć limit czasu, dodaj do swojej klasy zadania metodę `retryUntil`:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} Możesz także zdefiniować metodę `retryUntil` na detektorach zdarzeń w kolejce.

#### Timeout - Koniec czasu

> {note} Funkcja `timeout` jest zoptymalizowana pod kątem PHP 7.1+ i rozszerzenia PHP `pcntl`.

Podobnie maksymalna liczba sekund, przez które zadania mogą być uruchamiane, może być określona za pomocą przełącznika `--timeout` w linii poleceń Artisan:

    php artisan queue:work --timeout=30

Można jednak zdefiniować maksymalną liczbę sekund, przez którą zadanie powinno być uruchamiane w samej klasie zadania. Jeśli limit czasu jest określony w zadaniu, ma on pierwszeństwo przed dowolnym czasem określonym w wierszu poleceń:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of seconds the job can run before timing out.
         *
         * @var int
         */
        public $timeout = 120;
    }

<a name="rate-limiting"></a>
### Rate Limiting - Ograniczanie szybkości

> {note} Ta funkcja wymaga współpracy aplikacji z serwerem [Redis](/docs/{{version}}/redis).

Jeśli twoja aplikacja wchodzi w interakcję z Redis, możesz dezaktywować swoje zadania w kolejce według czasu lub współbieżności. Ta funkcja może być pomocna, gdy zadania w kolejce wchodzą w interakcje z interfejsami API, które są również ograniczone szybkością. Na przykład za pomocą metody `throttle` można dławić dany typ zadania, aby uruchamiać go tylko 10 razy co 60 sekund. Jeśli nie można uzyskać blokady, powinieneś zazwyczaj zwolnić zadanie z powrotem do kolejki, aby można było je później ponowić:

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} W powyższym przykładzie `key` może być dowolnym ciągiem, który jednoznacznie identyfikuje typ zadania, dla którego chcesz wyznaczyć limit. Na przykład możesz chcieć skonstruować klucz na podstawie nazwy klasy zadania i identyfikatorów modeli Eloquent, na których on działa.

Alternatywnie możesz określić maksymalną liczbę pracowników, którzy mogą jednocześnie przetwarzać dane zadanie. Może to być pomocne, gdy zadanie w kolejce modyfikuje zasób, który powinien być modyfikowany tylko przez jedno zadanie na raz. Na przykład, używając metody `funnel`, możesz ograniczać zadania danego typu do przetwarzania tylko przez jednego pracownika naraz:

    Redis::funnel('key')->limit(1)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} Podczas korzystania z ograniczania tempa, liczba prób, które twoje zadanie będzie musiało wykonać, może być trudna do ustalenia. Dlatego przydatne jest łączenie ograniczenia szybkości z [próbami opartymi na czasie](#time-based-attempts).

<a name="error-handling"></a>
### Error Handling - Obsługa błędów

Jeśli podczas przetwarzania zadania zostanie zgłoszony wyjątek, zadanie zostanie automatycznie zwolnione z powrotem do kolejki, aby można było spróbować ponownie. Zadanie będzie kontynuowane, dopóki nie zostanie podjęta próba maksymalnej dopuszczalnej liczby razy dozwolonej przez aplikację. Maksymalna liczba prób jest określona za pomocą przełącznika `--trade` używanego w komendzie `queue:work` Artisan. Alternatywnie, maksymalna liczba prób może być zdefiniowana w samej klasie zadania. Więcej informacji na temat uruchamiania pracownika kolejki [można znaleźć poniżej](#running-the-queue-worker).

<a name="running-the-queue-worker"></a>
## Running The Queue Worker - Uruchamianie pracownika kolejki

Laravel zawiera moduł kolejki, który przetwarza nowe zadania, gdy są one przekazywane do kolejki. Możesz uruchomić proces roboczy za pomocą komendy `queue:work` Artisan. Zwróć uwagę, że po uruchomieniu polecenia `queue:work` będzie ono działać do momentu ręcznego zatrzymania lub zamknięcia terminala:

    php artisan queue:work

> {tip} Aby utrzymać proces `queue:work` działający w tle, należy użyć monitora procesu, takiego jak [Supervisor](#supervisor-configuration), aby zapewnić, że pracownik kolejki nie przestanie działać.

Pamiętaj, że pracownicy kolejkowi są długotrwałymi procesami i przechowują załadowany stan aplikacji w pamięci. W rezultacie nie zauważą zmian w bazie kodu po ich uruchomieniu. Dlatego podczas procesu wdrażania pamiętaj o [zresetowaniu pracowników kolejki](# queue-workers-and-deployment).

#### Processing A Single Job - Przetwarzanie pojedynczej pracy

Opcja `--once` może być użyta do instruowania pracownika, aby przetwarzał tylko jedno zadanie z kolejki:

    php artisan queue:work --once

#### Specifying The Connection & Queue - Określanie połączenia i kolejki

Możesz także określić, z którym połączeniem kolejki powinien korzystać pracownik. Nazwa połączenia przekazana do polecenia `work` powinna odpowiadać jednemu z połączeń zdefiniowanych w pliku konfiguracyjnym `config/queue.php`:

    php artisan queue:work redis

Możesz jeszcze bardziej dostosować swojego pracownika kolejki, przetwarzając tylko określone kolejki dla danego połączenia. Jeśli na przykład wszystkie wiadomości e-mail są przetwarzane w kolejce `emails` w połączeniu kolejkowym `redis`, możesz wydać następujące polecenie, aby uruchomić pracownika, który przetwarza tylko tę kolejkę:

    php artisan queue:work redis --queue=emails

#### Resource Considerations - Rozważania dotyczące zasobów

Pracownicy kolejkowi demonów nie "reboot" architektury przed przetworzeniem każdego zadania. Dlatego po każdym zakończeniu pracy należy zwolnić ciężkie zasoby. Na przykład, jeśli manipulujesz obrazem przy pomocy biblioteki GD, powinieneś zwolnić pamięć z `imagedestroy` kiedy skończysz.

<a name="queue-priorities"></a>
### Queue Priorities - Priorytety kolejki

Czasami możesz chcieć nadać priorytet przetwarzaniu kolejek. Na przykład w twoim `config/queue.php` możesz ustawić domyślną `queue` dla twojego połączenia `redis` na `low`. Jednak czasami możesz chcieć przekazać zadanie do kolejki priorytetowej `high` w następujący sposób:

    dispatch((new Job)->onQueue('high'));

Aby uruchomić moduł roboczy, który sprawdza, czy wszystkie zadania kolejki `high` są przetwarzane przed kontynuowaniem zadań w kolejce `low`, należy podać rozdzielaną przecinkami listę nazw kolejek do komendy `work`:

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### Queue Workers & Deployment - Pracownicy kolejkowi i wdrażanie

Ponieważ pracownicy kolejkowi są długotrwałymi procesami, nie będą rejestrować zmian w kodzie bez ponownego uruchamiania. Najprostszym sposobem wdrożenia aplikacji przy użyciu pracowników kolejki jest ponowne uruchomienie pracowników podczas procesu wdrażania. Możesz z wdziękiem ponownie uruchomić wszystkich pracowników, wydając polecenie `queue:restart`:

    php artisan queue:restart

To polecenie poinstruuje wszystkich pracowników kolejki, aby z wdziękiem "zginęli" po zakończeniu przetwarzania ich obecnej pracy, aby żadne istniejące zadania nie zostały utracone. Ponieważ pracownicy kolejkowi zginą po wykonaniu polecenia `queue:restart`, powinieneś uruchomić menedżera procesów, takiego jak [Supervisor](#supervisor-configuration), aby automatycznie zrestartować pracowników kolejki.

> {tip} Kolejka korzysta z [cache](/docs/{{version}}/cache) do przechowywania sygnałów restartowania, więc powinieneś sprawdzić, czy sterownik pamięci podręcznej jest poprawnie skonfigurowany dla twojej aplikacji przed użyciem tej funkcji.

<a name="job-expirations-and-timeouts"></a>
### Job Expirations & Timeouts - Wygaśnięcia i limity czasu zadań

#### Job Expiration - Wygaśnięcia zadań

W pliku konfiguracyjnym `config/queue.php` każde połączenie kolejki definiuje opcję `retry_after`. Ta opcja określa, ile sekund połączenie kolejki ma czekać przed ponownym przetwarzaniem przetwarzanego zadania. Na przykład, jeśli wartość `retry_after` jest ustawiona na `90`, zadanie zostanie zwolnione z powrotem do kolejki, jeśli przetwarzanie trwało 90 sekund bez usunięcia. Zazwyczaj powinieneś ustawić wartość `retry_after` na maksymalną liczbę sekund, które powinieneś racjonalnie wykonać, aby zakończyć przetwarzanie.

> {note} Jedynym połączeniem kolejki, które nie zawiera wartości `retry_after` jest Amazon SQS. SQS ponowi zadanie w oparciu o [Domyślny limit czasu widoczności](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html), który jest zarządzany w konsoli AWS.

#### Worker Timeouts - Przerwy w pracy pracowników

Polecenie `queue:work` Artisan udostępnia opcję `--timeout`. Opcja `--timeout` określa, jak długo proces nadrzędny kolejki Laravel będzie czekać, zanim zabije pracownika potomnego, który przetwarza zadanie. Czasami proces kolejki potomnej może zostać "zamrożony" z różnych powodów, takich jak zewnętrzne wywołanie HTTP, które nie odpowiada. Opcja `--timeout` usuwa zamrożone procesy, które przekroczyły określony limit czasu:

    php artisan queue:work --timeout=60

Opcja konfiguracji `retry_after` i opcja `--timeout` CLI są różne, ale współpracują ze sobą, aby zapewnić, że zadania nie zostaną utracone, a zadania zostaną pomyślnie przetworzone tylko jeden raz.

> {note} Wartość `--timeout` powinna być zawsze o kilka sekund krótsza niż wartość konfiguracyjna `retry_after`. Zapewni to, że pracownik przetwarzający dane zadanie zostanie zawsze zabity przed ponownym wykonaniem zadania. Jeśli twoja opcja `--timeout` jest dłuższa niż twoja wartość `retry_after`, twoje zadania mogą być przetwarzane dwa razy.

#### Worker Sleep Duration - Czas trwania snu pracownika

Gdy zadania są dostępne w kolejce, pracownik będzie kontynuował przetwarzanie zleceń bez opóźnień między nimi. Jednak opcja `sleep` określa, jak długo pracownik będzie "spał", jeśli nie są dostępne żadne nowe zadania. Podczas snu pracownik nie będzie przetwarzał żadnych nowych zadań - zadania będą przetwarzane po tym, jak pracownik ponownie się obudzi.

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>
## Supervisor Configuration - Konfiguracja Nadzorcy

#### Installing Supervisor - Instalowanie Nadzorcy

Supervisor jest monitorem procesu dla systemu operacyjnego Linux i automatycznie zrestartuje proces `queue:work`, jeśli się nie powiedzie. Aby zainstalować Supervisor na Ubuntu, możesz użyć następującego polecenia:

    sudo apt-get install supervisor

> {tip} Jeśli konfigurowanie samego Nadzorcy brzmi przytłaczająco, rozważ użycie [Laravel Forge](https://forge.laravel.com), które automatycznie zainstaluje i skonfiguruje Nadzorcę dla twoich projektów Laravel.

#### Configuring Supervisor - Konfigurowanie Nadzorcy

Pliki konfiguracyjne Supervisor są zwykle przechowywane w katalogu `/etc/supervisor/conf.d`. W tym katalogu możesz utworzyć dowolną liczbę plików konfiguracyjnych, które instruują nadzorcę, w jaki sposób monitorować twoje procesy. Na przykład utwórzmy plik `laravel-worker.conf`, który uruchamia i monitoruje proces `queue:work`:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

W tym przykładzie dyrektywa `numprocs` poinstruuje nadzorcę, aby wykonał 8 procesów `queue:work` i monitorował wszystkie, automatycznie restartując je, jeśli się nie powiodą. Oczywiście należy zmienić część `queue:work sqs` dyrektywy `command`, aby odzwierciedlić pożądane połączenie kolejki.

#### Starting Supervisor - Startowanie Nadzorcy

Po utworzeniu pliku konfiguracyjnego można zaktualizować konfigurację Supervisor i uruchomić procesy za pomocą następujących poleceń:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

Aby uzyskać więcej informacji na temat Nadzorcy, zapoznaj się z [dokumentacją Supervisor](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Dealing With Failed Jobs - Radzenie sobie z nieudanymi zadaniami

Czasami twoje zadania w kolejce zawiedzie. Nie martw się, rzeczy nie zawsze idą zgodnie z planem! Laravel oferuje wygodny sposób na określenie maksymalnej liczby prób wykonania pracy. Po przekroczeniu tej liczby zadań zostanie ona wstawiona do tabeli bazy danych `failed_jobs`. Aby utworzyć migrację dla tabeli `failed_jobs`, możesz użyć polecenia `queue:failed-table`:

    php artisan queue:failed-table

    php artisan migrate

Następnie, podczas działania twojego [kolejki pracowników](# running-the-queue-worker), powinieneś określić maksymalną liczbę prób wykonania zadania za pomocą przełącznika `--tries` w komendzie `queue:work`. Jeśli nie określisz wartości dla opcji `--tries`, zadania będą podejmowane bezterminowo:

    php artisan queue:work redis --tries=3

<a name="cleaning-up-after-failed-jobs"></a>
### Cleaning Up After Failed Jobs - Sprzątanie po nieudanych zadaniach

Możesz zdefiniować metodę `failed` bezpośrednio na swojej klasie zadań, umożliwiając wykonanie specyficznego zadania, gdy wystąpi awaria. Jest to idealna lokalizacja do wysyłania alertów do użytkowników lub cofania wszelkich działań wykonywanych przez pracę. `Exception`, który spowodował niepowodzenie zadania, zostanie przekazany do metody `failed`:

    <?php

    namespace App\Jobs;

    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }

        /**
         * The job failed to process.
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $exception)
        {
            // Send user notification of failure, etc...
        }
    }

<a name="failed-job-events"></a>
### Failed Job Events - Nieudane zdarzenia zadań

Jeśli chcesz zarejestrować zdarzenie, które zostanie wywołane, gdy zadanie się nie powiedzie, możesz użyć metody `Queue::failing`. Wydarzenie to jest doskonałą okazją do powiadomienia swojego zespołu przez e-mail lub [Stride](https://www.stride.com). Na przykład możemy dołączyć wywołanie zwrotne do tego zdarzenia z `AppServiceProvider`, który jest dołączony do Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
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
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="retrying-failed-jobs"></a>
### Retrying Failed Jobs - Ponowne wykonywanie nieudanych zadań

Aby wyświetlić wszystkie nieudane zadania wstawione do tabeli bazy danych `failed_jobs`, można użyć polecenia` queue:failed` Artisan:

    php artisan queue:failed

Komenda `queue:failed` wyświetli identyfikator zadania, połączenie, kolejkę i czas awarii. Identyfikator zadania może zostać wykorzystany do ponowienia nieudanego zlecenia. Na przykład, aby ponowić nieudane zadanie o identyfikatorze "5", wydaj następującą komendę:

    php artisan queue:retry 5

Aby ponowić wszystkie nieudane zadania, wykonaj komendę `queue:retry` i przekaż `all` jako identyfikator:

    php artisan queue:retry all

Jeśli chcesz usunąć nieudane zadanie, możesz użyć polecenia `queue:forget`:

    php artisan queue:forget 5

Aby usunąć wszystkie nieudane zadania, możesz użyć polecenia `queue:flush`:

    php artisan queue:flush

<a name="job-events"></a>
## Job Events - Zdarzenia zadań

Używając metod `before` i `after` na `Queue` [elewacji](/docs/{{version}}/facades) możesz określić wywołania zwrotne, które mają być wykonywane przed lub po przetworzeniu kolejki. Te wywołania zwrotne są doskonałą okazją do wykonania dodatkowych rejestrów lub statystyk przyrostowych dla pulpitu nawigacyjnego. Zazwyczaj należy wywoływać te metody od [usługodawcy](/docs/{{version}}/providers). Na przykład możemy użyć `AppServiceProvider`, który jest dołączony do Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Używając metody `looping` w `Queue` [elewacji](/docs/{{version}}/facades), możesz określić wywołania zwrotne, które zostaną wykonane, zanim pracownik spróbuje pobrać zadanie z kolejki. Na przykład możesz zarejestrować Closure, aby wycofać wszystkie transakcje, które zostały otwarte przez poprzednio nieudane zadanie:

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });