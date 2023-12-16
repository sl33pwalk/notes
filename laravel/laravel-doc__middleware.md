# Middleware

- Middleware предоставляет удобный механизм для проверки и фильтрации HTTP-запросов, поступающих в ваше приложение

Создать новый Middleware

```
php artisan make:middleware EnsureTokenIsValid
```

эта команда поместит EnsureTokenIsValid в _app/Http/Middleware_

В этом мидлваре, мы даем доступ руту только если полученный _token_ из инпута совпадает с определенным значением, иначе директим в home URI

```php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('home');
        }

        return $next($request);
    }
}

```

- Лучше всего представлять middleware как ряд "уровней", через которые должны пройти HTTP-запросы, прежде чем они попадут в приложение. Каждый уровень может изучить запрос и даже полностью отклонить его

- Middleware может выполнять задачи до или после передачи запроса в приложение.

## Assigning Middleware To Routes (Назначение middleware рутам)

```php
use App\Http\Middleware\Authenticate;

Route::get('/profile', function () {
    // ...
})->middleware(Authenticate::class);
```

Можно назначить несколько middleware руту, передав массив middleware имён методу middleware:

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

```php
// Within App\Http\Kernel class...

protected $middlewareAliases = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
];
```

```php
Route::get('/profile', function () {
    // ...
})->middleware('auth');
```

Предотвратить middleware:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

## Middleware groups

можно сгруппировать несколько middleware под один ключ для более легкого вызова. Делается это с помощью свойства **$middlewareGroups** в HTTP kernel (ядре)

в Laravel предопределены web и api middleware группы.

```php

/**
 * The application's route middleware groups.
 *
 * @var array
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];

```

```php

Route::get('/', function () {
    // ...
})->middleware('web');

Route::middleware(['web'])->group(function () {
    // ...
});

```

На потом:

- Sorting Middleware ($middlewarePriority)

- Middleware Parameters

- Terminable Middleware
