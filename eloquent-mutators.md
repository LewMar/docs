# Eloquent: Mutators

- [Introduction - Wprowadzenie](#introduction)
- [Accessors & Mutators - Accessory i mutatory](#accessors-and-mutators)
    - [Defining An Accessor - Definiowanie Accessora](#defining-an-accessor)
    - [Defining A Mutator - Definiowanie Mutatora](#defining-a-mutator)
- [Date Mutators - Daty mutatorów](#date-mutators)
- [Attribute Casting - Rzutowanie atrybutów](#attribute-casting)
    - [Array & JSON Casting - Tablice i rzutowanie JSON](#array-and-json-casting)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Accessory i mutatory umożliwiają formatowanie wartości Eloquent Attribute podczas pobierania lub ustawiania ich w instancjach modelu. Na przykład możesz użyć [encryptera Laravel](/docs/{{version}}/encryption), aby zaszyfrować wartość, gdy jest ona przechowywana w bazie danych, a następnie automatycznie odszyfrować atrybut, gdy uzyskujesz dostęp do niego na elokwentnym Modelu.

Oprócz niestandardowych akcesorów i mutatorów, Eloquent może automatycznie odlewać pola dat na instancje [Carbon](https://github.com/briannesbitt/Carbon), a nawet [pola tekstowe rzutowania do JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>
## Accessors & Mutators - Accessory i mutatory

<a name="defining-an-accessor"></a>
### Defining An Accessor - Definiowanie Accessora

Aby zdefiniować akcesora, utwórz w swoim modelu metodę `getFooAttribute`, w której `Foo` jest "starannie" literową nazwą kolumny, do której chcesz uzyskać dostęp. W tym przykładzie zdefiniujemy akcesor dla atrybutu `first_name`. Akcesor będzie automatycznie wywoływany przez Eloquent podczas próby pobrania wartości atrybutu `first_name`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

Jak widać, oryginalna wartość kolumny jest przekazywana do akcesora, umożliwiając manipulowanie i zwracanie wartości. Aby uzyskać dostęp do wartości akcesora, możesz po prostu uzyskać dostęp do atrybutu `first_name` w instancji modelu:

    $user = App\User::find(1);

    $firstName = $user->first_name;

Oczywiście możesz także użyć akcesorów, aby zwrócić nowe, wyliczone wartości z istniejących atrybutów:

    /**
     * Get the user's full name.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

<a name="defining-a-mutator"></a>
### Defining A Mutator - Definiowanie Mutatora

Aby zdefiniować mutator, zdefiniuj metodę `setFooAttribute` w swoim modelu, gdzie `Foo` jest "starannie" literową nazwą kolumny, do której chcesz uzyskać dostęp. Tak więc, ponownie zdefiniujmy mutator dla atrybutu `first_name`. Ten mutator zostanie wywołany automatycznie, gdy spróbujemy ustawić wartość atrybutu `first_name` na modelu:
    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

Mutator otrzyma wartość, która jest ustawiana na atrybucie, pozwalając ci na manipulowanie wartością i ustawienie zmanipulowanej wartości we właściwości wewnętrznej atrybutu `$attributes` atrybutu Eloquent. Na przykład, jeśli spróbujemy ustawić atrybut `first_name` na `Sally`:

    $user = App\User::find(1);

    $user->first_name = 'Sally';

W tym przykładzie funkcja `setFirstNameAttribute` zostanie wywołana z wartością `Sally`. Mutator następnie zastosuje funkcję `strtolower` do nazwy i ustawi jej wynikową wartość w wewnętrznej tablicy `$attributes`.

<a name="date-mutators"></a>
## Date Mutators - Daty mutatorów

Domyślnie, Eloquent konwertuje kolumny `created_at` i `updated_at` na instancje [Carbon](https://github.com/briannesbitt/Carbon), co rozszerza klasę PHP `DateTime` w celu dostarczenia zestawu pomocnych metod . Możesz dostosować, które daty są automatycznie mutowane, a nawet całkowicie wyłączyć tę mutację, zastępując właściwość `$dates` w swoim modelu:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

Gdy kolumna jest uważana za datę, możesz ustawić jej wartość na znacznik czasu UNIX, ciąg daty (`Ymd`), ciąg daty i godziny oraz oczywiście instancję `DateTime` / `Carbon`, a wartość daty automatycznie być poprawnie przechowywane w bazie danych:

    $user = App\User::find(1);

    $user->deleted_at = now();

    $user->save();

Jak wspomniano powyżej, podczas pobierania atrybutów wymienionych w Twojej właściwości `$dates`, będą one automatycznie rzutowane na instancje [Carbon](https://github.com/briannesbitt/Carbon), co pozwala na użycie dowolnej z metod Carbon'a na twoich atrybutach:

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### Date Formats - Formaty daty

Domyślnie znaczniki czasu są sformatowane jako `'Y-m-d H:i:s'`. Jeśli chcesz dostosować format znacznika czasu, ustaw właściwość `$dateFormat` w swoim modelu. Ta właściwość określa sposób przechowywania atrybutów daty w bazie danych, a także ich format, gdy model jest serializowany do tablicy lub JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## Attribute Casting - Rzutowanie atrybutów

Właściwość `$casts` w modelu zapewnia wygodną metodę konwertowania atrybutów na typowe typy danych. Właściwość `$casts` powinna być tablicą, w której klucz jest nazwą rzutowanego atrybutu, a wartość jest typem, do którego kolumna ma być przesyłana. Obsługiwane typy rzutowania to: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, i `timestamp`.

Na przykład, rzućmy atrybut `is_admin`, który jest przechowywany w naszej bazie danych jako liczba całkowita  (`0` lub `1`) do wartości boolowskiej:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

Teraz atrybut `is_admin` zawsze będzie rzutowany na wartość logiczną, gdy uzyskasz do niej dostęp, nawet jeśli wartość bazowa zostanie zapisana w bazie danych jako liczba całkowita:

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="array-and-json-casting"></a>
### Array & JSON Casting - Tablice i rzutowanie JSON

The `array` cast type is particularly useful when working with columns that are stored as serialized JSON. For example, if your database has a `JSON` or `TEXT` field type that contains serialized JSON, adding the `array` cast to that attribute will automatically deserialize the attribute to a PHP array when you access it on your Eloquent model:
Typ rzutowania `array` jest szczególnie przydatny podczas pracy z kolumnami, które są przechowywane jako serializowane JSON. Na przykład, jeśli twoja baza danych ma typ pola `JSON` lub `TEXT`, który zawiera serializowany JSON, dodanie `tablicy` rzutowania do tego atrybutu automatycznie deserializuje atrybut do tablicy PHP, kiedy uzyskujesz dostęp do niej w Modelu Wymownym:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Once the cast is defined, you may access the `options` attribute and it will automatically be deserialized from JSON into a PHP array. When you set the value of the `options` attribute, the given array will automatically be serialized back into JSON for storage:
Po zdefiniowaniu rzutowania możesz uzyskać dostęp do atrybutu `options` i zostanie on automatycznie przekształcony z JSON do tablicy PHP. Po ustawieniu wartości atrybutu `options`, podana tablica zostanie automatycznie ponownie zserializowana do JSON w celu jej przechowywania:

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
