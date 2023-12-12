# Eloquent: Getting Started

- Eloquent - обхектно-реляционный mapper (ORM).

* При использовании Eloquent каждая таблица базы данных имеет соответствующую "модель", которая используется для взаимодействия с этой таблицей. Помимо получения записей из таблицы базы данных, модели Eloquent позволяют вставлять, обновлять и удалять записи из таблицы

`php artisan make:model Flight

Модели обычно живут в _app\Models_ и расширяют класс _Illuminate\Database\Eloquent\Model_

`php artisan model:show Flight`

- По соглашению, в качестве имени таблицы будет использоваться "snake case", множественное имя класса, если явно не указано другое имя. Так, в данном случае Eloquent предположит, что модель Flight хранит записи в таблице flights, а модель AirTrafficController будет хранить записи в таблице air_traffic_controllers.
