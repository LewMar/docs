# Mocking

- [Introduction - Wprowadzenie](#introduction)
- [Bus Fake - Fałszywy Bus](#bus-fake)
- [Event Fake - Fałszywe zdarzenie](#event-fake)
- [Mail Fake - Fałszywa Poczta](#mail-fake)
- [Notification Fake - Fałszywe powiadomienie](#notification-fake)
- [Queue Fake - Fałszywa kolejka](#queue-fake)
- [Storage Fake - Fałszywy dysk](#storage-fake)
- [Facades - Fasady](#mocking-facades)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Podczas testowania aplikacji Laravel możesz chcieć "kpić" z pewnych aspektów aplikacji, aby nie były faktycznie wykonywane podczas danego testu. Na przykład podczas testowania kontrolera, który wywołuje zdarzenie, możesz chcieć wyśmiać detektory zdarzeń, aby nie zostały faktycznie wykonane podczas testu. Pozwala to tylko przetestować odpowiedź HTTP kontrolera bez martwienia się o wykonanie detektorów zdarzeń, ponieważ detektory zdarzeń mogą być testowane we własnym przypadku testowym.

Laravel dostarcza pomocników do kpiących z wydarzeń, zadań i fasady po wyjęciu z pudełka. Te pomocniki zapewniają przede wszystkim warstwę wygodną przez Mockery, dzięki czemu nie trzeba ręcznie wykonywać skomplikowanych wywołań metod Mockery. Oczywiście możesz używać [Mockery](http://docs.mockery.io/en/latest/) lub PHPUnit do tworzenia własnych mocks lub szpiegów.

<a name="bus-fake"></a>
## Bus Fake - Fałszywy Bus

Jako alternatywę dla szyderstwa możesz użyć metody `fake` fasady `Bus`, aby zapobiec wysyłaniu zadań. Podczas korzystania z podróbek, asercje są wykonywane po wykonaniu kodu, który jest testowany:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Bus;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Bus::fake();

            // Perform order shipping...

            Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was not dispatched...
            Bus::assertNotDispatched(AnotherJob::class);
        }
    }

<a name="event-fake"></a>
## Event Fake - Fałszywe zdarzenie

Zamiast szyderstwa możesz użyć metody `fake` fasady `Event`, aby zapobiec wykonywaniu wszystkich detektorów zdarzeń. Możesz wtedy stwierdzić, że zdarzenia zostały wysłane, a nawet sprawdzić otrzymane dane. Podczas korzystania z podróbek, asercje są wykonywane po wykonaniu kodu, który jest testowany:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            Event::fake();

            // Perform order shipping...

            Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
                return $e->order->id === $order->id;
            });

            Event::assertNotDispatched(OrderFailedToShip::class);
        }
    }

<a name="mail-fake"></a>
## Mail Fake - Fałszywa Poczta

Możesz użyć metody `fake` w elewacji `Mail`, aby zapobiec wysyłaniu wiadomości. Możesz wtedy stwierdzić, że [mailables](/docs/{{version}}/mail) zostały wysłane do użytkowników, a nawet sprawdzić otrzymane dane. Podczas korzystania z podróbek, asercje są wykonywane po wykonaniu kodu, który jest testowany:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Mail\OrderShipped;
    use Illuminate\Support\Facades\Mail;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();

            // Perform order shipping...

            Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
                return $mail->order->id === $order->id;
            });

            // Assert a message was sent to the given users...
            Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
                return $mail->hasTo($user->email) &&
                       $mail->hasCc('...') &&
                       $mail->hasBcc('...');
            });

            // Assert a mailable was sent twice...
            Mail::assertSent(OrderShipped::class, 2);

            // Assert a mailable was not sent...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

Jeśli kolejkujesz pocztę do dostarczenia w tle, powinieneś użyć metody `assertQueued` zamiast `assertSent`:

    Mail::assertQueued(...);
    Mail::assertNotQueued(...);

<a name="notification-fake"></a>
## Notification Fake - Fałszywe powiadomienia

Możesz użyć metody `fake` fasady `Notification`, aby zapobiec wysyłaniu powiadomień. Możesz następnie stwierdzić, że [powiadomienia](/docs/{{version}}/notifications) zostały wysłane do użytkowników, a nawet sprawdzone przez nich dane. Podczas korzystania z podróbek, asercje są wykonywane po wykonaniu kodu, który jest testowany:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();

            // Perform order shipping...

            Notification::assertSentTo(
                $user,
                OrderShipped::class,
                function ($notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );

            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );
        }
    }

<a name="queue-fake"></a>
## Queue Fake - Fałszywa kolejka

Zamiast szyderstwa możesz użyć metody `fake` fasady `Queue`, aby zapobiec kolejkowaniu zadań. Możesz wtedy stwierdzić, że zlecenia zostały przeniesione do kolejki, a nawet sprawdzić otrzymane dane. Podczas korzystania z podróbek, asercje są wykonywane po wykonaniu kodu, który jest testowany:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();

            // Perform order shipping...

            Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Assert a job was pushed twice...
            Queue::assertPushed(ShipOrder::class, 2);

            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);
        }
    }

<a name="storage-fake"></a>
## Storage Fake - Fałszywy dysk

Metoda `fake` fasadowa` Storage` pozwala na łatwe generowanie fałszywego dysku, który w połączeniu z narzędziami do generowania plików klasy "UploadedFile" znacznie upraszcza testowanie przesyłania plików. Na przykład:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

> {tip} Domyślnie metoda `fake` usuwa wszystkie pliki z katalogu tymczasowego. Jeśli chcesz zachować te pliki, możesz zamiast tego użyć metody "persistentFake".

<a name="mocking-facades"></a>
## Facades - Fasady

W przeciwieństwie do tradycyjnych wywołań metod statycznych, wywołanie [fasady](/docs/{{version}}/facades). Zapewnia to ogromną przewagę nad tradycyjnymi metodami statycznymi i zapewnia taką samą testowalność, jaka byłaby w przypadku stosowania iniekcji zależności. Podczas testowania często możesz chcieć wyśmiewać połączenie z fasadą Laravel w jednym ze swoich kontrolerów. Na przykład rozważ następujące działanie kontrolera:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

Możemy kpić z połączenia z elewacją `Cache` za pomocą metody `shouldReceive`, która zwróci instancję [Mockery](https://github.com/padraic/mockery). Ponieważ fasady są faktycznie rozwiązywane i zarządzane przez Laravel [kontener usług](/docs/{{version}}/container), mają o wiele większą sprawność niż typowa klasa statyczna. Na przykład, wyśmiewajmy nasze wezwanie do metody `get` elewacji fasady:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> {note} Nie powinieneś kpić z fasady `Request`. Zamiast tego przekazuj dane wejściowe, które chcesz, do metod pomocnika HTTP, takich jak `get` i `post` podczas uruchamiania testu. Podobnie, zamiast kpić z fasady `Config`, wywołaj w testach metodę `Config::set`.
