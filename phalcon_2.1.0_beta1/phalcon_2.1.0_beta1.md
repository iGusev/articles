> Phalcon 2.1.0 beta 1 released
https://blog.phalconphp.com/post/phalcon-2-1-beta-released

Мы рады представить вам первый бета-релиз **Phalcon 2.1**!

Релизы 2.1.x будут поддерживаться в течении более длительного периода, 2.1 будет нашей первой версией с долгострочной поддержкой ([LTS](https://en.wikipedia.org/wiki/Long-term_support)).

В 2.0.x мы ввели несколько новых фич и поправили множество багов. Однако, наше внимание всегда было обращено на сохранение обратной совместимости с Phalcon 1.3.x, в то же время мы стимулировали разработчиков обновляться до 2.0.x. Это дало достаточно времени разработчикам на внесение изменений в свои приложения для работы с новой веткой.

Phalcon 2.1 предоставляет новые возможности, некоторые из которых несовместимы с предыдущими версиями, поэтому убедитесь, что вы проверили свои приложения перед обновлением production-систем.

Мы уверены, что изменения в этом релизе оправдают обновление :)

### Окончание поддержки PHP 5.3

Phalcon 2.0.x - последняя серия релизов с поддержкой PHP 5.3 (>= 5.3.21). Из-за этого ограничения мы не можем включать некоторые улучшения производительности в фреймворк.

С версии 2.1.x мы настоятельно реккомендуем разработчикам обновляться до 5.6, а там уже и PHP7 не за горами. Мы нацелены на работу с PHP 7, но в тоже время рекоммендуемой версией PHP для работы сейчас является 5.6.

### `Phalcon\Mvc\Model\Validation` объявлен устаревшим (deprecated)

[Phalcon\Mvc\Model\Validation](https://docs.phalconphp.com/en/latest/reference/models.html#validating-data-integrity) уступил место в пользу [Phalcon\Validation](https://docs.phalconphp.com/en/latest/reference/validation.html). Функциональность обоих компонентов сливается в одно целое, тем самым упрощая поддержку кодовой базы.

Ранее валидация осуществлялась следующим образом:

```php
namespace Invo\Models;

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Validator\Email as EmailValidator;
use Phalcon\Mvc\Model\Validator\Uniqueness as UniquenessValidator;

class Users extends Model
{
    public function validation()
    {
        $this->validate(
            new EmailValidator(
                [
                    'field' => 'email',
                ]
            )
        );

        $this->validate(
            new UniquenessValidator(
                [
                    'field'   => 'username',
                    'message' => 'Sorry, That username is already taken',
                ]
            )
        );

        if ($this->validationHasFailed() == true) {
            return false;
        }
    }
}
```

С введением [Phalcon\Validation](https://docs.phalconphp.com/en/latest/reference/validation.html), вам необходимо будет изменить вышеуказанный код:

```php
namespace Invo\Models;

use Phalcon\Mvc\Model;
use Phalcon\Validation;
use Phalcon\Validation\Validator\Email as EmailValidator;
use Phalcon\Validation\Validator\Uniqueness as UniquenessValidator;

class Users extends Model
{
    public function validation()
    {
        $validator = new Validation();

        $validator->add(
            'email',
            new EmailValidator()
        );

        $validator->add(
            'username',
            new UniquenessValidator(
                [
                    'message' => 'Sorry, That username is already taken',
                ]
            )
        );

        return $this->validate();
    }
}
```

Согласитесь, это изменение делает код намного читабельнее.

### Изменения в конструкторе `Phalcon\Mvc\Model`

Конструктор модели классов был изменен, чтобы позволить вам передавать массив данных для инициализации:

```php
$customer = new Customer(
    [
        'name'   => 'Peter',
        'status' => 'Active',
    ]
);
```

Использование этого метода является аналогом `assign()`, т.е. будет использоваться доступный сеттер (реализованный в конкретной модели или встроенный) для присваивания свойства.

### `Phalcon\Mvc\View` поддерживает несколько директорий представлений

Это была одна из фич, о которой наше сообщество просило множество раз. Рады сообщить, что теперь вы можете использовать любой вид иерархии при указании директорий с представлениями. Особенно это полезно для повторного использования в нескольких модулях:

```php
use Phalcon\Mvc\View;

// ...

$di->set(
    'view',
    function () {

        $view = new View();

        $view->setViewsDir(
            [
                '/var/www/htdocs/blog/modules/backend/views/',
                '/var/www/htdocs/blog/common/views/',
            ]
        );

        return $view;
    }
);
```

### `Phalcon\Mvc\View` теперь поддерживает абсолютные пути

Абсолютный путь теперь может быть использован в `Mvc\View::setLayoutsDir` и `Mvc\View::setPartialsDir`. Это позволяет использовать папки за пределами основной директории представления.

```php
use Phalcon\Mvc\View;

// ...

$di->set(
    'view',
    function () {

        $view = new View();

        $view->setViewsDir(
            [
                '/var/www/htdocs/blog/modules/backend/views/',
                '/var/www/htdocs/blog/common/views/',
            ]
        );

        $view->setLayoutsDir(
            '/var/www/htdocs/common/views/layouts/'
        );

        return $view;
    }
);
```

### `Phalcon\Di` теперь привязан к замыканиям сервисов

В прошлом мы должны были передавать контейнер зависимостей внутрь замыкания сервиса, если необходимо было выполнять некоторые действия внутри. Например, для доступа к конфигу или к event-менеджеру. Теперь мы можем использовать `$this`, чтобы получить доступ к `Phalcon\Di`, а также к уже зарегистрированным сервисам.

Код раньше:

```php
use Phalcon\Mvc\Dispatcher;

// ...

$di->set(
    'dispatcher',
     function () use ($di) {
        $eventsManager = $di->getEventsManager();

        $eventsManager->attach(
            'dispatch:beforeException',
            new NotFoundPlugin()
        );

        $dispatcher = new Dispatcher;
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    }
);
```

Теперь вы можете получить доступ к сервисам без передачи `$di`:

```php
use Phalcon\Mvc\Dispatcher;

// ...

$di->set(
    'dispatcher',
    function () {
        $eventsManager = $this->getEventsManager();

        $eventsManager->attach(
            'dispatch:beforeException',
            new NotFoundPlugin()
            );

        $dispatcher = new Dispatcher;
        $dispatcher->setEventsManager($eventsManager);
        return $dispatcher;
    }
);
```

### Разрешено переопределение сервисов

Если объект возвращается после события `beforeServiceResolve` в `Phalcon\Di`, возвращенный экземпляр переопределяет значение сервиса по умолчанию.

В следующем примере показано как переопределить создание сервиса `response` из своего плагина:

```php
use Phalcon\Di;
use Phalcon\Http\Response;
use Phalcon\Events\Manager;

use MyApp\Plugins\ResponseResolverInterceptor;

$di = new Di();

$eventsManager = new EventsManager;

// Intercept service creation
$eventsManager->attach(
    'di',
    new ResponseResolverInterceptor()
);

$di->set('response', Response::class);

$di->setInternalEventsManager($eventsManager);

```

С помощью плагинов можно перехватывать создание сервисов:

```php
namespace MyApp\Plugins;

use Phalcon\Http\Response;

class ResponseResolverInterceptor
{
    private $cache = false;

    public function beforeServiceResolve($event, $di, $parameters)
    {
        // Intercept creation of responses
        if ($parameters['name'] == 'response' && $this->cache == false) {
            $response = new Response();
            $response->setHeader('Cache-Control', 'no-cache, must-revalidate');

            return $response;
        }
    }
}
```

### Отключение представления из метода `action`

Иногда существует необходимость выключить представление путем вызова `$this->view->disable()` в рамках конкретного `action`-метода контроллера во избежание дальнейшей обработки результата компонентом `Phalcon\Mvc\View`.

Это стало гораздо проще; просто верните `false`:

```php
use Phalcon\Mvc\Controller;

class Api extends Controller
{
    public function loginAction()
    {
        if ($this->safeApp->isBanned()) {
            $this->response->setStatusCode(401, "Unauthorized");
            return false;
        }

        // ...
    }
}
```

### Возврат строки делает ее телом ответа

Возврат строки из действия контроллера воспринимается в качестве тела ответа:
(также как `return $this->response->setContent('Hello world')`)

```php
use Phalcon\Mvc\Controller;

class Session extends Controller
{
    public function welcomeAction()
    {
        return '<h1>Hello world!</h1>';
    }
}
```

Особенно удобно, если `Phalcon\Mvc\View\Simple` используется вместо `Phalcon\Mvc\View`:

```php
use Phalcon\Mvc\Controller;

class Session extends Controller
{
    public function welcomeAction($name)
    {
        return $this->view->render(
            'welcome/index',
            [
                'name' => $name,
            ]
        );
    }
}
```

Эта функция также доступна в обработчиках `Mvc\Micro`:

```php
use Phalcon\Mvc\Micro;

$app = new Micro();

// ...

$app->get(
    '/hello/{name}',
    function () {
        return $this->view->render(
            'hello',
            [
                'name' => $name,
            ]
        );
    }
);
```

### Переопределение поведения диспетчера+представления в маршрутах

Маршрутам теперь можно назначать коллбеки, которые могут переопределять поведение по умолчанию у диспетчера и представления:

```php
// Make a redirection if the /help route is matched
$router->add('/help', [])->match(function () {
    return $this->getResponse()->redirect('https://support.google.com/');
});

// Return a string directly from the route
$router->add('/', [])->match(function () {
    return '<h1>It works</h1>';
});
```

Читайте полный список изменений Phalcon 2.1 в [CHANGELOG](https://github.com/phalcon/cphalcon/blob/2.1.x/CHANGELOG.md#210-2015-xx-xx).

## Помощь с тестированием

Данная версия может быть установлена из ветки `2.1.х`. Если у вас нет Zephir, выполните следующие команды:

```sh
git clone https://github.com/phalcon/cphalcon
git checkout 2.1.x
cd cphalcon/ext
sudo ./install
```

Если же у вас установлен Zephir:

```sh
git clone https://github.com/phalcon/cphalcon
cd cphalcon/
git checkout 2.1.x
zephir build
```

Мы надеемся, что вам понравятся эти улучшения и дополнения. Приглашаем Вас поделиться своими мыслями и вопросами об этой версии на [Phosphorum](https://forum.phalconphp.com/).

<3 Команда Phalcon