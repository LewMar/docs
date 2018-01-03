# Hashing

- [Introduction - Wprowadzenie](#introduction)
- [Basic Usage - Podstawowe użycie](#basic-usage)

<a name="introduction"></a>
## Introduction - Wprowadzenie

[Fasada](/docs/{{version}}/facades) `Hash` Laravel zapewnia bezpieczne hashowanie Bcrypt do przechowywania haseł użytkowników. Jeśli używasz wbudowanych klas `LoginController` i` RegisterController`, które są dołączone do twojej aplikacji Laravel, automatycznie użyją Bcrypt do rejestracji i uwierzytelnienia.

> {tip} Bcrypt jest świetnym wyborem do mieszania haseł, ponieważ jego "współczynnik pracy" jest regulowany, co oznacza, że czas potrzebny na wygenerowanie skrótu może zostać zwiększony wraz ze wzrostem mocy sprzętowej.

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

Metoda `make` pozwala również zarządzać współczynnikiem roboczym algorytmu mieszania bcrypt za pomocą opcji `rounds`; jednak domyślne ustawienie jest akceptowalne w przypadku większości aplikacji:

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

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
