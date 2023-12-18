# Глава 4. Blade Templating

```php
<?
<h1>{{ $group->title }}</h1>
{!! $group->heroImageHtml() !!}

@forelse ($users as $user)
    • {{ $user->first_name }} {{ $user->last_name }}<br>
@empty
    No users in this group.
@endforelse
```

{{ сюда что-то что мы хотим вывести с помощью пхп }}

{{ $var }} == <? = htmlentities($var) ?>

## Control Structures (Структуры управления)

### Conditionals (условия)

```php
<?
@if (count($talks) === 1)
    There is one talk at this time period.
@elseif (count($talks) === 0)
    There are no talks at this time period.
@else
    There are {{ count($talks) }} talks at this time period.
@endif
```

@unless == <?php if (! $condition)

@endunless

@for @foreach @while => @end\*

@forelse

## Temlate Inheritance

- Blade обеспечивает структуру наследования шаблонов, которая позволяет представлениям расширять, изменять и включать другие представления.

### Defining Sections with @section/@show and @yield

```php
<?
<!-- resources/views/layouts/master.blade.php -->
<html>
    <head>
        <title>My Site | @yield('title', 'Home Page')</title>
    </head>
    <body>
        <div class="container">
            @yield('content')
        </div>
        @section('footerScripts')
            <script src="app.js"></script>
        @show
    </body>
</html>
```

- У @yield('content') нет содержимого по умолчанию.

Но, кроме того, содержимое по умолчанию в @yield('title') будет показано только в том случае, если оно никогда не расширялось (extends).

Если он будет расширен, его дочерние секции не будут иметь программного доступа к значению по умолчанию.

С другой стороны, @section/@show определяет значение по умолчанию и делает это таким образом, что его содержимое по умолчанию будет доступно его дочерним разделам через @parent

```php
<?
<!-- resources/views/dashboard.blade.php -->
@extends('layouts.master')

@section('title', 'Dashboard')

@section('content')
    Welcome to your application dashboard!
@endsection

@section('footerScripts')
    @parent
    <script src="dashboard.js"></script>
@endsection
```

#### @extends

с @extends мы определяем, что этот view не должен сам рендериться, но должен расширять другое представление

- Каждый файл должен расширять только один другой файл, а вызов @extends должен находиться в первой строке файла.

#### @section and @endsection

- С помощью @section('title', 'Dashboard') мы предоставляем содержимое для первого раздела, title.

Поскольку содержимое такое короткое, вместо использования @section и @endsection мы просто используем shortcut.

Это позволяет нам передать содержимое в качестве второго параметр @section и двигаться дальше.

### Including View Partials (Включение частей представления)

```php
<?
<!-- resources/views/home.blade.php -->
<div class="content" data-page-name="{{ $pageName }}">
    <p>Here's why you should sign up for our app: <strong>It's Great.</strong></p>

    @include('sign-up-button', ['text' => 'See just how great it is'])
</div>

<!-- resources/views/sign-up-button.blade.php -->
<a class="button button--callout" data-page-name="{{ $pageName }}">
    <i class="exclamation-icon"></i> {{ $text }}
</a>
```
