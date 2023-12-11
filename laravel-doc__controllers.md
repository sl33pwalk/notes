# Controllers

- app/Http/Controllers - директория по умолчанию

```language
php artisan make:controller UserController
```

```php

<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}

```

после создания контроллера задаем рут методу контроллера

```php

use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);

```

> без extends Controller не будут работать middleware и authorize методы

- Если действие контроллера особенно сложное, вам может показаться удобным выделить целый класс контроллера для этого единственного действия. Для этого вы можете определить единственный метод \_\_invoke внутри контроллера

```php

<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     */
    public function __invoke()
    {
        // ...
    }
}

```

если в контроллере только один метод, можно не указывать его в руте

```php

use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);

```

`php artisan make:controller ProvisionServer --invokable`

К контроллерам в руте можно добавить middleware:

```php

Route::get('profile', [UserController::class, 'show'])->middleware('auth');

```

Или можно определить middleware внутри конструктора контроллера.

```php

class UserController extends Controller
{
    /**
     * Instantiate a new controller instance.
     */
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');
    }
}

```

```php

use Closure;
use Illuminate\Http\Request;

$this->middleware(function (Request $request, Closure $next) {
    return $next($request);
});

```

## Resource Controllers

сразу под crud изкаробки:

`php artisan make:controller PhotoController --resource`

```php

use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);

```

```php

Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);

```

| Verb      | URI                  | Action  | Route Name     |
| --------- | -------------------- | ------- | -------------- |
| GET       | /photos              | index   | photos.index   |
| GET       | /photos/create       | create  | photos.create  |
| POST      | /photos              | store   | photos.store   |
| GET       | /photos/{photo}      | show    | photos.show    |
| GET       | /photos/{photo}/edit | edit    | photos.edit    |
| PUT/PATCH | /photos/{photo}      | update  | photos.update  |
| DELETE    | /photos/{photo}      | destroy | photos.destroy |

- Обычно неявный биндинг моделей не позволяет получить модели, которые были soft deleted, и вместо этого возвращает HTTP-ответ 404. Однако можно указать фреймворку разрешить soft deleting моделей, вызвав метод withTrashed при определении маршрута ресурса:

```php

use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();

```

- withTrashed без аргументов разрешает soft deleted models для show, edit и update

- Если ты используешь биндинги модели маршрута и хочешь, чтобы методы контроллера ресурсов указывали на экземпляр модели, можешь использовать опцию --model при генерации контроллера

Бывает нужно сделать вложенные ресуры, тогда:

```php

use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);

```

и этот рут зарегает такой вложенный ресурс:

`/photos/{photo}/comments/{comment}`

### Shallow Nesting

```php

use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();

```

будет тогда следующее:

| Verb      | URI                             | Action  | Route Name             |
| --------- | ------------------------------- | ------- | ---------------------- |
| GET       | /photos/{photo}/comments        | index   | photos.comments.index  |
| GET       | /photos/{photo}/comments/create | create  | photos.comments.create |
| POST      | /photos/{photo}/comments        | store   | photos.comments.store  |
| GET       | /comments/{comment}             | show    | comments.show          |
| GET       | /comments/{comment}/edit        | edit    | comments.edit          |
| PUT/PATCH | /comments/{comment}             | update  | comments.update        |
| DELETE    | /comments/{comment}             | destroy | comments.destroy       |

все маршруты, которые не Route::resource (get/post/...) должны быть определены до ресурса

- Иногда в вашем приложении есть ресурсы, которые могут иметь только один экземпляр. Например, "профиль" пользователя можно редактировать или обновлять, но у пользователя не может быть более одного "профиля". Такие ресурсы называются "singletone resources", то есть может существовать один и только один экземпляр ресурса. В этих сценариях можно зарегистрировать контроллер ресурса "singleton".

```php

use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);

```

| Verb      | URI           | Action | Route Name     |
| --------- | ------------- | ------ | -------------- |
| GET       | /profile      | show   | profile.show   |
| GET       | /profile/edit | edit   | profile.edit   |
| PUT/PATCH | /profile      | update | profile.update |

- singletone ресурсы могут быть вложены внутрь стандартного ресурса:

```php

Route::singleton('photos.thumbnail', ThumbnailController::class);

```

В этом примере ресурс photos получит все стандартные маршруты ресурсов, однако ресурс thumbnail будет единственным ресурсом

| Verb      | URI                            | Action | Route Name              |
| --------- | ------------------------------ | ------ | ----------------------- |
| GET       | /photos/{photo}/thumbnail      | show   | photos.thumbnail.show   |
| GET       | /photos/{photo}/thumbnail/edit | edit   | photos.thumbnail.edit   |
| PUT/PATCH | /photos/{photo}/thumbnail      | update | photos.thumbnail.update |

На потом:

- Customizing Missing Model Behavior []

- Generating Form Requests []

- Partial Resource Routes []

- Naming Resource []

- Добить Singletone []

- Dependency Injection []
