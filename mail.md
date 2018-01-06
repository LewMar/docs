# Mail

- [Introduction - Wprowadzenie](#introduction)
    - [Driver Prerequisites - Wymagania wstępne sterownika](#driver-prerequisites)
- [Generating Mailables - Generowanie usług pocztowych](#generating-mailables)
- [Writing Mailables - Pisanie Mailables](#writing-mailables)
    - [Configuring The Sender - Konfigurowanie nadawcy](#configuring-the-sender)
    - [Configuring The View - Konfigurowanie widoku](#configuring-the-view)
    - [View Data  Wyświetl dane](#view-data)
    - [Attachments - Załączniki](#attachments)
    - [Inline Attachments - Dołączone załączniki](#inline-attachments)
    - [Customizing The SwiftMailer Message - Dostosowywanie wiadomości SwiftMailer](#customizing-the-swiftmailer-message)
- [Markdown Mailables - Wiadomości Markdown](#markdown-mailables)
    - [Generating Markdown Mailables - Generowanie wiadomości Markdown](#generating-markdown-mailables)
    - [Writing Markdown Messages - Pisanie wiadomości Markdown](#writing-markdown-messages)
    - [Customizing The Components - Dostosowywanie komponentów](#customizing-the-components)
- [Previewing Mailables In The Browser - Wyświetlanie podglądu wiadomości w przeglądarce](#previewing-mailables-in-the-browser)
- [Sending Mail - Wysyłanie maila](#sending-mail)
    - [Queueing Mail - Usługa kolejkowania poczty](#queueing-mail)
- [Mail & Local Development - Mail i rozwój lokalny](#mail-and-local-development)
- [Events - Zdarzenia](#events)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel zapewnia czysty, prosty interfejs API w popularnej bibliotece [SwiftMailer](https://swiftmailer.symfony.com/) ze sterownikami dla SMTP, Mailgun, SparkPost, Amazon SES, PHP `mail` i` sendmail`, pozwalając aby szybko rozpocząć wysyłanie poczty za pośrednictwem usługi lokalnej lub opartej na chmurze.

<a name="driver-prerequisites"></a>
### Driver Prerequisites - Wymagania wstępne sterownika

Sterowniki oparte na API, takie jak Mailgun i SparkPost, są często prostsze i szybsze niż serwery SMTP. Jeśli to możliwe, powinieneś użyć jednego z tych sterowników. Wszystkie sterowniki interfejsu API wymagają biblioteki HTTP Guzzle, którą można zainstalować za pomocą menedżera pakietów Composer:

    composer require guzzlehttp/guzzle

#### Mailgun Driver - Sterownik Mailgun

Aby użyć sterownika Mailgun, najpierw zainstaluj Guzzle, a następnie ustaw opcję `driver` w pliku konfiguracyjnym `config/mail.php` na `mailgun`. Następnie sprawdź, czy plik konfiguracyjny `config/services.php` zawiera następujące opcje:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost Driver - Sterownik SparkPost

Aby użyć sterownika SparkPost, najpierw zainstaluj Guzzle, a następnie ustaw opcję `driver` w pliku konfiguracyjnym `config/mail.php` na `sparkpost`. Następnie sprawdź, czy plik konfiguracyjny `config/services.php` zawiera następujące opcje:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SES Driver - Sterownik SES

Aby użyć sterownika Amazon SES, należy najpierw zainstalować pakiet SDK Amazon AWS dla PHP. Możesz zainstalować tę bibliotekę, dodając następujący wiersz do sekcji `require` pliku `composer.json` i uruchamiając komendę `composer update`:

    "aws/aws-sdk-php": "~3.0"

Następnie ustaw opcję `driver` w pliku konfiguracyjnym `config/mail.php` na `ses` i sprawdź, czy plik konfiguracyjny `config/services.php` zawiera następujące opcje:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## Generating Mailables - Generowanie usług pocztowych

W Laravel każdy typ wiadomości e-mail wysłanej przez twoją aplikację jest reprezentowany jako klasa "mailable". Te klasy są przechowywane w katalogu `app/Mail`. Nie martw się, jeśli nie widzisz tego katalogu w aplikacji, ponieważ zostanie on wygenerowany podczas tworzenia pierwszej klasy do korespondowania za pomocą komendy `make:mail`:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Writing Mailables - Pisanie Mailables

Cała konfiguracja klasy mailowej odbywa się w metodzie `build`. W ramach tej metody możesz wywoływać różne metody, takie jak `from`, `subject`, `view`, i `attach`, aby skonfigurować prezentację i dostarczanie wiadomości e-mail.

<a name="configuring-the-sender"></a>
### Configuring The Sender - Konfigurowanie nadawcy

#### Using The `from` Method

Najpierw sprawdźmy konfigurowanie nadawcy wiadomości e-mail. Lub, innymi słowy,  `from`(od) kogo będzie email. Istnieją dwa sposoby konfiguracji nadawcy. Najpierw możesz użyć metody `from` w swojej klasie mailowej `build`:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### Using A Global `from` Address - Korzystanie z globalnego adresu "od"

Jeśli jednak aplikacja używa tego samego adresu "od" do wszystkich swoich wiadomości e-mail, może być uciążliwe wywoływanie metody `from` w każdej generowanej klasie wiadomości e-mail. Zamiast tego możesz podać globalny adres "od" w pliku konfiguracyjnym `config/mail.php`. Ten adres będzie użyty, jeśli w klasie mailable nie zostanie podany inny adres "od":

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### Configuring The View - Konfigurowanie widoku

W ramach klasy `build` klasy mailowej można użyć metody `view` do określenia, który szablon powinien być używany podczas renderowania treści wiadomości e-mail. Ponieważ każdy e-mail zazwyczaj używa szablonu [Blade template](/docs/{{version}}/blade) do renderowania jego zawartości, masz pełną moc i wygodę mechanizmu szablonowego Blade podczas budowania HTML twojego emaila:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} Możesz utworzyć katalog `resources/views/emails` zawierający wszystkie szablony e-maili; jednak możesz umieścić je w dowolnym miejscu w katalogu `resources/views`.

#### Plain Text Emails - Zwykłe wiadomości tekstowe

Jeśli chcesz zdefiniować wersję e-mailową w postaci zwykłego tekstu, możesz użyć metody `text`. Podobnie jak w przypadku metody `view`, metoda` text` akceptuje nazwę szablonu, która będzie używana do renderowania zawartości wiadomości e-mail. Możesz zdefiniować zarówno HTML, jak i tekstową wersję swojej wiadomości:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### View Data - Wyświetl dane

#### Via Public Properties - Przez właściwości publiczne

Zazwyczaj użytkownik chce przekazać niektóre dane do widoku, które można wykorzystać podczas renderowania kodu HTML wiadomości e-mail. Istnieją dwa sposoby udostępniania danych w widoku. Po pierwsze, dowolna publiczna właściwość zdefiniowana na twojej klasie mailowej zostanie automatycznie udostępniona do widoku. Na przykład możesz przekazać dane do swojego konstruktora klasy mailowej i ustawić te dane na właściwości publiczne zdefiniowane w klasie:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

Po ustawieniu danych na właściwość publiczną, będzie ona automatycznie dostępna w widoku, więc możesz uzyskać do niej dostęp tak, jakbyś miał dostęp do innych danych w szablonach Blade:

    <div>
        Price: {{ $order->price }}
    </div>

#### Via The `with` Method: - Metoda "z":

Jeśli chcesz dostosować format danych e-maili przed ich przesłaniem do szablonu, możesz ręcznie przekazać swoje dane do widoku za pomocą metody `with`. Zazwyczaj nadal będziesz przekazywał dane przez konstruktor klasy mailowej; należy jednak ustawić te dane na właściwości `protected` lub `private`, aby dane nie były automatycznie udostępniane szablonowi. Następnie, wywołując metodę `with`, przekaż tablicę danych, które chcesz udostępnić szablonowi:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

Gdy dane zostaną przekazane do metody `with`, będzie ona automatycznie dostępna w twoim widoku, więc możesz uzyskać do niej dostęp tak, jakbyś miał dostęp do innych danych w szablonach Blade:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### Attachments - Załączniki

Aby dodać załączniki do wiadomości e-mail, użyj metody `attach` w ramach metody `build` klasy mailable. Metoda `attach` akceptuje pełną ścieżkę do pliku jako swój pierwszy argument:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

Podczas dołączania plików do wiadomości można również podać nazwę wyświetlaną i / lub typ MIME, przekazując `array` jako drugi argument metody `attach`:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### Raw Data Attachments - Załączniki danych nieprzetworzonych

Metodę `attachData` można użyć do dołączenia nieprzetworzonego ciągu bajtów jako załącznika. Na przykład możesz użyć tej metody, jeśli wygenerowałeś plik PDF w pamięci i chcesz dołączyć go do e-maila bez zapisywania go na dysku. Metoda `attachData` akceptuje bajty danych surowych jako swój pierwszy argument, nazwę pliku jako drugi argument oraz tablicę opcji jako trzeci argument:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### Inline Attachments - Dołączone załączniki

Umieszczanie wbudowanych obrazów w wiadomościach e-mail jest zwykle kłopotliwe; jednak Laravel zapewnia wygodny sposób dołączania obrazów do wiadomości e-mail i pobierania odpowiedniego identyfikatora CID. Aby osadzić obraz wbudowany, użyj metody `embed` w zmiennej `$message` w szablonie wiadomości e-mail. Laravel automatycznie udostępnia zmienną `$message` dla wszystkich szablonów wiadomości e-mail, więc nie musisz się martwić ręcznym wprowadzaniem jej:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

> {note} Zmienna `$message` nie jest dostępna w komunikatach markdown.

#### Embedding Raw Data Attachments - Osadzanie załączników do surowych danych

Jeśli masz już surowy ciąg danych, który chcesz osadzić w szablonie wiadomości e-mail, możesz użyć metody `embedData` w zmiennej `$message`:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### Customizing The SwiftMailer Message - Dostosowywanie wiadomości SwiftMailer

Metoda `withSwiftMessage` klasy `Mailable` pozwala zarejestrować wywołanie zwrotne, które zostanie wywołane przez surową instancję komunikatu SwiftMailer przed wysłaniem wiadomości. Daje to możliwość dostosowania wiadomości przed jej dostarczeniem:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            $this->view('emails.orders.shipped');

            $this->withSwiftMessage(function ($message) {
                $message->getHeaders()
                        ->addTextHeader('Custom-Header', 'HeaderValue');
            });
        }

<a name="markdown-mailables"></a>
## Markdown Mailables - Wiadomości Markdown

Wiadomości e-mail w formacie Markdown umożliwiają korzystanie z gotowych szablonów i składników powiadomień pocztowych w skrzynkach pocztowych. Ponieważ wiadomości są napisane w Markdown, Laravel jest w stanie renderować piękne, responsywne szablony HTML dla wiadomości, jednocześnie automatycznie generując tekstowy odpowiednik.

<a name="generating-markdown-mailables"></a>
### Generating Markdown Mailables - Generowanie wiadomości Markdown

Aby wygenerować pocztę z odpowiednim szablonem Markdown, możesz użyć opcji `--markdown` polecenia `make:mail` Artisan:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

Następnie, konfigurując plik mailable w ramach jego metody `build`, wywołaj metodę `markdown` zamiast metody `view`. Metoda `markdown` akceptuje nazwę szablonu Markdown i opcjonalny zestaw danych do udostępnienia szablonowi:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### Writing Markdown Messages - Pisanie wiadomości Markdown

Skrzynki pocztowe programu Markdown wykorzystują kombinację komponentów Blade i Markdown, które umożliwiają łatwe tworzenie wiadomości e-mail przy użyciu wstępnie przygotowanych komponentów Laravel:

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

> {tip} Nie używaj nadmiernego wcięcia podczas pisania e-maili Markdown. Parsery Markdown będą renderować wcięte treści jako bloki kodu.

#### Button Component - Składnik przycisku

Komponent przycisku renderuje wyśrodkowane łącze przycisku. Komponent przyjmuje dwa argumenty: `url` i opcjonalny `color`. Obsługiwane kolory to `blue`, `green`, i `red`. Możesz dodać tyle elementów przycisku do wiadomości, ile chcesz:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Order
    @endcomponent

#### Panel Component - Składnik panelu

Składnik panelu renderuje dany blok tekstu w panelu, który ma nieco inny kolor tła niż reszta wiadomości. Pozwala to na zwrócenie uwagi na dany blok tekstu:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Table Component - Składnik tabeli

Składnik tabeli umożliwia przekształcenie tabeli Markdown w tabelę HTML. Komponent akceptuje tabelę Markdown jako jej zawartość. Wyrównanie kolumny tabeli jest obsługiwane przy użyciu domyślnej składni wyrównania tabeli Markdown:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### Customizing The Components - Dostosowywanie komponentów

Możesz wyeksportować wszystkie komponenty poczty Markdown do swojej aplikacji w celu dostosowania. Aby wyeksportować komponenty, użyj komendy `vendor:publish` Artisan, aby opublikować znacznik zasobu `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail

To polecenie opublikuje komponenty poczty Markdown w katalogu `resources/views/vendor/mail`. Katalog `mail` będzie zawierał katalog `html` i `markdown`, każdy zawierający odpowiednie reprezentacje każdego dostępnego komponentu. Możesz dowolnie dostosowywać te komponenty.

#### Customizing The CSS - Dostosowywanie CSS

Po wyeksportowaniu komponentów katalog `resources/views/vendor/mail/html/themes` będzie zawierał plik `default.css`. Możesz dostosować CSS w tym pliku, a twoje style zostaną automatycznie umieszczone w reprezentacjach HTML Twoich wiadomości pocztowych Markdown.

> {tip} Jeśli chcesz zbudować całkowicie nowy motyw komponentów Markdown, po prostu napisz nowy plik CSS w katalogu `html/themes` i zmień opcję `theme` w pliku konfiguracyjnym `mail`.

<a name="previewing-mailables-in-the-browser"></a>
## Previewing Mailables In The Browser - Wyświetlanie podglądu wiadomości w przeglądarce

Podczas projektowania szablonu wiadomości e-mail wygodnie jest szybko wyświetlić podgląd renderowanego pliku pocztowego w przeglądarce jak typowy szablon Blade. Z tego powodu Laravel pozwala na zwrot wszelkiej korespondencji bezpośrednio z Closure trasy lub kontrolera. Po zwróceniu pliku pocztowego zostanie on wyrenderowany i wyświetlony w przeglądarce, co umożliwi szybki podgląd jego projektu bez konieczności wysyłania go na rzeczywisty adres e-mail:

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="sending-mail"></a>
## Sending Mail - Wysyłanie maila

Aby wysłać wiadomość, użyj metody `to` na elemencie `Mail` [fasady](/docs/{{version}}/facades). Metoda `to` akceptuje adres e-mail, instancję użytkownika lub kolekcję użytkowników. W przypadku przekazania obiektu lub zbioru obiektów, program Mailer automatycznie użyje właściwości `email` i `name` podczas ustawiania odbiorców wiadomości e-mail, więc upewnij się, że te atrybuty są dostępne dla twoich obiektów. Po określeniu adresatów możesz przekazać instancję klasy mailowej do metody `send`:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Ship order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

Oczywiście nie ogranicza się tylko do określania odbiorców "to" podczas wysyłania wiadomości. Możesz ustawić odbiorcę "to", "cc" i "bcc" w pojedynczym, połączonym wywołaniu metody:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### Queueing Mail - Usługa kolejkowania poczty

#### Queueing A Mail Message - Kolejkowanie wiadomości pocztowej

Ponieważ wysyłanie wiadomości e-mail może drastycznie wydłużyć czas reakcji aplikacji, wielu programistów decyduje się na kolejkowanie wiadomości e-mail w celu wysłania w tle. Laravel ułatwia to za pomocą wbudowanego [API ujednoliconej kolejki](/docs/{{version}}/queues). Aby umieścić kolejną wiadomość pocztową, użyj metody `queue` na elewacji `Mail` po określeniu odbiorców wiadomości:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Ta metoda automatycznie zajmie się wypychaniem zadania do kolejki, aby wiadomość była wysyłana w tle. Oczywiście przed użyciem tej funkcji trzeba będzie [skonfigurować swoje kolejki](/docs/{{version}}/queues).

#### Delayed Message Queueing - Opóźniona kolejkowanie wiadomości

Jeśli chcesz opóźnić dostarczenie kolejki wiadomości e-mail, możesz użyć metody `later`. Jako pierwszy argument, metoda `later` akceptuje instancję `DateTime` wskazującą, kiedy wiadomość ma zostać wysłana:

    $when = now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### Pushing To Specific Queues - Pchanie do określonych kolejek

Ponieważ wszystkie klasy mailable generowane za pomocą polecenia `make:mail` używają funkcji `Illuminate\Bus\Queueable`, możesz wywoływać metody `onQueue` i` onConnection` na dowolnej instancji klasy poczty, co pozwala ci określić połączenie i nazwę kolejki dla wiadomości:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### Queueing By Default - Kolejkowanie domyślnie

Jeśli posiadasz klasy do korespondencji, które chcesz zawsze umieszczać w kolejce, możesz wdrożyć na nich klasę `ShouldQueue`. Teraz, nawet jeśli wywołasz metodę "send" podczas wysyłania wiadomości, poczta pozostanie w kolejce od momentu implementacji kontraktu:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## Mail & Local Development - Mail i rozwój lokalny

Rozwijając aplikację wysyłającą e-maile, prawdopodobnie nie chcesz wysyłać e-maili na adresy e-mail. Laravel oferuje kilka sposobów "wyłączenia" faktycznego wysyłania e-maili podczas lokalnego rozwoju.

#### Log Driver - Sterownik dziennika

Zamiast wysyłać wiadomości e-mail, sterownik poczty `log` będzie zapisywać wszystkie wiadomości e-mail do plików dziennika w celu sprawdzenia. Aby uzyskać więcej informacji na temat konfigurowania aplikacji na środowisko, zapoznaj się z [dokumentacją konfiguracyjną](/docs/{{version}}/configuration#environment-configuration).

#### Universal To - Uniwersalny odbiorca

Innym rozwiązaniem dostarczonym przez Laravel jest ustawienie uniwersalnego odbiorcy wszystkich wiadomości e-mail wysyłanych przez framework. W ten sposób wszystkie wiadomości e-mail generowane przez aplikację będą wysyłane na konkretny adres zamiast adresu faktycznie określonego podczas wysyłania wiadomości. Można to zrobić za pomocą opcji `to` w pliku konfiguracyjnym `config/mail.php`:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

Na koniec możesz użyć usługi takiej jak [Mailtrap](https://mailtrap.io) i sterownika `smtp` do wysyłania wiadomości e-mail do "fałszywej" skrzynki pocztowej, w której możesz wyświetlać je w prawdziwym kliencie poczty e-mail. Takie podejście ma tę zaletę, że pozwala na rzeczywistą kontrolę końcowych wiadomości e-mail w przeglądarce wiadomości Mailtrap.

<a name="events"></a>
## Events - Zdarzenia

Laravel uruchamia dwa zdarzenia podczas wysyłania wiadomości e-mail. Zdarzenie `MessageSending` jest uruchamiane przed wysłaniem komunikatu, natomiast zdarzenie` MessageSent` jest wywoływane po wysłaniu wiadomości. Pamiętaj, że te zdarzenia są uruchamiane, gdy poczta jest *wysyłana*, a nie kiedy jest w kolejce. Możesz zarejestrować detektor zdarzeń dla tego wydarzenia w swoim `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
