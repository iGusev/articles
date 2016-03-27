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

## Moving the Formatting

We can remove the noise by pushing the formatting into the custom Exception class. The best way to do this is with a static factory:

```php
class InvalidRowException extends \Exception
{
    public static function incorrectColumns(Row $row) {
        return new static("Row #" . $row->getIndex() . " is missing one or more columns");
    }
}
```

And now we can clean up the importing code without losing readability:

```php
if (!$row->hasColumns($expectedColumns)) {
    throw InvalidRowException::incorrectColumns($row);
}
```

The only extra code is the function block surrounding our message. That function block isn’t just noise though: it allows us to typehint and document what needs to be passed to generate a nicely formatted message. And if those requirements ever change, we can use static analysis tools to refactor those specific use cases.

This also frees us from the mental constraints of the space available. We’re not bound to writing code that fits into a single if clause, we have a whole method to do whatever makes sense.

Maybe common errors warrant more complex output, like including both the expected _and_ the received list of columns.

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

The code here got significantly richer but the consuming code only needed to pass one extra parameter.

```php
if (!$row->hasColumns($expectedColumns)) {
    throw InvalidRowException::incorrectColumns($row, $expectedColumns);
}
```

That’s easy to consume, especially when throwing it in multiple locations. It’s the type of error messages everyone wants to read but rarely take the time to write. If it’s an important enough part of your Developer Experience, you can even unit test that the exception message contains the missing column names. Bonus points if you array_diff/array_intersect to show the actual unexpected columns.

Again, that might seem like overkill and I wouldn’t recommend gold plating every error scenario to this extent. Still, if this is code you really want to own and you can anticipate the common fix for these errors, spending 1 extra minute to write a solid error message will pay big in debugging dividends.

## Multiple Use Cases

So far we’ve created a method for one specific use case, an incorrect number of columns.

Maybe we have other issues with our CSV file, like the existence of blank lines. Let’s add a second method to our exception:

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

Again, a bit of boilerplate but we get extra space to write more detailed messages. If the same issue keeps occurring, perhaps it’s worth adding some extra details, links or issue numbers (or you know, fixing it more permanently).

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