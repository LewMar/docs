# Eloquent: Serialization

- [Introduction - Wprowadzenie](#introduction)
- [Serializing Models & Collections - Serializacja modeli i kolekcji](#serializing-models-and-collections)
    - [Serializing To Arrays - Serializacja do tablic](#serializing-to-arrays)
    - [Serializing To JSON - Serializacja do JSON](#serializing-to-json)
- [Hiding Attributes From JSON - Ukrywanie atrybutów z JSON](#hiding-attributes-from-json)
- [Appending Values To JSON - Dołączanie wartości do JSON](#appending-values-to-json)
- [Date Serialization - Serializacja daty](#date-serialization)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Podczas budowania interfejsów JSON API często trzeba konwertować modele i relacje na tablice lub JSON. Eloquent zawiera wygodne metody dokonywania tych konwersji, a także kontrolowanie, które atrybuty są zawarte w twoich serializacjach.

<a name="serializing-models-and-collections"></a>
## Serializing Models & Collections - Serializacja modeli i kolekcji

<a name="serializing-to-arrays"></a>
### Serializing To Arrays - Serializacja do tablic

Aby przekonwertować model i jego załadowane[relacje](/docs/{{version}}/eloquent-relationships) do tablicy, należy użyć metody `toArray`. Ta metoda jest rekursywna, więc wszystkie atrybuty i wszystkie relacje (w tym relacje relacji) zostaną przekonwertowane na tablice:

    $user = App\User::with('roles')->first();

    return $user->toArray();

Możesz także konwertować całe [kolekcje](/docs/{{version}}/eloquent-collections) modeli na tablice:

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### Serializing To JSON - Serializacja do JSON

Aby przekonwertować model na JSON, należy użyć metody `toJson`. Podobnie jak `toArray`, metoda `toJson` jest rekursywna, więc wszystkie atrybuty i relacje zostaną przekonwertowane na JSON:

    $user = App\User::find(1);

    return $user->toJson();

Alternatywnie możesz rzucić model lub kolekcję na ciąg znaków, który automatycznie wywoła metodę `toJson` w modelu lub kolekcji:

    $user = App\User::find(1);

    return (string) $user;

Ponieważ modele i kolekcje są konwertowane na JSON po rzutowaniu na ciąg, możesz zwracać obiekty wymowne bezpośrednio z tras lub kontrolerów aplikacji:

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## Hiding Attributes From JSON - Ukrywanie atrybutów z JSON

Czasami możesz chcieć ograniczyć atrybuty, takie jak hasła, które są zawarte w tablicy modelu lub w reprezentacji JSON. Aby to zrobić, dodaj do swojego modelu właściwość `$hidden`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {note} Podczas ukrywania relacji użyj nazwy metody relacji.

Alternatywnie można użyć właściwości `visible` w celu zdefiniowania białej listy atrybutów, które powinny znaleźć się w tablicy twojego modelu i reprezentacji JSON. Wszystkie pozostałe atrybuty zostaną ukryte, gdy model zostanie przekonwertowany na tablicę lub JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### Temporarily Modifying Attribute Visibility - Tymczasowa modyfikacja widoczności atrybutu

Jeśli chciałbyś, aby niektóre typowe ukryte atrybuty były widoczne w danej instancji modelu, możesz użyć metody `makeVisible`. Metoda `makeVisible` zwraca instancję modelu dla wygodnego łączenia łańcuchów:

    return $user->makeVisible('attribute')->toArray();

Podobnie, jeśli chcesz uczynić niektóre typowo widoczne atrybuty ukryte w danej instancji modelu, możesz użyć metody `makeHidden`.

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## Appending Values To JSON - Dołączanie wartości do JSON

Czasami, podczas przesyłania modeli do tablicy lub JSON, możesz chcieć dodać atrybuty, które nie mają odpowiedniej kolumny w bazie danych. Aby to zrobić, najpierw zdefiniuj [akcesor](/docs/{{version}}/eloquent-mutators) dla wartości:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

Po utworzeniu akcesora dodaj nazwę atrybutu do właściwości `appends` w modelu. Zauważ, że nazwy atrybutów są zwykle przywoływane w "snake case", nawet jeśli akcesor jest zdefiniowany za pomocą "camel case":

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

Po dodaniu atrybutu do listy `appends`, zostanie on włączony zarówno do tablicy modelu, jak i do reprezentacji JSON. Atrybuty w tablicy `appends` będą również respektować ustawienia `visible` i `hidden` skonfigurowane w modelu.

<a name="date-serialization"></a>
## Date Serialization - Serializacja daty

Laravel rozszerza bibliotekę dat [Carbon](https://github.com/briannesbitt/Carbon), aby zapewnić wygodne dostosowywanie formatu serializacyjnego JSON firmy Carbon. Aby zindywidualizować sposób, w jaki wszystkie daty emisji Carbon w twojej aplikacji są serializowane, użyj metody `Carbon::serializeUsing`. Metoda `serializeUsing` akceptuje Closure, które zwraca ciąg znaków reprezentujący datę serializacji JSON:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Carbon;
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
            Carbon::serializeUsing(function ($carbon) {
                return $carbon->format('U');
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
