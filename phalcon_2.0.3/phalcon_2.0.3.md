>Phalcon 2.0.3 released
https://blog.phalconphp.com/post/phalcon-2-0-3-released


В рамках нашего расписания регулярных релизов, мы рады сообщить, что Phalcon 2.0.3 был выпущен!

Эта версия содержит множество исправлений, а также новую функциональность, востребованную сообществом.

## Изменения

* Реализованы псевдонимы для namespace в PHQL
* Возможность определять должен ли виртуальный внешний ключ игнорировать `null`-значения или нет
* Добавлена поддержка `Phalcon\Mvc\Collection` в поведениях (Behaviours)
* Добавлены поведения `SoftDelete` и `Timestampable` в коллекциях
* Исправлена ошибка, добавляющая двойной `?` в `Mvc\Url::get` при использовании параметров [#10421](https://github.com/phalcon/cphalcon/issues/10421)
* Строковые атрибуты в моделях теперь имеют опциональную поддержку пустых значений строки [#440](https://github.com/phalcon/cphalcon/issues/440)
* Добавлена возможность возвращать SQL, генерируемый в экземплярах `Mvc\Model\Query` [#1908](https://github.com/phalcon/cphalcon/issues/1908)
* Исправление некорректно генерируемого SELECT в `Phalcon\Db\Dialect::select()` [#10439](https://github.com/phalcon/cphalcon/issues/10439)
* Добавлена поддержка типа Double в MySQL
* `Phalcon\Tag\Select` теперь обрабатывает массив значений в виде строк избегая принятия нуля за пустую строку [#2921](https://github.com/phalcon/cphalcon/issues/2921)
* PHQL теперь поддерживает выражения CASE/WHEN/ELSE [#651](https://github.com/phalcon/cphalcon/issues/651)
* Исправлена ошибка, возникающая при добавлении нестроковых значений в `Phalcon\Crypt::encrypt` из `Phalcon\Http\Cookies`
* Исправлена ошибка непередачи имени схемы (PostgreSQL)
* Атрибут `persistent` был удален из DNS-атрибутов для PDO соединений, в целях избежания ошибок в PostgreSQL [#10484](https://github.com/phalcon/cphalcon/issues/10484)

## Основные моменты

### Поддержка CASE/WHEN/ELSE

Теперь выражения `CASE/WHEN/ELSE` доступны в [PHQL](https://docs.phalconphp.com/en/latest/reference/phql.html):

```php
$robots = $this->modelsManager->executeQuery("
    SELECT 
        CASE r.Type
            WHEN 'Mechanical' THEN 1
            WHEN 'Virtual' THEN 2
            ELSE 3
        END 
    FROM Store\Robots
");
```

## Псевдонимы для Namespace

Если вы используете пространства имен для организации ваших моделей, вы нередко оказывались в ситуации, когда для простой ссылки на модель необходимо набирать длинный namespace. Теперь вы можете добавлять псевдонимы для существующих пространств имен, ускоряя вашу разработку:

```php
// Before
$data = $this->modelsManager->executeQuery("
    SELECT r.*, rp.*
    FROM Store\Backend\Models\Robots AS r
    JOIN Store\Backend\Models\RobotsParts AS rp
");
```

Определения псевдонимов в менеджере моделей:

```php
use Phalcon\Mvc\Model\Manager as ModelsManager;

// ...

$di->set(
    'modelsManager', 
    function() {
        $modelsManager = new ModelsManager();
        $modelsManager->registerNamespaceAlias(
            'bm',
             'Store\Backend\Models\Robots'
         );
        return $modelsManager;
    }
);
```

И в запросах:

```php
// After
$data = $this->modelsManager->executeQuery("
    SELECT r.*, rp.*
    FROM bm:Robots AS r
    JOIN bm:RobotsParts AS rp
");
```

#### Custom Dialect Functions

This new functionality will help you to extend PHQL as you need using custom functions. In the following example we're going to implement the MySQL's extension MATCH/BINARY. First of all you have to instantiate the SQL dialect

```php
use Phalcon\Db\Dialect\MySQL as SqlDialect;
use Phalcon\Db\Adapter\Pdo\MySQL as Connection;

$dialect = new SqlDialect();

// Register a new function called MATCH_AGAINST
$dialect->registerCustomFunction(
    'MATCH_AGAINST', 
    function($dialect, $expression) {
        $arguments = $expression['arguments'];
        return sprintf(
            " MATCH (%s) AGAINST (%)",
            $dialect->getSqlExpression($arguments[0]),
            $dialect->getSqlExpression($arguments[1])
         );
    }
);

// The dialect must be passed in the connection constructor
$connection = new Connection(
    [
        "host"          => "localhost",
        "username"      => "root",
        "password"      => "",
        "dbname"        => "test",
        "dialectClass"  => $dialect
    ]
);
```

Now you can use this function in PHQL and it internally translates to the right SQL using the custom function:

```php
$phql = "SELECT * 
   FROM Posts 
   WHERE MATCH_AGAINST(title, :pattern:)";
$posts = $modelsManager->executeQuery($phql, ['pattern' => $pattern]);
```

#### Improvements in Subqueries

In Phalcon 2.0.2 subqueries were introduced in PHQL. Support for this feature had been improved in 2.0.3 by introducing the EXISTS operator:

```php
$phql = "SELECT c.* 
  FROM Shop\Cars c
  WHERE EXISTS (
     SELECT id 
     FROM Shop\Brands b 
     WHERE b.id = c.brandId
  )";
$cars = $this->modelsManager->executeQuery($phql);
```

### Update/Upgrade

This version can be installed from the master branch, if you don’t have Zephir installed follow these instructions:

```bash
git clone http://github.com/phalcon/cphalcon
git checkout master
cd ext
sudo ./install
```

The standard installation method also works:

```bash
git clone http://github.com/phalcon/cphalcon
git checkout master
cd build
sudo ./install
```

If you have Zephir installed:

```bash
git clone http://github.com/phalcon/cphalcon
git checkout master
zephir fullclean
zephir build
```

Note that running the installation script will replace any version of Phalcon installed before.

Windows DLLs are available in the [download page](http://phalconphp.com/en/download/windows).

See the [upgrading guide](https://blog.phalconphp.com/post/guide-upgrading-to-phalcon-2) for more information about upgrading to Phalcon 2.0.x from 1.3.x.

*   [Documentation](https://docs.phalconphp.com)
*   [API](https://api.phalconphp.com/) (Thanks to [gsouf](https://github.com/gsouf))

## Thanks

Thanks to everyone involved in making this version: collaborators and as well to the community for their continuous input and feedback!