# HTTP Requests

- [Accessing The Request - Dostęp do żądania](#accessing-the-request)
    - [Request Path & Method - ścieżka żadania i metody](#request-path-and-method)
    - [PSR-7 Requests - Żądania PSR-7](#psr7-requests)
- [Input Trimming & Normalization - Przycinanie wejścia i normalizacja](#input-trimming-and-normalization)
- [Retrieving Input - Pobieranie danych wejściowych](#retrieving-input)
    - [Old Input - Stare wejście](#old-input)
    - [Cookies - Ciasteczka](#cookies)
- [Files - Pliki](#files)
    - [Retrieving Uploaded Files - Uzyskiwanie pobranych plików](#retrieving-uploaded-files)
    - [Storing Uploaded Files - Przechowywanie przesłanych plików](#storing-uploaded-files)
- [Configuring Trusted Proxies - Konfigurowanie zaufanych serwerów proxy](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>
## Accessing The Request - Dostęp do żądania

Aby uzyskać wystąpienie bieżącego żądania HTTP przez wstrzyknięcie zależności, należy wpisać wskazówkę klasy `Illuminate\Http\Request` na swojej metodzie sterownika. Instancja przychodzącego żądania zostanie automatycznie wprowadzona przez [kontener usług](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

#### Dependency Injection & Route Parameters - Wstrzykiwanie zależności & Parametry trasy

Jeśli twoja metoda kontrolera oczekuje również wprowadzenia z parametru trasy, powinieneś podać parametry swojej trasy po innych zależnościach. Na przykład, jeśli twoja trasa jest zdefiniowana w następujący sposób:

    Route::put('user/{id}', 'UserController@update');

Nadal możesz wpisać wskazówkę `Illuminate\Http\Request` i uzyskać dostęp do parametru trasy `id`, definiując swoją metodę kontrolera w następujący sposób:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

#### Accessing The Request Via Route Closures - Uzyskiwanie dostępu do żądania za pośrednictwem Closures tras

Możesz również wpisać wskazówki klasy `Illuminate\Http\Request` na Closure trasy. Kontener usług automatycznie wstrzykuje przychodzące żądanie do Closure po jego wykonaniu:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>
### Request Path & Method - ścieżka żadania i metody

Instancja `Illuminate\Http\Request` udostępnia różne metody sprawdzania żądania HTTP dla aplikacji i rozszerza klasę `Symfony\Component\HttpFoundation\Request`. Omówimy kilka najważniejszych metod poniżej.

#### Retrieving The Request Path - Pobieranie ścieżki żądania

Metoda `path` zwraca informacje o ścieżce żądania. Tak więc, jeśli przychodzące żądanie jest kierowane na `http://domain.com/foo/bar`, metoda` path` zwróci `foo/bar`:

    $uri = $request->path();

Metoda `is` pozwala sprawdzić, czy ścieżka przychodzącego żądania pasuje do danego wzorca. Podczas korzystania z tej metody możesz użyć znaku `*` jako symbolu wieloznacznego:

    if ($request->is('admin/*')) {
        //
    }

#### Retrieving The Request URL - Pobieranie adresu URL żądania

Aby pobrać pełny adres URL dla przychodzącego żądania, możesz użyć metod `url` lub` fullUrl`. Metoda `url` zwróci adres URL bez ciągu zapytania, natomiast metoda` fullUrl` zawiera ciąg zapytania:

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

#### Retrieving The Request Method - Pobieranie metody żądania

Metoda `method` zwróci akcje HTTP dla żądania. Możesz użyć metody `isMethod`, aby sprawdzić, czy akcja HTTP pasuje do danego ciągu znaków:

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### PSR-7 Requests - Żądania PSR-7

[Standard PSR-7](http://www.php-fig.org/psr/psr-7/) określa interfejsy dla komunikatów HTTP, w tym żądania i odpowiedzi. Jeśli chcesz uzyskać wystąpienie żądania PSR-7 zamiast żądania Laravel, najpierw musisz zainstalować kilka bibliotek. Laravel używa komponentu *Symfony HTTP Message Bridge* do konwersji typowych żądań i odpowiedzi Laravel do implementacji zgodnych z PSR-7:

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros

Po zainstalowaniu tych bibliotek, możesz otrzymać żądanie PSR-7, wpisując w interfejsie żądania na swojej trasie Closure lub metodę kontrolera:

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} Jeśli zwrócisz instancję odpowiedzi PSR-7 z trasy lub kontrolera, zostanie ona automatycznie przekonwertowana z powrotem na instancję odpowiedzi Laravel i zostanie wyświetlona przez framework.

<a name="input-trimming-and-normalization"></a>
## Input Trimming & Normalization - Przycinanie wejścia i normalizacja

Domyślnie Laravel zawiera oprogramowanie pośredniczące `TrimStrings` i `ConvertEmptyStringsToNull` w globalnym stosie oprogramowania pośredniego twojej aplikacji. Te oprogramowanie pośrednie są wymienione w stosie przez klasę `App\Http\Kernel`. Te programy pośredniczące automatycznie usuwają wszystkie przychodzące pola tekstowe na żądanie, a także konwertują dowolne puste pola tekstowe na `null`. Dzięki temu nie musisz martwić się o problemy związane z normalizacją na trasach i kontrolerach.

Jeśli chcesz wyłączyć to zachowanie, możesz usunąć dwa oprogramowanie pośrednie ze stosu oprogramowania pośredniego aplikacji, usuwając je z właściwości `$middleware` klasy `App\Http\Kernel`.

<a name="retrieving-input"></a>
## Retrieving Input - Pobieranie danych wejściowych

#### Retrieving All Input Data - Pobieranie wszystkich danych wejściowych

Możesz także pobrać wszystkie dane wejściowe jako `array` używając metody` all`:

    $input = $request->all();

#### Retrieving An Input Value - Pobieranie wartości wejściowej

Używając kilku prostych metod, możesz uzyskać dostęp do wszystkich danych wejściowych użytkownika z instancji `Illuminate\Http\Request`, nie martwiąc się, który akcje HTTP został użyty dla żądania. Bez względu na akcje HTTP, metoda `input` może być użyta do pobrania danych wejściowych użytkownika:

    $name = $request->input('name');

Możesz przekazać wartość domyślną jako drugi argument metody `input`. Ta wartość zostanie zwrócona, jeśli żądana wartość wejściowa nie jest obecna w żądaniu:

    $name = $request->input('name', 'Sally');

Podczas pracy z formularzami, które zawierają wejścia tablicowe, należy użyć zapisu "kropka", aby uzyskać dostęp do tablic:

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### Retrieving Input From The Query String - Pobieranie danych wejściowych z ciągu zapytania

Podczas gdy metoda `input` pobiera wartości z całego ładunku żądania (w tym ciągu zapytania), metoda` query` pobiera tylko wartości z ciągu zapytania:

    $name = $request->query('name');

Jeśli żądane dane wartości ciągu zapytania nie są obecne, zostanie zwrócony drugi argument tej metody:

    $name = $request->query('name', 'Helen');

Możesz wywołać metodę `query` bez żadnych argumentów, aby pobrać wszystkie wartości ciągu zapytania jako tablicę asocjacyjną:

    $query = $request->query();

#### Retrieving Input Via Dynamic Properties - Pobieranie danych wejściowych za pośrednictwem właściwości dynamicznych

Możesz również uzyskać dostęp do danych wprowadzanych przez użytkownika za pomocą właściwości dynamicznych w instancji `Illuminate\Http\Request`. Na przykład, jeśli jeden z formularzy aplikacji zawiera pole `name`, możesz uzyskać dostęp do wartości pola w następujący sposób:

    $name = $request->name;

Podczas korzystania z właściwości dynamicznych Laravel najpierw szuka wartości parametru w ładunku żądania. Jeśli nie jest obecny, Laravel wyszuka pole w parametrach trasy.

#### Retrieving JSON Input Values - Pobieranie wartości wejściowych JSON

Wysyłając żądania JSON do aplikacji, możesz uzyskać dostęp do danych JSON za pomocą metody `input`, o ile nagłówek `Content-Type` jest prawidłowo ustawiony na `application/json`. Możesz nawet użyć składni "kropka", aby zagłębić się w tablice JSON:

    $name = $request->input('user.name');

#### Retrieving A Portion Of The Input Data - Pobieranie części danych wejściowych

Jeśli potrzebujesz pobrać podzbiór danych wejściowych, możesz użyć metod `only` i` except`. Obie te metody akceptują pojedynczą tablicę `array` lub dynamiczną listę argumentów:

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> {tip} Metoda `only` zwraca wszystkie żądane pary klucz / wartość; jednak nie będzie zwracać par klucz / wartość, które nie są obecne w żądaniu.

#### Determining If An Input Value Is Present - Określanie, czy wartość wejściowa jest obecna

Powinieneś użyć metody `has`, aby określić, czy wartość jest obecna w żądaniu. Metoda `has` zwraca` true`, jeśli wartość jest obecna na żądanie:

    if ($request->has('name')) {
        //
    }

Gdy podana zostanie tablica, metoda `has` określi, czy wszystkie podane wartości są obecne:

    if ($request->has(['name', 'email'])) {
        //
    }

Jeśli chcesz określić, czy wartość jest obecna na żądanie i nie jest pusta, możesz użyć metody `filled`:

    if ($request->filled('name')) {
        //
    }

<a name="old-input"></a>
### Old Input - Stare wejście

Laravel pozwala zachować dane wejściowe z jednego żądania podczas następnego żądania. Ta funkcja jest szczególnie przydatna do ponownego wypełniania formularzy po wykryciu błędów sprawdzania poprawności. Jeśli jednak używasz Laravel w opcji [funkcje sprawdzania poprawności](/docs/{{version}}/validation), jest mało prawdopodobne, że będziesz musiał ręcznie korzystać z tych metod, ponieważ niektóre z wbudowanych funkcji sprawdzania autentyczności Laravel będą je automatycznie wywoływać .

#### Flashing Input To The Session - Migawka danych wejściowych do sesji

Metoda `flash` na klasie `Illuminate\Http\Request` wyświetli bieżące dane wejściowe do [sesji](/docs/{{version}}/session), dzięki czemu będzie dostępna podczas następnego żądania użytkownika do aplikacji :

    $request->flash();

Możesz także użyć metod `flashOnly` i `flashExcept` do flashowania podzbioru danych żądania do sesji. Metody te są przydatne do przechowywania poufnych informacji, takich jak hasła z sesji:

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### Flashing Input Then Redirecting - igawka danych wejściowych, a póżniej przekierowanie

Ponieważ często będziesz chciał wczytać dane wejściowe do sesji, a następnie przekierować na poprzednią stronę, możesz łatwo włączyć flashowanie wejścia na przekierowanie za pomocą metody `withInput`:

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

#### Retrieving Old Input - Pobieranie starego wejścia

Aby pobrać dane z poprzedniego żądania, użyj metody `old` w instancji `Request`. Metoda `old` spowoduje pobranie wcześniej przesłanych danych wejściowych z [sesji](/docs/{{version}}/session):

    $username = $request->old('username');

Laravel zapewnia także globalnego `old` pomocnika. Jeśli wyświetlasz stare dane wejściowe w [szablonie Blade](/docs/{{version}}/blade), wygodniej jest użyć `old` pomocnika. Jeśli dla danego pola nie istnieje stare dane wejściowe, zwracana będzie wartość "null":

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Cookies - Ciasteczka

#### Retrieving Cookies From Requests - Pobieranie Ciasteczek z żądania.

Wszystkie pliki cookie utworzone przez frameworki Laravel są szyfrowane i podpisywane za pomocą kodu uwierzytelniającego, co oznacza, że zostaną uznane za nieważne, jeśli zostały zmienione przez klienta. Aby pobrać wartość cookie z żądania, użyj metody `cookie` na instancji `Illuminate\Http\Request`:

    $value = $request->cookie('name');

Alternatywnie możesz użyć elewacji `Cookie`, aby uzyskać dostęp do wartości plików cookie:

    $value = Cookie::get('name');

#### Attaching Cookies To Responses - Dołączanie plików cookie do odpowiedzi

Możesz dołączyć plik cookie do wychodzącej instancji `Illuminate\Http\Response` za pomocą metody `cookie`. Należy podać nazwę, wartość i liczbę minut, które ciasteczko powinno być uznane za ważne dla tej metody:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

Metoda `cookie` przyjmuje również kilka argumentów, które są rzadziej używane. Ogólnie rzecz biorąc, te argumenty mają ten sam cel i znaczenie, co argumenty, które zostaną podane do natywnej metody [setcookie](https://secure.php.net/manual/en/function.setcookie.php) PHP:

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

Możesz też użyć fasady `Cookie` do umieszczenia w "kolejce" plików cookie w celu dołączenia do odpowiedzi wychodzącej z aplikacji. Metoda `queue` akceptuje instancję `Cookie` lub argumenty potrzebne do utworzenia instancji `Cookie`. Te ciasteczka zostaną dołączone do odpowiedzi wychodzącej, zanim zostaną przesłane do przeglądarki:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

#### Generating Cookie Instances - Generowanie ciastek

Jeśli chcesz wygenerować instancję `Symfony\Component\HttpFoundation\Cookie`, która może zostać przekazana do instancji odpowiedzi w późniejszym czasie, możesz użyć globalnego helpera `cookie`. Ten plik cookie nie zostanie odesłany do klienta, chyba że jest dołączony do instancji odpowiedzi:

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="files"></a>
## Files - Pliki

<a name="retrieving-uploaded-files"></a>
### Retrieving Uploaded Files - Uzyskiwanie pobranych plików

Możesz uzyskać dostęp do przesłanych plików z instancji `Illuminate\Http\Request` za pomocą metody `file` lub używając właściwości dynamicznych. Metoda `file` zwraca instancję klasy `Illuminate\Http\UploadedFile`, która rozszerza klasę PHP `SplFileInfo` i udostępnia różne metody interakcji z plikiem:

    $file = $request->file('photo');

    $file = $request->photo;

Możesz określić, czy plik jest obecny w żądaniu, używając metody `hasFile`:

    if ($request->hasFile('photo')) {
        //
    }

#### Validating Successful Uploads - Sprawdzanie poprawności przesłanych plików

Oprócz sprawdzenia, czy plik jest obecny, możesz sprawdzić, czy nie wystąpiły problemy z przesłaniem pliku za pomocą metody `isValid`:

    if ($request->file('photo')->isValid()) {
        //
    }

#### File Paths & Extensions - Ścieżki plików i rozszerzenia

Klasa `UploadedFile` zawiera również metody dostępu do pełnej ścieżki pliku i jego rozszerzenia. Metoda `extension` spróbuje odgadnąć rozszerzenie pliku na podstawie jego zawartości. To rozszerzenie może być inne niż rozszerzenie dostarczone przez klienta:

    $path = $request->photo->path();

    $extension = $request->photo->extension();

#### Other File Methods - Inne metody plików

W instancjach `UploadedFile` dostępnych jest wiele innych metod. Sprawdź [dokumentację API dla klasy](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html), aby uzyskać więcej informacji na temat tych metod.

<a name="storing-uploaded-files"></a>
### Storing Uploaded Files - Przechowywanie przesłanych plików

Aby zapisać przesłany plik, zwykle używasz jednego ze skonfigurowanych [systemów plików](/docs/{{version}}/filesystem). Klasa `UploadedFile` ma metodę `store`, która przenosi przesłany plik na jeden z dysków, który może być lokalizacją w lokalnym systemie plików lub nawet w miejscu przechowywania w chmurze, takim jak Amazon S3.

Metoda `store` akceptuje ścieżkę, w której plik powinien być przechowywany względem skonfigurowanego katalogu głównego systemu plików. Ta ścieżka nie powinna zawierać nazwy pliku, ponieważ unikalny identyfikator zostanie automatycznie wygenerowany, aby służyć jako nazwa pliku.

Metoda `store` przyjmuje również opcjonalny drugi argument dla nazwy dysku, który powinien zostać użyty do przechowywania pliku. Metoda zwróci ścieżkę pliku względem katalogu głównego dysku:

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

Jeśli nie chcesz, aby nazwa pliku była generowana automatycznie, możesz użyć metody `storeAs`, która akceptuje ścieżkę, nazwę pliku i nazwę dysku jako swoje argumenty:

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

<a name="configuring-trusted-proxies"></a>
## Configuring Trusted Proxies - Konfigurowanie zaufanych serwerów proxy

Podczas uruchamiania aplikacji za modułem równoważenia obciążenia, który nadaje certyfikaty TLS / SSL, możesz zauważyć, że aplikacja czasami nie generuje linków HTTPS. Zazwyczaj dzieje się tak dlatego, że Twoja aplikacja jest przekierowywana z systemu równoważenia obciążenia na porcie 80 i nie wie, że powinna generować bezpieczne linki.

Aby rozwiązać ten problem, można użyć oprogramowania pośredniego `App\Http\Middleware\TrustProxies`, które jest zawarte w aplikacji Laravel, co pozwala szybko dostosować ustawienia równoważenia obciążenia lub serwery proxy, które powinny być zaufane przez aplikację. Twój zaufany serwer proxy powinien być wymieniony jako tablica we właściwości  `$proxies` tego oprogramowania pośredniego. Oprócz konfiguracji zaufanych serwerów proxy możesz skonfigurować `$headers`, które powinno być zaufane:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * The headers that should be used to detect proxies.
         *
         * @var string
         */
        protected $headers = Request::HEADER_X_FORWARDED_ALL;
    }

> {tip} Jeśli korzystasz z elastycznego równoważenia obciążenia AWS, wartość `$headers` powinna wynosić `Request::HEADER_X_FORWARDED_AWS_ELB`. Aby uzyskać więcej informacji na temat stałych, które mogą być używane w własności `$headers`, sprawdź dokumentację Symfony na temat [trusting proxy](http://symfony.com/doc/current/deployment/proxies.html).

#### Trusting All Proxies - Ufając wszystkim serwerom proxy

Jeśli korzystasz z usługi Amazon AWS lub innego dostawcy usług równoważenia obciążenia "w chmurze", możesz nie znać adresów IP rzeczywistych równoważników. W takim przypadku możesz użyć `**` do zaufania wszystkim proxy:

    /**
     * The trusted proxies for this application.
     *
     * @var array
     */
    protected $proxies = '**';