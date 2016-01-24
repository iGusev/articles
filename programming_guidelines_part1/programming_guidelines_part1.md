>Programming guidelines - Part 1: Reducing complexity
>https://www.ibuildings.nl/blog/2016/01/programming-guidelines-php-developers-part-1-reducing-complexity

## Введение в серию

PHP - это довольно свободный язык программирования. Он динамичный и довольно снисходительный по отношению к программисту. Следовательно, Вам, как php-разработчику, потребуется много дисциплины, чтобы создавать код правильно. За эти годы я прочитал множество книг по программированию и обсудил стиль кодирования со многими другими разработчиками. Я не вспомню какие правила приходят из каких книг или от каких людей, но эта статья (и последующие) отражает мое видение того, какие правила являются действительно полезными в создании хорошего кода, того кода, который является перспективным, потому что он может быть прочитан и понят достаточно хорошо. Коллеги-разработчики смогут с уверенностью утверждать что он делает, быстро выявлять проблемы и легко использовать его в других проектах.

## Снижение сложности тела функции

Уменьшайте сложность тела метода или функции как можно больше. Меньшая сложность приводит к снижению психической нагрузки для тех, кто читает ваш код. Постарайтесь также уменьшить количество возможных разночтений и недоразумений, пишите код так, чтобы однозначно можно было понять как он работает и как его изменять.

### Уменьшение количества ветвлений в теле функции

Избавляйтесь от `if`, `elseif`, `else` и `switch` настолько, насколько это возможно. Каждый из них создает одну или более возможностей ветвления исполнения программы. Это сделает код более трудным для понимания или проверки, а также усложнит его тестируемость (поскольку каждая из этих ветвей должна быть покрыта тестом). Хорошее решение часто лежит прямо на поверхности, стоит лишь немного подумать.

#### Делегация полномочий ("Сообщай, не спрашивай")

В некоторых случаях уместнее переместить `if` внутрь объекта для облегчения восприятия. Например:

```php
if ($a->somethingIsTrue()) {
    $a->doSomething();
}
```

может быть изменен так:

```php
$a->doSomething();
```

Процесс принятия решения был поглощен методом `doSomething()` объекта `$a`. Мы никогда не должны "думать" за него, необходимо иметь возможность вызывать `doSomething()` в любой ситуации и иметь гарантии того, что все будет работать. Данный подход основан на принципе ["Tell, don't ask"](http://martinfowler.com/bliki/TellDontAsk.html). Я рекомендую вам разобраться с ним и применять всякий раз, когда вы спрашиваете у объекта некоторую информацию и принимаете решения на ее основе.

#### Использование маппинга

Иногда количество `if`, `elseif` или `else` можно уменьшить путем введения маппинга. Пример:

```php
if ($type === 'json') {
    return $jsonDecoder->decode($body);
} elseif ($type === 'xml') {
    return $xmlDecoder->decode($body);
} else {
    throw new \LogicException(
        'Type "' . $type . '" is not supported'
    );
}
```

можно сократить до:

```php
$decoders = ...; // a map of type (string) to corresponding Decoder objects
 
if (!isset($decoders[$type])) {
    throw new \LogicException(
        'Type "' . $type . '" is not supported'
    );
}
 
return $decoders[$type]->decode($body);
```

Использование маппинга также делает ваш код [открытым для расширения и закрытым для модификаций](http://www.objectmentor.com/resources/articles/ocp.pdf).

#### Использование типов

Много `if` можно будет убрать, если мы будем использовать типизацию более серьезно. Например:

```php
if ($a instanceof A) {
    // happy path
    return $a->someInformation();
} elseif ($a === null) {
    // alternative path
    return 'default information';
}
```

можно существенно упростить путем форсированного слежения за тем, что `$а` является экземпляром `A`.

```php
return $a->someInformation();
```

Конечно, мы должны поддерживать случай с `null`. Но об этом мы поговорим в следующей статье.

#### Ранний `return`

Часто ветка в теле функции  на самом деле является пре- или иногда пост-условием, таким как:

```php
// pre-condition
if (!$a instanceof A) {
    throw new \InvalidArgumentException(...);
}
 
// happy path
return $a->someInformation();
```

В этом `if` нет функциональной логики, в большинстве случаев исполнение в нее не заходит. Это всего лишь проверка на условие. Иногда мы можем делегировать эту проверку самому PHP (т.е. использовать надлежащий type-hinting). Однако же, PHP не сможет проверить все возможные предпосылки, поэтому мы по-прежнему должны иметь некоторые проверки в нашем коде. Для уменьшения сложности мы должны постараться сделать `return` как можно раньше, в момент наличия минимально-необходимых данных для принятия подобного решения.

Данным приемом мы добъемся визуального эффекта, отделяющего ваш код для "правильного пути" от проверок состояния:

```php
// check precondition
if (...) {
    throw new ...();
}
 
// return early
if (...) {
    return ...;
}
 
// happy path
...
 
return ...;
```

Следуя этому шаблону вы сильно повысите читатемость и понимаемость кода.

### Создание небольших логических частей

If a function body is quite large, it will be hard to understand what is going on inside it. Keeping track of variables, their types, their lifecycle, the (helper) functions that are being called, etc. takes a lot of our mental energy. When trying to understand how a function works, it helps a lot when it is small (i.e. it accepts some input, transforms that input in some way, then returns it).

Если в теле функции довольно большие, трудно будет понять, что происходит внутри него. Отслеживание переменных, их типы, их жизненного цикла, (вспомогательные) функции, которые в настоящее время называются, и т. д. уходит много нашей психической энергии. При попытке понять, как работает функция, это очень помогает, когда он маленький (т. е. он принимает некоторый входной сигнал, преобразует входной сигнал в определенную сторону, потом возвращает его).

Если тело функции достаточно большое, трудно будет понять что происходит внутри. Отслеживание переменных, их типов, жизненного цикла, (вспомогательных) функций, которые в настоящее время вызываются и т.д. Уходит много усилий чтобы разобраться со всем этим. Когда функция маленькая, проще понять как она работает: принимает некоторый входной сигнал, преобразовывает его и возвращает результат. 

#### Использование вспомогательных функций

С помощью вышеописанных правил вы сократили количество ветвлений вашего кода, но его еще нельзя назвать хорошим. Разделите ваш код на более мелкие логические единицы. Сгруппируйте строки, отделив их несколькими пустыми линиями, так, чтобы они решали общую задачу. Затем повторяющийся код выделите в отдельные "вспомогательные" методы (Это правило также известно как один из шагов рефакторинга - "Extract method").

Вспомогательные методы, как правило, являются "приватными" и используются только объектами данного класса. Зачастую им не нужен доступ к переменным экземпляра объекта. В таком случае их следует делать "статическими". По моему опыту приватные (статические) вспомогательные методы часто эволюционируют в отдельные классы с публичными интерфейсами.

#### Уменьшение количества временных переменных

Длинное тело функции обычно содержит в себе несколько переменных для запоминания промежуточного результата. Они тяжело отслеживаемы и придают коду запутанности: вам придется вспомнить как они были инициализированы, со значением или без, действительно ли по-прежнему необходимы и что содержат сейчас.

Ввод вспомогательных функций из предыдущего раздела является одним из способов избавиться от временных значений:

```php
public function capitalizeAndReverse(array $names) {
    $capitalized = array_map('ucfirst', $names);
 
    $capitalizedAndReversed = array_map('strrev', $capitalized);
 
    return $capitalizedAndReversed;
}
```

При использовании вспомогательных методов нам не нужны больше временные переменные:

```php
public function capitalizeAndReverse(array $names) {
    return self::reverse(
        self::capitalize($names)
    );
}
 
private static function reverse(array $names) {
    return array_map('strrev', $names);
}
 
private static function capitalize(array $names) {
    return array_map('ucfirst', $names);
}
```

As you can see we are basically composing functions into new functions, thereby making it easier to understand what's going on and much easier to make changes. In a way, this code is a bit more "open/closed" now, since we will never have to modify the code of the helper functions anymore.

Since many algorithms require you to loop over a collection of things, thereby creating a new collection or reducing the collection to a single value, it makes sense to make the collection itself a "first-class citizen" and move related behavior to it:

```php
class Names
{
    private $names;
 
    public function __construct(array $names)
    {
        $this->names = $names;
    }
 
    public function reverse()
    {
        return new self(
            array_map('strrev', $names)
        );
    }
 
    public function capitalize()
    {
        return new self(
            array_map('ucfirst', $names)
        );
    }
}
 
$result = (new Names([...]))->capitalize()->reverse();
```

This makes it even easier to "compose" the functions.

While reducing the number of temporary variables usually leads to a better design, as in the example above, you don't have to eliminate all temporary variables. Sometimes they serve their purpose well in documenting what the outcome of an expression represents.

### Use single types

Keeping track of variables and their current values is pretty hard. It gets even harder when it's not entirely clear what the type of a variable is. And it gets more painful if that type is not consistent.

#### Arrays with just a single type of value

Whenever you use an array as a traversable collection, make sure you use it with one type of value. This will reduce the complexity in the loop that's going to read out the values:

```php
foreach ($collection as $value) {
    // no need to do any checks if we know the type of $value
}
```

Supporting the reader of your code as well as your editor, you should always type-hint the collection:

```php
/**
 * @param DateTime[] $collection
 */
public function doSomething(array $collection) {
    foreach ($collection as $value) {
        // $value is an instance of DateTime
    }
}
```

Since you can't be absolutely certain that `$value` is something else than a `DateTime` instance, you should add a pre-condition for it in the function body. The [beberlei/assert](https://github.com/beberlei/assert) library will make that very easy:

```php
use Assert\Assertion
 
public function doSomething(array $collection) {
    Assertion::allIsInstanceOf($collection, \DateTime::class);
 
    ...
}
```

If the collection contains a value that is not an instance of `DateTime`, this will throw an `InvalidArgumentException`. Besides enforcing correct input values, using assertions also reduces the complexity of your code, since you won't have to do the type checking in your head.

#### Single type of return value

Whenever a function could return different types of values, this greatly increases complexity in the client code:

```php
$result = someFunction();
if ($result === false) {
    ...
} elseif (is_int($result)) {
    ...
}
```

PHP will not prevent you from having "mixed" return types (or parameter types for that matter). They are greatly confusing though and your application will be full of `if` statements to take into account all the possible outcomes.

A more everyday example of mixed return types is this:

```php
/**
 * @param int $id
 * @return User|null
 */
public function findById($id)
{
    ...
}
```

This function will return either a `User` object or `null`, which is very problematic since now we won't be able to call any method on the return value of this method without first checking if it's indeed a `User` object. In fact PHP, before version 7, just crashed with a "Fatal error" in case you'd (accidentally) do that.

We will consider `null` values and how to get rid of them in the next article.

### Readable expressions

We have discussed many options for reducing the overall complexity of your functions. At the smallest level there are still some things we can do to make our code even less complex.

#### Hide complex logic

You can often move a complicated part of an expression to a "helper" method. Instead of

```php
if (($a || $b) && $c) {
    ...
}
```

you could have something much better, like this:

```php
if (somethingIsTheCase($a, $b, $c)) {
    ...
}
```

For the reader it is now clear that the outcome of the decision depends on `$a`, `$b` and `$c`, and the function name communicates what that decision is all about.

#### Логические выражения

Выражение, используемое в операторе `if` должно приводить к логическому значению. Однако, на самом деле PHP не заставляет *вас* делать это:

```php
$a = new \DateTime();
...
 
if ($a) {
    ...
}
```

`$a` будет автоматически преобразовано в `boolean`. Это и является основным источником ошибок. Из-за неявного преобразования такие ошибки сложнее найти. Вместо того чтобы полагаться на PHP преобразование, сделаем проверку более явной:

```php
if ($a instanceof DateTime) {
    ...
}
```

Но не переусердствуйте при сравнении логических переменных, таких как:

```php
if ($b === false) {
    ...
}
```

Вместо этого просто воспользуйтесь оператором `!`:

```php
if (!$b) {
    ...
}
```

#### Не используйте Йода-стиль

Выражения в Йода-стиле выглядят так:

```php
if ('hello' === $result) {
    ...
}
```

Предполагается, что они помогут вам избежать подобных ошибок:

```php
if ($result = 'hello') {
    ...
}
```

В данном случае переменной `$result` будет присвоено значение `'hello'`, оно же, в свою очередь, автоматически приведется к boolean значению `true`. Следовательно, код в конструкции `if` будет всегда выполняться.

Использование Йода-стиля должно помочь вам предотвратить такие ошибки:

```php
if ('hello' = $result) {
    ...
}
```

Я думаю, что никто на самом деле не делает подобных ошибок, за исключением изучающих базовый синтаксис PHP. В то же время, Йода-стиль имеет высокую цену: читаемость. Его гораздо труднее читать и воспринимать выражения, поскольку они не имитируют природные предложения.

## Дополнительная литература

Если вы хотите подробнее почитать о коде и о том, как сделать его лучше, могу порекомендовать следующие книги:

*   Совершенный код, Стив Макконнелл
*   Чистый код, Роберт Мартин
*   Рефакторинг, Мартин Фаулер

В следующей статье мы рассмотрим один конкретный кейс, усложняющий наш код больше, чем нужно: `null`.
