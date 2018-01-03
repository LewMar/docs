# Resetting Passwords

- [Introduction - Wprowadzenie](#introduction)
- [Database Considerations - Rozważania baz danych](#resetting-database)
- [Routing](#resetting-routing)
- [Views - Widoki](#resetting-views)
- [After Resetting Passwords - Po zresetowaniu haseł](#after-resetting-passwords)
- [Customization - Dostosowywanie](#password-customization)

<a name="introduction"></a>
## Introduction - Wprowadzenie

> {tip} **Chcesz szybko zacząć?** Uruchom `php artisan make:auth` w świeżej aplikacji Laravel i przejdź do przeglądarki na adres `http://your-app.dev/register` lub dowolny inny adres URL przypisany do twojej aplikacji. To pojedyncze polecenie zajmie się ruszeniem całego systemu uwierzytelniania, w tym resetowaniem haseł!

Większość aplikacji internetowych umożliwia użytkownikom resetowanie zapomnianych haseł. Zamiast zmuszać Cię do ponownego wdrożenia tego w każdej aplikacji, Laravel zapewnia wygodne metody wysyłania przypomnień o hasłach i resetowania hasła.

> {note} Przed użyciem funkcji resetowania hasła Laravel, użytkownik musi użyć cechy (trait) `Illuminate\Notifications\Notifiable`.

<a name="resetting-database"></a>
## Database Considerations - Rozważania baz danych

Aby rozpocząć, sprawdź, czy Twój model `App\User` implementuje umowę `Illuminate\Contracts\Auth\CanResetPassword`. Oczywiście model `App\User` dołączony do frameworka implementuje już ten interfejs i używa cechy (trait) `Illuminate\Auth\Passwords\CanResetPassword`, aby uwzględnić metody potrzebne do implementacji interfejsu.

#### Generating The Reset Token Table Migration - Generowanie resetowania migracji tabeli tokenów

Następnie należy utworzyć tabelę przechowującą tokeny resetowania hasła. Migracja do tej tabeli jest dołączana do programu Laravel po wyjęciu z pudełka i znajduje się w katalogu `database/migrations`. Wszystko, co musisz zrobić, to uruchomić migracje do bazy danych:

    php artisan migrate

<a name="resetting-routing"></a>
## Routing

Laravel zawiera klasy `Auth\ForgotPasswordController` i `Auth\ResetPasswordController` zawierające logikę niezbędną do przesyłania linków do resetowania haseł i resetowania haseł użytkowników. Wszystkie trasy potrzebne do resetowania hasła mogą być generowane za pomocą polecenia `make:auth` Artisan:

    php artisan make:auth

<a name="resetting-views"></a>
## Views - Widoki

Ponownie, Laravel wygeneruje wszystkie niezbędne widoki do zresetowania hasła po wykonaniu polecenia `make:auth`. Widoki te są umieszczone w `resources/views/auth/passwords`. Możesz dowolnie dostosować je do swoich potrzeb.

<a name="after-resetting-passwords"></a>
## After Resetting Passwords - Po zresetowaniu haseł

Po zdefiniowaniu tras i widoków w celu zresetowania haseł użytkownika możesz po prostu uzyskać dostęp do trasy w przeglądarce w `/password/reset`. `ResetPasswordController` dołączony do struktury zawiera już logikę do wysyłania e-maili z linkami do resetowania hasła, a `ResetPasswordController` zawiera logikę do resetowania haseł użytkowników.

Po zresetowaniu hasła użytkownik zostanie automatycznie zalogowany do aplikacji i przekierowany do `/home`. Możesz dostosować resetowanie hasła po skasowaniu, definiując właściwość `redirectTo` na` ResetPasswordController`:

    protected $redirectTo = '/dashboard';

> {note} Domyślnie tokeny resetowania hasła wygasają po godzinie. Możesz to zmienić poprzez opcję `expire` w pliku `config/auth.php`.

<a name="password-customization"></a>
## Customization - Dostosowywanie

#### Authentication Guard Customization - Personalizacja osłony uwierzytelnienia

W pliku konfiguracyjnym `auth.php` możesz skonfigurować wiele "strażników", które mogą być używane do definiowania zachowania uwierzytelniania dla wielu tabel użytkowników. Możesz dostosować dołączony `ResetPasswordController`, aby użyć wybranego strażnika, przesłoniwszy metodę `guard` na kontrolerze. Ta metoda powinna zwrócić instancję strażnika:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Password Broker Customization - Dostosowanie Brokera do hasła

W pliku konfiguracyjnym `auth.php` możesz skonfigurować wielu "maklerów" (brokers) hasła, które mogą być użyte do resetowania haseł na wielu tablicach użytkowników. Możesz dostosować dołączone `ForgotPasswordController` i `ResetPasswordController`, aby użyć wybranego brokera, zastępując metodę `broker`:

    use Illuminate\Support\Facades\Password;

    /**
     * Get the broker to be used during password reset.
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### Reset Email Customization - Zresetuj dostosowywanie wiadomości e-mail

Możesz łatwo zmodyfikować klasę powiadomień używaną do wysyłania linku do resetowania hasła do użytkownika. Aby rozpocząć, należy przesłonić metodę `sendPasswordResetNotification` w swoim modelu `User`. W ramach tej metody możesz wysłać powiadomienie za pomocą dowolnej wybranej klasy powiadomień. Resetowanie hasła `$token` jest pierwszym argumentem odebranym przez metodę:

    /**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }

