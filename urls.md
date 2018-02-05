# URL Generation

- [Introduction - Wprowadzenie](#introduction)
- [The Basics - Podstawy](#the-basics)
    - [Generating Basic URLs - Generowanie podstawowych URL-ów](#generating-basic-urls)
    - [Accessing The Current URL - Dostęp do bieżącego adresu URL](#accessing-the-current-url)
- [URLs For Named Routes - Adresy URL dla nazwanych tras](#urls-for-named-routes)
- [URLs For Controller Actions - Adresy URL dla akcji kontrolerów](#urls-for-controller-actions)
- [Default Values - Domyślne wartości](#default-values)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel oferuje pomocników pomagających w generowaniu adresów URL dla aplikacji. Oczywiście są one pomocne głównie podczas tworzenia linków w szablonach i odpowiedziach API lub podczas generowania przekierowań na inną część aplikacji.

<a name="the-basics"></a>
## The Basics - Podstawy

<a name="generating-basic-urls"></a>
### Generating Basic URLs - Generowanie podstawowych URL-ów

Pomocnik `url` może być używany do generowania dowolnych adresów URL dla twojej aplikacji. Wygenerowany adres URL automatycznie użyje schematu (HTTP lub HTTPS) i hosta z bieżącego żądania:

    $post = App\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Accessing The Current URL - Dostęp do bieżącego adresu URL

Jeśli nie podano ścieżki do helpera `url`, zwracana jest instancja `Illuminate\Routing\UrlGenerator`, która umożliwia dostęp do informacji o bieżącym adresie URL:

    // Get the current URL without the query string...
    echo url()->current();

    // Get the current URL including the query string...
    echo url()->full();

    // Get the full URL for the previous request...
    echo url()->previous();

Do każdej z tych metod można również uzyskać dostęp za pośrednictwem `URL` [fasady](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URLs For Named Routes - Adresy URL dla nazwanych tras

Pomocnik `route` może być używany do generowania adresów URL do nazwanych tras. Nazwane trasy umożliwiają generowanie adresów URL bez połączenia z rzeczywistym adresem URL zdefiniowanym na trasie. Dlatego jeśli adres URL trasy ulegnie zmianie, nie trzeba wprowadzać żadnych zmian w wywołaniach funkcji `route`. Na przykład wyobraź sobie, że twoja aplikacja zawiera trasę zdefiniowaną następująco:

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

Aby wygenerować adres URL tej trasy, możesz użyć helpera `route`, jak na przykład:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Często będziesz generował adresy URL używając klucza podstawowego [Eloquent models](/docs/{{version}}/eloquent). Z tego powodu możesz przekazać modelom Wymownym jako wartości parametrów. Pomocnik `route` automatycznie wyodrębni klucz podstawowy modelu:

    echo route('post.show', ['post' => $post]);

<a name="urls-for-controller-actions"></a>
## URLs For Controller Actions - Adresy URL dla akcji kontrolerów

Funkcja `action` generuje adres URL dla podanej akcji kontrolera. Nie musisz przekazywać pełnego obszaru nazw kontrolera. Zamiast tego podaj nazwę klasy kontrolera względem obszaru nazw `App\Http\Controllers`:

    $url = action('HomeController@index');

Jeśli metoda kontrolera akceptuje parametry trasy, możesz przekazać je jako drugi argument funkcji:

    $url = action('UserController@profile', ['id' => 1]);

<a name="default-values"></a>
## Default Values - Domyślne wartości

W przypadku niektórych aplikacji możesz chcieć określić domyślne wartości dla całego żądania dla niektórych parametrów URL. Na przykład wyobraź sobie, że wiele tras definiuje parametr `{locale}`:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Uciążliwe jest zawsze przekazywanie `locale` za każdym razem, gdy wywołujesz pomocnika `route`. Tak więc możesz użyć metody `URL::defaults` do zdefiniowania wartości domyślnej dla tego parametru, która będzie zawsze stosowana podczas bieżącego żądania. Możesz wywołać tę metodę z [trasy oprogramowania posredniego](/docs/{{version}}/middleware#assigning-middleware-to-routes), aby mieć dostęp do bieżącego żądania:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Po ustawieniu domyślnej wartości parametru `locale` nie musisz już przekazywać jej wartości podczas generowania adresów URL za pomocą helpera` route`.
