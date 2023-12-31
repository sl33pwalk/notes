# Laravel up & Running

## Глава 3. Маршруты и контроллеры

### Быстрое введение в MVC, HTTP команды и REST

#### Что такое MVC

Принцип работы MVC - **Пользователь** отправляет HTTP запрос **контроллеру**, контроллер, в ответ на запрос, может записать и/или извлечь данные из **модели (бд)**, затем контроллер отправляет данные в **представление**, и затем представление будет возвращено к конечному пользователю как отображение в броузере.

- **Model** - Уникальная Таблица в бд

- **View** - шаблон, который будет выводить данные конечному пользователю

- **Controller** - получает HTTP запрос от броузера, берет нужные данные из бд, производит валидацию инпутов, и, возможно, возвращает ответ обратно пользователю

#### HTTP команды (verbs)

- GET - Request a resource (or a list of resources).

- HEAD - Ask for a headers-only version of the GET response.

- POST - Create a resource.

- PUT - Overwrite a resource.

- PATCH - Modify a resource.

- DELETE - Delete a resource.

- OPTIONS - Ask the server which verbs are allowed at this URL.

Таблица снизу показывает доступные действия для каждого resource контроллера. Каждое действие предполагает от тебя вызов определенного IRL паттерна используя определенную команду, чтобы ты мог понять для чего каждая команда нужна

| Verb      | URL               | Controller method | Name          | Description               |
| --------- | ----------------- | ----------------- | ------------- | ------------------------- |
| GET       | tasks             | index()           | tasks.index   | Show all tasks            |
| GET       | tasks/create      | create()          | tasks.create  | Show the create task form |
| POST      | tasks             | store()           | tasks.store   | Accept form submission    |
|           |                   |                   |               | from the create task form |
| GET       | tasks/{task}      | show()            | tasks.show    | Show one task             |
| GET       | tasks/{task}/edit | edit()            | tasks.edit    | Edit one task             |
| PUT/PATCH | tasks/{task}      | update()          | tasks.update  | Accept form submission    |
|           |                   |                   |               | from the edit task form   |
| DELETE    | tasks/{task}      | destroy()         | tasks.destroy | Delete one task           |

#### Что такое REST

- **REST** - архитектурный стиль построения API.

Основа реста:

- Быть структурированным вокруг одного основного ресурса за раз (tasks к примеру).

- Состоять из взаимодействия с предопределенными структурами URL используя HTTP-команды (как в таблице)

- Возвращать JSON и как можно чаще запрашивать через JSON

* **RESTful** означает "созданные по образцу этих структур на основе URL-адресов, чтобы мы могли выполнять предсказуемые вызовы, такие как GET /tasks/14/edit для страницы редактирования"

create и edit отсутствуют в REST-based APIшках, т.к. API просто преподносят действия, но не страницы, которые готовятся к действиям.

- Можно вместо фасада Route::get использовать $router->get

```php
Route::get('/', function () {
    return view('welcome');
});

Route::get('about', function () {
    return view('about');
});

Route::get('products', function () {
    return view('products');
});

Route::get('services', function () {
    return view('services');
});
```

```php
$route->get('/', function () {
    return view('welcome')
})
```

другие виды рутов:

```
Route::get('/', function () {
    return 'Hello, World!';
});

Route::post('/', function () {
    // Handle someone sending a POST request to this route
});

Route::put('/', function () {
    // Handle someone sending a PUT request to this route
});

Route::delete('/', function () {
    // Handle someone sending a DELETE request to this route
});

Route::any('/', function () {
    // Handle any verb request to this route
});

Route::match(['get', 'post'], '/', function () {
    // Handle GET or POST requests to this route
});
```

Также вместо замыкания можно передать контроллер и метод в нем

```php
use App\Http\Controllers\WelcomeController;

Route::get('/', [WelcomeController::class, 'index']);
```

В Laravel есть соглашение по поводу ссылок на определенный метод в контроллере: [ControllerName::class, methodName], называемое кортежным синтаксисом / синтаксисом вызываемого массива.

#### Route Parameters

Route::get('users/{id?}/friends', function ($id = 'fallbackId') {
//
});

Регулярки:

```php
Route::get('posts/{id}.{slug}', function ($id, $slug) {
    //
})->where(['id' => '[0-9]+', 'slug' => '[A-Za-z]+']);
```

Также в Laravel существуют удобные методы для регулярок, по типу:

```php
Route::get('users/{id}/friends/{friendname}', function ($id, $friendname) {
    //
})->whereNumber('id')->whereAlpha('friendname');

Route::get('users/{name}', function ($name) {
    //
})->whereAlphaNumeric('name');

Route::get('users/{id}', function ($id) {
    //
})->whereUuid('id');

Route::get('users/{id}', function ($id) {
    //
})->whereUlid('id');

Route::get('friends/types/{type}', function ($type) {
    //
})->whereIn('type', ['acquaintance', 'bestie', 'frenemy']);
```

#### Названия маршрутов

url() global helper упрощает обращение к представлениям, если это нужно.

```php
<a href="<?php echo url('/'); ?>">
// Outputs <a href="http://myapp.com/">
```

Laravel позволяет дать имя каждому руту

```php
Route::get('members/{id}', [\App\Http\Controller\MemberController::class, 'show'])->name('members.show');
```

```php
<a href="<?php echo route('members.show', ['id' => 14]); ?>"
```

если у рута нет параметров, можно просто передать route('members.index')

_resourcePlural.action_ - общее соглашение о наименовании рутов и представлений.

при route() хелпере можно передать несуществующий параметр, он будет добавлен как параметр запроса

ну тип _ссылка_?opt=a

### Route Groups

Группы маршрутов позволяют снизить количество дубликатов, группируя несколько рутов вместе и применяя любые общие настройки конфигурации один раз для всей группы.

Для группировки двух и более рутов вместе, мы "окружаем" определения рута рут группой

```php
Route::group(function () {
    Route::get('hello', function () {
        return 'Hello';
    });
    Route::get('world', function () {
        return 'World';
    });
});
```

На самом деле ты передаешь замыкание в определение группы и определяешь сгруппированные маршруты внутри этого замыкания.

По дефолту, разницы нет, используешь ты рут группы, или так просто раздельно route::get используешь.

#### Middleware

Вероятно наиболее распространенным применением групп маршрутов является применение middleware к группе маршрутов.

Middleware Laravel использует, к примеру, для аутентификации пользователей и запрета гостям к некоторым местам сайта.

В этом примере мы создаем группу рутов с представлениями dashboard и account и применяем auth middleware для обоих. Тут уже более понятно, зачем нужны группы рутов.

```php
Route::middleware('auth')->group(function() {
    Route::get('dashboard', function () {
        return view('dashboard');
    });
    Route::get('account', function () {
        return view('account');
    });
});
```

То есть в этом примере пользователям нужно быть авторизованными, чтобы получить доступ к account и dashboard

- Часто бывает проще и понятнее прикрепить middleware к маршрутам в контроллере, а не в определении маршрута. Вы можете сделать это, вызвав метод middleware() в конструкторе вашего контроллера. Строка, которую вы передаете методу middleware(), является именем middleware, и вы можете дополнительно подключить методы-модификаторы (only() и except()), чтобы определить, какие методы будут получать этот middleware:

```php
class DashboardController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('admin-auth')
            ->only('editUsers');

        $this->middleware('team-member')
            ->except('editUsers');
    }
}
```

- Eloquent - это объектно-реляционный маппер (ORM) ActiveRecord базы данных в Laravel, который позволяет легко связать класс Post (модель) с таблицей базы данных posts и получить все записи с помощью вызова типа Post::all().

- Конструктор запросов - это инструмент, позволяющий выполнять вызовы типа Post::where('active', true)->get() или даже DB::table('users')->all(). Вы _строите_ запрос, соединяя методы в цепочку один за другим.

#### Path Prefixes

Если у вас есть группа маршрутов, которые имеют общий сегмент пути - например, если приборная панель вашего сайта имеет префикс /dashboard - вы можете использовать группы маршрутов, чтобы упростить эту структуру

```php
Route::prefix('dashboard')->group(function () {
    Route::get('/', function () {
        // Handles the path /dashboard
    });
    Route::get('users', function () {
        // Handles the path /dashboard/users
    });
});
```

Заметь, что каждая префиксная группа также имеет / рут, который подразумевает /название (/dashboard в примере)

#### Subdomain Routing

- Поддоменная маршрутизация - это то же самое, что и префиксация маршрутов, только вместо префикса маршрута используется поддомен.

Для этого есть два основных способа использования.

1. Вы можете захотеть представить разные разделы приложения (или совершенно разные приложения) на разных поддоменах:

```php
Route::domain('api.myapp.com')->group(function () {
    Route::get('/', function () {
        //
    });
});
```

2. Вы можете задать часть поддомена в качестве параметра. Чаще всего это делается в случаях мультитенантности (вспомните Slack или Harvest, где каждая компания получает свой собственный поддомен, например tighten.slack.co).

```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('/', function ($account) {
        //
    });
    Route::get('users/{id}', function ($account, $id) {
        //
    });
});
```

Заметь, что любые параметры для группы проходят через сгруппированные методы рута как первый параметр

#### Name Prefixes

- Обычно имена маршрутов отражают цепочку наследования элементов пути, поэтому users/comments/5 будет обслуживаться маршрутом с именем users.comments.show. В этом случае принято использовать группу маршрутов вокруг всех маршрутов, которые находятся под ресурсом users.comments.

- Точно так же, как мы можем префиксировать сегменты URL, мы можем префиксировать строки к имени маршрута. С помощью префиксов имен групп маршрутов мы можем определить, что каждый маршрут в этой группе должен иметь заданную строку в качестве префикса к своему имени.

```php
Route::name('users.')->prefix('users')->group(function () {
    Route::name('comments.')->prefix('comments')->group(function () {
        Route::get('{id}', function () {
            // ...
        })->name('show'); // Route named 'users.comments.show'

        Route::destroy('{id}', function () {})->name('destroy');
    });
});
```

Мы префиксим 'users.' для каждого имени маршрута, а затем 'comments.'

Можно сгруппировать руты, которых крышует один и тот же контроллер

```php
use App\Http\Controllers\UserController;

Route::controller(UserController::class)->group(function () {
    Route::get('/', 'index');
    Route::get('{id}', 'show');
});
```

- В Laravel вы можете определить "обратный маршрут" (который нужно задать в конце файла маршрутов), чтобы перехватывать все несоответствующие запросы:

```php
Route::fallback(function () {
    //
});
```

### Signed Routes

Множество приложений постоянно отправляют уведомления о каких-то действиях (ресетнуть пароль, принять приглашение и прочее), и предоставляют простую ссылку для принятия этих действий.

> Представим, что вы отправляете письмо с подтверждением того, что получатель хочет быть добавлен в список рассылки.

> Есть три способа отправить эту ссылку

> 1. Сделать этот URL общедоступным и надеяться, что никто другой не обнаружит URL одобрения или не изменит свой собственный URL одобрения, чтобы одобрить кого-то другого.

> 2. Поместить действие за аутентификацию, поставить ссылку на действие и потребовать от пользователя войти в систему, если он еще не вошел (что в данном случае может быть невозможно, так как многие получатели рассылки, скорее всего, не являются пользователями с учетными записями).

> 3. "Подпишите" ссылку так, чтобы она однозначно подтверждала, что пользователь получил ссылку из вашего письма, без необходимости входить в систему - что-то вроде http://myapp.com/invitations/5816/yes?signature=030ab0ef6a8237bd86a8b8.

Простой путь для совершения последней опции - использовать фичу, называемую **подписанные ссылки** (signed URLs), благодаря которым можно легко построить систему аутентификации подписи для расылки аутентифицированных ссылок.

- Подписанные ссылки состоят из обычной маршрутной ссылки с "подписью", которая доказывает, что URL не был изменен с момента отправки (и, следовательно, что никто не изменил URL для доступа к чужой информации).

- Для построения подписанной ссылки для предоставления доступа к даваемому маршруту, маршрут должен иметь имя:

```php
Route::get('invitations/{invitation}/{answer}', InvitationController::class)->name('invitations');
```

Для создания обычной ссылки на этот маршрут можно использовать помощник route()

Но также можно использовать URL-фасад для того, чтобы сделать то же самое: URL::route('invitations', ['invitation' => 12345, 'answer' => 'yes']).

Чтобы сгенерировать подписанную ссылку на этот маршрут, просто используйте метод signedRoute().

А если хочешь сгенерировать подписанный маршрут с истекающим сроком действия, используй метод temporarySignedRoute():

```php
// Generate a normal link
URL::route('invitations', ['invitation' => 12345, 'answer' => 'yes']);

// Generate a signed link
URL::signedRoute('invitations', ['invitation' => 12345, 'answer' => 'yes']);

// Generate an expiring (temporary) signed link
URL::temporarySignedRoute(
    'invitations',
    now()->addHours(4),
    ['invitation' => 12345, 'answer' => 'yes']
);
```

now() хелпер идентичен Carbon::now(), который возвращает сегодняшнее время, секунду в секунду. Carbon это библиотека даты/времени, включенная в Laravel.

#### Modifying Routes to Allow Signed Links (Изменение маршрутов для разрешения подписанных ссылок)

После генерации ссылки на подписанный маршрут, нужно защитить его от неподписанного доступа. Самый простой вариант - применить _signed_ middleware:

```php
Route::get('invitations/{invitation}/{answer}', InvitationController::class)
    ->name('invitations')
    ->middleware('signed');
```

При желании вы можете провести проверку вручную, используя метод hasValidSignature() на объекте Request, вместо того чтобы использовать _signed_ middleware:

```php
class InvitationController
{
    public function __invoke(Invitation $invitation, $answer, Request $request)
    {
        if (! $request->hasValidSignature()) {
            abort(403);
        }

        //
    }
}
```

### Views (Представления)

В Laravel искаробки доступны такие шаблоны: на чистом пхп и _Blade Templates_

Добавляем к файлам .blade чтобы сделать их шаблонами _Blade_

> about.blade.php

- view == View::make() == Illuminate\View\ViewFactory (?)

Этот код ищет представление в _resources/views_ и там либо _home.blade.php_ либо _home.php_

```php
Route::get('/', function () {
   return view('home');
});
```

Передача переменных в представления:

```php
Route::get('tasks', function () {
    return view('tasks.index')
        ->with('tasks', Task::all());
});
```

Это замыкание загружает представление и передает ему одну переменную tasks, которая содержит результат метода _Task::all()_

> _Task::all()_ - это Eloquent запрос к базе данных

#### Route::view() маршрут:

```php
// Returns resources/views/welcome.blade.php
Route::view('/', 'welcome');

// Passing simple data to Route::view()
Route::view('/', 'welcome', ['User' => 'Michael']);
```

#### Использование композиторов представлений для обмена переменными с каждым представлением

Могут существовать переменные, которые нужно сделать доступными для всех представлений / для определенного класса представлений и прочего

Можно поделиться определенными переменными с каждым/определенным шаблоном следующим образом:

```php
view()->share('variableName', 'variableValue');
```

Подробности в следующей жизни

### Контроллеры

- Контроллеры - это классы, который организуют логику маршрутов в одном месте.

- Основная задача контроллера заключается в том, чтобы улавливать намерение HTTP-запроса и передавать его остальным частям приложения.

```
php artisan make:controller TaskController
```

Эта команда создаст новый файл TaskController.php в _app/Http/Controllers_ с таким содержимым:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TaskController extends Controller
{
    //
}
```

```php
// TaskController.php
...
public function index()
{
    return view('tasks.index')
        ->with('tasks', Task::all());
}
```

Этот метод контроллера загружает представление и передает ему единственную переменную tasks, которая содержит результат работы метода Task::all() Eloquent.

- Если хочешь создать контроллер ресурсов с автогенерируемыми методами для всех основных маршрутов ресурсов, таких как create() и update(), можешь передать флаг --resource при использовании php artisan make:controller:

```
php artisan make:controller TaskController --resourcre
```

#### Getting user input

1. Binding basic form actions (биндим базовые действия формы)

```php
// routes/web.php
Route::get('tasks/create', [TaskController::class, 'create']);
Route::post('tasks', [TaskController::class, 'store']);
```

2. Common form input controller method (метод контроллера ввода в форму)

```php
// TaskController.php
...
public function store()
{
    Task::create(request()->only(['title', 'description']));

    return redirect('tasks');
}
```

Этот пример использует Eloquent модели и функцию redirect()

Мы используем помощник request() для представления HTTP-запроса и используем его метод only() для получения только полей заголовка и описания, отправленных пользователем.

Затем мы передаем эти данные в метод create() нашей модели Task, который создает новый экземпляр Task с названием, установленным в переданное название, и описанием, установленным в переданное описание.

Наконец, мы перенаправляемся обратно на страницу, где отображаются все задачи

> метод only() берет данные оттуда же, откуда методы all() и get(), и тут пох Get или POST, всё равно данные будут. Я так понял.

- _request()->only()_ берет ассоциативный массив названия инпутов и возвращает его.

- _Task::create_ берет ассоциативный массив и создает новый Task для него.

#### Внедрение зависимостей в контроллеры

- Все методы контроллера (включая конструкторы) разрешаются из контейнера Laravel, что означает, что все, что вы укажете, и что контейнер знает, как разрешить, будет автоматически внедрено.

- Typehinting в PHP означает размещение имени класса или интерфейса перед переменной в сигнатуре метода:

```php
public function __construct(Logger $logger) {}
```

Этот тайпхинт говорит PHP, что все, что передается в метод, должно быть типа Logger, который может быть как интерфейсом, так и классом.

Controller method injection via typehinting

```php
// TaskController.php
...
public function store(\Illuminate\Http\Request $request)
{
    Task::create($request->only(['title', 'description']));

    return redirect('tasks');
}
```

тут мы вместо использования глобальных хелперов тайпхинтим сущность $request

`php artisan route:list` - список всех доступных маршрутов

`php artisan route:list --except-vendor`

```php
<?
Route::get('conferences/{id}', function ($id) {
    $conference = Conference::findOrFail($id);
});

```

api resource controller:
`php artisan make:controller MySampleResourceController --api`

> у апи нет edit и create т.к. апи просто преподносит действия!

#### Single Action контроллеры (контроллеры единственного действия)

```php
<?

// \App\Http\Controllers\UpdateUserAvatar.php
public function __invoke(User $user)
{
    // Update the user's avatar image
}

// routes/web.php
Route::post('users/{user}/update-avatar', UpdateUserAvatar::class);

```

### Route Model Binding (привязка/связывание модели маршрута)

- Один из самых распространенных шаблонов маршрутизации заключается в том, что первая строка любого метода контроллера пытается найти ресурс с заданным идентификатором

```php
<?
Route::get('conferences/{id}', function ($id) {
    $conference = Conference::findOrFail($id);
});

```

- Laravel предоставляет функцию, которая упрощает этот шаблон, называемую **привязкой модели маршрута**.

- Это позволяет определить, что определенное имя параметра (например, {conference}) будет указывать распознавателю маршрута, что он должен найти запись в базе данных Eloquent с этим идентификатором и затем передать ее в качестве параметра вместо того, чтобы просто передать идентификатор.

Существует два вида привязки модели маршрута: неявная и пользовательская (или явная). (implicit and explicit)

#### Implicit Route Model Binding (неявная привязка модели маршрута)

- Назови параметр маршрута чем-то уникальным для данной модели, затем затайпхинти этот параметр в методе замыкания/контроллера и используй там то же имя переменной.

```php
<?
Route::get('conferences/{conference}', function (Conference $conference) {
    return view('conferences.show')->with('conference', $conference);
});

```

- Поскольку параметр маршрута _({conference})_ совпадает с параметром метода _($conference)_, а параметр метода связан с моделью _Conference (Conference $conference)_, Laravel воспринимает это как привязку модели маршрута.

- При каждом посещении этого маршрута приложение будет считать, что все, что передается в URL вместо {conference}, является идентификатором, который следует использовать для поиска конференции, и затем этот результирующий экземпляр модели будет передан в метод закрытия или контроллера.

Если есть два динамических сегмента в URL (organizers/{organizer}/conferences/{conference:slug), Laravel автоматически попытается ограничить запросы второй модели только теми, которые связаны с первой.

То есть он проверит модель Organizer на наличие связи с конференциями и, если она существует, вернет только те конференции, которые связаны с организатором, найденным в первом сегменте.

```php
<?

use App\Models\Conference;
use App\Models\Organizer;

Route::get(
    'organizers/{organizer}/conferences/{conference:slug}',
    function (Organizer $organizer, Conference $conference) {
        return $conference;
    });

```

#### Custom Route Model Binding (Явная привязка модели маршрута)

добавляем в метод _boot()_, который находится в _App\Providers\RouteServiceProvider_:

```php
<?
    public function boot()
    {
        // Perform the binding
        Route::model('event', Conference::class);
    }
```

теперь всегда, когда в руте есть параметр {event}, route resolver вернет сущность класса Conference с ID этого URL параметра.

```php
<?

Route::get('events/{event}', function (Conference $event) {
    return view('events.show')->with('event', $event);
});

```

### Route Caching (Кеширование Маршрута)

`php artisan route:clear`

`php artisan route:cache` - routes/\* файлы будут кэшированы

### Form Method Spoofing (Подмена Метода формы)

```php
<?
<form action="/tasks/5" method="POST">
    <input type="hidden" name="_method" value="DELETE">
    <!-- or: -->
    @method('DELETE')
</form>
```

ну то есть с помощью @method можно указать HTTP-глагол

### CSRF Protection

```php
<?

<form action="/tasks/5" method="POST">
    @csrf
</form>

```

- По умолчанию все маршруты в Laravel, кроме маршрутов "только для чтения" (использующих GET, HEAD или OPTIONS), защищены от атак подделки межсайтовых запросов (CSRF), требуя передачи токена в виде входного поля с именем \_token вместе с каждым запросом.

Этот токен генерируется в начале каждой сессии, и каждый маршрут, не предназначенный только для чтения, сравнивает переданный \_token с токеном сессии

- Cross-site request forget (подделка межсайтовых запросов) - это когда один вебсайт притворяется другим.

* Цель состоит в том, чтобы перехватить доступ пользователей к вашему сайту, отправляя формы со своего сайта на ваш сайт через браузер вошедшего пользователя.

* Лучшим способом защиты от CSRF-атак является защита всех входящих маршрутов - POST, DELETE и т. д. - с помощью токена, что Laravel делает из коробки.

### Redirects (Перенаправления)

2 способа создать redirect

1. вызвать как helper redirect()

2. Вызвать как фасад

Оба создают сущность Illuminate\Http\RedirectResponse, чето там делают и возвращают её.

```php
<?

// Using the global helper to generate a redirect response
Route::get('redirect-with-helper', function () {
    return redirect()->to('login');
});

// Using the global helper shortcut
Route::get('redirect-with-helper-shortcut', function () {
    return redirect('login');
});

// Using the facade to generate a redirect response
Route::get('redirect-with-facade', function () {
    return Redirect::to('login');
});

// Using the Route::redirect shortcut
Route::redirect('redirect-by-route', 'login');

```

- Если вы передаете параметры непосредственно в помощник, а не выстраиваете цепочку методов после него, то это сокращение метода перенаправления to() (?)

##### redirect()->to()

```php
<?

function to($to = null, $status = 302, $headers = [], $secure = null)

```

- $to - это валидный внутренний путь

- $status - это статус HTTP (по умолчанию 302)

- $headers позволяет определить, какие HTTP-заголовки отправлять вместе с перенаправлением

- $secure позволяет переопределить выбор по умолчанию между http и https (который обычно устанавливается на основе текущего URL-адреса запроса).

```php
<?
Route::get('redirect', function () {
    return redirect()->to('home');

    // Or same, using the shortcut:
    return redirect('home');
});
```

##### reditect()->route()

```php
<?
Route::get('redirect', function () {
    return redirect()->route('conferences.index');
});
```

route() метод похож на to(), но вместо указывания определенного пути, он указывает на конкретное имя маршрута.

из-за того, что некоторым рутам нужны параметры, route() имеет необязательный второй параметр для этого случая:

```php
<?
function route($to = null, $parameters = [], $status = 302, $headers = [])
```

```php
<?
Route::get('redirect', function () {
    return to_route('conferences.show', [
        'conference' => 99,
    ];
});
```

- Можно использовать helper to_route() в качестве псевдонима для метода redirect()->route()

##### redirect()->back()

редиректит на прошлую страницу, на которой был пользователь до этой страницы

##### redirect()->with()

- with() отличается определяет не то, куда вы перенаправляете, а то, _какие данные вы передаете_ вместе с перенаправлением.

```php
<?
Route::get('redirect-with-key-value', function () {
    return redirect('dashboard')
        ->with('error', true);
});

Route::get('redirect-with-array', function () {
    return redirect('dashboard')
        ->with(['error' => true, 'message' => 'Whoops!']);
});
```

- Также можно использовать функцию **withInput()** для перенаправления с отображением введенной пользователем формы.

эт когда хочеть отправить пользователя обратно к форме, от которой он пришел. Допустим, валидация выдала ошибки

```php
<?
Route::get('form', function () {
    return view('form');
});

Route::post('form', function () {
    return redirect('form')
        ->withInput()
        ->with(['error' => true, 'message' => 'Whoops!']);
});
```

- Самый простой способ получить данные, переданные с помощью функции withInput(), - использовать помощник old(), с помощью которого можно получить все старые данные (old()) или только значение для определенного ключа, как показано в следующем примере, причем второй параметр используется по умолчанию, если старого значения нет.

Обычно это можно увидеть в представлениях, что позволяет использовать этот HTML как в представлении "создание", так и в представлении "редактирование" для этой формы:

```php
<?
<input name="username" value="<?=
    old('username', 'Default username instructions here');
?>">
```

- Также существует метод для передачи ошибок вместе с ответом редиректа: withErrors().

Вы можете передать ему любого "провайдера" ошибок, который может быть строкой ошибок, массивом ошибок или, чаще всего, экземпляром валидатора Illuminate.

```php
<?
Route::post('form', function (Illuminate\Http\Request $request) {
    $validator = Validator::make($request->all(), $this->validationRules);

    if ($validator->fails()) {
        return back()
            ->withErrors($validator)
            ->withInput();
    }
});
```

Функция withErrors() автоматически разделяет переменную $errors с представлениями страницы, на которую она перенаправляет, чтобы вы могли обработать ее по своему усмотрению.

### Aborting the Request (Прерывание/Отмена Запроса)

- Помимо возврата представлений и перенаправления (view and redirects), наиболее распространенным способом завершения маршрута является **прерывание**.

Существует несколько глобально доступных методов (abort(), abort_if() и abort_unless()), которые опционально принимают в качестве параметров коды состояния HTTP, сообщение и массив заголовков

```php
<?
Route::post('something-you-cant-do', function (Illuminate\Http\Request $request) {
    abort(403, 'You cannot do that!');
    abort_unless($request->has('magicToken'), 403);
    abort_if($request->user()->isBanned, 403);
});
```

### Custom Responses (Пользовательские Ответы)

- response()->make()

Если вы хотите создать HTTP-ответ вручную, просто передайте свои данные в первый параметр response()->make(): например, return response()->make(Hello, World!). И снова, второй параметр - это код состояния HTTP, а третий - ваши заголовки.

- response()->json and ->jsonp()

### Testing

- В Laravel типично полагаться на тестирование приложений для проверки функциональности маршрутов

Простой тест маршрута POST

```php
<?
// tests/Feature/AssignmentTest.php
public function test_post_creates_new_assignment()
{
    $this->post('/assignments', [
        'title' => 'My great assignment',
    ]);

    $this->assertDatabaseHas('assignments', [
        'title' => 'My great assignment',
    ]);
}
```

Простой тест маршрута GET

```php
<?
// AssignmentTest.php
public function test_list_page_shows_all_assignments()
{
    $assignment = Assignment::create([
        'title' => 'My great assignment',
    ]);

    $this->get('/assignments')
        ->assertSee('My great assignment');
}
```
