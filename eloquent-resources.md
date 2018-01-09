# Eloquent: API Resources

- [Introduction - Wprowadzenie](#introduction)
- [Generating Resources - Generowanie zasobów](#generating-resources)
- [Concept Overview - Przegląd koncepcji](#concept-overview)
- [Writing Resources - Pisanie zasobów](#writing-resources)
    - [Data Wrapping - Opakowywanie danych](#data-wrapping)
    - [Pagination - Paginacja](#pagination)
    - [Conditional Attributes - Atrybuty warunkowe](#conditional-attributes)
    - [Conditional Relationships - Relacje warunkowe](#conditional-relationships)
    - [Adding Meta Data - Dodawanie danych meta](#adding-meta-data)
- [Resource Responses - Odpowiedzi zasobów](#resource-responses)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Podczas budowania interfejsu API może być potrzebna warstwa transformacji, która znajduje się między modelami wymownymi a odpowiedziami JSON, które są faktycznie zwracane użytkownikom aplikacji. Klasy zasobów Laravel umożliwiają ekspresyjną i łatwą transformację modeli i kolekcji modeli w JSON.

<a name="generating-resources"></a>
## Generating Resources - Generowanie zasobów

Aby wygenerować klasę zasobów, możesz użyć polecenia `make:resource` Artisan. Domyślnie zasoby zostaną umieszczone w katalogu `app/Http/Resources` aplikacji. Zasoby rozszerzają klasę `Illuminate\Http\Resources\Json\Resource`:

    php artisan make:resource UserResource

#### Resource Collections - Kolekcje zasobów

Oprócz generowania zasobów, które przekształcają poszczególne modele, można generować zasoby odpowiedzialne za przekształcanie kolekcji modeli. Dzięki temu Twoja odpowiedź może zawierać linki i inne meta informacje, które są istotne dla całej kolekcji danego zasobu.

Aby utworzyć kolekcję zasobów, podczas tworzenia zasobu powinieneś użyć flagi `--collection`. Lub po prostu wpisanie słowa `Collection` w nazwie zasobu wskaże Laravelowi, że powinien on utworzyć zasób kolekcji. Zasoby kolekcji rozszerzają klasę `Illuminate\Http\Resources\Json\ResourceCollection`:

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## Concept Overview - Przegląd koncepcji

> {tip} To jest ogólny przegląd zasobów i kolekcji zasobów. Zachęcamy do zapoznania się z pozostałymi rozdziałami tej dokumentacji, aby lepiej zrozumieć personalizację i moc oferowaną przez zasoby.

Zanim przejdziemy do wszystkich dostępnych opcji podczas pisania zasobów, najpierw przyjrzyjmy się, jak wykorzystujemy zasoby w Laravel. Klasa zasobów reprezentuje pojedynczy model, który musi zostać przekształcony w strukturę JSON. Na przykład tutaj jest prosta klasa `UserResource`:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Każda klasa zasobów definiuje metodę `toArray`, która zwraca tablicę atrybutów, które powinny zostać przekonwertowane na JSON podczas wysyłania odpowiedzi. Zauważ, że możemy uzyskać dostęp do właściwości modelu bezpośrednio ze zmiennej `$this`. Dzieje się tak dlatego, że klasa zasobów automatycznie pośredniczy w dostępie do właściwości i metody do podstawowego modelu w celu uzyskania wygodnego dostępu. Po zdefiniowaniu zasobu może on zostać zwrócony z trasy lub kontrolera:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

### Resource Collections - Kolekcje zasobów

Jeśli zwracasz kolekcję zasobów lub odpowiedź w postaci stronicowanej, możesz użyć metody `collection` podczas tworzenia instancji zasobu na trasie lub kontrolerze:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Oczywiście nie pozwala to na dodawanie metadanych, które mogą wymagać zwrotu z kolekcji. Jeśli chcesz dostosować odpowiedź kolekcji zasobów, możesz utworzyć dedykowany zasób do reprezentowania kolekcji:

    php artisan make:resource UserCollection

Po wygenerowaniu klasy kolekcji zasobów możesz łatwo zdefiniować dowolne dane meta, które powinny zostać dołączone do odpowiedzi:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Po zdefiniowaniu kolekcji zasobów można ją zwrócić z trasy lub kontrolera:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="writing-resources"></a>
## Writing Resources - Pisanie zasobów

> {tip} Jeśli nie przeczytałeś [przeglądu koncepcji](#concept-overview), bardzo cię zachencamy, aby to zrobić przed kontynuowaniem tej dokumentacji.

W istocie zasoby są proste. Wystarczy przekształcić dany model w tablicę. Tak więc każdy zasób zawiera metodę "toArray", która tłumaczy atrybuty twojego modelu na tablicę przyjazną dla interfejsu API, która może zostać zwrócona do użytkowników:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Po zdefiniowaniu zasobu może on zostać zwrócony bezpośrednio z trasy lub kontrolera:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

#### Relationships - Relacje

Jeśli chcesz uwzględnić powiązane zasoby w swojej odpowiedzi, możesz po prostu dodać je do tablicy zwróconej przez metodę `toArray`. W tym przykładzie użyjemy metody `collection` zasobów `Post`, aby dodać posty blogów użytkownika do odpowiedzi na zasoby:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} Jeśli chcesz uwzględnić relacje tylko wtedy, gdy zostały już załadowane, sprawdź dokumentację na temat [relacji warunkowych](#conditional-relationships).

#### Resource Collections - Kolekcje zasobów

Podczas gdy zasoby tłumaczą pojedynczy model w postaci tablicy, kolekcje zasobów tłumaczą kolekcję modeli w tablicy. Nie jest absolutnie konieczne zdefiniowanie klasy kolekcji zasobów dla każdego z typów modeli, ponieważ wszystkie zasoby udostępniają metodę `collection` do generowania kolekcji zasobów "ad-hoc" w locie:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Jeśli jednak chcesz dostosować metadane zwrócone w kolekcji, konieczne będzie zdefiniowanie kolekcji zasobów:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Podobnie jak pojedyncze zasoby, kolekcje zasobów mogą być zwracane bezpośrednio z tras lub kontrolerów:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### Data Wrapping - Opakowywanie danych

Domyślnie twój najbardziej zewnętrzny zasób jest zawijany w klucz `data`, kiedy odpowiedź na zasoby jest konwertowana na JSON. Na przykład typowa odpowiedź kolekcji zasobów wygląda następująco:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }

Jeśli chcesz wyłączyć zawijanie najbardziej zewnętrznego zasobu, możesz użyć metody `withoutWrapping` na podstawowej klasie zasobów. Zwykle powinieneś wywołać tę metodę z `AppServiceProvider` lub innego [usługodawcy](/docs/{{version}}/providers), który jest ładowany przy każdym żądaniu do twojej aplikacji:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
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

> {note} Metoda `withoutWrapping` wpływa tylko na najbardziej zewnętrzną odpowiedź i nie usuwa kluczy  `data`, które ręcznie dodajesz do własnych kolekcji zasobów.

### Wrapping Nested Resources - Zawijanie zasobów zagnieżdżonych

Masz całkowitą swobodę w określaniu, w jaki sposób relacje twojego zasobu są zawijane. Jeśli chcesz, aby wszystkie kolekcje zasobów były pakowane w klucz `data`, niezależnie od ich zagnieżdżenia, powinieneś zdefiniować klasę kolekcji zasobów dla każdego zasobu i zwrócić kolekcję w obrębie klucza `data`.

Oczywiście możesz się zastanawiać, czy to spowoduje, że twój najbardziej zewnętrzny zasób zostanie zawinięty w dwa klucze `data`. Nie przejmuj się, Laravel nigdy nie pozwoli, aby twoje zasoby przypadkowo zostały podwójnie owinięte, więc nie musisz martwić się o poziom zagnieżdżenia kolekcji zasobów, którą transformujesz:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

### Data Wrapping And Pagination - Zawijanie danych i paginacja

Podczas zwracania paginowanych kolekcji w odpowiedzi na zasób, Laravel opakuje dane zasobów w klucz `data`, nawet jeśli wywołano metodę `withoutWrapping`. Dzieje się tak dlatego, że w odpowiedzi na paginację zawsze znajdują się klucze `meta` i `links` z informacjami o stanie paginatora:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="pagination"></a>
### Pagination - Paginacja

Zawsze możesz przekazać instancję paginatora do metody `collection` zasobu lub do niestandardowego zbioru zasobów:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

Odpowiedzi podzielone na części zawsze zawierają klucze `meta` i `links` z informacjami o stanie paginatora:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="conditional-attributes"></a>
### Conditional Attributes - Atrybuty warunkowe

Czasami możesz chcieć uwzględnić atrybut tylko w odpowiedzi na zasób, jeśli dany warunek zostanie spełniony. Na przykład możesz chcieć uwzględnić tylko wartość, jeśli bieżący użytkownik jest "administratorem". Laravel oferuje różne metody pomocnicze, które pomogą Ci w tej sytuacji. Do warunkowego dodania atrybutu do odpowiedzi na zasób można użyć metody `when`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($this->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

W tym przykładzie klucz `secret` zostanie zwrócony tylko w ostatecznej odpowiedzi na zasób, jeśli` $this->isAdmin()` zwróci `true`. Jeśli metoda zwróci wartość `false`, klucz `secret` zostanie całkowicie usunięty z odpowiedzi zasobów, zanim zostanie odesłany do klienta. Metoda `when` pozwala na ekspresywne definiowanie zasobów bez odwoływania się do instrukcji warunkowych podczas budowania tablicy.

Metoda `when` akceptuje również zamknięcie jako swój drugi argument, pozwalający obliczyć wynikową wartość tylko wtedy, gdy podany warunek jest `true`:

    'secret' => $this->when($this->isAdmin(), function () {
        return 'secret-value';
    }),

> {tip} Pamiętaj, metoda wywołuje proxy zasobów w dół do podstawowej instancji modelu. Tak więc w tym przypadku metoda `isAdmin` jest proxy dla podstawowego modelu Eloquent, który został pierwotnie podany do zasobu.

#### Merging Conditional Attributes - Łączenie atrybutów warunkowych

Sometimes you may have several attributes that should only be included in the resource response based on the same condition. In this case, you may use the `mergeWhen` method to include the attributes in the response only when the given condition is `true`:
Czasami możesz mieć kilka atrybutów, które powinny być uwzględnione w odpowiedzi na zasoby w oparciu o ten sam warunek. W takim przypadku możesz użyć metody `mergeWhen` w celu włączenia atrybutów do odpowiedzi tylko wtedy, gdy podany warunek to `true`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($this->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

Ponownie, jeśli podany warunek to `false`, atrybuty te zostaną usunięte z odpowiedzi zasobów w całości przed wysłaniem do klienta.

> {note} Metody `mergeWhen` nie należy używać w tablicach, które mieszają klucze łańcuchowe i liczbowe. Ponadto nie należy go używać w tablicach z kluczami numerycznymi, które nie są uporządkowane sekwencyjnie.

<a name="conditional-relationships"></a>
### Conditional Relationships - Relacje warunkowe

Oprócz warunkowo ładujących atrybuty, możesz warunkowo zawierać relacje w odpowiedziach zasobów na podstawie tego, czy relacja została już załadowana do modelu. Dzięki temu kontroler może zdecydować, które relacje powinny zostać załadowane do modelu, a zasoby mogą je łatwo uwzględnić tylko wtedy, gdy zostały załadowane.

Oprócz warunkowo ładujących atrybuty, możesz warunkowo zawierać relacje w odpowiedziach zasobów na podstawie tego, czy relacja została już załadowana do modelu. Dzięki temu kontroler może zdecydować, które relacje powinny zostać załadowane do modelu, a zasoby mogą je łatwo uwzględnić tylko wtedy, gdy zostały załadowane.

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

W tym przykładzie, jeśli relacja nie została załadowana, klucz `posts` zostanie całkowicie usunięty z odpowiedzi na zasób zanim zostanie wysłany do klienta.

#### Conditional Pivot Information - Warunkowe informacje przestawne

Oprócz warunkowego uwzględnienia informacji o relacjach w odpowiedziach na zasoby, można warunkowo dołączyć dane z tabel pośrednich relacji wiele do wielu za pomocą metody `whenPivotLoaded`. Metoda `whenPivotLoaded` przyjmuje nazwę tabeli przestawnej jako swój pierwszy argument. Drugim argumentem powinno być Closure definiujące wartość, która ma zostać zwrócona, jeśli informacje o osiach są dostępne w modelu:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_users', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### Adding Meta Data - Dodawanie danych meta

Niektóre standardy interfejsu JSON API wymagają dodania metadanych do odpowiedzi na zasoby i kolekcje zasobów. To często obejmuje rzeczy takie jak `links` do zasobu lub powiązanych zasobów lub metadane dotyczące samego zasobu. Jeśli chcesz zwrócić dodatkowe metadane dotyczące zasobu, po prostu umieść go w swojej metodzie `toArray`. Na przykład możesz dołączyć informacje "link" podczas transformowania kolekcji zasobów:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

Zwracając dodatkowe metadane z zasobów, nigdy nie musisz się martwić o przypadkowe przesłonięcie kluczy `links` lub `meta`, które są automatycznie dodawane przez Laravel podczas zwracania odpowiedzi w postaci stronicowanej. Wszelkie dodatkowe `links`, które zdefiniujesz, zostaną po prostu scalone z linkami dostarczonymi przez paginator.

#### Top Level Meta Data - Najwyższy poziom danych meta

Czasami możesz chcieć uwzględnić tylko niektóre metadane z odpowiedzią na zasób, jeśli zasób jest zwracany przez najbardziej zewnętrzny zasób. Zazwyczaj obejmuje to meta informacje o odpowiedzi jako całości. Aby zdefiniować te metadane, dodaj metodę `with` do swojej klasy zasobów. Ta metoda powinna zwrócić tablicę metadanych, które będą dołączone do odpowiedzi zasobu tylko wtedy, gdy zasób jest renderowanym zasobem zewnętrznym:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * Get additional data that should be returned with the resource array.
         *
         * @param \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

#### Adding Meta Data When Constructing Resources - Dodawanie danych meta podczas konstruowania zasobów

Możesz także dodać dane najwyższego poziomu podczas tworzenia instancji zasobów na trasie lub kontrolerze. Metoda `additional`, dostępna we wszystkich zasobach, akceptuje tablicę danych, które należy dodać do odpowiedzi zasobu:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## Resource Responses - Odpowiedzi zasobów

Jak już przeczytałeś, zasoby mogą być zwracane bezpośrednio z tras i kontrolerów:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

Czasami jednak konieczne może być dostosowanie wychodzącej odpowiedzi HTTP przed wysłaniem jej do klienta. Są na to dwa sposoby. Najpierw możesz połączyć metodę `response` z zasobem. Ta metoda zwróci instancję `Illuminate\Http\Response`, pozwalającą ci na pełną kontrolę nagłówków odpowiedzi:

    use App\User;
    use App\Http\Resources\UserResource;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

Alternatywnie możesz zdefiniować metodę `withResponse` w samym zasobie. Ta metoda zostanie wywołana, gdy zasób zostanie zwrócony jako najbardziej zewnętrzny zasób w odpowiedzi:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * Customize the outgoing response for the resource.
         *
         * @param  \Illuminate\Http\Request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
