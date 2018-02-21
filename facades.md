# Facades

- [Introduction - Wprowadzenie](#introduction)
- [When To Use Facades - Kiedy używać fasad](#when-to-use-facades)
    - [Facades Vs. Dependency Injection - Fasady kontra wstrzykiwanie zależności](#facades-vs-dependency-injection)
    - [Facades Vs. Helper Functions - Fasady kontra Funkcje pomocnicze](#facades-vs-helper-functions)
- [How Facades Work - Jak działają fasady](#how-facades-work)
- [Real-Time Facades - Fasady w czasie rzeczywistym](#real-time-facades)
- [Facade Class Reference - odniesienia do fasad klas](#facade-class-reference)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Fasady zapewniają "static" (statyczne) interfejsy dla klas, które są dostępne w [service container (kontenerze usług)](/docs/{{version}}/container). Laravel ma wiele fasad, które zapewniają dostęp do prawie wszystkich funkcji Laravel. Fasady Laravel służą jako "static proxies" (statyczni pośrednicy) do leżących u podstaw klas w kontenerze usług, zapewniając korzyści w postaci zwięzłej, wyrazistej składni, przy zachowaniu większej testowalności i elastyczności niż tradycyjne metody statyczne.

Wszystkie fasady Laravel zdefiniowane są w przestrzeni nazw `Illuminate\Support\Facades`. Tak więc możemy łatwo uzyskać dostęp do fasade (elewacji), jak na przykład:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

W całej dokumentacji Laravela, wiele przykładów wykorzystuje fasady do zademonstrowania różnych cech frameworka.

<a name="when-to-use-facades"></a>
## When To Use Facades - Kiedy używać fasad

Fasady mają wiele zalet. Zapewniają one zwięzłą, zapadającą w pamięć składnię, która umożliwia korzystanie z funkcji Laravel bez pamiętania długich nazw klas, które muszą być wstrzykiwane lub konfigurowane ręcznie. Co więcej, ze względu na unikalne wykorzystanie metod dynamicznych PHP, są łatwe do przetestowania.

Należy jednak zachować ostrożność podczas korzystania z fasad. Podstawowym zagrożeniem dla fasad jest scope creep (płyność zakresu) klasy. Ponieważ fasady są tak łatwe w użyciu i nie wymagają wstrzyknięcia, łatwo można pozwolić, aby klasy nadal się rozwijały i używały wielu fasad w jednej klasie. Natomiast za pomocą wstrzykiwania zależności mamy wprost widoczne zależności w konstruktorze i gdy staje się on zbyt duży wiemy, że zakres klasy też staje sie zbyt duży. Tak więc, podczas korzystania z fasad, zwróć szczególną uwagę na wielkość swojej klasy, tak aby jej zakres odpowiedzialności pozostawał wąski.

> {tip} Podczas budowania zewnętrznego pakietu, który współdziała z Laravel-em, zamiast używac fasad lepiej wstrzyknąć [Kontrakty Laravela (Laravel contracts)](/docs/{{version}}/contracts). Ponieważ pakiety są budowane poza samym Laravel, nie będziesz mieć dostępu do pomocników testujących fasady Laravel-a.

<a name="facades-vs-dependency-injection"></a>
### Facades Vs. Dependency Injection - Fasady kontra wstrzykiwanie zależności

Jedną z głównych zalet wstrzykiwania zależności jest możliwość zamiany implementacji wstrzykniętej klasy. Jest to użyteczne podczas testowania, ponieważ można wstrzyknąć pozorny lub pośredniczący kod i stwierdzić, że różne metody zostały wywołane na kodzie pośredniczącym.

Zazwyczaj nie można do takich prawdziwych statycznych metod klas zastosować pozornego lub pośredniczącego kodu. Ponieważ jednak fasady używają metod dynamicznych do zastępowania metod wywolanych obiektów rozwiązanych za pomocą kontenera usług, faktycznie możemy testować fasady tak, jak testowalibyśmy instancję klasy wstrzykiwanej. Na przykład, biorąc pod uwagę następującą klasę Tras:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Możemy napisać następujący test, aby sprawdzić, czy metoda `Cache::get` sostała wywołana z argumentem, którego się spodziewaliśmy:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### Facades Vs. Helper Functions - Fasady kontra Funkcje pomocnicze.

Oprócz fasad, Laravel oferuje wiele funkcji pomocniczych, które umożliwiają wykonywanie typowych zadań, takich jak generowanie widoków, uruchamianie zdarzeń, wysyłanie zadań lub wysyłanie odpowiedzi HTTP. Wiele z tych funkcji pomocniczych pełni tę samą funkcję, co odpowiednia fasada. Na przykład to połączenie fasadowe i połączenie pomocnicze są równoważne:

    return View::make('profile');

    return view('profile');

Nie ma absolutnie żadnej praktycznej różnicy między fasadami i funkcjami pomocniczymi. Korzystając z funkcji pomocnika, możesz je przetestować dokładnie tak samo, jak w przypadku odpowiedniej fasady. Na przykład, biorąc pod uwagę następującą klasę Tras:

    Route::get('/cache', function () {
        return cache('key');
    });

Pod przykrywką pomocnika `cache` bedzie wywoływał metode `get` w klasie umieszczonej pod fasadą `Cache`. Tak więc, mimo że używamy funkcji pomocnika, możemy napisać następujący test, aby sprawdzić, czy metoda została wywołana z argumentem, którego się spodziewaliśmy:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="how-facades-work"></a>
## How Facades Work - Jak działają fasady

W aplikacji Laravel fasada jest klasą zapewniającą dostęp do obiektu z kontenera. Maszyna, która wykonuje tę pracę, znajduje się w klas `Facade`. Fasady Laravel-a i dowolne niestandardowe fasady, które stworzysz, rozszerzą podstawową klasę `Illuminate\Support\Facades\Facade`.

Klasa bazowa `Facade` używa magicznej metody `__callStatic()` aby odroczyć wywołanie z twojej fasady do obiektu rozwiązanego z kontenera. W poniższym przykładzie stworzone jest wywołanie do systemu pamięci podrecznej Laravel-a. Spoglądając na ten kod można założyć, że statyczna metoda `get` jest wywoływana w klasie `Cache`:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Zauważ, że w górnej części pliku "importujemy" fasadę `Cache`. Ta fasada służy jako pośrednik dostępu do podstawowej implementacji interfejsu `Illuminate\Contracts\Cache\Factory`. Wszelkie wywołania wykonywane za pomocą elewacji będą przekazywane do podstawowej instancji usługi pamięci podręcznej Laravel.

Jeśli spojrzymy na klasę `Illuminate\Support\Facades\Cache` zobaczymy, że nie ma metody statycznej `get`:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

Zamiast tego fasada `Cache` rozszerza klasę `Facade` i definiuje metodę `getFacadeAccessor()`. Zadaniem tej metody jest zwrócenie nazwy powiązania kontenera usług. Kiedy użytkownik odwołuje się do dowolnej statycznej metody na elewacji `Cache`, Laravel rozwiąże powiązanie `cache` z [usługi kontenera (service container)](/docs/{{version}}/container) i uruchomi żadaną metodę (w tym przypadku, `get`) wbrew temu obiektowi.

<a name="real-time-facades"></a>
## Real-Time Facades - Fasady w czasie rzeczywistym

Korzystając z fasad w czasie rzeczywistym, możesz traktować dowolną klasę w swojej aplikacji tak, jakby była elewacją. Aby zilustrować, jak można to wykorzystać, przyjrzyjmy się alternatywie. Na przykład załóżmy, że nasz model `Podcast` ma metodę `publish`. Jednakże, aby opublikować podcast, musimy wstrzyknąć instancję  `Publisher`:

    <?php

    namespace App;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @param  Publisher  $publisher
         * @return void
         */
        public function publish(Publisher $publisher)
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

Wstrzyknięcie implementacji wydawcy do metody pozwala nam z łatwością przetestować metodę w izolacji, ponieważ możemy upozorować wstrzykniętego wydawcę. Wymaga to jednak zawsze przesyłania instancji wydawcy za każdym razem, gdy wywołujemy metodę `publish`. Korzystając z fasad działających w czasie rzeczywistym, możemy zachować tę samą testowalność, a jednocześnie nie jest wymagane jawne przekazywanie instancji `Publisher`. Aby wygenerować fasadę w czasie rzeczywistym, należy umieścić przestrzeń nazw importowanej klasy za pomocą `Fasady`:

    <?php

    namespace App;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

Gdy używana jest fasada w czasie rzeczywistym, implementacja wydawcy zostanie rozwiązana poza kontenerem usługi za pomocą fragmentu interfejsu lub nazwy klasy, która pojawi się po przedrostku `Facades`. Podczas testowania możemy użyć wbudowanych pomocników testowania fasad Laravel-a do pozorowanego wywołania tej metody:

    <?php

    namespace Tests\Feature;

    use App\Podcast;
    use Tests\TestCase;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A test example.
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = factory(Podcast::class)->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>
## Facade Class Reference - odniesienia do fasad klas

Poniżej znajdziesz każdą fasadę i jej podstawową klasę. Jest to przydatne narzędzie do szybkiego kopiowania dokumentacji API dla danego katalogu głównego elewacji. W stosownych przypadkach dołączany jest również klucz [service container binding](/docs/{{version}}/container).

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laraveldoc.test/api/5.5/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laraveldoc.test/api/5.5/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laraveldoc.test/api/5.5/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](http://laraveldoc.test/api/5.5/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](http://laraveldoc.test/api/5.5/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](http://laraveldoc.test/api/5.5/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laraveldoc.test/api/5.5/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laraveldoc.test/api/5.5/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laraveldoc.test/api/5.5/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laraveldoc.test/api/5.5/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](http://laraveldoc.test/api/5.5/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laraveldoc.test/api/5.5/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](http://laraveldoc.test/api/5.5/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Logger](http://laraveldoc.test/api/5.5/Illuminate/Log/Logger.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laraveldoc.test/api/5.5/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](http://laraveldoc.test/api/5.5/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](http://laraveldoc.test/api/5.5/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laraveldoc.test/api/5.5/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Queue  |  [Illuminate\Queue\QueueManager](http://laraveldoc.test/api/5.5/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](http://laraveldoc.test/api/5.5/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](http://laraveldoc.test/api/5.5/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](http://laraveldoc.test/api/5.5/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](http://laraveldoc.test/api/5.5/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](http://laraveldoc.test/api/5.5/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](http://laraveldoc.test/api/5.5/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](http://laraveldoc.test/api/5.5/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](http://laraveldoc.test/api/5.5/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](http://laraveldoc.test/api/5.5/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laraveldoc.test/api/5.5/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](http://laraveldoc.test/api/5.5/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](http://laraveldoc.test/api/5.5/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](http://laraveldoc.test/api/5.5/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laraveldoc.test/api/5.5/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laraveldoc.test/api/5.5/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](http://laraveldoc.test/api/5.5/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laraveldoc.test/api/5.5/Illuminate/View/View.html)  |  &nbsp;
