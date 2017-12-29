# Validation

- [Introduction - Wprowadzenie](#introduction)
- [Validation Quickstart - Walidacja szybki start](#validation-quickstart)
    - [Defining The Routes - Definiowanie tras](#quick-defining-the-routes)
    - [Creating The Controller - Tworzenie kontrolera](#quick-creating-the-controller)
    - [Writing The Validation Logic - Pisanie logiki sprawdzania poprawności](#quick-writing-the-validation-logic)
    - [Displaying The Validation Errors - Wyświetlanie błędów sprawdzania poprawności](#quick-displaying-the-validation-errors)
    - [A Note On Optional Fields - Uwaga na opcjonalnych polach](#a-note-on-optional-fields)
- [Form Request Validation - Zatwierdzenie wniosku o formularz](#form-request-validation)
    - [Creating Form Requests - Tworzenie żądań formularzy](#creating-form-requests)
    - [Authorizing Form Requests - Autoryzacja formularzy wniosków](#authorizing-form-requests)
    - [Customizing The Error Messages - Dostosowywanie komunikatów o błędach](#customizing-the-error-messages)
- [Manually Creating Validators - Ręczne tworzenie walidatorów](#manually-creating-validators)
    - [Automatic Redirection - Automatyczne przekierowanie](#automatic-redirection)
    - [Named Error Bags - Nazwane pojemniki błedów](#named-error-bags)
    - [After Validation Hook - Po sprawdzeniu poprawności rozszerzenia](#after-validation-hook)
- [Working With Error Messages - Praca z komunikatami o błędach](#working-with-error-messages)
    - [Custom Error Messages - Niestandardowe komunikaty o błędach](#custom-error-messages)
- [Available Validation Rules - Dostępne zasady sprawdzania poprawności](#available-validation-rules)
- [Conditionally Adding Rules - Warunkowo dodawanie reguł](#conditionally-adding-rules)
- [Validating Arrays - Sprawdzanie poprawności tablic](#validating-arrays)
- [Custom Validation Rules - Niestandardowe reguły sprawdzania poprawności](#custom-validation-rules)
    - [Using Rule Objects - Korzystanie z obiektów reguł](#using-rule-objects)
    - [Using Extensions - Używanie rozszerzeń](#using-extensions)

<a name="introduction"></a>
## Introduction - Wprowadzenie

Laravel oferuje kilka różnych metod sprawdzania danych przychodzących aplikacji. Domyślnie klasa kontrolera podstawowego Laravel używa cechy `ValidatesRequests`, która zapewnia wygodną metodę sprawdzania przychodzących żądań HTTP za pomocą różnych potężnych reguł sprawdzania poprawności.

<a name="validation-quickstart"></a>
## Validation Quickstart - Walidacja szybki start

Aby dowiedzieć się więcej o potężnych funkcjach sprawdzania poprawności Laravel, spójrzmy na pełny przykład sprawdzania poprawności formularza i wyświetlania komunikatów o błędach z powrotem do użytkownika.

<a name="quick-defining-the-routes"></a>
### Defining The Routes - Definiowanie tras

Najpierw załóżmy, że mamy zdefiniowane następujące trasy w naszym pliku `routes/web.php`:

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

Oczywiście, trasa `GET` wyświetli formularz dla użytkownika, aby utworzyć nowy post na blogu, podczas gdy trasa `POST` będzie przechowywać nowy wpis blogu w bazie danych.

<a name="quick-creating-the-controller"></a>
### Creating The Controller - Tworzenie kontrolera

Następnie przyjrzyjmy się prostemu kontrolerowi, który obsługuje te trasy. Na razie opuścimy metodę `store` na teraz:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Show the form to create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### Writing The Validation Logic - Pisanie logiki sprawdzania poprawności

Teraz jesteśmy gotowi wypełnić naszą metodę `store` logiką, aby zweryfikować nowy wpis na blogu. W tym celu użyjemy metody `validate` dostarczonej przez obiekt `Illuminate\Http\Request`. Jeśli zasady sprawdzania poprawności przejdą pomyślnie, Twój kod będzie normalnie wykonywany; jednak w przypadku niepowodzenia sprawdzania poprawności zostanie zgłoszony wyjątek, a odpowiednia odpowiedź o błędzie zostanie automatycznie wysłana do użytkownika. W przypadku tradycyjnego żądania HTTP zostanie wygenerowana odpowiedź przekierowania, a odpowiedź JSON zostanie wysłana dla żądań AJAX.

Aby lepiej zrozumieć metodę `validate`, wróćmy do metody `store`:

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid...
    }

Jak widać, po prostu przekazujemy pożądane reguły sprawdzania poprawności do metody `validate`. Ponownie, jeśli walidacja nie powiedzie się, automatycznie zostanie wygenerowana odpowiednia odpowiedź. Jeśli sprawdzanie poprawności przejdzie, nasz kontroler będzie kontynuował normalne wykonywanie.

#### Stopping On First Validation Failure - Zatrzymywanie w przypadku błędu pierwszej walidacji

Czasami możesz przestać uruchamiać reguły sprawdzania poprawności dla atrybutu po pierwszym niepowodzeniu sprawdzania poprawności. Aby to zrobić, przypisz regułę `bail` do atrybutu:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

W tym przykładzie, jeśli reguła `unique` w atrybucie `title` nie powiedzie się, reguła `max` nie zostanie sprawdzona. Reguły zostaną zatwierdzone w kolejności, w jakiej zostały przypisane.

#### A Note On Nested Attributes - Uwaga na zagnieżdżonych atrybutach

Jeśli Twoje żądanie HTTP zawiera parametry "zagnieżdżone", możesz je określić w regułach sprawdzania poprawności, używając składni "kropka":

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Displaying The Validation Errors - Wyświetlanie błędów sprawdzania poprawności

Co się stanie, jeśli przychodzące parametry żądania nie przejdą określonych reguł sprawdzania poprawności? Jak wspomniano wcześniej, Laravel automatycznie przekieruje użytkownika z powrotem do poprzedniej lokalizacji. Ponadto wszystkie błędy sprawdzania poprawności będą automatycznie [przesłane do sesji](/docs/{{version}}/session#flash-data).

Ponownie zauważmy, że nie musieliśmy jawnie wiązać komunikatów o błędach z widokiem w naszej trasie `GET`. Dzieje się tak dlatego, że Laravel sprawdza błędy w danych sesji i automatycznie wiąże je z widokiem, jeśli są dostępne. Zmienna `$errors` będzie instancją `Illuminate\Support\MessageBag`. Aby uzyskać więcej informacji na temat pracy z tym obiektem, [sprawdź jego dokumentację](#working-with-error-messages).

> {tip} Zmienna `$errors` jest powiązana z widokiem przez oprogramowanie pośredniczące `Illuminate\View\Middleware\ShareErrorsFromSession`, które jest dostarczane przez grupę oprogramowania pośredniczącego `web`. **Gdy to oprogramowanie pośrednie zostanie zastosowane, zmienna `$errors` będzie zawsze dostępna w twoich widokach**, pozwalając ci wygodnie założyć, że zmienna `$errors` jest zawsze zdefiniowana i może być bezpiecznie użyta.

W naszym przykładzie użytkownik zostanie przekierowany do metody `create` naszego kontrolera, gdy walidacja nie powiedzie się, co pozwoli nam wyświetlić komunikaty o błędach w widoku:

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Form -->

<a name="a-note-on-optional-fields"></a>
### A Note On Optional Fields - Uwaga na opcjonalnych polach

Domyślnie Laravel zawiera oprogramowanie pośredniczące `TrimStrings` i `ConvertEmptyStringsToNull` w globalnym stosie oprogramowania pośredniego twojej aplikacji. Te oprogramowanie pośrednie są wymienione w stosie przez klasę `App\Http\Kernel`. Z tego powodu często trzeba oznaczyć pola "opcjonalne" jako `nullable`, jeśli nie chcesz, aby walidator traktował wartości `null` jako nieprawidłowe. Na przykład:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

W tym przykładzie określamy, że pole `publish_at` może mieć wartość `null` lub poprawną reprezentację daty. Jeśli modyfikator `nullable` nie zostanie dodany do definicji reguły, walidator potraktuje" null "jako niepoprawną datę.

<a name="quick-ajax-requests-and-validation"></a>
#### AJAX Requests & Validation - Żądania AJAX i walidacja

W tym przykładzie użyliśmy tradycyjnego formularza do wysłania danych do aplikacji. Jednak wiele aplikacji korzysta z żądań AJAX. Podczas używania metody `validate` podczas żądania AJAX, Laravel nie wygeneruje odpowiedzi przekierowania. Zamiast tego Laravel generuje odpowiedź JSON zawierającą wszystkie błędy sprawdzania poprawności. Ta odpowiedź JSON zostanie wysłana z kodem statusu HTTP 422.

<a name="form-request-validation"></a>
## Form Request Validation - Zatwierdzenie wniosku o formularz

<a name="creating-form-requests"></a>
### Creating Form Requests - Tworzenie żądań formularzy

Aby uzyskać bardziej złożone scenariusze walidacji, możesz utworzyć "żądanie formularza". Żądania formularza są niestandardowymi klasami żądań, które zawierają logikę sprawdzania poprawności. Aby utworzyć klasę żądania formularza, użyj polecenia `make:request` Artisan CLI:

    php artisan make:request StoreBlogPost

Wygenerowana klasa zostanie umieszczona w katalogu `app/Http/Requests`. Jeśli ten katalog nie istnieje, zostanie utworzony po uruchomieniu polecenia `make:request`. Dodajmy kilka reguł sprawdzania poprawności do metody `rules`:

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

Jak zatem ocenia się zasady walidacji? Wszystko, co musisz zrobić, to wpisać wskazowkę żądania w swojej metodzie sterownika. Żądanie formularza przychodzącego jest sprawdzane przed wywołaniem metody kontrolera, co oznacza, że nie musisz zaśmiecać kontrolera żadną logiką sprawdzania poprawności:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...
    }

Jeśli sprawdzanie poprawności nie powiedzie się, zostanie wygenerowana odpowiedź przekierowania w celu odesłania użytkownika do poprzedniej lokalizacji. Błędy będą również wyświetlane podczas sesji, dzięki czemu będą dostępne do wyświetlenia. Jeśli żądanie było żądaniem AJAX, odpowiedź HTTP z kodem statusu 422 zostanie zwrócona do użytkownika, w tym reprezentacja JSON błędów sprawdzania poprawności.

#### Adding After Hooks To Form Requests - Dodawanie rozszerzeń po formularzu żądania

Jeśli chcesz dodać rozszerzenie "po" żądaniu formularza, możesz użyć metody `withValidator`. Ta metoda odbiera w pełni skonstruowany moduł sprawdzania poprawności, umożliwiając wywoływanie dowolnych metod przed sprawdzeniem reguł sprawdzania poprawności:

    /**
     * Configure the validator instance.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }

<a name="authorizing-form-requests"></a>
### Authorizing Form Requests - Autoryzacja formularzy wniosków

Klasa żądania formularza zawiera również metodę `authorize`. W ramach tej metody można sprawdzić, czy uwierzytelniony użytkownik rzeczywiście ma uprawnienia do aktualizacji danego zasobu. Na przykład możesz ustalić, czy użytkownik rzeczywiście posiada komentarz do bloga, który próbuje zaktualizować:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

Ponieważ wszystkie żądania formularzy rozszerzają podstawową klasę żądania Laravel, możemy użyć metody `user` w celu uzyskania dostępu do aktualnie uwierzytelnionego użytkownika. Zwróć również uwagę na wywołanie metody `route` w powyższym przykładzie. Ta metoda zapewnia dostęp do parametrów identyfikatora URI zdefiniowanych dla wywoływanej trasy, takich jak parametr `{comment}` w poniższym przykładzie:

    Route::post('comment/{comment}');

Jeśli metoda `authorize` zwróci `false`, odpowiedź HTTP z kodem statusu 403 zostanie automatycznie zwrócona, a metoda kontrolera nie zostanie wykonana.

Jeśli planujesz mieć logikę autoryzacji w innej części aplikacji, po prostu zwróć `true` z metody` authorize`:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

<a name="customizing-the-error-messages"></a>
### Customizing The Error Messages - Dostosowywanie komunikatów o błędach

Możesz dostosować komunikaty o błędach używane przez żądanie formularza, przesłoniwszy metodę `messages`. Ta metoda powinna zwrócić tablicę par atrybutów / reguł i odpowiadających im komunikatów o błędach:

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="manually-creating-validators"></a>
## Manually Creating Validators - Ręczne tworzenie walidatorów

Jeśli nie chcesz używać metody `validate` na żądaniu, możesz utworzyć instancję sprawdzania poprawności ręcznie za pomocą opcji `Validator` [fasada](/docs/{{version}}/facades). Metoda `make` na elewacji generuje nową instancję sprawdzania poprawności:

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

Pierwszym argumentem przekazanym do metody `make` są dane podlegające walidacji. Drugi argument to zasady walidacji, które należy zastosować do danych.

Po sprawdzeniu, czy sprawdzanie poprawności żądania nie powiodło się, możesz użyć metody `withErrors`, aby przesłać komunikaty o błędach do sesji. Podczas korzystania z tej metody zmienna `$errors` zostanie automatycznie udostępniona Twoim widokom po przekierowaniu, co pozwala łatwo wyświetlić je ponownie użytkownikowi. Metoda `withErrors` akceptuje walidator, `MessageBag` lub  tablicę `array` PHP.
 
<a name="automatic-redirection"></a>
### Automatic Redirection - Automatyczne przekierowanie

Jeśli chcesz utworzyć instancję sprawdzania poprawności ręcznie, ale nadal możesz skorzystać z automatycznego przekierowania oferowanego przez metodę `validate` żądania, możesz wywołać metodę `validate` na istniejącej instancji sprawdzania poprawności. Jeśli sprawdzanie poprawności nie powiedzie się, użytkownik zostanie automatycznie przekierowany lub, w przypadku żądania AJAX, zwrócona zostanie odpowiedź JSON:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

<a name="named-error-bags"></a>
### Named Error Bags - Nazwane pojemniki błedów

Jeśli masz wiele formularzy na jednej stronie, możesz nazwać `MessageBag` błędów, pozwalając ci pobrać komunikaty o błędach dla konkretnego formularza. Po prostu podaj nazwę jako drugi argument do `withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');

Następnie możesz uzyskać dostęp do nazwanej instancji `MessageBag` ze zmiennej `$errors`:

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### After Validation Hook - Po sprawdzeniu poprawności rozszerzenia

Walidator umożliwia także dołączanie wywołań zwrotnych, które będą uruchamiane po zakończeniu sprawdzania poprawności. Pozwala to w łatwy sposób przeprowadzić dalszą walidację, a nawet dodać więcej komunikatów o błędach do kolekcji wiadomości. Aby rozpocząć, użyj metody `after` na instancji sprawdzania poprawności:

    $validator = Validator::make(...);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>
## Working With Error Messages - Praca z komunikatami o błędach

Po wywołaniu metody `errors` na instancji `Validator` otrzymasz instancję `Illuminate\Support\MessageBag`, która ma wiele wygodnych metod pracy z komunikatami o błędach. Zmienna `$errors`, która jest automatycznie udostępniana wszystkim widokom, jest również instancją klasy `MessageBag`.

#### Retrieving The First Error Message For A Field

To retrieve the first error message for a given field, use the `first` method:

    $errors = $validator->errors();

    echo $errors->first('email');

#### Retrieving All Error Messages For A Field - Pobieranie pierwszego komunikatu o błędzie dla pola

Jeśli potrzebujesz pobrać tablicę wszystkich wiadomości dla danego pola, użyj metody `get`:

    foreach ($errors->get('email') as $message) {
        //
    }

Jeśli sprawdzasz poprawność pola formularza tablicowego, możesz pobrać wszystkie wiadomości dla każdego elementu tablicy używając znaku `*`:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### Retrieving All Error Messages For All Fields - Pobieranie wszystkich komunikatów o błędach dla wszystkich pól

Aby pobrać tablicę wszystkich wiadomości dla wszystkich pól, użyj metody "all":

    foreach ($errors->all() as $message) {
        //
    }

#### Determining If Messages Exist For A Field - Określanie, czy wiadomości istnieją dla pola

Metodę `has` można wykorzystać do określenia, czy istnieją jakieś komunikaty o błędach dla danego pola:

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### Custom Error Messages - Niestandardowe komunikaty o błędach

W razie potrzeby można użyć niestandardowych komunikatów o błędach do sprawdzania poprawności zamiast wartości domyślnych. Istnieje kilka sposobów określania niestandardowych wiadomości. Po pierwsze możesz przekazać niestandardowe komunikaty jako trzeci argument metody `Validator::make`:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

W tym przykładzie element zastępujący `:attribute` zostanie zastąpiony rzeczywistą nazwą pola, którego dotyczy walidacja. Możesz także wykorzystywać innych posiadaczy miejsc w wiadomościach walidacyjnych. Na przykład:

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### Specifying A Custom Message For A Given Attribute - Określanie niestandardowej wiadomości dla danego atrybutu

Czasami możesz chcieć określić niestandardowe komunikaty o błędach tylko dla określonego pola. Możesz to zrobić za pomocą notacji "kropka". Podaj najpierw nazwę atrybutu, a następnie regułę:


    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### Specifying Custom Messages In Language Files - Określanie niestandardowych wiadomości w plikach językowych

W większości przypadków prawdopodobnie określisz własne komunikaty w pliku językowym zamiast przekazywać je bezpośrednio do `Validator`. Aby to zrobić, dodaj swoje wiadomości do tablicy `custom` w pliku językowym `resources/lang/xx/validation.php`.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

#### Specifying Custom Attributes In Language Files - Określanie atrybutów niestandardowych w plikach językowych

Jeśli chcesz, aby część komunikatu ":atrybut" została zamieniona na niestandardową nazwę atrybutu, możesz określić niestandardową nazwę w tablicy `attributes` pliku `resources/lang/xx/validation.php` :


    'attributes' => [
        'email' => 'email address',
    ],

<a name="available-validation-rules"></a>
## Available Validation Rules - Dostępne zasady sprawdzania poprawności

Poniżej znajduje się lista wszystkich dostępnych reguł walidacji i ich funkcji:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[After Or Equal (Date)](#rule-after-or-equal)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Nullable](#rule-nullable)
[Not In](#rule-not-in)
[Numeric](#rule-numeric)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)

</div>

<a name="rule-accepted"></a>
#### accepted - zaakceptowany

Pole w trakcie sprawdzania poprawności musi mieć wartość _yes_, _on_, _1_ lub _true_. Jest to przydatne do sprawdzania akceptacji "Warunków korzystania z usługi".

<a name="rule-active-url"></a>
#### active_url - aktywny adres URL

Pole w trakcie sprawdzania poprawności musi mieć poprawny rekord A lub AAAA zgodnie z funkcją PHP `dns_get_record`.

<a name="rule-after"></a>
#### after:_date_ - po:_data_

Sprawdzane pole musi być wartością po określonej dacie. Daty zostaną przekazane do funkcji PHP `strtotime`:

    'start_date' => 'required|date|after:tomorrow'

Zamiast podawać ciąg daty, który ma być oceniany przez `strtotime`, możesz określić inne pole do porównania z datą:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_ - po\_lub\_równe:_data_

Pole w trakcie sprawdzania poprawności musi być wartością równą lub dłuższą niż podana data. Aby uzyskać więcej informacji, zobacz regułę [po](#rule-after).

<a name="rule-alpha"></a>
#### alpha - litery

Sprawdzane pole musi składać się wyłącznie z liter alfabetu.

<a name="rule-alpha-dash"></a>
#### alpha_dash - litery, kreski i podkreślenia

W polu sprawdzania poprawności mogą znajdować się znaki alfanumeryczne, a także kreski i podkreślenia.

<a name="rule-alpha-num"></a>
#### alpha_num - alfanumeryczne

Sprawdzane pole musi składać się wyłącznie ze znaków alfanumerycznych.

<a name="rule-array"></a>
#### array - tablica

Sprawdzane pole musi być tablicą `array` PHP.

<a name="rule-before"></a>
#### before:_date_ - przed:_data_

Sprawdzane pole musi być wartością poprzedzającą podaną datę. Daty zostaną przekazane do funkcji PHP `strtotime`.

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_ - przed\_lub\_równe:_data_

Sprawdzane pole musi być wartością poprzedzającą lub równą podanej dacie. Daty zostaną przekazane do funkcji PHP `strtotime`.

<a name="rule-between"></a>
#### between:_min_,_max_ - pomiędzy:_minimum_,_maksimum_

Pole w walidacji musi mieć rozmiar między podanymi _min_ i _max_. Łańcuchy, cyfry, tablice i pliki są oceniane w taki sam sposób, jak reguła [`rozmiar`](#rule-size).

<a name="rule-boolean"></a>
#### boolean - zmienna logiczna

Pole podlegające walidacji musi mieć możliwość przesyłania jako wartość logiczną. Zaakceptowane dane wejściowe to `true`, `false`, `1`, `0`, `"1"` i `"0"`.

<a name="rule-confirmed"></a>
#### confirmed - porównanie pól

W polu sprawdzania poprawności musi znajdować się pasujące pole `foo_confirmation`. Na przykład, jeśli w polu sprawdzania poprawności jest `password`, w polu wejściowym musi znajdować się pasujące pole `password_confirmation`.

<a name="rule-date"></a>
#### date - data

Pole w trakcie sprawdzania poprawności musi być poprawną datą zgodnie z funkcją PHP `strtotime`.

<a name="rule-date-equals"></a>
#### date_equals:_date_ - data_równa:_dacie_

Pole podlegające walidacji musi być równe podanej dacie. Daty zostaną przekazane do funkcji PHP `strtotime`.

<a name="rule-date-format"></a>
#### date_format:_format_

Sprawdzane pole musi pasować do podanego _format_. Powinieneś użyć **albo** `date` lub `date_format` podczas sprawdzania poprawności pola, a nie obu.

<a name="rule-different"></a>
#### different:_field_ - inne:_pole_

Sprawdzane pole musi mieć inną wartość niż _field_.

<a name="rule-digits"></a>
#### digits:_value_ - numeryczne:_długość_

Pole w walidacji musi mieć wartość _numeric_ i musi mieć dokładną długość _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_ - numeryczne_pomiedzy:_długośćMinimalna_,_długośćMaksymalna_

Sprawdzane pole musi mieć długość między podanymi _min_ i _max_.

<a name="rule-dimensions"></a>
#### dimensions - wymiar

Plik sprawdzany pod kątem poprawności musi być obrazem spełniającym ograniczenia wymiarów określone przez parametry reguły:

    'avatar' => 'dimensions:min_width=100,min_height=200'

Dostępne ograniczenia to: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Ograniczenie _ratio_ powinno być reprezentowane jako szerokość podzielona przez wysokość. Może to być określone przez instrukcję typu `3/2` lub float jak` 1.5`:

    'avatar' => 'dimensions:ratio=3/2'

Ponieważ ta reguła wymaga kilku argumentów, możesz użyć metody `Rule::dimensions` do płynnego konstruowania reguły:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct - niedwuznaczny

Podczas pracy z tablicami sprawdzane pole nie może zawierać żadnych duplikatów.

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

Pole w trakcie sprawdzania poprawności musi być sformatowane jako adres e-mail.

<a name="rule-exists"></a>
#### exists:_table_,_column_ - istnieje:_tabela_,kolumna

Pole w trakcie sprawdzania poprawności musi istnieć na danej tabeli bazy danych.

#### Basic Usage Of Exists Rule - Podstawowe użycie istniejącej reguły

    'state' => 'exists:states'

#### Specifying A Custom Column Name - Określanie niestandardowej nazwy kolumny

    'state' => 'exists:states,abbreviation'

Czasami może zajść potrzeba określenia konkretnego połączenia z bazą danych, które będzie używane dla zapytania `exists`. Możesz tego dokonać, dodając nazwę połączenia do nazwy tabeli, używając składni "kropka":

    'email' => 'exists:connection.staff,email'

Jeśli chcesz dostosować zapytanie wykonywane przez regułę sprawdzania poprawności, możesz użyć klasy `Rule` do płynnego zdefiniowania reguły. W tym przykładzie określimy także reguły sprawdzania poprawności jako tablicę, zamiast używać znaku `|` do ograniczenia ich:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>
#### file - plik

W polu sprawdzania poprawności musi znajdować się pomyślnie przesłany plik.

<a name="rule-filled"></a>
#### filled - wypełniony

Pole w trakcie sprawdzania poprawności nie może być puste, gdy jest obecne.

<a name="rule-image"></a>
#### image - obrazek

Plik pod sprawdzaniem poprawności musi być obrazem (jpeg, png, bmp, gif lub svg)

<a name="rule-in"></a>
#### in:_foo_,_bar_,... - do:_bla_,_costam_,...

Pole podlegające walidacji musi być zawarte w podanej liście wartości. Ponieważ ta reguła często wymaga `implozji` tablicy, metoda `Rule::in` może być używana do płynnego konstruowania reguły:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_ - w_tablicy:_kolejnepole_

Pole w walidacji musi istnieć w wartościach _anotherfield_.

<a name="rule-integer"></a>
#### integer - liczba całkowita

Pole w trakcie sprawdzania poprawności musi być liczbą całkowitą.

<a name="rule-ip"></a>
#### ip

Pole w trakcie sprawdzania poprawności musi być adresem IP.

#### ipv4

Pole w trakcie sprawdzania poprawności musi być adresem IPv4.

#### ipv6

Pole w trakcie sprawdzania poprawności musi być adresem IPv6.

<a name="rule-json"></a>
#### json

Sprawdzane pole musi być poprawnym łańcuchem JSON.

<a name="rule-max"></a>
#### max:_value_ - maksymalna:_wartość_

Pole w trakcie sprawdzania poprawności musi być mniejsze lub równe maksymalnej wartości _value_. Łańcuchy, cyfry, tablice i pliki są oceniane w taki sam sposób, jak reguła [`size`](# rule-size).

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

Plik, którego dotyczy sprawdzenie, musi być zgodny z jednym z podanych typów MIME:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

Aby określić typ MIME załadowanego pliku, zawartość pliku zostanie odczytana, a framework podejmie próbę odgadnięcia typu MIME, który może być inny niż typ MIME klienta.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

Plik w ramach sprawdzania poprawności musi mieć typ MIME odpowiadający jednemu z wymienionych rozszerzeń.

#### Basic Usage Of MIME Rule - Podstawowe zastosowanie reguły MIME

    'photo' => 'mimes:jpeg,bmp,png'

Mimo że musisz tylko określić rozszerzenia, ta reguła faktycznie sprawdza poprawność względem typu MIME pliku, odczytując zawartość pliku i zgadując jego typ MIME.

Pełna lista typów MIME i odpowiadających im rozszerzeń znajduje się w następującej lokalizacji: [https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_ - minimum:_wartość_

Pole podlegające walidacji musi mieć minimalną wartość _value_. Łańcuchy, cyfry, tablice i pliki są oceniane w taki sam sposób, jak reguła [`rozmiar`](#rule-size).

<a name="rule-nullable"></a>
#### nullable - pustawe

The field under validation may be `null`. This is particularly useful when validating primitive such as strings and integers that can contain `null` values.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,... - nie_do:_bla_,_cośtam_

Sprawdzane pole nie może być zawarte w podanej liście wartości. Do płynnego konstruowania reguły można użyć metody `Rule::notIn`:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-numeric"></a>
#### numeric - numeryczne

Pole w trakcie sprawdzania poprawności musi być numeryczne.

<a name="rule-present"></a>
#### present - obecne

Pole podlegające walidacji musi być obecne w danych wejściowych, ale może być puste.

<a name="rule-regex"></a>
#### regex:_pattern_ - regex:_wzorzec_

Sprawdzane pole musi pasować do podanego wyrażenia regularnego.

**Uwaga:** W przypadku używania wzorca `regex` może być konieczne określenie reguł w tablicy zamiast używania ograniczników, szczególnie gdy wyrażenie regularne zawiera znak potoku.

<a name="rule-required"></a>
#### required - wymagany

Pole podlegające walidacji musi być obecne w danych wejściowych, a nie puste. Pole jest uważane za "puste", jeśli spełniony jest jeden z następujących warunków:

<div class="content-list" markdown="1">

- Ta wartość to `null`.
- Wartość jest pustym ciągiem.
- Wartością jest pusta tablica lub pustym obiektem "Countable".
- Wartość to przesłany plik bez ścieżki.

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,... - wymagane_jeśli:_innepole_,_wartość_,...

Pole podczas sprawdzanie poprawności musi być obecne i nie może być puste, jeśli pole _anotherfield_ jest równe dowolnej wartości _value_.

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,... - wymagane_chyba_że:_innepole_,_wartość_,...

Pole podczas sprawdzania musi być obecne i nie może być puste, chyba że pole _anotherfield_ jest równe dowolnej _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,... - wymagane_z:_bla_,_cośtam_

Pole w trakcie sprawdzania poprawności musi być obecne i nie może być puste, jeśli obecne są _jakiekolwiek inne_ określone pola.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,... - wymagane_z_wszystkimi:_bla_,_cośtam_

Pole pod sprawdzanie poprawności musi być obecne i nie może być puste, tylko jeśli _obecne są wszystkie_ inne określone pola.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,... - wymagane_bez:_bla_,_cośtam_

Pole w trakcie sprawdzania poprawności musi być obecne, a nie puste, tylko wtedy, _gdy nie ma żadnych innych_ określonych pól.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,... - wymagane_bez_wszystkich:_bla_,_cośtam_

Pole w trakcie sprawdzania poprawności musi być obecne, a nie puste, tylko wtedy, _gdy nie ma wszystkich pozostałych_ określonych pól.

<a name="rule-same"></a>
#### same:_field_

The given _field_ must match the field under validation.
Podane pole _field_ musi pasować do pola w trakcie sprawdzania poprawności.

<a name="rule-size"></a>
#### size:_value_ - rozmiar:_wartość_

Pole w walidacji musi mieć rozmiar zgodny z podaną  _value_. W przypadku danych ciągu wartość _value_ odpowiada liczbie znaków. W przypadku danych liczbowych _value_ odpowiada podanej wartości całkowitej. W przypadku tablicy _size_ odpowiada `count` tablicy. W przypadku plików _size_ odpowiada rozmiarowi pliku w kilobajtach.

<a name="rule-string"></a>
#### string - ciąg znaków

Pole w trakcie sprawdzania poprawności musi być łańcuchem. Jeśli chcesz, aby pole również miało wartość `null`, powinieneś przypisać regułę `nullable` do tego pola.

<a name="rule-timezone"></a>
#### timezone

Pole w trakcie sprawdzania poprawności musi być poprawnym identyfikatorem strefy czasowej zgodnie z funkcją PHP `timezone_identifiers_list`.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_ - unikalne:_tabela_,_kolumna_,

Pole podlegające walidacji musi być unikalne w danej tabeli bazy danych. Jeśli opcja `column` nie zostanie określona, zostanie użyta nazwa pola.

**Określanie niestandardowej nazwy kolumny:**

    'email' => 'unique:users,email_address'

**Niestandardowe połączenie z bazą danych**

Czasami może być konieczne ustawienie niestandardowego połączenia dla zapytań do bazy danych utworzonych przez walidator. Jak pokazano powyżej, ustawienie `unique:users` jako reguły sprawdzania poprawności spowoduje użycie domyślnego połączenia z bazą danych w celu wysłania zapytania do bazy danych. Aby to zmienić, określ połączenie i nazwę tabeli, używając składni "kropka":

    'email' => 'unique:connection.users,email_address'

**Wymuszenie unikalnej reguły ignorowania podanego ID:**

Czasami możesz zignorować dany identyfikator podczas wyjątkowej kontroli. Rozważmy na przykład ekran "profilu aktualizacji", który zawiera nazwę użytkownika, adres e-mail i lokalizację. Oczywiście będziesz chciał sprawdzić, czy adres e-mail jest unikatowy. Jeśli jednak użytkownik zmienia tylko pole nazwy, a nie pole e-mail, nie chcesz, aby błąd sprawdzania poprawności został zgłoszony, ponieważ użytkownik jest już właścicielem adresu e-mail.

To instruct the validator to ignore the user's ID, we'll use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit the rules:
Aby polecić walidatorowi zignorowanie identyfikatora użytkownika, użyjemy klasy `Rule` do płynnego zdefiniowania reguły. W tym przykładzie określimy również reguły sprawdzania poprawności jako tablicę, zamiast używać znaku `|` do rozdzielania reguł:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

Jeśli twoja tabela używa podstawowej nazwy kolumny klucza innej niż `id`, możesz podać nazwę kolumny podczas wywoływania metody `ignore`:

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')

**Dodawanie dodatkowych klauzul gdzie:**

Możesz również określić dodatkowe ograniczenia zapytania, dostosowując zapytanie za pomocą metody `where`. Na przykład dodajmy ograniczenie, które weryfikuje  `account_id` to `1`:

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

Pole w sprawdzaniu poprawności musi być prawidłowym adresem URL.

<a name="conditionally-adding-rules"></a>
## Conditionally Adding Rules - Warunkowo dodawanie reguł

#### Validating When Present - Sprawdzanie poprawności, gdy jest obecny

W niektórych sytuacjach możesz chcieć uruchomić sprawdzanie poprawności względem pola **tylko**, jeśli to pole jest obecne w tablicy wejściowej. Aby szybko to osiągnąć, dodaj regułę `sometimes` do listy reguł:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

W powyższym przykładzie pole `email` będzie sprawdzane tylko wtedy, gdy jest obecne w tablicy `$data`.

> {tip} Jeśli próbujesz zweryfikować pole, które zawsze powinno być obecne, ale może być puste, sprawdź [uwaga na opcjonalne pola](#a-note-on-optional-fields)

#### Complex Conditional Validation - Kompleksowa walidacja warunkowa

Czasami możesz chcieć dodać reguły sprawdzania poprawności w oparciu o bardziej złożoną logikę warunkową. Na przykład możesz chcieć wymagać danego pola tylko wtedy, gdy inne pole ma wartość większą niż 100. Lub możesz potrzebować dwóch pól, aby mieć określoną wartość tylko wtedy, gdy obecne jest inne pole. Dodanie tych reguł sprawdzania poprawności nie musi być uciążliwe. Najpierw utwórz instancję `Validator` ze swoimi _statycznymi regułami_, które nigdy się nie zmienią:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Załóżmy, że nasza aplikacja internetowa jest przeznaczona dla kolekcjonerów gier. Jeśli kolekcjoner gier rejestruje się w naszej aplikacji i posiada ponad 100 gier, chcemy, aby wyjaśnili, dlaczego posiadają tyle gier. Na przykład, może prowadzą sklep odsprzedaży gier, a może po prostu lubią kolekcjonować. Aby warunkowo dodać to wymaganie, możemy użyć metody `sometimes` w instancji` Validator`.

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

Pierwszym argumentem przekazanym do metody `sometimes` jest nazwa pola, które warunkowo sprawdzamy. Drugi argument to reguły, które chcemy dodać. Jeśli `Closure` przekazane jako trzeci argument zwróci `true`, reguły zostaną dodane. Ta metoda sprawia, że szybkie tworzenie złożonych warunkowych sprawdzeń. Możesz nawet dodać warunkowe sprawdzanie poprawności dla kilku pól jednocześnie:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} Parametr `$input` przekazany do `Closure` będzie instancją `Illuminate\Support\Fluent` i może być użyty do uzyskania dostępu do danych wejściowych i plików.

<a name="validating-arrays"></a>
## Validating Arrays - Sprawdzanie poprawności tablic

Sprawdzanie poprawności pól wejściowych formularzy nie musi być uciążliwe. Możesz używać "notacji kropkowej" do sprawdzania poprawności atrybutów w tablicy. Na przykład, jeśli przychodzące żądanie HTTP zawiera pole `photos[profil]`, możesz je sprawdzić w następujący sposób:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

Możesz także sprawdzić każdy element tablicy. Na przykład, aby sprawdzić, czy każda wiadomość e-mail w danym polu wejściowym tablicy jest unikatowa, możesz wykonać następujące czynności:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Podobnie możesz użyć znaku `*` podczas określania komunikatów sprawdzania poprawności w plikach językowych, dzięki czemu korzystanie z pojedynczego komunikatu sprawdzania poprawności dla pól opartych na tablicach będzie proste:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## Custom Validation Rules - Niestandardowe reguły sprawdzania poprawności

<a name="using-rule-objects"></a>
### Using Rule Objects - Korzystanie z obiektów reguł

Laravel oferuje szereg pomocnych reguł walidacji; możesz jednak podać własne. Jedną z metod rejestrowania niestandardowych reguł sprawdzania poprawności jest używanie obiektów reguł. Aby wygenerować nowy obiekt reguły, możesz użyć polecenia `make:rule` Artisan. Użyjmy tego polecenia, aby wygenerować regułę, która weryfikuje, czy łańcuch jest pisany wielkimi literami. Laravel umieści nową regułę w katalogu `app/Rules`:

    php artisan make:rule Uppercase

Po utworzeniu reguły jesteśmy gotowi zdefiniować jej zachowanie. Obiekt reguły zawiera dwie metody: `passses` i` message`. Metoda `passes` przyjmuje wartość atrybutu i nazwę i powinna zwracać wartość `true` lub `false` w zależności od tego, czy wartość atrybutu jest poprawna czy nie. Metoda `message` powinna zwrócić komunikat o błędzie sprawdzania poprawności, który powinien zostać użyty podczas niepowodzenia sprawdzania poprawności:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

Oczywiście, możesz wywołać `trans` helper z twojej metody `message`, jeśli chciałbyś zwrócić komunikat o błędzie z twoich plików tłumaczeniowych:

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

Po zdefiniowaniu reguły możesz dołączyć ją do walidatora, przekazując instancję obiektu reguły wraz z innymi regułami sprawdzania poprawności:

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', new Uppercase],
    ]);

<a name="using-extensions"></a>
### Using Extensions - Używanie rozszerzeń

Inną metodą rejestrowania niestandardowych reguł sprawdzania poprawności jest użycie metody `extend` na elemencie `Validator` [fasada](/docs/{{version}}/facades). Użyjmy tej metody w [dostawcy usług](/docs/{{version}}/providers), aby zarejestrować niestandardową regułę sprawdzania:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
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

Zwyczajna funkcja sprawdzania poprawności otrzyma cztery argumenty: nazwę walidowanego `$attribute`, wartość `$value` atrybutu, tablicę `$parameters` przekazaną do reguły i instancję `Validator`.

Możesz także przekazać klasę i metodę do metody `extend` zamiast do Closure:

    Validator::extend('foo', 'FooValidator@validate');

#### Defining The Error Message - Definiowanie komunikatu o błędzie

Będziesz musiał również zdefiniować komunikat o błędzie dla niestandardowej reguły. Możesz to zrobić za pomocą wbudowanej niestandardowej tablicy komunikatów lub dodając wpis w pliku językowym sprawdzania poprawności. Ta wiadomość powinna znajdować się na pierwszym poziomie tablicy, a nie w tablicy `custom`, która jest tylko dla komunikatów o błędach specyficznych dla atrybutu:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

Podczas tworzenia niestandardowej reguły sprawdzania poprawności czasami może być konieczne zdefiniowanie niestandardowych zamienników zastępczych dla komunikatów o błędach. Możesz to zrobić, tworząc niestandardowy Validator, jak opisano powyżej, a następnie wykonując wywołanie metody `replacer` na fasadzie `Validator`. Możesz to zrobić za pomocą metody `boot` dostawcy [usługodawcy](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### Implicit Extensions - Niejawne rozszerzenia

Domyślnie, gdy sprawdzany atrybut nie jest obecny lub zawiera pustą wartość zdefiniowaną przez regułę [`wymagany`](#rule-required), normalne reguły sprawdzania poprawności, w tym niestandardowe, nie są uruchamiane. Na przykład reguła [`unique`](##rule-unique) nie zostanie uruchomiona dla wartości `null`:

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true

Aby reguła działała nawet wtedy, gdy atrybut jest pusty, reguła musi oznaczać, że atrybut jest wymagany. Aby utworzyć takie "niejawne" rozszerzenie, użyj metody `Validator::extendImplicit()`:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} "Niejawne" rozszerzenie tylko _implies_, że atrybut jest wymagany. To, czy faktycznie unieważnia brakujący lub pusty atrybut, zależy od Ciebie.
