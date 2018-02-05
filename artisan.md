# Artisan Console

- [Introduction - Wprowadzenie](#introduction)
- [Writing Commands - Pisanie poleceń](#writing-commands)
    - [Generating Commands - Generowanie poleceń](#generating-commands)
    - [Command Structure - Struktura dowodzenia](#command-structure)
    - [Closure Commands - Polecenia Closure](#closure-commands)
- [Defining Input Expectations - Definiowanie oczekiwań wejściowych](#defining-input-expectations)
    - [Arguments - Argumenty](#arguments)
    - [Options - Opcje](#options)
    - [Input Arrays - Tablice wejściowe](#input-arrays)
    - [Input Descriptions - Opisy wejściowe](#input-descriptions)
- [Command I/O - Polecenie Wejścia/Wyjścia](#command-io)
    - [Retrieving Input - Pobieranie danych wejściowych](#retrieving-input)
    - [Prompting For Input - Podpowiedzi dla wejścia](#prompting-for-input)
    - [Writing Output - Pisanie wyjścia](#writing-output)
- [Registering Commands - Rejestrowanie poleceń](#registering-commands)
- [Programmatically Executing Commands - Programowanie wykonywania poleceń](#programmatically-executing-commands)
    - [Calling Commands From Other Commands - Wywoływanie poleceń z innych poleceń](#calling-commands-from-other-commands)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Artisan to interfejs wiersza poleceń dołączony do Laravel. Udostępnia szereg pomocnych poleceń, które mogą pomóc w budowaniu aplikacji. Aby wyświetlić listę wszystkich dostępnych poleceń Artisan, możesz użyć polecenia `list`:

    php artisan list

Każde polecenie zawiera również ekran "pomocy", który wyświetla i opisuje dostępne argumenty i opcje polecenia. Aby wyświetlić ekran pomocy, poprzedź nazwę polecenia za pomocą `help`:

    php artisan help migrate

#### Laravel REPL

Wszystkie aplikacje Laravel obejmują Tinker, REPL z pakietem [PsySH](https://github.com/bobthecow/psysh). Tinker pozwala na interakcję z całą aplikacją Laravel na linii poleceń, w tym Eloquent ORM, zadania, zdarzenia i wiele innych. Aby wejść do środowiska Tinker, uruchom polecenie `tinker` Artisan:

    php artisan tinker

<a name="writing-commands"></a>
## Writing Commands - Pisanie poleceń

Oprócz poleceń dostarczanych z Artisan możesz także tworzyć własne niestandardowe polecenia. Polecenia są zwykle przechowywane w katalogu `app/Console/Commands`; możesz jednak wybrać własną lokalizację pamięci, o ile polecenia mogą być ładowane przez program Composer.

<a name="generating-commands"></a>
### Generating Commands - Generowanie poleceń

Aby utworzyć nowe polecenie, użyj polecenia `make:command` Artisan-a. To polecenie utworzy nową klasę poleceń w katalogu `app/Console/Commands`. Nie martw się, jeśli ten katalog nie istnieje w twojej aplikacji, ponieważ zostanie utworzony przy pierwszym uruchomieniu polecenia `make:command` Artisan-a. Wygenerowane polecenie będzie zawierało domyślny zestaw właściwości i metod obecnych we wszystkich poleceniach:

    php artisan make:command SendEmails

<a name="command-structure"></a>
### Command Structure - Struktura dowodzenia

Po wygenerowaniu polecenia, należy wypełnić właściwości `signature` i `description` klasy, które będą używane podczas wyświetlania komendy na ekranie `list`. Metoda `handle` zostanie wywołana po wykonaniu polecenia. Możesz umieścić swoją logikę poleceń w tej metodzie.

> {tip} W celu większego ponownego użycia kodu dobrze jest zachować polecenia konsoli i pozwolić im odroczyć usługi aplikacji w celu wykonania swoich zadań. W poniższym przykładzie zwróć uwagę, że wprowadzamy klasę usług, aby wykonać "ciężkie podnoszenie" (heavy lifting) wysyłania wiadomości e-mail.

Rzućmy okiem na przykładowe polecenie. Zauważ, że jesteśmy w stanie wstrzyknąć wszelkie potrzebne zależności do konstruktora polecenia. Laravel [kontener usługi](/docs/{{version}}/container) automatycznie wstrzyknie wszystkie zależności wskazane w konstruktorze:

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### Closure Commands - Polecenia Closure

Polecenia oparte na Closure stanowią alternatywę dla definiowania poleceń konsoli jako klas. W ten sam sposób, w jaki zamknięcie trasy jest alternatywą dla kontrolerów, pomyśl o poleceniu Closure jako alternatywy dla klas dowodzenia. W metodzie `commands` pliku `app/Console/Kernel.php` Laravel ładuje plik `routes/console.php`:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Mimo że ten plik nie definiuje tras HTTP, definiuje punkty wejścia (trasy) oparte na konsoli do twojej aplikacji. W tym pliku możesz zdefiniować wszystkie trasy oparte na Closure za pomocą metody `Artisan::command`. Metoda `command` przyjmuje dwa argumenty: [sygnatura polecenia](#defining-input-expectations) i Closure, które odbiera argumenty poleceń i opcje:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

Closure jest powiązane z podstawową instancją polecenia, więc masz pełny dostęp do wszystkich metod pomocniczych, do których zwykle będziesz mógł uzyskać dostęp z pełną klasą poleceń.

#### Type-Hinting Dependencies - Zależności wstrzyknięte wskazówką

Oprócz otrzymywania argumentów i opcji polecenia Closures może również wpisać-wskazówkę dodatkowych zależności, które chciałbyś rozwiązać z [kontenera usług](/docs/{{version}}/container):

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### Closure Command Descriptions - Opisy poleceń zamknięcia

Podczas definiowania polecenia opartego na Closure można użyć metody `describe` w celu dodania opisu do polecenia. Ten opis będzie wyświetlany po uruchomieniu poleceń `php artisan list` lub `php artisan help`:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## Defining Input Expectations - Definiowanie oczekiwań wejściowych

Podczas pisania komend konsolowych powszechne jest zbieranie danych wejściowych od użytkownika za pomocą argumentów lub opcji. Laravel bardzo ułatwia definiowanie danych wejściowych, których użytkownik oczekuje od użytkownika za pomocą właściwości `signature` w komendach. Właściwość `signature` umożliwia zdefiniowanie nazwy, argumentów i opcji dla polecenia w pojedynczej, wyrazistej, przypominającej trasę składni.

<a name="arguments"></a>
### Arguments - Argumenty

Wszystkie dostarczone przez użytkownika argumenty i opcje są zawijane w nawiasy klamrowe. W poniższym przykładzie polecenie definiuje jeden **wymagany** argument: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

Możesz także ustawić argumenty jako opcjonalne i zdefiniować wartości domyślne dla argumentów:

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### Options - Opcje

Opcje, takie jak argumenty, są inną formą wprowadzania danych przez użytkownika. Opcje są poprzedzane dwoma łącznikami (`--`), gdy są określone w wierszu poleceń. Istnieją dwa rodzaje opcji: te, które otrzymują wartość i te, które jej nie mają. Opcje, które nie otrzymują wartości, służą jako "przełącznik" boolean. Rzućmy okiem na przykład tego typu opcji:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

W tym przykładzie przełącznik `--queue` może zostać określony podczas wywoływania polecenia Artisan. Jeśli zostanie przekazany przełącznik `--queue`, wartością opcji będzie  `true`. W przeciwnym razie wartością będzie `false`:

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### Options With Values - Opcje z wartościami

Następnie przyjrzyjmy się opcji, która oczekuje wartości. Jeśli użytkownik musi podać wartość dla opcji, należy nadać jej nazwę ze znakiem `=`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

W tym przykładzie użytkownik może przekazać wartość dla opcji takiej jak:

    php artisan email:send 1 --queue=default

Możesz przypisać wartości domyślne do opcji, określając domyślną wartość po nazwie opcji. Jeśli żadna wartość opcji nie zostanie przekazana przez użytkownika, zostanie użyta wartość domyślna:

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### Option Shortcuts - Skróty opcji

Aby przypisać skrót podczas definiowania opcji, możesz określić ją przed nazwą opcji i użyć znaku | ogranicznik w celu oddzielenia skrótu od pełnej nazwy opcji:

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### Input Arrays - Tablice wejściowe

Jeśli chciałbyś zdefiniować argumenty lub opcje, aby oczekiwać wejść tablicowych, możesz użyć znaku `*`. Najpierw przyjrzyjmy się przykładowi, który określa argument tablicowy:

    email:send {user*}

Podczas wywoływania tej metody argumenty `user` mogą być przekazywane do wiersza poleceń. Na przykład poniższe polecenie ustawi wartość `user` na `['foo', 'bar'] `:

    php artisan email:send foo bar

Podczas definiowania opcji, która oczekuje wejścia tablicy, każdej wartości opcji przekazanej do polecenia należy poprzedzić nazwą opcji:

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### Input Descriptions - Opisy wejściowe

Możesz przypisać opisy do argumentów i opcji, oddzielając parametr od opisu za pomocą dwukropka. Jeśli potrzebujesz trochę więcej miejsca na zdefiniowanie swojego polecenia, możesz podzielić definicję na wiele linii:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## Command I/O - Polecenie Wejścia/Wyjścia

<a name="retrieving-input"></a>
### Retrieving Input - Pobieranie danych wejściowych

Podczas wykonywania polecenia, będziesz oczywiście musiał uzyskać dostęp do wartości argumentów i opcji zaakceptowanych przez twoje polecenie. Aby to zrobić, możesz użyć metod `argument` i `option`:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

Jeśli potrzebujesz pobrać wszystkie argumenty jako `tablica`, wywołaj metodę` arguments`:

    $arguments = $this->arguments();

Opcje mogą być pobierane równie łatwo jak argumenty za pomocą metody `option`. Aby pobrać wszystkie opcje jako tablicę, wywołaj metodę `options`:

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

Jeśli argument lub opcja nie istnieje, zwracane jest `null`.

<a name="prompting-for-input"></a>
### Prompting For Input - Podpowiedzi dla wejścia

Oprócz wyświetlania danych wyjściowych możesz również poprosić użytkownika o podanie danych wejściowych podczas wykonywania polecenia. Metoda `ask` poprosi użytkownika o dane pytanie, zaakceptuje jego dane wejściowe, a następnie zwróci dane użytkownika z powrotem do polecenia:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

Metoda `secret` jest podobna do `ask`, ale dane wejściowe użytkownika nie będą dla nich widoczne podczas pisania w konsoli. Ta metoda jest przydatna przy pytaniu o poufne informacje, takie jak hasło:

    $password = $this->secret('What is the password?');

#### Asking For Confirmation - Pytanie o potwierdzenie

Jeśli chcesz poprosić użytkownika o proste potwierdzenie, możesz użyć metody `confirm`. Domyślnie ta metoda zwraca `false`. Jednakże, jeśli użytkownik wpisze `y` lub `yes` w odpowiedzi na monit, metoda zwróci wartość `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### Auto-Completion - Automatyczne uzupełnianie

Metodę `anticipate` można zastosować do automatycznego uzupełniania dla możliwych wyborów. Użytkownik wciąż może wybrać dowolną odpowiedź, niezależnie od wskazówek dotyczących automatycznego uzupełniania:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### Multiple Choice Questions - Pytania wielokrotnego wyboru

Jeśli chcesz dać użytkownikowi predefiniowany zestaw opcji, możesz użyć metody `choice`. Możesz ustawić indeks tablicy wartości domyślnej, która ma zostać zwrócona, jeśli nie wybrano żadnej opcji:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

<a name="writing-output"></a>
### Writing Output - Pisanie wyjścia

Aby wysłać dane wyjściowe do konsoli, użyj metod `line`, `info`, `comment`, `question` i `error`. Każda z tych metod będzie wykorzystywać odpowiednie kolory ANSI do swoich celów. Na przykład, wyświetlmy użytkownikowi ogólne informacje. Zazwyczaj metoda `info` wyświetla się w konsoli jako zielony tekst:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

Aby wyświetlić komunikat o błędzie, użyj metody `error`. Tekst komunikatu o błędzie jest zazwyczaj wyświetlany na czerwono:

    $this->error('Something went wrong!');

Jeśli chcesz wyświetlić proste, bezbarwne wyjście konsoli, użyj metody `line`:

    $this->line('Display this on the screen');

#### Table Layouts - Układy tabel

Metoda `table` ułatwia prawidłowe formatowanie wielu wierszy / kolumn danych. Wystarczy przekazać nagłówki i wiersze do metody. Szerokość i wysokość będą obliczane dynamicznie na podstawie danych:

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### Progress Bars - Paski postępu

W przypadku długotrwałych zadań pomocne może być wyświetlenie wskaźnika postępu. Za pomocą obiektu wyjściowego możemy uruchomić, przesuwać i zatrzymywać pasek postępu. Najpierw zdefiniuj całkowitą liczbę kroków, które proces będzie przetwarzał. Następnie przesuń pasek postępu po przetworzeniu każdego elementu:

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

Aby uzyskać bardziej zaawansowane opcje, zapoznaj się z [dokumentacją komponentu Symfony Progress Bar](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Registering Commands - Rejestrowanie poleceń

Ze względu na wywołanie metody `load` w poleceniu `command` jądra konsoli, wszystkie polecenia z katalogu `app/Console/Commands` zostaną automatycznie zarejestrowane w Artisan. W rzeczywistości możesz wykonywać dodatkowe wywołania metody `load`, aby skanować inne katalogi dla poleceń Artisan:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

Możesz również ręcznie zarejestrować polecenia, dodając ich nazwę klasy do właściwości `$commands` pliku `app/Console/Kernel.php`. Gdy Artisan uruchamia się, wszystkie polecenia wymienione w tej właściwości zostaną rozwiązane przez [kontener usługi](/docs/{{version}}/container) i zarejestrowane przez Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Programmatically Executing Commands - Programowanie wykonywania poleceń

Czasami możesz chcieć wykonać polecenie Artisan poza CLI. Na przykład, możesz chcieć wystrzelić polecenie Artisan-a z trasy lub kontrolera. Możesz użyć metody `call` na fasadzie `Artisan`, aby to osiągnąć. Metoda `call` przyjmuje nazwę polecenia jako pierwszy argument, a tablica parametrów polecenia jako drugi argument. Kod powrotu zostanie zwrócony:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Używając metody `queue` na elewacji `Artisan`, możesz nawet kolejkować polecenia Artisan, aby były przetwarzane w tle przez [pracowników kolejki](/docs/{{version}}/queues). Przed użyciem tej metody upewnij się, że skonfigurowano kolejkę i uruchomiono moduł nasłuchujący kolejki:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Możesz również określić połączenie lub kolejkę, do której polecenie Artisan powinno zostać wysłane:

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### Passing Array Values - Przekazywanie tablicy wartości

Jeśli twoje polecenie definiuje opcję, która akceptuje tablicę, możesz przekazać tablicę wartości do tej opcji:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### Passing Boolean Values - Przekazywanie wartości logicznych

Jeśli musisz podać wartość opcji, która nie akceptuje wartości ciągu, takich jak flaga `--force` w poleceniu `migrate:refresh`, powinieneś podać `true` lub `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### Calling Commands From Other Commands - Wywoływanie poleceń z innych poleceń

Czasami możesz chcieć wywołać inne polecenia z istniejącego polecenia Artisana. Możesz to zrobić za pomocą metody `call`. Ta metoda `call` akceptuje nazwę polecenia i tablicę parametrów polecenia:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

Jeśli chcesz wywołać inne polecenie konsoli i wyłączyć całe jego wyjście, możesz użyć metody `callSilent`. Metoda `callSilent` ma taki sam podpis jak metoda` call`:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
