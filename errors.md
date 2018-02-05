# Errors & Logging

- [Introduction - Worowadzenie](#introduction)
- [Configuration - Konfiguracja](#configuration)
- [The Exception Handler - Obsługa wyjątków](#the-exception-handler)
    - [Report Method -  Metoda raportu](#report-method)
    - [Render Method - Metoda renderingu](#render-method)
    - [Reportable & Renderable Exceptions - Raportujące i Renderujące Wyjątki](#renderable-exceptions)
- [HTTP Exceptions - Wyjątki HTTP](#http-exceptions)
    - [Custom HTTP Error Pages - Niestandardowe strony błędów HTTP](#custom-http-error-pages)

<a name="introduction"></a>
## Introduction - Worowadzenie

Po uruchomieniu nowego projektu Laravel obsługa błędów i wyjątków jest już skonfigurowana. Klasa `App\Exceptions\Handler` to miejsce, w którym wszystkie wyjątki wywoływane przez aplikację są rejestrowane, a następnie zwracane do użytkownika. Pogłębimy się w tej klasie w tej dokumentacji.

<a name="configuration"></a>
## Configuration - Konfiguracja

Opcja `debug` w pliku konfiguracyjnym `config/app.php` określa, ile informacji o błędzie faktycznie wyświetla się użytkownikowi. Domyślnie ta opcja jest ustawiona na respektowanie wartości zmiennej środowiskowej `APP_DEBUG`, która jest przechowywana w twoim pliku` .env`.

Dła środowiska lokalnego należy ustawić zmienną środowiskową `APP_DEBUG` na 'true`. W środowisku produkcyjnym ta wartość powinna zawsze mieć wartość "false". Jeśli wartość jest ustawiona na "true" w produkcji, ryzykujesz ujawnienie poufnych wartości konfiguracyjnych użytkownikom końcowym aplikacji.

<a name="the-exception-handler"></a>
## The Exception Handler - Obsługa wyjątków

<a name="report-method"></a>
### The Report Method - Metoda raportu

Wszystkie wyjątki są obsługiwane przez klasę `App\Exceptions\Handler`. Ta klasa zawiera dwie metody: `report` i `render`. Zbadamy każdą z tych metod w szczegółach. Metoda `report` służy do rejestrowania wyjątków lub wysyłania ich do zewnętrznej usługi, takiej jak [Bugsnag](https://bugsnag.com) lub [Sentry](https://github.com/getsentry/sentry-laravel). Domyślnie metoda `report` przekazuje wyjątek do klasy bazowej, w której rejestrowany jest wyjątek. Możesz jednak rejestrować wyjątki według własnego uznania.

Na przykład, jeśli chcesz zgłosić różne typy wyjątków na różne sposoby, możesz użyć operatora porównania `instanceof` PHP:

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

{tip} Zamiast wykonywać wiele testów typu `instanceof` w swojej metodzie `report`, rozważ użycie [wyjątków raportowanych](/docs/{{version}}/errors#renderable-exceptions)

#### The `report` Helper - Pomocnik  `report`

Czasami może być konieczne zgłoszenie wyjątku, ale i kontynuowanie obsługi bieżącego żądania. Funkcja pomocnicza `report` pozwala na szybkie zgłoszenie wyjątku za pomocą metody `report` obsługi wyjątku bez renderowania strony błędu:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Exception $e) {
            report($e);

            return false;
        }
    }

#### Ignoring Exceptions By Type - Ignorowanie wyjątków według typu

Właściwość `$dontReport` obsługi wyjątku zawiera tablicę typów wyjątków, które nie będą rejestrowane. Na przykład wyjątki wynikające z błędów 404, a także kilka innych typów błędów, nie są zapisywane w plikach dziennika. W razie potrzeby możesz dodać do tej tablicy inne typy wyjątków:

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### The Render Method - Metoda renderingu

Metoda `render` jest odpowiedzialna za przekształcenie danego wyjątku w odpowiedź HTTP, która powinna zostać odesłana do przeglądarki. Domyślnie wyjątek jest przekazywany do klasy bazowej, która generuje dla ciebie odpowiedź. Możesz jednak sprawdzić typ wyjątku lub zwrócić własną niestandardową odpowiedź:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="renderable-exceptions"></a>
### Reportable & Renderable Exceptions - Raportujące i Renderujące Wyjątki

Zamiast wyjątków sprawdzających typ w metodach `report` i `render` obsługi wyjątku, możesz zdefiniować metody `report` i `render` bezpośrednio w swoim niestandardowym wyjątku. Gdy te metody istnieją, będą automatycznie wywoływane przez framework:

    <?php

    namespace App\Exceptions;

    use Exception;

    class RenderException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * Render the exception into an HTTP response.
         *
         * @param  \Illuminate\Http\Request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

<a name="http-exceptions"></a>
## HTTP Exceptions - Wyjątki HTTP

Niektóre wyjątki opisują kody błędów HTTP z serwera. Na przykład może to być błąd "nie znaleziono strony" (404), "nieautoryzowany błąd" (401) lub nawet błąd programisty 500. Aby wygenerować taką odpowiedź z dowolnego miejsca w aplikacji, możesz użyć pomocnika `abort`:

    abort(404);

Pomocnik `abort` natychmiast podniósł wyjątek, który zostanie wyrenderowany przez procedurę obsługi wyjątku. Opcjonalnie możesz podać tekst odpowiedzi:

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### Custom HTTP Error Pages - Niestandardowe strony błędów HTTP

Laravel ułatwia wyświetlanie niestandardowych stron błędów dla różnych kodów stanu HTTP. Na przykład, jeśli chcesz dostosować stronę błędu dla 404 kodów stanu HTTP, utwórz `resources/views/errors/404.blade.php`. Ten plik zostanie wyświetlony na wszystkich błędach 404 wygenerowanych przez aplikację. Widoki w tym katalogu powinny mieć nazwę odpowiadającą kodowi statusu HTTP, którego odpowiadają. Instancja `HttpException` podniesiona przez funkcję `abort` zostanie przekazana do widoku jako zmienna `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>
