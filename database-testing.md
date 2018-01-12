# Database Testing

- [Introduction - Wprowadzenie](#introduction)
- [Generating Factories - Generowanie fabryk](#generating-factories)
- [Resetting The Database After Each Test - Resetowanie bazy danych po każdym teście](#resetting-the-database-after-each-test)
- [Writing Factories - Pisanie fabryk](#writing-factories)
    - [Factory States - Stany fabryki](#factory-states)
- [Using Factories - Korzystanie z fabryk](#using-factories)
    - [Creating Models - Tworzenie modeli](#creating-models)
    - [Persisting Models - Trwałe modele](#persisting-models)
    - [Relationships - Relacje](#relationships)
- [Available Assertions- Dostępne twierdzenia](#available-assertions)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel oferuje szereg pomocnych narzędzi, które ułatwiają testowanie aplikacji opartych na bazach danych. Po pierwsze, możesz użyć pomocnika `assertDatabaseHas`, aby potwierdzić, że dane istnieją w bazie danych pasującej do podanego zestawu kryteriów. Na przykład, jeśli chcesz sprawdzić, czy w tabeli `users` znajduje się rekord z wartością `sally@example.com` kolumny `email`, możesz wykonać następujące czynności:

    public function testDatabase()
    {
        // Make call to application...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }

Można również użyć helpera `assertDatabaseMissing`, aby stwierdzić, że dane nie istnieją w bazie danych.

Oczywiście, metoda `assertDatabaseHas` i inne podobne pomoce są dla wygody. Możesz dowolnie korzystać z wbudowanych metod asercji PHPUnit, aby uzupełnić swoje testy.

<a name="generating-factories"></a>
## Generating Factories - Generowanie fabryk

Aby utworzyć fabrykę, użyj `make:factory` [polecenie Artisan](/docs/{{version}}/artisan):

    php artisan make:factory PostFactory

Nowa fabryka zostanie umieszczona w twoim katalogu `database/factories`.

Opcja `--model` może być użyta do wskazania nazwy modelu stworzonego przez fabrykę. Ta opcja wstępnie wypełni wygenerowany plik fabryki danym modelem:

    php artisan make:factory PostFactory --model=Post

<a name="resetting-the-database-after-each-test"></a>
## Resetting The Database After Each Test - Resetowanie bazy danych po każdym teście

Często przydaje się resetowanie bazy danych po każdym teście, aby dane z poprzedniego testu nie kolidowały z kolejnymi testami. Właściwość `RefreshDatabase` przyjmuje najbardziej optymalne podejście do migracji bazy testowej w zależności od tego, czy korzystasz z bazy danych w pamięci czy z tradycyjnej bazy danych. Po prostu użyj cechy na swojej klasie testowej i wszystko będzie dla ciebie obsługiwane:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

<a name="writing-factories"></a>
## Writing Factories - Pisanie fabryk

Podczas testowania może zajść potrzeba wstawienia kilku rekordów do bazy danych przed wykonaniem testu. Zamiast ręcznie określać wartość każdej kolumny podczas tworzenia tych danych testowych, Laravel umożliwia zdefiniowanie domyślnego zestawu atrybutów dla każdego z [modeli wymownych](/docs/{{version}}/eloquent) przy użyciu modeli. Aby rozpocząć, spójrz na plik `database/factoryories/UserFactory.php` w swojej aplikacji. Po wyjęciu z pudełka ten plik zawiera jedną definicję fabryczną:

    use Faker\Generator as Faker;

    $factory->define(App\User::class, function (Faker $faker) {
        static $password;

        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => $password ?: $password = bcrypt('secret'),
            'remember_token' => str_random(10),
        ];
    });

W ramach Closure, które służy jako definicja fabryczna, możesz zwrócić domyślne wartości testowe wszystkich atrybutów w modelu. The Closure otrzyma instancję biblioteki PHP [Faker](https://github.com/fzaninotto/Faker), która pozwala wygodnie generować różne rodzaje losowych danych do testowania.

Możesz także utworzyć dodatkowe pliki fabryczne dla każdego modelu dla lepszej organizacji. Na przykład możesz utworzyć pliki `UserFactory.php` i `CommentFactory.php` w swoim katalogu `database/factories`. Wszystkie pliki w katalogu `factories` zostaną automatycznie załadowane przez Laravel.

<a name="factory-states"></a>
### Factory States - Stany fabryki

Stany umożliwiają zdefiniowanie dyskretnych modyfikacji, które można zastosować do fabryk modeli w dowolnej kombinacji. Na przykład twój model `User` może mieć stan` delinquent`, który modyfikuje jedną z jego domyślnych wartości atrybutów. Możesz zdefiniować transformacje stanu za pomocą metody `state`. W przypadku prostych stanów możesz przekazać tablicę modyfikacji atrybutów:

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);

Jeśli twój stan wymaga obliczenia lub instancji `$faker`, możesz użyć Closure, aby obliczyć modyfikacje atrybutów stanu:

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });

<a name="using-factories"></a>
## Using Factories - Korzystanie z fabryk

<a name="creating-models"></a>
### Creating Models - Tworzenie modeli

Po zdefiniowaniu swoich fabryk możesz użyć globalnej funkcji `factory` w swoich testach lub plikach źródłowych do wygenerowania instancji modelu. Spójrzmy więc na kilka przykładów tworzenia modeli. Najpierw użyjemy metody `make` do tworzenia modeli, ale nie zapisywania ich w bazie danych:

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // Use model in tests...
    }

Możesz także utworzyć kolekcję wielu modeli lub utworzyć modele danego typu:

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();

#### Applying States - Dołaczanie stanów

Możesz również zastosować dowolny ze swoich [stanów](#factory-states) do modeli. Aby zastosować do modelu wiele transformacji stanów, należy podać nazwę każdego stanu, który ma zostać zastosowany:

    $users = factory(App\User::class, 5)->states('delinquent')->make();

    $users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();

#### Overriding Attributes - Przesłanianie atrybutów

Jeśli chcesz zastąpić niektóre z domyślnych wartości modeli, możesz przekazać tablicę wartości do metody `make`. Tylko określone wartości zostaną zastąpione, a pozostałe wartości pozostaną ustawione na wartości domyślne określone przez producenta:

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);

<a name="persisting-models"></a>
### Persisting Models - Trwałe modele

Metoda `create` nie tylko tworzy instancje modelu, ale także zapisuje je w bazie danych przy użyciu metody `save` Eloquent:

    public function testDatabase()
    {
        // Create a single App\User instance...
        $user = factory(App\User::class)->create();

        // Create three App\User instances...
        $users = factory(App\User::class, 3)->create();

        // Use model in tests...
    }

Można nadpisywać atrybuty w modelu, przekazując tablicę do metody `create`:

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);

<a name="relationships"></a>
### Relationships - Relacje

W tym przykładzie dołączymy relację do niektórych utworzonych modeli. Używając metody  `create` do tworzenia wielu modeli, zwracana jest Eloquent [instancja kolekcji](/docs/{{version}}/eloquent-collections), co umożliwia korzystanie z wygodnych funkcji dostępnych w kolekcji, takich jak  `each`:

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### Relations & Attribute Closures - Relacje i atrybuty Closures

You may also attach relationships to models using Closure attributes in your factory definitions. For example, if you would like to create a new `User` instance when creating a `Post`, you may do the following:
Możesz także dołączać relacje do modeli przy użyciu atrybutów Closure w definicjach fabrycznych. Na przykład, jeśli chcesz utworzyć nową instancję `User` podczas tworzenia `Post`, możesz wykonać następujące czynności:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

Te Closures otrzymują również oszacowaną tablicę atrybutów fabryki, która je definiuje:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });

<a name="available-assertions"></a>
## Available Assertions - Dostępne twierdzenia

Laravel zapewnia kilka asercji bazy danych dla testów [PHPUnit](https://phpunit.de/):

Metoda  | Opis
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  Twierdzimy, że tabela w bazie danych zawiera dane.
`$this->assertDatabaseMissing($table, array $data);`  |  Twierdzimy, że tabela w bazie danych nie zawiera danych.
`$this->assertSoftDeleted($table, array $data);`  |  Twierdzimy, że dany rekord został delikatnie usunięty.
