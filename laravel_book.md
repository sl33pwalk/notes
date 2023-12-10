# Laravel up & Running

## Глава 3. Маршруты и контроллеры

### Быстрое введение в MVC, HTTP команды и REST


#### Что такое MVC

Принцип работы MVC - **Пользователь** отправляет HTTP запрос **контроллеру**, контроллер, в ответ на запрос, может записать и/или извлечь данные из **модели (бд)**, затем контроллер отправляет данные в **представление**, и затем представление будет возвращено к конечному пользователю как отображение в броузере.

* **Model** - Уникальная Таблица в бд

* **View** - шаблон, который будет выводить данные конечному пользователю

* **Controller** - получает HTTP запрос от броузера, берет нужные данные из бд, производит валидацию инпутов, и, возможно, возвращает ответ обратно пользователю

#### HTTP команды (verbs)

* GET - Request a resource (or a list of resources).

* HEAD - Ask for a headers-only version of the GET response.

* POST - Create a resource.

* PUT - Overwrite a resource.

* PATCH - Modify a resource.

* DELETE - Delete a resource.

* OPTIONS - Ask the server which verbs are allowed at this URL.

Таблица снизу показывает доступные действия для каждого resource контроллера. Каждое действие предполагает от тебя вызов определенного IRL паттерна используя определенную команду, чтобы ты мог понять для чего каждая команда нужна

| Verb      | URL               | Controller method | Name          | Description               |
|-----------|-------------------|-------------------|---------------|---------------------------|
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

* **REST** - архитектурный стиль построения API. 

Основа реста:

+ Быть структурированным вокруг одного основного ресурса за раз (tasks к примеру).

+ Состоять из взаимодействия с предопределенными структурами URL используя HTTP-команды (как в таблице)

+ Возвращать JSON и как можно чаще запрашивать через JSON

* **RESTful** означает "созданные по образцу этих структур на основе URL-адресов, чтобы мы могли выполнять предсказуемые вызовы, такие как GET /tasks/14/edit для страницы редактирования"

create и edit отсутствуют в REST-based APIшках, т.к. API просто преподносят действия, но не страницы, которые готовятся к действиям.


* Можно вместо фасада Route::get использовать $router->get

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

*resourcePlural.action* - общее соглашение о наименовании рутов и представлений.

при route() хелпере можно передать несуществующий параметр, он будет добавлен как параметр запроса

ну тип *ссылка*?opt=a

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

* Часто бывает проще и понятнее прикрепить middleware к маршрутам в контроллере, а не в определении маршрута. Вы можете сделать это, вызвав метод middleware() в конструкторе вашего контроллера. Строка, которую вы передаете методу middleware(), является именем middleware, и вы можете дополнительно подключить методы-модификаторы (only() и except()), чтобы определить, какие методы будут получать этот middleware:

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


* Eloquent - это объектно-реляционный маппер (ORM) ActiveRecord базы данных в Laravel, который позволяет легко связать класс Post (модель) с таблицей базы данных posts и получить все записи с помощью вызова типа Post::all().


* Конструктор запросов - это инструмент, позволяющий выполнять вызовы типа Post::where('active', true)->get() или даже DB::table('users')->all(). Вы *строите* запрос, соединяя методы в цепочку один за другим.

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


#### Subdomain  Routing

* Поддоменная маршрутизация - это то же самое, что и префиксация маршрутов, только вместо префикса маршрута используется поддомен.

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

* Обычно имена маршрутов отражают цепочку наследования элементов пути, поэтому users/comments/5 будет обслуживаться маршрутом с именем users.comments.show. В этом случае принято использовать группу маршрутов вокруг всех маршрутов, которые находятся под ресурсом users.comments.

* Точно так же, как мы можем префиксировать сегменты URL, мы можем префиксировать строки к имени маршрута. С помощью префиксов имен групп маршрутов мы можем определить, что каждый маршрут в этой группе должен иметь заданную строку в качестве префикса к своему имени.

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


* В Laravel вы можете определить "обратный маршрут" (который нужно задать в конце файла маршрутов), чтобы перехватывать все несоответствующие запросы:

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

* Подписанные ссылки состоят из обычной маршрутной ссылки с "подписью", которая доказывает, что URL не был изменен с момента отправки (и, следовательно, что никто не изменил URL для доступа к чужой информации).

* Для построения подписанной ссылки для предоставления доступа к даваемому маршруту, маршрут должен иметь имя: 

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

После генерации ссылки на подписанный маршрут, нужно защитить его от неподписанного доступа. Самый простой вариант - применить *signed* middleware:

```php
Route::get('invitations/{invitation}/{answer}', InvitationController::class)
    ->name('invitations')
    ->middleware('signed');
```

При желании вы можете провести проверку вручную, используя метод hasValidSignature() на объекте Request, вместо того чтобы использовать *signed* middleware: 

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

В Laravel искаробки доступны такие шаблоны: на чистом пхп и *Blade Templates*

Добавляем к файлам .blade чтобы сделать их шаблонами *Blade*

> about.blade.php

* view == View::make() == Illuminate\View\ViewFactory (?)

Этот код ищет представление в *resources/views* и там либо *home.blade.php* либо *home.php*
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

Это замыкание загружает представление и передает ему одну переменную tasks, которая содержит результат метода *Task::all()* 

> *Task::all()* - это Eloquent запрос к базе данных


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

* Контроллеры - это классы, который организуют логику маршрутов в одном месте. 

* Основная задача контроллера заключается в том, чтобы улавливать намерение HTTP-запроса и передавать его остальным частям приложения.

```
php artisan make:controller TaskController
```

Эта команда создаст новый файл TaskController.php в *app/Http/Controllers* с таким содержимым:

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


* Если хочешь создать контроллер ресурсов с автогенерируемыми методами для всех основных маршрутов ресурсов, таких как create() и update(), можешь передать флаг --resource при использовании php artisan make:controller:

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

* *request()->only()* берет ассоциативный массив названия инпутов и возвращает его.

* *Task::create* берет ассоциативный массив и создает новый Task для него.


#### Внедрение зависимостей в контроллеры

* Все методы контроллера (включая конструкторы) разрешаются из контейнера Laravel, что означает, что все, что вы укажете, и что контейнер знает, как разрешить, будет автоматически внедрено.

* Typehinting в PHP означает размещение имени класса или интерфейса перед переменной в сигнатуре метода:

```php
public function __construct(Logger $logger) {}
```
Эта подсказка говорит PHP, что все, что передается в метод, должно быть типа Logger, который может быть как интерфейсом, так и классом.

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

