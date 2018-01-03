# Authorization

- [Introduction - Wprowadzenie](#introduction)
- [Gates - Bramy](#gates)
    - [Writing Gates - Pisanie bram](#writing-gates)
    - [Authorizing Actions - Działania autoryzujące](#authorizing-actions-via-gates)
- [Creating Policies - Tworzenie polityk](#creating-policies)
    - [Generating Policies - Generowanie polityk](#generating-policies)
    - [Registering Policies - Rejestrowanie polityk](#registering-policies)
- [Writing Policies - Pisanie polityk](#writing-policies)
    - [Policy Methods - Metody polityki](#policy-methods)
    - [Methods Without Models - Metody bez modeli](#methods-without-models)
    - [Policy Filters - Filtry polityk](#policy-filters)
- [Authorizing Actions Using Policies - Autoryzacja działań przy użyciu polityk](#authorizing-actions-using-policies)
    - [Via The User Model - Przez model użytkownika](#via-the-user-model)
    - [Via Middleware - Przez oprogramowanie pośredniczące](#via-middleware)
    - [Via Controller Helpers - Za pomocą pomocników kontrolera](#via-controller-helpers)
    - [Via Blade Templates - Przez szablony Blade](#via-blade-templates)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Oprócz dostarczania usług [uwierzytelniania](/docs/{{version}}/authentication) po wyjęciu z pudełka, Laravel zapewnia również prosty sposób autoryzowania działań użytkownika względem danego zasobu. Podobnie jak uwierzytelnianie, podejście Laravel do autoryzacji jest proste i istnieją dwa główne sposoby autoryzacji działań: bramki i polityki.

Pomyśl o bramkach i politykach, takich jak trasy i kontrolery. Gates zapewnia proste, oparte na Closure podejście do autoryzacji, podczas gdy zasady, takie jak kontrolery, grupują swoją logikę wokół określonego modelu lub zasobu. Najpierw zbadamy bramy, a następnie sprawdzimy polityki.

Nie musisz wybierać pomiędzy wyłącznym korzystaniem z bramek lub wyłącznie przy użyciu zasad podczas budowania aplikacji. Większość aplikacji będzie najprawdopodobniej zawierać mieszankę bramek i polityk, i to jest w porządku! Bramki są najbardziej odpowiednie dla działań, które nie są związane z żadnym modelem ani zasobem, takich jak wyświetlanie pulpitu administratora. Natomiast zasady powinny być stosowane, gdy chcesz autoryzować działanie dla konkretnego modelu lub zasobu.

<a name="gates"></a>
## Gates - Bramy

<a name="writing-gates"></a>
### Writing Gates - Pisanie bram

Bramki są Closures, które określają, czy użytkownik jest uprawniony do wykonania danej czynności i są zazwyczaj zdefiniowane w klasie `App\Providers\AuthServiceProvider` za pomocą elewacji `Gate`. Bramki zawsze otrzymują instancję użytkownika jako swój pierwszy argument i mogą opcjonalnie otrzymać dodatkowe argumenty, takie jak odpowiedni model Eloquent:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

Bramki można również zdefiniować za pomocą łańcucha wywołań stylu `Class@method`, takich jak kontrolery:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'PostPolicy@update');
    }

#### Resource Gates - Bramy zasobów

Możesz także zdefiniować wiele zdolności Bram jednocześnie za pomocą metody `resource`:

    Gate::resource('posts', 'PostPolicy');

Jest to identyczne z ręcznym definiowaniem następujących definicji bramek:

    Gate::define('posts.view', 'PostPolicy@view');
    Gate::define('posts.create', 'PostPolicy@create');
    Gate::define('posts.update', 'PostPolicy@update');
    Gate::define('posts.delete', 'PostPolicy@delete');

Domyślnie zdefiniowane zostaną zdolności `view`, `create`, `update` oraz `delete`. Możesz przesłonić lub dodać domyślne umiejętności, przekazując tablicę jako trzeci argument do metody `resource`. Klucze tablicy definiują nazwy zdolności, podczas gdy wartości definiują nazwy metod. Na przykład poniższy kod utworzy dwie nowe definicje bramek - "posts.image" i "posts.photo":

    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);

<a name="authorizing-actions-via-gates"></a>
### Authorizing Actions - Działania autoryzujące

Aby autoryzować akcję za pomocą bramek, powinieneś użyć metod `allow` lub `denies`. Pamiętaj, że nie musisz przekazywać aktualnie uwierzytelnionego użytkownika do tych metod. Laravel automatycznie zajmie się przekazaniem użytkownika do Closure bramy:

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }

Jeśli chcesz określić, czy dany użytkownik jest uprawniony do wykonania działania, możesz użyć metody `forUser` na elewacji `Gate`:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

<a name="creating-policies"></a>
## Creating Policies - Tworzenie polityk

<a name="generating-policies"></a>
### Generating Policies - Generowanie polityk

Polityki to klasy organizujące logikę autoryzacji wokół określonego modelu lub zasobu. Na przykład, jeśli twoja aplikacja jest blogiem, możesz mieć model `Post` i odpowiadającą mu `PostPolicy`, aby autoryzować akcje użytkownika, takie jak tworzenie lub aktualizowanie wpisów.

Możesz wygenerować politykę za pomocą `make:policy` [polecenie artisan](/docs/{{version}}/artisan). Wygenerowana polityka zostanie umieszczona w katalogu `app/Policies`. Jeśli ten katalog nie istnieje w Twojej aplikacji, Laravel utworzy go dla ciebie:

    php artisan make:policy PostPolicy

Komenda `make:policy` wygeneruje pustą klasę polityk. Jeśli chcesz wygenerować klasę z podstawowymi metodami "CRUD" już zawartymi w klasie, możesz określić `--model` podczas wykonywania polecenia:

    php artisan make:policy PostPolicy --model=Post

> {tip} Wszystkie zasady są rozwiązywane za pośrednictwem Laravel [kontener usługi](/docs/{{version}}/container), umożliwiając wpisanie podpowiedzi potrzebnych zależności w konstruktorze zasady w celu automatycznego ich wstrzyknięcia.

<a name="registering-policies"></a>
### Registering Policies - Rejestrowanie polityk

Gdy zasady już istnieją, muszą zostać zarejestrowane. `AuthServiceProvider` dołączony do świeżych aplikacji Laravel zawiera właściwość `policies`, która odwzorowuje modele Eloquent na odpowiadające im zasady. Rejestrowanie zasad będzie instruować Laravel, jaką politykę zastosować podczas autoryzowania działań w stosunku do danego modelu:

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## Writing Policies - Pisanie polityk

<a name="policy-methods"></a>
### Policy Methods - Metody polityki

Po zarejestrowaniu zasad możesz dodać metody dla każdej akcji, którą autoryzujesz. Na przykład, zdefiniujmy metodę `update` w naszym `PostPolicy`, która określa, czy dany `User` może aktualizować daną instancję `Post`.

Metoda `update` otrzyma jako argumenty instancję `User` i `Post` i powinna zwrócić wartość `true` lub `false` wskazującą, czy użytkownik jest uprawniony do aktualizacji danego `Post`. Tak więc dla tego przykładu sprawdźmy, czy `id` użytkownika pasuje do `user_id`  w poście:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Możesz nadal definiować dodatkowe metody polityki, zgodnie z potrzebami różnych akcji, które autoryzuje. Na przykład możesz zdefiniować metody `view` lub `delete`, aby autoryzować różne akcje `Post`, ale pamiętaj, że możesz dowolnie nadawać swoim metodom politycznym dowolną nazwę.

> {tip} Jeśli użyłeś opcji `--model` podczas generowania polityki poprzez konsolę Artisan, będzie ona już zawierała metody dla akcji `view`, `create`, `update`, oraz `delete`.

<a name="methods-without-models"></a>
### Methods Without Models - Metody bez modeli

Niektóre metody polityki otrzymują tylko uwierzytelnionego użytkownika, a nie instancję autoryzowanego modelu. Ta sytuacja jest najczęstsza podczas autoryzowania akcji `create`. Na przykład, jeśli tworzysz bloga, możesz chcieć sprawdzić, czy użytkownik jest uprawniony do tworzenia jakichkolwiek postów w ogóle.

Podczas definiowania metod strategii, które nie będą otrzymywać instancji modelu, takich jak metoda `create`, nie będzie ona otrzymywać instancji modelu. Zamiast tego powinieneś zdefiniować metodę jako oczekującą tylko uwierzytelnionego użytkownika:

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

<a name="policy-filters"></a>
### Policy Filters - Filtry polityk

W przypadku niektórych użytkowników możesz autoryzować wszystkie działania w ramach danej polityki. Aby to osiągnąć, zdefiniuj metodę `before` w strategii. Metoda `before` zostanie wykonana przed innymi metodami polityki, dając możliwość autoryzacji akcji przed wywołaniem zamierzonej metody polityki. Ta funkcja jest najczęściej używana do autoryzowania administratorów aplikacji do wykonywania dowolnych czynności:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Jeśli chcesz odmówić wszystkim autoryzacji dla użytkownika, powinieneś zwrócić `false` z metody `before`. Zwrócenie `null` spowoduje, że autoryzacja przejdzie do metody polityki.

> {note} Metoda `before` klasy polityki nie zostanie wywołana, jeśli klasa nie zawiera metody o nazwie pasującej do nazwy sprawdzanej umiejętności.

<a name="authorizing-actions-using-policies"></a>
## Authorizing Actions Using Policies - Autoryzacja działań przy użyciu polityk

<a name="via-the-user-model"></a>
### Via The User Model - Przez model użytkownika

Model `User`, który jest dołączony do twojej aplikacji Laravel zawiera dwie pomocne metody autoryzacji akcji: `can` i `cant`. Metoda `can` odbiera akcję, którą chcesz autoryzować, i odpowiedni model. Na przykład, ustalmy, czy użytkownik jest uprawniony do aktualizacji danego modelu `Post`:

    if ($user->can('update', $post)) {
        //
    }

Jeśli [polityka zostanie zarejestrowana](#registering-policies) dla danego modelu, metoda `can` automatycznie wywoła odpowiednią politykę i zwróci wynik boolowski. Jeśli żadna strategia nie jest zarejestrowana dla modelu, metoda `can` spróbuje wywołać Bramę opartą na Closure, pasującą do danej nazwy akcji.


#### Actions That Don't Require Models - Działania, które nie wymagają modeli

Pamiętaj, że niektóre akcje, takie jak `create`, mogą nie wymagać wystąpienia modelu. W takich sytuacjach możesz przekazać nazwę klasy do metody `can`. Nazwa klasy zostanie użyta do określenia, której zasady należy użyć podczas autoryzowania działania:

    use App\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>
### Via Middleware - Przez oprogramowanie pośredniczące

Laravel zawiera oprogramowanie pośrednie, które może autoryzować akcje zanim przychodzące żądanie dotrze do twoich tras lub kontrolerów. Domyślnie oprogramowanie pośrednie `Illuminate\Auth\Middleware\Authorize` ma przypisany klucz `can` w klasie `App\Http\Kernel`. Przyjrzyjmy się przykładowi używania oprogramowania pośredniego `can` do autoryzacji, żeby użytkownik mogł zaktualizować post na blogu:

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

W tym przykładzie podajemy dwa argumenty `can` middleware. Pierwsza to nazwa akcji, którą chcemy autoryzować, a druga to parametr trasy, który chcemy przekazać do metody polityki. W tym przypadku, ponieważ używamy [niejawnego powiązania modelu](/docs/{{version}}/routing#implicit-binding), model `Post` zostanie przekazany do metody policy. Jeśli użytkownik nie jest uprawniony do wykonania danej czynności, oprogramowanie pośredniczące wygeneruje odpowiedź HTTP z kodem statusu "403".

#### Actions That Don't Require Models - Działania, które nie wymagają modeli

Znowu niektóre akcje, takie jak `create`, mogą nie wymagać wystąpienia modelu. W takich sytuacjach możesz przekazać nazwę klasy do oprogramowania pośredniego. Nazwa klasy zostanie użyta do określenia, której zasady należy użyć podczas autoryzowania działania:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### Via Controller Helpers - Za pomocą pomocników kontrolera

Oprócz pomocnych metod dostarczonych do modelu `User`, Laravel dostarcza pomocną metodę `authorize` dla każdego ze swoich kontrolerów, które rozszerzają klasę bazową `App\Http\Controllers\Controller`. Podobnie jak metoda `can`, ta metoda akceptuje nazwę akcji, którą chcesz autoryzować i odpowiedni model. Jeśli akcja nie jest autoryzowana, metoda `authorize` wygeneruje `Illuminate\Auth\Access\AuthorizationException`, który domyślny program obsługi wyjątków Laravel będzie konwertował na odpowiedź HTTP z kodem statusu `403`:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

#### Actions That Don't Require Models - Działania, które nie wymagają modeli

Jak wcześniej wspomniano, niektóre akcje, takie jak `create`, mogą nie wymagać wystąpienia modelu. W takich sytuacjach możesz przekazać nazwę klasy do metody `authorize`. Nazwa klasy zostanie użyta do określenia, której zasady należy użyć podczas autoryzowania działania:

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="via-blade-templates"></a>
### Via Blade Templates - Przez szablony Blade

Podczas pisania szablonów Blade możesz chcieć wyświetlić część strony tylko wtedy, gdy użytkownik jest uprawniony do wykonania danej akcji. Na przykład możesz chcieć pokazać formularz aktualizacji dla wpisu na blogu tylko wtedy, gdy użytkownik może zaktualizować wpis. W tej sytuacji możesz korzystać z rodziny dyrektyw  `@can` i `@cannot`:


    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', App\Post::class)
        <!-- The Current User Can Create New Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @elsecannot('create', App\Post::class)
        <!-- The Current User Can't Create New Post -->
    @endcannot

Te dyrektywy są wygodnymi skrótami do pisania instrukcji `@if` i `@unless`. Powyższe instrukcje `@can` i `@cannot` oznaczają odpowiednio następujące wyrażenia:

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless

#### Actions That Don't Require Models - Działania, które nie wymagają modeli

Podobnie jak w przypadku większości innych metod autoryzacji, możesz przekazać nazwę klasy do dyrektyw `@can` i `@cannot`, jeśli akcja nie wymaga wystąpienia modelu:

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot
