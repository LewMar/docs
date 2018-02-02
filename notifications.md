# Notifications

- [Introduction - Wprowadzenie](#introduction)
- [Creating Notifications - Tworzenie powiadomień](#creating-notifications)
- [Sending Notifications - Wysyłanie powiadomień](#sending-notifications)
    - [Using The Notifiable Trait - Używanie cechy podlegającej notyfikacji](#using-the-notifiable-trait)
    - [Using The Notification Facade - Korzystanie z fasady powiadamiania](#using-the-notification-facade)
    - [Specifying Delivery Channels - Określanie kanałów dostarczania](#specifying-delivery-channels)
    - [Queueing Notifications - Powiadomienia w kolejce](#queueing-notifications)
    - [On-Demand Notifications - Powiadomienia na żądanie](#on-demand-notifications)
- [Mail Notifications - Powiadomienia pocztą](#mail-notifications)
    - [Formatting Mail Messages - Formatowanie wiadomości pocztowych](#formatting-mail-messages)
    - [Customizing The Recipient - Dostosowywanie odbiorcy](#customizing-the-recipient)
    - [Customizing The Subject - Dostosowywanie tematu](#customizing-the-subject)
    - [Customizing The Templates - Dostosowywanie szablonów](#customizing-the-templates)
- [Markdown Mail Notifications - Powiadomienia pocztowe Markdown](#markdown-mail-notifications)
    - [Generating The Message - Generowanie wiadomości](#generating-the-message)
    - [Writing The Message - Pisanie wiadomości](#writing-the-message)
    - [Customizing The Components - Dostosowywanie komponentów](#customizing-the-components)
- [Database Notifications - Powiadomienia bazy danych](#database-notifications)
    - [Prerequisites - Wymagania wstępne](#database-prerequisites)
    - [Formatting Database Notifications - Formatowanie powiadomień bazy danych](#formatting-database-notifications)
    - [Accessing The Notifications - Dostęp do powiadomień](#accessing-the-notifications)
    - [Marking Notifications As Read - Oznaczanie powiadomień jako przeczytane](#marking-notifications-as-read)
- [Broadcast Notifications - Powiadomienia rozgłoszeniowe](#broadcast-notifications)
    - [Prerequisites - Wymagania wstępne](#broadcast-prerequisites)
    - [Formatting Broadcast Notifications - Formatowanie powiadomień o transmisji](#formatting-broadcast-notifications)
    - [Listening For Notifications - Nasłuchiwanie powiadomień](#listening-for-notifications)
- [SMS Notifications - Powiadomienia SMS](#sms-notifications)
    - [Prerequisites - Wymagania wstępne](#sms-prerequisites)
    - [Formatting SMS Notifications - Formatowanie powiadomień SMS](#formatting-sms-notifications)
    - [Customizing The "From" Number - Dostosowywanie numeru "Od"](#customizing-the-from-number)
    - [Routing SMS Notifications - Routing powiadomień SMS](#routing-sms-notifications)
- [Slack Notifications - Powiadomienia Slack](#slack-notifications)
    - [Prerequisites - Wymagania wstępne](#slack-prerequisites)
    - [Formatting Slack Notifications - Formatowanie powiadomień Slack](#formatting-slack-notifications)
    - [Slack Attachments - Załaczniki Slack](#slack-attachments)
    - [Routing Slack Notifications - Routing powiadomień Slack](#routing-slack-notifications)
- [Notification Events - Zdarzenia powiadomień](#notification-events)
- [Custom Channels - Kanały niestandardowe](#custom-channels)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Oprócz wsparcia dla [wysyłania e-maili](/docs/{{version}}/mail), Laravel zapewnia obsługę wysyłania powiadomień przez różne kanały dostarczania, w tym pocztę, SMS-y (przez [Nexmo](https://www.nexmo.com/)) i [Slack](https://slack.com). Powiadomienia mogą być również przechowywane w bazie danych, dzięki czemu mogą być wyświetlane w interfejsie internetowym.

Zazwyczaj powiadomienia powinny być krótkie, informacyjne, które powiadamiają użytkowników o czymś, co wystąpiło w Twojej aplikacji. Na przykład, jeśli piszesz aplikację rozliczeniową, możesz wysyłać powiadomienie "Faktura zapłacona" do użytkowników za pośrednictwem kanałów e-mail i SMS.

<a name="creating-notifications"></a>
## Creating Notifications - Tworzenie powiadomień

W Laravel każde powiadomienie jest reprezentowane przez pojedynczą klasę (zazwyczaj przechowywaną w katalogu `app/Notifications`). Nie martw się, jeśli nie widzisz tego katalogu w swojej aplikacji, zostanie on utworzony dla Ciebie po uruchomieniu polecenia `make:notification` Artisan:

    php artisan make:notification InvoicePaid

To polecenie umieści nową klasę powiadomień w katalogu `app/Notifications`. Każda klasa powiadomień zawiera metodę `via` i zmienną liczbę metod budowania wiadomości (takich jak `toMail` lub `toDatabase`), które przekształcają powiadomienie w wiadomość zoptymalizowaną dla tego konkretnego kanału.

<a name="sending-notifications"></a>
## Sending Notifications - Wysyłanie powiadomień

<a name="using-the-notifiable-trait"></a>
### Using The Notifiable Trait - Używanie cechy podlegającej notyfikacji

Powiadomienia mogą być wysyłane na dwa sposoby: za pomocą metody `notify` w funkcji `Notifiable` lub za pomocą opcji `Notification` [fasada](/docs/{{version}}/facades). Najpierw sprawdźmy, wykorzystując to trait:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Ta trait jest wykorzystywana przez domyślny model `App\User` i zawiera jedną metodę, która może być używana do wysyłania powiadomień: `notify`. Metoda `notify` spodziewa się otrzymać instancję powiadomienia:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} Pamiętaj, możesz użyć cechy `Illuminate\Notifications\Notifiable` na każdym ze swoich modeli. Nie ograniczasz się tylko do włączenia go do swojego modelu `User`.

<a name="using-the-notification-facade"></a>
### Using The Notification Facade - Korzystanie z fasady powiadamiania

Alternatywnie możesz wysyłać powiadomienia przez `Notification` [fasada](/docs/{{version}}/facades). Jest to przydatne przede wszystkim wtedy, gdy trzeba wysłać powiadomienie do wielu podmiotów podlegających obowiązkowi zgłoszenia, takich jak zbiór użytkowników. Aby wysyłać powiadomienia za pomocą elewacji, należy przekazać wszystkie podmioty podlegające obowiązkowi zgłoszenia oraz instancję powiadomienia do metody `send`:

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### Specifying Delivery Channels - Określanie kanałów dostarczania

Każda klasa powiadomień ma metodę `via`, która określa, na których kanałach powiadomienie zostanie dostarczone. Po wyjęciu z pudełka powiadomienia mogą być wysyłane na kanałach `mail`, `database`, `broadcast`, `nexmo`, i `slack`.

> {tip} Jeśli chcesz korzystać z innych kanałów dostarczania, takich jak Telegram lub Pusher, odwiedź stronę [Laravel Notification Channels](http://laravel-notification-channels.com) prowadzoną przez społeczność.

Metoda `via` odbiera instancję `$notifiable`, która będzie instancją klasy, do której wysyłane jest powiadomienie. Możesz użyć `$notifiable` w celu określenia, na których kanałach powinno być dostarczane powiadomienie:

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### Queueing Notifications - Powiadomienia w kolejce

> {note} Przed powiadomieniami w kolejce należy skonfigurować kolejkę i [uruchomić pracownika](/docs/{{version}}/queues).

Wysyłanie powiadomień może zająć trochę czasu, zwłaszcza jeśli kanał wymaga zewnętrznego wywołania interfejsu API w celu dostarczenia powiadomienia. Aby przyspieszyć czas reakcji aplikacji, pozwól, aby powiadomienie było umieszczane w kolejce, dodając do klasy traits `ShouldQueue` i` Queueable`. Interfejs i traits są już zaimportowane dla wszystkich powiadomień wygenerowanych za pomocą `make:notification`, dzięki czemu możesz natychmiast dodać je do swojej klasy powiadomień:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

Gdy do powiadomienia zostanie dodany interfejs `ShouldQueue`, możesz wysłać powiadomienie jak zwykle. Laravel wykryje interfejs `ShouldQueue` na klasie i automatycznie ustawi kolejkę dostarczenia powiadomienia:

    $user->notify(new InvoicePaid($invoice));

Jeśli chcesz opóźnić dostarczenie powiadomienia, możesz powiązać metodę `delay` ze swoim wystąpieniem z powiadomieniem:

    $when = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));

<a name="on-demand-notifications"></a>
### On-Demand Notifications - Powiadomienia na żądanie

Czasami może być konieczne wysłanie powiadomienia do osoby, która nie jest zapisana jako "użytkownik" aplikacji. Korzystając z metody `Notification::route`, możesz określić dane routingu powiadomienia ad-hoc przed wysłaniem powiadomienia:

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## Mail Notifications - Powiadomienia pocztą

<a name="formatting-mail-messages"></a>
### Formatting Mail Messages - Formatowanie wiadomości pocztowych

Jeśli powiadomienie obsługuje wysyłanie jako wiadomość e-mail, powinieneś zdefiniować metodę `toMail` w klasie powiadomień. Ta metoda otrzyma jednostkę `$notifiable` i powinna zwrócić instancję `Illuminate\Notifications\Messages\MailMessage`. Wiadomości e-mail mogą zawierać linie tekstu oraz "wezwanie do działania". Rzućmy okiem na przykładową metodę `toMail`:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} Uwaga: używamy `$this->invoice->id` w swojej metodzie `toENail`. Możesz przekazać dowolne dane potrzebne do wygenerowania wiadomości do konstruktora powiadomień.

W tym przykładzie rejestrujemy powitanie, linię tekstu, wezwanie do działania, a następnie kolejną linię tekstu. Te metody dostarczane przez obiekt `MailMessage` sprawiają, że formatowanie małych e-maili transakcyjnych jest proste i szybkie. Kanał poczty przetłumaczy komponenty wiadomości na ładny, responsywny szablon wiadomości e-mail w formacie HTML z odpowiednikiem tekstowym. Oto przykład wiadomości e-mail wygenerowanej przez kanał `mail`:

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

> {tip} Podczas wysyłania powiadomień o poczcie upewnij się, że ustawiłeś wartość `name` w pliku konfiguracyjnym `config/app.php`. Ta wartość będzie używana w nagłówku i stopce wiadomości e-mail z powiadomieniami.

#### Other Notification Formatting Options - Inne opcje formatowania powiadomień

Zamiast definiować "linie" tekstu w klasie powiadomień, możesz użyć metody `view` w celu określenia niestandardowego szablonu, który będzie używany do renderowania wiadomości e-mail z powiadomieniem:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

Ponadto możesz zwrócić [obiekt do wysłania](/docs/{{version}}/mail) z metody `toMail`:

    use App\Mail\InvoicePaid as Mailable;

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }

<a name="error-messages"></a>
#### Error Messages - Komunikaty o błędach

Niektóre powiadomienia informują użytkowników o błędach, na przykład o nieudanej płatności za fakturę. Możesz wskazać, że wiadomość e-mail dotyczy błędu, wywołując metodę `error` podczas budowania wiadomości. Gdy używasz metody `error` w wiadomości e-mail, przycisk wezwania do działania będzie czerwony zamiast niebieskiego:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### Customizing The Recipient - Dostosowywanie odbiorcy

Podczas wysyłania powiadomień za pośrednictwem kanału `mail`, system powiadomień automatycznie wyszuka właściwość `e-mail` w twoim obiekcie podlegającym obowiązkowi zgłoszenia. Możesz dostosować, który adres e-mail jest używany do dostarczania powiadomienia, definiując metodę `routeNotificationForMail` w encji:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the mail channel.
         *
         * @return string
         */
        public function routeNotificationForMail()
        {
            return $this->email_address;
        }
    }

<a name="customizing-the-subject"></a>
### Customizing The Subject - Dostosowywanie tematu

Domyślnie tematem wiadomości e-mail jest nazwa klasy powiadomienia sformatowanego jako "tytuł sprawy". Tak więc, jeśli twoja klasa powiadomień nazywa się `InvoicePaid`, tematem wiadomości e-mail będzie "Faktura opłacona". Jeśli chcesz określić konkretny temat wiadomości, możesz wywołać metodę `subject` przy tworzeniu wiadomości:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### Customizing The Templates - Dostosowywanie szablonów

Możesz zmodyfikować szablon HTML i zwykły tekst używany przez powiadomienia pocztowe, publikując zasoby pakietu powiadomień. Po uruchomieniu tego polecenia szablony powiadomień pocztowych zostaną umieszczone w katalogu `resources/views/vendor/notifications`:

    php artisan vendor:publish --tag=laravel-notifications

<a name="markdown-mail-notifications"></a>
## Markdown Mail Notifications - Powiadomienia pocztowe Markdown

Powiadomienia e-mail z Markdown umożliwiają korzystanie z gotowych szablonów powiadomień pocztowych, dając jednocześnie więcej swobody w pisaniu dłuższych, niestandardowych wiadomości. Ponieważ wiadomości są napisane w Markdown, Laravel jest w stanie renderować piękne, responsywne szablony HTML dla wiadomości, jednocześnie automatycznie generując tekstowy odpowiednik.

<a name="generating-the-message"></a>
### Generating The Message - Generowanie wiadomości

Aby wygenerować powiadomienie z odpowiednim szablonem Markdown, możesz użyć opcji `--markdown` polecenia `make:notification` Artisan:

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

Podobnie jak wszystkie inne powiadomienia pocztowe, powiadomienia korzystające z szablonów Markdown powinny definiować metodę `toMail` na ich klasie powiadomień. Jednak zamiast używać metod `line` i `action` do skonstruowania powiadomienia, użyj metody `markdown` do określenia nazwy szablonu Markdown, który ma być użyty:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### Writing The Message - Pisanie wiadomości

Powiadomienia poczty w Markdown wykorzystują połączenie składników Blade i składni Markdown, które umożliwiają łatwe tworzenie powiadomień przy jednoczesnym wykorzystaniu wstępnie przygotowanych składników powiadomień Laravel:

    @component('mail::message')
    # Invoice Paid

    Your invoice has been paid!

    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

#### Button Component - Składnik przycisku

Komponent przycisku renderuje wyśrodkowane łącze przycisku. Komponent przyjmuje dwa argumenty: `url` i opcjonalny `color`. Obsługiwane kolory to  `blue`, `green`, i `red`. Możesz dodać do powiadomienia dowolną liczbę elementów przycisku:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
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

> {tip} Jeśli chcesz zbudować całkowicie nowy motyw komponentów Markdown, napisz nowy plik CSS w katalogu `html/themes` i zmień opcję `theme` w pliku konfiguracyjnym `mail`.

<a name="database-notifications"></a>
## Database Notifications - Powiadomienia bazy danych

<a name="database-prerequisites"></a>
### Prerequisites - Wymagania wstępne

Kanał powiadomień `database` przechowuje informacje o powiadomieniu w tabeli bazy danych. Ta tabela zawiera informacje, takie jak typ powiadomienia, a także niestandardowe dane JSON opisujące powiadomienie.

Możesz wysłać zapytanie do tabeli, aby wyświetlić powiadomienia w interfejsie użytkownika aplikacji. Ale zanim to zrobisz, będziesz musiał utworzyć tabelę bazy danych do przechowywania powiadomień. Możesz użyć polecenia `notifications:table` do wygenerowania migracji z poprawnym schematem tabeli:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### Formatting Database Notifications - Formatowanie powiadomień bazy danych

Jeśli powiadomienie obsługuje zapisywanie w tabeli bazy danych, powinieneś zdefiniować metodę `toDatabase` lub `toArray` w klasie powiadomień. Ta metoda otrzyma jednostkę `$notifiable` i powinna zwrócić prostą tablicę PHP. Zwrócona tablica zostanie zakodowana jako JSON i przechowywana w kolumnie `data` tabeli `notifications`. Rzućmy okiem na przykładową metodę `toArray`:

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` Vs. `toArray` - `toDatabase` kontra `toArray`

Metoda `toArray` jest również używana przez kanał `broadcast` w celu określenia, które dane mają być transmitowane do twojego klienta JavaScript. Jeśli chcesz mieć dwie różne reprezentacje tablic dla kanałów `database` i `broadcast`, powinieneś zdefiniować metodę `toDatabase` zamiast metody` toArray`.

<a name="accessing-the-notifications"></a>
### Accessing The Notifications - Dostęp do powiadomień

Gdy powiadomienia są przechowywane w bazie danych, potrzebujesz wygodnego sposobu dostępu do nich z powiadomień. Trait `Illuminate\Notifications\Notifiable`, która jest zawarta w domyślnym modelu `App\User` Laravel, zawiera relację `notifications` Eloquent-a, która zwraca powiadomienia dla encji. Aby pobrać powiadomienia, możesz uzyskać dostęp do tej metody, jak w przypadku każdej innej relacji Eloquent. Domyślnie powiadomienia będą sortowane według znacznika czasu `created_at`:

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Jeśli chcesz odzyskać tylko "nieprzeczytane" powiadomienia, możesz użyć relacji `unreadNotifications`. Znów te powiadomienia będą sortowane według znacznika czasu `created_at`:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} Aby uzyskać dostęp do powiadomień z poziomu klienta JavaScript, należy zdefiniować kontroler powiadomień dla aplikacji, który zwraca powiadomienia dla podmiotu podlegającego obowiązkowi zgłoszenia, takiego jak bieżący użytkownik. Następnie możesz wysłać żądanie HTTP do identyfikatora URI tego kontrolera z twojego klienta JavaScript.

<a name="marking-notifications-as-read"></a>
### Marking Notifications As Read - Oznaczanie powiadomień jako przeczytane

Zazwyczaj użytkownik chce oznaczyć powiadomienie jako "odczytane", gdy użytkownik je wyświetli. Funkcja `Illuminate\Notifications\Notifiable` udostępnia metodę  `markAsRead`, która aktualizuje kolumnę `read_at` w rekordzie bazy danych powiadomienia:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Jednak zamiast zapętlać każde powiadomienie, możesz użyć metody `markAsRead` bezpośrednio w zbiorze powiadomień:

    $user->unreadNotifications->markAsRead();

Możesz również użyć kwerendy aktualizacji zbiorczej do oznaczenia wszystkich powiadomień jako przeczytanych bez pobierania ich z bazy danych:

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Oczywiście możesz `delete` powiadomienia, aby całkowicie je usunąć z tabeli:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Broadcast Notifications - Powiadomienia rozgłoszeniowe

<a name="broadcast-prerequisites"></a>
### Prerequisites - Wymagania wstępne

Przed nadaniem powiadomień należy skonfigurować i zapoznać się z usługami Laravel [rozgłaszania](/docs/{{version}}/broadcasting). Transmisje zdarzeń umożliwiają reagowanie na zdarzenia Laravel uruchamiane po stronie serwera od klienta JavaScript.

<a name="formatting-broadcast-notifications"></a>
### Formatting Broadcast Notifications - Formatowanie powiadomień o transmisji

Kanał `broadcast` rozgłasza powiadomienia za pomocą usług [rozgłaszania wydarzeń]/docs/{{version}}/broadcasting) Laravel-a, umożliwiając klientowi JavaScript złapanie powiadomień w czasie rzeczywistym. Jeśli powiadomienie obsługuje nadawanie, należy zdefiniować metodę `toBroadcast` w klasie powiadomień. Ta metoda otrzyma jednostkę `$notifiable` i powinna zwrócić instancję `BroadcastMessage`. Zwrócone dane zostaną zakodowane jako JSON i nadane do twojego klienta JavaScript. Rzućmy okiem na przykładową metodę `toBroadcast`:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Get the broadcastable representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

#### Broadcast Queue Configuration - Konfiguracja kolejki rozgłoszeniowej

Wszystkie powiadomienia rozgłoszeniowe są ustawiane w kolejce do nadawania. Jeśli chcesz skonfigurować połączenie kolejki lub nazwę kolejki, która jest używana do kolejkowania operacji rozgłaszania, możesz użyć metod `onConnection` i `onQueue` w `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} Oprócz podanych danych powiadomienia rozgłaszane będą również zawierać pole `type` zawierające nazwę klasy powiadomienia.

<a name="listening-for-notifications"></a>
### Listening For Notifications - Nasłuchiwanie powiadomień

Powiadomienia będą transmitowane na prywatnym kanale sformatowanym przy użyciu konwencji `{notification}.{Id}`. Tak więc, jeśli wysyłasz powiadomienie do instancji `App\User` z identyfikatorem" 1 ", powiadomienie będzie transmitowane na prywatnym kanale `App.User.1`. Używając [Laravel Echo](/docs/{{version}}/broadcasting), możesz łatwo nasłuchiwać powiadomień na kanale za pomocą metody helpera `notification`:

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

#### Customizing The Notification Channel - Dostosowywanie kanału powiadomień

Jeśli chcesz dostosować, na jakie kanały powiadamia się nadawcę, możesz zdefiniować metodę `receivesBroadcastNotificationsOn` na zgłaszającej się jednostce:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The channels the user receives notification broadcasts on.
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## SMS Notifications - Powiadomienia SMS

<a name="sms-prerequisites"></a>
### Prerequisites - Wymagania wstępne

Wysyłanie powiadomień SMS w Laravel jest obsługiwane przez [Nexmo](https://www.nexmo.com/). Zanim będziesz mógł wysyłać powiadomienia przez Nexmo, musisz zainstalować pakiet `nexmo/client` Composer i dodać kilka opcji konfiguracyjnych do pliku konfiguracyjnego `config/services.php`. Możesz skopiować przykładową konfigurację poniżej, aby rozpocząć:

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

Opcja `sms_from` to numer telefonu, z którego będą wysyłane twoje wiadomości SMS. Powinieneś wygenerować numer telefonu do swojej aplikacji w panelu kontrolnym Nexmo.

<a name="formatting-sms-notifications"></a>
### Formatting SMS Notifications - Formatowanie powiadomień SMS

Jeśli powiadomienie obsługuje wysyłanie jako SMS, powinieneś zdefiniować metodę `toNexmo` w klasie powiadomień. Ta metoda otrzyma jednostkę `$notifiable` i powinna zwrócić instancję `Illuminate\Notifications\Messages\NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

#### Unicode Content - Treść Unicode

Jeśli twoja wiadomość SMS będzie zawierała znaki Unicode, powinieneś wywołać metodę `unicode` podczas konstruowania instancji` NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### Customizing The "From" Number - Dostosowywanie numeru "Od"

Jeśli chcesz wysyłać powiadomienia z numeru telefonu, który różni się od numeru telefonu określonego w pliku `config/services.php`, możesz użyć metody `from` w instancji `NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### Routing SMS Notifications - Routing powiadomień SMS

Podczas wysyłania powiadomień za pośrednictwem kanału `nexmo`, system powiadomień automatycznie poszukuje atrybutu `numer_telefonu` w jednostce podlegającej obowiązkowi zgłoszenia. Jeśli chcesz dostosować numer telefonu, do którego dostarczane jest powiadomienie, zdefiniuj metodę `routeNotificationForNexmo` w obiekcie:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Nexmo channel.
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Slack Notifications - Powiadomienia Slack

<a name="slack-prerequisites"></a>
### Prerequisites - Wymagania wstępne

Zanim będziesz mógł wysyłać powiadomienia przez Slack, musisz zainstalować bibliotekę HTTP Guzzle przez Composer:

    composer require guzzlehttp/guzzle

Będziesz także musiał skonfigurować integrację ["Przychodzące Webhook"](https://api.slack.com/incoming-webhooks) dla zespołu Slack. Ta integracja zapewni Ci adres URL, którego możesz użyć, gdy [routing powiadomień o Slack](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### Formatting Slack Notifications - Formatowanie powiadomień Slack

Jeśli powiadomienie obsługuje wysyłanie jako wiadomość Slack, powinieneś zdefiniować metodę `toSlack` w klasie powiadomień. Ta metoda otrzyma jednostkę `$notifiable` i powinna zwrócić instancję `Illuminate\Notifications\Messages\SlackMessage`. Wiadomości Slack mogą zawierać treść tekstową, a także "załącznik", który formatuje dodatkowy tekst lub tablicę pól. Rzućmy okiem na podstawowy przykład `toSlack`:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

W tym przykładzie wysyłamy pojedynczą linię tekstu do Slacka, co spowoduje utworzenie komunikatu wyglądającego następująco:

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

#### Customizing The Sender & Recipient - Dostosowywanie nadawcy i odbiorcy

Możesz użyć metod `from` i `to`, aby dostosować nadawcę i odbiorcę. Metoda `from` akceptuje nazwę użytkownika i identyfikator emoji, natomiast metoda `to` akceptuje kanał lub nazwę użytkownika:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }

Możesz także użyć obrazu logo zamiast emoji:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/favicon.png')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>
### Slack Attachments - Załaczniki Slack

Możesz również dodać "załączniki" do wiadomości Slack. Załączniki zapewniają bogatsze opcje formatowania niż proste wiadomości tekstowe. W tym przykładzie wyślemy powiadomienie o błędzie dotyczące wyjątku, który wystąpił w aplikacji, w tym link umożliwiający wyświetlenie więcej szczegółów na temat wyjątku:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

Powyższy przykład wygeneruje komunikat Slack, który wygląda następująco:

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

Załączniki pozwalają również określić tablicę danych, które powinny być prezentowane użytkownikowi. Podane dane zostaną przedstawione w formie tabelarycznej dla łatwego czytania:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

W powyższym przykładzie zostanie utworzona wiadomość Slack, która wygląda następująco:

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

#### Markdown Attachment Content - Zawartość załacznika Markdown

Jeśli niektóre z twoich załączników zawierają Markdown, możesz użyć metody `markdown`, aby polecić Slackowi, aby analizował i wyświetlał dane pola załącznika jako tekst sformatowany w Markdown. Wartości akceptowane przez tę metodę to: `pretext`, `text`, i / lub `fields`. Aby uzyskać więcej informacji na temat formatowania załączników Slack, zapoznaj się z [dokumentacją API Slack API](https://api.slack.com/docs/message-formatting#message_formatting):

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### Routing Slack Notifications - Routing powiadomień Slack

Aby skierować powiadomienia o zwarciu do właściwej lokalizacji, zdefiniuj metodę `routeNotificationForSlack` na swoim obiekcie podlegającym obowiązkowi zgłoszenia. Powinno to zwrócić adres URL webhooka, do którego powinno zostać dostarczone powiadomienie. Adresy URL Webhook mogą być generowane przez dodanie usługi "Przychodzący Webhook" do zespołu Slack:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## Notification Events - Zdarzenia powiadomień

Po wysłaniu powiadomienia zdarzenie `Illuminate\Notifications\Events\NotificationSent` jest wywoływane przez system powiadomień. Zawiera ono "notyfikowaną" encję i samą instancję powiadomienia. Możesz zarejestrować detektory dla tego wydarzenia w swoim `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} Po zarejestrowaniu słuchaczy w `EventServiceProvider`, użyj polecenia` event:generate` Artisan, aby szybko wygenerować klasy słuchacza.

W odbiorniku zdarzeń możesz uzyskać dostęp do właściwości `notifiable`, `notification`, and `channel` w zdarzeniu, aby dowiedzieć się więcej o odbiorcy powiadomienia lub o samym powiadomieniu:

    /**
     * Handle the event.
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="custom-channels"></a>
## Custom Channels - Kanały niestandardowe

Laravel jest dostarczany z kilkoma kanałami powiadomień, ale możesz napisać własne sterowniki, aby dostarczać powiadomienia za pośrednictwem innych kanałów. Laravel czyni to prostym. Aby rozpocząć, zdefiniuj klasę zawierającą metodę `send`. Metoda powinna otrzymać dwa argumenty: `$notifiable` oraz `$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Send the given notification.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Send notification to the $notifiable instance...
        }
    }

Po zdefiniowaniu klasy kanałów powiadomień możesz zwrócić nazwę klasy za pomocą metody `via` dowolnego z powiadomień:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Get the notification channels.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Get the voice representation of the notification.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
