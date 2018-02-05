# Logging

- [Introduction - Wprowadzenie](#introduction)
- [Configuration - Konfiguracja](#configuration)
    - [Building Log Stacks - Budowanie stosów dzienników](#building-log-stacks)
- [Writing Log Messages - Pisanie logów](#writing-log-messages)
    - [Writing To Specific Channels - Pisanie do określonych kanałów](#writing-to-specific-channels)
- [Customizing Monolog For Channels - Dostosowywanie Monologa dla kanałów](#customizing-monolog-for-channels)
- [Creating Custom Channels - Tworzenie niestandardowych kanałów](#creating-custom-channels)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Aby pomóc ci dowiedzieć się więcej o tym, co dzieje się w twojej aplikacji, Laravel oferuje niezawodne usługi rejestrowania, które umożliwiają logowanie wiadomości do plików, dziennika błędów systemu, a nawet do Slack, aby powiadomić cały zespół.

Pod maską Laravel wykorzystuje bibliotekę [Monolog](https://github.com/Seldaek/monolog), która zapewnia wsparcie dla wielu potężnych programów do obsługi dziennika. Laravel ułatwia konfigurowanie tych programów obsługi, umożliwiając ich łączenie i dopasowywanie w celu dostosowania obsługi dzienników aplikacji.

<a name="configuration"></a>
## Configuration - Konfiguracja

Cała konfiguracja systemu rejestracji aplikacji znajduje się w pliku konfiguracyjnym `config/logging.php`. Ten plik umożliwia skonfigurowanie kanałów dziennika aplikacji, dlatego należy przejrzeć każdy z dostępnych kanałów i ich opcji. Oczywiście przejrzymy kilka typowych opcji poniżej.

Domyślnie Laravel użyje kanału `stack` podczas rejestrowania wiadomości. Kanał `stack` służy do agregowania wielu kanałów dziennika w jeden kanał. Aby uzyskać więcej informacji na temat budowania stosów, zapoznaj się z [dokumentacją poniżej](#building-log-stacks).

#### Configuring The Channel Name - Konfigurowanie nazwy kanału

Domyślnie Monolog tworzy instancję z "nazwą kanału", która pasuje do bieżącego środowiska, na przykład `production` lub` local`. Aby zmienić tę wartość, dodaj opcję `name` do konfiguracji swojego kanału:

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

#### Configuring The Slack Channel - Konfigurowanie kanału Slack

Kanał `slack` wymaga opcji konfiguracyjnej `url`. Ten adres URL powinien być zgodny z adresem URL dla [webhooku przychodzącego](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks), który skonfigurowałeś dla zespołu Slack.

<a name="building-log-stacks"></a>
### Building Log Stacks - Budowanie stosów dzienników

Jak wcześniej wspomniano, sterownik `stack` umożliwia łączenie wielu kanałów w jeden kanał dziennika. Aby zilustrować sposób korzystania z stosów dziennika, rzućmy okiem na przykładową konfigurację, którą możesz zobaczyć w aplikacji produkcyjnej:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],

Rozłóżmy tę konfigurację. Po pierwsze, zauważmy, że nasz kanał `stack` agreguje dwa inne kanały poprzez jego opcję `channels`: `syslog` i `slack`. Tak więc podczas rejestrowania wiadomości oba te kanały będą miały możliwość zarejestrowania wiadomości.

#### Log Levels - Poziomy logowania

Zwróć uwagę na opcję konfiguracyjną `level` znajdującą się w konfiguracjach `syslog` i `slack` w powyższym przykładzie. Ta opcja określa minimalny "poziom", którego komunikat musi zostać zarejestrowany przez kanał. Monolog, który obsługuje usługi rejestrowania Laravel, oferuje wszystkie poziomy dziennika zdefiniowane w [specyfikacji RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, i **debug**.

Wyobraźmy sobie, że logujemy wiadomość przy użyciu metody `debug`:

    Log::debug('An informational message.');

Biorąc pod uwagę naszą konfigurację, kanał `syslog` zapisze komunikat w dzienniku systemu; jednak ponieważ komunikat o błędzie nie jest `critical` lub wyższy, nie zostanie wysłany do Slacka. Jeśli jednak zarejestrujemy komunikat `emergency`, zostanie on wysłany zarówno do logu systemowego, jak i do Slacka, ponieważ poziom `emergency` przekracza próg minimalnego poziomu dla obu kanałów:

    Log::emergency('The system is down!');

<a name="writing-log-messages"></a>
## Writing Log Messages - Pisanie logów

Możesz zapisywać informacje w dziennikach za pomocą [fasady](/docs/{{version}}/facades) `Log`. Jak wcześniej wspomniano, rejestrator zapewnia osiem poziomów rejestrowania zdefiniowanych w [specyfikacji RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, i **debug**:

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

Możesz więc wywołać dowolną z tych metod, aby zalogować wiadomość dla odpowiedniego poziomu. Domyślnie wiadomość zostanie zapisana w domyślnym kanale dziennika skonfigurowanym przez plik konfiguracyjny `config/logging.php`:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

#### Contextual Information - Informacja kontekstowa

Tablica danych kontekstowych może również zostać przekazana do metod rejestrowania. Dane kontekstowe zostaną sformatowane i wyświetlone wraz z komunikatem dziennika:

    Log::info('User failed to login.', ['id' => $user->id]);

<a name="writing-to-specific-channels"></a>
### Writing To Specific Channels - Pisanie do określonych kanałów

Czasami możesz chcieć zarejestrować wiadomość na kanale innym niż domyślny kanał aplikacji. Możesz użyć metody `channel` na elewacji `Log`, aby pobrać i zalogować się do dowolnego kanału zdefiniowanego w pliku konfiguracyjnym:

    Log::channel('slack')->info('Something happened!');

Jeśli chcesz utworzyć stos rejestrowania na żądanie składający się z wielu kanałów, możesz użyć metody `stack`:

    Log::stack(['single', 'slack'])->info('Something happened!');

<a name="customizing-monolog-for-channels"></a>
## Customizing Monolog For Channels - Dostosowywanie Monologa dla kanałów

Czasami możesz potrzebować pełnej kontroli nad konfiguracją Monolog dla istniejącego kanału. Na przykład możesz chcieć skonfigurować niestandardową implementację Monolog `FormatterInterface` dla programów obsługi danego kanału.

Aby rozpocząć, zdefiniuj tablicę `tap` w konfiguracji kanału. Tablica `tap` powinna zawierać listę klas, które powinny mieć możliwość dostosowania (lub "puknięcia" przez) instancji Monolog po jej utworzeniu:

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],

Po skonfigurowaniu opcji `tap` na kanale, możesz zdefiniować klasę, która dostosuje twoją instancję Monolog. Ta klasa wymaga tylko jednej metody: `__invoke`, która odbiera instancję Monolog:

    <?php

    namespace App\Logging;

    class CustomizeFormatter
    {
        /**
         * Customize the given Monolog instance.
         *
         * @param  \Monolog\Logger
         * @return void
         */
        public function __invoke($monolog)
        {
            foreach ($monolog->getHandlers() as $handler) {
                $handler->setFormatter(...);
            }
        }
    }

> {tip} Wszystkie klasy "puknij" są rozwiązywane przez [kontener usługi](/docs/{{version}}/container), więc wszelkie zależności konstruktorów, których wymagają, będą automatycznie wprowadzane.

<a name="creating-custom-channels"></a>
## Creating Custom Channels - Tworzenie niestandardowych kanałów

Jeśli chcesz zdefiniować całkowicie niestandardowy kanał, w którym masz pełną kontrolę nad tworzeniem i konfiguracją Monologa, możesz określić typ sterownika `custom` w swoim pliku konfiguracyjnym `config/logging.php`. Ponadto twoja konfiguracja powinna zawierać opcję `via`, która określa klasę, która powinna zostać wywołana w celu utworzenia instancji Monolog:

    'channels' => [
        'custom' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

Po skonfigurowaniu kanału `custom` możesz przystąpić do zdefiniowania klasy, która utworzy instancję Monolog. Ta klasa wymaga tylko jednej metody: `__invoke`, która powinna zwrócić instancję Monolog:

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * Create a custom Monolog instance.
         *
         * @return \Monolog\Logger
         */
        public function __invoke()
        {
            return new Logger(...);
        }
    }
