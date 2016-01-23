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

#### Use helper functions

When you have applied the rules above and you have reduced the number of branches in your code, you can simplify your functions even more by splitting them into smaller logical units. Group the lines of code that together accomplish a sub-task, separate them by some whitespace. Then figure out how to move them to separate "helper" methods (also known as the "Extract method" refactoring step).

Helper methods are usually `private` methods, only used by objects of this particular class. Often they don't need access to instance variables. In that case they should be `static` methods. In my experience private (static) helper methods often evolve into separate classes with public (static or instance) methods - at least if you test-drive the code of such a _collaborator_ class.

#### Reduce the number of temporary variables

A long function body usually needs several variables to remember intermediate results. These temporary variables are hard to mentally keep track of: you'll have to remember whether they have been initialized with a value already, whether or not they are still needed, and what their current value is.

Introducing helper functions as described in the previous section is one way to get rid of temporary values:

```php
public function capitalizeAndReverse(array $names) {
    $capitalized = array_map('ucfirst', $names);
 
    $capitalizedAndReversed = array_map('strrev', $capitalized);
 
    return $capitalizedAndReversed;
}
```

Using helper methods we don't need any temporary variables anymore:

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

#### Use boolean expressions

The expression used in an `if` statement should result in a boolean value. However, PHP doesn't force _you_ to actually deliver a boolean value:

```php
$a = new \DateTime();
...
 
if ($a) {
    ...
}
```

`$a` will automatically be converted to a boolean. Type coercion has always been a major source of bugs, but it's also more complex to follow along while trying to understand the code, since the conversion is implicit. Instead of relying on PHP to convert `$a` to a boolean, we should do that explicitly, for example:

```php
if ($a instanceof DateTime) {
    ...
}
```

Once you know that you're comparing booleans, don't overdo it, like this:

```php
if ($b === false) {
    ...
}
```

Just use the `!` operator instead:

```php
if (!$b) {
    ...
}
```

#### No Yoda-style expressions

Yoda-style expressions look like this:

```php
if ('hello' === $result) {
    ...
}
```

It is supposed to prevent you from making mistakes like this:

```php
if ($result = 'hello') {
    ...
}
```

In this case `'hello'` will be assigned to `$result`, which will become the value for the entire expression. `'hello'` will automatically be cast to a boolean, in this case `true`. Hence, the code in the `if` clause will always be executed.

Using Yoda-style expressions should help you prevent such mistakes:

```php
if ('hello' = $result) {
    ...
}
```

I think nobody actually makes these mistakes, except if they're still learning the basic syntax of the PHP language. At the same time, Yoda-style expressions have a high price: readability. It's much harder to read and understand these expressions, since they don't mimic natural sentences.

## Дополнительная литература

Если вы хотите подробнее почитать о коде и о том, как сделать его лучше, могу порекомендовать следующие книги:

*   Совершенный код, Стив Макконнелл
*   Чистый код, Роберт Мартин
*   Рефакторинг, Мартин Фаулер

В следующей статье мы рассмотрим один конкретный кейс, усложняющий наш код больше, чем нужно: `null`.
