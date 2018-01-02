# Compiling Assets (Laravel Mix)

- [Introduction - Wprowadzenie](#introduction)
- [Installation & Setup - Instalacja i konfiguracja](#installation)
- [Running Mix - Uruchamianie Mix](#running-mix)
- [Working With Stylesheets - Praca z arkuszami stylów](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [Plain CSS - Spłaszczony CSS](#plain-css)
    - [URL Processing - Przetwarzanie URL](#url-processing)
    - [Source Maps - Mapy źródłowe](#css-source-maps)
- [Working With JavaScript - Praca z JavaScript](#working-with-scripts)
    - [Vendor Extraction - Ekstrakcja dostawcy](#vendor-extraction)
    - [React](#react)
    - [Vanilla JS](#vanilla-js)
    - [Custom Webpack Configuration - Niestandardowa konfiguracja pakietu Webpack](#custom-webpack-configuration)
- [Copying Files & Directories - Kopiowanie plików i katalogów](#copying-files-and-directories)
- [Versioning / Cache Busting - Wersjonowanie / Pomijanie pamięci podręcznej](#versioning-and-cache-busting)
- [BrowserSync Reloading - Przeładowanie BrowserSync](#browsersync-reloading)
- [Environment Variables - Zmienne środowiska](#environment-variables)
- [Notifications - Powiadomienia](#notifications)

<a name="introduction"></a>
## Introduction - Wprowadzenie

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) zapewnia płynne API do definiowania kroków budowania Webpack dla aplikacji Laravel przy użyciu kilku popularnych pre-procesorów CSS i JavaScript. Dzięki prostemu łańcuchowi metod można płynnie zdefiniować potok zasobów. Na przykład:

    mix.js('resources/assets/js/app.js', 'public/js')
       .sass('resources/assets/sass/app.scss', 'public/css');

Jeśli kiedykolwiek byłeś zdezorientowany i przytłoczony tematem rozpoczęcia pracy z pakietem Webpack i kompilacją zasobów, pokochasz Laravel Mix. Jednak nie musisz go używać podczas tworzenia aplikacji. Oczywiście możesz dowolnie korzystać z dowolnego narzędzia do zarządzania zasobami lub nawet z niego nie korzystać.

<a name="installation"></a>
## Installation & Setup - Instalacja i konfiguracja

#### Installing Node - Instalowanie Node

Zanim uruchomisz Mix, musisz najpierw upewnić się, że Node.js i NPM są zainstalowane na Twoim komputerze.

    node -v
    npm -v

Domyślnie Laravel Homestead zawiera wszystko, czego potrzebujesz; Jeśli jednak nie używasz Vagrant, możesz łatwo zainstalować najnowszą wersję Node i NPM, używając prostych instalatorów graficznych z [ich strony pobierania](https://nodejs.org/en/download/).

#### Laravel Mix

Jedynym pozostałym krokiem jest zainstalowanie Laravel Mix. W nowej instalacji Laravel znajdziesz plik `package.json` w katalogu głównym struktury katalogów. Domyślny plik `package.json` zawiera wszystko, czego potrzebujesz, aby zacząć. Pomyśl o tym jak o pliku `composer.json`, z wyjątkiem tego, że definiuje zależności węzła zamiast PHP. Możesz zainstalować zależności, do których się odwołuje, uruchamiając:

    npm install

<a name="running-mix"></a>
## Running Mix - Uruchamianie Mix

Mix jest górna warstwą konfiguracyjną [Webpack](https://webpack.js.org), więc aby uruchomić zadania Mix, wystarczy wykonać jeden ze skryptów NPM, który jest dołączony do domyślnego pakietu Laravel `package.json `plik:

    // Run all Mix tasks...
    npm run dev

    // Run all Mix tasks and minify output...
    npm run production

#### Watching Assets For Changes - Wypatrywanie zasobów dla zmian

Polecenie `npm run watch` będzie nadal działać w twoim terminalu i będzie obserwować wszystkie odpowiednie pliki dla zmian. Pakiet Webpack automatycznie przekompiluje zasoby po wykryciu zmiany:

    npm run watch

Może się okazać, że w niektórych środowiskach Webpack nie aktualizuje się po zmianie plików. Jeśli tak jest w twoim systemie, rozważ użycie polecenia `watch-poll`:

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## Working With Stylesheets - Praca z arkuszami stylów

Plik `webpack.mix.js` jest punktem wejścia dla wszystkich kompilacji zasobów. Pomyśl o tym, jako o lekkim opakowaniu konfiguracji w Webpack. Wymieszane zadania można połączyć ze sobą, aby dokładnie określić, jak należy skompilować zasoby.

<a name="less"></a>
### Less

Metoda `less` może być używana do kompilacji [Less](http://lesscss.org/) do CSS. Skompilujmy nasz podstawowy plik `app.less` do `public/css/app.css`.

    mix.less('resources/assets/less/app.less', 'public/css');

Wiele połączeń do metody `less` można wykorzystać do kompilacji wielu plików:

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

Jeśli chcesz dostosować nazwę skompilowanego pliku CSS, możesz przekazać pełną ścieżkę do pliku jako drugi argument metody `less`:

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

Jeśli chcesz zastąpić [opcje podstawowych wtyczek Less](https://github.com/webpack-contrib/less-loader#options), możesz przekazać obiekt jako trzeci argument do `mix.less()`:

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

Metoda `sass` pozwala na kompilację [Sass](http://sass-lang.com/) do CSS. Możesz użyć takiej metody jak:

    mix.sass('resources/assets/sass/app.scss', 'public/css');

Znowu, podobnie jak metoda `less`, możesz skompilować wiele plików Sass do swoich własnych plików CSS, a nawet dostosować katalog wyjściowy wynikowego CSS:

    mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

Jako trzeci argument można podać dodatkowe [opcje wtyczki Node-Sass](https://github.com/sass/node-sass#options):

    mix.sass('resources/assets/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

Podobnie jak Less i Sass, metoda `stylus` pozwala na kompilację [Stylus](http://stylus-lang.com/) w CSS:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css');

Możesz także zainstalować dodatkowe wtyczki Stylus, takie jak [Rupture](https://github.com/jescalan/rupture). Najpierw zainstaluj odpowiednią wtyczkę poprzez NPM (`npm install rupture`), a następnie wymagaj jej w swoim wywołaniu do `mix.stylus()`:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

[PostCSS](http://postcss.org/), potężne narzędzie do przekształcania twojego CSS, jest dołączone do Laravel Mix po wyjęciu z pudełka. Domyślnie Mix wykorzystuje popularną wtyczkę [Autoprefixer](https://github.com/postcss/autoprefixer), aby automatycznie zastosować wszystkie niezbędne przedrostki dostawcy CSS3. Możesz jednak dodać dodatkowe wtyczki odpowiednie dla aplikacji. Najpierw zainstaluj żądaną wtyczkę za pomocą NPM, a następnie odwołaj ją do pliku `webpack.mix.js`:

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });

<a name="plain-css"></a>
### Plain CSS - Spłaszczony CSS

Jeśli chciałbyś połączyć kilka zwykłych arkuszy stylów CSS w jeden plik, możesz użyć metody `styles`.

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>
### URL Processing - Przetwarzanie URL

Ponieważ Laravel Mix jest zbudowany na pakiecie Webpack, ważne jest zrozumienie kilku koncepcji pakietu Webpack. Do kompilacji CSS, Webpack przerobi i zoptymalizuje wszystkie wywołania `url()` w twoich arkuszach stylów. Chociaż może to brzmieć początkowo dziwnie, jest to niesamowicie potężna funkcja. Wyobraź sobie, że chcemy skompilować Sass, który zawiera względny adres URL do obrazu:

    .example {
        background: url('../images/example.png');
    }

> {note} Ścieżki bezwzględne dla danego `url()` zostaną wykluczone z przepisywania adresów URL. Na przykład `url('/images/thing.png')` lub `url('http://example.com/images/thing.png')` nie zostaną zmodyfikowane.

Domyślnie Laravel Mix i Webpack odnajdą `example.png`, skopiują go do folderu `public/images`, a następnie przepisują `url()` w wygenerowanym arkuszu stylów. W związku z tym Twój skompilowany CSS będzie:

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

Tak użyteczna jak ta funkcja może się okazać, że istniejąca struktura folderów jest już skonfigurowana w sposób, jaki lubisz. W takim przypadku możesz wyłączyć przepisywanie `url()` w taki sposób:

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });

Dzięki temu dodatkowi do twojego pliku `webpack.mix.js`, Mix nie będzie już pasował do żadnego `url()` lub kopiował aktywa do twojego publicznego katalogu. Innymi słowy, skompilowany CSS będzie wyglądał tak, jak pierwotnie go wpisałeś:

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### Source Maps - Mapy źródłowe

Chociaż domyślnie wyłączone, mapy źródłowe można aktywować, wywołując metodę `mix.sourceMaps()` w pliku `webpack.mix.js`. Mimo że wiąże się to z kosztem kompilacji / wydajności, zapewni to dodatkowe informacje dotyczące debugowania narzędzi programistycznych przeglądarki podczas korzystania ze skompilowanych zasobów.


    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## Working With JavaScript - Praca z JavaScript

Mix udostępnia kilka funkcji ułatwiających pracę z plikami JavaScript, takich jak kompilowanie ECMAScript 2015, łączenie modułów, minimalizowanie i łączenie zwykłych plików JavaScript. Co więcej, wszystko działa bezproblemowo, nie wymagając ani odrobiny niestandardowej konfiguracji:


    mix.js('resources/assets/js/app.js', 'public/js');

Dzięki tej jednej linii kodu możesz teraz skorzystać z:

<div class="content-list" markdown="1">
- Składnia ES2015.
- Moduły
- Kompilacja plików `.vue`.
- Minifikacji dla środowisk produkcyjnych.
</div>

<a name="vendor-extraction"></a>
### Vendor Extraction - Ekstrakcja dostawcy

Potencjalną wadą łączenia całego skryptu JavaScript specyficznego dla aplikacji z bibliotekami dostawców jest to, że utrudnia buforowanie długoterminowe. Na przykład pojedyncza aktualizacja kodu aplikacji zmusi przeglądarkę do ponownego pobrania wszystkich bibliotek dostawców, nawet jeśli nie uległy one zmianie.

Jeśli zamierzasz dokonywać częstych aktualizacji kodu JavaScript aplikacji, powinieneś rozważyć wyodrębnienie wszystkich bibliotek dostawców do własnego pliku. W ten sposób zmiana kodu aplikacji nie wpłynie na buforowanie dużego pliku `vendor.js`. Metoda "ekstrakcji" Mixa sprawia, że jest to bardzo proste:

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

Metoda `extract` akceptuje tablicę wszystkich bibliotek lub modułów, które chcesz wyodrębnić do pliku `vendor.js`. Używając powyższego fragmentu jako przykładu, Mix wygeneruje następujące pliki:

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *Środowisko wykonawcze manifestu pakietu Webpack*
- `public/js/vendor.js`: *Twoje biblioteki dostawców*
- `public/js/app.js`: *Twój kod aplikacji*
</div>

Aby uniknąć błędów JavaScript, należy załadować te pliki w odpowiedniej kolejności:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>
### React

Mix może automatycznie zainstalować wtyczki Babel niezbędne do obsługi React. Aby rozpocząć, zamień wywołanie `mix.js()` na `mix.react()`:

    mix.react('resources/assets/js/app.jsx', 'public/js');

Za kulisami Mix pobierze i doda odpowiednią wtyczkę `babel-preset-react` Babel.


<a name="vanilla-js"></a>
### Vanilla JS

Podobnie jak w przypadku łączenia arkuszy stylów z `mix.styles()`, możesz również łączyć i minimalizować dowolną liczbę plików JavaScript za pomocą metody `scripts()`:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

Ta opcja jest szczególnie przydatna w przypadku starszych projektów, w których nie jest wymagana kompilacja Webpack dla JavaScript.

> {tip} Niewielką odmianą `mix.scripts()` jest `mix.babel()`. Jego sygnatura metody jest identyczna z `scripts`; jednak połączony plik otrzyma kompilację Babel, która tłumaczy każdy kod ES2015 na vanilla JavaScript, który zrozumieją wszystkie przeglądarki.

<a name="custom-webpack-configuration"></a>
### Custom Webpack Configuration - Niestandardowa konfiguracja pakietu Webpack

Za kulisami Laravel Mix odwołuje się do wstępnie skonfigurowanego pliku `webpack.config.js`, aby uruchomić i uruchomić tak szybko, jak to możliwe. Czasami konieczne może być ręczne zmodyfikowanie tego pliku. Możesz mieć specjalny program ładujący lub wtyczkę, do której należy się odwoływać, a może wolisz używać Stylus-a zamiast Sass-a. W takich przypadkach masz dwie możliwości:

#### Merging Custom Configuration - Scalanie konfiguracji niestandardowej

Mix dostarcza użyteczną metodę `webpackConfig`, która pozwala łączyć dowolne krótkie nadpisania konfiguracji Webpack. Jest to szczególnie interesujący wybór, ponieważ nie wymaga kopiowania i utrzymywania własnej kopii pliku `webpack.config.js`. Metoda `webpackConfig` akceptuje obiekt, który powinien zawierać dowolną [konfigurację specyficzną dla Webpacka](https://webpack.js.org/configuration/), którą chcesz zastosować.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### Custom Configuration Files - Niestandardowe pliki konfiguracyjne

Jeśli chcesz całkowicie dostosować konfigurację Webpacka, skopiuj plik `node_modules/laravel-mix/setup/webpack.config.js` do katalogu głównego projektu. Następnie skieruj wszystkie odwołania do `--config` w pliku `package.json` do nowo skopiowanego pliku konfiguracyjnego. Jeśli zdecydujesz się na takie podejście do dostosowywania, wszelkie przyszłe aktualizacje upstream do `webpack.config.js` Mix-a muszą zostać ręcznie połączone w dostosowany plik.

<a name="copying-files-and-directories"></a>
## Copying Files & Directories - Kopiowanie plików i katalogów

Metoda `copy` może być używana do kopiowania plików i katalogów do nowych lokalizacji. Może to być przydatne, gdy konkretny zasób w twoim katalogu `node_modules` musi zostać przeniesiony do twojego folderu `public`.

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

Podczas kopiowania katalogu metoda `copy` spłaszcza strukturę katalogu. Aby zachować oryginalną strukturę katalogu, należy zamiast tego użyć metody `copyDirectory`:

    mix.copyDirectory('assets/img', 'public/img');

<a name="versioning-and-cache-busting"></a>
## Versioning / Cache Busting - Wersjonowanie / Pomijanie pamięci podręcznej

Wielu programistów dodaje do swoich skompilowanych zasobów znacznik czasu lub unikalny token, aby zmusić przeglądarki do załadowania nowych zasobów zamiast podawania nieaktualnych kopii kodu. Mix może obsłużyć to za pomocą metody `version`.

Metoda `version` automatycznie doda unikalny skrót do nazw plików wszystkich skompilowanych plików, pozwalając na wygodniejsze pomijanie pamięci podręcznej:

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

Po wygenerowaniu wersjonowanego pliku nie będziesz znać dokładnej nazwy pliku. Powinieneś więc użyć globalnej funkcji `mix` Laravel-a w swoich [widokach](/docs/{{version}}/views), aby załadować odpowiednio zakodowany zasób. Funkcja `mix` automatycznie określi bieżącą nazwę pliku mieszanego:

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">

Ponieważ wersjonowane pliki są zazwyczaj niepotrzebne w fazie rozwoju, możesz nakazać procesowi wersjonowania, aby działał tylko podczas `npm run production`:

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="browsersync-reloading"></a>
## BrowseSync Reloading - Przeładowanie BrowserSync

[BrowserSync](https://browsersync.io/) może automatycznie monitorować pliki w poszukiwaniu zmian i wprowadzać zmiany w przeglądarce bez konieczności ręcznego odświeżania. Możesz włączyć obsługę, wywołując metodę `mix.browserSync()`:

    mix.browserSync('my-domain.test');

    // Or...

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.test'
    });

Możesz przekazać do tej metody ciąg (proxy) lub obiekt (ustawienia BrowserSync). Następnie uruchom serwer deweloperski Webpack za pomocą komendy `npm run watch`. Teraz, gdy modyfikujesz skrypt lub plik PHP, obserwuj jak przeglądarka odświeża stronę, aby odzwierciedlić zmiany.

<a name="environment-variables"></a>
## Environment Variables - Zmienne środowiska

Możesz wstawiać zmienne środowiskowe do Mix, prefiksując klucz w swoim pliku `.env` za pomocą` MIX_`:

    MIX_SENTRY_DSN_PUBLIC=http://example.com

Po zdefiniowaniu zmiennej w pliku `.env` możesz uzyskać dostęp za pośrednictwem obiektu `process.env`. Jeśli wartość zmieni się podczas wykonywania zadania `watch`, konieczne będzie ponowne uruchomienie zadania:

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## Notifications - Powiadomienia

Gdy będzie dostępny, Mix automatycznie wyświetli powiadomienia systemu operacyjnego dla każdego pakietu. Zapewni to natychmiastową informację zwrotną, czy kompilacja zakończyła się powodzeniem, czy nie. Jednak mogą wystąpić przypadki wyłączenia tych powiadomień. Jednym z takich przykładów może być wyzwolenie Mix na serwerze produkcyjnym. Powiadomienia można dezaktywować za pomocą metody `disableNotifications`.

    mix.disableNotifications();
