# Лара бля...

Значит так, директории делятся на два типа:

### The Root Directory
  The app Directory
  The bootstrap Directory
  The config Directory
  The database Directory
  The public Directory
  The resources Directory
  The routes Directory
  The storage Directory
  The tests Directory
  The vendor Directory

### The App Directory
  The Broadcasting Directory
  The Console Directory
  The Events Directory
  The Exceptions Directory
  The Http Directory
  The Jobs Directory
  The Listeners Directory
  The Mail Directory
  The Models Directory
  The Notifications Directory
  The Policies Directory
  The Providers Directory
  The Rules Directory


ух бля... ну погнали

начинаем с рута

* директория **app** содержит в себе ядро кода нашего приложения. Почти все классы в твоем приложении будут находиться именно там.

* директория **bootstrap** содержит файл *app.php*, который автозагружает фреймворк (?). Также в этой директории поселилась **cache** директория, ну она для оптимизации типа понятное дело. Тебе здесь не рады, ничего здесь не меняй.

* директория **config**, по названию уже понятно, что в себе содержит. Действительно, было бы неплохо если бы ты туда заглянул посмотрел, но... идем дальше

* директория **database** содержит в себе миграции бд, заводы модельные (ну вроде типа чтобы заполнить данными). Также можно там держать SQLite базу данных.

* **public** содержит *index.php* файл, который является входной точкой для всех реквестов, входящих в твое приложение и настраивает автозагрузку. В этом каталоге так же хранятся различные ассеты по типу картинок, js css. 

* **resources** содержит views (представления), а также сырые, некомпилированные ассеты по типу css и js.

* **routes** содержит все определения маршрутов приложения. По дефолту, в ларавеле включено несклько маршрутов: web.php, api.php, console.php и channels.php

  + **web.php** содержит маршруты, которые RouteServiceProvider помещает в группу веб-промежуточного ПО (web middleware), которое обеспечивает состояние сессии (?), CSRF защиту и шифрование куки. Если моё приложение не предлагает stateless, RESTful API, тогда все мои руты скорее всего будут определены именно в **web.php**.

  + **api.php** содержит маршруты, которые RouteServiceProvider помещает в группу *api middleware*. Эти маршруты предназначены для того, чтобы быть *stateless* (ааа, stateless это значит, что состояние не хранится на сервере), поэтому реквесты, заходящие в приложение через эти маршруты должны быть авторизованы через токены и не будут иметь доступ к session state (состояние сессии(?)).

  + **console.php** - файл, где мы определяем все наши замыкания, основанные на консольных командах. Каждое замыкание привязано к экземпляру команды, что обеспечивает простой подход к взаимодействию с методами ввода-вывода (IO methods) каждой команды. Несмотря на то, что этот файл не определяет HTTP маршруты, он определяет основанные на консоли входные точки (маршруты) в твоём приложении.

  + **channels.php** - файл, где можно зарегистрировать все каналы трансляции событий, которые поддерживает наше приложение.

* **storage** содержит логи, blade шаблоны скомпилированные, сессии (file based), кэш и многое другое, сгенерированное фреймворком. Ну сральня офк. Директория поделена на app, framework и logs директории. 


КОРОЧЕ, ПОТОМ ДОБИВАЙ ЭТО, Я НАСТОЯЩИЙ СЕЙЧАС ЗАЕБАЛСЯ

## Обзор жизненного цикла (Lyfecycle Overview)

### Первые Шаги

Точкой входа для всех реквестов, направленных на приложение Laravelm, является *public/index.php*. 

Все реквесты направляются в этот файл благодаря конфигурации твоего веб-сервера (Apache, Nginx)

*index.php* загружает сгенерированные Composer-ом определения автозагрузки, и затем получает экземпляр Laravel приложения из файла *bootstrap/app.php*

Первым делом laravel создаст экземпляр приложения / service container

### HTTP / Консольные ядра (Console Kernels)

После первых шагов, входящий запрос отправляется или в HTTP Kernel или в Console Kernel, это зависит типа запроса, который входит в программу (?)
 
Эти карты, деньги, два ядра служат центральной прослойкой, через которую проходят все запросы.
 
HTTP: *app/Http/Kernel.php*
 
* HTTP ядро наследует (extends) класс *Illuminate\Foundation\Http\Kernel*,  который определяет массив из бутстраперов, которые будут запущены перед тем, как выполнится запрос.
 
Бутстраперы конфигурируют error handling, logging, определяют окружение приложения и прочее.

> обычно, эти классы обрабатывают внутреннюю конфигурацию Laravel, о которой нам не нужно беспокоиться

* Также HTTP ядро определяет список *HTTP middleware*, через который все запросы должны пройти перед их обработкой приложением.

* Middleware (промежуточное ПО) обрабатывает чтение и запись HTTP сессии

### Service Providers (Поставщики Услуг)

* Service Providers ответственны за bootstrapping (начальную загрузку) всех компонентов фреймворка, по типу бд, запрос, валидация и прочее. 

Все service providers конфигурируются в *config/app.php* файле, в массиве *providers*

Laravel поэтапно пройдет через этот список провайдеров/поставщиков и создаст экземпляр для каждого/

* Экземпляр класса (instance) - это описание конкретного объекта в памяти

После создания экземпляра провайдеров, будет вызван метод *register* для всех поставщиков.

Затем, после того, ка все провайдеры будут зарегистрированы, метод *boot* будет вызван так же для всех провайдеров. 

Это сделано для того, чтобы service providers могли зависеть от того, что каждая привязка контейнера будет зарегистрирована и доступна к моменту выполнения метода *boot*

Service Providers - самый важный аспект всего Laravel bootstrap процесса.

### Routing (Маршрутизация)

* Service provider *App\Providers\RouteServiceProvider* загружает файлы маршрутов, которые находятся внутри директории *routes* нашего приложения

После того, как приложение было загружено (bootstrapped) и все service providers были зарегистрированы, будет передан *Request* маршрутизатору для отправки.

Маршрутизатор отправит запрос на маршрут (route) или контроллер, а также запустит любое промежуточное ПО (middleware), специфичное для маршрута.

Если запрос пройдет через всё назначенное middleware соответствующего маршрута, метод контроллера или маршрута будет выполнен и ответ, возвращенный методом маршрута или контроллера, будет отправлен обратно через цепочку middleware




