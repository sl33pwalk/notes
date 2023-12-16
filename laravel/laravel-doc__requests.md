# HTTP Requests

- класс _Illuminate\Http\Request_ предоставляет объектно-ориентированный способ взаимодействия с текущим HTTP-запросом, обрабатываемым твоим приложением, а также получения входных данных, файлов cookie и файлов, которые были отправлены вместе с запросом.

## Взаимодействие с запросом

Чтобы получить сущность текущего HTTP реквеста через внедрение зависимостей - нужно затайпхинить _Illuminate\Http\Request_ класс в рутовском замыкании либо в методе контроллера. Сущность входящего реквеста автоматически будет внедрена с помощью service container

```php

<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Store the user...

        return redirect('/users');
    }
}

```

```php
<?php

use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});

```

- Если метод вашего контроллера также ожидает входные данные от параметров маршрута, вам следует перечислить параметры маршрута после других зависимостей.

`public function update(Request $request, string $id): RedirectResponse`

## Request Path, Host and Method

### Получение пути реквеста

```php
<?

$uri = $request->path();

```

метод **path** возвращает инфу о пути реквеста.

Если реквест из http://example.com/foo/bar - то path вернет foo/bar

### Inspecting The Request path / route

- метод **is** позволяет проверить, совпадает ли реквест с данным паттерном.

```php
<?

if ($request->is('admin/*')) {
    // ...
}

```

- метод **routeIs** определяет, соответствует ли входящий запрос именованному маршруту

```php
<?

if ($request->routeIs('admin.*')) {
    // ...
}

```

### Получение URL реквеста

```php

<?

$url = $request->url();

$urlWithQueryString = $request->fullUrl();

```

Этот метод объединяет заданный массив переменных строки запроса с текущей строкой запроса:

```php
<?
$request->fullUrlWithQuery(['type' => 'phone']);

```

если хочешь получить текущий URL без даваемых строк запроса:

```php
<?
$request->fullUrlWithoutQuery(['type']);

```

```php
<?

$request->host();
$request->httpHost();
$request->schemeAndHttpHost();

```

- Метод _method_ возвращает HTTP-verb для запроса.

- Можно использовать _isMethod_ для проверки соответствия HTTP-verb заданной строке

```php

<?

$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}

```

## Input

- Можно получить все входные данные входящего запроса в виде массива, используя метод **all**

```php
<?
$input = $request->all();

```

- метод **input** может быть использован для получения инпутов пользователя

```php
<?
$name = $request->input('name');

```

- можно передать значение по умолчанию в качестве второго аргумента метода input. Это значение будет возвращено, если запрашиваемое значение ввода не присутствует в запросе

```php
<?

$name = $request->input('name', 'Sally');

```

- При работе с формами, которые содержат массив инпутов, юзаем 'dot' нотация для получения доступа к массивам:

```php

<?

$name = $request->input('products.0.name');

$names = $request->input('products.*.name');

```

чтобы получить все инпуты, просто вызываем input без аргументов:

```php

<?

$input = $request->input();

```

- В то время как метод input получает значения из всей полезной (?) нагрузки запроса (entire request payload) - включая строку запроса - метод **query** извлекает значения только из query string

```php
<?

$name = $request->query('name');

```

и у него такая же история со вторым аргументом

и такая же история без аргументов

```php
<?

$birthday = $request->date('birthday');

```

### Получение части данных от инпутов

- если нужно получить подмножество input data, можно использовать методы **only** и **except**. Оба метода принимают одинарный массив или динамический лист аргументов

```php
<?

$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');

```

### Определяем, существует ли инпут

```php
<?

if($request->has('name')) {
    // ...
}

```

```php
<?
if ($request->has(['name', 'email'])) {
    // ...
}

```

- метод **hasAny** возвращает true если любое из перечисленных значений существует:

```php

if ($request->hasAny(['name', 'email'])) {
    // ...
}

```

- метод **whenHas** запустит передаваемое замыкание если значение присутствуетв реквесте:

```php
<?
$request->whenHas('name', function (string $input) {
    // ...
});

```

также можно добавить второе замыкание, которое будет рабоать, если значение отсутствует:

```php
<?

$request->whenHas('name', function (string $input) {
    // The "name" value is present...
}, function () {
    // The "name" value is not present...
});

```

- Если вы хотите определить, присутствует ли значение в запросе и не является ли оно пустой строкой, вы можете использовать метод **filled**

```php
<?
if ($request->filled('name')) {
    // ...
}

```

- метод **anyFilled** возвращает true если любое из перечисленных значений не пустая строка

- **whenFilled** та же история что и с whenHas

- Чтобы определить, отсутствует ли данный ключ в запросе, вы можете использовать методы missing и whenMissing:

```php
<?
if ($request->missing('name')) {
    // ...
}

$request->whenMissing('name', function (array $input) {
    // The "name" value is missing...
}, function () {
    // The "name" value is present...
});

```

### Merging Additional Input (объединение доп. ввода)

- Иногда вам может потребоваться вручную объединить дополнительные данные с существующими входными данными запроса. Для этого можно использовать метод **merge** . Если данный ключ ввода уже существует в запросе, он будет заменен данными, предоставленными методу **merge**

```php
<?
$request->merge(['votes' => 0]);

```

- метод **mergeIfMissing** может быть использован для объединения входных данных в запрос, если соответствующие ключи еще не существуют во входных данных запроса:

```php
<?
$request->mergeIfMissing(['votes' => 0]);

```

### Cookies

- Все куки, созданные фреймворком Laravel, зашифрованы и подписаны кодом аутентификации, поэтому они будут считаться недействительными, если были изменены клиентом. Чтобы получить значение cookie из запроса, используйте метод **cookie** на экземпляре _Illuminate\Http\Request_

```php
<?
$value = $request->cookie('name');

```

- По умолчанию Laravel включает middleware App\Http\Middleware\TrimStrings и Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull в глобальный стек middleware приложения. Эти промежуточные программы перечислены в глобальном стеке middleware классом App\Http\Kernel.

- Эти промежуточные программы автоматически обрезают все входящие строковые поля в запросе, а также преобразуют все пустые строковые поля в null. Это позволяет вам не беспокоиться об этих проблемах нормализации в маршрутах и контроллерах.

## Files

- Вы можете получить загруженные файлы из экземпляра Illuminate\Http\Request с помощью метода **file** или используя динамические свойства.

* Метод **file** возвращает экземпляр класса Illuminate\Http\UploadedFile, который расширяет класс PHP SplFileInfo и предоставляет различные методы для работы с файлом:

```php
<?
$file = $request->file('photo');

$file = $request->photo;

```

```php
<?
if ($request->hasFile('photo')) {
    // ...
}

```

- Помимо проверки наличия файла, вы можете убедиться в том, что при загрузке файла не возникло проблем, с помощью метода **isValid**

```php
<?
if ($request->file('photo')->isValid()) {
    // ...
}

```

- Класс _UploadedFile_ также содержит методы для доступа к полному пути файла и его расширению. Метод **extension** попытается определить расширение файла, основываясь на его содержимом. Это расширение может отличаться от расширения, предоставленного клиентом:

```php
<?
$path = $request->photo->path();

$extension = $request->photo->extension();

```

## Storing Uploaded Files

- Класс UploadedFile имеет метод **store**, который отправляет загруженный файл на диск/хост-машину

* Метод **store** принимает путь, по которому файл должен быть сохранен относительно сконфигурированного корневого каталога файловой системы. Этот путь не должен содержать имени файла, так как в качестве имени файла будет автоматически сгенерирован уникальный идентификатор.

```php
<?
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');

```

если не хочешь сам задать имя, **storeAs** в помощь.

```php
<?
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');

```

TODO:

- [ ] метод collect ($request->collect())

- [ ] JSON input

- [ ] Dates

- [ ] Old Input

- [ ] Disabling Input Normalization

- [ ] Configuring Trusted Proxies + Hosts
