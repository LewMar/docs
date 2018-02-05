# Hashing

- [Introduction - Wprowadzenie](#introduction)
- [Configuration - Konfiguracja](#configuration)
- [Basic Usage - Podstawowe użycie](#basic-usage)

<a name="introduction"></a>
## Introduction - Wprowadzenie

[Fasada](/docs/{{version}}/facades) `Hash` Laravel-a zapewnia bezpieczne hashowanie Bcrypt i Argon2 do przechowywania haseł użytkowników. Jeśli używasz wbudowanych klas `LoginController` i `RegisterController`, które są dołączone do twojej aplikacji Laravel, będą one domyślnie używać Bcrypt do rejestracji i uwierzytelniania.

> {tip} Bcrypt jest świetnym wyborem do mieszania haseł, ponieważ jego "współczynnik pracy" jest regulowany, co oznacza, że czas potrzebny na wygenerowanie skrótu może zostać zwiększony wraz ze wzrostem mocy sprzętowej.

<a name="configuration"></a>
## Configuration - Konfiguracja

Domyślny sterownik skrótu dla twojej aplikacji jest skonfigurowany w pliku konfiguracyjnym `config/hashing.php`. Istnieją obecnie dwa obsługiwane sterowniki: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) i [Argon2](https://en.wikipedia.org/wiki/Argon2).

> {note} Sterownik Argon2 wymaga PHP w wersji 7.2.0 lub wyższej.

<a name="basic-usage"></a>
## Basic Usage - Podstawowe użycie

Możesz hashować hasło, wywołując metodę `make` na elewacji `Hash`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### Adjusting The Bcrypt Work Factor - Dostosowanie współczynnika pracy Bcrypt

Jeśli używasz algorytmu Bcrypt, metoda `make` pozwala ci zarządzać czynnikiem roboczym algorytmu za pomocą opcji `rounds`; jednak domyślne ustawienie jest akceptowalne w przypadku większości aplikacji:

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

#### Adjusting The Argon2 Work Factor - Dostosowanie współczynnika pracy Argon2

Jeśli używasz algorytmu Argon2, metoda `make` pozwala ci zarządzać czynnikiem roboczym algorytmu za pomocą opcji `memory`, `time` i `threads`; jednak ustawienia domyślne są dopuszczalne w przypadku większości aplikacji:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} Aby uzyskać więcej informacji na temat tych opcji, sprawdź [oficjalna dokumentacja PHP](http://php.net/manual/en/function.password-hash.php).

#### Verifying A Password Against A Hash - Weryfikacja hasła o haszowanie

Metoda `check` pozwala sprawdzić, czy dany ciąg tekstowy odpowiada danemu hashowi. Jednakże, jeśli używasz `LoginController` [zawartego w Laravel](/docs/{{version}}/authentication), prawdopodobnie nie będziesz musiał używać tego bezpośrednio, ponieważ kontroler automatycznie wywołuje tę metodę:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### Checking If A Password Needs To Be Rehashed - Sprawdzanie, czy hasło wymaga ponownego przehaszowania

Funkcja `needsRehash` pozwala określić, czy współczynnik pracy używany przez moduł mieszający zmienił się od czasu, gdy hasło zostało zmieszane:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
