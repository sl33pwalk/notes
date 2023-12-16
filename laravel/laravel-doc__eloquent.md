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

# TODO

- [ ] Chunking

- [ ] Cursors

- [ ] Advanced Subqueries
