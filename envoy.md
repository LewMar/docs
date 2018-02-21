# Envoy Task Runner

- [Introduction - Wprowadzenie](#introduction)
    - [Installation - Instalacja](#installation)
- [Writing Tasks - Pisanie zadań](#writing-tasks)
    - [Setup - Ustawienia](#setup)
    - [Variables - Zmienne](#variables)
    - [Stories - Kondygnacje](#stories)
    - [Multiple Servers - Wiele serwerów](#multiple-servers)
- [Running Tasks - Uruchamianie zadań](#running-tasks)
    - [Confirming Task Execution - Potwierdzanie wykonania zadania](#confirming-task-execution)
- [Notifications - Powiadomienia](#notifications)
    - [Slack](#slack)

<a name="introduction"></a>
## Introduction - Wprowadzenie

[Laravel Envoy](https://github.com/laravel/envoy) zapewnia czystą, minimalną składnię do definiowania typowych zadań uruchamianych na serwerach zdalnych. Używając składni stylu Blade, możesz łatwo skonfigurować zadania do wdrożenia, polecenia Artisan i wiele więcej. Obecnie Envoy obsługuje tylko systemy operacyjne Mac i Linux.

<a name="installation"></a>
### Installation - Instalacja

Najpierw zainstaluj Envoy za pomocą komendy Compeser `global require`:

    composer global require laravel/envoy

Ponieważ globalne biblioteki Composer mogą czasami powodować konflikty wersji pakietu, możesz rozważyć użycie `cgr`, który jest zastępczym zamiennikiem dla polecenia `composer global require`. Instrukcje instalacji biblioteki `cgr` można znaleźć na [GitHub](https://github.com/consolidation-org/cgr).

> {note} Upewnij się, że umieściłeś katalog `~/.composer/vendor/bin` w PATH, więc plik wykonywalny `envoy` zostanie znaleziony podczas uruchamiania komendy `envoy` w twoim terminalu.

#### Updating Envoy - Aktualizowanie Envoy

Możesz również użyć Composer, aby zachować aktualność instalacji Envoy. Wydanie komendy `composer global update` zaktualizuje wszystkie zainstalowane globalnie pakiety Composer:

    composer global update

<a name="writing-tasks"></a>
## Writing Tasks - Pisanie zadań

Wszystkie twoje zadania Envoy powinny być zdefiniowane w pliku `Envoy.blade.php` w katalogu głównym projektu. Oto przykład, aby zacząć:

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

Jak widać, tablica `@servers` jest zdefiniowana na górze pliku, pozwalając ci odwoływać się do tych serwerów w opcji `on` deklaracji zadań. W ramach deklaracji `@task` powinieneś umieścić kod Bash, który powinien działać na Twoim serwerze po wykonaniu zadania.

Możesz wymusić uruchomienie skryptu lokalnie, określając adres IP serwera jako `127.0.0.1`:

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### Setup - Ustawienia

Czasami może zajść potrzeba wykonania kodu PHP przed wykonaniem zadań Envoy. Możesz użyć dyrektywy ```@setup``` do zadeklarowania zmiennych i wykonania innych ogólnych prac PHP zanim wykonasz inne zadania:

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

Jeśli potrzebujesz innych plików PHP przed wykonaniem zadania, możesz użyć dyrektywy `@include` na górze pliku `Envoy.blade.php`:

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

<a name="variables"></a>
### Variables - Zmienne

W razie potrzeby możesz przekazywać wartości opcji do zadań Envoy przy użyciu wiersza poleceń:

    envoy run deploy --branch=master

Możesz uzyskać dostęp do opcji w swoich zadaniach za pomocą "echa" Blade'a. Oczywiście możesz również używać instrukcji i pętli `if` w swoich zadaniach. Na przykład zweryfikujmy obecność zmiennej `$branch` przed wykonaniem polecenia` git pull`:

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site

        @if ($branch)
            git pull origin {{ $branch }}
        @endif

        php artisan migrate
    @endtask

<a name="stories"></a>
### Stories - Kondygnacje

Kondygnacje grupują zestaw zadań pod jedną wygodną nazwą, pozwalając grupować małe, skoncentrowane zadania w duże zadania. Na przykład kondygnacja `deploy` może uruchamiać zadania `git` i` composer`, wymieniając nazwy zadań w ramach definicji:

    @servers(['web' => '192.168.1.1'])

    @story('deploy')
        git
        composer
    @endstory

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

Po napisaniu kondygnacji możesz uruchomić ją tak, jak typowe zadanie:

    envoy run deploy

<a name="multiple-servers"></a>
### Multiple Servers - Wiele serwerów

Envoy pozwala łatwo uruchomić zadanie na wielu serwerach. Najpierw dodaj dodatkowe serwery do deklaracji `@servers`. Każdemu serwerowi należy nadać unikalną nazwę. Po zdefiniowaniu dodatkowych serwerów, wyświetl listę wszystkich serwerów w tablicy `on` zadania:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### Parallel Execution - Równoległe wykonanie

Domyślnie zadania będą wykonywane kolejno na każdym serwerze. Innymi słowy, zadanie zostanie zakończone na pierwszym serwerze, zanim rozpocznie się wykonywanie na drugim serwerze. Jeśli chcesz uruchomić zadanie na wielu serwerach równolegle, dodaj opcję "parallel" do deklaracji zadania:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## Running Tasks - Uruchamianie zadań

Aby uruchomić zadanie lub kondygnację zdefiniowaną w pliku `Envoy.blade.php`, wykonaj polecenie `run` Envoy-aa, przekazując nazwę zadania lub historii, którą chcesz wykonać. Envoy uruchomi zadanie i wyświetli dane wyjściowe z serwerów podczas wykonywania zadania:

    envoy run task

<a name="confirming-task-execution"></a>
### Confirming Task Execution - Potwierdzanie wykonania zadania

Jeśli chciałbyś otrzymać prośbę o potwierdzenie przed uruchomieniem danego zadania na swoich serwerach, powinieneś dodać do deklaracji zadania dyrektywę `confirm`. Ta opcja jest szczególnie przydatna w przypadku destrukcyjnych operacji:

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="notifications"></a>
## Notifications - Powiadomienia

<a name="slack"></a>
### Slack

Envoy obsługuje także wysyłanie powiadomień do [Slack](https://slack.com) po wykonaniu każdego zadania. Dyrektywa `@slack` akceptuje Slack hook URL i nazwę kanału. Możesz pobrać swój URL webhook, tworząc integrację "Przychodzące WebHooks" w panelu sterowania Slack. Powinieneś przekazać cały URL webhooka do dyrektywy `@slack`:

    @finished
        @slack('webhook-url', '#bots')
    @endfinished

Możesz podać jeden z następujących argumentów jako argument kanału:

<div class="content-list" markdown="1">
- To send the notification to a channel: `#channel`
- To send the notification to a user: `@user`
</div>