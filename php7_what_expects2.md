>What to Expect When You're Expecting: PHP 7, Part 2
https://blog.engineyard.com/2015/what-to-expect-php-7-2


As you probably already know, PHP 7 is a thing, and it’s coming this year! Which makes this as good a time as any to go over what’s new and improved.

In the [first part of this series](/2015/what-to-expect-php-7), we looked at the some of the most important inconsistency fixes coming up in PHP 7 as well as two of the biggest new features. In this post, we take a look another six big features to land in PHP 7 that you’ll want to know about.

## Unicode Codepoint Escape Syntax


The syntax used is `\u{CODEPOINT}`, for example the green heart, 💚, can be expressed as the PHP string: "\u{1F49A}".
The addition of a new escape character, `\u`, allows us to specify Unicode character code points (in hexidecimal) unambiguously inside PHP strings:

<a name="read-more" class="read-more-anchor"></a>

## Null Coalesce Operator

Another new operator, the [Null Coalesce Operator](https://wiki.php.net/rfc/isset_ternary), `??`, is effectively the fabled ifsetor. It will return the left operand if it is not `NULL`, otherwise it will return the right.

The important thing is that it will not raise a notice if the left operand is a non-existent variable. This is like `isset()` and unlike the `?:` short ternary operator.

You can also chain the operators to return the first non-null of a given set:

```php
$config = $config ?? $this->config ?? static::$defaultConfig;
```

## Bind Closure on Call

With PHP 5.4 we saw the addition of `Closure->bindTo()` and `Closure::bind()` which allows you change the binding of `$this` and the calling scope, together, or separately, creating a duplicate closure.

PHP 7 now adds an easy way to do this at call time, binding both `$this` and the calling scope to the same object with the addition of `Closure->call()`. This method takes the object as it’s first argument, followed by any arguments to pass into the closure, like so:

```php
class HelloWorld {
     private $greeting = "Hello";
}

$closure = function($whom) { echo $this->greeting . ' ' . $whom; }

$obj = new HelloWorld();
$closure->call($obj, 'World'); // Hello World
```

## Group Use Declarations

If you’ve ever had to import many classes from the same namespace, you’ve probably been very happy if your IDE will auto-complete them for you. For others, and for brevity, PHP 7 now has [Group Use Declarations](https://wiki.php.net/rfc/group_use_declarations). This allows you to quickly specify many similar imports with better clarity:

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

It can also be used with constant and function imports with use function, and use const, as well as supporting mixed imports:

```php
use Framework\Component\{
     SubComponent\ClassA,
     function OtherComponent\someFunction,
     const OtherComponent\SOME_CONSTANT
};
```

## Generators Improvements

### Generator Return Expressions

There are two new features added to generators. The first is [Generator Return Expressions](https://wiki.php.net/rfc/generator-return-expressions), which allows you to now return a value upon (successful) completion of a generator.

Prior to PHP 7, if you tried to return anything, this would result in an error. However, now you can call `$generator->getReturn()` to retrieve the return value.

If the generator has not yet returned, or has thrown an uncaught exception, calling `$generator->getReturn()` will throw an exception.

If the generator has completed but there was no return, null is returned.

Here’s an example:

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

### Generator Delegation

The second feature is much more exciting: [Generator Delegation](https://wiki.php.net/rfc/generator-delegation). This allows you to return another iterable structure that can itself be traversed — whether that is an array, an iterator, or another generator.

It is important to understand that iteration of sub-structures is done by the outer-most original loop as if it were a single flat structure, rather than a recursive one.

This is also true when sending data or exceptions into a generator. They are passed directly onto the sub-structure as if it were being controlled directly by the call.

This is done using the `yield from <expression>` syntax, like so:

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

On each iteration, this will output:

1.  "Hello"
2.  " "
3.  "World!"
4.  "Goodbye"
5.  " "
6.  "Moon!"

One other caveat worth mentioning is that because the sub-structure can yield it’s own keys, it is entirely possibly that the same key will be returned for multiple iterations — it is your responsibility to avoid that, if it matters to you.

## Engine Exceptions

Handling of fatal and catchable fatal errors has traditionally been impossible, or at least difficult, in PHP. But with the addition of [Engine Exceptions](https://wiki.php.net/rfc/engine_exceptions_for_php7), many of these errors will now throw exceptions instead.

Now, when a fatal or catchable fatal error occurs, it will throw an exception, allowing you to handle it gracefully. If you do not handle it at all, it will result in a traditional fatal error as an uncaught exception.

These exceptions are `\EngineException` objects, and unlike all userland exceptions, do not extend the base `\Exception` class. This is to ensure that existing code that catches the `\Exception` class does not start catching fatal errors moving forward. It thus maintains backwards compatibility.

In the future, if you wish to catch both traditional exceptions _and_ engine exceptions, you will need to catch their new shared parent class, `\BaseException`.

Additionally, parse errors in `eval()`’ed code will throw a `\ParseException`, while type mismatches will throw a `\TypeException`.

Here’s an example:

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

## Coming Soon!

PHP 7.0.0 is only eight months away, making it quite possibly the quickest major version in PHP’s history. While it’s still alpha quality, PHP 7 is shaping up really nicely.

And you can help make it better.

### Test Your Code

Grab Rasmus’s [PHP 7 vagrant box](http://github.com/rlerdorf/php7dev) and start running your test suites, or performing your regular QA. Report [bugs](http://bugs.php.net) to the project, and try again regularly.

### Help GoPHP7-ext

One of the major hurdles for PHP 7 is ensuring that all extensions are updated to work with the new Zend Engine 3.

If you use an extension that isn’t well known, and doesn’t get much love from it’s maintainers – or if you have your own extensions – check out the [GoPHP7-ext project](http://gophp7.org/gophp7-ext/) and get involved in making sure that everything is ready to go on day one.

### Write Documentation

Every new feature in PHP 7 has an RFC. They can all be found on the [PHP.net wiki](http://wiki.php.net/rfc) and are a good starting point for writing new documentation. You can do that [online in a GUI environment](https://edit.php.net), including commiting (if you have karma) or by submitting patches for review.

## In Summary

PHP 7 is going to be _great_!

Go test your applications. Help migrate extensions.

P.S. Are you already playing around with PHP 7? How do you feel about the new features? Is there anything you disagree with, or that you wish had made it in that didn’t? When do you think you’ll be making the switch? Let us know your thoughts!