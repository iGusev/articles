>What to Expect When You're Expecting: PHP 7, Part 2
https://blog.engineyard.com/2015/what-to-expect-php-7-2


Как вы наверное уже знаете, PHP7 придет в этом году! И сейчас как раз самое время узнать что же он нам принесет.

В [первой части данной серии статей](http://habrahabr.ru/post/257237/) мы рассмотрели некоторые наиболее важные изменения в PHP7 и две крупные новые возможности. В этой статье мы рассмотрим еще шесть, о которых вы точно захотите узнать.

## Новый экранирующий символ для Unicode

Добавление нового escape-символа `\u` позволяет нам указываеть специфические unicode символы внутри PHP-строк _(да-да, те самые emoji)_.

Синтаксис выглядит так - `\u{CODEPOINT}`, например, зеленое сердце, 💚, может быть выражено как PHP-строка: `"\u{1F49A}"`.

<habracut/>

## Оператор объединения со значеним NULL ([Null Coalesce Operator](https://wiki.php.net/rfc/isset_ternary))

Еще один новый оператор - `??`. Он возвращает левый операнд, если этот операнд не имеет значение NULL; в противном случае возвращается правый операнд.

Самое главное, что он не генерирует уведомление, если левый операнд является несуществующей переменной. В отличии от короткого тернарного оператора `?:`, он работает как `isset()`.

Вы также можете использовать цепочки операторов, чтобы вернуть первый ненулевой из данного набора:

```php
$config = $config ?? $this->config ?? static::$defaultConfig;
```

## Привязка замыканий во время вызова

С PHP5.4 к нам пришли нововведения `Closure->bindTo()` и `Closure::bind()`, которые позволяют изменить привязку `$this` и области вызова, вместе, или по отдельности, создавая дубликат замыкания.

PHP7 теперь добавляет легкий способ сделать это прямо во время вызова, связывая `$this` и область вызова с помощью `Closure->call()`. Этот метод принимает объект в качестве своего первого аргумента, а затем любые другие аргументы, которые пойдут в замыкание:

```php
class HelloWorld {
     private $greeting = "Hello";
}

$closure = function($whom) { echo $this->greeting . ' ' . $whom; }

$obj = new HelloWorld();
$closure->call($obj, 'World'); // Hello World
```

## Группировка деклараций `use`

Если вам когда-либо приходилось импортировать много классов из одного и того же пространства имен, вы, наверное, были очень счастливы, когда IDE делала всю основную работу за вас. Для всех остальных, и для краткости, в PHP7 теперь есть возможность [группировать декларирование операторов `use`](https://wiki.php.net/rfc/group_use_declarations). Это позволит быстрее и удобнее работать с большим количеством импортов и сделает код читаемее:

```php
// Original
use Framework\Component\SubComponent\ClassA;
use Framework\Component\SubComponent\ClassB as ClassC;
use Framework\Component\OtherComponent\ClassD;
```

```php
// With Group Use
use Framework\Component\{
     SubComponent\ClassA,
     SubComponent\ClassB as ClassC,
     OtherComponent\ClassD
};
```

Группировка может использоваться с константами и импортируемыми функциями, вы можете смешивать все вместе:

```php
use Framework\Component\{
     SubComponent\ClassA,
     function OtherComponent\someFunction,
     const OtherComponent\SOME_CONSTANT
};
```

## Улучшение генераторов

### `return` в генераторах

Есть две новых фичи, добавленных для генераторов. Первая - [Generator Return Expressions](https://wiki.php.net/rfc/generator-return-expressions), которая позволяет возвращать значение после (успешного) завершения работы генератора.

До PHP7, если вы пытались что-нибудь вернуть в генераторе, это приводило к ошибке. Однако, теперь вы можете вызвать `$generator->getReturn()`, чтобы получить возвращаемое значение.

Если генератор еще завершился или выбросил непойманное исключение, вызов `$generator->getReturn()` сгенерирует исключение.

Если же генератор завершен, но не объявлен `return`, то метод вернет `NULL`.

Пример:

```php
function gen() {
    yield "Hello";
    yield " ";
    yield "World!";

    return "Goodbye Moon!";
}

$gen = gen();

foreach ($gen as $value) {
    echo $value; 
}

// Outputs "Hello" on iteration 1, " " on iterator 2, and "World!" on iteration 3

echo $gen->getReturn(); // Goodbye Moon!
```

### Делегирование генератора

Вторая особенность является гораздо более захватывающей: [делегирование генератора](https://wiki.php.net/rfc/generator-delegation). Это позволяет вернуть другую итерабельную структуру - будь то массив, итератор или другой генератор.

Важно понимать, что итерации под-структур осуществляются именно через самую внешнюю петлю, как если бы это была одна плоская структура, а не рекурсивные вызовы.

Это утверждение также справедливо при отправке данных в генератор или выбросе исключений. Они передаются в суб-структуру, как если бы это был ее непосредственный вызов.

Синтаксис такой - `yield from <expression>`. Посмотрим на примере:

```php
function hello() {
     yield "Hello";
     yield " ";
     yield "World!";

     yield from goodbye();
}

function goodbye() {
     yield "Goodbye";
     yield " ";
     yield "Moon!";
}

$gen = hello();
foreach ($gen as $value) {
     echo $value;
}
```

При каждой итерации будет выводиться:

1.  "Hello"
2.  " "
3.  "World!"
4.  "Goodbye"
5.  " "
6.  "Moon!"

Стоит упомянуть еще один ньюанс: поскольку суб-структуры привносят свои собственные ключи, вполне возможно, что один и тот же ключ будет возвращен за несколько итераций. Недопущение подобного - это ваша ответственность, конечно же, если для вас это важно.

## Engine исключения

Обработка фатальный и catchable фатальных ошибок в PHP традиционна была невозможна, или по крайней мере очень сложна. Но с добавлением [Engine исключений](https://wiki.php.net/rfc/engine_exceptions_for_php7), многие из этих ошибок будут теперь выбрасывать исключение вместо самой ошибки.

Теперь, когда фатальная или catchable фатальная неустранимая ошибка возникнут, выбросится исключение, позволяющее обработать ошибку корректно. Если его не трогать, то это приведет к традиционной фатальной ошибке необработанного исключения.

Эти исключения являются `\EngineException` объектами, и в отличии от всех пользовательских исключений, они не наследуются от базового класса `\Exception`. Это сделано специально, чтобы существующий код, который ловит класс `\Exception` не отлавливал и фатальные ошибки, изменяя свое поведение. Таким образом сохраняется обратная совместимость.

В будущем, если вы хотите поймать как традиционные исключения, так и engine-исключения, вам нужно будет отлавливать их новый общий родительский класс `\BaseException`.

Кроме того, ошибки парсинга в выполняемом функцией `eval()` коде теперь будут выбрасывать `\ParseException`, а несоответствие типов приведет к `\TypeException`.

Пример:

```php
try {
    nonExistentFunction();
} catch (\EngineException $e) {
     var_dump($e);
}

object(EngineException)#1 (7) {
  ["message":protected]=>
  string(32) "Call to undefined function nonExistantFunction()"
  ["string":"BaseException":private]=>
  string(0) ""
  ["code":protected]=>
  int(1)
  ["file":protected]=>
  string(17) "engine-exceptions.php"
  ["line":protected]=>
  int(1)
  ["trace":"BaseException":private]=>
  array(0) {
  }
  ["previous":"BaseException":private]=>
  NULL
}
```

## Скоро!

РНР 7.0.0 исполнилось всего восемь месяцев, и, вполне возможно, это будет самым быстрым релизом мажорной версии в истории PHP. Пока все еще в альфа-версии, но уже сейчас все складывается очень хорошо.

И вы можете помочь сделать еще лучше.

### Проверь свой код

Возьмите [PHP7 vagrant box](http://github.com/rlerdorf/php7dev) от Расмуса и запустите свои тесты или проверьте по своему чек-листу ваше приложение. Сообщите о [багах](http://bugs.php.net) в проект, повторяйте регулярно :)

### Помоги GoPHP7-ext

One of the major hurdles for PHP 7 is ensuring that all extensions are updated to work with the new Zend Engine 3.

If you use an extension that isn’t well known, and doesn’t get much love from it’s maintainers – or if you have your own extensions – check out the [GoPHP7-ext project](http://gophp7.org/gophp7-ext/) and get involved in making sure that everything is ready to go on day one.

### Документация

Every new feature in PHP 7 has an RFC. They can all be found on the [PHP.net wiki](http://wiki.php.net/rfc) and are a good starting point for writing new documentation. You can do that [online in a GUI environment](https://edit.php.net), including commiting (if you have karma) or by submitting patches for review.

## Заключение

PHP 7 is going to be _great_!

Протестируйте ваши приложения. Помогите перенести расширения

P.S. Are you already playing around with PHP 7? How do you feel about the new features? Is there anything you disagree with, or that you wish had made it in that didn’t? When do you think you’ll be making the switch? Let us know your thoughts!