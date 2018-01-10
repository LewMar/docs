# Contribution Guide

- [Bug Reports - Zgłaszanie błędów](#bug-reports)
- [Core Development Discussion - Dyskusja o rozwoju podstawowym](#core-development-discussion)
- [Which Branch? - Który oddział?](#which-branch)
- [Security Vulnerabilities - Luki bezpieczeństwa](#security-vulnerabilities)
- [Coding Style - Styl kodowania](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Bug Reports - Zgłaszanie błędów

Aby zachęcić do aktywnej współpracy, Laravel zachęca do wysyłania żądań, a nie tylko raportów o błędach. "Raporty o błędach" mogą być również wysyłane w formie żądania pobrania zawierającego test awarii.

Jeśli jednak zgłosisz zgłoszenie błędu, Twój problem powinien zawierać tytuł i jasny opis problemu. Powinieneś również dołączyć jak najwięcej istotnych informacji oraz próbkę kodu, która pokazuje problem. Celem zgłoszenia błędu jest ułatwienie sobie - i innym - powielenia błędu i opracowania poprawki.

Pamiętaj, że raporty o błędach są tworzone z nadzieją, że inni z tym samym problemem będą mogli współpracować z Tobą przy jego rozwiązywaniu. Nie oczekuj, że raport o błędach automatycznie zobaczy jakąkolwiek aktywność lub że inni będą skakać, aby to naprawić. Utworzenie raportu o błędzie służy pomocy samemu sobie i innym w rozpoczęciu rozwiązywania problemu.

Kod źródłowy Laravel jest zarządzany na GitHub i istnieją repozytoria dla każdego z projektów Laravel:

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Website](https://github.com/laravel/laravel.com)
</div>

<a name="core-development-discussion"></a>
## Core Development Discussion - Dyskusja o rozwoju podstawowym

Możesz zaproponować nowe funkcje lub ulepszenia istniejącego zachowania Laravel w Laravel Internals [tablica ogłoszeń](https://github.com/laravel/internals/issues). Jeśli zaproponujesz nową funkcję, zechciej zaimplementować przynajmniej część kodu, który byłby potrzebny do ukończenia tej funkcji.

Nieformalna dyskusja na temat błędów, nowych funkcji i implementacji istniejących funkcji odbywa się na kanale `#internals` w [LaraChat](https://larachat.co) zespołu Slack. Taylor Otwell, opiekun Laravel, jest zazwyczaj obecny w kanale w dni powszednie od 8 rano do 5 po południu (UTC-06: 00 lub Ameryka/Chicago), a sporadycznie obecny na kanale w innym czasie.

<a name="which-branch"></a>
## Which Branch? - Który oddział?

**Wszystkie poprawki** powinny być wysyłane do najnowszego stabilnego oddziału lub do bieżącego oddziału LTS (5.5). Poprawki błędów **nigdy** nie powinny być wysyłane do gałęzi `master`, chyba że naprawią funkcje, które istnieją tylko w nadchodzącym wydaniu.

**Drobne(Minor)** funkcje, które są **w pełni kompatybilne wstecz** z aktualną wersją Laravel mogą być wysyłane do najnowszego stabilnego oddziału.

**Główne(Major)** nowe funkcje powinny zawsze być wysyłane do gałęzi `master`, która zawiera nadchodzące wydanie Laravel.

Jeśli nie masz pewności, czy twoja funkcja kwalifikuje się jako główna lub drobne, poproś Taylor'a Otwella na kanał `#internals` w [LaraChat](https://larachat.co)  zespołu Slack.

<a name="security-vulnerabilities"></a>
## Security Vulnerabilities - Luki bezpieczeństwa

Jeśli wykryjesz lukę w zabezpieczeniach w Laravel, wyślij wiadomość e-mail do Taylor'a Otwella pod adresem <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Wszystkie luki w zabezpieczeniach zostaną szybko rozwiązane.

<a name="coding-style"></a>
## Coding Style - Styl kodowania

Laravel postępuje zgodnie ze standardem kodowania [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) i [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) standardem automatycznego ładowania.

<a name="phpdoc"></a>
### PHPDoc

Poniżej znajduje się przykład prawidłowego bloku dokumentacji Laravel. Zwróć uwagę, że po atrybucie `@param` następują dwie spacje, typ argumentu, dwie spacje i na końcu nazwa zmiennej:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

Nie przejmuj się, jeśli Twój styl kodowania nie jest doskonały! [StyleCI](https://styleci.io/) automatycznie połączy wszystkie poprawki stylów z repozytorium Laravel po scaleniu żądań pobierania. Pozwala nam to skupić się na treściach wkładu, a nie na stylu kodu.
