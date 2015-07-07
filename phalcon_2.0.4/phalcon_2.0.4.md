>Phalcon 2.0.4 released
https://blog.phalconphp.com/post/phalcon-2-0-4-released

В рамках графика наших трех-пятинедельных минорных релизов, мы рады сообщить, что вышел Phalcon 2.0.4!

Число улучшений и исправлений значительно увеличилось по сравнению с другими релизами 2.0.x:

## Изменения

*   Испавлен баг в  `Phalcon\Mvc\Model::update()` ошибочно выдающий исключение, когда запись действительно существует
*   Теперь ссылки в `Phalcon\Debug` указывают на [https://api.phalconphp.com](https://api.phalconphp.com) вместо [http://docs.phalconphp.com](http://docs.phalconphp.com)

*   Implemented a more versatile way to assign variables in Volt allowing to assign properties and array indexes
*   Реализован универсальный способ назначения переменных в Volt, позволяющий назначать переменные и массив индексов


*   Улучшены макросы в Volt через использование анонимных функций, позволяющих связывать адаптер объекта и DI-сервисы вместе
*   Исправлена генерация и валидация стандартных параметров в макросах Volt
*   Добавлен метод `Phalcon\Assets\Manager::getCollections()` возвращающий все зарегистрированные коллекции [#2488](https://github.com/phalcon/cphalcon/pull/2488)
*   Теперь `Phalcon\Mvc\Url::getStatic()` генерирует URLы из роутинга
*   Добавлен `Phalcon\Mvc\EntityInterface` для общей абстракции над `Phalcon\Mvc\Model` и `Phalcon\Mvc\Collection`. Этот интерфейс поддерживает `Mvc\Model\Validators` для использования в `Mvc\Collection`
*   Добавлен метод `Phalcon\Session\Adapter::setName()` для изменения имени сессии
*   Добавлена поддержка колонки `BIGINT` в `Phalcon\Db`
*   Добавлены новые типы `Phalcon\Db\Column::BLOB` и `Phalcon\Db\Column::DOUBLE` [#10506](https://github.com/phalcon/cphalcon/pull/10506)
*   Автоматическая привязка Large Object data (LOB) в ORM
*   Поддержка для MySQL типа `BIT` c привязкой в качестве `boolean`
*   Добавлен метод `Phalcon\Flash\Direct::output()` позволяющий разместить flash-сообщения в определенном месте шаблона [#629](https://github.com/phalcon/cphalcon/pull/629)
*   Добавлена опция 'autoescape', которая позволяет включить на глобальном уровне autoescape в любом Volt-шаблоне
*   Добавлены `readAttribute`/`writeAttribute` в `Phalcon\Mvc\Collection\Document`
*   Добавлен `toArray` в `Phalcon\Mvc\Collection\Document`
*   Глобальный параметр `db.force_casting` теперь позволяет форсировать приведение указанных типов
*   Введен новый синтаксис в PHQL, позволяющий установить тип: `{name:str}` или `{names:array}`
*   Теперь вы можете работать с массивами в качестве параметров в PHQL
*   Глобальный параметр `orm.cast_on_hydrate` позволяет отдавать атрибуты с оригинальными типами из сопоставленной таблицы вместо использования строк
*   Значения в `LIMIT`/`OFFSET` теперь возможно добавлять через привязанные параметры в PHQL
*   Поддержка позднего статического связывания в Simple/Complex результатах для переопределения `Mvc\Model::cloneResultMap`
*   Добавлен метод `distinct()` в `Phalcon\Mvc\Model\Criteria` [#10536](https://github.com/phalcon/cphalcon/issues/10536)
*   Добавлен глобальный параметр `orm.ignore_unknown_columns` для игнорирования неучтенных колонок в ORM. Это убирает лишние вспомогательные столбцы, используемые в `Db\Adapter\Pdo\Oracle`
*   Добавлена поддержка для `afterFetch` в `Mvc\Collection`
*   Добавлен параметр `beforeMatch` в `@Route` аннотацию из `Mvc\Router\Annotations`
*   Добавлены `groupBy`/`getGroupBy`/`having`/`getHaving` в `Mvc\Model\Criteria`
*   `Phalcon\Mvc\Model::count()` теперь возвращает значение с типом int
*   Удален `__construct` из `Phalcon\Mvc\View\EngineInterface`
*   Добавлен метод `Phalcon\Debug\Dump::toJson()` для возврата значения в виде JSON с информацией о переменной
*   Экземпляры в `Phalcon\Di` строятся с использованием внутренних оптимизаторов вместо `\ReflectionClass` (PHP 5.6)
*   Добавлен `Phalcon\Mvc\Model\Validator\IP` из `phalcon/incubator`
*   Добавлен возвращаемый параметр `defaultValue` в `Phalcon\Mvc\Model\Validator::getOption()`
*   Теперь разработчики могут определять связи с помощью условных операторов

## Основные моменты

### Типизированные плейсхолдеры в ORM

До этой версии, только стандартные плейсхолдеры (строки и численные) поддерживались в [PHQL](https://docs.phalconphp.com/en/latest/reference/phql.html). Они позволяли связать параметры для избежания SQL-инъекций:

```php
$phql = "SELECT * FROM Store\Robots WHERE id > :id:";
$robots = $this->modelsManager->executeQuery($phql, ['id' => 100]);
```

Впрочем, некоторые СУБД требуют дополнительных действий при использовании плейсхолдеров, таких как указание типа:

```php
use Phalcon\Db\Column;

// ...

$phql = "SELECT * FROM Store\Robots LIMIT :number:";
$robots = $this->modelsManager->executeQuery(
    $phql,
    ['number' => 10],
    Column::BIND_PARAM_INT
);
```

Для облегчения этой задачи Phalcon 2.0.4 вводит типизированные плейсхолдеры, работающие точно также как и раньше, но с возможностью указать тип:

```php
$phql = "SELECT * FROM Store\Robots LIMIT {number:int}";
$robots = $this->modelsManager->executeQuery(
    $phql,
    ['number' => 10]
);

$phql = "SELECT * FROM Store\Robots WHERE name <> {name:str}";
$robots = $this->modelsManager->executeQuery(
    $phql,
    ['name' => $name]
);
```

Вы можете также опустить указание типа если вам не нужно:

```php
$phql = "SELECT * FROM Store\Robots WHERE name <> {name}";
$robots = $this->modelsManager->executeQuery(
    $phql,
    ['name' => $name]
);
```

Типизированные плейсхолдеры также более функциональны, так как теперь мы можем привязать статический массив без необходимости передавать каждый элемент отдельноб в качестве плейсхолдера:

```php
$phql = "SELECT * FROM Store\Robots WHERE id IN ({ids:array})";
$robots = $this->modelsManager->executeQuery(
    $phql,
    ['ids' => [1, 2, 3, 4]]
);
```

Доступны следующие типы:

| Тип        | Константа типа                     | Пример            |
|------------|------------------------------------|-------------------|
| str        | `Column::BIND_PARAM_STR`           | `{name:str}`      |
| int        | `Column::BIND_PARAM_INT`           | `{number:int}`    |
| double     | `Column::BIND_PARAM_DECIMAL`       | `{price:double}`  |
| bool       | `Column::BIND_PARAM_BOOL`          | `{enabled:bool}`  |
| blob       | `Column::BIND_PARAM_BLOB`          | `{image:blob}`    |
| null       | `Column::BIND_PARAM_NULL`          | `{exists:null}`   |
| array      | Array of `Column::BIND_PARAM_STR`  | `{codes:array}`   |
| array-str  | Array of `Column::BIND_PARAM_STR`  | `{names:array}`   |
| array-int  | Array of `Column::BIND_PARAM_INT`  | `{flags:array}`   |

### Валидация привязанных параметров

По умолчанию привязанные к плейсхолдерам параметры не поддерживали указание типа, теперь же есть возможность проверять типы параметров прежде, чем начинать работу с PDO.

Классической ситуацией, когда возникает проблема, является передача строки в плейсхолдер `LIMIT`/`OFFSET`:

```php
$number = '100';
$robots = $modelsManager->executeQuery(
    'SELECT * FROM Some\Robots LIMIT {number:int}',
    ['number' => $number]
);
```

Такой код выбросит следующее исключение:

`` Fatal error: Uncaught exception 'PDOException' with message 'SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''100'' at line 1' in /Users/scott/demo.php:78 ``

Это происходит потому, что 100 - строковая переменная. Поправить легко, нужно всего лишь привести тип к int:

```php
$number = '100';
$robots = $modelsManager->executeQuery(
    'SELECT * FROM Some\Robots LIMIT {number:int}',
    ['number' => (int) $number]
);
```

Однако такое решение требует от разработчика особого внимания к работе с типами. Чтобы облегчить задачу и избежать неожиданных исключений можно заставить Phalcon делать эту работу за вас:

```php
\Phalcon\Db::setup(['forceCasting' => true]);
```

Следующие действия выполняются согласно привязке указанного типа:


| Тип                          | Действие                                         |
|------------------------------|--------------------------------------------------|
| `Column::BIND_PARAM_STR`     | Передает значение как нативный тип PHP string    |
| `Column::BIND_PARAM_INT`     | Передает значение как нативный тип PHP integer   |
| `Column::BIND_PARAM_BOOL`    | Передает значение как нативный тип PHP boolean   |
| `Column::BIND_PARAM_DECIMAL` | Передает значение как нативный тип PHP double    |

### Приведение типа в получаемых из PDO значениях

Значения, возвращаемые из базы данных системы всегда передаются как строковые значения посредством PDO, независимо от того, принадлежит ли значение числовому или логическому типу данных столбца. Это происходит потому, что некоторые типы столбцов не могут быть представлены с помощью родных типов PHP вследствие ограничения их размера.

Например, значение типа `BIGINT` в MySQL может хранить большие целые числа, которые нельзя представить в виде 32-битного целого в PHP. Из-за этого, PDO и ORM по умолчанию, выбирают безопасное решение и оставляют все значения в виде строк.

Тем не менее, некоторые разработчики могут найти это неожиданным и неудобным. С версии Phalcon 2.0.4, вы можете настроить ОРМ для автоматического приведения типов к соответствующим примитивам PHP, при условии, что это безопасно:

```php
\Phalcon\Mvc\Model::setup(['castOnHydrate' => true]);
```

Таким образом, вы можете использовать операторы строгого сравнения или делать предположения о типе переменных:

```php
$robot = Robots::findFirst();
if ($robot->id === 11) {
    echo $robot->name;
}
```

### Связи с условными операторами

С 2.0.4 вы можете создать отношения, основанные на условных операторах. Phalcon будет заботиться об остальном :)

```php
// Companies have invoices issued to them (paid/unpaid)
// Invoices model
class Invoices extends Phalcon\Mvc\Model
{
    public function getSource()
    {
        return 'invoices';
    }
}

// Companies model
class Companies extends Phalcon\Mvc\Model
{
    public function getSource()
    {
        return 'companies';
    }

    public function initialize()
    {
        // All invoices relationship
        $this->hasMany(
            'id',
            'Invoices',
            'inv_id',
            [
                'alias' => 'invoices',
                'reusable' => true,
            ]
        );

        // Paid invoices relationship
        $this->hasMany(
            'id',
            'Invoices',
            'inv_id',
            [
                'alias'    => 'invoicesPaid',
                'reusable' => true,
                'params'   => [
                    'conditions' => "inv_status = 'paid'"
                ]
            ]
        );

        // Unpaid invoices relationship
        $this->hasMany(
            'id',
            'Invoices',
            'inv_id',
            [
                'alias'    => 'invoicesUnpaid',
                'reusable' => true,
                'params'   => [
                    'conditions' => "inv_status <> 'paid'"
                ]
            ]
        );
    }
}
```

## Обновление/Установка

Данная версия может быть установлена из master ветки. Если у вас не установлен Zephir, выполните следующие команды:

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
