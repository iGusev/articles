> Phalcon 2.1.0 beta 1 released
https://blog.phalconphp.com/post/phalcon-2-1-beta-released

Мы рады представить Вам первый бета-релиз **Phalcon 2.1**!

Релизы 2.1.x будут поддерживаться в течении более длительного периода, 2.1 будет нашей первой версией с долгострочной поддержкой ([LTS](https://en.wikipedia.org/wiki/Long-term_support))

В 2.0.x мы ввели несколько новых фич и поправили множество багов. Однако, наше внимание всегда было обращено на сохранение обратной совместимости с Phalcon 1.3.x, в то же время мы стимулировали разработчиков обновляться до 2.0.x. Это дало достаточно времени разработчикам на внесение изменений в свои приложения для работы с 2.0.x.

Phalcon 2.1 предоставляет новые возможности, некоторые из которых несовместимы с предыдущими версиями, поэтому убедитесь, что вы проверили свои приложения перед обновлением production-систем.

Мы уверены, что изменения в этом релизе оправдают обновление :)

## Окончание поддержки PHP 5.3

Phalcon 2.0.x - последняя серия релизов с поддержкой PHP 5.3 (>= 5.3.21). Из-за этого ограничения мы не можем включать некоторые улучшения производительности в фреймворк.

С версии 2.1.x мы настоятельно реккомендуем разработчикам обновляться до 5.6, а там уже и PHP7 не за горами. Мы нацелены на работу с PHP 7, но в тоже время рекоммендуемой версией PHP для работы с Phalcon является 5.6

### `Phalcon\Mvc\Model\Validation` объявлен устаревшим (deprecated)

[Phalcon\Mvc\Model\Validation](https://docs.phalconphp.com/en/latest/reference/models.html#validating-data-integrity) уступил место в пользу [Phalcon\\Validation](https://docs.phalconphp.com/en/latest/reference/validation.html). Функциональность обоих компонентов сливается в одно целое, тем самым упрощая поддержку кодовой базы.

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

С введением [Phalcon\\Validation](https://docs.phalconphp.com/en/latest/reference/validation.html), вам необходимо будет изменить вышеуказанный код:

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

Использование этого метода является аналогом `assign()`, будет использоваться доступный сеттер (реализованный в конкретной модели или встроенный) для присваивания свойства.

### `Phalcon\Mvc\View` supports many views directories

This has been one of the features that our community requested many times in the past. We are happy to announce that you can use any kind of folder hierarchy with your application for your view files. This is specially useful for reusing views and layouts between modules:

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

### `Phalcon\Di` is now bound to services closures

In the past, we had to pass the dependency injector inside a service closure, if we had to perform some actions inside the closure. Examples of that were accessing the configuration object or the events manager. Now we can use `$this` to access the `Phalcon\Di` along with the registered services.

Code before:

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

Now you can access services without passing `$di`:

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

Теперь плагин может перехватывать создание сервисов:

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

###Disabling the view from an action

Sometimes the view must be disabled by calling `$this->view->disable()` within an action in order to avoid the `Phalcon\Mvc\View` component to process the relevant view(s).

Now this much easier; simply return `false`:

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

### Returning a string makes it the body of the response

Return a string from an action takes it as the body of the response:
(same as `return $this->response->setContent('Hello world')`)

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

This feature is specially handy if `Phalcon\Mvc\View\Simple` is used instead of `Phalcon\Mvc\View`:

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

This feature is also available in `Mvc\Micro` handlers:

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

### Override dispatcher+view behavior in routes
Routes now can have an associated callback that can override the default dispatcher + view behavior:

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

See the full [CHANGELOG](https://github.com/phalcon/cphalcon/blob/2.1.x/CHANGELOG.md#210-2015-xx-xx) for Phalcon 2.1.

## Help with Testing

This version can be installed from the 2.1.x branch. If you don't have Zephir installed follow these instructions:

```sh
git clone https://github.com/phalcon/cphalcon
git checkout 2.1.x
cd cphalcon/ext
sudo ./install
```

If you have Zephir installed:

```sh
git clone https://github.com/phalcon/cphalcon
cd cphalcon/
git checkout 2.1.x
zephir build
```

We hope that you will enjoy these improvements and additions. We invite you to share your thoughts and questions about this version on [Phosphorum](https://forum.phalconphp.com/).

<3 Phalcon Team