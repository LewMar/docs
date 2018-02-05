# File Storage

- [Introduction - Wprowadzenie](#introduction)
- [Configuration - Konfiguracja](#configuration)
    - [The Public Disk - Dysk publiczny](#the-public-disk)
    - [The Local Driver - Lokalny sterownik](#the-local-driver)
    - [Driver Prerequisites - Wymagania wstępne sterownika](#driver-prerequisites)
- [Obtaining Disk Instances - Uzyskiwanie wystąpień dyskowych](#obtaining-disk-instances)
- [Retrieving Files - Pobieranie plików](#retrieving-files)
    - [Downloading Files](#downloading-files)
    - [File URLs - Adresy URL plików](#file-urls)
    - [File Metadata - Adresy URL plików](#file-metadata)
- [Storing Files - Przechowywanie plików](#storing-files)
    - [File Uploads - Przesyłanie plików](#file-uploads)
    - [File Visibility - Widoczność pliku](#file-visibility)
- [Deleting Files - Usuwanie plików](#deleting-files)
- [Directories - Katalogi](#directories)
- [Custom Filesystems - Niestandardowe systemy plików](#custom-filesystems)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel zapewnia potężną abstrakcję systemu plików dzięki wspaniałemu pakietowi [Flysystem](https://github.com/thephpleague/flysystem) PHP autorstwa Franka de Jonge. Integracja Laravel Flysystem zapewnia proste w obsłudze sterowniki do pracy z lokalnymi systemami plików, Amazon S3 i Rackspace Cloud Storage. Co więcej, przełączanie się pomiędzy tymi opcjami jest niezwykle proste, ponieważ interfejs API pozostaje taki sam dla każdego systemu.

<a name="configuration"></a>
## Configuration - Konfiguracja

Plik konfiguracyjny systemu plików znajduje się w `config/filesystems.php`. W tym pliku możesz skonfigurować wszystkie swoje "dyski". Każdy dysk reprezentuje określony sterownik pamięci i miejsce przechowywania. Przykładowe konfiguracje dla każdego obsługiwanego sterownika są zawarte w pliku konfiguracyjnym. Zmodyfikuj konfigurację tak, aby odzwierciedlała preferencje i dane logowania do pamięci masowej.

Oczywiście możesz skonfigurować dowolną liczbę dysków, a nawet kilka dysków z tym samym sterownikiem.

<a name="the-public-disk"></a>
### The Public Disk - Dysk publiczny

Dysk `public` jest przeznaczony dla plików, które będą publicznie dostępne. Domyślnie dysk `public` używa sterownika `local` i przechowuje te pliki w `storage/app/public`. Aby były dostępne z internetu, należy utworzyć dowiązanie symboliczne z `public/storage` do `storage/app/public`. Ta konwencja będzie przechowywać publicznie dostępne pliki w jednym katalogu, który może być łatwo udostępniony we wszystkich wdrożeniach przy użyciu zerowych systemów wdrażania, takich jak [Envoyer](https://envoyer.io).

Aby utworzyć dowiązanie symboliczne, możesz użyć polecenia `storage:link` Artisan:

    php artisan storage:link

Oczywiście po zapisaniu pliku i utworzeniu dowiązania symbolicznego można utworzyć adres URL do plików przy użyciu helpera `asset`:

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### The Local Driver - Lokalny sterownik

Używając sterownika `local`, wszystkie operacje na plikach odnoszą się do katalogu `root` zdefiniowanego w pliku konfiguracyjnym. Domyślnie ta wartość jest ustawiona na katalog `storage/app`. Dlatego poniższa metoda zapisuje plik w `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### Driver Prerequisites - Wymagania wstępne sterownika

#### Composer Packages - Pakiety Composer-a

Przed użyciem sterowników S3 lub Rackspace należy zainstalować odpowiedni pakiet za pośrednictwem Composer:

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3 Driver Configuration - S3 konfiguracja sterownika

Informacje dotyczące konfiguracji sterownika S3 znajdują się w pliku konfiguracyjnym `config/filesystems.php`. Ten plik zawiera przykładową tablicę konfiguracji dla sterownika S3. Możesz modyfikować tę tablicę przy pomocy własnej konfiguracji S3 i poświadczeń. Dla wygody te zmienne środowiskowe są zgodne z konwencją nazewnictwa używaną przez interfejs wiersza polecenia AWS.

#### FTP Driver Configuration - Konfiguracja sterownika FTP

Integracja Laravel's Flysystem działa świetnie z FTP; jednak przykładowa konfiguracja nie jest dołączona do domyślnego pliku konfiguracyjnego `filesystems.php`. Jeśli potrzebujesz skonfigurować system plików FTP, możesz użyć przykładowej konfiguracji poniżej:

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Rackspace Driver Configuration - Konfiguracja sterownika Rackspace

Integracje systemu plików Laravel-a działają doskonale z Rackspace; jednak przykładowa konfiguracja nie jest dołączona do domyślnego pliku konfiguracyjnego `filesystems.php`. Jeśli potrzebujesz skonfigurować system plików Rackspace, możesz użyć przykładowej konfiguracji poniżej:

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="obtaining-disk-instances"></a>
## Obtaining Disk Instances - Uzyskiwanie wystąpień dyskowych

Fasada `Storage` może być używana do interakcji z dowolnym skonfigurowanym dyskiem. Na przykład możesz użyć metody `put` na elewacji, aby zapisać awatar na domyślnym dysku. Jeśli wywołasz metody na fasadzie `Storage` bez wcześniejszego wywołania metody` disk`, wywołanie metody zostanie automatycznie przekazane na dysk domyślny:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

Jeśli twoje aplikacje współdziałają z wieloma dyskami, możesz użyć metody `disk` na elewacji `Storage` do pracy z plikami na określonym dysku:

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## Retrieving Files - Pobieranie plików

Do pobrania zawartości pliku można użyć metody `get`. Surowa zawartość łańcucha pliku zostanie zwrócona przez metodę. Pamiętaj, że wszystkie ścieżki plików powinny być określone względem lokalizacji "root" skonfigurowanej dla dysku:

    $contents = Storage::get('file.jpg');

Do określenia, czy plik istnieje na dysku, można użyć metody `exists`:

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="downloading-files"></a>
### Downloading Files - Pobieranie plików

Metodę `download` można wykorzystać do wygenerowania odpowiedzi, która zmusza przeglądarkę użytkownika do pobrania pliku w podanej ścieżce. Metoda `download` akceptuje nazwę pliku jako drugi argument metody, który określi nazwę pliku widoczną dla użytkownika pobierającego plik. Na koniec możesz przekazać tablicę nagłówków HTTP jako trzeci argument metody:

    return Storage::download('file.jpg');

    return Storage::download('file.jpg', $name, $headers);

<a name="file-urls"></a>
### File URLs - Adresy URL plików

Możesz użyć metody `url`, aby uzyskać adres URL dla danego pliku. Jeśli używasz sterownika `local`, zwykle po prostu dodasz `/storage` do podanej ścieżki i zwrócisz względny adres URL do pliku. Jeśli używasz sterownika `s3` lub `rackspace`, zwrócony zostanie pełny zdalny adres URL:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file.jpg');

> {note} Pamiętaj, że jeśli używasz sterownika `local`, wszystkie pliki, które powinny być publicznie dostępne, powinny być umieszczone w katalogu  `storage/app/public`. Ponadto powinieneś [stworzyć dowiązanie symboliczne](#the-public-disk) w `public/storage`, które wskazuje na katalog `storage/app/public`.

#### Temporary URLs - Tymczasowe adresy URL

W przypadku plików przechowywanych za pomocą sterownika `s3` lub `rackspace` można utworzyć tymczasowy adres URL dla danego pliku, używając metody `temporaryUrl`. Ta metoda akceptuje ścieżkę i instancję `DateTime` określającą datę wygaśnięcia adresu URL:

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

#### Local URL Host Customization - Dostosowanie lokalnych adresów URL do potrzeb

Jeśli chcesz wstępnie zdefiniować host dla plików przechowywanych na dysku za pomocą sterownika `local`, możesz dodać opcję `url` do tablicy konfiguracji dysku:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### File Metadata - Metadane pliku

Oprócz czytania i zapisywania plików, Laravel może również udostępniać informacje o samych plikach. Na przykład można użyć metody `size`, aby uzyskać rozmiar pliku w bajtach:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

Metoda `lastModified` zwraca znacznik czasu UNIX po ostatniej modyfikacji pliku:

    $time = Storage::lastModified('file.jpg');

<a name="storing-files"></a>
## Storing Files - Przechowywanie plików

Metoda `put` może być używana do przechowywania nieprzetworzonej zawartości pliku na dysku. Można również przekazać PHP `resource` do `put` metody, która będzie wykorzystać podstawową obsługę strumienia Flysystem użytkownika. Używanie strumieni jest bardzo zalecane w przypadku dużych plików:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### Automatic Streaming - Automatyczne przesyłanie strumieniowe

Jeśli chcesz, aby Laravel automatycznie zarządzał przesyłaniem strumieniowym danego pliku do miejsca przechowywania, możesz użyć metody `putFile` lub` putFileAs`. Ta metoda akceptuje instancję `Illuminate\Http\File` lub `Illuminate\Http\UploadedFile` i automatycznie przesyła plik do wybranej lokalizacji:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Automatically generate a unique ID for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a file name...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Jest kilka ważnych rzeczy do zapamiętania na temat metody `putFile`. Zwróć uwagę, że określiliśmy tylko nazwę katalogu, a nie nazwę pliku. Domyślnie metoda `putFile` generuje unikalny identyfikator służący jako nazwa pliku. Ścieżka do pliku zostanie zwrócona przez metodę `putFile`, dzięki czemu możesz zapisać ścieżkę, w tym wygenerowaną nazwę pliku, w bazie danych.

Metody `putFile` i` putFileAs` również przyjmują argument określający "widoczność" przechowywanego pliku. Jest to szczególnie przydatne, jeśli przechowujesz plik na dysku w chmurze, taki jak S3 i chcesz, aby plik był publicznie dostępny:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### Prepending & Appending To Files - Przygotowywanie i dołączanie do plików

Metody `prepend` i `append` umożliwiają zapisanie na początku lub końcu pliku:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### Copying & Moving Files - Kopiowanie i przenoszenie plików

Metodę `copy` można użyć do skopiowania istniejącego pliku do nowej lokalizacji na dysku, natomiast metoda `move` może posłużyć do zmiany nazwy lub przeniesienia istniejącego pliku do nowej lokalizacji:

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="file-uploads"></a>
### File Uploads - Przesyłanie plików

W aplikacjach internetowych jednym z najczęściej używanych przypadków przechowywania plików jest przechowywanie przesłanych przez użytkownika plików, takich jak zdjęcia profilowe, zdjęcia i dokumenty. Laravel bardzo ułatwia przechowywanie przesłanych plików za pomocą metody `store` na przesłanej instancji pliku. Wywołaj metodę `store` ze ścieżką, w której chcesz zapisać przesłany plik:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * Update the avatar for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

Jest kilka ważnych rzeczy do zapamiętania na temat tego przykładu. Zwróć uwagę, że określiliśmy tylko nazwę katalogu, a nie nazwę pliku. Domyślnie metoda `store` generuje unikalny identyfikator służący jako nazwa pliku. Ścieżka do pliku zostanie zwrócona przez metodę `store`, dzięki czemu możesz zapisać ścieżkę, w tym wygenerowaną nazwę pliku, w bazie danych.

Możesz również wywołać metodę `putFile` na elewacji `Storage`, aby wykonać tę samą operację na pliku jak w powyższym przykładzie:

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### Specifying A File Name - Określanie nazwy pliku

Jeśli nie chcesz, aby nazwa pliku była automatycznie przypisywana do przechowywanego pliku, możesz użyć metody `storeAs`, która otrzymuje ścieżkę, nazwę pliku i (opcjonalny) dysk jako swoje argumenty:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Oczywiście możesz również użyć metody `putFileAs` na elewacji `Storage`, która wykona tę samą operację na pliku jak w powyższym przykładzie:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### Specifying A Disk - Określanie dysku

Domyślnie ta metoda użyje domyślnego dysku. Jeśli chcesz określić inny dysk, podaj nazwę dysku jako drugi argument metody `store`:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### File Visibility - Widoczność pliku

W integracji laravel z Flysystem „widoczność” jest abstrakcją i uprawnienia do plików na różnych platformach. Pliki mogą być zadeklarowane jako `public` i `private`. Kiedy plik jest deklarowany jako `public`, oznacza to, że plik powinien być ogólnie dostępny dla innych. Na przykład, podczas korzystania ze sterownika S3, możesz pobrać adresy URL dla plików `public`.

Możesz ustawić widoczność podczas ustawiania pliku za pomocą metody `put`:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

Jeśli plik został już zapisany, jego widoczność można pobrać i ustawić za pomocą metod `getVisibility` i` setVisibility`:

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## Deleting Files - Usuwanie plików

Metoda `delete` akceptuje pojedynczą nazwę pliku lub tablicę plików do usunięcia z dysku:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

W razie potrzeby możesz określić dysk, z którego plik ma zostać usunięty:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>
## Directories - Katalogi

#### Get All Files Within A Directory - Pobierz wszystkie pliki w katalogu

Metoda `files` zwraca tablicę wszystkich plików w danym katalogu. Jeśli chcesz pobrać listę wszystkich plików w danym katalogu zawierającym wszystkie podkatalogi, możesz użyć metody `allFiles`:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### Get All Directories Within A Directory - Pobierz wszystkie katalogi w katalogu

Metoda `directories` zwraca tablicę wszystkich katalogów w danym katalogu. Dodatkowo, możesz użyć metody `allDirectories`, aby uzyskać listę wszystkich katalogów w danym katalogu i wszystkich jego podkatalogach:

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### Create A Directory - Utwórz katalog

Metoda `makeDirectory` utworzy dany katalog, w tym wszelkie potrzebne podkatalogi:


    Storage::makeDirectory($directory);

#### Delete A Directory - Usuń katalog

Wreszcie, `deleteDirectory` może zostać użyty do usunięcia katalogu i wszystkich jego plików:


    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## Custom Filesystems - Niestandardowe systemy plików

Integracja laravel z Flysystem dostarcza sterowniki dla wielu „sterowników” na start; jednak Flysystem nie ogranicza się do tych i ma adaptery do wielu innych systemów pamięci masowej. Możesz utworzyć niestandardowy sterownik, jeśli chcesz użyć jednego z tych dodatkowych adapterów w swojej aplikacji Laravel.

Aby skonfigurować niestandardowy system plików, potrzebujesz adaptera Flysystem. Dodajmy społecznie obsługiwany adapter Dropbox do naszego projektu:


    composer require spatie/flysystem-dropbox

Następnie powinieneś utworzyć [dostawcę usług](/docs/{{version}}/providers), takich jak `DropboxServiceProvider`. W metodzie `boot` dostawcy możesz użyć metody `extend` fasady, aby zdefiniować niestandardowy sterownik:

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Illuminate\Support\ServiceProvider;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorizationToken']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Pierwszym argumentem metody `extend` jest nazwa sterownika, a druga to Closure, które otrzymuje zmienne `$app` i `$config`. Closure rozwiazywacza musi zwrócić instancję `League\Flysystem\Filesystem`. Zmienna `$config` zawiera wartości zdefiniowane w `config/filesystems.php` dla określonego dysku.

Po utworzeniu dostawcy usług, aby zarejestrować rozszerzenie, możesz użyć sterownika `dropbox` w pliku konfiguracyjnym `config/filesystems.php`.
