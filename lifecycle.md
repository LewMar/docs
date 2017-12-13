# Request Lifecycle - Cykl życia żądania

- [Introduction - Wprowadzenie](#introduction)
- [Lifecycle Overview - Przegląd cyklu życia](#lifecycle-overview)
- [Focus On Service Providers](#focus-on-service-providers)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Korzystając z dowolnego narzędzia w "prawdziwym świecie", czujesz się pewniej, jeśli rozumiesz, jak działa to narzędzie. Rozwój aplikacji nie jest inny. Kiedy zrozumiesz, jak działają twoje narzędzia programistyczne, czujesz się bardziej komfortowo i pewniej, używając ich.

Celem tego dokumentu jest zapewnienie dobrego, wysokiego poziomu przeglądu działania frameworka Laravel. Lepiej poznając ogólne ramy, wszystko wydaje się mniej "magiczne", a Ty będziesz bardziej pewny budowania swoich aplikacji. Jeśli nie rozumiesz wszystkich terminów od razu, nie trać serca! Po prostu spróbuj zrozumieć, co się dzieje, a twoja wiedza będzie rosnąć w miarę odkrywania innych części dokumentacji.

<a name="lifecycle-overview"></a>
## Lifecycle Overview - Przegląd cyklu życia

### First Things - Początkowe działania

Punktem wejścia dla wszystkich żądań do aplikacji Laravel jest plik `public/index.php`. Wszystkie żądania są kierowane do tego pliku przez plik konfiguracji serwera WWW (Apache / Nginx). Plik `index.php` nie zawiera dużo kodu. Raczej jest to po prostu punkt startowy do ładowania reszty frameworka.

Plik `index.php` ładuje definicję automatycznego generatora programu Composer, a następnie pobiera instancję aplikacji Laravel ze skryptu `bootstrap/app.php`. Pierwszą czynnością podjętą przez samego Laravel jest utworzenie instancji kontenera aplikacji / [service container - kontener usług](/docs/{{version}}/container).

### HTTP / Console Kernels - Jądra HTTP / konsola

Następnie przychodzące żądanie jest wysyłane do jądra HTTP lub jądra konsoli, w zależności od typu żądania, które wchodzi do aplikacji. Te dwa jądra służą jako centralna lokalizacja, przez którą przepływają wszystkie żądania. Na razie skupmy się tylko na jądrze HTTP, które znajduje się w `app/Http/Kernel.php`.

Jądro HTTP rozszerza klasę `Illuminate\Foundation\Http\Kernel`, która definiuje tablicę `bootstrappers` które będą uruchamiane przed wykonaniem żądania. Te bootstrappery konfigurują obsługę błędów, konfigurują rejestrowanie, [wykrywają środowisko aplikacji](/docs/{{version}}/configuration#environment-configuration), i wykonują inne zadania, które muszą być wykonane przed rzeczywistym obsłużeniem żądania.

Jądro HTTP definiuje również listę oprogramowania [middleware - pośredniczącego](/docs/{{version}}/middleware) HTTP, przez którą wszystkie żądania muszą przejść, zanim zostaną obsłużone przez aplikację. Te instrukcje oprogramowania pośredniego odczytują i zapisują [sesję HTTP](/docs/{{version}}/session), określając, czy aplikacja działa w trybie konserwacji [weryfikując token CSRF](/docs/{{version}}/csrf) i inne.

Sygnatura metody dla `uchwytu (handle)` obsługi jądra HTTP jest dość prosta: odbierz `Request (żądanie)` i zwróć `Response (odpowiedź)`. Pomyśl o jądrze jako dużym czarnym polu, które reprezentuje całą twoją aplikację. Nakarm go żądaniami HTTP i zwróci odpowiedzi HTTP.

#### Service Providers - Dostawcy usług

Jednym z najważniejszych działań związanych z ładowaniem jądra jest ładowanie [service providers (dostawców usług)](/docs/{{version}}/providers) dla twojej aplikacji. Wszyscy dostawcy usług dla aplikacji są skonfigurowani w tablicy `providers (dostawców)` pliku konfiguracyjnego `config/app.php`. Po pierwsze, metoda `register (rejestru)` będzie wywoływana u wszystkich dostawców, a po zarejestrowaniu wszystkich dostawców zostanie wywołana metoda `boot (rozruchu)`.

Dostawcy usług są odpowiedzialni za ładowanie wszystkich różnych komponentów struktury, takich jak baza danych, kolejkowanie, sprawdzanie poprawności i komponenty routingu. Ponieważ ładują i konfigurują wszystkie funkcje oferowane przez framework, dostawcy usług są najważniejszym aspektem całego procesu bootstrap Laravel.

#### Dispatch Request - Żądanie wysyłki

Po uruchomieniu aplikacji i zarejestrowaniu wszystkich dostawców usług, `Request (żądanie)` zostanie przekazane do routera w celu wysłania. Router wyśle żądanie do trasy lub kontrolera, a także uruchomi dowolne oprogramowanie pośredniczące dla trasy.

<a name="focus-on-service-providers"></a>
## Focus On Service Providers - Skoncentruj się na dostawcach usług

Dostawcy usług są naprawdę kluczem do uruchomienia aplikacji Laravel. Instancja aplikacji zostanie utworzona, dostawcy usług zostaną zarejestrowani, a żądanie zostanie przekazane do aplikacji bootstrapped. To naprawdę takie proste!

Posiadanie solidnego zrozumienia, w jaki sposób aplikacja Laravel jest tworzona i ładowana przez dostawców usług, jest bardzo cenne. Oczywiście domyślni dostawcy usług aplikacji są zapisani w katalogu `app/Providers`.

Domyślnie `AppServiceProvider` jest dość pusty. Ten dostawca jest doskonałym miejscem do dodawania własnych powiązań ładowania i obsługi kontenera usług. Oczywiście, w przypadku dużych aplikacji, możesz chcieć utworzyć kilku dostawców usług, z których każdy ma bardziej ziarnisty sposób ładowania.