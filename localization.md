# Localization

- [Introduction - Wprowadzenie](#introduction)
- [Defining Translation Strings - Definiowanie łańcuchów translacji](#defining-translation-strings)
    - [Using Short Keys - Używanie krótkich klawiszy](#using-short-keys)
    - [Using Translation Strings As Keys - Używanie łańcuchów translacji jako kluczy](#using-translation-strings-as-keys)
- [Retrieving Translation Strings - Pobieranie ciągów translacji](#retrieving-translation-strings)
    - [Replacing Parameters In Translation Strings - Zastępowanie parametrów w łańcuchach translacji](#replacing-parameters-in-translation-strings)
    - [Pluralization - Liczba mnoga](#pluralization)
- [Overriding Package Language Files - Nadpisywanie plików językowych pakietów](#overriding-package-language-files)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Funkcje lokalizacji Laravel zapewniają wygodny sposób pobierania ciągów znaków w różnych językach, co pozwala w łatwy sposób obsługiwać wiele języków w obrębie aplikacji. Łańcuchy językowe są przechowywane w plikach w katalogu `resources/lang`. W tym katalogu powinien znajdować się podkatalog dla każdego języka obsługiwanego przez aplikację:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Wszystkie pliki językowe po prostu zwracają tablicę ciągów z kluczami. Na przykład:

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

### Configuring The Locale - Konfigurowanie ustawień regionalnych

Domyślny język aplikacji jest przechowywany w pliku konfiguracyjnym `config/app.php`. Oczywiście możesz modyfikować tę wartość, aby dostosować ją do potrzeb aplikacji. Możesz również zmienić aktywny język w środowisku wykonawczym, używając metody `setLocale` na elewacji `App`:

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

Możesz skonfigurować "język zapasowy", który będzie używany, gdy aktywny język nie zawiera danego ciągu tłumaczenia. Podobnie jak język domyślny, język zapasowy jest również skonfigurowany w pliku konfiguracyjnym `config/app.php`:

    'fallback_locale' => 'en',

#### Determining The Current Locale - Określanie bieżących ustawień regionalnych

Możesz użyć metod `getLocale` i `isLocale` na elewacji `App`, aby określić bieżące ustawienia regionalne lub sprawdzić, czy ustawienia narodowe mają określoną wartość:

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## Defining Translation Strings - Definiowanie łańcuchów translacji

<a name="using-short-keys"></a>
### Using Short Keys - Używanie krótkich klawiszy

Zazwyczaj ciągi tłumaczenia są przechowywane w plikach w katalogu `resources/lang`. W tym katalogu powinien znajdować się podkatalog dla każdego języka obsługiwanego przez aplikację:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Wszystkie pliki językowe po prostu zwracają tablicę ciągów z kluczami. Na przykład:

    <?php

    // resources/lang/en/messages.php

    return [
        'welcome' => 'Welcome to our application'
    ];

<a name="using-translation-strings-as-keys"></a>
### Using Translation Strings As Keys - Używanie łańcuchów translacji jako kluczy

W przypadku aplikacji o dużych wymaganiach dotyczących tłumaczenia zdefiniowanie każdego ciągu za pomocą "krótkiego klucza" może szybko stać się mylące, gdy odwołuje się do nich w swoich widokach. Z tego powodu Laravel zapewnia również obsługę definiowania ciągów translacji przy użyciu "domyślnego" tłumaczenia ciągu znaków jako klucza.

Pliki tłumaczeń używające łańcuchów translacji jako kluczy są przechowywane jako pliki JSON w katalogu `resources/lang`. Na przykład, jeśli twoja aplikacja ma tłumaczenie na hiszpański, powinieneś utworzyć plik `resources/lang/es.json`:

    {
        "I love programming.": "Me encanta programar."
    }

<a name="retrieving-translation-strings"></a>
## Retrieving Translation Strings - Pobieranie ciągów translacji

Możesz pobierać linie z plików językowych za pomocą funkcji pomocniczej `__`. Metoda `__` akceptuje plik i klucz łańcucha tłumaczenia jako swój pierwszy argument. Na przykład pobierzmy łańcuch tłumaczenia `welcome` z pliku językowego `resources/lang/messages.php`:

    echo __('messages.welcome');

    echo __('I love programming.');

Oczywiście, jeśli używasz [silnika szablonów Blade](/docs/{{version}}/blade), możesz użyć składni `{{ }}` do wyświetlenia ciągu tłumaczenia lub użyć dyrektywy `@lang`:

    {{ __('messages.welcome') }}

    @lang('messages.welcome')

Jeśli podany łańcuch translacji nie istnieje, funkcja `__` po prostu zwróci klucz łańcucha tłumaczenia. Tak więc, używając powyższego przykładu, funkcja `__` zwróci komunikat `messages.welcome`, jeśli ciąg tłumaczenia nie istnieje.

<a name="replacing-parameters-in-translation-strings"></a>
### Replacing Parameters In Translation Strings - Zastępowanie parametrów w łańcuchach translacji

Jeśli chcesz, możesz zdefiniować zmienne w swoich ciągach translacji. Wszyscy posiadacze miejsc mają przedrostek  `:`. Na przykład możesz zdefiniować wiadomość powitalną z nazwą właściciela miejsca:

    'welcome' => 'Welcome, :name',

Aby zamienić elementy zastępcze podczas pobierania ciągu translacji, należy przekazać tablicę zastępczą jako drugi argument funkcji `__`:

    echo __('messages.welcome', ['name' => 'dayle']);

Jeśli Twój posiadacz-miejsce zawiera wszystkie wielkie litery lub ma tylko pierwszą literę wielką, przetłumaczona wartość zostanie odpowiednio zamieniona na:

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

<a name="pluralization"></a>
### Pluralization - Liczba mnoga

Pluralizacja jest złożonym problemem, ponieważ różne języki mają wiele złożonych reguł dla pluralizacji. Używając znaku "potoku" możesz rozróżniać formy w liczbie pojedynczej i mnogiej ciągu:

    'apples' => 'There is one apple|There are many apples',

Możesz nawet utworzyć bardziej złożone reguły plurowania, które określają łańcuchy translacji dla wielu zakresów liczbowych:

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

Po zdefiniowaniu ciągu translacji, który ma opcje liczby mnogiej, możesz użyć funkcji `trans_choice` do pobrania linii dla danego "licznika". W tym przykładzie, ponieważ liczba jest większa niż jeden, zwracana jest liczba mnoga ciągu tłumaczenia:

    echo trans_choice('messages.apples', 10);

<a name="overriding-package-language-files"></a>
## Overriding Package Language Files - Nadpisywanie plików językowych pakietów

Niektóre pakiety mogą być dostarczane z własnymi plikami językowymi. Zamiast zmieniać podstawowe pliki pakietu, aby poprawić te linie, możesz je zastąpić, umieszczając pliki w katalogu `resources/lang/vendor/{package}/{locale}`.

Tak więc, na przykład, jeśli chcesz przesłonić angielskie ciągi tłumaczenia w `messages.php` dla pakietu o nazwie `skyrim/hearthfire`, powinieneś umieścić plik językowy w: `resources/lang/vendor/hearthfire/en/messages.php`. W tym pliku należy tylko zdefiniować łańcuchy translacji, które mają zostać zastąpione. Wszystkie ciągi tłumaczenia, których nie zastąpisz, będą nadal ładowane z oryginalnych plików językowych pakietu.
