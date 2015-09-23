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

`loadUser()` всегда возвращает объект типа `User`, а `findPostsForUser()` всегда возвращает integer, нет никакой возможности сделать этот код верным. Об этом можно сказать лишь взглянув на функции и способ их использования. А это, в свою очередь, означает, что и IDE тоже знает заранее и может предупредить нас об ошибке до запуска. И поскольку IDE может отслеживать намного больше частей, чем мы, то она может и предупреждать о большем количестве ошибок, чем можно заметить самостоятельно... при этом не исполняя код!

Этот процесс называется "статический анализ", и он является невероятно мощным способом оценки программ с целью поиска и исправления ошибок. Этому немного мешает стандартная слабая типизация PHP. Передача целого числа в функцию, которая ожидает строку, или строку в функцию, ожидающую целое, все это продолжает работать и PHP молча конвертирует примитивы между собой, также как и всегда. Что делает статический анализ, нами или с помощью утилит, менее полезным. 

Введение режима строгой типизации является краеугольным камнем новой системы типов PHP 7.

По умолчанию, при работе со скалярными типами (параметрами или возвращаемыми значениями), PHP будет делать все возможное, чтобы привести значение к ожидаемому. То есть, передача `int` в функцию, ожидающую `string` будет прекрасно работать, а передавая `bool` при ожидаемом `int` вы получите целое число 0 или 1, потому что это естественное, ожидаемое от языка поведение. У объекта, переданного в фунцию, ожидающую `string`, будет вызваться `__toString()`, тоже самое произойдет и с возвращаемыми значениями.

Единственным исключением является передача строки в ожидаемый `int` или `float`. Традиционно, когда функция расчитывает получить значения `int`/`float`, а передается `string`, PHP просто полча будет обрезать строку до первого нечислового символа, в результате чего возможно потеря данных. В случае со скалярными типами, параметр будет работать нормально, если строка действительно является числовой, но если же значение будет усекаться, то это приведет к вызову `E_NOTICE`. Все будет работать, но на текущий момент такая ситуация рассматривается как незначительная ошибка в условии.

Авто-преобразование имеет смысл, в тех случаях, когда практически все входные данные передаются как строки (из базы данных или http-запросы), но в то же время оно ограничивает полезность проверки типов. Как раз для этого в PHP 7 прелагается `strict_types` режим. Его использование является несколько тонким и неочевидным, но при должном понимании разработчик получает невероятно мощный инструмент.

Чтобы включить режим строгой типизации, добавьте объявление в начало файла, вот так:

```php
declare(strict_types=1);
```

Это объявление должно быть первой строкой в файле, до выполнения какого-либо кода. Оно затрагивает только логику, расположенную в файле и только _вызовы и возвращамые значения_ в этом файле. Чтобы понять как работает `strict_types`, давайте разобьем наш код на несколько отдельных файлов и немного его изменим:

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