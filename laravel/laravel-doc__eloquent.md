# Eloquent: Getting Started

- Eloquent - обхектно-реляционный mapper (ORM).

* При использовании Eloquent каждая таблица базы данных имеет соответствующую "модель", которая используется для взаимодействия с этой таблицей. Помимо получения записей из таблицы базы данных, модели Eloquent позволяют вставлять, обновлять и удалять записи из таблицы

`php artisan make:model Flight

Модели обычно живут в _app\Models_ и расширяют класс _Illuminate\Database\Eloquent\Model_

`php artisan model:show Flight`

- По соглашению, в качестве имени таблицы будет использоваться "snake case", множественное имя класса, если явно не указано другое имя. Так, в данном случае Eloquent предположит, что модель Flight хранит записи в таблице flights, а модель AirTrafficController будет хранить записи в таблице air_traffic_controllers.

если че можно просто въебать `protected $table = 'my_flights';`

```php
<?
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUuids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Europe']);

$article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"

```

- если не хочешь timestamps:

`public $timestamps = false;`

$dateFormat свойство для изменения timestamps

```php
<?

<?php

class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'updated_date';
}

```

- Если вы хотите определить значения по умолчанию для некоторых атрибутов вашей модели, вы можете определить свойство **\$attributes** в вашей модели. Значения атрибутов, помещенные в массив **\$attributes**, должны быть в их необработанном, "хранимом" формате, как если бы они были только что считаны из базы данных:

```php
<?

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The model's default values for attributes.
     *
     * @var array
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
}

```

## Получение моделей

- Метод модели **all** получит все записи из связанной с моделью таблицы базы данных.

```php
<?
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}

```

### Построение запросов

- Каждая Eloquent модель служит как query builder

- Можно добавить запросам дополнительные ограничения и затем вызвать метод **get** для получения результата:

```php
<?
$flights = Flight::where('active', 1)
               ->orderBy('name')
               ->take(10)
               ->get();
```

### Refreshing Models

- если уже имеется сущность Eloquent модели, которая была получена из бд, то ее (модель) можно обновить с помощью методов **fresh** и **refresh**.

- Метод **fresh** переполучит модель из бд. Уже существующая сущность модели не будет затронута:

```php
<?
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

- Метод **refresh** заново наполняет существующую модель свежими данными из бд. Кроме того, будут обновлены и все загруженные в нее отношения.

```php
<?
$flight = Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

## Коллекции

- all() и get() извлекают записи из бд. Но возвращают они не обычный php массив, а экземпляр _Illuminate\Database\Eloquent\Collection_

### Retrieving Single Models / Aggregates

```php
<?
use App\Models\Flight;

// Retrieve a model by its primary key...
$flight = Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = Flight::where('active', 1)->first();

// Alternative to retrieving the first model matching the query constraints...
$flight = Flight::firstWhere('active', 1);
```

```php
<?
$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();
```

```php
<?
use App\Models\Flight;

// Retrieve flight by name or create it if it doesn't exist...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);

// Retrieve flight by name or create it with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// Retrieve flight by name or instantiate a new Flight instance...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);

// Retrieve flight by name or instantiate with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

ну то есть при firstOrNew мы создаем новую сущность, которую нужно засейвить еще, а firstOrCreate сразу инсёртит в бд

получаем агрегаты

```php
<?
$count = Flight::where('active', 1)->count();

$max = Flight::where('active', 1)->max('price');
```

## Inserting & Updating Models

```php
<?
    public function store(Request $request): RedirectResponse
    {
        // Validate the request...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();

        return redirect('/flights');
    }
```

после save() запись попадает в бд

### Updates

save() так же можно использовать для апдейта моделей, которые уже существуют в бд

```php
<?
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
```

- Метод update ожидает массив пар столбцов и значений, представляющих столбцы, которые должны быть обновлены.

Метод update возвращает количество затронутых строк.

```php
<?
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```

- Уязвимость массового назначения возникает, когда пользователь передает неожиданное поле HTTP-запроса, и это поле изменяет столбец в вашей базе данных, чего вы не ожидали.Например, злоумышленник может отправить параметр is_admin через HTTP-запрос, который затем передается в метод create вашей модели, позволяя пользователю повысить свои полномочия до уровня администратора.

- необходимо определить, какие атрибуты модели вы хотите сделать массово назначаемыми.Это можно сделать с помощью свойства $fillable модели.

## Deleting Models

```php
<?
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```

- Вы можете вызвать метод truncate, чтобы удалить все связанные с моделью записи базы данных.

- Операция truncate также сбросит все автоинкрементные идентификаторы в связанной с моделью таблице

```php
<?
Flight::truncate();
```

```php
<?
$deleted = Flight::where('active', 0)->delete();
```

### Soft Deleting

## Query Scopes

## Comparing Models

## Events

# TODO

- [ ] Chunking

- [ ] Cursors

- [ ] Advanced Subqueries

- [ ] Pruning Models
