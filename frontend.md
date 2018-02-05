# JavaScript & CSS Scaffolding

- [Introduction - Wprowadzenie](#introduction)
- [Writing CSS - Pisanie CSS](#writing-css)
- [Writing JavaScript - Pisanie JavaScript](#writing-javascript)
    - [Writing Vue Components - Pisanie komponentów Vue](#writing-vue-components)
    - [Using React - Używanie React](#using-react)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel nie dyktuje używanych pre-procesorów JavaScript ani CSS, ale zapewnia podstawowy punkt wyjścia za pomocą [Bootstrap](https://getbootstrap.com/) i [Vue](https://vuejs.org), będzie pomocny w wielu aplikacjach. Domyślnie Laravel używa [NPM](https://www.npmjs.org) do zainstalowania obu tych pakietów frontend.

#### CSS

[Laravel Mix](/docs/{{version}}/mix) zapewnia czysty, ekspresyjny interfejs API nad kompilowaniem SASS lub Less, które są rozszerzeniami zwykłego CSS, które dodają zmienne, mixy i inne zaawansowane funkcje, które znacznie ułatwiają pracę z CSS sprawiający więcej satysfakcji. W tym dokumencie pokrótce omówimy kompilację CSS w ogóle; powinieneś jednak zapoznać się z pełną dokumentacją [Laravel Mix](/docs/{{version}}/mix), aby uzyskać więcej informacji na temat kompilowania SASS lub Less.

#### JavaScript

Laravel nie wymaga używania określonego środowiska JavaScript lub biblioteki do budowy aplikacji. W rzeczywistości wcale nie musisz używać JavaScript. Jednak Laravel zawiera podstawowe rusztowania, które ułatwiają pisanie współczesnego JavaScriptu przy użyciu biblioteki [Vue](https://vuejs.org). Vue zapewnia ekspresyjny interfejs API do budowania wydajnych aplikacji JavaScript wykorzystujących komponenty. Podobnie jak w CSS, możemy użyć Laravel Mix, aby łatwo skompilować komponenty JavaScript w jednym, gotowym do przeglądarki pliku JavaScript.

#### Removing The Frontend Scaffolding

Jeśli chcesz usunąć rusztowanie frontendowe z aplikacji, możesz użyć polecenia `preset` Artisan. To polecenie, w połączeniu z opcją `none`, usunie z twojej aplikacji rusztowania Bootstrap i Vue, pozostawiając tylko pusty plik SASS i kilka popularnych bibliotek narzędziowych JavaScript:

    php artisan preset none

<a name="writing-css"></a>
## Writing CSS  - Pisanie CSS

Plik `package.json` firmy Laravel zawiera pakiet `bootstrap`, który ułatwia rozpoczęcie prototypowania interfejsu aplikacji za pomocą Bootstrap. Jednak możesz dowolnie dodawać lub usuwać pakiety z pliku `package.json` zgodnie z potrzebami swojej aplikacji. Nie musisz używać struktury Bootstrap do budowania aplikacji Laravel - jest to dobry punkt wyjścia dla tych, którzy z niej korzystają.

Przed skompilowaniem swojego CSS, zainstaluj zależności frontendu projektu za pomocą [menedżera pakietów Node (NPM)](https://www.npmjs.org):

    npm install

Po zainstalowaniu zależności za pomocą `npm install`, możesz skompilować swoje pliki SASS do zwykłego CSS używając [Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets). Polecenie `npm run dev` przetworzy instrukcje w twoim pliku` webpack.mix.js`. Zazwyczaj skompilowany CSS zostanie umieszczony w katalogu `public/css`:

    npm run dev

Domyślny `webpack.mix.js` dołączony do Laravel skompiluje plik SASS `resources/assets/sass/app.scss`. Ten plik `app.scss` importuje plik zmiennych SASS i ładuje Bootstrap, który stanowi dobry punkt wyjścia dla większości aplikacji. Możesz dowolnie dostosować plik `app.scss` lub nawet całkowicie inny preprocesor, [konfigurując Laravel Mix](/docs/{{version}}/mix).

<a name="writing-javascript"></a>
## Writing JavaScript  - Pisanie JavaScript

Wszystkie zależności JavaScript wymagane przez twoją aplikację można znaleźć w pliku `package.json` w katalogu głównym projektu. Ten plik jest podobny do pliku `composer.json`, z wyjątkiem tego, że określa zależności JavaScriptu zamiast zależności PHP. Możesz zainstalować te zależności za pomocą [menedżera pakietów Node (NPM)](https://www.npmjs.org):

    npm install

> {tip} Domyślnie plik Laravel`package.json` zawiera kilka pakietów, takich jak `vue` i `axios`, które ułatwiają rozpoczęcie tworzenia aplikacji JavaScript. Możesz dodać lub usunąć z pliku `package.json` zgodnie z potrzebami swojej własnej aplikacji.

Po zainstalowaniu pakietów możesz użyć polecenia `npm run dev` do [kompilacji zasobów](/docs/{{version}}/mix). Webpack to pakiet modułów dla nowoczesnych aplikacji JavaScript. Po uruchomieniu polecenia `npm run dev`, Webpack wykona instrukcje w twoim pliku` webpack.mix.js`:

    npm run dev

Domyślnie plik Laravel `webpack.mix.js` kompiluje twój SASS i plik `resources/assets/js/app.js`. W pliku `app.js` możesz zarejestrować swoje komponenty Vue lub, jeśli wolisz inną strukturę, skonfigurować własną aplikację JavaScript. Twój skompilowany JavaScript będzie zazwyczaj umieszczany w katalogu `public/js`.

> {tip} Plik `app.js` ładuje plik `resources/assets/js/bootstrap.js`, który ładuje i konfiguruje Vue, Axios, jQuery i wszystkie inne zależności JavaScript. Jeśli masz dodatkowe zależności JavaScript do skonfigurowania, możesz to zrobić w tym pliku.

<a name="writing-vue-components"></a>
### Writing Vue Components  - Pisanie komponentów Vue

Domyślnie nowe aplikacje Laravel zawierają komponent Vue `ExampleComponent.vue` znajdujący się w katalogu `resources/assets/js/components`. Plik `ExampleComponent.vue` jest przykładem [komponentu Vue pojedynczego pliku](https://vuejs.org/guide/single-file-components), który definiuje swój szablon JavaScript i HTML w tym samym pliku. Pojedyncze komponenty plików zapewniają bardzo wygodne podejście do tworzenia aplikacji opartych na JavaScript. Przykładowy składnik jest zarejestrowany w pliku `app.js`:

    Vue.component(
        'example-component',
        require('./components/ExampleComponent.vue')
    );

Aby użyć komponentu w aplikacji, możesz umieścić go w jednym z szablonów HTML. Na przykład, po uruchomieniu polecenia `make:auth` Artisan, aby zarchiwizować ekrany uwierzytelniania i rejestracji twojej aplikacji, możesz umieścić komponent w szablonie Blade `home.blade.php`:

    @extends('layouts.app')

    @section('content')
        <example-component></example-component>
    @endsection

> {tip} Pamiętaj, że powinieneś uruchomić polecenie `npm run dev` za każdym razem, gdy zmieniasz komponent Vue. Lub możesz uruchomić komendę `npm run watch`, aby monitorować i automatycznie rekompilować swoje komponenty za każdym razem, gdy są modyfikowane.

Oczywiście, jeśli chcesz dowiedzieć się więcej na temat pisania komponentów Vue, powinieneś przeczytać [dokumentację Vue](https://vuejs.org/guide/), która zapewnia dokładny, łatwy do odczytania przegląd całego Framework-a Vue.

<a name="using-react"></a>
### Using React  - Używanie React

Jeśli wolisz używać React do budowania aplikacji JavaScript, Laravel sprawia, że możesz zamienić rusztowanie Vue na Rusztowania React. W dowolnej świeżej aplikacji Laravel możesz użyć polecenia `preset` z opcją` react`:

    php artisan preset react

To pojedyncze polecenie usunie rusztowanie Vue i zastąpi je Rusztowaniem React, w tym przykładowym komponencie.
