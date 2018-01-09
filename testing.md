# Testing: Getting Started

- [Introduction - Wprowadzenie](#introduction)
- [Environment - Środowisko](#environment)
- [Creating & Running Tests - Tworzenie i uruchamianie testów](#creating-and-running-tests)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel jest zbudowany z myślą o testowaniu. W rzeczywistości wsparcie testowania za pomocą PHPUnit jest dołączone do zestawu, a plik `phpunit.xml` jest już skonfigurowany dla twojej aplikacji. Platforma jest również dostarczana z wygodnymi metodami pomocniczymi, które umożliwiają ekspresyjne testowanie aplikacji.

Domyślnie katalog `tests` aplikacji zawiera dwa katalogi:` Feature` i `Unit`. Testy jednostkowe (Unit) to testy koncentrujące się na bardzo małej, odizolowanej części kodu. W rzeczywistości większość testów jednostkowych prawdopodobnie koncentruje się na pojedynczej metodzie. Testy cech (Feature) mogą testować większą część twojego kodu, w tym, jak wiele obiektów współdziała ze sobą, a nawet pełne żądanie HTTP do punktu końcowego JSON.

Plik `ExampleTest.php` znajduje się w katalogach testowych `Feature` i `Unit`. Po zainstalowaniu nowej aplikacji Laravel po prostu uruchom `phpunit` w linii poleceń, aby uruchomić testy.

<a name="environment"></a>
## Environment - Środowisko

Podczas testowania za pomocą `phpunit`, Laravel automatycznie ustawi środowisko konfiguracyjne na `testing` ze względu na zmienne środowiskowe zdefiniowane w pliku `phpunit.xml`. Laravel automatycznie konfiguruje sesję i bufor pamięci do sterownika `array` podczas testowania, co oznacza, że dane sesji lub pamięci podręcznej nie zostaną utrwalone podczas testowania.

Możesz dowolnie definiować inne wartości konfiguracji środowiska testowego. Zmienne środowiskowe `testing` mogą być skonfigurowane w pliku `phpunit.xml`, ale pamiętaj o wyczyszczeniu pamięci podręcznej konfiguracji za pomocą polecenia `config:clear` Artisan przed uruchomieniem testów!

<a name="creating-and-running-tests"></a>
## Creating & Running Tests - Tworzenie i uruchamianie testów

Aby utworzyć nowy przypadek testowy, użyj polecenia `make:test` Artisan:

    // Create a test in the Feature directory... - Utwórz test w katalogu cech ...
    php artisan make:test UserTest

    // Create a test in the Unit directory... - Utwórz test w katalogu jednostek ...
    php artisan make:test UserTest --unit

Po wygenerowaniu testu możesz zdefiniować metody testowania, tak jak zwykle używasz PHPUnit. Aby uruchomić testy, wystarczy wykonać polecenie `phpunit` z terminala:

    <?php

    namespace Tests\Unit;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $this->assertTrue(true);
        }
    }

> {note} Jeśli zdefiniujesz własną metodę `setUp` w klasie testowej, upewnij się, że wywołujesz `parent::setUp()`.
