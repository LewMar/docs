# HTTP Tests

- [Introduction - Wprowadzenie](#introduction)
    - [Customizing Request Headers - Dostosowywanie nagłówków żądań](#customizing-request-headers)
- [Session / Authentication - Sesja / uwierzytelnianie](#session-and-authentication)
- [Testing JSON APIs - Testowanie interfejsów API JSON](#testing-json-apis)
- [Testing File Uploads - Testowanie przesyłania plików](#testing-file-uploads)
- [Available Assertions - Dostępne twierdzenia](#available-assertions)
    - [Response Assertions - Odpowiadające twierdzenia](#response-assertions)
    - [Authentication Assertions - Twierdzenia uwierzytelniania](#authentication-assertions)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel zapewnia bardzo płynny interfejs API do wysyłania żądań HTTP do aplikacji i sprawdzania danych wyjściowych. Na przykład spójrz na test zdefiniowany poniżej:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

Metoda `get` wysyła żądanie `GET` do aplikacji, natomiast metoda `assertStatus` zapewnia, że zwrócona odpowiedź powinna mieć podany kod statusu HTTP. Oprócz tego prostego stwierdzenia, Laravel zawiera także wiele asercji do sprawdzania nagłówków odpowiedzi, treści, struktury JSON i innych.

<a name="customizing-request-headers"></a>
### Customizing Request Headers - Dostosowywanie nagłówków żądań

Możesz użyć metody `withHeaders` w celu dostosowania nagłówków żądania przed jego wysłaniem do aplikacji. Dzięki temu możesz dodać dowolne niestandardowe nagłówki, które chcesz uwzględnić w żądaniu:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

<a name="session-and-authentication"></a>
## Session / Authentication - Sesja / uwierzytelnianie

Laravel zapewnia kilku pomocników do pracy z sesją podczas testów HTTP. Najpierw możesz ustawić dane sesji do danej tablicy przy użyciu metody `withSession`. Jest to przydatne do ładowania sesji danymi przed wysłaniem żądania do aplikacji:

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Oczywiście jednym z typowych zastosowań sesji jest utrzymywanie stanu uwierzytelnionego użytkownika. Metoda `actingAs` pomocnika zapewnia prosty sposób uwierzytelnienia danego użytkownika jako bieżącego użytkownika. Na przykład możemy użyć [fabryki modelu](/docs/{{version}}/database-testing#writing-factories) do wygenerowania i uwierzytelnienia użytkownika:

    <?php

    use App\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

Możesz również określić, która osłona powinna zostać użyta do uwierzytelnienia danego użytkownika, przekazując nazwę strażnika jako drugi argument metody `actingAs`:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
## Testing JSON APIs - Testowanie interfejsów API JSON

Laravel zapewnia również kilka pomocników do testowania interfejsów API JSON i ich odpowiedzi. Na przykład metody `json`, `get`, `post`, `put`, `patch`, i `delete` mogą być używane do wysyłania żądań z różnymi akcjami HTTP. Możesz także łatwo przekazywać dane i nagłówki do tych metod. Aby rozpocząć, napiszmy test, aby przesłać żądanie `POST` do `/user` i potwierdzić, że oczekiwane dane zostały zwrócone:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} Metoda `assertJson` konwertuje odpowiedź na tablicę i używa `PHPUnit::assertArraySubset` w celu sprawdzenia, czy dana tablica istnieje w odpowiedzi JSON zwróconej przez aplikację. Tak więc, jeśli w odpowiedzi JSON istnieją inne właściwości, test ten będzie nadal przebiegać tak długo, jak długo dany fragment jest obecny.

<a name="verifying-exact-match"></a>
### Verifying An Exact JSON Match - Weryfikacja dokładnego dopasowania JSON

Jeśli chcesz sprawdzić, czy dana tablica jest dopasowana **dokładnie** dla JSON zwróconego przez aplikację, powinieneś użyć metody `assertExactJson`:

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="testing-file-uploads"></a>
## Testing File Uploads - Testowanie przesyłania plików

Klasa `Illuminate\Http\UploadedFile` udostępnia metodę `fake`, która może być używana do generowania fałszywych plików lub obrazów do testowania. To w połączeniu z metodą `fake` fasady `Storage` znacznie upraszcza testowanie przesyłania plików. Na przykład możesz połączyć te dwie funkcje, aby łatwo przetestować formularz przesyłania awatara:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // Assert the file was stored... - Twierdzenie, że plik został zapisany ...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // Assert a file does not exist... - Twierdzenie, że plik nie istnieje ...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### Fake File Customization - Dostosowywanie fałszywych plików

Podczas tworzenia plików za pomocą metody `fake` możesz określić szerokość, wysokość i rozmiar obrazu, aby lepiej przetestować zasady sprawdzania poprawności:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

Oprócz tworzenia obrazów możesz tworzyć pliki dowolnego innego typu za pomocą metody `create`:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

<a name="available-assertions"></a>
## Available Assertions - Dostępne twierdzenia

<a name="response-assertions"></a>
### Response Assertions - Odpowiadające twierdzenia

Laravel oferuje wiele niestandardowych metod asercji dla testów [PHPUnit](https://phpunit.de/). Dostęp do tych asercji można uzyskać na podstawie odpowiedzi zwracanej przez metody testowe `json`, `get`, `post`, `put`, i `delete`:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertExactJson](#assert-exact-json)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertJson](#assert-json)
[assertJsonFragment](#assert-json-fragment)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertSee](#assert-see)
[assertSeeText](#assert-see-text)
[assertSessionHas](#assert-session-has)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionMissing](#assert-session-missing)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)
[assertJsonCount](#assert-json-count)

</div>

<a name="assert-cookie"></a>
#### assertCookie

Twierdzimy, że odpowiedź zawiera dany plik cookie:

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

Twierdzimy, że odpowiedź zawiera dany plik cookie i wygasła:

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Twierdzimy, że odpowiedź nie zawiera podanego pliku cookie:

    $response->assertCookieMissing($cookieName);

<a name="assert-dont-see"></a>
#### assertDontSee

Twierdzimy, że dany ciąg nie jest zawarty w odpowiedzi:

    $response->assertDontSee($value);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

Twierdzimy, że dany ciąg nie jest zawarty w tekście odpowiedzi:

    $response->assertDontSeeText($value);

<a name="assert-exact-json"></a>
#### assertExactJson

Twierdzimy, że odpowiedź zawiera dokładne dopasowanie danych JSON:

    $response->assertExactJson(array $data);

<a name="assert-header"></a>
#### assertHeader

Twierdzimy, że dany nagłówek jest obecny w odpowiedzi:

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

Twierdzimy, że dany nagłówek nie jest obecny w odpowiedzi:

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>
#### assertJson

Twierdzimy, że odpowiedź zawiera dane JSON:

    $response->assertJson(array $data);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

wierdzimy, że odpowiedź zawiera dany fragment JSON:

    $response->assertJsonFragment(array $data);

<a name="assert-json-missing"></a>
#### assertJsonMissing

Twierdzimy, że odpowiedź nie zawiera danego fragmentu JSON:

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

Twierdzimy, że odpowiedź nie zawiera dokładnego fragmentu JSON:

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-structure"></a>
#### assertJsonStructure

Twierdzimy, że odpowiedź ma określoną strukturę JSON:

    $response->assertJsonStructure(array $structure);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

Twierdzimy, że odpowiedź ma podane błędy sprawdzania poprawności JSON dla podanych kluczy:

    $response->assertJsonValidationErrors($keys);

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

Twierdzimy, że odpowiedź zawiera dany plik cookie (niezaszyfrowany):

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

Twierdzimy, że odpowiedź jest przekierowaniem do danego URI:

    $response->assertRedirect($uri);

<a name="assert-see"></a>
#### assertSee

Twierdzimy, że dany ciąg jest zawarty w odpowiedzi:

    $response->assertSee($value);

<a name="assert-see-text"></a>
#### assertSeeText

Twierdzimy, że podany ciąg jest zawarty w tekście odpowiedzi:

    $response->assertSeeText($value);

<a name="assert-session-has"></a>
#### assertSessionHas

Twierdzimy, że sesja zawiera część danych:

    $response->assertSessionHas($key, $value = null);

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

Twierdzimy, że sesja ma podaną listę wartości:

    $response->assertSessionHasAll($key, $value = null);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

Twierdzimy, że sesja zawiera błąd dla danego pola:

    $response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

Twierdzimy, że sesja ma podane błędy:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-missing"></a>
#### assertSessionMissing

Twierdzimy, że sesja nie zawiera podanego klucza:

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

Twierdzimy, że odpowiedź ma podany kod:

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

Twierdzimy, że odpowiedź zawiera udany kod statusu:

    $response->assertSuccessful();

<a name="assert-view-has"></a>
#### assertViewHas

Twierdzimy, że widok odpowiedzi otrzymał pewną ilość danych:

    $response->assertViewHas($key, $value = null);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

Twierdzimy, że widok odpowiedzi zawiera podaną listę danych:

    $response->assertViewHasAll(array $data);

<a name="assert-view-is"></a>
#### assertViewIs

Twierdzimy, że dany widok został zwrócony przez trasę:

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

Twierdzimy, że w widoku odpowiedzi brakuje części powiązanych danych:

    $response->assertViewMissing($key);

<a name="assert-json-count"></a>
#### assertJsonCount

Twierdzimy, że odpowiedź JSON ma oczekiwaną liczbę pozycji na danym kluczu:

    $response->assertJsonCount(int $count, $key = null);



<a name="authentication-assertions"></a>
### Authentication Assertions - Twierdzenia uwierzytelniania

Laravel zapewnia również szereg asercji związanych z uwierzytelnianiem dla testów [PHPUnit](https://phpunit.de/):

Metoda  | Opis
------------- | -------------
`$this->assertAuthenticated($guard = null);`  |  Twierdzimy, że użytkownik jest uwierzytelniony.
`$this->assertGuest($guard = null);`  |  Twierdzimy, że użytkownik nie jest uwierzytelniony.
`$this->assertAuthenticatedAs($user, $guard = null);`  |  Twierdzimy, że dany użytkownik jest uwierzytelniony.
`$this->assertCredentials(array $credentials, $guard = null);`  |  Twierdzimy, że podane poświadczenia są prawidłowe.
`$this->assertInvalidCredentials(array $credentials, $guard = null);`  |  Twierdzimy, że podane poświadczenia są nieprawidłowe.
