# Helpers

- [Introduction - Wprowadzenie](#introduction)
- [Available Methods - Dostępne metody](#available-methods)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel zawiera wiele globalnych "pomocniczych" funkcji PHP. Wiele z tych funkcji jest wykorzystywanych przez samą strukturę; jednak możesz używać je we własnych aplikacjach, jeśli uznasz je za wygodne.

<a name="available-methods"></a>
## Available Methods - Dostępne metody

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### Arrays & Objects - Tablice i obiekty

<div class="collection-method-list" markdown="1">

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_random](#method-array-random)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[head](#method-head)
[last](#method-last)
</div>

### Paths - Ścieżki

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### Strings - Ciągi znaków

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[preg_replace_array](#method-preg-replace-array)
[snake_case](#method-snake-case)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_limit](#method-str-limit)
[Str::orderedUuid](#method-str-ordered-uuid)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_replace_array](#method-str-replace-array)
[str_replace_first](#method-str-replace-first)
[str_replace_last](#method-str-replace-last)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[str_start](#method-str-start)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)
[Str::uuid](#method-str-uuid)

</div>

### URLs - Adresy URL

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

### Miscellaneous - Różne

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[today](#method-today)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)

</div>

<a name="method-listing"></a>
## Method Listing - Lista metod

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## Arrays & Objects - Tablice i obiekty

<a name="method-array-add"></a>
#### `array_add()` - dodaj do tablicy {#collection-method .first-collection-method}

Funkcja `array_add` dodaje daną parę klucz / wartość do tablicy, jeśli dany klucz nie istnieje w tablicy:

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` - zwiń tablicę tablic {#collection-method}

Funkcja `array_collapse` zwija tablicę tablic w jedną tablicę:

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` - podziel tablice k/w {#collection-method}

Funkcja `array_divide` zwraca dwie tablice, jedną zawierającą klucze, a drugą zawierającą wartości danej tablicy:

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()`- kropkuj tablicę zagnieżdzoną {#collection-method}

Funkcja `array_dot` spłaszcza wielowymiarową tablicę w jednej tablicy poziomów, która używa notacji "kropki" do wskazania głębokości:

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = array_dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `array_except()` - usuń element k/w {#collection-method}

Funkcja `array_except` usuwa podane pary klucz / wartość z tablicy:

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` - pierwszy z tablicy {#collection-method}

Funkcja `array_first` zwraca pierwszy element tablicy przechodzącej dany test prawdy:

    $array = [100, 200, 300];

    $first = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Wartość domyślna może być również przekazana jako trzeci parametr metody. Ta wartość zostanie zwrócona, jeśli żadna wartość nie przejdzie testu prawdy:

    $first = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` - spłaszcz tablicę do 1 wymiaru {#collection-method}

Funkcja `array_flatten` spłaszcza wielowymiarową tablicę w jednej tablicy poziomów:

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `array_forget()` - usuń z zagnieżdzonej tablicy po kropce {#collection-method}

Funkcja `array_forget` usuwa daną parę klucz / wartość z głęboko zagnieżdżonej tablicy używając notacji "kropka":

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` - weż z zagnieżdzonej tablicy po kropce  {#collection-method}

Funkcja `array_get` pobiera wartość z głęboko zagnieżdżonej tablicy używając notacji "kropka":

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = array_get($array, 'products.desk.price');

    // 100

Funkcja `array_get` akceptuje również wartość domyślną, która zostanie zwrócona, jeśli określony klucz nie zostanie znaleziony:

    $discount = array_get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `array_has()` - sprawdz w zagnieżdzonej tablicy po kropce {#collection-method}

Funkcja `array_has` sprawdza, czy dana pozycja lub elementy istnieją w tablicy za pomocą notacji "kropka":

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = array_has($array, 'product.name');

    // true

    $contains = array_has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `array_last()` - ostatni element tablicy {#collection-method}

Funkcja `array_last` zwraca ostatni element tablicy przechodzącej dany test prawdy:

    $array = [100, 200, 300, 110];

    $last = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

Wartość domyślna może zostać przekazana jako trzeci argument metody. Ta wartość zostanie zwrócona, jeśli żadna wartość nie przejdzie testu prawdy:

    $last = array_last($array, $callback, $default);

<a name="method-array-only"></a>
#### `array_only()` - weż tylko k/w {#collection-method}

Funkcja `array_only` - zwraca tylko określone pary klucz / wartość z podanej tablicy:

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` - weź wszystkie wartości po kluczu {#collection-method}

Funkcja `array_pluck` pobiera wszystkie wartości dla danego klucza z tablicy:

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

Możesz również określić, jak chcesz, aby powstała lista była kluczowana:

    $names = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `array_prepend()` - dodaj na poczatku {#collection-method}

Funkcja `array_prepend` dodaj element na początek tablicy:

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

W razie potrzeby możesz określić klucz, który powinien zostać użyty dla wartości:

    $array = ['price' => 100];

    $array = array_prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>
#### `array_pull()`- weż i usuń po k/w {#collection-method}

Funkcja `array_pull` zwraca i usuwa parę klucz / wartość z tablicy:


    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Wartość domyślna może zostać przekazana jako trzeci argument metody. Ta wartość zostanie zwrócona, jeśli klucz nie istnieje:

    $value = array_pull($array, $key, $default);

<a name="method-array-random"></a>
#### `array_random()` - losowy element {#collection-method}

Funkcja `array_random` zwraca losową wartość z tablicy:

    $array = [1, 2, 3, 4, 5];

    $random = array_random($array);

    // 4 - (retrieved randomly)

Możesz również określić liczbę elementów do zwrócenia jako opcjonalnego drugiego argumentu. Zwróć uwagę, że dostarczenie tego argumentu zwróci tablicę, nawet jeśli pożądany jest tylko jeden element:

    $items = array_random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

Funkcja `array_set` ustawia wartość w głęboko zagnieżdżonej tablicy używając notacji "kropka":

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` - sortuj tablicę {#collection-method}

Funkcja `array_sort` sortuje tablicę według jej wartości:

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = array_sort($array);

    // ['Chair', 'Desk', 'Table']

Możesz również posortować tablicę według wyników danego Closure:

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` - sortowanie rekursywne {#collection-method}

Funkcja `array_sort_recursive` rekursywnie sortuje tablicę za pomocą funkcji `sort`:

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
    ];

    $sorted = array_sort_recursive($array);

    /*
        [
            ['Li', 'Roman', 'Taylor'],
            ['JavaScript', 'PHP', 'Ruby'],
        ]
    */

<a name="method-array-where"></a>
#### `array_where()` - filtruj gdzie {#collection-method}

Funkcja `array_where` filtruje tablicę przy użyciu zadanej Closure:

    $array = [100, '200', 300, '400', 500];

    $filtered = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-array-wrap"></a>
#### `array_wrap()` - opakuj w tablicę {#collection-method}

Funkcja `array_wrap` zawija podaną wartość w tablicy. Jeśli podana wartość jest już tablicą, nie zostanie zmieniona:

    $string = 'Laravel';

    $array = array_wrap($string);

    // ['Laravel']

Jeśli podana wartość jest pusta, zwrócona zostanie pusta tablica:

    $nothing = null;

    $array = array_wrap($nothing);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` - ustawia brakującą wartość {#collection-method}

Funkcja `data_fill` ustawia brakującą wartość w zagnieżdżonej tablicy lub obiekcie za pomocą notacji "kropka":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Ta funkcja akceptuje również gwiazdki jako symbole wieloznaczne i odpowiednio wypełnia cel:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()` - pobież wartość po kropce {#collection-method}

Funkcja `data_get` pobiera wartość z zagnieżdżonej tablicy lub obiektu za pomocą notacji "kropka":

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

Funkcja `data_get` akceptuje również wartość domyślną, która zostanie zwrócona, jeśli określony klucz nie zostanie znaleziony:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

<a name="method-data-set"></a>
#### `data_set()` -  ustaw wartość po kropce {#collection-method}

Funkcja `data_set` ustawia wartość wewnątrz zagnieżdżonej tablicy lub obiektu za pomocą notacji "kropka":

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Ta funkcja przyjmuje również symbole wieloznaczne i odpowiednio ustawia wartości na celu:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

Domyślnie wszelkie istniejące wartości są zastępowane. Jeśli chcesz ustawić tylko wartość, jeśli nie istnieje, możesz podać `false` jako trzeci argument:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-head"></a>
#### `head()`- pierwszy element tablicy {#collection-method}

Funkcja `head` zwraca pierwszy element w podanej tablicy:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` - ostatni element tablicy {#collection-method}

Funkcja `last` zwraca ostatni element w podanej tablicy:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## Paths - Ścieżki

<a name="method-app-path"></a>
#### `app_path()` - ścieżka do aplikacji {#collection-method}

Funkcja `app_path` zwraca pełną ścieżkę do katalogu `app`. Możesz także użyć funkcji `app_path` do wygenerowania w pełni kwalifikowanej ścieżki do pliku względem katalogu aplikacji:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` - ścieżka do projektu {#collection-method}

Funkcja `base_path` zwraca pełną ścieżkę do katalogu głównego projektu. Możesz także użyć funkcji `base_path` do wygenerowania pełnej ścieżki do danego pliku względem katalogu głównego projektu:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` - ścieżka do konfiuracji {#collection-method}

Funkcja `config_path` zwraca pełną ścieżkę do katalogu `config`. Możesz także użyć funkcji `config_path` do wygenerowania pełnej ścieżki do podanego pliku w katalogu konfiguracyjnym aplikacji:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` - ścieżka do bazydanych {#collection-method}

Funkcja `database_path` zwraca pełną ścieżkę do katalogu `database`. Możesz także użyć funkcji `database_path` do wygenerowania pełnej ścieżki do podanego pliku w katalogu bazy danych:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>
#### `mix()` - ścieżka do Mix {#collection-method}

Funkcja `mix` zwraca ścieżkę do [wersjonowanego pliku Mix](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` - ścieżka do public {#collection-method}

Funkcja `public_path` zwraca pełną ścieżkę do katalogu `public`. Możesz także użyć funkcji `public_path` do wygenerowania pełnej ścieżki do danego pliku w katalogu publicznym:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` - ścieżka do zasobów {#collection-method}

Funkcja `resource_path` zwraca pełną ścieżkę do katalogu `resources`. Możesz również użyć funkcji `resource_path` do wygenerowania pełnej ścieżki do podanego pliku w katalogu zasobów:

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` - ścieżka do przechowalni {#collection-method}

Funkcja `storage_path` zwraca pełną ścieżkę do katalogu `storage`. Możesz także użyć funkcji `storage_path` do wygenerowania pełnej ścieżki do podanego pliku w katalogu przechowalni:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Strings - Ciągi znaków

<a name="method-__"></a>
#### `__()` - tłumaczy text {#collection-method}

Funkcja `__` tłumaczy dany łańcuch translacji lub klucz tłumaczenia za pomocą twoich [plików lokalizacyjnych](/docs/{{version}}/localization):

    echo __('Welcome to our application');

    echo __('messages.welcome');

Jeśli podany łańcuch lub klucz translacji nie istnieje, funkcja `__` zwróci podaną wartość. Tak więc, używając powyższego przykładu, funkcja `__` zwróci komunikat `messages.welcome`, jeśli klucz ten nie istnieje.

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

Funkcja `camel_case` konwertuje podany ciąg na` camelCase`:

    $converted = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

Pomocnik `class_basename` zwraca nazwę klasy danej klasy z usuniętą przestrzenią nazw klasy:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` - htmlspecialchars {#collection-method}

Funkcja `e` uruchamia funkcję PHP `htmlspecialchars` z opcją `double_encode` ustawioną na `false`:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` - koniec z {#collection-method}

Funkcja `ends_with` określa, czy dany łańcuch kończy się podaną wartością:

    $result = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

Funkcja `kebab_case` konwertuje podany ciąg na `kebab-case`:

    $converted = kebab_case('fooBar');

    // foo-bar

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` - podmienia dane po wzorcu {#collection-method}

Funkcja `preg_replace_array` zastępuje dany wzorzec w łańcuchu sekwencyjnie za pomocą tablicy:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

Funkcja `snake_case` konwertuje dany ciąg na `snake_case`:

    $converted = snake_case('fooBar');

    // foo_bar

<a name="method-starts-with"></a>
#### `starts_with()` - początek z {#collection-method}

Funkcja `starts_with` określa, czy dany ciąg znaków zaczyna się od podanej wartości:

    $result = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` - po ciągu znaków {#collection-method}

Funkcja `str_after` zwraca wszystko po podanej wartości w ciągu znaków:

    $slice = str_after('This is my name', 'This is');

    // ' my name'

<a name="method-str-before"></a>
#### `str_before()` - przed ciągiem znaków {#collection-method}

Funkcja `str_before` zwraca wszystko przed podaną wartością w ciągu znaków:

    $slice = str_before('This is my name', 'my name');

    // 'This is '

<a name="method-str-contains"></a>
#### `str_contains()` - pasujący ciąg {#collection-method}

The `str_contains` function determines if the given string contains the given value:
Funkcja `str_contains` określa, czy podany łańcuch zawiera daną wartość:

    $contains = str_contains('This is my name', 'my');

    // true

Możesz również przekazać tablicę wartości, aby określić, czy dany ciąg zawiera jakąkolwiek z wartości:

    $contains = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` - zakańcza ciąg {#collection-method}

Funkcja `str_finish` dodaje pojedynczą instancję podanej wartości do łańcucha, jeśli nie kończy się ona na wartości:

    $adjusted = str_finish('this/string', '/');

    // this/string/

    $adjusted = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` - czy pasuje do wzorca? {#collection-method}

Funkcja `str_is` określa, czy dany ciąg pasuje do danego wzorca. Do oznaczenia symboli wieloznacznych można używać gwiazdek:

    $matches = str_is('foo*', 'foobar');

    // true

    $matches = str_is('baz*', 'foobar');

    // false

<a name="method-str-limit"></a>
#### `str_limit()`- limit ciągu {#collection-method}

Funkcja `str_limit` obcina podany łańcuch o określonej długości:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

Możesz również przekazać trzeci argument, aby zmienić ciąg, który zostanie dodany do końca:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {#collection-method}

Metoda `Str::orderedUuid` generuje najpierw identyfikator UUID "datownika", który może być skutecznie przechowywany w kolumnie indeksowanej bazy danych:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-plural"></a>
#### `str_plural()` - liczba mnoga {#collection-method}

Funkcja `str_plural` przekształca ciąg w jego liczbę mnogą. Ta funkcja obsługuje obecnie tylko język angielski:

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

Możesz podać liczbę całkowitą jako drugi argument funkcji, aby pobrać pojedynczą lub mnogą postać ciągu:

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` - losowy ciąg {#collection-method}

Funkcja `str_random` generuje losowy ciąg o określonej długości. Ta funkcja używa funkcji `random_bytes` w PHP:

    $random = str_random(40);

<a name="method-str-replace-array"></a>
#### `str_replace_array()` - zastępuje po ciągu {#collection-method}

Funkcja `str_replace_array` zastępuje daną wartość w ciągu sekwencyjnie za pomocą tablicy:

    $string = 'The event will take place between ? and ?';

    $replaced = str_replace_array('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `str_replace_first()` - zastąp 1 wystąpienie {#collection-method}

Funkcja `str_replace_first` zastępuje pierwsze wystąpienie danej wartości w ciągu znaków:

    $replaced = str_replace_first('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `str_replace_last()` - zastąp ostatnie wystąpienie {#collection-method}

Funkcja `str_replace_last` zastępuje ostatnie wystąpienie danej wartości w ciągu znaków:

    $replaced = str_replace_last('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `str_singular()` - liczba pojedyńcza {#collection-method}

Funkcja `str_singular` przekształca ciąg w jego pojedynczą formę. Ta funkcja obsługuje obecnie tylko język angielski:

    $singular = str_singular('cars');

    // car

    $singular = str_singular('children');

    // child

<a name="method-str-slug"></a>
#### `str_slug()` - ciąg ślimak {#collection-method}

Funkcja `str_slug` generuje przyjazny dla adresu URL "ślimak" z podanego ciągu:

    $slug = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-str-start"></a>
#### `str_start()` - początkowy ciąg {#collection-method}

Funkcja `str_start` dodaje pojedynczą instancję o podanej wartości do napisu, jeśli jeszcze nie zaczyna się od wartości:

    $adjusted = str_start('this/string', '/');

    // /this/string

    $adjusted = str_start('/this/string/', '/');

    // /this/string

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

Funkcja `studly_case` konwertuje podany ciąg na` StudlyCase`:

    $converted = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

Funkcja `title_case` konwertuje podany ciąg na `Title Case`:

    $converted = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` - tłumaczy {#collection-method}

Funkcja `trans` tłumaczy dany klucz tłumaczenia za pomocą twoich [plików lokalizacyjnych](/docs/{{version}}/localization):

    echo trans('messages.welcome');

Jeśli podany klucz translacji nie istnieje, funkcja `trans` zwróci dany klucz. Tak więc, używając powyższego przykładu, funkcja `trans` zwróci komunikat `messages.welcome`, jeśli klucz translacyjny nie istnieje.

<a name="method-trans-choice"></a>
#### `trans_choice() - tłumaczy na fleksyjny` {#collection-method}

Funkcja `trans_choice` tłumaczy dany klucz tłumaczenia na fleksyjny:

    echo trans_choice('messages.notifications', $unreadCount);

Jeśli podany klucz translacji nie istnieje, funkcja `trans_choice` zwróci dany klucz. Tak więc, używając powyższego przykładu, funkcja `trans_choice` zwróci komunikat `messages.notifications`, jeśli klucz translacyjny nie istnieje.

<a name="method-str-uuid"></a>
#### `Str::uuid() - identyfikator UUID` {#collection-method}

Metoda `Str::uuid` generuje identyfikator UUID (wersja 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="urls"></a>
## URLs - Adresy Url

<a name="method-action"></a>
#### `action()` - akcje kontrolera {#collection-method}

Funkcja `action` generuje adres URL dla podanej akcji kontrolera. Nie musisz przekazywać pełnego obszaru nazw kontrolera. Zamiast tego podaj nazwę klasy kontrolera względem obszaru nazw `App\Http\Controllers`:

    $url = action('HomeController@index');

Jeśli metoda akceptuje parametry trasy, możesz przekazać je jako drugi argument metody:

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` - adres do zasobów {#collection-method}

Funkcja `asset` generuje adres URL dla zasobu przy użyciu bieżącego schematu żądania (HTTP lub HTTPS):

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` - adres do zasobów HTTPS {#collection-method}

Funkcja `secure_asset` generuje URL dla zasobu przy użyciu HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-route"></a>
#### `route()` - adres do tras {#collection-method}

Funkcja `route` generuje URL dla podanej nazwanej trasy:

    $url = route('routeName');

Jeśli trasa przyjmuje parametry, możesz przekazać je jako drugi argument metody:

    $url = route('routeName', ['id' => 1]);

Domyślnie funkcja `route` generuje bezwzględny adres URL. Jeśli chcesz wygenerować względny adres URL, możesz podać `false` jako trzeci argument:

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-url"></a>
#### `secure_url()` - adres do podanej ścieżki HTTPS {#collection-method}

Funkcja `secure_url` generuje w pełni kwalifikowany URL HTTPS do podanej ścieżki:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` - adres do podanej ścieżki {#collection-method}

Funkcja `url` generuje pełny adres URL do podanej ścieżki:

    $url = url('user/profile');

    $url = url('user/profile', [1]);

Jeśli nie podano ścieżki, zwracana jest instancja `Illuminate\Routing\UrlGenerator`:

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## Miscellaneous - Różne

<a name="method-abort"></a>
#### `abort()` - zgłoś wyjątek {#collection-method}

Funkcja `abort` zgłasza [wyjątek HTTP](/docs/{{version}}/errors#http-exceptions), które będą renderowane przez [wyjątek handler](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

Możesz również podać tekst odpowiedzi wyjątku i niestandardowe nagłówki odpowiedzi:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` - zgłoś wyjatej jeżeli {#collection-method}

Funkcja `abort_if` zgłasza wyjątek HTTP, jeśli podane wyrażenie boolowskie ma wartość `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Podobnie jak w przypadku metody `abort`, możesz również podać tekst odpowiedzi wyjątku jako trzeci argument i tablicę niestandardowych nagłówków odpowiedzi jako czwarty argument.

<a name="method-abort-unless"></a>
#### `abort_unless()` - zgłoś wyjątek chyba że {#collection-method}

Funkcja `abort_unless` zgłasza wyjątek HTTP, jeśli podane wyrażenie boolowskie ma wartość `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Podobnie jak w przypadku metody `abort`, możesz również podać tekst odpowiedzi wyjątku jako trzeci argument i tablicę niestandardowych nagłówków odpowiedzi jako czwarty argument.

<a name="method-app"></a>
#### `app()` - instancja aplikacji {#collection-method}

Funkcja `app` zwraca instancje [kontener usługi](/docs/{{version}}/container):

    $container = app();

Możesz przekazać nazwę klasy lub interfejsu, aby otrzymać objekt z kontenera:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` - uwierzytelniacz {#collection-method}

Funkcja `auth` zwraca instancję [uwierzytelniacza](/docs/{{version}}/authentication). Możesz użyć go zamiast fasady `Auth` dla wygody:

    $user = auth()->user();

W razie potrzeby możesz określić, do której instancji wartownika chcesz uzyskać dostęp:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` - cofnij url {#collection-method}

Funkcja `back` generuje [przekierowanie odpowiedzi HTTP](/docs/{{version}}/responses#redirects) do poprzedniej lokalizacji użytkownika:

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` - haszuje {#collection-method}

Funkcja `bcrypt` [haszuje](/docs/{{version}}/hashing) podana wartość za pomocą Bcrypt. Możesz użyć go jako alternatywy dla fasady `Hash`:

    $password = bcrypt('my-secret-password');

<a name="method-broadcast"></a>
#### `broadcast()` - rozgłasza {#collection-method}

Funkcja `broadcast` [rozgłasza](/docs/{{version}}/broadcasting) podanego [zdarzenia](/docs/{{version}}/events) do swoich odbiorców:

    broadcast(new UserRegistered($user));

<a name="method-blank"></a>
#### `blank()` - czy pusta? {#collection-method}

Funkcja `blank` zwraca informację, czy podana wartość jest "pusta":

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

Odwrotność `blank`, jest metoda [`filled`](#method-filled).

<a name="method-cache"></a>
#### `cache()` - wartość z pamięci podręcznej {#collection-method}

Funkcja `cache` może służyć do pobierania wartości z [cache](/docs/{{version}}/cache). Jeśli podany klucz nie istnieje w pamięci podręcznej, zwracana jest opcjonalna wartość domyślna:

    $value = cache('key');

    $value = cache('key', 'default');

Możesz dodawać elementy do pamięci podręcznej, przekazując do pary funkcję par klucz / wartość. Powinieneś także podać liczbę minut lub czas trwania wartości buforowanej, która powinna być uznana za ważną:

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {#collection-method}

Funkcja `class_uses_recursive` zwraca wszystkie cechy używane przez klasę, w tym cechy używane przez dowolne podklasy:

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` - tworzy kolekcję {#collection-method}

Funkcja `collect` tworzy instancję [kolekcje](/docs/{{version}}/collections) o podanej wartości:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` - konfiguracja {#collection-method}

Funkcja `config` pobiera wartość zmiennej [konfiguracji](/docs/{{version}}/configuration). Dostęp do wartości konfiguracyjnych można uzyskać za pomocą składni "kropka", która zawiera nazwę pliku i opcję, do której chcesz uzyskać dostęp. Wartość domyślna może zostać określona i jest zwracana, jeśli opcja konfiguracji nie istnieje:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Możesz ustawić zmienne konfiguracyjne w środowisku wykonawczym, przekazując tablicę par klucz / wartość:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` - ciasteczka {#collection-method}

Funkcja `cookie` tworzy nową instancję [ciasteczka](/docs/{{version}}/requests#cookies) :

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()` pole csrf {#collection-method}

Funkcja `csrf_field` generuje ukryte pole HTML `hidden` zawierające wartość tokenu CSRF. Na przykład, używając [składni Blade](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` - żeton csrf {#collection-method}

Funkcja `csrf_token` pobiera wartość bieżącego tokenu CSRF:

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` - rzuć zmienną i kończ {#collection-method}

Funkcja `dd` zrzuca podane zmienne i kończy działanie skryptu:

    dd($value);

    dd($value1, $value2, $value3, ...);

Jeśli nie chcesz zatrzymywać wykonywania skryptu, użyj zamiast tego funkcji [`dump`] (# method-dump).

<a name="method-decrypt"></a>
#### `decrypt()` - odszyfruj {#collection-method}

Funkcja `decrypt` odszyfrowuje daną wartość za pomocą [encryptera](/docs/{{version}}/encryption) Laravel:

    $decrypted = decrypt($encrypted_value);

<a name="method-dispatch"></a>
#### `dispatch()` - wyślij zadanie {#collection-method}

Funkcja `dispatch` przekazuje dane [zadanie](/docs/{{version}}/queues#creating-jobs) do Laravel poprzez [kolejkę zadań](/docs/{{version}}/queues):

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>
#### `dispatch_now()` - wyślij zadanie teraz {#collection-method}

Funkcja `dispatch_now` uruchamia podane [zadanie](/docs/{{version}}/queues#creating-jobs) i zwraca wartość z metody `handle`:

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` - zrzut zmiennej{#collection-method}

Funkcja `dump` zrzuca określone zmienne:

    dump($value);

    dump($value1, $value2, $value3, ...);

Jeśli chcesz przestać wykonywać skrypt po zrzuceniu zmiennych, użyj zamiast tego funkcji [`dd`](#method-dd).

<a name="method-encrypt"></a>
#### `encrypt()` - zaszyfruj {#collection-method}

Funkcja `encrypt` szyfruje daną wartość za pomocą [encryptera](/docs/{{version}}/encryption):

    $encrypted = encrypt($unencrypted_value);

<a name="method-env"></a>
#### `env()` {#collection-method}

Funkcja `env` pobiera wartość [zmiennej środowiskowej](/docs/{{version}}/configuration#environment-configuration) lub zwraca wartość domyślną:

    $env = env('APP_ENV');

    // Returns 'production' if APP_ENV is not set...
    $env = env('APP_ENV', 'production');

> {note} Jeśli wykonasz komendę `config:cache` podczas procesu wdrażania, powinieneś być pewien, że wywołujesz funkcję `env` z plików konfiguracyjnych. Gdy konfiguracja została zbuforowana, plik `.env` nie zostanie załadowany, a wszystkie wywołania funkcji `env` zwrucą `null`.


<a name="method-event"></a>
#### `event()` - wywołuje zdarzenie {#collection-method}

Funkcja `event` wywołuje podane [zadanie](/docs/{{version}}/events) do swoich detektorów:

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` - fabrykuje {#collection-method}

Funkcja `factory` tworzy obiekty za pomocą fabyki podając, klasę, nazwę i wartość. Może być używany podczas [testowania](/docs/{{version}}/database-testing#writing-factories) lub [seedowania](/docs/{{version}}/seeding#using-model-factories):

    $user = factory(App\User::class)->make();

<a name="method-filled"></a>
#### `filled()` - czy nie pusta? {#collection-method}

Funkcja `filled` zwraca informację, czy podana wartość nie jest "pusta":

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

Dla odwrotności `filled`, jest metoda [`blank`](#method-blank).

<a name="method-info"></a>
#### `info()` - zapisz do dziennika {#collection-method}

Funkcja `info` zapisze informacje w dzienniku [log](/docs/{{version}}/errors#logging):

    info('Some helpful information!');

Tablica danych kontekstowych może również zostać przekazana do funkcji:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` - zapisz do debugera {#collection-method}

Funkcja `logger` może być wykorzystana do napisania komunikatu `debug` do dziennika [log][log](/docs/{{version}}/errors#logging):

    logger('Debug message');

Tablica danych kontekstowych może również zostać przekazana do funkcji:

    logger('User has logged in.', ['id' => $user->id]);

Instancja [logger](/docs/{{version}}/errors#logging) zostanie zwrócona, jeśli do funkcji nie zostanie przekazana żadna wartość:

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` - akcja HTTP {#collection-method}

Funkcja `method_field` generuje ukryte pole HTML `hidden` zawierające sfałszowaną wartość akcji  HTTP formularza. Na przykład, używając [składni Blade](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()` - czas teraz{#collection-method}

Funkcja `now` tworzy nową instancję `Illuminate\Support\Carbon` dla bieżącego czasu:

    $now = now();

<a name="method-old"></a>
#### `old()` - stary input {#collection-method}

Funkcja `old` [pobiera](/docs/{{version}}/requests#retrieving-input) wartość [starą wartośc danej wejściowej](/docs/{{version}}/requests#old-input), wartość wyświetlana w sesji:

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>
#### `optional()` - opcjonalnie {#collection-method}

Funkcja `optional` akceptuje dowolny argument i umożliwia dostęp do właściwości lub metod wywoływania tego obiektu. Jeśli podany obiekt ma wartość `null`, właściwości i metody zwrócą `null` zamiast powodować błąd:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

<a name="method-policy"></a>
#### `policy()` - polityka {#collection-method}

The `policy` method retrieves a [policy](/docs/{{version}}/authorization#creating-policies) instance for a given class:
Metoda `policy` pobiera instancję [polityki](/docs/{{version}}/authorization#creating-policies) dla danej klasy:

    $policy = policy(App\User::class);

<a name="method-redirect"></a>
#### `redirect()` - przekierowanie {#collection-method}

Funkcja `redirect` zwraca [odpowiedź HTTP przekierowania](/docs/{{version}}/responses#redirects) lub zwraca instancję przekierowującą, jeśli została wywołana bez żadnych argumentów:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` - zgłoś wyjątek {#collection-method}

Funkcja `report` zgłasza wyjątek za pomocą metody `report` [(procedury obsługi wyjątków)](/docs/{{version}}/errors#the-exception-handler):

    report($e);

<a name="method-request"></a>
#### `request()` - żądanie {#collection-method}

Funkcja `request` zwraca bieżącą instancję [żądania](/docs/{{version}}/requests) lub pobiera element wejściowy:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` - przechwytuje wyjatki {#collection-method}

Funkcja `rescue` wykonuje wybraną Closure i przechwytuje wszelkie wyjątki, które występują podczas jego wykonywania. Wszystkie przechwycone wyjątki będą wysyłane do metody `report` [procedury obsługi wyjątku](/docs/{{version}}/errors#the-exception-handler); jednak żądanie będzie kontynuować przetwarzanie:

    return rescue(function () {
        return $this->method();
    });

Możesz także przekazać drugi argument do funkcji `rescue`. Ten argument będzie wartością "domyślną", która powinna zostać zwrócona, jeśli wystąpi wyjątek podczas wykonywania Closure:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>
#### `resolve()` - rozwiąż klasę {#collection-method}

Funkcja `resolve` rozwiązuje daną klasę lub nazwę interfejsu do swojej instancji za pomocą [kontenera usług](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` - odpowiedz {#collection-method}

Funkcja `response` tworzy instancję [odpowiedzi](/docs/{{version}}/responses) lub pobiera instancję fabryki odpowiedzi:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

Funkcja `retry` próbuje wykonać dane wywołanie, dopóki nie zostanie osiągnięty określony próg maksymalnego próbowania. Jeśli wywołanie zwrotne nie wygeneruje wyjątku, zwrócona zostanie jego wartość zwracana. Jeśli wywołanie zwróci wyjątek, zostanie ono automatycznie ponowione. Jeśli maksymalna liczba prób zostanie przekroczona, zostanie zgłoszony wyjątek:

    return retry(5, function () {
        // Attempt 5 times while resting 100ms in between attempts...
    }, 100);

<a name="method-session"></a>
#### `session()` - sesja {#collection-method}

Funkcja `session` może być używana do uzyskania lub ustawienia wartości [sesji](/docs/{{version}}/session):

    $value = session('key');

Możesz ustawić wartości, przekazując do pary funkcję par klucz / wartość:

    session(['chairs' => 7, 'instruments' => 3]);

Magazyn sesji zostanie zwrócony, jeśli do funkcji nie zostanie przekazana żadna wartość:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

Funkcja `tap` przyjmuje dwa argumenty: dowolną wartość `$value` i Closure. `$value` zostanie przekazana do Closure, a następnie zwrócona przez funkcję `tap`. Zwracana wartość Closure jest nieistotna:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Jeśli żadne Closure nie zostanie przekazana do funkcji `tap`, możesz wywołać dowolną metodę na podanej wartości `$value`. Wartością zwracaną wywołanej metody zawsze będzie `$value`, niezależnie od tego, co metoda faktycznie zwraca w swojej definicji. Na przykład, metoda Eloquent `update` zwykle zwraca liczbę całkowitą. Możemy jednak zmusić metodę do zwrócenia samego modelu przez łańcuchowe wywołanie metody `update` poprzez funkcję `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

<a name="method-today"></a>
#### `today()` - dzisiaj {#collection-method}

Funkcja `today` tworzy nową instancję `Illuminate\Support\Carbon` dla bieżącej daty:

    $today = today();

<a name="method-throw-if"></a>
#### `throw_if()` - zgłoś wyjątek jeżeli {#collection-method}

Funkcja `throw_if` zgłasza podany wyjątek, jeśli podane wyrażenie boolowskie ma wartość `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` - zgłoś wyjątek chyba że {#collection-method}

Funkcja `throw_if` zgłasza podany wyjątek, jeśli podane wyrażenie boolowskie ma wartość `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

Funkcja `trait_uses_recursive` zwraca wszystkie traits używane przez trait:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` - przekształć {#collection-method}

Funkcja `transform` wykonuje `Closure` na podanej wartości, jeśli wartość nie jest [puste](#method-blank) i zwraca wynik z  `Closure`:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

Wartość domyślna lub `Closure` może również zostać przekazana jako trzeci parametr metody. Ta wartość zostanie zwrócona, jeśli podana wartość jest pusta:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {#collection-method}

Funkcja `validator` tworzy nową instancję [walidacji](/docs/{{version}}/validation) z podanymi argumentami. Możesz użyć go zamiast wygodnej fasady `Validator`:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {#collection-method}

Funkcja `value` zwraca podaną wartość. Jednakże, jeśli przekażesz `Closure` do funkcji, `Closure` zostanie wykonane, wtedy jego wynik zostanie zwrócony:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>
#### `view()` - widok {#collection-method}

Funkcja `view` pobiera instancję [widok](/docs/{{version}}/views):

    return view('auth.login');

<a name="method-with"></a>
#### `with()` - z {#collection-method}

Funkcja `with` zwraca podaną wartość. Jeśli `Closure` zostanie przekazany jako drugi argument funkcji,`Closure` zostanie wykonane i jego wynik zostanie zwrócony:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
