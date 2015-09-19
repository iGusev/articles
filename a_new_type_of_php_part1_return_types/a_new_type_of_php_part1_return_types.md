>A new type of PHP, part 1: Return types
http://devzone.zend.com/6620/a-new-type-of-php-part-1-return-types/

Каждый мажорный релиз PHP добавляет ряд новых возможностей, некоторые из которых действительно имеет значение. Для PHP 5.3 - это были пространства имен и анонимные функции. Для PHP 5.4 - качество. Для PHP5.5 - генераторы. Для 5.6 - cписки аргументов переменной длины.

PHP 7 имеет большое количество новшеств и улучшений, делающих жизнь разработчика легче. Но я считаю, что самым важным и долгосрочным изменением является работа с типами. Совокупность этих новых фич изменит взгляд на PHP разработку в лучшую сторону.

Почему поддержка строгой типизации так важна? Она предоставляет программе - компилятору или рантайму и другим разработчикам ценную информацию о том, что вы пытались сделать без необходимости исполнять код. Это дает три типа преимуществ:

1. Становится намного легче сообщать другим разработчикам цель части кода. Это практически как документация, только лучше!
2. Строгая типизация дает коду узкую направленность поведения, что способствует повышению изоляции.
3. Программа читает и понимает строгую типизацию точно также как человек, появляется возможность анализировать код и находить ошибки за вас... прежде чем вы его исполните!

В подавляющем большинстве случаев, явно типизированный код будет легче понять, он лучше структурирован, и имеет меньше ошибок, чем слаботипизированный. Только в небольшом проценте случаев строгая типизация доставляет больше проблем, чем пользы, но не беспокойтесь, в PHP проверка типов все еще является опциональный, также как и раньше. Но на самом деле, ее стоит использовать всегда.

### Return types

The first addition to the type system is support for return types.  Functions and methods can now specify the type of value they return explicitly.  Consider the following example:

```php
class Address {

  protected $street;
  protected $city;
  protected $state;
  protected $zip;

  public function __construct($street, $city, $state, $zip) {
    $this->street = $street;
    $this->city = $city;
    $this->state = $state;
    $this->zip = $zip;
  }

  public function getStreet() { return $this->street; }
  public function getCity() { return $this->city; }
  public function getState() { return $this->state; }
  public function getZip() { return $this->zip; }

}

class Employee {
  protected $address;

  public function __construct(Address $address) {
    $this->address = $address;
  }

  public function getAddress() : Address {
    return $this->address;
  }
}

$a = new Address('123 Main St.', 'Chicago', 'IL', '60614');
$e = new Employee($a);

print $e->getAddress()->getStreet() . PHP_EOL;
// Prints 123 Main St.
```

In this rather mundane example, we have an Employee object that has only one meaningful property, a well-formed US postal address.  Note the syntax of the getAddress() method, however.  After the function parameters we have a colon and a type.  That type is the one and only legal type that the return value of that method may have.

The postfix syntax for return types may look odd to developers used to C, C++, or Java, where it comes before the method.  However, as a practical matter that didn’t work for PHP as it would have confused the parser with the many optional keywords that can come before the function name.  Instead, PHP opted for a postfix notation similar to Go, Rust, or Scala.

The result is that we know, as does PHP itself, that what comes back from getAddress() can only ever be an Address object.  Any other option is an error, and will throw a TypeError.  Even null won’t satisfy that requirement; it must be an Address. That’s what makes our print statement safe: We know, with absolute certainty, that the return value is a full and real Address object, not null, not false, not a string, not some other object.  And therefore we can make assumptions about what we can safely do with it, which makes our own code cleaner.  If somehow it’s not, PHP will catch it for us because the author of getAddress() messed up by promising an Address and not coming through.

But what if we have a less trivial case, and there is no Address?  Let’s introduce an EmployeeRepository, which of course may not have a given record.  First we give Employee an ID:

```php
class Employee {

    protected $id;
    protected $address;

    public function __construct($id, Address $address) {

        $this->id = $id;
        $this->address = $address;

    }

    public function getAddress() : Address {
        return $this->address;
    }

}
```

And now create our repository.  (We’re just stubbing out dummy data in the constructor for now, but in practice of course you’d have a database of some sort behind it.)

```php
class EmployeeRepository {

    private $data = [];
    public function __construct() {

        $this->data[123] = new Employee(123, new Address('123 Main St.', 'Chicago', 'IL', '60614'));
        $this->data[456] = new Employee(456, new Address('45 Hull St', 'Boston', 'MA', '02113'));

    }

    public function findById($id) : Employee {
        return $this->data[$id];
    }

}

$r = new EmployeeRepository();

print $r->findById(123)->getAddress()->getStreet() . PHP_EOL;
```

Most readers will quickly notice that findById() is quite buggy, as if we ask for a non-existent Employee ID PHP will return NULL, and then our getAddress() call will die with the dreaded “method called on non-object” error.  But that’s not actually where the bug is.  The bug is that findById() is supposed to return an Employee; if it doesn’t findById() has the bug, not our code.  By specifying a return type of Employee, we make it clear whose bug it is.

What do we do if there really is no such employee?  There’s two options: One is to throw an exception; if we can’t return what we promised we’d return, that’s cause for special handling outside of the normal flow of our code.  The other is to specify an interface to return instead (which you really really should do) and define an “empty” implementation of it.  That way, remaining code will still work (as in, not fatal) and we can control what happens in “empty” cases.

Which approach you take depends on the use case, and what the implications of an object of the correct type not being available are.  In the case of a repository like here, I would argue an exception is preferred as the odds of the code that follows being able to work are minimal.  If we were dealing with a value object with a logical “empty” – say, Address – then an “empty value” object is probably workable.  Let’s modify our code accordingly (Just the parts with changes are shown for brevity):

```php
interface AddressInterface {

    public function getStreet();
    public function getCity();
    public function getState();
    public function getZip();

}

class EmptyAddress implements AddressInterface {

    public function getStreet() { return ''; }
    public function getCity() { return ''; }
    public function getState() { return ''; }
    public function getZip() { return ''; }

}

class Address implements AddressInterface {

    // ...

}

class Employee {

    // ...

    public function getAddress() : AddressInterface {

        return $this->address;

    }

}

class EmployeeRepository {

    // ...

    public function findById($id) : Employee {

        if (!isset($this->data[$id])) {
            throw new InvalidArgumentException('No such Employee: ' . $id);
        }
        return $this->data[$id];
    }

}

try {
    print $r->findById(123)->getAddress()->getStreet() . PHP_EOL;
    print $r->findById(789)->getAddress()->getStreet() . PHP_EOL;
} catch (InvalidArgumentException $e) {
    print $e->getMessage() . PHP_EOL;
}

/* 
 * Prints:
 * 123 Main St.
 * No such Employee: 789
 */
```

Now `getStreet()` will give us a nice empty value, while a missing employee is cause for a freak-out, er, Exception.

One other important note about return types: When extending a class, the return type cannot be changed, even to make it more specific (to a subclass for instance).  The reason for that is PHP’s lazy-loaded nature.  The engine doesn’t know what classes are defined when compiling the code, so it doesn’t know if Address really is a specialization of AddressInterface.

Return types are great, but they’re not the only addition to PHP’s type system. In part 2, we’ll look at the other, arguably more important change: Scalar typing.