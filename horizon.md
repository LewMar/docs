# Laravel Horizon

- [Introduction - Wprowadzenie](#introduction)
- [Installation - Instalacja](#installation)
    - [Configuration - Konfiguracja](#configuration)
    - [Dashboard Authentication - Uwierzytelnianie pulpitu nawigacyjnego](#dashboard-authentication)
- [Running Horizon - Uruchomienie Horizon](#running-horizon)
    - [Deploying Horizon - Wdrażanie Horizon](#deploying-horizon)
- [Tags - Tagi](#tags)
- [Notifications - Powiadomienia](#notifications)
- [Metrics - Metryki](#metrics)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Horizon zapewnia piękną pulpit nawigacyjny i konfigurację opartą na kodzie dla twoich kolejek Redis zasilanych Laravel. Horizon umożliwia łatwe monitorowanie kluczowych parametrów systemu kolejki, takich jak przepustowość pracy, środowisko wykonawcze i awarie zadań.

Cała konfiguracja pracownika jest przechowywana w jednym, prostym pliku konfiguracyjnym, dzięki czemu konfiguracja pozostaje pod kontrolą źródła, w którym cały zespół może współpracować.

<a name="installation"></a>
## Installation - Instalacja

> {note} Ze względu na użycie asynchronicznych sygnałów procesowych, Horizon wymaga PHP 7.1+.

Możesz użyć Composer do zainstalowania Horizona w swoim projekcie Laravel:

    composer require laravel/horizon

Po zainstalowaniu Horizona opublikuj jego zasoby przy użyciu polecenia `vendor:publish` Artisan:

    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

<a name="configuration"></a>
### Configuration - Konfiguracja

Po opublikowaniu zasobów Horizona jego podstawowy plik konfiguracyjny będzie znajdować się w `config/horizon.php`. Ten plik konfiguracyjny umożliwia skonfigurowanie opcji roboczych, a każda opcja konfiguracji zawiera opis jego przeznaczenia, dlatego należy dokładnie zapoznać się z tym plikiem.

#### Balance Options - Rownoważenie opcji

Horizon pozwala wybrać jedną z trzech strategii równoważenia: `simple`, `auto` i `false`. Strategia `simple`, która jest domyślna, rozdziela zadania przychodzące równomiernie między procesami:

    'balance' => 'simple',

Strategia `auto` dostosowuje liczbę procesów roboczych na kolejkę w oparciu o bieżące obciążenie kolejki. Na przykład, jeśli twoja kolejka `notifications` ma 1000 oczekujących zadań, podczas gdy twoja kolejka `render` jest pusta, Horizon przydzieli więcej pracowników do kolejki `notifications`, dopóki nie będzie pusta. Gdy opcja `balance` jest ustawiona na `false`, użyte zostanie domyślne zachowanie Laravel, które przetwarza kolejki w kolejności, w jakiej są wymienione w twojej konfiguracji.

<a name="dashboard-authentication"></a>
### Dashboard Authentication - Uwierzytelnianie pulpitu nawigacyjnego

Horizon odsłania pulpitą w `/horizon`. Domyślnie dostęp do tego pulpitu będzie możliwy tylko w środowisku `local`. Aby zdefiniować bardziej szczegółową politykę dostępu do pulpitu, należy użyć metody `Horizon::auth`. Metoda `auth` akceptuje wywołanie zwrotne, które powinno zwracać wartość `true` lub `false`, wskazując, czy użytkownik powinien mieć dostęp do pulpitu programu Horizon:

    Horizon::auth(function ($request) {
        // return true / false;
    });

<a name="running-horizon"></a>
## Running Horizon - Uruchomienie Horizon

Po skonfigurowaniu pracowników w pliku konfiguracyjnym `config/horizon.php` możesz uruchomić Horizon za pomocą polecenia `horizon` Artisan. To pojedyncze polecenie uruchomi wszystkich skonfigurowanych pracowników:

    php artisan horizon

Możesz wstrzymać proces Horizon i poinstruować go, aby kontynuował przetwarzanie zadań za pomocą poleceń `horizon:pause` i` horizon:continue` Artisan:


    php artisan horizon:pause

    php artisan horizon:continue

Możesz z gracją zakończyć proces Master Horizon na swoim komputerze za pomocą polecenia `horizon:terminate` Artisan. Wszystkie zadania przetwarzane przez Horizon zostaną ukończone, a następnie Horizon zakończy działanie:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Deploying Horizon - Wdrażanie Horizon

Jeśli wdrażasz Horizon na serwerze na żywo, powinieneś skonfigurować monitor procesu, aby monitorować komendę `php artisan horizon` i restartować go, jeśli zostanie nieoczekiwanie zamknięty. Podczas wdrażania nowego kodu na serwerze, musisz poinstruować główny proces Horizon, aby zakończył działanie, aby mógł zostać uruchomiony ponownie przez monitor procesu i otrzymać zmiany kodu.

Możesz z gracją zakończyć proces Master Horizon na swoim komputerze za pomocą polecenia `horizon:terminate` Artisan. Wszystkie zadania przetwarzane przez Horizon zostaną ukończone, a następnie Horizon zakończy działanie:

    php artisan horizon:terminate

#### Supervisor Configuration - Konfiguracja administratora

Jeśli używasz monitora procesu Supervisor do zarządzania procesem `horizon`, powinien wystarczyć następujący plik konfiguracyjny:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log

> {tip} Jeśli nie lubisz zarządzać własnymi serwerami, rozważ użycie [Laravel Forge](https://forge.laravel.com). Forge zaopatruje serwery PHP 7+ we wszystko, co potrzebne do uruchomienia nowoczesnych, solidnych aplikacji Laravel z Horizon.

<a name="tags"></a>
## Tags - Tagi

Horizon pozwala przypisać "znaczniki" do zadań, w tym do wiadomości, transmisji zdarzeń, powiadomień i oczekujących zdarzeń w kolejce. W rzeczywistości Horizon inteligentnie i automatycznie oznaczy większość zadań w zależności od modeli Eloquent, które są dołączone do zadania. Na przykład spójrz na następujące zadanie:

    <?php

    namespace App\Jobs;

    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * The video instance.
         *
         * @var \App\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

Jeśli to zadanie zostanie umieszczone w kolejce z instancją `App\Video`, która ma `id` o nr `1`, automatycznie otrzyma znacznik `App\Video:1`. Dzieje się tak dlatego, że Horizon sprawdzi właściwości zadania dla dowolnych modeli Eloquent. Jeśli znalezione zostaną modele Eloquent, Horizon inteligentnie oznaczy zadanie za pomocą nazwy klasy modelu i klucza podstawowego:

    $video = App\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

#### Manually Tagging - Ręczne oznaczanie

Jeśli chcesz ręcznie zdefiniować tagi dla jednego z twoich obiektów nadających się do kolejkowania, możesz zdefiniować metodę `tags` na klasie:

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## Notifications - Powiadomienia

> **Note:** Przed użyciem powiadomień dodaj do projektu projekt `guzzlehttp/guzzle` Composer-a. Podczas konfigurowania Horizon do wysyłania powiadomień SMS, należy również zapoznać się z [wymaganiami wstępnymi sterownika powiadomienia Nexmo](https://laravel.com/docs/5.5/notifications#sms-notifications).

Jeśli chcesz otrzymywać powiadomienia, gdy jedna z Twoich kolejek ma długi czas oczekiwania, możesz użyć metod `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, i `Horizon::routeSmsNotificationsTo`. Możesz wywołać te metody ze `AppServiceProvider` aplikacji:

    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    Horizon::routeSmsNotificationsTo('15556667777');

#### Configuring Notification Wait Time Thresholds - Konfigurowanie progów czasu oczekiwania na powiadomienie

Możesz skonfigurować ile sekund jest uważanych za "długie oczekiwanie" w pliku konfiguracyjnym `config/horizon.php`. Opcja konfiguracyjna `waits` w tym pliku pozwala kontrolować próg długiego oczekiwania dla każdej kombinacji połączenia / kolejki:

    'waits' => [
        'redis:default' => 60,
    ],

<a name="metrics"></a>
## Metrics - Metryki

Horizon zawiera pulpit metryk, który dostarcza informacji na temat czasu pracy i czasu oczekiwania na kolejkę oraz przepustowości. Aby wypełnić ten pulpit, powinieneś skonfigurować komendę `snapshot` programu Artisan programu Horizon, co pięć minut za pośrednictwem [scheduler](/docs/{{version}}/scheduling) aplikacji:

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }
