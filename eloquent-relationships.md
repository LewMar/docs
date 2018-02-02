# Eloquent: Relationships

- [Introduction - Wprowadzenie](#introduction)
- [Defining Relationships - Definiowanie relacji](#defining-relationships)
    - [One To One - Jeden do jednego](#one-to-one)
    - [One To Many - Jeden do wielu](#one-to-many)
    - [One To Many (Inverse)](#one-to-many-inverse)
    - [Many To Many - Wiele do wielu](#many-to-many)
    - [Has Many Through - Ma wielu przez](#has-many-through)
    - [Polymorphic Relations - Relacje polimorficzne](#polymorphic-relations)
    - [Many To Many Polymorphic Relations - Wiele do wielu polimorficznych relacji](#many-to-many-polymorphic-relations)
- [Querying Relations - Zapytanie o relacje](#querying-relations)
    - [Relationship Methods Vs. Dynamic Properties - Metody relacji kontra Właściwości dynamiczne](#relationship-methods-vs-dynamic-properties)
    - [Querying Relationship Existence - Zapytanie o istnienie relacji](#querying-relationship-existence)
    - [Querying Relationship Absence - Zapytanie o nieobecność w relacji](#querying-relationship-absence)
    - [Counting Related Models - Zliczanie powiązanych modeli](#counting-related-models)
- [Eager Loading - Chetne wczytywanie](#eager-loading)
    - [Constraining Eager Loads - Ograniczanie chętnych obciążeń](#constraining-eager-loads)
    - [Lazy Eager Loading - Leniwe chciwe ładowanie](#lazy-eager-loading)
- [Inserting & Updating Related Models - Wstawianie i aktualizowanie powiązanych modeli](#inserting-and-updating-related-models)
    - [The `save` Method - Metoda zapisu](#the-save-method)
    - [The `create` Method - Metoda Stwórz](#the-create-method)
    - [Belongs To Relationships - Należy do relacji](#updating-belongs-to-relationships)
    - [Many To Many Relationships - Wiele do wielu relacji](#updating-many-to-many-relationships)
- [Touching Parent Timestamps - Dotykanie znaczników czasu dla rodziców](#touching-parent-timestamps)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Tabele bazy danych często są ze sobą powiązane. Na przykład wpis w blogu może zawierać wiele komentarzy lub zamówienie może być powiązane z użytkownikiem, który go umieścił. Eloquent ułatwia zarządzanie i pracę z tymi relacjami i obsługuje kilka różnych typów relacji:

- [One To One - Jeden do jednego](#one-to-one)
- [One To Many - Jeden do wielu](#one-to-many)
- [Many To Many - Wiele do wielu](#many-to-many)
- [Has Many Through - Ma wielu przez](#has-many-through)
- [Polymorphic Relations - Relacje polimorficzne](#polymorphic-relations)
- [Many To Many Polymorphic Relations - Wiele do wielu polimorficznych relacji](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## Defining Relationships - Definiowanie relacji

Wymowne relacje są definiowane jako metody na klasach modelu Eloquent. Ponieważ, podobnie jak same modele Eloquent, relacje służą również jako potężne [konstruktory zapytań](/docs/{{version}}/queries), definiując relacje, metody zapewniają potężne metody łączenia łańcuchów i wyszukiwania zapytań. Na przykład możemy łączyć dodatkowe ograniczenia w relacji `posts`:

    $user->posts()->where('active', 1)->get();

Ale zanim zanurkujemy zbyt głęboko w relacje, nauczmy się definiować każdy typ.

<a name="one-to-one"></a>
### One To One - Jeden do jednego

Relacja jeden-do-jednego jest bardzo podstawową relacją. Na przykład model `User` może być powiązany z jednym  `Phone`. Aby zdefiniować tę relację, umieszczamy metodę `phone` w modelu `User`. Metoda `phone` powinna wywołać metodę `hasOne` i zwrócić jej wynik:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

Pierwszym argumentem przekazanym do metody `hasOne` jest nazwa powiązanego modelu. Po zdefiniowaniu relacji możemy pobrać powiązany rekord przy użyciu właściwości dynamicznych Eloquent. Właściwości dynamiczne umożliwiają dostęp do metod relacji tak, jakby były właściwościami zdefiniowanymi w modelu:

    $phone = User::find(1)->phone;

Eonquent określa klucz obcy relacji na podstawie nazwy modelu. W tym przypadku model `Phone` automatycznie zakłada, że ma klucz obcy "user_id". Jeśli chcesz zastąpić tę konwencję, możesz przekazać drugi argument do metody `hasOne`:

    return $this->hasOne('App\Phone', 'foreign_key');

Dodatkowo, Eloquent zakłada, że klucz obcy powinien mieć wartość zgodną z kolumną `id` (lub niestandardową `$primaryKey`) jednostki nadrzędnej. Innymi słowy, Eloquent będzie szukał wartości kolumny `id` użytkownika w kolumnie `user_id` w rekordzie `Phone`. Jeśli chcesz, aby relacja wykorzystywała wartość inną niż `id`, możesz przekazać trzeci argument do metody `hasOne` określającej twój klucz niestandardowy:

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### Defining The Inverse Of The Relationship - Definiowanie odwrotności relacji

Tak więc, możemy uzyskać dostęp do modelu `Phone` z naszego `User`. Teraz, definiujmy relację w modelu `Phone`, który pozwoli nam uzyskać dostęp do `User`, który jest właścicielem telefonu. Możemy zdefiniować odwrotność relacji `hasOne` za pomocą metody `belongsTo`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

W powyższym przykładzie, Eloquent spróbuje dopasować `user_id` z modelu `Phone` do `id` w modelu `User`. Eonquent określa domyślną nazwę klucza obcego, sprawdzając nazwę metody relacji i dodając nazwę metody za pomocą `_id`. Jeśli jednak klucz obcy w modelu `Phone` nie jest identyfikatorem `user_id`, można przekazać nazwę klucza niestandardowego jako drugi argument metody `belongsTo`:


    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

Jeśli twój model macierzysty nie używa `id` jako klucza głównego lub chcesz dołączyć do modelu potomnego do innej kolumny, możesz przekazać trzeci argument do metody `belongsTo` określającej niestandardowy klucz twojej tabeli nadrzędnej:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="default-models"></a>
#### Default Models - Domyślne modele

Relacja `belongsTo` pozwala zdefiniować model domyślny, który zostanie zwrócony, jeśli podana relacja ma wartość `null`. Ten wzorzec jest często określany jako [wzorzec obiektu zerowego](https://en.wikipedia.org/wiki/Null_Object_pattern) i może pomóc w usunięciu warunkowych kontroli w kodzie. W poniższym przykładzie relacja `user` zwróci pusty model `App\User`, jeśli do tego posta nie zostanie dołączony `user`:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

Aby zapełnić domyślny model atrybutami, możesz przekazać tablicę lub Closure do metody `withDefault`:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = 'Guest Author';
        });
    }

<a name="one-to-many"></a>
### One To Many - Jeden do wielu

Relacja "jeden do wielu" służy do definiowania relacji, w których pojedynczy model jest właścicielem dowolnej liczby innych modeli. Na przykład wpis na blogu może mieć nieskończoną liczbę komentarzy. Podobnie jak wszystkie inne Wymowne relacje, relacje jeden-do-wielu definiowane są przez umieszczenie funkcji w Modelu Wymownym:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

Pamiętaj, że Eloquent automatycznie określi właściwą kolumnę klucza obcego w modelu `Comment`. Zgodnie z konwencją, Eloquent przyjmie nazwę "snake case" nazwy posiadanego modelu i doda ją do `_id`. Tak więc dla tego przykładu, Eloquent zakłada, że klucz obcy w modelu `Comment` to `post_id`.

Po zdefiniowaniu relacji możemy uzyskać dostęp do kolekcji komentarzy, uzyskując dostęp do właściwości `comments`. Pamiętaj, że ponieważ Eloquent zapewnia "właściwości dynamiczne", możemy uzyskać dostęp do metod relacji tak, jakby zostały zdefiniowane jako właściwości w modelu:

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

Oczywiście, ponieważ wszystkie relacje służą również jako budownicze zapytań, możesz dodać kolejne więzy, do których są pobierane komentarze, wywołując metodę `comments` i kontynuując warunki łańcucha na zapytanie:

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

Podobnie jak w przypadku metody `hasOne`, możesz również zastąpić klucze obce i lokalne, przekazując dodatkowe argumenty do metody` hasMany`:

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### One To Many (Inverse) - Jeden do wielu (Odwrotność)

Teraz, gdy mamy dostęp do wszystkich komentarzy do wpisu, określmy relację, aby umożliwić dostęp do jej postu nadrzędnego. Aby zdefiniować odwrotność relacji `hasMany`, zdefiniuj funkcję relacji w modelu podrzędnym, która wywołuje metodę `belongsTo`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Po zdefiniowaniu relacji możemy odzyskać model `Post` dla `Comment`, uzyskując dostęp do "właściwości dynamicznej" `post`:

    $comment = App\Comment::find(1);

    echo $comment->post->title;

W powyższym przykładzie, Eloquent spróbuje dopasować `post_id` z modelu `Comment` do `id` w modelu `Post`. Eonquent określa domyślną nazwę klucza obcego, sprawdzając nazwę metody relacji i dodając nazwę metody za pomocą `_id`. Jeśli jednak klucz obcy w modelu `Comment` nie jest `post_id`, możesz przekazać nazwę klucza niestandardowego jako drugi argument metody `belongsTo`:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

Jeśli twój model macierzysty nie używa `id` jako klucza głównego lub chcesz dołączyć do modelu potomnego do innej kolumny, możesz przekazać trzeci argument do metody `belongsTo` określającej niestandardowy klucz twojej tabeli nadrzędnej:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### Many To Many - Wiele do wielu

Relacje wiele do wielu są nieco bardziej skomplikowane niż relacje `hasOne` i` hasMany`. Przykładem takiej relacji jest użytkownik z wieloma rolami, w którym role są również udostępniane innym użytkownikom. Na przykład wielu użytkowników może pełnić rolę "administratora". Aby zdefiniować tę relację, potrzebne są trzy tabele bazy danych: `users`,` role` i `role_user`. Tabela `role_user` pochodzi z porządku alfabetycznego powiązanych nazw modeli i zawiera kolumny `user_id` i `role_id`.

Relacje wiele do wielu definiuje się, pisząc metodę zwracającą wynik metody `belongsToMany`. Na przykład, zdefiniujmy metodę `role` w naszym modelu `User`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

Po zdefiniowaniu relacji możesz uzyskać dostęp do ról użytkownika za pomocą dynamicznej właściwości `role`:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

Oczywiście, podobnie jak wszystkie inne typy relacji, możesz wywołać metodę `role`, aby kontynuować łączenie więzi zapytania z relacją:

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

Jak wspomniano wcześniej, aby określić nazwę tabeli dla tabeli łączenia, program Eloquent dołącza dwie powiązane nazwy modeli w kolejności alfabetycznej. Możesz jednak zastąpić tę konwencję. Możesz to zrobić, przekazując drugi argument do metody `belongsToMany`:

    return $this->belongsToMany('App\Role', 'role_user');

Oprócz dostosowania nazwy tabeli łączenia można również dostosować nazwy kolumn kluczy w tabeli, przekazując dodatkowe argumenty do metody `belongsToMany`. Trzeci argument to nazwa klucza obcego modelu, w którym definiujesz relację, a czwarty argument to nazwa klucza obcego modelu, do którego się łączysz:

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### Defining The Inverse Of The Relationship - Definiowanie odwrotności relacji

Aby zdefiniować odwrotność relacji wiele do wielu, należy umieścić inne połączenie z `belongsToMany` na powiązanym modelu. Aby kontynuować przykład naszych ról użytkowników, zdefiniujmy metodę `users` w modelu` Role`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

Jak widać, relacja jest dokładnie taka sama, jak jej odpowiednik `User`, z wyjątkiem odniesienia do modelu `App\User`. Ponieważ ponownie używamy metody `belongsToMany`, wszystkie standardowe opcje dostosowywania tabeli i klucza są dostępne podczas definiowania odwrotności relacji wiele do wielu.

#### Retrieving Intermediate Table Columns - Pobieranie pośrednich kolumn tabeli

Jak już się nauczyłeś, praca ze związkami wiele do wielu wymaga obecności tabeli pośredniej. Eloquent zapewnia bardzo pomocne sposoby interakcji z tą tabelą. Na przykład załóżmy, że nasz obiekt `User` ma wiele obiektów `Role`, z którymi jest związany. Po uzyskaniu dostępu do tej relacji możemy uzyskać dostęp do tabeli pośredniej za pomocą atrybutu `pivot` w modelach:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Zauważ, że każdy pobrany przez nas model `Role` jest automatycznie przypisywany do atrybutu `pivot`. Ten atrybut zawiera model reprezentujący tabelę pośrednią i może być używany tak jak każdy inny model Eloquent.

Domyślnie tylko klucze modelu będą obecne w obiekcie `pivot`. Jeśli twoja tabela przestawna zawiera dodatkowe atrybuty, musisz określić je podczas definiowania relacji:

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

Jeśli chcesz, aby tabela przestawna automatycznie zachowywała znaczniki czasu `created_at` i `updated_at`, użyj metody `withTimestamps` w definicji relacji:

    return $this->belongsToMany('App\Role')->withTimestamps();

#### Customizing The `pivot` Attribute Name - Dostosowywanie nazwy atrybutu `pivot`

Jak wspomniano wcześniej, atrybuty z tabeli pośredniej mogą być dostępne w modelach używających atrybutu `pivot`. Można jednak dostosować nazwę tego atrybutu, aby lepiej odzwierciedlał on jego cel w aplikacji.

Na przykład, jeśli aplikacja zawiera użytkowników, którzy mogą subskrybować podcasty, prawdopodobnie istnieje wiele relacji między użytkownikami i podcastami. W takim przypadku możesz zmienić nazwę pośrednika w tabeli pośredniej na `subscription` zamiast` pivot`. Można to zrobić za pomocą metody `as` przy definiowaniu relacji:

    return $this->belongsToMany('App\Podcast')
                    ->as('subscription')
                    ->withTimestamps();

Gdy to zrobisz, możesz uzyskać dostęp do danych tabeli pośredniej, używając niestandardowej nazwy:

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }

#### Filtering Relationships Via Intermediate Table Columns - Filtrowanie relacji przez pośrednie kolumny tabeli


Możesz również filtrować wyniki zwracane przez `belongsToMany` używając metod `wherePivot` i `wherePivotIn` podczas definiowania relacji:

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

#### Defining Custom Intermediate Table Models - Definiowanie niestandardowych modeli tabel pośrednich

Jeśli chcesz zdefiniować model niestandardowy, który będzie reprezentował tabelę pośrednią relacji, możesz wywołać metodę `using` podczas definiowania relacji. Wszystkie niestandardowe modele używane do reprezentowania pośrednich tabel relacji muszą rozszerzać klasę `Illuminate\Database\Eloquent\Relations\Pivot`. Na przykład możemy zdefiniować `Role`, która używa niestandardowego modelu` UserRole`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\UserRole');
        }
    }

Definiując model `UserRole`, rozszerzymy klasę` Pivot`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class UserRole extends Pivot
    {
        //
    }

<a name="has-many-through"></a>
### Has Many Through - Ma wielu przez

Relacja "ma wiele przez" zapewnia wygodny skrót do uzyskiwania dostępu do odległych relacji za pośrednictwem relacji pośredniej. Na przykład model `Country` może mieć wiele modeli `Post` przez pośredni model `User`. W tym przykładzie możesz łatwo zebrać wszystkie posty na blogu dla danego kraju. Spójrzmy na tabele wymagane do zdefiniowania tej relacji:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

Chociaż `posts` nie zawiera kolumny `country_id`, relacja `hasManyThrough` zapewnia dostęp do wpisów w danym kraju za pośrednictwem `$country->posts`. Aby wykonać to zapytanie, Eloquent sprawdza `country_id` w pośredniej tabeli `users`. Po znalezieniu pasujących ID użytkowników są one używane do wysyłania zapytań do tabeli `posts`.

Po przeanalizowaniu struktury tabeli relacji, zdefiniujmy ją w modelu `Country`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

Pierwszym argumentem przekazanym do metody `hasManyThrough` jest nazwa ostatecznego modelu, do którego chcemy uzyskać dostęp, a drugim argumentem jest nazwa modelu pośredniego.

Podczas wykonywania zapytań dotyczących relacji będą używane typowe Wymowne konwencje kluczy obcych. Jeśli chcesz dostosować klucze relacji, możesz przekazać je jako trzeci i czwarty argument do metody `hasManyThrough`. Trzeci argument to nazwa klucza obcego w modelu pośrednim. Czwartym argumentem jest nazwa klucza obcego w ostatecznym modelu. Piąty argument to klucz lokalny, natomiast szósty to klucz lokalny modelu pośredniego:

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // Foreign key on users table...
                'user_id', // Foreign key on posts table...
                'id', // Local key on countries table...
                'id' // Local key on users table...
            );
        }
    }

<a name="polymorphic-relations"></a>
### Polymorphic Relations - Relacje polimorficzne

#### Table Structure - Struktura tabeli

Relacje polimorficzne umożliwiają modelowi przynależność do więcej niż jednego innego modelu na pojedynczym powiązaniu. Załóżmy na przykład, że użytkownicy Twojej aplikacji mogą "komentować" zarówno posty, jak i filmy. Używając polimorficznych relacji, możesz użyć pojedynczej tabeli `comments` dla obu tych scenariuszy. Najpierw przyjrzyjmy się strukturze tabeli wymaganej do zbudowania tej relacji:

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

Dwie ważne kolumny do odnotowania to kolumny `commentable_id` i `commentable_type` w tabeli `comments`. Kolumna `commentable_id` będzie zawierała wartość ID posta lub wideo, podczas gdy kolumna `commentable_type` będzie zawierała nazwę klasy modelu będącego właścicielem. Kolumna `commentable_type` określa sposób, w jaki ORM określa, który "typ" modelu będącego własnością ma zwracać podczas uzyskiwania dostępu do relacji `commentable`.

#### Model Structure - Struktura Modelu

Następnie przejrzyjmy definicje modeli potrzebne do zbudowania tej relacji:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get all of the owning commentable models.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### Retrieving Polymorphic Relations - Odzyskiwanie relacji polimorficznych

Po zdefiniowaniu tabeli i modeli bazy danych można uzyskać dostęp do relacji za pośrednictwem modeli. Na przykład, aby uzyskać dostęp do wszystkich komentarzy do wpisu, możemy skorzystać z właściwości dynamicznej `comments`:

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

Możesz również odzyskać właściciela polimorficznej zależności od modelu polimorficznego, uzyskując dostęp do nazwy metody wykonującej wywołanie funkcji `morphTo`. W naszym przypadku jest to metoda `commentable` w modelu `Comment`. Tak więc uzyskamy dostęp do tej metody jako właściwości dynamicznej:

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

Relacja `commentable` w modelu `Comment` zwróci instancję `Post` lub `Video`, w zależności od tego, który typ modelu jest właścicielem komentarza.

#### Custom Polymorphic Types - Niestandardowe typy polimorficzne

Domyślnie Laravel będzie używać w pełni kwalifikowanej nazwy klasy do przechowywania typu pokrewnego modelu. Na przykład, biorąc pod uwagę powyższy przykład, gdzie `Comment` może należeć do `Post` lub `Video`, domyślnym `commentable_type` będzie odpowiednio `App\Post` lub `App\Video`. Możesz jednak odłączyć swoją bazę danych od wewnętrznej struktury aplikacji. W takim przypadku możesz zdefiniować "mapę przekształceń", aby polecić Eloquentowi używanie niestandardowej nazwy dla każdego modelu zamiast nazwy klasy:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);

Możesz zarejestrować `morphMap` w funkcji `boot` swojego `AppServiceProvider` lub utworzyć oddzielnego dostawcę usług, jeśli chcesz.

<a name="many-to-many-polymorphic-relations"></a>
### Many To Many Polymorphic Relations - Wiele do wielu polimorficznych relacji

#### Table Structure - Struktura tabeli

Oprócz tradycyjnych relacji polimorficznych można również zdefiniować relacje polimorficzne "wiele do wielu". Na przykład model `Post` i `Video` bloga może dzielić relację polimorficzną z modelem `Tag`. Stosowanie polimorficznej relacji wielu do wielu pozwala mieć jedną listę unikatowych tagów, które są udostępniane w blogach i filmach. Najpierw przyjrzyjmy się strukturze tabeli:

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### Model Structure  - Struktura Modeli

Następnie jesteśmy gotowi zdefiniować relacje w modelu. Modele `Post` i `Video` będą miały metodę `tags`, która wywoła metodę `morphToMany` na podstawie klasy Eloquent:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### Defining The Inverse Of The Relationship - Definiowanie odwrotności relacji

Następnie w modelu `Tag` należy zdefiniować metodę dla każdego z powiązanych modeli. Tak więc dla tego przykładu zdefiniujemy metodę `posts` i `video`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### Retrieving The Relationship - Odzyskiwanie relacji

Po zdefiniowaniu tabeli i modeli bazy danych można uzyskać dostęp do relacji za pośrednictwem modeli. Na przykład, aby uzyskać dostęp do wszystkich tagów dla postu, możesz użyć właściwości dynamicznej `tags`:

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

Możesz także odzyskać właściciela polimorficznej relacji z modelu polimorficznego, uzyskując dostęp do nazwy metody wykonującej wywołanie `morphedByMany`. W naszym przypadku jest to metoda `posts` lub `videos` w modelu `Tag`. W ten sposób uzyskasz dostęp do tych metod jako właściwości dynamiczne:

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## Querying Relations - Zapytanie o relacje

Ponieważ wszystkie typy wymownych relacji są definiowane za pomocą metod, możesz wywoływać te metody, aby uzyskać instancję relacji bez wykonywania kwerend relacyjnych. Ponadto, wszystkie typy Wymownych relacji służą również jako [konstruktorzy zapytań](/docs/{{version}}/queries), co umożliwia ci dalsze wiązanie więzi z zapytaniem o relacje, zanim ostatecznie wykonasz SQL w bazie danych.

Na przykład wyobraź sobie system blogów, w którym model `User` ma wiele skojarzonych modeli `Post`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

Możesz zapytać o relację `posts` i dodać dodatkowe więzy do relacji:

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

Możesz użyć dowolnej z metod [query builder](/docs/{{version}}/queries) w relacji, więc zapoznaj się z dokumentacją budującego zapytania, aby poznać wszystkie dostępne metody.

<a name="relationship-methods-vs-dynamic-properties"></a>
### Relationship Methods Vs. Dynamic Properties - Metody relacji kontra Właściwości dynamiczne

Jeśli nie trzeba dodawać dodatkowych ograniczeń do kwerendy Eloquent relacji, można uzyskać dostęp do relacji tak, jakby była właściwość. Na przykład, kontynuując korzystanie z naszych przykładowych modeli `User` i `Post`, możemy uzyskać dostęp do wszystkich postów użytkownika, takich jak:

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Właściwości dynamiczne to "leniwe ładowanie", co oznacza, że będą ładować swoje dane relacji tylko wtedy, gdy faktycznie uzyskasz do nich dostęp. Z tego powodu programiści często używają [chętne ładowania] (# eager-loading) do wstępnego załadowania relacji, o których wiedzą, że będą dostępni po załadowaniu modelu. Chętne ładowanie zapewnia znaczną redukcję zapytań SQL, które muszą zostać wykonane, aby wczytać relacje modelu.

<a name="querying-relationship-existence"></a>
### Querying Relationship Existence - Zapytanie o istnienie relacji

Uzyskując dostęp do rekordów modelu, można ograniczyć wyniki w oparciu o istnienie relacji. Na przykład wyobraź sobie, że chcesz odzyskać wszystkie posty na blogu, które mają co najmniej jeden komentarz. Aby to zrobić, możesz przekazać nazwę relacji do metod `has` i` orHas`:

    // Retrieve all posts that have at least one comment...
    $posts = App\Post::has('comments')->get();

Możesz także określić operatora i policzyć, aby jeszcze bardziej dostosować zapytanie:

    // Retrieve all posts that have three or more comments...
    $posts = App\Post::has('comments', '>=', 3)->get();

Zagnieżdżone instrukcje `has` mogą być również tworzone przy użyciu notacji "kropki". Możesz na przykład pobrać wszystkie posty z co najmniej jednym komentarzem i głosować:

    // Retrieve all posts that have at least one comment with votes...
    $posts = App\Post::has('comments.votes')->get();

Jeśli potrzebujesz jeszcze więcej mocy, możesz użyć metod `whereHas` i` orWhereHas`, aby umieścić warunki "where" na swoich zapytaniach `has`. Te metody umożliwiają dodawanie dostosowanych wiązań do więzów relacji, takich jak sprawdzanie treści komentarza:

    // Retrieve all posts with at least one comment containing words like foo%
    $posts = App\Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="querying-relationship-absence"></a>
### Querying Relationship Absence - Zapytanie o nieobecność w relacji

Uzyskując dostęp do rekordów modelu, możesz ograniczyć wyniki w oparciu o brak relacji. Na przykład wyobraź sobie, że chcesz odzyskać wszystkie posty na blogu, które **nie mają** żadnych komentarzy. Aby to zrobić, możesz przekazać nazwę relacji do metod `doesntHave` i` orDoesntHave`:

    $posts = App\Post::doesntHave('comments')->get();

Jeśli potrzebujesz jeszcze więcej mocy, możesz użyć metod `whereDoesntHave` i `orWhereDoesntHave`, aby umieścić warunki "where" na swoich zapytaniach `doesnthave`. Te metody umożliwiają dodawanie dostosowanych ograniczeń do więzów relacji, takich jak sprawdzanie treści komentarza:

    $posts = App\Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### Counting Related Models - Zliczanie powiązanych modeli

Jeśli chcesz policzyć liczbę wyników z relacji bez faktycznego ich ładowania, możesz użyć metody `withCount`, która umieści kolumnę `{relation}_count` na wynikowych modelach. Na przykład:

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

Możesz dodać "liczniki" dla wielu relacji, a także dodać ograniczenia do zapytań:

    $posts = App\Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

Możesz także dać alias wyniku liczenia relacji, zezwalając na wiele zliczeń na tej samej relacji:

    $posts = App\Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();

    echo $posts[0]->comments_count;

    echo $posts[0]->pending_comments_count;

<a name="eager-loading"></a>
## Eager Loading - Chetne wczytywanie

Podczas uzyskiwania dostępu do relacji Eloquent jako właściwości dane relacji są "wczytane". Oznacza to, że dane relacji nie są faktycznie ładowane, dopóki nie uzyskasz dostępu do właściwości. Jednak Eloquent może "chciwie wczytać" relacje w czasie, gdy wyszukujesz model macierzysty. Chętne ładowanie zmniejsza problem z zapytaniem N + 1. Aby zilustrować problem z zapytaniem N + 1, rozważ model `Book`, który jest powiązany z `Author`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

Teraz pobierzmy wszystkie książki i ich autorów:

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Ta pętla wykona 1 zapytanie, aby pobrać wszystkie książki z tabeli, a następnie kolejne zapytanie do każdej książki, aby pobrać autora. Tak więc, jeśli mamy 25 książek, w tej pętli pojawi się 26 zapytań: 1 dla oryginalnej książki i 25 dodatkowych zapytań w celu pobrania autora każdej książki.

Na szczęście możemy użyć szybkiego ładowania, aby zmniejszyć tę operację do zaledwie dwóch zapytań. Podczas odpytywania możesz określić, które relacje powinny być ładowane za pomocą metody `with`:

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

W przypadku tej operacji zostaną wykonane tylko dwa zapytania:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager Loading Multiple Relationships - Chętne ładowanie wielu relacji

Czasami może zajść potrzeba załadowania kilku różnych relacji w jednej operacji. Aby to zrobić, wystarczy przekazać dodatkowe argumenty do metody `with`:

    $books = App\Book::with(['author', 'publisher'])->get();

#### Nested Eager Loading - Zagnieżdżone chętne ładowanie

Do pożądanych relacji zagnieżdżonych można użyć składni "kropka". Na przykład, spróbujmy załadować wszystkich autorów książki i wszystkich osobistych kontaktów autora w jednym Wymownym oświadczeniu:

    $books = App\Book::with('author.contacts')->get();

#### Eager Loading Specific Columns - Chętne ładowanie określonych kolumn

Nie zawsze możesz potrzebować każdej kolumny z relacji, które odzyskujesz. Z tego powodu, Eloquent pozwala ci określić, które kolumny relacji chcesz odzyskać:

    $users = App\Book::with('author:id,name')->get();

> {note} Korzystając z tej funkcji, należy zawsze umieszczać kolumnę "id" na liście kolumn, które chcesz odzyskać.

<a name="constraining-eager-loads"></a>
### Constraining Eager Loads - Ograniczanie chętnych obciążeń

Czasami możesz chcieć załadować relację, ale także określić dodatkowe ograniczenia zapytań dla wymagającego zapytania ładowania. Oto przykład:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

W tym przykładzie Eloquent będzie tylko chętny do ładowania wpisów, w których kolumna `title` zawiera słowo `first`. Oczywiście można wywoływać inne metody [konstruktora zapytań](/docs/{{version}}/queries), aby jeszcze bardziej dostosować wymaganą operację ładowania:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading - Leniwe chciwe ładowanie

Czasami może zajść potrzeba szybkiego załadowania relacji po pobraniu modelu nadrzędnego. Na przykład może to być przydatne, jeśli musisz dynamicznie decydować, czy załadować powiązane modele:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

Jeśli chcesz ustawić dodatkowe ograniczenia zapytań w chciwym ładowaniu zapytania, możesz przekazać tablicy kluczy przez relacje, które chcesz załadować. Wartości tablicowe powinny być instancjami typu `Closure`, które odbierają instancję zapytania:

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

Aby wczytać relację tylko wtedy, gdy nie została jeszcze załadowana, użyj metody `loadMissing`:

    public function format(Book $book)
    {
        $book->loadMissing('author');

        return [
            'name' => $book->name,
            'author' => $book->author->name
        ];
    }

<a name="inserting-and-updating-related-models"></a>
## Inserting & Updating Related Models - Wstawianie i aktualizowanie powiązanych modeli

<a name="the-save-method"></a>
### The Save Method - Metoda zapisu

Eloquent zapewnia wygodne metody dodawania nowych modeli do relacji. Na przykład, być może musisz wstawić nowy `Comment` do modelu `Post`. Zamiast ręcznie ustawiać atrybut `post_id` w` Comment`, możesz wstawić `Comment` bezpośrednio z metody` save` relacji:

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

Zauważ, że nie mamy dostępu do relacji `comments` jako właściwości dynamicznej. Zamiast tego wywołaliśmy metodę `comments`, aby uzyskać instancję relacji. Metoda `save` automatycznie doda odpowiednią wartość` post_id` do nowego modelu `Comment`.

Jeśli chcesz zapisać wiele powiązanych modeli, możesz użyć metody `saveMany`:

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-create-method"></a>
### The Create Method - Metoda Stwórz

Oprócz metod `save` i `saveMany` możesz również użyć metody `create`, która akceptuje tablicę atrybutów, tworzy model i wstawia go do bazy danych. Znowu różnica między `save` i `create` jest taka, że `save` akceptuje pełną instancję modelu Eloquent, podczas gdy `create` akceptuje zwykłą tablicę `array` PHP:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

> {tip} Przed użyciem metody `create` należy zapoznać się z dokumentacją dotyczącą atrybutu [masowego przypisywania](/docs/{{version}}/eloquent#mass-assignment).

Możesz użyć metody `createMany` do stworzenia wielu powiązanych modeli:

    $post = App\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);

<a name="updating-belongs-to-relationships"></a>
### Belongs To Relationships - Należy do relacji

Podczas aktualizowania relacji `belongsTo` możesz użyć metody `associate`. Ta metoda ustawi klucz obcy w modelu podrzędnym:

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

Podczas usuwania relacji `belongsTo` możesz użyć metody` dissociate`. Ta metoda ustawi klucz obcy relacji na `null`:

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### Many To Many Relationships - Wiele do wielu relacji

#### Attaching / Detaching - Dołączanie / odłączanie

Eloquent udostępnia również kilka dodatkowych metod pomocniczych, które ułatwiają pracę z powiązanymi modelami. Na przykład, wyobraźmy sobie, że użytkownik może mieć wiele ról, a rola może mieć wielu użytkowników. Aby dołączyć rolę do użytkownika, wstawiając rekord do tabeli pośredniej, która dołącza do modeli, użyj metody `attach`:

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

Podczas dołączania relacji do modelu można również przekazać szereg dodatkowych danych do wstawienia do tabeli pośredniej:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Oczywiście czasami może być konieczne usunięcie roli od użytkownika. Aby usunąć rekord relacji wielu do wielu, użyj metody `detach`. Metoda `detach` usunie odpowiedni rekord z tabeli pośredniej; jednak oba modele pozostaną w bazie danych:

    // Detach a single role from the user... - Odłącz jedną rolę od użytkownika ...
    $user->roles()->detach($roleId);

    // Detach all roles from the user... - Odłącz wszystkie role od użytkownika ...
    $user->roles()->detach();

Dla wygody, `attach` i `detach` również akceptują tablice identyfikatorów jako dane wejściowe:

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires]
    ]);

#### Syncing Associations - Synchronizowanie powiązań

Możesz również użyć metody `sync` do skonstruowania wielu skojarzeń. Metoda `sync` akceptuje tablicę identyfikatorów do umieszczenia w tabeli pośredniej. Wszelkie identyfikatory, które nie znajdują się w podanej macierzy, zostaną usunięte z tabeli pośredniej. Tak więc po zakończeniu tej operacji w tabeli pośredniej będą znajdować się tylko identyfikatory z danej tablicy:

    $user->roles()->sync([1, 2, 3]);

Możesz również przekazać dodatkowe wartości tabeli pośredniej z identyfikatorami:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

Jeśli nie chcesz odłączyć istniejących identyfikatorów, możesz użyć metody `syncWithoutDetaching`:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### Toggling Associations - Przełączanie powiązań

Relacja wiele-do-wielu zapewnia również metodę `toggle`, która "przełącza" status przyłączenia podanych identyfikatorów. Jeśli dany identyfikator jest obecnie dołączony, zostanie odłączony. Podobnie, jeśli jest on obecnie odłączony, zostanie dołączony:

    $user->roles()->toggle([1, 2, 3]);

#### Saving Additional Data On A Pivot Table - Zapisywanie dodatkowych danych w tabeli przestawnej

Podczas pracy z relacją wiele do wielu metoda `save`` przyjmuje jako argument drugi tablicę dodatkowych atrybutów tabeli pośredniej:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Updating A Record On A Pivot Table - Aktualizowanie rekordu w tabeli przestawnej

Jeśli chcesz zaktualizować istniejący wiersz w tabeli przestawnej, możesz użyć metody `updateExistingPivot`. Ta metoda akceptuje klucz obcy rekordu przestawnego i tablicę atrybutów do zaktualizowania:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## Touching Parent Timestamps - Dotykanie znaczników czasu dla rodziców

Gdy model `belongsTo` lub `belongToMany` jest inny, np. `Comment`, który należy do `Post`, czasami pomocne jest zaktualizowanie znacznika czasu nadrzędnego, gdy model podrzędny jest aktualizowany. Na przykład, gdy model `Comment` jest aktualizowany, możesz automatycznie "dotknąć" znacznika czasu `updated_at` posiadacza `Post`. Eloquent to ułatwia. Wystarczy dodać właściwość touches` zawierającą nazwy relacji do modelu potomnego:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Teraz, po aktualizacji `Comment`, posiadanie `Post` będzie aktualizowało również kolumnę `updated_at`, dzięki czemu wygodniej będzie wiedzieć, kiedy unieważnić pamięć podręczną modelu `Post`:

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
