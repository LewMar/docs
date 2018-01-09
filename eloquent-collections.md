# Eloquent: Collections

- [Introduction - Wprowadzenie](#introduction)
- [Available Methods - Dostępne metody](#available-methods)
- [Custom Collections - Kolekcje niestandardowe](#custom-collections)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Wszystkie zestawy wielu wyników zwracane przez Eloquent są instancjami obiektu `Illuminate\Database\Eloquent\Collection`, w tym wyniki pobierane za pomocą metody `get` lub dostępne za pośrednictwem relacji. Obiekt Eloquent kolekcji rozszerza [kolekcja podstawową](/docs/{{version}}/collections) Laravel-a, więc w naturalny sposób dziedziczy dziesiątki metod używanych do płynnej pracy z podstawową tablicą modeli Eloquent.

Oczywiście wszystkie kolekcje służą również jako iteratory, pozwalając ci na ich przechwytywanie, jak gdyby były prostymi tablicami PHP:

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

Jednak zbiory są o wiele potężniejsze niż tablice i udostępniają różne operacje mapowania / zmniejszania, które mogą być powiązane za pomocą intuicyjnego interfejsu. Na przykład usuń wszystkie nieaktywne modele i zbierz imię dla każdego pozostałego użytkownika:

    $users = App\User::all();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} Podczas gdy większość metod Eloquent zwraca nową instancję kolekcji Eloquent, metody `pluck`, `keys`, `zip`, `collapse`, `flatten` i `flip`  zwracają [kolekcję podstawową](/docs/{{version}}/collections) wystąpienia. Podobnie, jeśli operacja `map` zwraca kolekcję, która nie zawiera żadnych modeli Eloquent, będzie ona automatycznie rzutowana do kolekcji podstawowej.

<a name="available-methods"></a>
## Available Methods - Dostępne metody

### The Base Collection - Kolekcje podstawowe

Wszystkie elokwentne kolekcje rozszerzają podstawowy obiekt [kolekcji Laravel-a](/docs/{{version}}/collections); w związku z tym dziedziczą wszystkie potężne metody dostarczone przez podstawową klasę kolekcji:

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

[all](/docs/{{version}}/collections#method-all)
[average](/docs/{{version}}/collections#method-average)
[avg](/docs/{{version}}/collections#method-avg)
[chunk](/docs/{{version}}/collections#method-chunk)
[collapse](/docs/{{version}}/collections#method-collapse)
[combine](/docs/{{version}}/collections#method-combine)
[concat](/docs/{{version}}/collections#method-concat)
[contains](/docs/{{version}}/collections#method-contains)
[containsStrict](/docs/{{version}}/collections#method-containsstrict)
[count](/docs/{{version}}/collections#method-count)
[crossJoin](/docs/{{version}}/collections#method-crossjoin)
[dd](/docs/{{version}}/collections#method-dd)
[diff](/docs/{{version}}/collections#method-diff)
[diffKeys](/docs/{{version}}/collections#method-diffkeys)
[dump](/docs/{{version}}/collections#method-dump)
[each](/docs/{{version}}/collections#method-each)
[eachSpread](/docs/{{version}}/collections#method-eachspread)
[every](/docs/{{version}}/collections#method-every)
[except](/docs/{{version}}/collections#method-except)
[filter](/docs/{{version}}/collections#method-filter)
[first](/docs/{{version}}/collections#method-first)
[flatMap](/docs/{{version}}/collections#method-flatmap)
[flatten](/docs/{{version}}/collections#method-flatten)
[flip](/docs/{{version}}/collections#method-flip)
[forget](/docs/{{version}}/collections#method-forget)
[forPage](/docs/{{version}}/collections#method-forpage)
[get](/docs/{{version}}/collections#method-get)
[groupBy](/docs/{{version}}/collections#method-groupby)
[has](/docs/{{version}}/collections#method-has)
[implode](/docs/{{version}}/collections#method-implode)
[intersect](/docs/{{version}}/collections#method-intersect)
[isEmpty](/docs/{{version}}/collections#method-isempty)
[isNotEmpty](/docs/{{version}}/collections#method-isnotempty)
[keyBy](/docs/{{version}}/collections#method-keyby)
[keys](/docs/{{version}}/collections#method-keys)
[last](/docs/{{version}}/collections#method-last)
[map](/docs/{{version}}/collections#method-map)
[mapInto](/docs/{{version}}/collections#method-mapinto)
[mapSpread](/docs/{{version}}/collections#method-mapspread)
[mapToGroups](/docs/{{version}}/collections#method-maptogroups)
[mapWithKeys](/docs/{{version}}/collections#method-mapwithkeys)
[max](/docs/{{version}}/collections#method-max)
[median](/docs/{{version}}/collections#method-median)
[merge](/docs/{{version}}/collections#method-merge)
[min](/docs/{{version}}/collections#method-min)
[mode](/docs/{{version}}/collections#method-mode)
[nth](/docs/{{version}}/collections#method-nth)
[only](/docs/{{version}}/collections#method-only)
[pad](/docs/{{version}}/collections#method-pad)
[partition](/docs/{{version}}/collections#method-partition)
[pipe](/docs/{{version}}/collections#method-pipe)
[pluck](/docs/{{version}}/collections#method-pluck)
[pop](/docs/{{version}}/collections#method-pop)
[prepend](/docs/{{version}}/collections#method-prepend)
[pull](/docs/{{version}}/collections#method-pull)
[push](/docs/{{version}}/collections#method-push)
[put](/docs/{{version}}/collections#method-put)
[random](/docs/{{version}}/collections#method-random)
[reduce](/docs/{{version}}/collections#method-reduce)
[reject](/docs/{{version}}/collections#method-reject)
[reverse](/docs/{{version}}/collections#method-reverse)
[search](/docs/{{version}}/collections#method-search)
[shift](/docs/{{version}}/collections#method-shift)
[shuffle](/docs/{{version}}/collections#method-shuffle)
[slice](/docs/{{version}}/collections#method-slice)
[sort](/docs/{{version}}/collections#method-sort)
[sortBy](/docs/{{version}}/collections#method-sortby)
[sortByDesc](/docs/{{version}}/collections#method-sortbydesc)
[splice](/docs/{{version}}/collections#method-splice)
[split](/docs/{{version}}/collections#method-split)
[sum](/docs/{{version}}/collections#method-sum)
[take](/docs/{{version}}/collections#method-take)
[tap](/docs/{{version}}/collections#method-tap)
[toArray](/docs/{{version}}/collections#method-toarray)
[toJson](/docs/{{version}}/collections#method-tojson)
[transform](/docs/{{version}}/collections#method-transform)
[union](/docs/{{version}}/collections#method-union)
[unique](/docs/{{version}}/collections#method-unique)
[uniqueStrict](/docs/{{version}}/collections#method-uniquestrict)
[unless](/docs/{{version}}/collections#method-unless)
[values](/docs/{{version}}/collections#method-values)
[when](/docs/{{version}}/collections#method-when)
[where](/docs/{{version}}/collections#method-where)
[whereStrict](/docs/{{version}}/collections#method-wherestrict)
[whereIn](/docs/{{version}}/collections#method-wherein)
[whereInStrict](/docs/{{version}}/collections#method-whereinstrict)
[whereNotIn](/docs/{{version}}/collections#method-wherenotin)
[whereNotInStrict](/docs/{{version}}/collections#method-wherenotinstrict)
[zip](/docs/{{version}}/collections#method-zip)

</div>

<a name="custom-collections"></a>
## Custom Collections - Kolekcje niestandardowe

Jeśli chcesz użyć niestandardowego obiektu `Collection` z własnymi metodami rozszerzenia, możesz zastąpić metodę `newCollection` w swoim modelu:

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

Po zdefiniowaniu metody `newCollection` otrzymasz instancję kolekcji niestandardowej w dowolnym momencie, gdy funkcja Eloquent zwróci instancję `Collection` tego modelu. Jeśli chcesz użyć niestandardowej kolekcji dla każdego modelu w aplikacji, powinieneś zastąpić metodę `newCollection` na podstawowej klasie modelu rozszerzonej przez wszystkie modele.
