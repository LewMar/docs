# Collections

- [Introduction - Wprowadzenie](#introduction)
    - [Creating Collections - Tworzenie Kolekcji](#creating-collections)
    - [Extending Collections - Rozszerzanie kolekcji](#extending-collections)
- [Available Methods - Dostępne metody](#available-methods)
- [Higher Order Messages - Komunikaty wyższego rzędu](#higher-order-messages)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Klasa `Illuminate\Support\Collection` zapewnia płynne i wygodne opakowanie do pracy z tablicami danych. Na przykład sprawdź poniższy kod. Użyjemy helpera `collect` do utworzenia nowej instancji kolekcji z tablicy, uruchomienia funkcji `strtoupper` na każdym elemencie, a następnie usunięcia wszystkich pustych elementów:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

Jak widać, klasa `Collection` pozwala na łańcuchowanie swoich metod w celu płynnego mapowania i zmniejszania podstawowej tablicy. Ogólnie rzecz biorąc, kolekcje są niezmienne, co oznacza, że każda metoda `Collection` zwraca całkowicie nową instancję `Collection`.

<a name="creating-collections"></a>
### Creating Collections - Tworzenie Kolekcji

Jak wspomniano powyżej, pomocnik `collect` zwraca nową instancję `Illuminate\Support\Collection` dla danej tablicy. Stworzenie kolekcji jest tak proste, jak:

    $collection = collect([1, 2, 3]);

> {tip} Wyniki zapytań [Eloquent](/docs/{{version}}/eloquent) są zawsze zwracane jako instancje `Collection`.

<a name="extending-collections"></a>
### Extending Collections - Rozszerzanie kolekcji

Kolekcje są "makrowalne", co pozwala dodawać dodatkowe metody do klasy `Collection` w czasie wykonywania. Na przykład poniższy kod dodaje metodę `toUpper` do klasy `Collection`:

    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });

    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

Zazwyczaj należy zadeklarować makra kolekcji w [dostawcy usług](/docs/{{version}}/providers).

<a name="available-methods"></a>
## Available Methods - Dostępne metody

W pozostałej części tej dokumentacji omówimy każdą metodę dostępną w klasie `Collection`. Pamiętaj, że wszystkie te metody mogą być powiązane z płynną manipulacją podstawową tablicą. Co więcej, prawie każda metoda zwraca nową instancję `Collection`, co pozwala zachować oryginalną kopię kolekcji, gdy jest to konieczne:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

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

<a name="method-all"></a>
#### `all()` - wszystko {#collection-method .first-collection-method}

Metoda `all` zwraca tablicę bazową reprezentowaną przez kolekcję:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` - średnia {#collection-method}

Alias do metody [`avg`](#method-avg).

<a name="method-avg"></a>
#### `avg()` - średnia {#collection-method}

Metoda `avg` zwraca [wartość średnią](https://en.wikipedia.org/wiki/Average) danego klucza:

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` - porcjuje {#collection-method}

Metoda `chunk` dzieli kolekcję na wiele mniejszych kolekcji o danym rozmiarze:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

Ta metoda jest szczególnie użyteczna w [widokach](/docs/{{version}}/views) podczas pracy z siatkami, takim jak [Bootstrap grid](https://getbootstrap.com/css/#grid). Wyobraź sobie, że masz kolekcję modeli [Eloquent](/docs/{{version}}/eloquent), które chcesz wyświetlić w siatce:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` - zwiń, połacz {#collection-method}

Metoda `collapse` zwija kolekcję tablic w jedną, płaską kolekcję:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` - łaczyć w sobie {#collection-method}

Metoda `combine` łączy klucze kolekcji z wartościami innej tablicy lub kolekcji:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` - dołącz {#collection-method}

Metoda `concat` dołącza podaną wartość `array` lub kolekcji do końca kolekcji:

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

<a name="method-contains"></a>
#### `contains()` - czy zawiera ? {#collection-method}

Metoda `contains` określa, czy kolekcja zawiera dany element:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

Możesz także przekazać parę klucz / wartość do metody `contains`, która określi, czy dana para istnieje w kolekcji:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

Na koniec możesz również przekazać wywołanie zwrotne do metody `contains`, aby wykonać swój test prawdy:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

Metoda `contains` używa "luźnych" porównań podczas sprawdzania wartości pozycji, co oznacza, że ciąg o wartości całkowitej będzie uznawany za równy liczbie całkowitej o tej samej wartości. Użyj metody [`containsStrict`](#method-containsstrict) do filtrowania przy użyciu "ścisłych" porównań.

<a name="method-containsstrict"></a>
#### `containsStrict()` - ściśle zawiera {#collection-method}

Ta metoda ma te sam oznaczenie co metoda [`contains`](#method-contains); jednak wszystkie wartości są porównywane przy użyciu "ścisłych" porównań.

<a name="method-count"></a>
#### `count()` - zlicz {#collection-method}

Metoda `count` zwraca całkowitą liczbę elementów w kolekcji:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-crossjoin"></a>
#### `crossJoin()` - łącz krzyżowo {#collection-method}

Metoda `crossJoin` łączy krzyżowo wartości kolekcji z podanymi tablicami lub kolekcjami, zwracając iloczyn kartezjański z wszystkimi możliwymi kombinacjami:

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b']);

    $matrix->all();

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

    $matrix->all();

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-dd"></a>
#### `dd()` - podejżyj kolekcję {#collection-method}

Metoda `dd` zrzuca elementy kolekcji i kończy wykonywanie skryptu:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dd();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Jeśli nie chcesz przestać wykonywać skryptu, użyj zamiast tego metody [`dump`](#method-dump).

<a name="method-diff"></a>
#### `diff()` - porównaj {#collection-method}

Metoda `diff` porównuje kolekcję z inną kolekcją lub zwykłą tablicą `array` PHP opartą na jej wartościach. Ta metoda zwróci wartości z oryginalnego zbioru, które nie są obecne w podanej kolekcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` - porownaj asocjacyjnie {#collection-method}

Metoda `diffAssoc` porównuje kolekcję z inną kolekcją lub prostą tablicą `array` PHP opartą na jej kluczach i wartościach. Ta metoda zwróci pary klucz / wartość z oryginalnego zbioru, które nie są obecne w podanej kolekcji:

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` - porównaj klucze {#collection-method}

Metoda `diffKeys` porównuje kolekcję z inną kolekcją lub prostą tablicą `array` PHP opartą na jego kluczach. Ta metoda zwróci pary klucz / wartość z oryginalnego zbioru, które nie są obecne w podanej kolekcji:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-dump"></a>
#### `dump()` - zrzut {#collection-method}

Metoda `dump` zrzuca elementy kolekcji:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dump();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Jeśli chcesz przestać wykonywać skrypt po zrzuceniu kolekcji, użyj zamiast tego metody [`dd`] (# method-dd).

<a name="method-each"></a>
#### `each()` - iterowanie {#collection-method}

Metoda `each` iteruje nad elementami w kolekcji i przekazuje każdy element do wywołania zwrotnego:

    $collection = $collection->each(function ($item, $key) {
        //
    });

Jeśli chcesz przerwać iterowanie elementów, możesz zwrócić `false` z twojego wywołania zwrotnego:

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>
#### `eachSpread()` - iteruj zagnieżdzone {#collection-method}

Metoda `eachSpread` iteruje nad elementami kolekcji, przekazując każdą wartość zagnieżdżonego elementu do danego wywołania zwrotnego:

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

Jeśli chcesz przerwać iterowanie elementów, możesz zwrócić `false` z twojego wywołania zwrotnego:

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>
#### `every()` - wszystkie spełnione {#collection-method}

Metodę `every` można wykorzystać do sprawdzenia, czy wszystkie elementy kolekcji przechodzą dany test prawdy:


    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

<a name="method-except"></a>
#### `except()` - z wyjątkiem {#collection-method}

Metoda `except` zwraca wszystkie elementy w kolekcji oprócz tych z określonymi kluczami:

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

Odwrotność `except`, zobacz metodę [only](#method-only)).

<a name="method-filter"></a>
#### `filter()` - filtruj {#collection-method}

Metoda `filter` filtruje kolekcję za pomocą danego wywołania zwrotnego, zachowując tylko te elementy, które pomyślnie przejdą dany test prawdy:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Jeśli nie zostanie podane żadne wywołanie zwrotne, wszystkie wpisy kolekcji, które są równoważne `false`, zostaną usunięte:

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

Odwrotność `filter`, patrz metoda [reject](#method-reject).

<a name="method-first"></a>
#### `first()` - pierwszy {#collection-method}

Metoda `first` zwraca pierwszy element w kolekcji, który przechodzi dany test prawdy:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

Możesz także wywołać metodę `first` bez argumentów, aby uzyskać pierwszy element w kolekcji. Jeśli kolekcja jest pusta, zwracana jest wartość `null`:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-first-where"></a>
#### `firstWhere()` - pierwszy jeżeli {#collection-method}

Metoda `firstWhere` zwraca pierwszy element w kolekcji z daną parą klucz / wartość:

    $collection = collect([
        ['name' => 'Regena', 'age' => 12],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

Możesz również wywołać metodę `firstWhere` z operatorem:

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

<a name="method-flatmap"></a>
#### `flatMap()` - spłaszcz mapując {#collection-method}

Metoda `flatMap` iteruje przez kolekcję i przekazuje każdą wartość do danego wywołania zwrotnego. Wywołanie zwrotne może modyfikować element i zwracać go, tworząc w ten sposób nową kolekcję zmodyfikowanych elementów. Następnie tablica jest spłaszczona do 1 poziomu:

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` - spłaszcz {#collection-method}

Metoda `flatten` spłaszcza wielowymiarową kolekcję w jeden wymiar:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

Opcjonalnie możesz przekazać funkcję argument "depth" (głebokość):

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

W tym przykładzie, wywołanie funkcji `flatten` bez podania głębi również spłaszczyłoby zagnieżdżone tablice, czego wynikiem są `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Podanie głębokości pozwala ograniczyć poziomy zagnieżdżonych tablic, które zostaną spłaszczone.

<a name="method-flip"></a>
#### `flip()` - zamienia klucze z wartością {#collection-method}

Metoda `flip` zamienia klucze kolekcji na odpowiadające im wartości:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` - zapomnij {#collection-method}

Metoda `forget` usuwa element z kolekcji za pomocą swojego klucza:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} W przeciwieństwie do większości innych metod zbierania, `forget` nie zwraca nowej zmodyfikowanej kolekcji; modyfikuje kolekcję, do której jest wywoływana.

<a name="method-forpage"></a>
#### `forPage()` stronicuje {#collection-method}

Metoda `forPage` zwraca nową kolekcję zawierającą elementy, które byłyby obecne na danym numerze strony. Metoda przyjmuje numer strony jako pierwszy argument i liczbę pozycji do wyświetlenia na stronie jako drugi argument:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` - weż {#collection-method}

Metoda `get` zwraca element dla danego klucza. Jeśli klucz nie istnieje, zwracana jest wartość `null`:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Opcjonalnie można podać wartość domyślną jako drugi argument:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

Możesz nawet przekazać wywołanie zwrotne jako wartość domyślną. Wynik wywołania zwrotnego zostanie zwrócony, jeśli określony klucz nie istnieje:

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` - grupuj po {#collection-method}

Metoda `groupBy` grupuje elementy kolekcji według podanego klucza:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

Oprócz przekazywania `key` ciągu, możesz również przekazać wywołanie zwrotne. Callback powinien zwrócić wartość, którą chcesz przypisać do grupy przez:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` - czy jest? {#collection-method}

Metoda `has` określa, czy dany klucz istnieje w kolekcji:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` - sklejaj {#collection-method}

Metoda `implode` łączy elementy w kolekcji. Jego argumenty zależą od rodzaju elementów w kolekcji. Jeśli kolekcja zawiera tablice lub obiekty, powinieneś przekazać klucz atrybutów, które chcesz dołączyć, oraz ciąg "kleju", który chcesz umieścić między wartościami:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Jeśli kolekcja zawiera proste ciągi lub wartości liczbowe, po prostu podaj "klej" jako jedyny argument metody:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()`- wytnij {#collection-method}

Metoda `intersect` usuwa wszelkie wartości z oryginalnej kolekcji, które nie są obecne w danej `array` lub kolekcji. Powstała kolekcja zachowa klucze do oryginalnej kolekcji:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` - wytnij po kluczu {#collection-method}

Metoda `intersectByKeys` usuwa wszystkie klucze z oryginalnej kolekcji, które nie są obecne w danej `array` lub kolekcji:

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` - czy pusty? {#collection-method}

Metoda `isEmpty` zwraca `true`, jeśli kolekcja jest pusta; w przeciwnym razie zwracane jest `false`:

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` - czy nie pusty? {#collection-method}

Metoda `isNotEmpty` zwraca `true`, jeśli kolekcja nie jest pusta; w przeciwnym razie zwracane jest `false`:

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` - wypisuj według klucza {#collection-method}

Metoda `keyBy` wpisuje kolekcję według podanego klucza. Jeśli wiele elementów ma ten sam klucz, tylko nowa pojawi się w nowej kolekcji:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

Możesz również przekazać wywołanie zwrotne do metody. Callback powinien zwrócić wartość kluczową do kolekcji przez:

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` - klucze {#collection-method}

Metoda `keys` zwraca wszystkie klucze kolekcji:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` - ostatni {#collection-method}

Metoda `last` zwraca ostatni element z kolekcji, która przechodzi dany test prawdy:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

Możesz również wywołać metodę `last` bez argumentów, aby uzyskać ostatni element w kolekcji. Jeśli kolekcja jest pusta, zwracana jest wartość `null`:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-macro"></a>
#### `macro()` - makro {#collection-method}

Statyczna metoda `macro` pozwala dodawać metody do klasy `Collection` w czasie wykonywania. Więcej informacji można znaleźć w dokumentacji dotyczącej [rozszerzania kolekcji](#extending-collections).

<a name="method-make"></a>
#### `make()` stwórz {#collection-method}

Statyczna metoda `make` tworzy nową instancję kolekcji. Zobacz sekcję [tworzenie kolekcji](#creating-collections).

<a name="method-map"></a>
#### `map()` - przetwórz {#collection-method}

Metoda `map` iteruje przez kolekcję i przekazuje każdą wartość do danego wywołania zwrotnego. Callback może modyfikować element i zwracać go, tworząc nową kolekcję zmodyfikowanych elementów:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note} Podobnie jak większość innych metod zbierania, `map` zwraca nową instancję kolekcji; nie modyfikuje kolekcji, do której jest wywoływana. Jeśli chcesz przekształcić oryginalną kolekcję, użyj metody [`transform`](#method-transform).

<a name="method-mapinto"></a>
#### `mapInto()` - przetwórz na klasę {#collection-method}

Metoda `mapInto()` iteruje nad kolekcją, tworząc nową instancję danej klasy, przekazując ją do konstruktora:

    class Currency
    {
        /**
         * Create a new currency instance.
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` - przetwórz zagnieżdzone {#collection-method}

Metoda `mapSpread` iteruje nad elementami kolekcji, przekazując każdą pozycję zagnieżdżonego elementu do danego wywołania zwrotnego. Callback może modyfikować element i zwracać go, tworząc nową kolekcję zmodyfikowanych elementów:

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($odd, $even) {
        return $odd + $even;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` - przetwórz do grupy {#collection-method}

Metoda `mapToGroups` grupuje pozycje kolekcji według danego wywołania zwrotnego. Zwrot powinien zwrócić tablicę asocjacyjną zawierającą pojedynczą parę klucz / wartość, tworząc w ten sposób nową kolekcję zgrupowanych wartości:

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);

    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });

    $grouped->toArray();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johhny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()`- przetwórz z kluczami {#collection-method}

Metoda `mapWithKeys` iteruje przez kolekcję i przekazuje każdą wartość do danego wywołania zwrotnego. Zwrot powinien zwrócić tablicę asocjacyjną zawierającą jedną parę klucz / wartość:

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` - maksymalne {#collection-method}

Metoda `max` zwraca maksymalną wartość danego klucza:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` - mediana {#collection-method}

Metoda `median` zwraca wartość [mediany](https://pl.wikipedia.org/wiki/Mediana) danego klucza:

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` - scalanie {#collection-method}

Metoda `merge` scala daną tablicę lub kolekcję z oryginalną kolekcją. Jeśli klucz łańcuchowy w podanych elementach pasuje do klucza ciągu w oryginalnej kolekcji, wartość danego elementu spowoduje nadpisanie wartości z oryginalnej kolekcji:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

Jeśli klucze danego przedmiotu mają wartość numeryczną, wartości zostaną dodane na końcu kolekcji:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` - minimalna {#collection-method}

Metoda `min` zwraca minimalną wartość danego klucza:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` - dominanta (najczęstrzy) {#collection-method}

Metoda `mode` zwraca wartość [dominanta](https://en.wikipedia.org/wiki/Mode_(statistics)) danego klucza:

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` - enty element {#collection-method}

Metoda `nth` tworzy nową kolekcję składającą się z każdego n-tego elementu:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

Opcjonalnie można przekazać przesunięcie jako drugi argument:

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` - tylko {#collection-method}

Metoda `only` zwraca elementy w kolekcji z określonymi kluczami:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Odwrotność `only`, patrz metoda [except](#method-except).

<a name="method-pad"></a>
#### `pad()` - dopełnij {#collection-method}

Metoda `pad` będzie wypełniać tablicę podaną wartością, dopóki tablica nie osiągnie określonego rozmiaru. Ta metoda zachowuje się jak funkcja PHP [array_pad](https://secure.php.net/manual/en/function.array-pad.php).

Aby dopełniania po lewej stronie, należy określić rozmiar ujemny. Nie ma dopełnienia, jeśli bezwzględna wartość podanego rozmiaru jest mniejsza lub równa długości tablicy:

    $collection = collect(['A', 'B', 'C']);

    $filtered = $collection->pad(5, 0);

    $filtered->all();

    // ['A', 'B', 'C', 0, 0]

    $filtered = $collection->pad(-5, 0);

    $filtered->all();

    // [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>
#### `partition()` {#collection-method}

Metoda `partition` może być łączona z funkcją PHP `list` w celu oddzielenia elementów, które przechodzą dany test prawdy od tych, które nie:

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

<a name="method-pipe"></a>
#### `pipe()` - przetwórz przez {#collection-method}

Metoda `pipe` przekazuje kolekcję do danego wywołania zwrotnego i zwraca wynik:

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pluck"></a>
#### `pluck()` pobierz dla klucza {#collection-method}

Metoda `pluck` pobiera wszystkie wartości dla danego klucza:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

Możesz również określić, jak chcesz, aby kolekcja wynikowa była kluczowana:

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` - zdejmij ostatni element {#collection-method}

Metoda `pop` usuwa i zwraca ostatni element z kolekcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` - dodaj na początku {#collection-method}

Metoda `prepend` dodaje element do początku kolekcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

Możesz również przekazać drugi argument, aby ustawić klucz pozycji wstępnej:

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` - usuń po kluczu {#collection-method}

Metoda `pull` usuwa i zwraca element z kolekcji za pomocą swojego klucza:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` - dodaje na końcu {#collection-method}

Metoda `push` dołącza element do końca kolekcji:

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` - ustaw klucz i wartość {#collection-method}

Metoda `put` ustawia określony klucz i wartość w kolekcji:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

Metoda `random` zwraca losowy element z kolekcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

Możesz opcjonalnie przekazać liczbę całkowitą do `random`, aby określić liczbę elementów, które chcesz losowo pobrać. Zbiór przedmiotów jest zawsze zwracany, gdy jednoznacznie podajesz liczbę przedmiotów, które chcesz otrzymać:


    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` - redukuje {#collection-method}

Metoda `reduce` redukuje kolekcję do pojedynczej wartości, przekazując wynik każdej iteracji do kolejnej iteracji:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

Wartość `$carry` w pierwszej iteracji to `null`; możesz jednak określić jego wartość początkową, przekazując drugi argument do `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` - odrzucony {#collection-method}

Metoda `reject` filtruje kolekcję za pomocą danego wywołania zwrotnego. Zwrot powinien zwrócić wartość `true`, jeśli element powinien zostać usunięty z wynikowej kolekcji:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

Odwrotność metody `reject`, patrz metoda [`filter`](#method-filter).

<a name="method-reverse"></a>
#### `reverse()` - odwróć {#collection-method}

Metoda `reverse` odwraca kolejność elementów kolekcji, zachowując oryginalne klucze:

    $collection = collect(['a', 'b', 'c', 'd', 'e']);

    $reversed = $collection->reverse();

    $reversed->all();

    /*
        [
            4 => 'e',
            3 => 'd',
            2 => 'c',
            1 => 'b',
            0 => 'a',
        ]
    */

<a name="method-search"></a>
#### `search()` - szukaj {#collection-method}

Metoda `search` przeszukuje zbiór dla podanej wartości i zwraca jego klucz, jeśli został znaleziony. Jeśli element nie zostanie znaleziony, zwracane jest "false".

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

Wyszukiwanie odbywa się za pomocą "luźnego" porównania, co oznacza, że ciąg z wartością całkowitą będzie uznawany za równy liczbie całkowitej o tej samej wartości. Aby użyć porównania "ścisłego", należy podać `true` jako drugi argument metody:

    $collection->search('4', true);

    // false

Alternatywnie możesz przekazać własne wywołanie zwrotne, aby wyszukać pierwszy przedmiot, który przejdzie test prawdy:

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` - usuń i zwruć 1-el {#collection-method}

Metoda `shift` usuwa i zwraca pierwszy element z kolekcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` - potasuj {#collection-method}

Metoda `shuffle` losowo tasuje elementy w kolekcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` zwróć od pozycji {#collection-method}

Metoda `slice` zwraca fragment kolekcji rozpoczynający się od danego indeksu:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

Jeśli chcesz ograniczyć rozmiar zwracanego plasterka, podaj pożądany rozmiar jako drugi argument metody:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

Zwrócony plaster domyślnie zachowuje klucze. Jeśli nie chcesz zachować oryginalnych kluczy, możesz użyć metody [`values`](#method-values) do ponownego zindeksowania.

<a name="method-sort"></a>
#### `sort()` - sortuj {#collection-method}

Metoda `sort` sortuje kolekcję. Sortowana kolekcja zachowuje oryginalne klucze tablicy, więc w tym przykładzie użyjemy metody [`values`](#method-values) do zresetowania kluczy do kolejno ponumerowanych indeksów:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

Jeśli twoje potrzeby związane z sortowaniem są bardziej zaawansowane, możesz przekazać wywołanie zwrotne do `sort` za pomocą własnego algorytmu. Zapoznaj się z dokumentacją PHP na [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), która jest metodą `sort` kolekcji wzywa pod maską.

> {tip} Jeśli chcesz posortować kolekcję zagnieżdżonych tablic lub obiektów, zapoznaj się z metodami [`sortBy`](#method-sortby)  i [` sortByDesc`](#method-sortbydesc).

<a name="method-sortby"></a>
#### `sortBy()` - sortuj po {#collection-method}

Metoda `sortBy` sortuje kolekcję według podanego klucza. Sortowana kolekcja zachowuje oryginalne klucze tablicy, więc w tym przykładzie użyjemy metody [`values`](#method-values) do zresetowania kluczy do kolejno ponumerowanych indeksów:

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

Możesz także przekazać własne wywołanie zwrotne, aby określić sposób sortowania wartości kolekcji:

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` - sortuj po malejąco {#collection-method}

Ta metoda ma ten samo oznaczenie co metoda [`sortBy`](#method-sortby), ale sortuje kolekcję w odwrotnej kolejności.

<a name="method-splice"></a>
#### `splice()` - wycina kawałek {#collection-method}

Metoda `splice` usuwa i zwraca kawałek elementów rozpoczynających się od określonego indeksu:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

Możesz przekazać drugi argument, aby ograniczyć rozmiar wynikowej porcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

Ponadto możesz przekazać trzeci argument zawierający nowe elementy, aby zastąpić elementy usunięte z kolekcji:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` - dziel {#collection-method}

Metoda `split` dzieli kolekcję na określoną liczbę grup:

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` - sumuj {#collection-method}

Metoda `sum` zwraca sumę wszystkich elementów w kolekcji:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

Jeśli kolekcja zawiera zagnieżdżone tablice lub obiekty, należy przekazać klucz do użycia w celu określenia wartości do zsumowania:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

Ponadto możesz przekazać własne wywołanie zwrotne, aby określić, które wartości kolekcji mają być sumą:

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` - weż ile {#collection-method}

Metoda `take` zwraca nową kolekcję z określoną liczbą elementów:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

Możesz także przekazać ujemną liczbę całkowitą, aby pobrać określoną liczbę elementów z końca kolekcji:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` - dotknięcie {#collection-method}

The `tap` method passes the collection to the given callback, allowing you to "tap" into the collection at a specific point and do something with the items while not affecting the collection itself:
Metoda `tap` przekazuje kolekcję do danego wywołania zwrotnego, umożliwiając "dotknięcie" kolekcji w określonym punkcie i zrobienie czegoś z elementami bez wpływu na samą kolekcję:

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` - ile razy {#collection-method}

Statyczna metoda `times` tworzy nową kolekcję, wywołując wywołanie zwrotne określoną ilość razy:

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

Ta metoda może być przydatna w połączeniu z fabrykami do tworzenia modeli [Eloquent](/docs/{{version}}/eloquent):

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });

    $categories->all();

    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */

<a name="method-toarray"></a>
#### `toArray()` - do tablicy {#collection-method}

Metoda `toArray` konwertuje kolekcję na zwykłą tablicę `array` PHP. Jeśli wartości kolekcji to [Eloquent](/docs/{{version}}/eloquent), modele również zostaną przekonwertowane na tablice:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray` także konwertuje wszystkie zagnieżdżone obiekty kolekcji do tablicy. Jeśli chcesz uzyskać surową, podstawową tablicę, użyj zamiast tego metody [`all`](#method-all).

<a name="method-tojson"></a>
#### `toJson()` - do Json-a {#collection-method}

Metoda `toJson` konwertuje kolekcję na ciąg serializowany JSON:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` - transformuj {#collection-method}

Metoda `transform` iteruje nad kolekcją i wywołuje dane wywołanie zwrotne z każdym elementem w kolekcji. Pozycje w kolekcji zostaną zastąpione wartościami zwracanymi przez wywołanie zwrotne:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note} W przeciwieństwie do większości innych metod zbierania, `transform` modyfikuje samą kolekcję. Jeśli chcesz zamiast tego utworzyć nową kolekcję, użyj metody [`map`](#method-map).

<a name="method-union"></a>
#### `union()` - zjednocz {#collection-method}

Metoda `union` dodaje podaną tablicę do kolekcji. Jeśli podana tablica zawiera klucze, które są już w oryginalnej kolekcji, preferowane będą wartości oryginalnego zbioru:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` - unikalne {#collection-method}

Metoda `unique` zwraca wszystkie unikalne elementy w kolekcji. Zwrócona kolekcja zachowuje oryginalne klucze tablicy, więc w tym przykładzie użyjemy metody [`values`](#method-values) do zresetowania kluczy do kolejno ponumerowanych indeksów:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

W przypadku zagnieżdżonych tablic lub obiektów można określić klucz używany do określenia wyjątkowości:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

Możesz również przekazać własne wywołanie zwrotne, aby określić unikalność elementu:

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

Metoda `unique` używa "luźnych" porównań podczas sprawdzania wartości pozycji, co oznacza, że ciąg o wartości całkowitej będzie uznawany za równy liczbie całkowitej o tej samej wartości. Użyj metody [`uniqueStrict`](#method-uniquestrict) do filtrowania przy użyciu "ścisłych" porównań.

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` - unikalne ściśle {#collection-method}

Ta metoda ma taki samo oznaczenie jak metoda [`unique`](#method-unique); jednak wszystkie wartości są porównywane przy użyciu "ścisłych" porównań.

<a name="method-unless"></a>
#### `unless()` {#collection-method}

Metoda `unless` będzie wykonywała dane wywołanie, chyba że pierwszy argument podany do metody zwróci wartość 'true`:

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

Odwrotnością `unless` jest metodę [`when`](#method-when).

<a name="method-unwrap"></a>
#### `unwrap()` {#collection-method}

Statyczna metoda `unwrap` zwraca podstawowe elementy kolekcji z podanej wartości, gdy ma to zastosowanie:

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-values"></a>
#### `values()` - wartości {#collection-method}

Metoda `values` zwraca nową kolekcję z kluczami zresetowanymi do kolejnych liczb całkowitych:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` - kiedy {#collection-method}

Metoda `when` wykona dane wywołanie zwrotne, gdy pierwszy argument danej metody zostanie zwrócony do `true`:

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->when(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 4]

Odwrotnością `when` jest metoda [` unless`](#method-unless).

<a name="method-where"></a>
#### `where()` - gdzie {#collection-method}

Metoda `where` filtruje kolekcję według danej pary klucz / wartość:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

Metoda `where` używa "luźnych" porównań podczas sprawdzania wartości pozycji, co oznacza, że ciąg z wartością całkowitą będzie uznawany za równy liczbie całkowitej o tej samej wartości. Użyj metody [`whereStrict`](#method-wherestrict) do filtrowania przy użyciu "ścisłych" porównań.

<a name="method-wherestrict"></a>
#### `whereStrict()` - gdzie ściśle {#collection-method}

Ta metoda ma te samo znaczenie co metoda [`where`](# method-where); jednak wszystkie wartości są porównywane przy użyciu "ścisłych" porównań.

<a name="method-wherein"></a>
#### `whereIn()` gdzie w {#collection-method}

Metoda `whereIn` filtruje kolekcję według podanego klucza / wartości zawartej w podanej macierzy:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */

Metoda `whereIn` używa "luźnych" porównań podczas sprawdzania wartości pozycji, co oznacza, że ciąg z wartością całkowitą będzie uznawany za równy liczbie całkowitej o tej samej wartości. Użyj metody [`whereInStrict`](#method-heldstrict) do filtrowania przy użyciu "ścisłych" porównań.

<a name="method-whereinstrict"></a>
#### `whereInStrict()` - gdzie w ściśle {#collection-method}

Ta metoda ma taki sam podpis jak metoda [`whereIn`](#metoda-w której); jednak wszystkie wartości są porównywane przy użyciu "ścisłych" porównań.

<a name="method-wherenotin"></a>
#### `whereNotIn()` - gdzie nie w {#collection-method}

Metoda `whereNotIn` filtruje kolekcję według podanego klucza / wartości, która nie jest zawarta w podanej macierzy:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

Metoda `whereNotIn` używa "luźnych" porównań podczas sprawdzania wartości pozycji, co oznacza, że łańcuch z wartością całkowitą będzie uznawany za równy liczbie całkowitej o tej samej wartości. Użyj metody [`whereNotInStrict`](#method-wherenotinstrict) do filtrowania przy użyciu "ścisłych" porównań.

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` - gdzie nie w ściśle {#collection-method}

Ta metoda ma taki sam podpis jak metoda [`whereNotIn`](#metoda-wherenotin); jednak wszystkie wartości są porównywane przy użyciu "ścisłych" porównań.

<a name="method-wrap"></a>
#### `wrap()` - opakuj {#collection-method}

Statyczna metoda `wrap` opakowuje podaną wartość w zbiorze, gdy ma zastosowanie:

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>
#### `zip()` - zepnij {#collection-method}

Metoda `zip` łączy wartości danej tablicy z wartościami kolekcji oryginalnej w odpowiednim indeksie:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## Higher Order Messages - Komunikaty wyższego rzędu

Kolekcje zapewniają również obsługę "komunikatów wyższego rzędu", które stanowią skróty do wykonywania wspólnych działań w kolekcjach. Metody gromadzenia wiadomości o wyższym poziomie to: `average`, `avg`, `contains`, `each`, `every`, `filter`, `first`, `flatMap`, `map`, `partition`, `reject`, `sortBy`, `sortByDesc`, i `sum`.

Każda wiadomość wyższego rzędu może być dostępna jako właściwość dynamiczna w instancji kolekcji. Na przykład, użyjmy `each` wyższego rzędu, aby wywołać metodę na każdym obiekcie w kolekcji:

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

Podobnie, możemy użyć `sum wiadomości wyższego rzędu, aby zebrać całkowitą liczbę "głosów" dla zbioru użytkowników:

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;
