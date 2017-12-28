# Views

- [Creating Views - Tworzenie widoków](#creating-views)
- [Passing Data To Views - Przekazywanie danych do widoków](#passing-data-to-views)
    - [Sharing Data With All Views - Udostępnianie danych przy wszystkich widokach](#sharing-data-with-all-views)
- [View Composers - Wyświetl kompozytorów](#view-composers)

<a name="creating-views"></a>
## Creating Views - Tworzenie widoków

> {tip} Szukasz więcej informacji na temat pisania szablonów Blade? Aby rozpocząć, zapoznaj się z pełną dokumentacją programu [Blade](/docs/{{version}}/blade) .

Widoki zawierają kod HTML obsługiwany przez aplikację i oddzielają logikę kontrolera / aplikacji od logiki prezentacji. Widoki są przechowywane w katalogu `resources/views`. Prosty widok może wyglądać mniej więcej tak:

    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

Ponieważ widok ten jest przechowywany w `resources/views/greeting.blade.php`, możemy go zwrócić używając globalnego helpera` view`, tak jak poniżej:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Jak widać, pierwszy argument przekazany do helpera `view` odpowiada nazwie pliku widoku w katalogu `resources/views`. Drugi argument to tablica danych, które powinny zostać udostępnione do widoku. W tym przypadku przekazujemy zmienną `name`, która jest wyświetlana w widoku przy użyciu [składni Blade](/docs/{{version}}/blade).

Oczywiście widoki mogą być również zagnieżdżane w podkatalogach katalogu `resources/views`. Notacja "kropka" może służyć do odniesienia widoków zagnieżdżonych. Na przykład, jeśli twój widok jest przechowywany w `resources/views/admin/profile.blade.php`, możesz odwoływać się do niego w następujący sposób:

    return view('admin.profile', $data);

#### Determining If A View Exists - Określanie czy widok istnieje

Jeśli chcesz określić, czy widok istnieje, możesz użyć fasady `View`. Metoda `exists` zwróci `true`, jeśli widok istnieje:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### Creating The First Available View - Tworzenie pierwszego dostępnego widoku

Korzystając z metody `first`, możesz utworzyć pierwszy widok, który istnieje w danej tablicy widoków. Jest to przydatne, jeśli aplikacja lub pakiet umożliwia dostosowywanie lub zastępowanie widoków:

    return view()->first(['custom.admin', 'admin'], $data);

Oczywiście możesz również wywołać tę metodę za pomocą `View` [fasada](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>
## Passing Data To Views - Przekazywanie danych do widoków

Jak widać w poprzednich przykładach, możesz przekazać tablicę danych do widoków:

    return view('greetings', ['name' => 'Victoria']);

Podczas przekazywania informacji w ten sposób dane powinny być tablicą zawierającą pary klucz / wartość. Wewnątrz widoku możesz uzyskać dostęp do każdej wartości za pomocą odpowiedniego klucza, na przykład `<?php echo $key; ?>`. Zamiast przekazywania pełnej tablicy danych do funkcji pomocniczej `view` możesz użyć metody `with` do dodania poszczególnych elementów danych do widoku:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Sharing Data With All Views - Udostępnianie danych przy wszystkich widokach

Czasami może być konieczne udostępnienie danych przy użyciu wszystkich widoków renderowanych przez aplikację. Możesz to zrobić za pomocą metody `share` fasady widoku. Zwykle powinieneś umieszczać wywołania `share` w metodzie `boot` usługodawcy. Możesz dodać je do `AppServiceProvider` lub wygenerować oddzielnego dostawcę usług, aby je pomieścić:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## View Composers - Wyświetl kompozytorów

Wyświetl kompozytorzy to wywołania zwrotne lub metody klasy, które są wywoływane podczas renderowania widoku. Jeśli dane, które chcesz powiązać z widokiem za każdym razem, gdy widok jest renderowany, kompozytor widoku może pomóc w uporządkowaniu tej logiki w jednej lokalizacji.

W tym przykładzie zarejestrujmy kompozytorów widoku w [dostawcy usług](/docs/{{version}}/providers). Użyjemy fasady `View`, aby uzyskać dostęp do podstawowej realizacji kontraktu `Illuminate\Contracts\View\Factory`. Pamiętaj, że Laravel nie zawiera domyślnego katalogu dla przeglądających kompozytorzy. Możesz je dowolnie organizować. Na przykład możesz utworzyć katalog `app/Http/ViewComposers`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} Pamiętaj, że jeśli utworzysz nowego dostawcę usług, który będzie zawierał rejestrację widoku użytkownika, musisz dodać dostawcę usług do tablicy `provider` w pliku konfiguracyjnym `config/app.php`.

Po zarejestrowaniu kompozytora metoda `ProfileComposer@compose` będzie wykonywana za każdym razem, gdy renderowany jest widok `profile`. Zdefiniujmy więc klasę kompozytora:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Tuż przed renderowaniem widoku metoda `compose` kompozytora jest wywoływana za pomocą instancji `Illuminate\View\View` . Możesz użyć metody `with` do powiązania danych z widokiem.

> {tip} Wszyscy kompozytorzy widoku są rozwiązywani za pośrednictwem [kontenera usług](/docs/{{version}}/container), więc możesz wpisać podpowiedz wszelkich zależności, których potrzebujesz w konstruktorze kompozytora.

#### Attaching A Composer To Multiple Views - Dołączanie kompozytora do wielu widoków

Możesz dołączyć kompozytor widoku do wielu widoków jednocześnie, przekazując tablicę widoków jako pierwszy argument metody `composer`:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

etoda `composer` również akceptuje znak `*` jako symbol wieloznaczny, umożliwiając dołączenie kompozytora do wszystkich widoków:

    View::composer('*', function ($view) {
        //
    });

#### View Creators - Wyświetl twórców

Widok **twórców** jest bardzo podobny do widoku kompozytorów; są one jednak wykonywane natychmiast po utworzeniu instancji zamiast czekania, aż widok będzie renderowany. Aby zarejestrować twórcę widoku, użyj metody `creator`:

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
