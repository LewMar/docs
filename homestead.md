# Laravel Homestead

- [Introduction - Wprowadzenie](#introduction)
- [Installation & Setup - Instalacja i konfiguracja](#installation-and-setup)
    - [First Steps - Pierwsze kroki](#first-steps)
    - [Configuring Homestead - Konfigurowanie Homestead](#configuring-homestead)
    - [Launching The Vagrant Box - Uruchomienie The Vagrant Box](#launching-the-vagrant-box)
    - [Per Project Installation - Instalacja do poszczegulnego Projektu (Spersonalizowane)](#per-project-installation)
    - [Installing MariaDB - Instalacja MariaDB](#installing-mariadb)
    - [Installing Elasticsearch - Instalacja Elasticsearch](#installing-elasticsearch)
    - [Aliases - Aliasy](#aliases)
- [Daily Usage - Codzienne użytkowanie](#daily-usage)
    - [Accessing Homestead Globally - Dostęp do Homestead globalnie](#accessing-homestead-globally)
    - [Connecting Via SSH - Połaczenie przez SSH](#connecting-via-ssh)
    - [Connecting To Databases - Łączenie z bazami danych](#connecting-to-databases)
    - [Adding Additional Sites - Dodawanie dodatkowych stron](#adding-additional-sites)
    - [Environment Variables - Zmienne środowiska](#environment-variables)
    - [Configuring Cron Schedules - Konfigurowanie harmonogramów Cron](#configuring-cron-schedules)
    - [Configuring Mailhog - Konfigurowanie Mailhog](#configuring-mailhog)
    - [Ports - Porty](#ports)
    - [Sharing Your Environment - Dzielenie się środowiskiem](#sharing-your-environment)
    - [Multiple PHP Versions - Wiele wersji PHP](#multiple-php-versions)
- [Network Interfaces - Interfejsy sieciowe](#network-interfaces)
- [Updating Homestead - Aktualizacja Homestead](#updating-homestead)
- [Provider Specific Settings - Ustawienia określone przez dostawcę](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Firma Laravel dokłada wszelkich starań, aby cały proces programowania PHP był przyjemny, w tym lokalne środowisko programistyczne. [Vagrant](https://www.vagrantup.com) zapewnia prosty, elegancki sposób zarządzania i udostępniania maszyn wirtualnych.

Laravel Homestead to oficjalny, pakowany pakiet Vagrant, który zapewnia wspaniałe środowisko programistyczne bez konieczności instalowania PHP, serwera WWW i innego oprogramowania serwera na komputerze lokalnym. Nigdy więcej nie martw się o zepsucie systemu operacyjnego! Pudełka Vagrant są całkowicie jednorazowe. Jeśli coś pójdzie nie tak, możesz zniszczyć i ponownie utworzyć skrzynię w kilka minut!

Homestead działa na dowolnym systemie Windows, Mac lub Linux i zawiera serwer WWW Nginx, PHP 7.2, PHP 7.1, PHP 7.0, PHP 5.6, MySQL, PostgreSQL, Redis, Memcached, Node i wszystkie inne potrzebne do rozwijać niesamowite aplikacje Laravel.

> {note} Jeśli korzystasz z systemu Windows, może być konieczne włączenie wirtualizacji sprzętowej (VT-x). Zwykle można go włączyć poprzez BIOS. Jeśli używasz Hyper-V w systemie UEFI, możesz dodatkowo wyłączyć Hyper-V, aby uzyskać dostęp do VT-x.

<a name="included-software"></a>
### Included Software - Dołączone oprogramowanie

<div class="content-list" markdown="1">
- Ubuntu 16.04
- Git
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL
- MariaDB
- Sqlite3
- PostgreSQL
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- ngrok
</div>

<a name="installation-and-setup"></a>
## Installation & Setup - Instalacja i konfiguracja

<a name="first-steps"></a>
### First Steps - Pierwsze kroki

Przed uruchomieniem środowiska Homestead należy zainstalować [VirtualBox 5.2](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com), [Parallels](https://www.parallels.com/products/desktop/) lub [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) oraz [Vagrant](https://www.vagrantup.com/downloads.html). Wszystkie te pakiety oprogramowania zapewniają łatwe w użyciu wizualne instalatory dla wszystkich popularnych systemów operacyjnych.

Aby skorzystać z dostawcy VMware, musisz zakupić zarówno VMware Fusion / Workstation, jak i [wtyczkę VMware Vagrant](https://www.vagrantup.com/vmware). Choć nie jest to darmowe, VMware może zapewnić szybsze udostępnianie folderów wydajności po wyjęciu z pudełka.

Aby skorzystać z usługi Parallels, musisz zainstalować [wtyczkę Parallels Vagrant](https://github.com/Parallels/vagrant-parallels). To jest bezpłatne.

Ze względu na [ograniczenia Vagrant](https://www.vagrantup.com/docs/hyperv/limitations.html) dostawca Hyper-V ignoruje wszystkie ustawienia sieciowe.

#### Installing The Homestead Vagrant Box - Instalowanie The Homestead Vagrant Box

Po zainstalowaniu VirtualBox / VMware i Vagrant powinieneś dodać pudełko `laravel/homestead` do swojej instalacji Vagrant za pomocą następującego polecenia w twoim terminalu. Pobranie pudełka zajmie kilka minut, w zależności od szybkości połączenia internetowego:

    vagrant box add laravel/homestead

Jeśli to polecenie się nie powiedzie, upewnij się, że instalacja Vagrant jest aktualna.

#### Installing Homestead - Instalowanie Homestead

Możesz zainstalować Homestead po prostu klonując repozytorium. Rozważ sklonowanie repozytorium do folderu `Homestead` w twoim katalogu domowym, ponieważ pole Homestead będzie służyć jako host dla wszystkich twoich projektów Laravel:

    git clone https://github.com/laravel/homestead.git ~/Homestead

Powinieneś sprawdzić oznaczoną wersję Homestead, ponieważ gałąź `master` nie zawsze jest stabilna. Najnowszą stabilną wersję możesz znaleźć na stronie [Strona wydania GitHub](https://github.com/laravel/homestead/releases):

    cd ~/Homestead

    // Clone the desired release...
    git checkout v7.0.1

Po sklonowaniu repozytorium Homestead, uruchom komendę `bash init.sh` z katalogu Homestead, aby utworzyć plik konfiguracyjny `Homestead.yaml`. Plik `Homestead.yaml` zostanie umieszczony w katalogu Homestead:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Configuring Homestead - Konfigurowanie Homestead

#### Setting Your Provider - Ustawianie dostawcy

Klucz `provider` w pliku` Homestead.yaml` wskazuje, który dostawca Vagrant powinien zostać użyty: `virtualbox`, `vmware_fusion`, `vmware_workstation`, `parallels` lub `hyperv`. Możesz ustawić to dla preferowanego operatora:

    provider: virtualbox

#### Configuring Shared Folders - Konfigurowanie folderów współdzielonych

Właściwość `folders` pliku` Homestead.yaml` wyświetla listę wszystkich folderów, które chcesz udostępnić w środowisku Homestead. Ponieważ pliki w tych folderach zostaną zmienione, będą one zsynchronizowane między lokalnym komputerem a środowiskiem Homestead. W razie potrzeby możesz skonfigurować dowolną liczbę folderów współdzielonych:

    folders:
        - map: ~/code
          to: /home/vagrant/code

Jeśli tworzysz tylko kilka witryn, to ogólne mapowanie będzie działało dobrze. Ponieważ liczba witryn nadal rośnie, mogą wystąpić problemy z wydajnością. Ten problem może być boleśnie widoczny na urządzeniach lub projektach niższego rzędu, które zawierają bardzo dużą liczbę plików. Jeśli występuje ten problem, spróbuj zmapować każdy projekt do jego własnego folderu Vagrant:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/code/project1

        - map: ~/code/project2
          to: /home/vagrant/code/project2

Aby włączyć [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), wystarczy dodać prostą flagę do konfiguracji zsynchronizowanego folderu:


    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "nfs"

> {note} Korzystając z NFS, powinieneś rozważyć zainstalowanie wtyczki [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs). Ta wtyczka zachowa prawidłowe uprawnienia użytkownika / grupy dla plików i katalogów w pudełku Homestead.

Możesz również przekazać dowolne opcje obsługiwane przez [Zsynchronizowane foldery](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) Vagrant-a, wymieniając je pod kluczem `options`:

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

#### Configuring Nginx Sites - Configuring Nginx Sites

Nie znasz Nginx? Nie ma problemu. Właściwość `sites` umożliwia łatwe mapowanie "domeny" do folderu w środowisku Homestead. Przykładowa konfiguracja witryny jest zawarta w pliku `Homestead.yaml`. Również w razie potrzeby możesz dodać tyle witryn do swojego środowiska Homestead. Homestead może służyć jako wygodne, zwirtualizowane środowisko dla każdego projektu Laravel, nad którym pracujesz:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public

Jeśli zmienisz właściwość `sites` po udostępnieniu pudełka Homestead, powinieneś ponownie uruchomić `vagrant reload --provision`, aby zaktualizować konfigurację Nginx na maszynie wirtualnej.

#### The Hosts File - Plik hostów

Musisz dodać "domeny" do swoich stron Nginx do pliku `hosts` na twoim komputerze. Plik `hosts` przekieruje żądania dotyczące twoich stron Homestead do twojej maszyny Homestead. W systemach Mac i Linux plik ten znajduje się w `/etc/hosts`. W systemie Windows znajduje się w katalogu `C:\Windows\System32\drivers\etc\hosts`. Wiersze, które dodasz do tego pliku, będą wyglądały następująco:

    192.168.10.10  homestead.test

Upewnij się, że podany adres IP jest ustawiony w pliku `Homestead.yaml`. Po dodaniu domeny do pliku `hosts` i uruchomieniu skrzynki Vagrant będziesz mógł uzyskać dostęp do witryny za pośrednictwem przeglądarki:

    http://homestead.test

<a name="launching-the-vagrant-box"></a>
### Launching The Vagrant Box - Uruchomienie The Vagrant Box

Po zakończeniu edycji `Homestead.yaml` uruchom polecenie `vagrant up` z katalogu Homestead. Vagrant uruchomi maszynę wirtualną i automatycznie skonfiguruje foldery udostępnione i witryny Nginx.

Aby zniszczyć maszynę, możesz użyć polecenia `vagrant destroy --force`.

<a name="per-project-installation"></a>
### Per Project Installation - Instalacja do poszczegulnego Projektu (Spersonalizowane)

Zamiast instalować Homestead na całym świecie i udostępniać to samo pole Homestead we wszystkich swoich projektach, możesz zamiast tego skonfigurować instancję Homestead dla każdego zarządzanego projektu. Instalacja Homestead na projekt może być korzystna, jeśli chcesz przesłać `Vagrantfile` z projektem, umożliwiając innym osobom pracującym nad projektem po prostu `vagrant up`.

Aby zainstalować Homestead bezpośrednio w projekcie, wymagane jest użycie Composer:

    composer require laravel/homestead --dev

Po zainstalowaniu Homestead użyj polecenia `make`, aby wygenerować plik `Vagrantfile` i `Homestead.yaml` w katalogu głównym projektu. Polecenie `make` automatycznie skonfiguruje dyrektywy `sites` i `folders` w pliku` Homestead.yaml`.

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

Następnie uruchom polecenie `vagrant up` w twoim terminalu i uzyskaj dostęp do twojego projektu na `http://homestead.test` w twojej przeglądarce. Pamiętaj, że będziesz nadal musiał dodać wpis `/etc/hosts` dla `homestead.test` lub wybranej domeny.

<a name="installing-mariadb"></a>
### Installing MariaDB - Instalacja MariaDB

Jeśli wolisz używać MariaDB zamiast MySQL, możesz dodać opcję `mariadb` do pliku `Homestead.yaml`. Ta opcja usunie MySQL i zainstaluje MariaDB. MariaDB służy jako zamiennik dla MySQL, więc nadal powinieneś używać sterownika bazy danych `mysql` w konfiguracji bazy danych swojej aplikacji:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="installing-elasticsearch"></a>
### Installing Elasticsearch - Instalacja Elasticsearch

Aby zainstalować Elasticsearch, dodaj opcję `elasticsearch` do pliku `Homestead.yaml` i określ obsługiwaną wersję. Domyślna instalacja utworzy klaster o nazwie 'homestead'. Nigdy nie powinieneś udostępniać Elasticsearch więcej niż połowę pamięci systemu operacyjnego, więc upewnij się, że Twoja maszyna Homestead ma co najmniej dwukrotność alokacji Elasticsearch:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 4096
    cpus: 4
    provider: virtualbox
    elasticsearch: 6

> {tip} Zapoznaj się z dokumentacją [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current), aby dowiedzieć się, jak dostosować konfigurację.

<a name="aliases"></a>
### Aliases - Aliasy

Możesz dodać aliasy Bash do swojego komputera Homestead, modyfikując plik `aliases` w swoim katalogu Homestead:

    alias c='clear'
    alias ..='cd ..'

Po zaktualizowaniu pliku `aliases` należy ponownie udostępnić maszynę Homestead za pomocą komendy `vagrant reload --provision`. Zapewni to, że twoje nowe aliasy będą dostępne na komputerze.

<a name="daily-usage"></a>
## Daily Usage - Codzienne użytkowanie

<a name="accessing-homestead-globally"></a>
### Accessing Homestead Globally - Dostęp do Homestead globalnie

Czasami możesz chcieć `vagrant up` swoją maszynę Homestead z dowolnego miejsca w systemie plików. Możesz to zrobić w systemach Mac / Linux, dodając funkcję Bash do swojego profilu Bash. W systemie Windows możesz to zrobić, dodając plik "wsadowy" do swojej `PATH`. Skrypty te pozwalają na uruchamianie dowolnego polecenia Vagrant z dowolnego miejsca w systemie i automatycznie wskazują to polecenie na instalację Homestead:

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

Upewnij się, że zmieniłeś ścieżkę `~/Homestead` w funkcji na lokalizację twojej aktualnej instalacji Homestead. Po zainstalowaniu tej funkcji możesz uruchamiać komendy takie jak `homestead up` lub `homestead ssh` z dowolnego miejsca w systemie.


#### Windows

Utwórz plik wsadowy `homestead.bat` w dowolnym miejscu na komputerze z następującą zawartością:

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

Upewnij się, że zmieniłeś ścieżkę `C:\Homestead` w skrypcie do rzeczywistej lokalizacji instalacji Homestead. Po utworzeniu pliku dodaj położenie pliku do `PATH`. Możesz wtedy uruchamiać komendy takie jak `homestead up` lub `homestead ssh` z dowolnego miejsca w systemie.

<a name="connecting-via-ssh"></a>
### Connecting Via SSH - Połaczenie przez SSH

Możesz zalogować się na maszynie wirtualnej, wydając komendę `vagrant ssh` w terminal-u z katalogu Homestead.

Ale ponieważ prawdopodobnie będziesz musiał często korzystać z SSH w swoim komputerze Homestead, rozważ dodanie "funkcji" opisanej powyżej do swojego hosta, aby szybko wprowadzić SSH do skrzynki Homestead.

<a name="connecting-to-databases"></a>
### Connecting To Databases - Łączenie z bazami danych

Baza danych `homestead` jest skonfigurowana zarówno dla MySQL jak i PostgreSQL po wyjęciu z pudełka. Dla jeszcze większej wygody, plik `.env` Laravel'a konfiguruje strukturę do korzystania z tej bazy danych po wyjęciu z pudełka.

Aby połączyć się z bazą danych MySQL lub PostgreSQL z poziomu klienta bazy danych twojego hosta, powinieneś połączyć się z `127.0.0.1` i port `33060` (MySQL) lub `54320` (PostgreSQL). Nazwa użytkownika i hasło dla obu baz danych to `homestead` /`secret`.

> {note} Z tych niestandardowych portów należy korzystać tylko podczas łączenia się z bazami danych z komputera hosta. Użyjesz domyślnych portów 3306 i 5432 w pliku konfiguracyjnym bazy danych Laravel, ponieważ Laravel uruchamia _wirtualną maszynę.

<a name="adding-additional-sites"></a>
### Adding Additional Sites - Dodawanie dodatkowych stron

Gdy Twoje środowisko Homestead zostanie skonfigurowane i uruchomione, możesz chcieć dodać dodatkowe witryny Nginx do swoich aplikacji Laravel. Możesz uruchomić dowolną liczbę instalacji Laravel w jednym środowisku Homestead. Aby dodać dodatkową witrynę, po prostu dodaj witrynę do pliku `Homestead.yaml`:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
        - map: another.test
          to: /home/vagrant/code/another/public

Jeśli Vagrant nie będzie automatycznie zarządzał Twoim plikiem "hosts", może być konieczne dodanie nowej witryny do tego pliku:

    192.168.10.10  homestead.test
    192.168.10.10  another.test

Po dodaniu witryny uruchom polecenie `vagrant reload --provision` z katalogu Homestead.

<a name="site-types"></a>
#### Site Types - Rodzaje witryn

Homestead obsługuje kilka typów witryn, które pozwalają łatwo uruchamiać projekty, które nie są oparte na Laravel. Na przykład możemy łatwo dodać aplikację Symfony do Homestead za pomocą typu witryny `symfony2`:

    sites:
        - map: symfony2.test
          to: /home/vagrant/code/Symfony/web
          type: "symfony2"

Dostępne typy witryn to: `apache`, `laravel` (domyślnie), `proxy`, `silverstripe`, `statamic`, `symfony2`, and `symfony4`.

<a name="site-parameters"></a>
#### Site Parameters - Parametry witryny

Możesz dodać dodatkowe wartości Nginx `fastcgi_param` do swojej witryny poprzez dyrektywę strony `params`. Na przykład dodamy parametr `FOO` o wartości `BAR`:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          params:
              - key: FOO
                value: BAR

<a name="environment-variables"></a>
### Environment Variables - Zmienne środowiska

Możesz ustawić globalne zmienne środowiskowe, dodając je do pliku `Homestead.yaml`:

    variables:
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar

Po aktualizacji `Homestead.yaml`, upewnij się, że ponownie przeladujesz maszynę, uruchamiając `vagrant reload --provision`. To zaktualizuje konfigurację PHP-FPM dla wszystkich zainstalowanych wersji PHP, a także zaktualizuje środowisko dla użytkownika `vagrant`.

<a name="configuring-cron-schedules"></a>
### Configuring Cron Schedules - Konfigurowanie harmonogramów Cron

Laravel zapewnia wygodny sposób na [planowanie zadań Cron](/docs/{{version}}/scheduling) poprzez planowanie pojedynczej komendy `schedule:run` Artisan-a uruchamianej co minutę. Polecenie `schedule:run` zbada harmonogram zadań zdefiniowany w klasie `App\Console\Kernel`, aby określić, które zadania powinny zostać uruchomione.

Jeśli chciałbyś uruchomić polecenie `schedule:run` dla witryny Homestead, możesz ustawić opcję `schedule` na `true` podczas definiowania strony:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          schedule: true

Zadanie Cron witryny zostanie zdefiniowane w folderze `/etc/cron.d` na maszynie wirtualnej.

<a name="configuring-mailhog"></a>
### Configuring Mailhog - Konfigurowanie Mailhog

Mailhog pozwala na łatwe wychwycenie poczty wychodzącej i jej sprawdzenie bez faktycznego wysłania wiadomości do jej adresatów. Aby rozpocząć, zaktualizuj plik `.env`, aby użyć następujących ustawień poczty:

    MAIL_DRIVER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

<a name="ports"></a>
### Ports - Porty

Domyślnie następujące porty są przekazywane do twojego środowiska Homestead:

- **SSH:** 2222 &rarr; Przekierowanie do 22
- **ngrok UI:** 4040 &rarr; Przekierowanie do 4040
- **HTTP:** 8000 &rarr; Przekierowanie do 80
- **HTTPS:** 44300 &rarr; Przekierowanie do 443
- **MySQL:** 33060 &rarr; Przekierowanie do 3306
- **PostgreSQL:** 54320 &rarr; Przekierowanie do 5432
- **Mailhog:** 8025 &rarr; Przekierowanie do 8025

#### Forwarding Additional Ports - Przekazywanie dodatkowych portów

Jeśli chcesz, możesz przekazać dodatkowe porty do skrzynki Vagrant, a także określić ich protokół:

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>
### Sharing Your Environment - Dzielenie się środowiskiem

Czasami możesz chcieć podzielić się tym, nad czym aktualnie pracujesz ze współpracownikami lub klientem. Vagrant ma wbudowany sposób na wsparcie tego poprzez `vagrant share`; nie będzie to jednak działać, jeśli masz wiele stron skonfigurowanych w pliku `Homestead.yaml`.

Aby rozwiązać ten problem, Homestead zawiera własne polecenie `share`. Aby rozpocząć, Zaloguj się poprzez SSH do maszyny Homestead przez `vagrant ssh` i uruchom `share homestead.test`. Spowoduje to udostępnienie strony `homestead.test` z pliku konfiguracyjnego `Homestead.yaml`. Oczywiście możesz zamienić dowolną z twoich skonfigurowanych witryn na `homestead.test`:

    share homestead.test

Po uruchomieniu polecenia pojawi się ekran Ngrok zawierający dziennik aktywności i publicznie dostępne adresy URL dla udostępnianej witryny. Jeśli chcesz określić region niestandardowy, poddomeny lub inną opcję środowiska wykonawczego Ngrok, możesz dodać je do polecenia `share`:

    share homestead.test -region=eu -subdomain=laravel

> {note} Pamiętaj, że Vagrant jest z natury niebezpieczny i narażasz swoją wirtualną maszynę na działanie Internetu, kiedy uruchomisz komendę `share`.

<a name="multiple-php-versions"></a>
### Multiple PHP Versions - Wiele wersji PHP

> {note} Ta funkcja jest kompatybilna tylko z Nginx

Homestead 6 wprowadził obsługę wielu wersji PHP na tej samej maszynie wirtualnej. Możesz określić, której wersji PHP użyć dla danej witryny w pliku `Homestead.yaml`. Dostępne wersje PHP to: "5.6", "7.0", "7.1" i "7.2" (domyślnie):

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          php: "5.6"

Ponadto możesz używać dowolnej obsługiwanej wersji PHP za pośrednictwem interfejsu CLI:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list

<a name="network-interfaces"></a>
## Network Interfaces - Interfejsy sieciowe

Właściwość `networks` pliku ` Homestead.yaml` konfiguruje interfejsy sieciowe dla twojego środowiska Homestead. Możesz skonfigurować tyle interfejsów, ile potrzeba:

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

Aby włączyć interfejs [bridged](https://www.vagrantup.com/docs/networking/public_network.html), skonfiguruj ustawienie `bridge` i zmień typ sieci na `public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

Aby włączyć [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), po prostu usuń opcję `ip` ze swojej konfiguracji:

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## Updating Homestead - Aktualizacja Homestead

Możesz zaktualizować Homestead w dwóch prostych krokach. Po pierwsze, należy zaktualizować pole Vagrant za pomocą polecenia `vagrant box update`:

    vagrant box update

Następnie musisz zaktualizować kod źródłowy Homestead. Jeśli sklonowałeś repozytorium, możesz po prostu `git pull origin master` w miejscu, w którym pierwotnie sklonowałeś repozytorium.

Jeśli zainstalowałeś Homestead poprzez plik `composer.json` twojego projektu, powinieneś upewnić się, że twój plik `composer.json` zawiera `"laravel/homestead": "^7"` i zaktualizować twoje zależności:

    composer update

<a name="provider-specific-settings"></a>
## Provider Specific Settings - Ustawienia określone przez dostawcę

<a name="provider-specific-virtualbox"></a>
### VirtualBox

#### `natdnshostresolver`

Domyślnie Homestead konfiguruje ustawienie `natdnshostresolver` na `on`. Dzięki temu Homestead może korzystać z ustawień DNS twojego systemu operacyjnego. Jeśli chcesz zastąpić to zachowanie, dodaj następujące linie do pliku `Homestead.yaml`:

    provider: virtualbox
    natdnshostresolver: off

#### Symbolic Links On Windows - Linki symboliczne w systemie Windows

Jeśli linki symboliczne nie działają poprawnie na komputerze z systemem Windows, może być konieczne dodanie następującego bloku do pliku `Vagrantfile`:
    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end
