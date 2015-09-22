>A new type of PHP, part 2: Scalar types
http://devzone.zend.com/6622/a-new-type-of-php-part-2-scalar-types/

В нашей [предыдущей статье](http://habrahabr.ru/post/267257/) мы говорили о преимуществах системы типов PHP 7, и в частности, о новой поддержке типизированных возвращаемых значений. Что само по себе является не только большим подспорьем в поддержке кода, но делает для PHP большой шаг вперед.

До сих пор мы говорили о типах только в отношении классов и интерфейсов. В течение многих лет мы только их (и массивы) и могли использовать. Однако же, PHP 7 добавляет возможность использовать и скалярные величины тоже, такие как `int`, `string` и `float`.

Но постойте. В PHP большинство примитивов являются взаимозаменяемыми. Мы можем передать `"123"` в функцию, которая хочет `int`, и довериться PHP, который все сделает "правильно". Так для чего же тогда нужны скалярные типы?

Также как и возвращаемые типы, скаляры повышают ясность кода, дают возможность поймать больше ошибок на раннем этапе. Что, в свою очередь, повышает надежность кода.

PHP 7 добавляет четыре новых типа, которые могут быть заданы параметрами или возвращаемым значениям: `int`, `float`, `string` и `bool`. Они присоединятся к уже существующим `array`, `callable`, классам и интерфейсам. Давайте дополним наш предыдущий пример с учетом новой возможности:

```php
interface AddressInterface {
  public function getStreet() : string;
  public function getCity() : string;
  public function getState() : string;
  public function getZip() : string;
}
```
```php
class EmptyAddress implements AddressInterface {
  public function getStreet() : string { return ''; }
  public function getCity() : string { return ''; }
  public function getState() : string { return ''; }
  public function getZip() : string { return ''; }
}
```
```php
class Address implements AddressInterface {
  protected $street;
  protected $city;
  protected $state;
  protected $zip;

  public function __construct(string $street, string $city, string $state, string $zip) {
    $this->street = $street;
    $this->city = $city;
    $this->state = $state;
    $this->zip = $zip;
  }

  public function getStreet() : string { return $this->street; }
  public function getCity() : string { return $this->city; }
  public function getState() : string { return $this->state; }
  public function getZip() : string { return $this->zip; }
}
```
```php
class Employee {
  protected $id;
  protected $address;

  public function __construct(int $id, AddressInterface $address) {
    $this->id = $id;
    $this->address = $address;
  }

  public function getId() : int {
    return $this->id;
  }

  public function getAddress() : AddressInterface {
    return $this->address;
  }
}
```
```php
class EmployeeRepository {
  private $data = [];

  public function __construct() {
    $this->data[123] = new Employee(123, new Address('123 Main St.', 'Chicago', 'IL', '60614'));
    $this->data[456] = new Employee(456, new Address('45 Hull St', 'Boston', 'MA', '02113'));
  }

  public function findById(int $id) : Employee {
    if (!isset($this->data[$id])) {
      throw new InvalidArgumentException('No such Employee: ' . $id);
    }

    return $this->data[$id];
  }
}
```
```php
$r = new EmployeeRepository();

try {
  print $r->findById(123)->getAddress()->getStreet() . PHP_EOL;
  print $r->findById(789)->getAddress()->getStreet() . PHP_EOL;
}
catch (InvalidArgumentException $e) {
  print $e->getMessage() . PHP_EOL;
}
```

Изменения коснулись некоторых методов, отдающих строку или число. Даже сделав такой простой шаг, мы уже получили некоторые преимущества.

1. Теперь известно, что различные поля класса `Address` являются просто строками. Раньше можно было только предполагать, что они были строками, а не объектами `Street`  (состоящими из номера улицы, ее названия и номера квартиры) или ID города из базы данных. Конечно, обе эти вещи совершенно разумны в определенных обстоятельствах, но в данной статье они не рассматриваются.
2. Известно, что идентификаторы сотрудников - целые числа. Во многих компаниях ID сотрудника является алфавитно-цифровой строкой или, возможно, номером с лидирующим нулем. Раньше не было способа узнать, теперь же иные трактовки исключены.
3. Плюсом является и безопасность. Мы гарантированно знаем, что внутри `findById()` `$id` - это значение типа `int`. Даже если оно изначально пришло из пользовательского ввода, оно станет целочисленным. Это означает, что оно не может содержать, например, SQL-инъекции. Опора на проверку типов при работе с пользовательским вводом не единственная, и даже не лучшая защита от нападения, но еще один слой защиты.

Кажется, что первые два преимущества избыточны при наличии документации. Если у вас есть хорошие doc-блоки в коде, вы уже знаете, что `Address` состоит из строк и ID сотрудника является целочисленным, верно? Это правда; однако, не все придерживаются фанатичности в вопросе документирования своегокода, или же просто забывают обновить ее. С "активной" информацией из самого языка вы гарантированно будете знать, что рассинхрона нет, ведь PHP выбросит исключение, если это не так.

Явное указание типов также открывает двери к более мощным инструментам. Программы - либо сам PHP, либо инструменты для анализа от третьих лиц - могут изучать исходный код, находя возможные ошибки или возможности оптимизации на основании полученной информации.

Например, мы можем проверить, что следующий код является некорректным и всегда будет падать, даже без его запуска:

```php
function loadUser(int $id) : User {
  return new User($id);
}
```
```php
function findPostsForUser(int $uid) : array {
  // Obviously something more robust.
  return [new Post(), new Post()];
}
```
```php
$u = loadUser(123);

$posts = findPostsForUser($u);
```

Since `loadUser` always returns an object of type user, and `findPostsForUser` always wants an integer, there is no possible way this code is right. We can tell just by looking at the function signatures that it’s wrong, without looking at the code inside.  That means a program can figure that out, too, and warn us before we even try running it. And because a program can keep track of a lot more moving parts than we can it could scan the entire code base and find bugs caused by incompatible types in different parts of the program… all without ever executing it!

This process is called “static analysis”, and is an incredibly powerful way to evaluate programs to locate and fix errors.  It is hampered a bit, though, by PHP’s normal weak typing.  Passing an integer to a function that expects a string, or a string to a function that expects an integer, still works as far as types are concerned because PHP will silently convert between them for us, just as it always has.  That makes static analysis, by us or by a program, less useful.

Enter strict mode, the cornerstone of PHP 7’s new typing system.

By default, with scalar types (either on parameters or return values), PHP will do its best to cast a value to what is expected.  That is, passing an int to a string parameter will work fine, and result in a variable of type string. Passing a boolean value to an int parameter will convert to a 0 or 1 integer, because that’s what the natural thing to do is.  Returning an object that implements `__toString()` when a function says it returns a string will result in, you guessed it, a primitive string.

The one exception is passing a string to an int or float.  Traditionally, when a PHP function expects an int or float and a string is passed, PHP will silently truncate the string at the first non-numeric character resulting in possible data loss.  With scalar types, passing a string to an int or float parameter will work normally if it’s a numeric string, but will trigger an E_NOTICE if values are truncated.  It will still continue, but such data loss is now treated as a minor error condition.

That auto-conversion makes sense in a world where virtually all input is a string (from a database or HTTP request), but does limit the usefulness of type checking.  Instead, PHP 7 offers a strict_types mode.  Its usage is a little subtle and non-obvious, but once understood is incredibly powerful.

To enable strict mode, add a declare statement to the top of a file like so:

```php
declare(strict_types=1);
```

That statement must be the first line in the file, before any executable code. It affects only that file, and only _calls and returns_ in that file.  To understand what that means, let’s break our code from above into a few separate files and tweak it a bit:

```php
// EmployeeRespository.php

class EmployeeRepository {

  private $data = [];

  public function __construct() {
    $this->data[123] = new Employee(123, new Address('123 Main St.', 'Chicago', 'IL', '60614'));
    $this->data[456] = new Employee(456, new Address('45 Hull St', 'Boston', 'MA', 02113));
  }

  public function findById(int $id) : Employee {
    if (!isset($this->data[$id])) {
      throw new InvalidArgumentException('No such Employee: ' . $id);
    }
    return $this->data[$id];
  }
}
```
```php
// index.php

$r = new EmployeeRepository();

try {

  $employee_id = get_from_request_query('employee_id');

  print $r->findById($employee_id)->getAddress()->getStreet() . PHP_EOL;
}
catch (InvalidArgumentException $e) {
  print $e->getMessage() . PHP_EOL;
}
```

What can we guarantee here?  Most importantly, we know, for certain, that `$id` inside a `findById()` is an int, not a string.  It doesn’t matter what $employee_id is, $id will be an int, even if an E_NOTICE may have been thrown.  If we add the strict types declaration to EmployeeRepository.php, therefore, nothing happens. We still have an int inside `findById()`.

However, if we declare `index.php` to use strict mode then the call to `findById()` is affected… in index.php.  If `$employee_id` is a string, or float, or anything other than in int, even if it is an int-like string like “123”, a TypeError will be thrown.

That is, your file will be affected by strict mode if and only if your file declares it wants to be.  Code in any other file is unaffected by whether or not you ask PHP to be pedantic and picky with your code.  Someone else’s desire to ask PHP to be strict and picky will not affect code in your files, only in theirs.

So what good is it? The astute reader will notice I introduced a very subtle bug in the last code sample.  Check out `EntityRespository`‘s constructor, where we create our dummy data.  The second record passes the ZIP code as an int, not as a string.  Most of the time that will make no difference.  However, US ZIP codes in the northeast begin with a leading 0.  If you have an int with a leading 0, PHP will interpret that as an octal number, that is, in base 8.

With weak typing, that means the Boston address will get interpreted not as the ZIP code 02113 but as the integer 02113, which in base 10 is 1099, which PHP will dutifully turn into “1099” as the ZIP code.  Trust me, Bostonians hate that.  Such an error may eventually get caught somewhere in the database or somewhere that enforces that ZIP codes be 6 characters, but by then it’s so far removed you’ll have no idea where 1099 even came from.  Maybe you’ll figure it out, 8 hours of debugging later.

Instead, if we switch EntityRespository.php to strict mode the type mismatch of an int going into a string will get caught for us immediately.  If we try to run the code we’ll get a very specific error that tells us the exact line that we’re passing an int when a string is expected.  And good tooling (either a good IDE or some static analysis program) can catch it even before that.

The one place where strict types mode will allow a variable to be automatically converted is from an int to a float.  That is a safe conversion to make (except with extremely large or small values that have overflow implications, a situation that exists in many languages) and makes logical sense since all integral values are, by definition, floating point values as well.

So when should we use strict typing?  My answer is simple: As often as possible.  Scalar typing, return types, and strict mode offer huge benefits for debuggability, and in driving maintainable code.  They should all be leveraged as much as possible, as they will long-term result in a more robust, maintainable, and bug-free codebase.

The exception would be in code that directly interacts with an input channel, such as the database or incoming HTTP data.  In those cases, the incoming data is always going to be strings because working on the web is simply an over-engineered way of concatenating strings.  While manually casting values from HTTP or a database from a string to the appropriate type is possible, it’s often a lot of unnecessary busy work; if an SQL field is of type int, then you know (even if the computer doesn’t) that the value that comes back from it will be a numeric string and therefore safe from data loss if you pass it to a function expecting an int.

That means in our example, `EmployeeRespository`, `Employee`, `Address`, `AddressInterface`, and `EmptyAddress` should probably all be set to strict types mode.  `index.php`, however, is interacting with the incoming request (by calling `get_from_request_query()`), and so it’s likely easier to let PHP sort out the variable casting rather than doing it all manually ourselves.  Once the values of indeterminate type from the request are passed to a typed function, though, we know what they are and can safely switch into strict mode to keep it that way.

Hopefully, the switch to PHP 7 will be a lot faster than the switch to PHP 5.  It’s a worthwhile switch.  One of the biggest reasons is the expanded type system, which gives us the ability to make our code more self-documenting, self-error-checking, and more easily understood by each other and our tools.  The result will be better code with fewer bugs and fewer “I don’t even know what to do with this” moments than ever before.