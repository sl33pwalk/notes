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




