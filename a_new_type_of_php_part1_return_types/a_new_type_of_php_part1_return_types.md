>A new type of PHP, part 1: Return types
http://devzone.zend.com/6620/a-new-type-of-php-part-1-return-types/

Каждый мажорный релиз PHP добавляет ряд новых возможностей, некоторые из которых действительно имеет значение. Для PHP 5.3 - это были пространства имен и анонимные функции. Для PHP 5.4 - качество. Для PHP5.5 - генераторы. Для 5.6 - cписки аргументов переменной длины.

PHP 7 имеет большое количество новшеств и улучшений, делающих жизнь разработчика легче. Но я считаю, что самым важным и долгосрочным изменением является работа с типами. Совокупность этих новых фич изменит взгляд на PHP разработку в лучшую сторону.

Почему поддержка строгой типизации так важна? Она предоставляет программе - компилятору или рантайму и другим разработчикам ценную информацию о том, что вы пытались сделать без необходимости исполнять код. Это дает три типа преимуществ:

1. Становится намного легче сообщать другим разработчикам цель части кода. Это практически как документация, только лучше!
2. Строгая типизация дает коду узкую направленность поведения, что способствует повышению изоляции.
3. Программа читает и понимает строгую типизацию точно также как человек, появляется возможность анализировать код и находить ошибки за вас... прежде чем вы его исполните!

В подавляющем большинстве случаев, явно типизированный код будет легче понять, он лучше структурирован, и имеет меньше ошибок, чем слаботипизированный. Только в небольшом проценте случаев строгая типизация доставляет больше проблем, чем пользы, но не беспокойтесь, в PHP проверка типов все еще является опциональный, также как и раньше. Но на самом деле, ее стоит использовать всегда.

### Возвращаемые типы

Первым дополнением к уже существующей системе типизации будет поддержка возвращаемых типов. Теперь можно указывать в функциях и методах тип возвращаемого значения в явном виде. Рассмотрим следующий пример:

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

В этом довольно приземленном примере у нас есть объект `Employee`, который имеет только одно свойство, содержащее переданный нами почтовый адрес. Обратите внимание на метод `getAddress()`. После параметров функции у нас есть двоеточие и тип. Он является единственным типом, который может принимать возвращаемое значение.

Постфиксный синтаксис для возвращаемых типов может показаться странным для разработчиков, привыкших к C/C++ или Java. Однако, на практике подход с префиксным объявлением не подходит для PHP, т.к. перед именем функции может идти множество ключевых слов. Во избежание проблем с парсером PHP выбрала путь Go, Rust и Scala.

При возврате любого другого типа методом `getAddress()` PHP будет выбрасывать исключение [`TypeError`](http://php.net/manual/en/class.typeerror.php). Даже `null` не будет удовлетворят требованиям типа. Это позволяет нам с абсолютной уверенностью обращаться в `print` к методом объекта `Address`. Мы точно будем знать, что действительно вернется объект именно этого типа, не `null`, не false, не строка или какой-то другой объект. Именно этим обеспечивается безопасность работы и отсутсвие необходимости в дополнительных проверках, что в свою очередь делает наш собственный код чище. Даже если что-то пойдет не так, PHP обязательно предупредит нас.

Но что делать, если у нас менее тривиальный случай и необходимо обрабатывать ситуации, когда нет объекта `Address`? Введем `EmployeeRepository`, логика которого позволяет не иметь записей. Сначала мы добавит классу `Employee` поле ID:


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

А теперь создадим наш репозиторий. (В качестве заглушки добавим фиктивные данные прямо в конструктор, но на практике, конечно же, необходимо иметь источник данных для нашего репозитория).

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

Большинство читателей быстро заметит, что `findById()` имеет баг, т.к. в случае, если мы попросим несуществующий идентификатор сотрудника PHP будет возвращать `null` и наш вызов `getAddress()` умрет с ошибкой "method called on non-object". Но на самом деле ошибка не там. Она заключается в том, что `findById()` должен возвращать сотрудника. Мы указываем возвращаемый тип `Employee`, чтобы было ясно чья это ошибка.

Что же делать, если действительно нет такого сотрудника? Есть два варианта: первый - исключение; если мы не можем вернуть то, что мы обещаем - это повод для особой обработки за пределами нормального течения кода. Другой - указание интерфейса, имплементация которого и будет возвращена (в том числе и "пустая"). Таким образом, оставшаяся часть кода будет работать и мы сможем контролировать происходящее в "пустых" случаях.

Выбор подхода зависит от варианта использования и также определяется рамками недопустимости последствий в случае невозвращения правильного типа. В случае репозитория, я бы поспорил с выбором в пользу исключений, поскольку шансы попасть именно в такую ситуацию минимальны, а работа через исключения является довольно дорогостоящей по производительности. Если бы мы имели дело со скалярной переменной, то обработка "пустого значения" было бы приемлимым выбором. Модифицируем наш код соответственно (для краткости показаны только изменные части):


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

Теперь `getStreet()` будет отдавать хорошее пустое значение.

One other important note about return types: When extending a class, the return type cannot be changed, even to make it more specific (to a subclass for instance).  The reason for that is PHP’s lazy-loaded nature.  The engine doesn’t know what classes are defined when compiling the code, so it doesn’t know if Address really is a specialization of AddressInterface.

Одно важное Примечание о возвращаемых типов: при расширении класса, возвращаемый Тип Не может быть изменен, даже сделать его более конкретным (к подклассу, например). Причина этого в PHP ленивый-загружен природы. Двигатель не знает, что классы определяются при компиляции кода, поэтому она не знаю если Адрес действительно является специализация AddressInterface.

Одно важное примечание о возвращаемых типах: при наследовании тип не может быть изменен, даже нельзя сделать его более конкретным (подклассом, например). Причина в особенностях ленивой загрузки PHP.

Возвращаемые типы являются большой, но далеко не едиственной новой особенностью, расширяющей систему типов PHP. Во второй части мы рассмотрим другое, пожалуй даже более важное изменение: декларирование скалярных типов.

