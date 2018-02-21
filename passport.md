# API Authentication (Passport)

- [Introduction - Wprowadzenie](#introduction)
- [Installation - Instalacja](#installation)
    - [Frontend Quickstart - Fronend szybki start](#frontend-quickstart)
    - [Deploying Passport - Wdrażanie Passport](#deploying-passport)
- [Configuration - Konfiguracja](#configuration)
    - [Token Lifetimes - Życia Tokenów](#token-lifetimes)
- [Issuing Access Tokens - Wydawanie tokenów dostępu](#issuing-access-tokens)
    - [Managing Clients - Zarządzanie klientami](#managing-clients)
    - [Requesting Tokens - Żądanie tokenów](#requesting-tokens)
    - [Refreshing Tokens - Odświeżanie tokenów](#refreshing-tokens)
- [Password Grant Tokens - Żeton udzielania hasła](#password-grant-tokens)
    - [Creating A Password Grant Client - Tworzenie żetonu udzielania hasła](#creating-a-password-grant-client)
    - [Requesting Tokens - Żądanie tokenów](#requesting-password-grant-tokens)
    - [Requesting All Scopes - Żądanie wszystkich zakresów](#requesting-all-scopes)
- [Implicit Grant Tokens - Niejawny przyznawane tokeny](#implicit-grant-tokens)
- [Client Credentials Grant Tokens - Żetony Grantów Poświadczających Klienta](#client-credentials-grant-tokens)
- [Personal Access Tokens - Osobiste żetony dostępu](#personal-access-tokens)
    - [Creating A Personal Access Client - Tworzenie osobistego klienta dostępu](#creating-a-personal-access-client)
    - [Managing Personal Access Tokens - Zarządzanie osobistymi tokenami dostępu](#managing-personal-access-tokens)
- [Protecting Routes - Ochrona tras](#protecting-routes)
    - [Via Middleware - Przez oprogramowanie pośredniczące](#via-middleware)
    - [Passing The Access Token  Przekazywanie tokena dostępu](#passing-the-access-token)
- [Token Scopes - Zakresy żetonu](#token-scopes)
    - [Defining Scopes - Definiowanie żetonu](#defining-scopes)
    - [Assigning Scopes To Tokens - Przypisywanie zakresów do tokenów](#assigning-scopes-to-tokens)
    - [Checking Scopes - Sprawdzanie zakresów](#checking-scopes)
- [Consuming Your API With JavaScript - Spożywanie Twojego API za pomocą JavaScript](#consuming-your-api-with-javascript)
- [Events - Zdarzenia](#events)
- [Testing - Testowanie](#testing)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel już ułatwia uwierzytelnianie za pomocą tradycyjnych formularzy logowania, ale co z API? Interfejsy API zazwyczaj używają tokenów do uwierzytelniania użytkowników i nie utrzymują stanu sesji między żądaniami. Laravel sprawia, że uwierzytelnianie za pomocą API jest proste dzięki użyciu Laravel Passport, który zapewnia pełną implementację serwera OAuth2 dla Twojej aplikacji Laravel w ciągu kilku minut. Passport jest zbudowany na serwerze [League OAuth2](https://github.com/thephpleague/oauth2-server) obsługiwanym przez Alex Bilbie.

> {note} W tej dokumentacji założono, że znasz już OAuth2. Jeśli nie wiesz nic o OAuth2, rozważ zapoznanie się z ogólną terminologią i funkcjami OAuth2 przed kontynuowaniem.

<a name="installation"></a>
## Installation - Instalacja

Aby rozpocząć, zainstaluj usługę Passport za pomocą menedżera pakietów Composer:

    composer require laravel/passport

Dostawca usługi Passport rejestruje swój własny katalog migracji bazy danych za pomocą frameworka, więc po zarejestrowaniu dostawcy należy przeprowadzić migrację bazy danych. Migracje Passport utworzą tabele potrzebne aplikacji do przechowywania klientów i tokenów dostępu:

    php artisan migrate

> {note} Jeśli nie zamierzasz korzystać z domyślnych migracji usługi Passport, powinieneś wywołać metodę `Passport::ignoreMigrations` w metodzie `register` swojego `AppServiceProvider`. Możesz wyeksportować domyślne migracje za pomocą `php artisan vendor:publish --tag=passport-migrations`.

Następnie uruchom polecenie `passport:install`. To polecenie utworzy klucze szyfrowania potrzebne do wygenerowania tokenów bezpiecznego dostępu. Ponadto polecenie utworzy klienty "dostęp osobisty(personal access)" i "udzielenie hasła(password grant)", które będą używane do generowania tokenów dostępu:

    php artisan passport:install

Po uruchomieniu tego polecenia dodaj cechę `Laravel\Passport\HasApiTokens` do swojego modelu `App\User`. Ta cecha zapewni kilka metod pomocniczych dla Twojego modelu, które umożliwiają sprawdzanie tokena i zakresów uwierzytelnionego użytkownika:

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

Następnie powinieneś wywołać metodę `Passport::routes` w metodzie `boot` swojego `AuthServiceProvider`. Ta metoda rejestruje trasy niezbędne do wydawania tokenów dostępu i unieważnia tokeny dostępu, klientów i tokeny dostępu osobistego:

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
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
            'App\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

Na koniec, w pliku konfiguracyjnym `config/auth.php`, powinieneś ustawić opcję `driver` dla `api` na `passport`. Spowoduje to, że twoja aplikacja użyje `TokenGuard` Passport-u podczas uwierzytelniania przychodzących żądań API:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="frontend-quickstart"></a>
### Frontend Quickstart - Fronend szybki start

> {note} Aby korzystać z komponentów Passport Vue, musisz używać struktury JavaScript [Vue](https://vuejs.org). Te komponenty również wykorzystują framework Bootstrap CSS. Jednak nawet jeśli nie korzystasz z tych narzędzi, komponenty służą jako cenne odniesienie do Twojej implementacji frontendu.

Usługa Passport jest dostarczana z interfejsem API JSON, którego możesz użyć do umożliwienia użytkownikom tworzenia klientów i tokenów dostępu osobistego. Jednak kodowanie interfejsu użytkownika do interakcji z tymi interfejsami API może być czasochłonne. W związku z tym w usłudze Passport uwzględnione są również gotowe komponenty [Vue](https://vuejs.org), których można użyć jako przykładu wdrożenia lub punktu wyjścia do własnej implementacji.

Aby opublikować komponenty Passport Vue, użyj polecenia `vendor:publish` Artisan:

    php artisan vendor:publish --tag=passport-components

Opublikowane komponenty zostaną umieszczone w katalogu  `resources/assets/js/components`. Po opublikowaniu komponentów należy je zarejestrować w pliku `resources/assets/js/app.js`:

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );

Po zarejestrowaniu komponentów upewnij się, że uruchom `npm run dev`, aby przekompilować swoje zasoby. Po ponownym skompilowaniu zasobów możesz upuścić komponenty do jednego z szablonów aplikacji, aby rozpocząć tworzenie klientów i tokenów dostępu osobistego:

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### Deploying Passport - Wdrażanie Passport

Podczas wdrażania usługi Passport na serwerach produkcyjnych po raz pierwszy najprawdopodobniej konieczne będzie uruchomienie polecenia `passport:keys`. To polecenie generuje klucze szyfrowania Passport potrzebne do wygenerowania tokenu dostępu. Wygenerowane klucze zazwyczaj nie są przechowywane w kontroli źródła:

    php artisan passport:keys

<a name="configuration"></a>
## Configuration - Konfiguracja

<a name="token-lifetimes"></a>
### Token Lifetimes - Życia Tokenów

Domyślnie usługa Passport wydaje żetony dostępu długoterminowego, które wygasają po roku. Jeśli chcesz skonfigurować dłuższy / krótszy okres ważności tokenu, możesz użyć metod `tokensExpireIn` i` refreshTokensExpireIn`. Te metody powinny być wywoływane z metody `boot` twojego` AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(now()->addDays(15));

        Passport::refreshTokensExpireIn(now()->addDays(30));
    }

<a name="issuing-access-tokens"></a>
## Issuing Access Tokens - Wydawanie tokenów dostępu

Używanie protokołu OAuth2 z kodami autoryzacji to sposób, w jaki większość programistów zna OAuth2. Podczas korzystania z kodów autoryzacji aplikacja kliencka przekieruje użytkownika na serwer, gdzie może zatwierdzić lub odrzucić żądanie udostępnienia tokenu dostępu do klienta.

<a name="managing-clients"></a>
### Managing Clients - Zarządzanie klientami

Po pierwsze, programiści tworzący aplikacje, które muszą wchodzić w interakcje z interfejsem API twojej aplikacji, będą musieli zarejestrować swoją aplikację przy pomocy swojego "klienta". Zazwyczaj polega to na podaniu nazwy aplikacji i adresu URL, na który aplikacja może przekierowywać po zatwierdzeniu przez użytkowników ich wniosku o autoryzację.

#### The `passport:client` Command

Najprostszym sposobem na utworzenie klienta jest użycie polecenia `passport:client` Artisan. To polecenie może być używane do tworzenia własnych klientów w celu testowania funkcjonalności OAuth2. Po uruchomieniu polecenia `client` program Passport wyświetli monit o więcej informacji o kliencie i udostępni identyfikator klienta oraz sekret:

    php artisan passport:client

#### JSON API

Ponieważ twoi użytkownicy nie będą mogli korzystać z polecenia `client`, Passport udostępnia interfejs API JSON, którego możesz użyć do tworzenia klientów. Pozwala to zaoszczędzić sobie trudu ręcznego kodowania kontrolerów w celu tworzenia, aktualizowania i usuwania klientów.

Konieczne będzie jednak sparowanie interfejsu JSON API usługi Passport z własnym interfejsem, aby udostępnić panel kontrolny dla użytkowników do zarządzania swoimi klientami. Poniżej opiszemy wszystkie punkty końcowe API do zarządzania klientami. Dla wygody użyjemy [Axios](https://github.com/mzabriskie/axios), aby zademonstrować wysyłanie żądań HTTP do punktów końcowych.

> {tip} Jeśli nie chcesz zaimplementować całego interfejsu klienta do zarządzania, możesz skorzystać z [szybkiego uruchomienia frontendu](#frontend-quickstart), aby w pełni funkcjonalny interfejs uruchomić w ciągu kilku minut.

#### `GET /oauth/clients`

Ta trasa zwraca wszystkich klientów uwierzytelnionego użytkownika. Jest to przydatne przede wszystkim do wyświetlania wszystkich klientów użytkownika, aby mogli je edytować lub usunąć:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

Ta trasa służy do tworzenia nowych klientów. Wymaga dwóch danych: adresu URL `name` i `redirect`. Adres URL `redirect` to miejsce, w którym użytkownik zostanie przekierowany po zatwierdzeniu lub odrzuceniu wniosku o autoryzację.

Po utworzeniu klienta zostanie mu nadany identyfikator klienta i tajny klucz klienta. Te wartości będą używane przy żądaniu tokenów dostępu z twojej aplikacji. Trasa tworzenia klienta zwróci nową instancję klienta:

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `PUT /oauth/clients/{client-id}`

Ta trasa służy do aktualizacji klientów. Wymaga dwóch danych: adresu URL `name` i `redirect`. Adres URL `redirect` to miejsce, w którym użytkownik zostanie przekierowany po zatwierdzeniu lub odrzuceniu wniosku o autoryzację. Trasa zwróci zaktualizowaną instancję klienta:

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/clients/{client-id}`

Ta trasa służy do usuwania klientów:

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### Requesting Tokens - Żądanie tokenów

#### Redirecting For Authorization - Przekierowanie w celu autoryzacji

Po utworzeniu klienta programiści mogą używać swojego identyfikatora klienta i hasła, aby zażądać kodu autoryzacji i dostępu do tokena z aplikacji. Po pierwsze aplikacja pobierająca powinna przekierować żądanie do trasy `/oauth/authorize` aplikacji:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Pamiętaj, że trasa `/oauth/authorize` jest już zdefiniowana metodą  `Passport::routes`. Nie musisz ręcznie definiować tej trasy.

#### Approving The Request - Zatwierdzenie wniosku

Podczas odbierania wniosków o autoryzację, Passport automatycznie wyświetli szablon dla użytkownika, umożliwiając mu zatwierdzenie lub odrzucenie żądania autoryzacji. Jeśli zatwierdzą żądanie, zostaną przekierowani z powrotem do `redirect_uri`, który został określony przez aplikację korzystającą. `redirect_uri` musi pasować do adresu URL `redirect`, który został określony podczas tworzenia klienta.

Jeśli chcesz dostosować ekran zatwierdzania autoryzacji, możesz opublikować widoki usługi Passport za pomocą komendy `vendor:publish` Artisan. Opublikowane widoki zostaną umieszczone w  `resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views

#### Converting Authorization Codes To Access Tokens - Konwersja kodów autoryzacyjnych w celu uzyskania dostępu do tokenów

Jeśli użytkownik zatwierdzi żądanie autoryzacji, zostanie przekierowany z powrotem do aplikacji korzystającej. Konsument powinien następnie wysłać do aplikacji żądanie `POST`, aby poprosić o token dostępu. Żądanie powinno zawierać kod autoryzacji, który został wydany przez aplikację po zatwierdzeniu przez użytkownika żądania autoryzacji. W tym przykładzie użyjemy biblioteki HTTP Guzzle do wykonania żądania `POST`:

    Route::get('/callback', function (Request $request) {
        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

Ta trasa `/oauth/token` zwraca odpowiedź JSON zawierającą atrybuty `access_token`, `refresh_token` i `expires_in`. Atrybut `expires_in` zawiera liczbę sekund do wygaśnięcia tokena dostępu.

> {tip} Podobnie jak w przypadku trasy `/oauth/authorize`, trasa `/oauth/token` jest zdefiniowana dla ciebie metodą `Passport::routes`. Nie ma potrzeby ręcznego definiowania tej trasy.

<a name="refreshing-tokens"></a>
### Refreshing Tokens - Odświeżanie tokenów

Jeśli twoja aplikacja wyda żetony dostępu krótkotrwałego, użytkownicy będą musieli odświeżyć swoje tokeny dostępu za pomocą odświeżającego tokena, który został im dostarczony, gdy wydano token dostępu. W tym przykładzie użyjemy biblioteki HTTP Guzzle do odświeżenia tokena:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

Ta trasa `/oauth/token` zwraca odpowiedź JSON zawierającą atrybuty `access_token`, `refresh_token` i `expires_in`. Atrybut `expires_in` zawiera liczbę sekund do wygaśnięcia tokena dostępu.

<a name="password-grant-tokens"></a>
## Password Grant Tokens - Żeton udzielania hasła

Przydzielenie hasła OAuth2 umożliwia innym klientom, takim jak aplikacja mobilna, uzyskanie tokenu dostępu za pomocą adresu e-mail / nazwy użytkownika i hasła. Dzięki temu możesz bezpiecznie wystawiać tokeny swoim klientom, nie wymagając od użytkowników przejścia przez cały proces przekierowania kodu autoryzacji OAuth2.

<a name="creating-a-password-grant-client"></a>
### Creating A Password Grant Client - Tworzenie żetonu udzielania hasła

Zanim aplikacja będzie mogła wydać tokeny poprzez przyznanie hasła, musisz utworzyć klienta przydziału haseł. Możesz to zrobić za pomocą polecenia `paszport:client` z opcją` --password`. Jeśli uruchomiłeś już polecenie `passport:install`, nie musisz uruchamiać tego polecenia:

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### Requesting Tokens - Żądanie tokenów

Po utworzeniu klienta do przydzielania haseł możesz zażądać tokenu dostępu, wysyłając żądanie `POST` do trasy `/oauth/token` za pomocą adresu e-mail użytkownika i hasła. Pamiętaj, że ta trasa jest już zarejestrowana metodą `Passport::routes`, więc nie ma potrzeby definiowania jej ręcznie. Jeśli żądanie zakończy się powodzeniem, otrzymasz od serwera odpowiedź: `access_token` oraz` refresh_token` w odpowiedzi JSON:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} Pamiętaj, że tokeny dostępu są domyślnie długie. Możesz jednak [skonfigurować maksymalny czas życia tokena dostępu](#configuration), jeśli to konieczne.

<a name="requesting-all-scopes"></a>
### Requesting All Scopes - Żądanie wszystkich zakresów

Podczas korzystania z przyznawania haseł możesz chcieć autoryzować token dla wszystkich zakresów obsługiwanych przez aplikację. Możesz to zrobić, żądając zakresu `*`. Jeśli zażądasz zakresu `*`, metoda `can` na instancji tokena zawsze zwróci wartość `true`. Ten zakres może być przypisany tylko do tokena wystawionego przy użyciu przydziału `password`:

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="implicit-grant-tokens"></a>
## Implicit Grant Tokens - Niejawny przyznawane tokeny

Niejawna dotacja jest podobna do przyznania kodu autoryzacyjnego; jednak token jest zwracany klientowi bez wymiany kodu autoryzacyjnego. Dotacja jest najczęściej używana w JavaScript lub aplikacjach mobilnych, w których poświadczenia klienta nie mogą być bezpiecznie przechowywane. Aby włączyć akceptację, wywołaj metodę `enableImplicitGrant` w swoim` AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

Po włączeniu grantu programiści mogą używać swojego identyfikatora klienta do żądania tokenu dostępu z aplikacji. Aplikacja zużywająca powinna przekierować żądanie do trasy `/oauth/authorize` aplikacji:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Pamiętaj, że trasa `/oauth/authorize` jest już zdefiniowana metodą `Passport::routes`. Nie musisz ręcznie definiować tej trasy.

<a name="client-credentials-grant-tokens"></a>
## Client Credentials Grant Tokens - Żetony Grantów Poświadczających Klienta

Przypisanie poświadczeń klienta jest odpowiednie dla uwierzytelniania maszyna-maszyna. Na przykład można użyć tego przydziału w zaplanowanym zadaniu, które wykonuje zadania konserwacyjne za pośrednictwem interfejsu API. Aby skorzystać z tej metody, musisz najpierw dodać nowe oprogramowanie pośrednie do `$routeMiddleware` w `app/Http/Kernel.php`:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

Następnie podłącz to oprogramowanie pośrednie do trasy:

    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');

Aby pobrać token, wprowadź żądanie do punktu końcowego `oauth/token`:

    $guzzle = new GuzzleHttp\Client;

    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);

    return json_decode((string) $response->getBody(), true)['access_token'];

<a name="personal-access-tokens"></a>
## Personal Access Tokens - Osobiste żetony dostępu

Czasami Twoi użytkownicy mogą chcieć wydać tokenom dostęp do nich bez przechodzenia przez typowy przepływ przekierowania kodu autoryzacyjnego. Zezwalanie użytkownikom na wydawanie tokenów sobie za pośrednictwem interfejsu użytkownika aplikacji może być przydatne dla umożliwienia użytkownikom eksperymentowania z interfejsem API lub może służyć jako prostsze podejście do wydawania tokenów dostępu w ogóle.

> {note} Osobiste żetony dostępu są zawsze długożyciowe. Ich żywotność nie jest modyfikowana podczas używania metod `tokensExpireIn` lub` refreshTokensExpireIn`.

<a name="creating-a-personal-access-client"></a>
### Creating A Personal Access Client - Tworzenie osobistego klienta dostępu

Zanim aplikacja będzie mogła wydawać osobiste tokeny dostępu, musisz utworzyć osobistego klienta dostępu. Możesz to zrobić za pomocą polecenia `paszport:client` z opcją `--personal`. Jeśli uruchomiłeś już polecenie `passport:install`, nie musisz uruchamiać tego polecenia:

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### Managing Personal Access Tokens - Zarządzanie osobistymi tokenami dostępu

Po utworzeniu osobistego klienta dostępu możesz wydać tokeny dla danego użytkownika, używając metody `createToken` w instancji modelu `User`. Metoda `createToken` akceptuje nazwę tokenu jako swój pierwszy argument i opcjonalną tablicę [zakresów](#token-scopes) jako swojego drugiego argumentu:

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport zawiera również interfejs API JSON do zarządzania osobistymi tokenami dostępu. Możesz sparować to z własnym frontendem, aby zaoferować swoim użytkownikom pulpit do zarządzania osobistymi tokenami dostępu. Poniżej opiszemy wszystkie punkty końcowe API do zarządzania osobistymi tokenami dostępu. Dla wygody użyjemy [Axios](https://github.com/mzabriskie/axios), aby zademonstrować wysyłanie żądań HTTP do punktów końcowych.

> {tip} Jeśli nie chcesz samodzielnie implementować osobnego tokenu dostępu do tokena, możesz skorzystać z szybkiego [frontend szybki start](#frontend-quickstart), aby w pełni funkcjonalny interfejs uruchomić w ciągu kilku minut.

#### `GET /oauth/scopes`

Ta trasa zwraca wszystkie [zakresy](#token-scopes) zdefiniowane dla twojej aplikacji. Możesz użyć tej trasy, aby wyświetlić listę zakresów, które użytkownik może przypisać do osobistego tokena dostępu:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

Ta trasa zwraca wszystkie osobiste tokeny dostępu, które utworzył uwierzytelniony użytkownik. Jest to przydatne przede wszystkim w przypadku umieszczania na liście wszystkich tokenów użytkownika, które mogą edytować lub usuwać:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

Ta trasa tworzy nowe osobiste żetony dostępu. Wymaga dwóch danych: token `name` i `scopes`, które powinny zostać przypisane do tokena:

    const data = {
        name: 'Token Name',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

Ta trasa może zostać wykorzystana do usunięcia osobistych tokenów dostępu:

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## Protecting Routes - Ochrona tras

<a name="via-middleware"></a>
### Via Middleware - Przez oprogramowanie pośredniczące

Paszport zawiera [strażnik-a uwierzytelniania](/docs/{{version}}/authentication#adding-custom-guards), który będzie sprawdzać tokeny dostępu na przychodzących żądaniach. Po skonfigurowaniu aplikacji `api` strażnika do korzystania ze sterownika `passport`, musisz tylko określić oprogramowanie pośredniczące `auth:api` na dowolnych trasach, które wymagają poprawnego tokena dostępu:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### Passing The Access Token - Przekazywanie tokena dostępu

Podczas nawiązywania połączeń chronionych za pomocą usługi Passport, użytkownicy interfejsu API aplikacji powinni określić swój token dostępu jako token  `Bearer` (na okaziciela) w nagłówku `Authorization` swojego żądania. Na przykład podczas korzystania z biblioteki HTTP Guzzle:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Token Scopes - Zakresy żetonu

<a name="defining-scopes"></a>
### Defining Scopes - Definiowanie żetonu

Zakresy umożliwiają Twoim klientom API żądanie określonego zestawu uprawnień przy żądaniu autoryzacji dostępu do konta. Na przykład, jeśli tworzysz aplikację handlu elektronicznego, nie wszyscy konsumenci API będą potrzebować możliwości składania zamówień. Zamiast tego możesz zezwolić konsumentom tylko na żądanie autoryzacji, aby uzyskać dostęp do statusów przesyłek zamówień. Innymi słowy, zakresy pozwalają użytkownikom aplikacji ograniczyć działanie, które aplikacja zewnętrzna może wykonywać w ich imieniu.

Możesz zdefiniować zakresy swojego API za pomocą metody `Passport::tokensCan` w metodzie` boot` swojego `AuthServiceProvider`. Metoda `tokensCan` akceptuje tablicę nazw zakresów i opisów zakresów. Opis zakresu może być dowolnie wybrany i będzie wyświetlany użytkownikom na ekranie zatwierdzania autoryzacji:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### Assigning Scopes To Tokens - Przypisywanie zakresów do tokenów

#### When Requesting Authorization Codes - Podczas żądania kodów autoryzacyjnych

Żądając tokenu dostępu przy użyciu przyznawania kodu autoryzacji, konsumenci powinni określać swoje pożądane zakresy jako parametr ciągu zapytania `scope`. W parametrze `scope`  lista zakresów powinien być rozdzieloną spacjami:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### When Issuing Personal Access Tokens - Podczas wydawania osobistych tokenów dostępu

Jeśli wydajesz osobiste tokeny dostępu za pomocą metody `createToken` modelu  `User`, możesz przekazać tablicę żądanych zakresów jako drugi argument metody:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Checking Scopes - Sprawdzanie zakresów

Paszport zawiera dwa oprogramowania pośrednie, które mogą być używane do sprawdzania, czy przychodzące żądanie jest uwierzytelniane za pomocą tokena,a któremu przyznano dany zakres. Aby rozpocząć, dodaj następujące oprogramowanie pośrednie do właściwości `$routeMiddleware` pliku `app/Http/Kernel.php`:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### Check For All Scopes - Sprawdź wszystkie zakresy

Program pośredniczący `scopes` może zostać przypisany do trasy w celu sprawdzenia, czy token dostępu żądania przychodzącego ma *wszystkie* wymienione zakresy:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware('scopes:check-status,place-orders');

#### Check For Any Scopes - Sprawdź dla dowolnych zakresów

Program pośredniczący `scope` może zostać przypisany do trasy w celu sprawdzenia, czy token dostępu żądania przychodzącego ma *co najmniej jeden* z wymienionych zakresów:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');

#### Checking Scopes On A Token Instance - Sprawdzanie zakresów na wystąpieniu tokena

Po wprowadzeniu do aplikacji żądania uwierzytelnionego tokena dostępu możesz nadal sprawdzić, czy token ma określony zakres, używając metody `tokenCan` na uwierzytelnionej instancji `User`:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## Consuming Your API With JavaScript - Spożywanie Twojego API za pomocą JavaScript

Podczas tworzenia interfejsu API niezwykle przydatna może być możliwość korzystania z własnego API z aplikacji JavaScript. Takie podejście do rozwoju API pozwala Twojej własnej aplikacji korzystać z tego samego API, z którego dzielisz się ze światem. To samo API może być używane przez twoją aplikację internetową, aplikacje mobilne, aplikacje innych firm i wszelkie pakiety SDK, które możesz publikować na różnych menedżerach pakietów.

Zwykle, jeśli chcesz korzystać z interfejsu API z aplikacji JavaScript, musisz ręcznie przesłać token dostępu do aplikacji i przekazać go przy każdym żądaniu do aplikacji. Jednak usługa Passport obejmuje oprogramowanie pośrednie, które może obsłużyć to za Ciebie. Wszystko, co musisz zrobić, to dodać oprogramowanie pośredniczące `CreateFreshApiToken` do `web` swojej grupy oprogramowania pośredniego w pliku `app/Http/Kernel.php`:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

To oprogramowanie pośredniczące do usługi Passport dołączy do ciastek plik cookie `laravel_token` na Twoje odpowiedzi wychodzące. Ten plik cookie zawiera zaszyfrowaną ścieżkę JWT, której usługa Passport będzie używać do uwierzytelniania żądań interfejsu API z aplikacji JavaScript. Teraz możesz wysyłać żądania do interfejsu API aplikacji bez jawnego przekazywania tokena dostępu:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

Podczas korzystania z tej metody uwierzytelniania domyślne rusztowanie Laravel JavaScript nakazuje Axiosowi wysyłanie zawsze nagłówków `X-CSRF-TOKEN` i `X-Requested-With`. Należy jednak pamiętać o umieszczeniu tokenu CSRF w [metatagu HTML](/docs/{{version}}/csrf#csrf-x-csrf-token):

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };

> {note} Jeśli używasz innej architektury JavaScript, powinieneś upewnić się, że jest skonfigurowana do wysyłania nagłówków `X-CSRF-TOKEN` i `X-Requested-With` przy każdym wychodzącym żądaniu.

<a name="events"></a>
## Events - Zdarzenia

Paszport podnosi zdarzenia podczas wydawania żetonów dostępu i odświeżania tokenów. Możesz wykorzystać te zdarzenia do usunięcia lub unieważnienia innych tokenów dostępu w bazie danych. Możesz dołączyć detektory do tych zdarzeń w `EventServiceProvider` aplikacji:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Events\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Events\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];
```

<a name="testing"></a>
## Testing - Testowanie

Metoda `actingAs` Passport-u może być używana do określenia aktualnie uwierzytelnionego użytkownika, a także jego zakresów. Pierwszym argumentem przypisanym do metody `actingAs` jest instancja użytkownika, a druga to tablica zakresów, które powinny zostać nadane tokenowi użytkownika:

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(200);
    }