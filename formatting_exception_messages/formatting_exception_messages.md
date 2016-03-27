>Formatting Exception Messages
http://rosstuck.com/formatting-exception-messages/

За последние пару лет я пришел к тому, что начал раскладывать свои сообщения исключений внутрь статических методовов моих Exception-классов. Врядли это будет чем-то новым, Doctrine так [делает уже лет десять](https://github.com/doctrine/doctrine2/blob/4fc1781d78fab42377fedda843045371b14f8f1e/lib/Doctrine/ORM/ORMException.php). Тем не менее для многих людей это становится открытием, поэтому я решил написать статью, объясняющую что, как и почему.

## Как это работает?

Предположим, вы пишите импорт большого CSV и натыкаетесь на некорректную строку с отсутствующим столбцом. Ваш код может выглядеть следующим образом:

```php
if (!$row->hasColumns($expectedColumns)) {
    throw new Exception("Row is missing one or more columns");
}
```

Эта конструкция отлично работает в плане остановки выполнения программы, но она не слишком гибка для разработчика. Мы можем улучшить ее путем создания своего класса исключения.

```php
class InvalidRowException extends \Exception
{
}
```

Теперь выбрасываем наше пользовательское исключение вместо стандартного:

```php
if (!$row->hasColumns($expectedColumns)) {
    throw new InvalidRowException("Row is missing one or more columns");
}
```

This might look like boilerplate but it allows higher level code to recognize which error was raised and handle it accordingly. For example, we might stop the entire program on a `NoDatabaseConnectionException` but only log an `InvalidRowException` before continuing.

И еще, сообщение об ошибке - не слишком полезная информация с точки зрения отладки. В какой строке произошел сбой? Было бы лучше, если бы мы всегда включали номер строки в наше сообщение об ошибке.

```php
if (!$row->hasColumns($expectedColumns)) {
    throw new InvalidRowException(
        "Row #" . $row->getIndex() . " is missing one or more columns"
    );
}
```

В таком виде логи будут выглядеть лучше, но теперь форматирование этого небольшого сообщения стало немного шумным и отвлекающим. Тут мы сталкиваемся с дилеммой: логи выглядят лучше, но код становится уродливым. Не говоря уже о том, что может быть несколько причин по которым мы могли бы выбрасывать `InvalidRowException`, и нам придется форматировать их все, включая номера строк. Скуууука.

## Инкапсуляция форматирования

Уберем шум путем перемещения форматирования в наш класс исключения. Лучший способ сделать это - через статическую фабрику:

```php
class InvalidRowException extends \Exception
{
    public static function incorrectColumns(Row $row) {
        return new static("Row #" . $row->getIndex() . " is missing one or more columns");
    }
}
```

Теперь можно использовать удобные форматированные сообщения в логах без потери читабельности:

```php
if (!$row->hasColumns($expectedColumns)) {
    throw InvalidRowException::incorrectColumns($row);
}
```

Единственным дополнительным кодом остался блок вызова функции, оборачивающей сообщение. Сама конструкция - не просто шум, она позволяет нам делать typehint и сообщает что необходимо передать для создания красиво оформленного сообщения. И если вдруг требования изменятся, мы сможем использовать инструменты статического анализа для рефакторинга этих конкретных случаев.

Такой подход также освобождает нас от психологического ограничения имеющегося пространства. Больше нет необходимости писать код, который умещается в одно `if`-условие, у нас есть целый метод для всего, что имеет смысл.

Например, распространенные ошибки потребуют более сложный вывод, включающий в себя как ожидаемый, так и полученный список столбцов.

```php
public static function incorrectColumns(Row $row, $expectedColumns)
{
    $expectedList = implode(', ', $expectedColumns);
    $receivedList = implode(', ', $row->getColumns());

    return new static(
        "Row #" . $row->getIndex() . " did not contain the expected columns. " .
        " Expected columns: " . $expectedList .
        " Received columns: " . $receivedList
    );
}
```

Код внутри стал содержать значительно больше логики, но консьюмеру потребуется передать лишь один дополнительный параметр.

```php
if (!$row->hasColumns($expectedColumns)) {
    throw InvalidRowException::incorrectColumns($row, $expectedColumns);
}
```

Его легко использовать, особенно когда мы выбрасываем исключение в нескольких местах, Это именно тот тип сообщений об ошибках, который все хотят читать, но редко бывает возможность писать. Вы можете использовать подобный подход даже в unit-тестах, передавая сообщение об исключительной ситуации, содержащей недостающие имена столбцов. Бонус - используйте `array_diff`/`array_intersect` чтобы показать реальные расхождения в столбцах.

Опять же, это может показаться излишним и я точно не рекомендовал бы полировать до такой степени каждый сценарий обработки ошибок. Но если это код, в котором вы действительно хотите и можете предугадывать простой способ исправления ошибок, затратьте одну лишнюю минуту на написание хорошего сообщения об ошибке, это даст вам хорошие дивиденды в отладке.

## Варианты использования

До этого момента мы делали метод только для одного варианта использовани - для неверного количества столбцов.

Вполне возможно, что есть и другие варианты проблем с нашим CSV-файлом, например, наличие пустых строк. Добавим второй метод к исключению:

```php
class InvalidRowException extends \Exception
{
    // ... incorrectColumns ...

    public static function blankLine($rowNumber)
    {
        return new static("Row #$rowNumber was a blank line.");
    }
}
```

И снова, звучит немного шаблонно, но мы получаем дополнительное пространство для написания более подробных сообщений. Если же проблема продолжает возникать, возможно, стоит добавить некоторые дополнительные детали, ссылки или номера багов.

```php
public static function blankLine($rowNumber)
{
    return new static(
        "Row #$rowNumber was a blank line. This can happen if the user " .
        "exported the source file with incorrect line endings."
    );
}
```

## Locking It Down

When you’re creating “named constructors” for domain objects, you use a similar technique but also declare the original constructor as private. I don’t do this with exceptions though.

First off, we can’t in PHP. Exceptions all extend from a parent Exception class and can’t change their parent’s `__construct` access level from public to private.

More importantly, there’s little benefit for Exceptions. With domain objects, we do this to capture the ubiquitous language and prevent users from instantiating the object in an invalid state. But with an exception, there’s very little state to keep valid. Furthermore, we’re inherently dealing with exceptional circumstances: we can’t foresee every reason a user might throw an exception.

So, the best you can do is create a base factory that you recommend other folks use when creating their exceptions. This can typehint for useful things commonly included in the message:

```php
class InvalidRowException extends \Exception
{
    // One off error messages can use this method...
    public static function create(Row $row, $reason)
    {
        return new static("Row " . $row->getIndex() . " is invalid because: " . $reason);
    }

    // ...and predefined methods can use it internally.
    public static function blankLine(Row $row)
    {
        return static::create($row, "Is a blank line.");
    }
}
```

Which might be useful but is probably pretty far out there. I haven’t seen a convenient way to enforce it though.

## The Big Picture

There’s one final benefit I’d like to touch on.

Normally, when you write your exception messages inline, the various error cases might be split across different files. This makes it harder to see the reasons you’re raising them, which is a shame since exception types are an important part of your API.

When you co-locate the messages inside the exception, however, you gain an overview of the error cases. If these cases multiply too fast or diverge significantly, it’s a strong smell to split the exception class and create a better API.

```php
// One of these isn’t like the others and should probably be a different Exception class
class InvalidRowException extends \Exception
{
    public static function missingColumns(Row $row, $expectedColumns);
    public static function blankLine(Row $row);
    public static function invalidFileHandle($fh);
}
```

Sometimes we underestimate the little things that shape our code. We’d like to pretend that we’re not motivated by getting our error messages neatly on one line or that we regularly do a “Find Usages” to see our custom exception messages, but the truth is: these little details matter. Creating good environments at a high level starts with encouraging them at the lowest levels. Pay attention to what your habits encourage you to do.