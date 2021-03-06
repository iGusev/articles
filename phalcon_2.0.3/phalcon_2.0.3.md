>Phalcon 2.0.3 released
https://blog.phalconphp.com/post/phalcon-2-0-3-released


В рамках нашего расписания регулярных релизов, мы рады сообщить, что Phalcon 2.0.3 был выпущен!

Эта версия содержит множество исправлений, а также новые возможности, основанные на фидбеке сообщества.

## Изменения

* Реализованы псевдонимы для namespace в PHQL
* Возможность определять должен ли виртуальный внешний ключ игнорировать `null`-значения или нет
* Добавлена поддержка `Phalcon\Mvc\Collection` в поведениях (Behaviours)
* Добавлены поведения `SoftDelete` и `Timestampable` в коллекциях
* Исправлена ошибка, добавляющая двойной `?` в `Mvc\Url::get` при использовании параметров [#10421](https://github.com/phalcon/cphalcon/issues/10421)
* Строковые атрибуты в моделях теперь имеют опциональную поддержку пустых значений строки [#440](https://github.com/phalcon/cphalcon/issues/440)
* Добавлена возможность возвращать SQL, генерируемый в экземплярах `Mvc\Model\Query` [#1908](https://github.com/phalcon/cphalcon/issues/1908)
* Исправление некорректно генерируемого запроса в `Phalcon\Db\Dialect::select()` [#10439](https://github.com/phalcon/cphalcon/issues/10439)
* Добавлена поддержка типа Double в MySQL
* `Phalcon\Tag\Select` теперь обрабатывает массив значений строк, избегая принятия нуля за пустую строку [#2921](https://github.com/phalcon/cphalcon/issues/2921)
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
// До
$data = $this->modelsManager->executeQuery("
    SELECT r.*, rp.*
    FROM Store\Backend\Models\Robots AS r
    JOIN Store\Backend\Models\RobotsParts AS rp
");
```

Определение псевдонимов в менеджере моделей:

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
// После
$data = $this->modelsManager->executeQuery("
    SELECT r.*, rp.*
    FROM bm:Robots AS r
    JOIN bm:RobotsParts AS rp
");
```

### Функции пользовательского диалекта

Эта новая функоциональность поможет вам расширить PHQL с помощью пользовательских функций так как вам необходимо. В следующем примере мы реализуем поддержку MATCH/BINARY из MySQL. Прежде всего вы должны инстанцировать SQL диалект:

```php
use Phalcon\Db\Dialect\MySQL as SqlDialect;
use Phalcon\Db\Adapter\Pdo\MySQL as Connection;

$dialect = new SqlDialect();

// Регистрация новой функции MATCH_AGAINST
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

// Диалект должен быть передан в конструктор соединения
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

Теперь вы можете использовать эту функцию в PHQL и он транслирует ее в правильный SQL:

```php
$phql = "SELECT * 
   FROM Posts 
   WHERE MATCH_AGAINST(title, :pattern:)";
$posts = $modelsManager->executeQuery($phql, ['pattern' => $pattern]);
```

### Улучшения в подзапросах

В Phalcon 2.0.2 были введены подзапросы PHQL. Поддержка этой функции была улучшена в 2.0.3 путем введения оператора EXISTS:

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

## Обновление/Установка

Данная версия может быть установлена из master ветки. Если у вас не установлен Zephir, выполните следующие команды:

```bash
git clone http://github.com/phalcon/cphalcon
git checkout master
cd ext
sudo ./install
```

Стандартный способ установки также работает:

```bash
git clone http://github.com/phalcon/cphalcon
git checkout master
cd build
sudo ./install
```

Если у вас уже установлен Zephir:

```bash
git clone http://github.com/phalcon/cphalcon
git checkout master
zephir fullclean
zephir build
```

_Обратите внимание, что при запуске установочный скрипт заменит уже установленную версию Phalcon._

DLL для Windows доступны на [странице загрузке](http://phalconphp.com/en/download/windows).

Смотрите [руководство по обновлению](https://blog.phalconphp.com/post/guide-upgrading-to-phalcon-2), если хотите обновиться до Phalcon 2.0.x с 1.3.x.

*   [Документация](https://docs.phalconphp.com)
*   [API](https://api.phalconphp.com/) (Спасибо [gsouf](https://github.com/gsouf))

## Благодарности

Спасибо всем, кто принималь участие в создании этой версии: и контрибьюторам и сообществу, за веру в нас и обратную связь!