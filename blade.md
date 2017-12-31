# Blade Templates

- [Introduction - Wprowadzenie](#introduction)
- [Template Inheritance - Dziedziczenie szablonów](#template-inheritance)
    - [Defining A Layout - Definiowanie układu](#defining-a-layout)
    - [Extending A Layout - Rozszerzanie układu](#extending-a-layout)
- [Components & Slots - Komponenty i gniazda](#components-and-slots)
- [Displaying Data - Wyświetlanie danych](#displaying-data)
    - [Blade & JavaScript Frameworks - Blade i fremework-ki JavaScript-owe](#blade-and-javascript-frameworks)
- [Control Structures - Struktury kontrolne](#control-structures)
    - [If Statements - Deklaracje Jeżeli](#if-statements)
    - [Switch Statements - Deklaracje Przełączania](#switch-statements)
    - [Loops - Pętle](#loops)
    - [The Loop Variable - Zmienna pętli](#the-loop-variable)
    - [Comments - Komentarze](#comments)
    - [PHP](#php)
- [Including Sub-Views Dodawanie pod widoków](#including-sub-views)
    - [Rendering Views For Collections - Rendering widoków dla kolekcji](#rendering-views-for-collections)
- [Stacks - Półki na książki](#stacks)
- [Service Injection - Wstrzyknięcie usługi](#service-injection)
- [Extending Blade - Rozszerzenie Blade](#extending-blade)
    - [Custom If Statements - Deklaracja niestandardowe jeżeli](#custom-if-statements)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Blade to prosty, ale potężny silnik szablonowy dostarczany z Laravel. W przeciwieństwie do innych popularnych silników szablonowych PHP, Blade nie ogranicza używania zwykłego kodu PHP w widokach. W rzeczywistości, wszystkie widoki Blade są kompilowane do zwykłego kodu PHP i przechowywane w pamięci podręcznej, dopóki nie zostaną zmodyfikowane, co oznacza, że Blade dodaje w zasadzie zero narzutów do twojej aplikacji. Pliki widoku używają rozszerzenia pliku `.blade.php` i zwykle są przechowywane w katalogu` resources / views`.

<a name="template-inheritance"></a>
## Template Inheritance - Dziedziczenie szablonów

<a name="defining-a-layout"></a>
### Defining A Layout - Definiowanie układu

Two of the primary benefits of using Blade are _template inheritance_ and _sections_. To get started, let's take a look at a simple example. First, we will examine a "master" page layout. Since most web applications maintain the same general layout across various pages, it's convenient to define this layout as a single Blade view:
Dwie główne korzyści z używania Blade to _szablon dziedziczenia_ i _sekcji_. Na początek rzućmy okiem na prosty przykład. Najpierw sprawdzimy układ strony "master". Ponieważ większość aplikacji internetowych zachowuje ten sam ogólny układ na różnych stronach, wygodnie jest zdefiniować ten układ jako pojedynczy widok Blade:

    <!-- Stored in resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

Jak widać, plik ten zawiera typowe oznaczenia HTML. Należy jednak zwrócić uwagę na dyrektywy `@section` i `@yield`. Dyrektywa `@section`, jak sama nazwa wskazuje, definiuje sekcję treści, podczas gdy dyrektywa `@yield` służy do wyświetlania zawartości danej sekcji.

Po zdefiniowaniu układu dla naszej aplikacji określmy stronę podrzędną, która dziedziczy układ.

<a name="extending-a-layout"></a>
### Extending A Layout - Rozszerzanie układu

Podczas definiowania widoku podrzędnego użyj dyrektywy Blade `@extends`, aby określić, który widok podrzędny powinien "dziedziczyć". Widoki, które rozszerzają układ Blade, mogą wprowadzać zawartość do sekcji layoutu za pomocą dyrektyw `@section`. Pamiętaj, jak widać w powyższym przykładzie, zawartość tych sekcji zostanie wyświetlona w układzie za pomocą `@yield`:

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

In this example, the `sidebar` section is utilizing the `@@parent` directive to append (rather than overwriting) content to the layout's sidebar. The `@@parent` directive will be replaced by the content of the layout when the view is rendered.
W tym przykładzie sekcja `sidebar` używa dyrektywy `@@parent` do dołączania (zamiast nadpisywania) zawartości do paska bocznego układu. Dyrektywa `@@parent` zostanie zastąpiona zawartością układu podczas renderowania widoku.

> {tip} W przeciwieństwie do poprzedniego przykładu sekcja `sidebar` kończy się `@endsection` zamiast `@show`. Dyrektywa `@endsection` definiuje tylko sekcję, podczas gdy `@show` definiuje i **natychmiast ustepuje** sekcji.

Widoki blade mogą być zwracane z tras za pomocą globalnego helpera `view`:

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## Components & Slots - Komponenty i gniazda

Komponenty i gniazda zapewniają podobne korzyści dla sekcji i układów; jednak niektórzy mogą uznać, że mentalny model komponentów i gniazd jest łatwiejszy do zrozumienia. Po pierwsze, wyobraźmy sobie komponent "alert" wielokrotnego użytku, który chcielibyśmy ponownie wykorzystać w naszej aplikacji:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

Zmienna `{{ $slot }}` zawiera treść, którą chcemy wprowadzić do komponentu. Teraz, aby skonstruować ten komponent, możemy użyć dyrektywy Blade `@component`:

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent

Czasami pomocne jest zdefiniowanie wielu gniazd dla komponentu. Zmodyfikujmy nasz komponent alertu, aby umożliwić wstrzyknięcie "title". Nazwane gniazda mogą być wyświetlane po prostu "echo" zmiennej, która pasuje do ich nazwy:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

Teraz możemy wstrzyknąć zawartość do nazwanego gniazda, używając dyrektywy `@slot`. Wszelkie treści spoza dyrektywy `@slot` zostaną przekazane do składnika w zmiennej `$slot`:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### Passing Additional Data To Components - Przekazywanie dodatkowych danych do komponentów

Czasami może być konieczne przekazanie dodatkowych danych do komponentu. Z tego powodu możesz przekazać tablicę danych jako drugi argument do dyrektywy `@component`. Wszystkie dane zostaną udostępnione szablonowi komponentu jako zmienne:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

<a name="displaying-data"></a>
## Displaying Data - Wyświetlanie danych

Możesz wyświetlać dane przekazywane do widoków Blade, otaczajać zmienną w nawiasy klamrowe. Na przykład, biorąc pod uwagę następującą trasę:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Możesz wyświetlić zawartość zmiennej `name` w następujący sposób:

    Hello, {{ $name }}.

Oczywiście nie ogranicza się do wyświetlania zawartości zmiennych przekazywanych do widoku. Możesz również wywołać echo wyników dowolnej funkcji PHP. W rzeczywistości możesz umieścić dowolny kod PHP wewnątrz instrukcji echa Blade:

    The current UNIX timestamp is {{ time() }}.

> {tip} Instrukcje Blade `{{ }}` są automatycznie wysyłane za pośrednictwem funkcji PHP `htmlspecialchars`, aby zapobiec atakom XSS.

#### Displaying Unescaped Data - Wyświetlanie danych bez nadzoru

Domyślnie instrukcje Blade `{{ }}` są automatycznie przesyłane przez funkcję `htmlspecialchars` PHP, aby zapobiec atakom XSS. Jeśli nie chcesz, aby twoje dane zostały usunięte, możesz użyć następującej składni:

    Hello, {!! $name !!}.

> {note} Zachowaj ostrożność, powtarzając treść dostarczaną przez użytkowników aplikacji. Zawsze stosuj składnię klamrową ze zmienioną sekwencją, aby zapobiec atakom XSS podczas wyświetlania danych dostarczanych przez użytkownika.

#### Rendering JSON - Renderowanie JSON

Czasami możesz przekazać tablicę do widoku z zamiarem renderowania jej jako JSON w celu zainicjowania zmiennej JavaScript. Na przykład:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

Jednak zamiast ręcznie wywoływać `json_encode`, możesz użyć dyrektywy `@json` Blade:

    <script>
        var app = @json($array);
    </script>

<a name="blade-and-javascript-frameworks"></a>
### Blade & JavaScript Frameworks - Blade i fremework-ki JavaScript-owe

Ponieważ wiele frameworków JavaScript również używa nawiasów klamrowych, aby wskazać, że dane wyrażenie powinno być wyświetlane w przeglądarce, możesz użyć symbolu `@', aby poinformować silnik renderujący Blade, że wyrażenie powinno pozostać nietknięte. Na przykład:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

W tym przykładzie symbol `@` zostanie usunięty przez Blade; jednak wyrażenie `{{ name }}` pozostanie niezmienione przez mechanizm Blade, pozwalając na jego renderowanie przez twoją strukturę JavaScript.

#### The `@verbatim` Directive - Dyrektywa `@verbatim`

Jeśli wyświetlasz zmienne JavaScript w dużej części twojego szablonu, możesz zawijać kod HTML w dyrektywie `@verbatim`, dzięki czemu nie musisz prefiksować każdej instrukcji echa Blade z symbolem `@`:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## Control Structures - Struktury kontrolne

Oprócz dziedziczenia szablonów i wyświetlania danych, Blade zapewnia również wygodne skróty do popularnych struktur kontrolnych PHP, takich jak instrukcje warunkowe i pętle. Skróty te zapewniają bardzo czysty, zwięzły sposób pracy ze strukturami kontrolnymi PHP, a jednocześnie są znane ich odpowiednikom PHP.

<a name="if-statements"></a>
### If Statements - Deklaracje Jeżeli

Możesz konstruować instrukcje `if` używając dyrektyw `@if`, `@elseif`, `@else`, i `@endif`. Te dyrektywy działają identycznie jak ich odpowiedniki PHP:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

Dla wygody Blade dostarcza także dyrektywę "chyba żę" `@unless`:

    @unless (Auth::check())
        You are not signed in.
    @endunless

Oprócz już omówionych dyrektyw warunkowych, dyrektywy `@isset` i `@empty` mogą być używane jako wygodne skróty do odpowiednich funkcji PHP:

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### Authentication Shortcuts - Skróty uwierzytelniania

Dyrektywy `@auth` i `@guest` mogą być użyte do szybkiego ustalenia, czy bieżący użytkownik jest uwierzytelniony, czy jest gościem:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

Jeśli jest taka potrzeba, możesz określić [zabezpieczenie autoryzacji](/docs/{{version}}/authentication), które powinno zostać sprawdzone podczas używania dyrektyw `@ auth` i `@guest`:

    @auth('admin')
        // The user is authenticated...
    @endauth

    @guest('admin')
        // The user is not authenticated...
    @endguest

<a name="switch-statements"></a>
### Switch Statements - Deklaracje Przełączania

Instrukcje switch można konstruować przy użyciu dyrektyw `@switch`, `@case`, `@break`, `@default` i `@endswitch`:

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>
### Loops - Pętle

Oprócz instrukcji warunkowych, Blade dostarcza prostych dyrektyw do pracy ze strukturami pętli PHP. Ponownie, każda z tych dyrektyw działa identycznie jak ich odpowiedniki PHP:

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

> {tip} Podczas zapętlania możesz użyć [zmienna pętli](#the-loop-variable), aby uzyskać cenne informacje o pętli, na przykład, czy jesteś w pierwszej czy ostatniej iteracji przez pętlę.

Podczas używania pętli możesz również zakończyć pętlę lub pominąć bieżącą iterację:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

Możesz również umieścić warunek z deklaracją dyrektywy w jednym wierszu:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### The Loop Variable - Zmienna pętli

Podczas zapętlenia w pętli będzie dostępna zmienna `$loop`. Ta zmienna zapewnia dostęp do użytecznych bitów informacji, takich jak bieżący indeks pętli i czy jest to pierwsza lub ostatnia iteracja w pętli:

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

Jeśli jesteś w pętli zagnieżdżonej, możesz uzyskać dostęp do zmiennej `$loop` nadrzędnej pętli poprzez właściwość `parent`:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

Zmienna `$loop` zawiera również wiele innych użytecznych właściwości:

Property  | Description
------------- | -------------
`$loop->index`  |  Indeks bieżącej iteracji pętli (rozpoczyna się od 0).
`$loop->iteration`  |  Aktualna iteracja pętli (zaczyna się od 1).
`$loop->remaining`  |  Iteracja pozostająca w pętli.
`$loop->count`  |  Całkowita liczba elementów w tablicy, która jest iterowana.
`$loop->first`  |  Czy jest to pierwsza iteracja w pętli.
`$loop->last`  |  Czy jest to ostatnia iteracja w pętli.
`$loop->depth`  |  Poziom zagnieżdżenia bieżącej pętli.
`$loop->parent`  |  Kiedy znajduje się w pętli zagnieżdżonej, zmienna pętli rodzica.

<a name="comments"></a>
### Comments - Komentarze

Blade pozwala również na definiowanie komentarzy w swoich widokach. Jednak w przeciwieństwie do komentarzy HTML komentarze Blade nie są zawarte w kodzie HTML zwróconym przez aplikację:

    {{-- This comment will not be present in the rendered HTML --}}

<a name="php"></a>
### PHP

W niektórych sytuacjach warto umieścić kod PHP w swoich widokach. Możesz użyć dyrektywy Blade `@php` do wykonania bloku zwykłego PHP w twoim szablonie:

    @php
        //
    @endphp

> {tip} Podczas gdy funkcja Blade zapewnia tę funkcję, często jej używanie może być sygnałem, że masz za dużo logiki osadzonej w szablonie.

<a name="including-sub-views"></a>
## Including Sub-Views - Dodawanie pod widoków

Dyrektywa `@include` firmy Blade umożliwia dołączenie widoku Blade z innego widoku. Wszystkie zmienne, które są dostępne dla widoku nadrzędnego, zostaną udostępnione do uwzględnionego widoku:

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

Nawet jeśli widok uwzględniony odziedziczy wszystkie dane dostępne w widoku nadrzędnym, możesz również przekazać szereg dodatkowych danych do dołączonego widoku:

    @include('view.name', ['some' => 'data'])

Oczywiście, jeśli spróbujesz `@include` widok, który nie istnieje, Laravel rzuci błąd. Jeśli chcesz uwzględnić widok, który może być obecny lub nie, powinieneś użyć dyrektywy `@includeIf`:

    @includeIf('view.name', ['some' => 'data'])

Jeśli chcesz `@include` widok w zależności od danego warunku boolowskiego, możesz użyć dyrektywy `@includeWhen`:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

Aby dołączyć pierwszy widok istniejący z danej tablicy widoków, możesz użyć dyrektywy `includeFirst`:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note} Powinieneś unikać używania stałych `__DIR__` i `__FILE__` w widokach Blade, ponieważ będą one odnosić się do położenia buforowanego, skompilowanego widoku.

<a name="rendering-views-for-collections"></a>
### Rendering Views For Collections - Rendering widoków dla kolekcji

Możesz łączyć pętle i fragmenty w jedną linię z dyrektywą `@each` Blade:

    @each('view.name', $jobs, 'job')

Pierwszy argument to widok częściowy do renderowania dla każdego elementu w tablicy lub kolekcji. Drugi argument to tablica lub kolekcja, którą chcesz powtórzyć, a trzeci argument to nazwa zmiennej, która zostanie przypisana do bieżącej iteracji w widoku. Na przykład, jeśli przeprowadzasz iterację w tablicy `jobs`, zazwyczaj będziesz chciał uzyskać dostęp do każdego zadania jako  `job` w części widoku. Klucz do bieżącej iteracji będzie dostępny jako zmienna `key` w twoim widoku częściowym.

Możesz także przekazać czwarty argument do dyrektywy `@each`. Ten argument określa widok, który zostanie wyświetlony, jeśli dana tablica jest pusta.

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} Widoki wyrenderowane przez `@each` nie dziedziczą zmiennych z widoku macierzystego. Jeśli widok podrzędny wymaga tych zmiennych, należy zamiast tego użyć `@foreach` i `@include`.

<a name="stacks"></a>
## Stacks - Półki na książki

Blade pozwala ci popychać do nazwanych stosów, które mogą być renderowane gdzie indziej w innym widoku lub układzie. Może to być szczególnie przydatne do określania dowolnych bibliotek JavaScript wymaganych przez widoki podrzędne:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

Możesz pchać do stosu tyle razy, ile potrzeba. Aby wyrenderować całą zawartość stosu, należy przekazać nazwę stosu do dyrektywy `@stack`:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## Service Injection - Wstrzyknięcie usługi

Dyrektywa `@inject` może być wykorzystana do pobrania usługi z Laravel [kontener usługi](/docs/{{version}}/container). Pierwszy argument przekazany do `@inject` jest nazwą zmiennej, do której zostanie dodana usługa, podczas gdy drugi argument jest nazwą klasy lub interfejsu usługi, którą chcesz rozwiązać:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Extending Blade - Rozszerzenie Blade

Blade umożliwia zdefiniowanie własnych dyrektyw użytkownika za pomocą metody `directive`. Gdy kompilator Blade napotka dyrektywę niestandardową, wywoła to wywołanie zwrotne z wyrażeniem zawartym w dyrektywie.

Poniższy przykład tworzy dyrektywę `@datetime($var)`, która formatuje dany `$var`, który powinien być instancją` DateTime`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Jak widać, będziemy łączyć metodę `format` z każdym wyrażeniem przekazanym do dyrektywy. W tym przykładzie ostateczny PHP wygenerowany przez tę dyrektywę będzie:

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} Po zaktualizowaniu logiki dyrektywy Blade będziesz musiał usunąć wszystkie buforowane widoki Blade. Zbuforowane widoki Blade mogą być usunięte za pomocą polecenia `view:clear` Artisan.

<a name="custom-if-statements"></a>
### Custom If Statements - Deklaracja niestandardowe jeżeli

Programowanie niestandardowej dyrektywy jest czasami bardziej złożone niż to konieczne podczas definiowania prostych, niestandardowych instrukcji warunkowych. Z tego powodu Blade udostępnia metodę `Blade::if`, która pozwala szybko zdefiniować niestandardowe dyrektywy warunkowe za pomocą Closures. Na przykład, zdefiniujmy niestandardowy warunek, który sprawdza bieżące środowisko aplikacji. Możemy to zrobić w metodzie  `boot`  naszego `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

Po zdefiniowaniu warunku niestandardowego możemy z łatwością użyć go w naszych szablonach:

    @env('local')
        // The application is in the local environment...
    @elseenv('testing')
        // The application is in the testing environment...
    @else
        // The application is not in the local or testing environment...
    @endenv
