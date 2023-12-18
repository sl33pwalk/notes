# Chapter 9. User Authentication and Authorization

- **Аутентификация** означает проверку личности пользователя и разрешение ему действовать в вашей системе в качестве этого человека.Сюда входят процессы входа и выхода из системы, а также любые инструменты, позволяющие пользователям идентифицировать себя во время работы с приложением.

- **Авторизация** означает определение того, разрешено ли аутентифицированному пользователю выполнять определенное поведение (авторизация). Например, система авторизации позволяет запретить просмотр доходов сайта лицам, не являющимся администраторами.

* Интерфейс PHP - это, по сути, соглашение между двумя классами о том, что один из классов будет вести себя определенным образом. Это немного похоже на договор между ними, и представление о нем как о договоре придает названию немного больше внутреннего смысла, чем название "интерфейс".

contract => договор

Однако в конечном итоге это одно и то же: соглашение о том, что класс будет предоставлять определенные методы с определенной сигнатурой.

В связи с этим пространство имен Illuminate\Contracts содержит группу интерфейсов, которые реализуют компоненты Laravel и указывают на их тип.

Это позволяет легко разрабатывать похожие компоненты, реализующие те же интерфейсы, и устанавливать их в приложение вместо штатных компонентов Illuminate.

> Например, когда ядро и компоненты Laravel указывают на почтовый ящик, они не указывают на класс Mailer. Вместо этого они указывают контракт (интерфейс) Mailer, что упрощает создание собственного почтового сервера.

```php
<?php
// Illuminate\Foundation\Auth\User

namespace Illuminate\Foundation\Auth;

use Illuminate\Auth\Authenticatable;
use Illuminate\Auth\MustVerifyEmail;
use Illuminate\Auth\Passwords\CanResetPassword;
use Illuminate\Contracts\Auth\Access\Authorizable as AuthorizableContract;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Foundation\Auth\Access\Authorizable;

class User extends Model implements
    AuthenticatableContract,
    AuthorizableContract,
    CanResetPasswordContract
{
    use Authenticatable, Authorizable, CanResetPassword, MustVerifyEmail;
}
```

- Контракт Authenticatable требует методы (например, getAuthIdentifier()), которые позволяют фреймворку аутентифицировать экземпляры этой модели в системе аутентификации;

трейт Authenticatable включает методы, необходимые для удовлетворения этого контракта с помощью обычной модели Eloquent.

- Контракт Authorizable требует наличия метода (can()), который позволяет фреймворку авторизовать экземпляры данной модели для получения разрешений на доступ в различных контекстах.

Неудивительно, что трейт Authorizable предоставляет методы, которые удовлетворяют контракту Authorizable для средней модели Eloquent.

- контракт CanResetPassword требует методы (getEmailForPasswordReset(), sendPasswordResetNotification()), которые позволяют фреймворку сбросить пароль любой сущности, которая удовлетворяет этому контракту.

Трейт CanResetPassword предоставляет методы, удовлетворяющие этому контракту для средней модели Eloquent.

## Using the auth() Global Helper and the Auth Facade
