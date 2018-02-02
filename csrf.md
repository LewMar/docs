# CSRF Protection

- [Introduction - Wprowadzenie](#csrf-introduction)
- [Excluding URIs - Wykluczanie URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Introduction - Wprowadzenie

Laravel ułatwia ochronę aplikacji przed atakami typu [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF). Ataki typu "cross-site request" są rodzajem złośliwego exploita polegającego na wykonywaniu nieautoryzowanych poleceń w imieniu uwierzytelnionego użytkownika.

Laravel automatycznie generuje "token" CSRF dla każdej aktywnej sesji użytkownika zarządzanej przez aplikację. Ten token służy do sprawdzania, czy uwierzytelniony użytkownik faktycznie wysyła żądania do aplikacji.

Za każdym razem, gdy definiujesz formularz HTML w aplikacji, w formularzu należy umieścić ukryte pole tokenu CSRF, aby oprogramowanie pośredniczące do ochrony CSRF mogło potwierdzić żądanie. Możesz użyć pomocnika `csrf_field` do wygenerowania pola tokena:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

`VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), które jest zawarte w grupie oprogramowania `web`, automatycznie sprawdzi, czy token wejściowy żądania pasuje do tokena zapisanego w sesji.

#### CSRF Tokens & JavaScript - Tokeny CSRF i JavaScript

Podczas tworzenia aplikacji sterowanych JavaScriptiem wygodnie jest, aby biblioteka HTTP JavaScript automatycznie dołączała token CSRF do każdego wychodzącego żądania. Domyślnie plik `resources/assets/js/bootstrap.js` rejestruje wartość metatagu `csrf-token` przy użyciu biblioteki HTTP Axios. Jeśli nie używasz tej biblioteki, musisz ręcznie skonfigurować to zachowanie dla swojej aplikacji.

<a name="csrf-excluding-uris"></a>
## Excluding URIs From CSRF Protection - Wykluczanie identyfikatorów URI z ochrony CSRF

Czasami możesz chcieć wykluczyć zbiór identyfikatorów URI z ochrony CSRF. Na przykład, jeśli korzystasz z [Stripe](https://stripe.com) do przetwarzania płatności i korzystasz z ich systemu webhook, musisz wykluczyć trasę obsługi Webhook'a Stripe z ochrony CSRF, ponieważ Stripe nie będzie wiedział, co token CSRF wysłać na swoje trasy.

Zazwyczaj takie trasy należy umieszczać poza grupą oprogramowania pośredniczącego `web`, która ma zastosowanie do `RouteServiceProvider` dla wszystkich tras w pliku `routes/web.php`. Możesz jednak wykluczyć trasy, dodając swoje identyfikatory URI do właściwości `$except` programu pośredniczącego `VerifyCsrfToken`:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Oprócz sprawdzania tokenu CSRF jako parametru POST, oprogramowanie pośredniczące `VerifyCsrfToken` również sprawdzi nagłówek żądania `X-CSRF-TOKEN`. Można na przykład przechowywać token w tagu HTML `meta`:

    <meta name="csrf-token" content="{{ csrf_token() }}">

Następnie, po utworzeniu znacznika `meta`, możesz polecić bibliotece takiej jak jQuery, aby automatycznie dodać token do wszystkich nagłówków żądań. Zapewnia to prostą, wygodną ochronę CSRF dla aplikacji wykorzystujących technologię AJAX:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} Domyślnie plik  `resources/assets/js/bootstrap.js` rejestruje wartość metatagu `csrf-token` przy użyciu biblioteki HTTP Axios. Jeśli nie używasz tej biblioteki, musisz ręcznie skonfigurować to zachowanie dla swojej aplikacji.

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel przechowuje bieżący token CSRF w pliku cookie `XSRF-TOKEN`, który jest dołączony do każdej odpowiedzi wygenerowanej przez framework. Możesz użyć wartości cookie, aby ustawić nagłówek żądania `X-XSRF-TOKEN`.

Ten plik cookie jest przede wszystkim wysyłany jako udogodnienie, ponieważ niektóre struktury i biblioteki JavaScript, takie jak Angular i Axios, automatycznie umieszczają swoją wartość w nagłówku `X-XSRF-TOKEN`.
