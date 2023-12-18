# Глава 5. Базы данных и Eloquent

- конфигурацияф доступа к бд в config/database.php и .env

## Migrations

### Defining Migrations

миграция это простой файл, делающий два дела: up и down

Миграции всегда запускаются по очереди, которая определяется по дате.

Начинается миграция с самой ранней (? разве?)

```language
php artisan make:migration create_users_table
php artisan make:migration add_votes_to_users_table --table=users
php artisan make:migration create_users_table --create=users

```

Всё что мы можем сделать с миграцией основывается на методах фасада Schema

- foreignId(colName)

---

- nullable() - allows NULL значения

- default('default content')

- unsigned - маркирует integer колонки неподписанными (?)

- collation(collation)

- unique()

---

- Если вы не используете базу данных, которая поддерживает переименование и удаление столбцов, прежде чем вы сможете изменить какие-либо столбцы, вам нужно будет запустить composer require doctrine/dbal

для изменения бд добавляем change(), как я понял

- Чтобы изменить столбец, просто напишите код, который вы написали бы для создания нового столбца, а затем добавьте после него вызов метода change().

```php
<?
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 100)->change();
});
```

```php
<?
Schema::table('contacts', function (Blueprint $table)
{
    $table->renameColumn('promoted', 'is_promoted');
});
```

```php
<?
Schema::table('contacts', function (Blueprint $table)
{
    $table->dropColumn('votes');
});
```

#### Сплющивание Миграций (Squashing migrations)

- Если у вас слишком много миграций, вы можете объединить их все в один SQL-файл, который Laravel будет запускать перед выполнением всех последующих миграций. Это называется "сплющиванием" миграций

```php
<?
// Squash the schema but keep your existing migrations
php artisan schema:dump

// Dump the current database schema and delete all existing migrations
php artisan schema:dump --prune
```

#### Indexes and foreign keys (индексы и ссылочные ключи)

добавление индексов в миграцию:

```php
<?
// After columns are created...
$table->primary('primary_id'); // Primary key; unnecessary if used increments()
$table->primary(['first_name', 'last_name']); // Composite keys
$table->unique('email'); // Unique index
$table->unique('email', 'optional_custom_index_name'); // Unique index
$table->index('amount'); // Basic index
$table->index('amount', 'optional_custom_index_name'); // Basic index

```

Удаление индексов:

```php
<?
$table->dropPrimary('contacts_id_primary');
$table->dropUnique('contacts_email_unique');
$table->dropIndex('optional_custom_index_name');

// If you pass an array of column names to dropIndex, it will
// guess the index names for you based on the generation rules
$table->dropIndex(['email', 'amount']
```

добавление внешнего ключа:

```php
<?
$table->foreign('user_id')->references('id')->on('users');
```

- If we want to specify foreign key constraints, we can do that too, with cascadeOnUpdate(), restrictOnUpdate(), cascadeOnDelete(), restrictOnDelete(), and nullOnDelete()

сокращение для создания ограничений:

```php
<?
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
```

```php
<?
$table->dropForeign(['user_id']);
```

`php artisan migrate`

`php artisan migrate --seed`

`migrate:refresh` == `migrate:reset` + `migrate`

`migrate:fresh` - дропает все таблицы и заново делает миграции

`migrate:rollback` - последнюю миграцию убирает

`migrate:status`

db:show
Shows a table overview of your entire database, including the connection details, tables, size, and open connections

db:table {tableName}
Passed a table name, shows the size and lists the columns

db:monitor
List, the number of open connections to the database

## Seeding

- database/seeders

`php artisan migrate --seed`

`php artisan migrate:refresh --seed`

создания сида:

`php artisan make:seeder ContactsTableSeeder`

вызываем свой сидр из DatabaseSeeder.php

```php
<?
// database/seeders/DatabaseSeeder.php
...
public function run(): void
{
    $this->call(ContactsTableSeeder::class);
}
```

вносим бд записи в кастомный сидр:

```php
<?
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class ContactsTableSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('contacts')->insert([
            'name' => 'Lupita Smith',
            'email' => 'lupita@gmail.com',
        ]);
    }
}
```

### Model Factories

- database/factories

`php artisan make:factory ContactFactory`

в модели нужно прописать use HasFactory

```php
<?
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Contact extends Model
{
    use HasFactory;
}
```

- Трейт HasFactory предоставляет статический метод factory(), который использует соглашения Laravel для определения подходящей фабрики для модели. Он будет искать фабрику в пространстве имен Database\Factories, имя класса которой совпадает с именем модели и имеет суффикс Factory.

TODO:

- [ ] Factory

## Query Builder (Строитель запросов)

- **Query builder** - свободный (fluent) интерфейс для взаимодействия с несколькими различными типами баз данных с помощью единого понятного API

* fluent interface (Беглый интерфейс) - это интерфейс, который в основном использует цепочки методов, чтобы предоставить конечному пользователю более простой API. Вместо того чтобы ожидать, что все необходимые данные будут переданы в конструктор или вызов метода, цепочки вызовов могут быть построены постепенно, с последовательными вызовами.

```php
<?
// Non-fluent:
$users = DB::select(['table' => 'users', 'where' => ['type' => 'donor']]);

// Fluent:
$users = DB::table('users')->where('type', 'donor')->get();
```

#### Basic Usage of the DB Facade

```php
<?
// Basic statement
DB::statement('drop table users');

// Raw select, and parameter binding
DB::select('select * from contacts where validated = ?', [true]);

// Select using the fluent builder
$users = DB::table('users')->get();

// Joins and other complex calls
DB::table('users')
    ->join('contacts', function ($join) {
        $join->on('users.id', '=', 'contacts.user_id')
             ->where('contacts.type', 'donor');
    })
    ->get();
```

- Сырой SQL: select() insert() update() delete()

```php
<?
$users = DB::select('select * from users'); // вернет массив объектов stdClass
```

##### Parameter bindings and named bindings

- Архитектура базы данных Laravel позволяет использовать привязку параметров PDO (PHP data object, собственный уровень доступа к базе данных PHP), которая защищает ваши запросы от потенциальных SQL-атак.

- Передать параметр в оператор очень просто: замените значение в операторе на ?, а затем добавьте это значение во второй параметр вызова:

тут вместо ? подставиться $type

```php
<?
$usersOfType = DB::select(
    'select * from users where type = ?',
    [$type]
);

$usersOfType = DB::select(
    'select * from users where type = :type',
    ['type' => $userType]
);

DB::insert(
    'insert into contacts (name, email) values (?, ?)',
    ['sally', 'sally@me.com']
);

$countUpdated = DB::update(
    'update contacts set status = ? where id = ?',
    ['donor', $id]
);


$countDeleted = DB::delete(
    'delete from contacts where archived = ?',
    [true]
);
```

### Chaining with the Query Builder

```php
<?
$usersOfType = DB::table('users')
    ->where('type', $type)
    ->get();

```

этот код вернет коллекцию, а не массив

- Фасад DB, как и Eloquent, возвращает коллекцию для любого цепочечного метода, который возвращает (или может возвращать) несколько строк, и массив для любого нецепочечного метода, который возвращает (или может возвращать) несколько строк.

- Фасад DB возвращает экземпляр Illuminate\Support\Collection, а Eloquent возвращает экземпляр Illuminate\Database\Eloquent\Collection, который расширяет Illuminate\Support\Collection несколькими специфическими для Eloquent методами.

- Коллекция - это как массив PHP с суперспособностями, позволяющий вам выполнять map(), filter(), reduce(), each() и многое другое для ваших данных.

- select() where() orWhere()

- Если хочешь написать SQL команду, которая говорит _if this OR (this and this)_, то можно передать замыкания в orWhere() вызов:

```php
<?

$canEdit = DB::table('users')
    ->where('admin', true)
    ->orWhere(function ($query) {
    $query->where('plan', 'premium')
    ->where('is_plan_owner', true);
})->get();

// то есть

SELECT * FROM users
    WHERE admin = 1
    OR (plan = 'premium' AND is_plan_owner = 1);

```

whereBetween(ColName, [low, high])

вернуть только те строки где значения от low до high

```php
<?

$mediumDrinks = DB::table('drinks')
    ->whereBetween('size', [6, 12])
    ->get();

```

- whereNotBetween ещё есть, ну тут и так понятно ж.

* whereIn(colName, [1, 2, 3])

Позволяет ограничить запрос, чтобы вернуть только строки, в которых значение столбца находится в явно указанном списке опций

```php
<?
$closeBy = DB::table('contacts')
    ->whereIn('state', ['FL', 'GA', 'AL'])
    ->get();
```

- groupBy(), having(), havingRaw()

- value() это как first(), только если нужен лишь один столбец

- count() min() max() sum() avg()

##### Joins

```php
<?
$users = DB::table('users')
    ->join('contacts', 'user.id', '=', 'contacts.user_id')
    ->select('users.*', 'contacts.name', 'contacts.status')
    ->get();
```

метод join() создаёт inner join

leftJoin()

insert() update() delete()

```php
<?
DB::table('contacts')->truncate();
```

truncate() удаляет все записи и ресетит id

### Transactions

- Транзакции в базе данных - это инструменты, позволяющие упаковать серию запросов к базе данных для выполнения в пакетном режиме, который можно отменить, отменив всю серию запросов.

- Транзакции часто используются для того, чтобы гарантировать, что будет выполнено всё или ничто, но не некоторое, из серии связанных запросов - если один из них не будет выполнен, ORM откатит всю серию запросов.

```php
<?
DB::transaction(function () use ($userId, $numVotes) {
    DB::table('users')
        ->where('id', $userId)
        ->update(['votes' => $numVotes]);

    DB::table('votes')
        ->where('user_id', $userId)
        ->delete();
});
```

В этом примере мы можем предположить, что ранее выполнялся процесс, который суммировал количество голосов из таблицы голосов для данного пользователя.

Мы хотим кэшировать это число в таблице пользователей, а затем стереть эти голоса из таблицы голосов.

Но, конечно, мы не хотим стирать голоса до тех пор, пока обновление таблицы пользователей не будет выполнено успешно.

И мы не хотим сохранять обновленное количество голосов в таблице пользователей, если удаление таблицы голосов завершится неудачей.

можно самому начинать и заканчивать транзакции.

```php
<?
DB::beginTransaction();

// Take database actions

if ($badThingsHappened) {
    DB::rollBack();
}

// Take other database actions

DB::commit();
```

старт: DB::beginTransaction

Конец: DB::commit()

Аборт: DB::rollback

## Introduction to Eloquent

- Eloquent - это ActiveRecord ORM, что означает, что это слой абстракции базы данных, который предоставляет единый интерфейс для взаимодействия с несколькими типами баз данных.

- "ActiveRecord" означает, что один класс Eloquent отвечает не только за возможность взаимодействия с таблицей в целом (например, User::all() получает всех пользователей), но и за представление отдельной строки таблицы (например, $sharon = new User).

> Кроме того, каждый экземпляр способен управлять своей собственной сохранностью; вы можете вызвать $sharon->save() или $sharon->delete().

`php artisan make:model Contact --migration` - создаем модель и миграцию

```php
<?
$allContacts = Contact::all();
```

```php
<?
$vipContacts = Contact::where('vip',  true)->get();
```

```php
<?
$newestContacts = Contact::orderBy('created_at', 'desc')
    ->take(10)
    ->get();
```

всё что можно делать в DB фасаде, можно делать в Eloquent объектах

```php
<?
public function show($contactId)
{
    return view('contacts.show')
        ->with('contact', Contact::findOrFail($contactId));
}
```

Лучше использовать get() вместо all()

агрегаты:

```php
<?
$countVips = Contact::where('vip', true)->count();
$sumVotes = Contact::sum('votes');
$averageSkill = User::avg('skill_level');
```

TODO:

- [ ] Добить чейнинг через query builder (методы)

- [ ] чанки
