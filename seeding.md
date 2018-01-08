# Database: Seeding

- [Introduction - Wprowadzenie](#introduction)
- [Writing Seeders - Pisanie siewników](#writing-seeders)
    - [Using Model Factories - Używanie fabryk modelu](#using-model-factories)
    - [Calling Additional Seeders - Wywoływanie dodatkowych siewników](#calling-additional-seeders)
- [Running Seeders - Uruchamianie siewników](#running-seeders)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel zawiera prostą metodę wysiewu bazy danych z danymi testowymi za pomocą klas początkowych. Wszystkie klasy seed są przechowywane w katalogu `database/seeds`. Klasy seed mogą mieć dowolną nazwę, ale prawdopodobnie powinny być zgodne z pewną rozsądną konwencją, taką jak `UsersTableSeeder` itp. Domyślnie zdefiniowana jest dla ciebie klasa `DatabaseSeeder`. Z tej klasy możesz używać metody `call` do uruchamiania innych klas początkowych, co pozwala kontrolować kolejność inicjowania.

<a name="writing-seeders"></a>
## Writing Seeders - Pisanie siewników

Aby wygenerować siewnik, wykonaj polecenie `make:seeder` [polecenie Artisan](/docs/{{version}}/artisan). Wszystkie seedery wygenerowane przez framework zostaną umieszczone w katalogu `database/seeds`:

    php artisan make:seeder UsersTableSeeder

Klasa seeder zawiera domyślnie tylko jedną metodę: `run`. Ta metoda jest wywoływana, gdy wykonywane jest `db:seed` [polecenie Artisan](/docs/{{version}}/artisan). W metodzie `run` możesz wstawiać dane do swojej bazy danych. Możesz użyć [konstruktora zapytań](/docs/{{version}}/queries), aby ręcznie wstawić dane, lub możesz użyć [fabryki modeli Eloquen](/docs/{{version}}/database-testing#writing-factories).

> {tip} [Ochrona przypisania masowego](/docs/{{version}}/eloquent#mass-assignment) jest automatycznie wyłączany podczas inicjalizacji bazy danych.

Jako przykład, zmodyfikujmy domyślną klasę `DatabaseSeeder` i dodaj instrukcję wstawiania bazy danych do metody `run`:

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeds.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### Using Model Factories - Używanie fabryk modelu

Oczywiście ręczne określanie atrybutów dla każdego modelu jest nieporęczne. Zamiast tego możesz użyć [fabryki modeli](/docs/{{version}}/database-testing#writing-factories), aby wygodnie generować duże ilości rekordów bazy danych. Najpierw przejrzyj [dokumentację fabryczną modelu](/docs/{{version}}/database-testing#writing-factories), aby dowiedzieć się jak zdefiniować swoje fabryki. Po zdefiniowaniu swoich fabryk możesz użyć funkcji `factory`, aby wstawić rekordy do bazy danych.

Na przykład utwórzmy 50 użytkowników i dołącz do każdego użytkownika relację:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function ($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### Calling Additional Seeders - Wywoływanie dodatkowych siewników

W klasie `DatabaseSeeder` możesz użyć metody `call`, aby wykonać dodatkowe klasy seed. Użycie metody `call` umożliwia rozbicie bazy danych na wiele plików, dzięki czemu żadna pojedyncza klasa siewnika nie stanie się zbyt duża. Po prostu podaj nazwę klasy siewnika, którą chcesz uruchomić:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UsersTableSeeder::class,
            PostsTableSeeder::class,
            CommentsTableSeeder::class,
        ]);
    }

<a name="running-seeders"></a>
## Running Seeders - Uruchamianie siewników

Po napisaniu swojego siewnika może zajść potrzeba regeneracji autoloadera Composer za pomocą polecenia `dump-autoload`:

    composer dump-autoload

Teraz możesz użyć polecenia `db:seed` Artisan do zarzucenia bazy danych. Domyślnie komenda `db:seed` uruchamia klasę `DatabaseSeeder`, która może być używana do wywoływania innych klas początkowych. Możesz jednak użyć opcji `--class`, aby określić konkretną klasę siewnika do indywidualnego uruchamiania:

    php artisan db:seed

    php artisan db:seed --class=UsersTableSeeder

Możesz również zaszczepić swoją bazę danych za pomocą komendy `migrate:refresh`, która również wycofa i ponownie uruchom wszystkie migracje. To polecenie jest przydatne do całkowitego ponownego utworzenia bazy danych:

    php artisan migrate:refresh --seed
