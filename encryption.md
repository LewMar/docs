# Encryption

- [Introduction - Wprowadzenie](#introduction)
- [Configuration - Konfiguracja](#configuration)
- [Using The Encrypter - Używanie Encryptera](#using-the-encrypter)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Szyfrowanie Laravel wykorzystuje OpenSSL do zapewnienia szyfrowania AES-256 i AES-128. Zachęcamy do korzystania z wbudowanych funkcji szyfrowania Laravel, a nie do próbowania własnych "rodzimych" algorytmów szyfrowania. Wszystkie zaszyfrowane wartości Laravel są podpisane przy użyciu kodu uwierzytelniania wiadomości (MAC), tak że ich wartości podstawowej nie można zmodyfikować po zaszyfrowaniu.

<a name="configuration"></a>
## Configuration - Konfiguracja

Przed użyciem szyfratora Laravel należy ustawić opcję `key` w pliku konfiguracyjnym `config/app.php`. Do wygenerowania tego klucza powinieneś użyć polecenia `php artisan key: generate`, ponieważ to polecenie Artisana użyje generatora bezpiecznych losowych bajtów PHP do zbudowania twojego klucza. Jeśli ta wartość nie zostanie poprawnie ustawiona, wszystkie wartości zaszyfrowane przez Laravel będą niepewne.

<a name="using-the-encrypter"></a>
## Using The Encrypter - Używanie Encryptera

#### Encrypting A Value - Szyfrowanie wartości

Możesz zaszyfrować wartość za pomocą pomocnika `encrypt`. Wszystkie zaszyfrowane wartości są szyfrowane przy użyciu OpenSSL i szyfru `AES-256-CBC`. Ponadto wszystkie zaszyfrowane wartości są podpisane za pomocą kodu uwierzytelnienia wiadomości (MAC) w celu wykrycia wszelkich modyfikacji zaszyfrowanego ciągu znaków:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

#### Encrypting Without Serialization - Szyfrowanie bez Serializacji

Zaszyfrowane wartości są przesyłane przez `serialize` podczas szyfrowania, co pozwala na szyfrowanie obiektów i tablic. Zatem klienci nie-PHP otrzymujący zaszyfrowane wartości będą musieli "odserializować" dane. Jeśli chcesz szyfrować i odszyfrowywać wartości bez serializacji, możesz użyć metod `encryptString` i` decryptString` z elewacji `Crypt`:

    use Illuminate\Support\Facades\Crypt;

    $encrypted = Crypt::encryptString('Hello world.');

    $decrypted = Crypt::decryptString($encrypted);

#### Decrypting A Value - Odszyfrowywanie wartości

Możesz odszyfrować wartości za pomocą pomocnika `decrypt`. Jeśli wartości nie można poprawnie odszyfrować, na przykład gdy MAC jest nieważny, zostanie zgłoszony komunikat `Illuminate\Contracts\Encryption\DecryptException`:

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
