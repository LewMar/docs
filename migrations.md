# Database: Migrations

- [Introduction - Wprowadzenie](#introduction)
- [Generating Migrations - Generowanie migracji](#generating-migrations)
- [Migration Structure - Struktura migracji](#migration-structure)
- [Running Migrations - Uruchamianie migracji](#running-migrations)
    - [Rolling Back Migrations - Wycofywanie migracji](#rolling-back-migrations)
- [Tables - Tabele](#tables)
    - [Creating Tables - Tworzenie tabel](#creating-tables)
    - [Renaming / Dropping Tables - Zmiana nazwy / usunięcie tabel](#renaming-and-dropping-tables)
- [Columns - Kolumny](#columns)
    - [Creating Columns - Tworzenie kolumn](#creating-columns)
    - [Column Modifiers - Modyfikatory kolumny](#column-modifiers)
    - [Modifying Columns - Modyfikowanie kolumn](#modifying-columns)
    - [Dropping Columns - Usówanie kolumn](#dropping-columns)
- [Indexes - Indeksy](#indexes)
    - [Creating Indexes - Tworzenie Indeksów](#creating-indexes)
    - [Dropping Indexes - Usuń Indeksy](#dropping-indexes)
    - [Foreign Key Constraints - Ograniczenie klucza obcego](#foreign-key-constraints)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Migracje są jak kontrola wersji dla Twojej bazy danych, dzięki czemu Twój zespół może łatwo modyfikować i udostępniać schemat bazy danych aplikacji. Migracje są zazwyczaj sparowane z narzędziem do tworzenia schematów Laravel, aby łatwo budować schemat bazy danych aplikacji. Jeśli kiedykolwiek musiałeś powiedzieć członkowi drużyny, aby ręcznie dodać kolumnę do swojego lokalnego schematu bazy danych, napotkasz problem, który rozwiązuje migracja baz danych.

The Laravel `Schema` [fasada](/docs/{{version}}/facades) zapewnia agnostyczną obsługę baz danych przy tworzeniu i operowaniu tabelami we wszystkich obsługiwanych systemach baz danych Laravel.

<a name="generating-migrations"></a>
## Generating Migrations - Generowanie migracji

Aby utworzyć migrację, użyj polecenia `make:migration` [polecenie Artisan](/docs/{{version}}/artisan)):

    php artisan make:migration create_users_table

Nowa migracja zostanie umieszczona w katalogu `database/migrations`. Każda nazwa pliku migracji zawiera znacznik czasu, który pozwala Laravel-owi ustalić kolejność migracji.

Opcje `--table` i `--create` mogą również służyć do wskazywania nazwy tabeli i tego, czy migracja spowoduje utworzenie nowej tabeli. Te opcje wstępnie wypełniają wygenerowany plik pośredniczący migracji określoną tabelą:

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

Jeśli chcesz określić niestandardową ścieżkę wyjściową dla wygenerowanej migracji, możesz użyć opcji `--path` podczas wykonywania komendy `make:migration`. Podana ścieżka powinna być względna względem ścieżki podstawowej aplikacji.

<a name="migration-structure"></a>
## Migration Structure - Struktura migracji

Klasa migracji zawiera dwie metody: `up` i `down`. Metoda `up` służy do dodawania nowych tabel, kolumn lub indeksów do bazy danych, natomiast metoda `down` powinna odwracać operacje wykonywane metodą `up`.

W obu tych metodach można użyć kreatora schematów Laravel do ekspresywnego tworzenia i modyfikowania tabel. Aby zapoznać się ze wszystkimi metodami dostępnymi w programie budującym `Schema`, [sprawdź jego dokumentację](#creating-tables). Na przykład ten przykład migracji tworzy tabelę "loty"(flights):

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }

<a name="running-migrations"></a>
## Running Migrations - Uruchamianie migracji

Aby uruchomić wszystkie zaległe migracje, wykonaj polecenie `migrate` Artisan:

    php artisan migrate

> {note} Jeśli używasz [maszyny wirtualnej Homestead](/docs/{{version}}/homestead), powinieneś uruchomić to polecenie z poziomu maszyny wirtualnej.

#### Forcing Migrations To Run In Production - Zmuszanie migracji do uruchamiania w produkcji

Niektóre operacje migracyjne są destrukcyjne, co oznacza, że mogą spowodować utratę danych. Aby zabezpieczyć użytkownika przed uruchomieniem tych poleceń w bazie danych produkcyjnych, użytkownik zostanie poproszony o potwierdzenie przed wykonaniem poleceń. Aby wymusić wykonywanie poleceń bez pytania, użyj flagi `--force`:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### Rolling Back Migrations - Wycofywanie migracji

Aby przywrócić ostatnią operację migracji, możesz użyć komendy `rollback`. To polecenie wycofuje ostatnią "partię" migracji, która może zawierać wiele plików migracji:

    php artisan migrate:rollback

Możesz wycofać ograniczoną liczbę migracji, zapewniając opcję `step` dla polecenia `rollback`. Na przykład następujące polecenie spowoduje wycofanie ostatnich pięciu migracji:

    php artisan migrate:rollback --step=5

Komenda `migrate:reset` cofnie wszystkie migracje aplikacji:

    php artisan migrate:reset

#### Rollback & Migrate In Single Command - Cofnij i przenieś w pojedynczej komendzie

Komenda `migrate:refresh` cofnie wszystkie twoje migracje, a następnie uruchom komendę `migrate`. To polecenie skutecznie odtwarza całą twoją bazę danych:

    php artisan migrate:refresh

    // Refresh the database and run all database seeds... - Odśwież bazę danych i uruchom wszystkie nasiona bazy danych ...
    php artisan migrate:refresh --seed

Możesz wycofać i ponownie zmigrować ograniczoną liczbę migracji, zapewniając opcję `step` dla polecenia `refresh`. Na przykład poniższe polecenie spowoduje wycofanie i ponowną migrację ostatnich pięciu migracji:

    php artisan migrate:refresh --step=5

#### Drop All Tables & Migrate - Usuń wszystkie tabele i przeprowadź migrację

Komenda `migrate:fresh` usunie wszystkie tabele z bazy danych, a następnie uruchom komendę `migrate`:

    php artisan migrate:fresh

    php artisan migrate:fresh --seed

<a name="tables"></a>
## Tables - Tabele

<a name="creating-tables"></a>
### Creating Tables - Tworzenie tabel

Aby utworzyć nową tabelę bazy danych, użyj metody `create` na elewacji `Schema`. Metoda `create` przyjmuje dwa argumenty. Pierwsza to nazwa tabeli, a druga to `Closure`, które otrzymuje obiekt `Blueprint`, który może być użyty do zdefiniowania nowej tabeli:

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Oczywiście podczas tworzenia tabeli można użyć dowolnej z [metod kolumnowych](#creating-columns), aby zdefiniować kolumny tabeli.

#### Checking For Table / Column Existence - Sprawdzanie istnienia tabeli / kolumny

Możesz łatwo sprawdzić, czy istnieje tabela lub kolumna za pomocą metod `hasTable` i `hasColumn`:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### Database Connection & Table Options - Połączenie z bazą danych i opcje tabel

Jeśli chcesz wykonać operację schematu na połączeniu z bazą danych, która nie jest domyślnym połączeniem, użyj metody `connection`:

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Możesz użyć następujących poleceń w konstruktorze schematu, aby zdefiniować opcje tabeli:

Komenda  |  Opis
-------  |  -----------
`$table->engine = 'InnoDB';`  |  Określ mechanizm przechowywania tabeli (MySQL).
`$table->charset = 'utf8';`  |  Określ domyślny zestaw znaków dla tabeli (MySQL).
`$table->collation = 'utf8_unicode_ci';`  |  Określ domyślne sortowanie dla tabeli (MySQL).
`$table->temporary();`  |  Utwórz tabelę tymczasową (z wyjątkiem SQL Server).

<a name="renaming-and-dropping-tables"></a>
### Renaming / Dropping Tables - Zmiana nazwy / usunięcie tabel

Aby zmienić nazwę istniejącej tabeli bazy danych, użyj metody `rename`:

    Schema::rename($from, $to);

Aby upuścić istniejącą tabelę, możesz użyć metod `drop` lub` dropIfExists`:

    Schema::drop('users');

    Schema::dropIfExists('users');

#### Renaming Tables With Foreign Keys - Zmiana nazwy tabel za pomocą kluczy obcych

Przed zmianą nazwy tabeli należy sprawdzić, czy wszelkie ograniczenia klucza obcego w tabeli mają jawnie określoną nazwę w plikach migracyjnych, zamiast pozwalać Laravel przypisywać nazwę opartą na konwencji. W przeciwnym razie nazwa ograniczenia klucza obcego będzie odnosić się do starej nazwy tabeli.

<a name="columns"></a>
## Columns - Kolumny

<a name="creating-columns"></a>
### Creating Columns - Tworzenie kolumn

Metodę `table` na elewacji `Schema` można użyć do aktualizacji istniejących tabel. Podobnie jak w przypadku metody `create`, metoda `table` przyjmuje dwa argumenty: nazwę tabeli i `Closure`, która otrzymuje instancję `Blueprint`, której możesz użyć do dodania kolumn do tabeli:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

#### Available Column Types - Dostępne typy kolumn

Oczywiście program do tworzenia schematów zawiera różne typy kolumn, które można określić podczas budowania tabel:

Komenda  |  Opis
-------  |  -----------
`$table->bigIncrements('id');`  |  Auto-inkrementująca kolumnę równoważną UNSIGNED BIGINT (klucz podstawowy).
`$table->bigInteger('votes');`  |  Kolumna równoważna BIGINT.
`$table->binary('data');`  |   Kolumna równoważna BLOB.
`$table->boolean('confirmed');`  |  Kolumna równoważna BOOLEAN.
`$table->char('name', 100);`  |  Kolumna równoważna CHAR z opcjonalną długością.
`$table->date('created_at');`  |  Kolumna równoważna DATE.
`$table->dateTime('created_at');`  | Kolumna równoważna DATETIME.
`$table->dateTimeTz('created_at');`  |  Kolumna równoważna DATETIME (z strefą czasową).
`$table->decimal('amount', 8, 2);`  |  Kolumna równoważna DECIMAL z dokładnością (całkowita liczba) i skalą (cyfry dziesiętne).
`$table->double('amount', 8, 2);`  |  Kolumna równoważna DOUBLE z dokładnością (całkowita liczba) i skalą (cyfry dziesiętne).
`$table->enum('level', ['easy', 'hard']);`  |  Kolumna równoważna ENUM.
`$table->float('amount', 8, 2);`  |  Kolumna równoważna FLOAT z dokładnością (całkowita liczba) i skalą (cyfry dziesiętne).
`$table->geometry('positions');`  |  Kolumna równoważna GEOMETRIA.
`$table->geometryCollection('positions');`  | Kolumna równoważna GEOMETRYCOLLECTION.
`$table->increments('id');`  |  Automatycznie zwiększająca się kolumna równoważna  UNSIGNED INTEGER (klucz podstawowy).
`$table->integer('votes');`  |  Kolumna równoważna INTEGER.
`$table->ipAddress('visitor');`  |  Kolumna równoważna adresowi IP.
`$table->json('options');`  |  Kolumna równoważna JSON.
`$table->jsonb('options');`  |  Kolumna równoważna JSONB.
`$table->lineString('positions');`  |  Kolumna równoważna LINESTRING.
`$table->longText('description');`  |  Kolumna równoważna LONGTEXT.
`$table->macAddress('device');`  |  Kolumna równoważna adresu MAC.
`$table->mediumIncrements('id');`  |  Auto-inkrementacja kolumny UNSIGNED MEDIUMINT (klucz podstawowy).
`$table->mediumInteger('votes');`  |  Kolumna równoważna MEDIUMINT.
`$table->mediumText('description');`  |  Kolumna równoważna MEDIUMTEXT.
`$table->morphs('taggable');`  |  Dodaje kolumny `taggable_id` UNSIGNED INTEGER i `taggable_type` VARCHAR.
`$table->multiLineString('positions');`  |  Kolumna równoważna MULTILINESTRING.
`$table->multiPoint('positions');`  |  Kolumna równoważna MULTIPOINT.
`$table->multiPolygon('positions');`  |  Kolumna równoważna MULTIPOLYGON.
`$table->nullableMorphs('taggable');`  |  Dodaje wersje z nullable kolumn `morphs()`.
`$table->nullableTimestamps();`  |  Alias metody `timestamps ()`.
`$table->point('position');`  |  Kolumna równoważna POINT.
`$table->polygon('positions');`  | Odpowiednik kolumny POLYGON.
`$table->rememberToken();`  |  Dodaje nullable kolumnę równoważną `remember_token` VARCHAR (100).
`$table->smallIncrements('id');`  |  Auto-inkrementacja kolumny UNSIGNED SMALLINT (klucz podstawowy).
`$table->smallInteger('votes');`  |  Kolumna równoważna SMALLINT.
`$table->softDeletes();`  |  Dodaje równoważną kolumnę nullable `deleted_at` TIMESTAMP dla miękkich operacji usuwania.
`$table->softDeletesTz();`  |  Dodaje nullable `deleted_at` TIMESTAMP (z strefą czasową) odpowiednik dla miękkich operacji usuwania.
`$table->string('name', 100);`  |  Kolumna równoważna VARCHAR z opcjonalną długością.
`$table->text('description');`  |  Kolumna równoważna TEXT.
`$table->time('sunrise');`  |  Kolumna równoważna TIME.
`$table->timeTz('sunrise');`  |  Kolumna równoważna TIME (z strefą czasową).
`$table->timestamp('added_on');`  |  Kolumna równoważna TIMESTAMP.
`$table->timestampTz('added_on');`  |  Kolumna równoważna TIMESTAMP (z strefą czasową).
`$table->timestamps();`  |  Dodaje Nullable kolumny `created_at` i  `updated_at` TIMESTAMP.
`$table->timestampsTz();`  |  Dodaje wartości nullable kolumny `created_at` i` updated_at` TIMESTAMP (z timezone).
`$table->tinyIncrements('id');`  |  Auto-inkrementacja kolumny UNSIGNED TINYINT (klucz podstawowy).
`$table->tinyInteger('votes');`  |  Kolumna równoważna TINYINT.
`$table->unsignedBigInteger('votes');`  |  Kolumna równoważna UNSIGNED BIGINT.
`$table->unsignedDecimal('amount', 8, 2);`  |  Kolumna równoważna UNSIGNED DECIMAL z dokładnością (całkowita liczba) i skalą (cyfry dziesiętne).
`$table->unsignedInteger('votes');`  |  Kolumna równoważna UNSIGNED INTEGER.
`$table->unsignedMediumInteger('votes');`  |  Kolumna równoważna UNSIGNED MEDIUMINT
`$table->unsignedSmallInteger('votes');`  | Kolumna równoważna UNSIGNED SMALLINT.
`$table->unsignedTinyInteger('votes');`  |  Kolumna równoważna UNSIGNED TINYINT.
`$table->uuid('id');`  |  Kolumna równoważna UUID.
`$table->year('birth_year');`  |  Kolumna równoważna YEAR.

<a name="column-modifiers"></a>
### Column Modifiers - Modyfikatory kolumny

Oprócz wymienionych powyżej typów kolumn, istnieje kilka "modyfikatorów" kolumn, których możesz użyć podczas dodawania kolumny do tabeli bazy danych. Na przykład, aby kolumna "nullable", możesz użyć metody `nullable`:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

Poniżej znajduje się lista wszystkich dostępnych modyfikatorów kolumn. Ta lista nie zawiera [modyfikatorów indeksu](#creating-indexes):

Modifikator  |  Opis
--------  |  -----------
`->after('column')`  |  Umieść kolumnę "po" innej kolumnie (MySQL)
`->autoIncrement()`  |  Ustaw kolumny INTEGER jako automatyczne inkrementowanie (klucz podstawowy)
`->charset('utf8')`  |  Określ zestaw znaków dla kolumny (MySQL)
`->collation('utf8_unicode_ci')`  |  Określ sortowanie dla kolumny (MySQL / SQL Server)
`->comment('my comment')`  |  Dodaj komentarz do kolumny (MySQL)
`->default($value)`  |  Określ "domyślną" wartość dla kolumny
`->first()`  |  Umieść kolumnę "pierwszy" w tabeli (MySQL)
`->nullable($value = true)`  |  Pozwala (domyślnie) na wstawianie wartości NULL do kolumny
`->storedAs($expression)`  |  Utwórz zapisaną wygenerowaną kolumnę (MySQL)
`->unsigned()`  |  Ustaw kolumny INTEGER jako UNSIGNED (MySQL)
`->useCurrent()`  |  Ustaw TIMESTAMP kolumny, by używać CURRENT_TIMESTAMP jako wartości domyślnej
`->virtualAs($expression)`  |  Utwórz wirtualnie wygenerowaną kolumnę (MySQL)

<a name="modifying-columns"></a>
### Modifying Columns - Modyfikowanie kolumn

#### Prerequisites - Wymagania wstępne

Przed zmodyfikowaniem kolumny pamiętaj o dodaniu zależności `doctrine/dbal` do pliku `composer.json`. Biblioteka Doctrine DBAL służy do określania bieżącego stanu kolumny i tworzenia zapytań SQL potrzebnych do wprowadzenia określonych poprawek do kolumny:

    composer require doctrine/dbal

#### Updating Column Attributes - Aktualizowanie atrybutów kolumn

Metoda `change` pozwala modyfikować niektóre istniejące typy kolumn na nowy typ lub modyfikować atrybuty kolumny. Na przykład możesz zwiększyć rozmiar kolumny z ciągami znaków. Aby zobaczyć metodę `change` w akcji, zwiększmy rozmiar kolumny `name` z 25 na 50:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

Możemy również zmodyfikować kolumnę do unieważnienia:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} Tylko następujące typy kolumn można "zmienić": bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json, longText, mediumText, smallInteger, string, text, time, unsignedBigInteger, unsignedInteger and unsignedSmallInteger.

#### Renaming Columns - Zmiana nazwy kolumn

Aby zmienić nazwę kolumny, możesz użyć metody `renameColumn` w Konstruktorze schematów. Przed zmianą nazwy kolumny pamiętaj o dodaniu zależności `doctrine/dbal` do pliku `composer.json`:

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {note} Zmiana nazwy dowolnej kolumny w tabeli, która ma również kolumnę typu `enum`, nie jest obecnie obsługiwana.

<a name="dropping-columns"></a>
### Dropping Columns - Usówanie kolumn

Aby usunąć kolumnę, użyj metody `dropColumn` w Konstruktorze schematów. Przed usunięciem kolumn z bazy danych SQLite trzeba dodać zależność `doctrine/dbal` do pliku `composer.json` i uruchomić komendę `composer update` w terminalu, aby zainstalować bibliotekę:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

Możesz upuścić wiele kolumn z tabeli, przekazując tablicę nazw kolumn do metody `dropColumn`:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} Usuwanie lub modyfikowanie wielu kolumn w ramach jednej migracji podczas korzystania z bazy danych SQLite nie jest obsługiwane.

#### Available Command Aliases - Dostępne aliasy poleceń

Komenda  |  Opis
-------  |  -----------
`$table->dropRememberToken();`  |  Usuń kolumnę `remember_token`.
`$table->dropSoftDeletes();`  |  Usuń kolumnę `deleted_at`
`$table->dropSoftDeletesTz();`  |  Alias metody `dropSoftDeletes()`.
`$table->dropTimestamps();`  |  Usuń kolumnę `created_at` i `updated_at`.
`$table->dropTimestampsTz();` |  Alias metody `dropTimestamps()` method.

<a name="indexes"></a>
## Indexes - Indeksy

<a name="creating-indexes"></a>
### Creating Indexes - Tworzenie Indeksów

Kreator schematów obsługuje kilka typów indeksów. Najpierw spójrzmy na przykład, który określa wartości kolumn, które powinny być unikalne. Aby utworzyć indeks, możemy powiązać metodę `unique` z definicją kolumny:

    $table->string('email')->unique();

Alternatywnie możesz utworzyć indeks po zdefiniowaniu kolumny. Na przykład:

    $table->unique('email');

Możesz nawet przekazać tablicę kolumn do metody indeksu, aby utworzyć indeks dwuczłonowy (lub złożony):

    $table->index(['account_id', 'created_at']);

Laravel automatycznie wygeneruje uzasadnioną nazwę indeksu, ale możesz przekazać drugi argument do metody, aby samemu podać nazwę:

    $table->unique('email', 'unique_email');

#### Available Index Types - Dostępne typy indeksów

Komenda  |  Opis
-------  |  -----------
`$table->primary('id');`  |  Dodaje klucz podstawowy.
`$table->primary(['id', 'parent_id']);`  |  Dodaje klucze złożone.
`$table->unique('email');`  |  Dodaje unikalny indeks.
`$table->index('state');`  |  Dodaje zwykły indeks.
`$table->spatialIndex('location');`  |  Dodaje indeks przestrzenny. (oprócz SQLite)

#### Index Lengths & MySQL / MariaDB - Długości indeksów i MySQL / MariaDB

Laravel domyślnie używa zestawu znaków `utf8mb4`, który obejmuje obsługę przechowywania "emojis" w bazie danych. Jeśli używasz wersji MySQL starszej niż wersja 5.7.7 lub MariaDB starszej niż wersja 10.2.2, możesz ręcznie skonfigurować domyślną długość łańcucha generowaną przez migracje, aby MySQL mógł utworzyć dla nich indeksy. Możesz to skonfigurować, wywołując metodę `Schema::defaultStringLength` w swoim `AppServiceProvider`:

    use Illuminate\Support\Facades\Schema;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

Alternatywnie możesz włączyć opcję `innodb_large_prefix` dla swojej bazy danych. Zapoznaj się z dokumentacją swojej bazy danych, aby uzyskać instrukcje, jak prawidłowo włączyć tę opcję.

<a name="dropping-indexes"></a>
### Dropping Indexes - Usuń Indeksy

Aby usunąć indeks, musisz podać jego nazwę. Domyślnie Laravel automatycznie przypisuje do indeksów sensowną nazwę. Połącz nazwę tabeli, nazwę indeksowanej kolumny i typ indeksu. Oto kilka przykładów:

Komendy  |  Opis
-------  |  -----------
`$table->dropPrimary('users_id_primary');`  |  Usuń klucz podstawowy z tabeli "Użytkownicy".
`$table->dropUnique('users_email_unique');`  |  Usuń unikalny indeks z tabeli "Użytkownicy".
`$table->dropIndex('geo_state_index');`  |  Usuń podstawowy indeks z tabeli "geo".
`$table->dropSpatialIndex('geo_location_spatialindex');`  |  Usuń indeks przestrzenny z tabeli "geo" (z wyjątkiem SQLite).

Jeśli przekażesz tablicę kolumn do metody, która zrzuci indeksy, tradycyjna nazwa indeksu zostanie wygenerowana na podstawie nazwy tabeli, kolumn i typu klucza:

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### Foreign Key Constraints - Ograniczenie klucza obcego

Laravel zapewnia również obsługę tworzenia ograniczeń klucza obcego, które są używane do wymuszania integralności referencyjnej na poziomie bazy danych. Na przykład, zdefiniujmy kolumnę `user_id` w tabeli `posts`, która odwołuje się do kolumny `id` w tabeli` users`:

    Schema::table('posts', function (Blueprint $table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

Możesz również określić żądaną akcję dla właściwości "przy usuwaniu" i "przy aktualizacji" ograniczenia:

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

Aby usunąć klucz obcy, możesz użyć metody `dropForeign`. Ograniczenia klucza obcego używają tej samej konwencji nazewnictwa co indeksy. Tak więc połączymy nazwę tabeli i kolumny w ograniczeniu, a następnie nadamy jej nazwę przez "_foreign":

    $table->dropForeign('posts_user_id_foreign');

Możesz też przekazać wartość tablicy, która automatycznie użyje tradycyjnej nazwy ograniczenia podczas usuwania:

    $table->dropForeign(['user_id']);

Można włączać i wyłączać więzy klucza obcego w ramach migracji za pomocą następujących metod:

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();
