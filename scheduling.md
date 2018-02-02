# Task Scheduling

- [Introduction - Wprowadzenie](#introduction)
- [Defining Schedules - Definiowanie harmonogramów](#defining-schedules)
    - [Scheduling Artisan Commands - Planowanie poleceń Artisan](#scheduling-artisan-commands)
    - [Scheduling Queued Jobs - Planowanie zadań w kolejce](#scheduling-queued-jobs)
    - [Scheduling Shell Commands - Planowanie poleceń powłoki](#scheduling-shell-commands)
    - [Schedule Frequency Options - Zaplanuj opcje częstotliwości](#schedule-frequency-options)
    - [Preventing Task Overlaps - Zapobieganie nakładaniu się zadań](#preventing-task-overlaps)
    - [Maintenance Mode - Tryb konserwacji](#maintenance-mode)
- [Task Output - Zadanie wyjściowe](#task-output)
- [Task Hooks - Haczyki zadań](#task-hooks)

<a name="introduction"></a>
## Introduction - Wprowadzenie

W przeszłości mogłeś wygenerować wpis Cron dla każdego zadania, które musisz zaplanować na swoim serwerze. Jednak może to szybko stać się problemem, ponieważ harmonogram zadań nie jest już kontrolowany przez źródło, a do serwera należy dodać SSH, aby dodać dodatkowe wpisy Cron.

Harmonogram poleceń Laravel pozwala ci płynnie i ekspresyjnie definiować twój harmonogram poleceń w samym Laravel-u. Podczas korzystania z programu planującego na twoim serwerze potrzebny jest tylko jeden wpis Cron. Twój harmonogram zadań jest zdefiniowany w metodzie `schedule` pliku  `app/Console/Kernel.php`. Aby pomóc Ci zacząć, prosty przykład jest zdefiniowany w ramach metody.

### Starting The Scheduler - Uruchamianie programu planującego

Podczas korzystania z programu planującego wystarczy dodać do serwera następujący wpis Cron. Jeśli nie wiesz, jak dodać wpisy Cron do swojego serwera, rozważ skorzystanie z usługi takiej jak [Laravel Forge](https://forge.laravel.com), która może zarządzać wpisami Cron:

    * * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1

Cron będzie wywoływał harmonogram poleceń Laravel co minutę. Po uruchomieniu polecenia `schedule:run` program Laravel oceni zaplanowane zadania i wykona zadania, które są należne.

<a name="defining-schedules"></a>
## Defining Schedules - Definiowanie harmonogramów

Możesz zdefiniować wszystkie zaplanowane zadania w metodzie `schedule` klasy `App\Console\Kernel` . Na początek przyjrzyjmy się przykładowi planowania zadania. W tym przykładzie zaplanujemy `Closure`, które będzie uruchamiane codziennie o północy. Wewnątrz  `Closure` będziemy wykonywać zapytanie do bazy danych, aby wyczyścić tabelę:

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * The Artisan commands provided by your application.
         *
         * @var array
         */
        protected $commands = [
            //
        ];

        /**
         * Define the application's command schedule.
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

<a name="scheduling-artisan-commands"></a>
### Scheduling Artisan Commands - Planowanie poleceń Artisan

Oprócz planowania wywołań Closure możesz również zaplanować [polecenia Artisana](/docs/{{version}}/artisan) i polecenia systemu operacyjnego. Na przykład możesz użyć metody `command`, aby zaplanować polecenie Artisan za pomocą nazwy lub klasy polecenia:

    $schedule->command('emails:send --force')->daily();

    $schedule->command(EmailsCommand::class, ['--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### Scheduling Queued Jobs - Planowanie zadań w kolejce

Metodę `job` można użyć do zaplanowania [zadania w kolejce](/docs/{{version}}/queues). Ta metoda zapewnia wygodny sposób planowania zadań bez użycia metody `call` do ręcznego tworzenia Closures w celu kolejkowania zadania:

    $schedule->job(new Heartbeat)->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### Scheduling Shell Commands - Planowanie poleceń powłoki

Do wydania polecenia do systemu operacyjnego można użyć metody `exec`:

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### Schedule Frequency Options - Zaplanuj opcje częstotliwości

Oczywiście istnieje wiele harmonogramów, które możesz przypisać do swojego zadania:

Metoda  | Opis
------------- | -------------
`->cron('* * * * * *');`  |  Uruchom zadanie w niestandardowym harmonogramie Cron
`->everyMinute();`  |  Uruchom zadanie co minutę
`->everyFiveMinutes();`  |  Uruchom zadanie co pięć minut
`->everyTenMinutes();`  |  Uruchom zadanie co dziesięć minut
`->everyFifteenMinutes();`  |  Uruchom zadanie co piętnaście minut
`->everyThirtyMinutes();`  |  Uruchom zadanie co trzydzieści minut
`->hourly();`  |  Uruchom zadanie co godzinę
`->hourlyAt(17);`  |  Uruchom zadanie co godzinę, o 17 minut po godzinie
`->daily();`  |  Uruchom zadanie codziennie o północy
`->dailyAt('13:00');`  |  Uruchom zadanie codziennie o 13:00
`->twiceDaily(1, 13);`  |  Uruchom zadanie codziennie o 1:00 i 13:00
`->weekly();`  |  Uruchom zadanie co tydzień
`->monthly();`  |  Uruchom zadanie co miesiąc
`->monthlyOn(4, '15:00');`  |  Uruchom zadanie co miesiąc 4 dnia o 15:00
`->quarterly();` |  Uruchom zadanie co kwartał
`->yearly();`  |  Uruchom zadanie co roku
`->timezone('America/New_York');` | Ustaw strefę czasową

Te metody można łączyć z dodatkowymi ograniczeniami w celu stworzenia jeszcze bardziej precyzyjnie opracowanych harmonogramów, które działają tylko w określone dni tygodnia. Na przykład, aby zaplanować uruchamianie polecenia cotygodniowego w poniedziałek:

    // Run once per week on Monday at 1 PM...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // Run hourly from 8 AM to 5 PM on weekdays...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

Poniżej znajduje się lista dodatkowych ograniczeń harmonogramu:

Metoda  | Opis
------------- | -------------
`->weekdays();`  |  Ogranicz zadanie do dni powszednich
`->sundays();`  |  Ogranicz zadanie do Niedzieli
`->mondays();`  |  Ogranicz zadanie do Poniedziałku
`->tuesdays();`  |  Ogranicz zadanie to Wtorku
`->wednesdays();`  |  Ogranicz zadanie to Środy
`->thursdays();`  |  Ogranicz zadanie to Czwartku
`->fridays();`  |  Ogranicz zadanie to Piątku
`->saturdays();`  |  Ogranicz zadanie Soboty
`->between($start, $end);`  |  Ogranicz zadanie do uruchomienia między czasem rozpoczęcia i zakończenia
`->when(Closure);`  |  Ogranicz zadanie w oparciu o test prawdy

#### Between Time Constraints - Między ograniczeniami czasowymi

Metoda `between` może być używana do ograniczenia wykonania zadania w zależności od pory dnia:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

Podobnie, można użyć metody `unlessBetween`, aby wykluczyć wykonanie zadania przez pewien okres czasu:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### Truth Test Constraints - Ograniczenia testu prawdy

Metodę `when` można użyć do ograniczenia wykonania zadania w oparciu o wynik danego testu prawdy. Innymi słowy, jeśli podane `Closure` zwraca `true`, zadanie zostanie wykonane, dopóki żadne inne warunki ograniczające uniemożliwiają uruchomienie zadania:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

Metoda `skip` może być postrzegana jako odwrotność `when`. Jeśli metoda `skip` zwraca `true`, zaplanowane zadanie nie zostanie wykonane:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

W przypadku użycia połączonych metod `when`, zaplanowane polecenie zostanie wykonane, jeśli wszystkie warunki `when` zwrócą `true`.

<a name="preventing-task-overlaps"></a>
### Preventing Task Overlaps - Zapobieganie nakładaniu się zadań

Domyślnie zaplanowane zadania będą uruchamiane, nawet jeśli poprzednia instancja zadania nadal działa. Aby temu zapobiec, możesz użyć metody `withoutOverlapping`:

    $schedule->command('emails:send')->withoutOverlapping();

W tym przykładzie `emails:send` [polecenie Artisana](/docs/{{version}}/artisan) będą uruchamiane co minutę, jeśli jeszcze nie jest uruchomiona. Metoda `withoutOverlapping` jest szczególnie użyteczna, jeśli masz zadania drastycznie różniące się czasem wykonania, co uniemożliwia dokładne przewidzenie czasu wykonania danego zadania.

W razie potrzeby możesz określić, ile minut musi upłynąć, zanim wygaśnie blokada "bez nakładania się". Domyślnie blokada wygaśnie po 24 godzinach:

    $schedule->command('emails:send')->withoutOverlapping(10);

<a name="maintenance-mode"></a>
### Maintenance Mode - Tryb konserwacji

Zadania zaplanowane przez Laravel nie będą działać, jeśli Laravel jest w [trybie konserwacji](/docs/{{version}}/configuration#maintenance-mode), ponieważ nie chcemy, aby twoje zadania kolidowały z niedokończoną konserwacją, którą możesz wykonywać na komputerze. twój serwer. Jeśli jednak chcesz wymusić uruchomienie zadania nawet w trybie konserwacji, możesz użyć metody `evenInMaintenanceMode`:

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="task-output"></a>
## Task Output - Zadanie wyjściowe

Program planujący Laravel oferuje kilka wygodnych metod pracy z danymi wyjściowymi generowanymi przez zaplanowane zadania. Po pierwsze, używając metody `sendOutputTo`, możesz wysłać dane wyjściowe do pliku w celu późniejszej inspekcji:

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

Jeśli chcesz dołączyć dane wyjściowe do danego pliku, możesz użyć metody `appendOutputTo`:

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

Korzystając z metody `emailOutputTo`, możesz wysłać wiadomość e-mail z danymi wyjściowymi na wybrany adres e-mail. Przed wysłaniem wiadomości e-mail do wyjścia zadania należy skonfigurować [usługi e-mailowe](/docs/{{version}}/mail) Laravel:

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> {note} Metody `emailOutputTo`, `sendOutputTo` i `appendOutputTo` są wyłącznie dla metody `command` i nie są obsługiwane dla `call`.

<a name="task-hooks"></a>
## Task Hooks - Haczyki zadań

Korzystając z metod `before` i `after`, możesz określić kod, który ma zostać wykonany przed zakończeniem zaplanowanego zadania i po jego zakończeniu:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

#### Pinging URLs - Pingujące adresy URL

Korzystając z metod `pingBefore` i `thenPing`, program planujący może automatycznie pingować dany URL przed zakończeniem lub po zakończeniu zadania. Ta metoda jest przydatna do powiadamiania usługi zewnętrznej, takiej jak [Laravel Envoyer](https://envoyer.io), że zaplanowane zadanie rozpoczyna się lub zakończyło wykonanie:

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

Użycie funkcji  `pingBefore($url)` lub `thenPing($url)` wymaga biblioteki HTTP Guzzle. Możesz dodać Guzzle do swojego projektu za pomocą menedżera pakietów Composer:

    composer require guzzlehttp/guzzle
