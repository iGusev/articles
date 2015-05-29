>Comparing the PHP 7 and Hack Type Systems
>http://www.dmiller.io/blog/2015/4/26/comparing-the-php7-and-hack-type-systems


One of the exciting things about PHP 7, aside from the incredible performance improvements, is the introduction of [scalar type hinting](https://wiki.php.net/rfc/scalar_type_hints_v5) coupled with an optional "strict" mode. When reading the RFC I noticed that PHP 7 code written with type hinting begins to look a lot like [Hack](http://hacklang.org/). I wanted to find out if you could execute the same code in PHP 7 and Hack, and what the differences in execution might be. Here's what I found out.

## The Setup

Just to get this out of the way:

```bash
$ php --version
PHP 7.0.0-dev (cli) (built: Apr 23 2015 01:12:36) (DEBUG)
Copyright (c) 1997-2015 The PHP Group
Zend Engine v3.0.0-dev, Copyright (c) 1998-2015 Zend Technologies
with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2015, by Zend     Technologies
```

```bash
$ hhvm --version
HipHop VM 3.8.0-dev (rel)
Compiler: heads/master-0-gd71bec94dedc8ca2e722f5619f565a06ef587efc
Repo schema: fa9b8305f616ca35f368f3c24ed30d00563544d1
```

In order to execute the PHP code in HHVM without modifying the opening tags in the source code files I had to execute hhvm with the `-vEval.EnableHipHopSyntax=true` flag set.

## Some Examples

Let's look at a simple example.

```php
declare(strict_types=1);

function myLog(string $message): string {
	return $message;
}

function add(int $a, int $b): int {
    myLog($a + $b);
    return $a + $b;
}

$result = add(1, 3);
echo $result;
```

Executing this in PHP 7 returns:

```bash
Fatal error: Argument 1 passed to myLog() must be of the type string, integer given, called in /home/vagrant/basic/main.php on line 9 and defined in /home/vagrant/basic/main.php on line 4
```

Looks good! PHP 7 is correctly telling us that we're passing an integer (`$a + $b`) into a function that is expecting a string and throws an appropiate error. Let's see what HHVM says:

```bash
Catchable fatal error: Argument 1 passed to myLog() must be an instance of string, int given in /home/vagrant/basic/main.php on line 6
```

There are a couple differences evident here:

*   HHVM calls this a "catchable" fatal error. This is interesting as in the [RFC](https://wiki.php.net/_export/code/rfc/scalar_type_hints_v5?codeblock=9) the error shown actually matches HHVM's error.
*   HHVM says the error occurred on line 6 where PHP says it occurred on line 9\. I prefer HHVM's approach here as it shows us where we called the function with the bad data, not where the function is defined that we are calling with bad data. **UPDATE**: I was confused, PHP 7 is the one telling us more information here. I always like more information with my errors. :) Thanks commenters! Here's another example that illustrates an important difference between PHP 7 and Hack.

```
<?hh
declare(strict_types=1);

function myLog(string $message=null): string {
  if ($message === null) {
    return '';
  } else {
    return $message;
  }
}

echo myLog("Hello world!\n");

echo myLog();
```

When executed in PHP this function happily executes. When executed in Hack we get this type error:

```bash
/home/vagrant/nullable/main.php:4:16,21: Please add a ?, this argument can be null (Typing[4065])
```

Hack doesn't allow default arguments with a value of null as it "conflates the concept of an optional argument with that of a required argument that allows a placeholder value" (See O'Reilly's new book _[Hack and HHVM](http://shop.oreilly.com/product/0636920037194.do)_). Instead, Hack recommends that you make such an argument nullable in addition to providing the default value, like so:

```
<?hh
declare(strict_types=1);

function myLog(?string $message=null): string {
  if ($message === null) {
    return '';
  } else {
    return $message;
  }
}

echo myLog("Hello world!\n");

echo myLog();
```

Let's try something a bit more complicated. What happens if we mix strict and non-strict files in PHP? Note that defining strict mode at the top of the file has no effect in HHVM.

```
<?php

function add(int $a, int $b): int {
    myLog($a + $b);
    return $a + $b;
}
```

```
<?php
declare(strict_types=1);

function myLog(string $message): string {
  return $message;
}
```

```
<?php

require 'add.php';
require 'logger.php';

$result = add(1, 3);
echo $result;
```

```bash
$ php main.php
4
 
$ hhvm -vEval.EnableHipHopSyntax=true main.php
Catchable fatal error: Argument 1 passed to myLog() must be an instance of string, int given in /home/vagrant/separate_files_mixed/logger.php on line 6
```

logger.php is defined as being in strict mode, yet PHP allows us to pass an int into it from a non-strict mode file. HHVM throws an exception in the same scenario. What happens if we make add.php strict?

```bash
Fatal error: Argument 1 passed to myLog() must be of the type string, integer given, called in /home/vagrant/separate_files_mixed/add.php on line 5 and defined in /home/vagrant/separate_files_mixed/logger.php on line 4
```

That's better. So it looks like functions defined in a strict file are only strictly type checked if the calling code is also defined in a strict file. On the flip side, what happens if we call a non-strict function that is type annotated from a strict function? To test this I changed logger.php to be non-strict and made add.php strict:

```bash
Fatal error: Argument 1 passed to myLog() must be of the type string, integer given, called in /home/vagrant/separate_files_mixed/add.php on line 5 and defined in /home/vagrant/separate_files_mixed/logger.php on line 3
```

So it appears that functions are only strictly type checked if they are called from a function that is defined in a file declared as strict. However, this only affects direct child calls of the file declared as strict. If we declare main.php strict, PHP happily returns 4 despite the mismatched type we are passing into log().

In Hack this relationship is reversed. If HHVM executes main.php in non-strict mode, and logger is [written in Hack](https://gist.github.com/jazzdan/fe0648a6848dadda5039) (with a hh tag at the top of the file) we still get a type error despite the fact that the file making the call is not written in Hack.

```bash
Catchable fatal error: Argument 1 passed to myLog() must be an instance of string, int given in /home/vagrant/separate_files_mixed/logger.php on line 5
```

Another interesting difference between Hack and PHP's type system comes from PHP's handling of the float annotation. Take this code example:

```
<?php
declare(strict_types=1);

function add(float $a, float $b): float {
    return $a + $b;
}

echo add(1, 2);
```

When executed in PHP this returns '3' even though we are passing ints where we have annotated a float, and despite the fact that strict mode is enabled. The reason for this is that [widening primative conversion](http://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.2) is supported in PHP 7's strict mode. This means that parameters annotated as float can accept an int as ([almost](https://wiki.php.net/rfc/scalar_type_hints_v5#int_-_float_conversion_isn_t_lossless)) any int can be safely converted to a float. HHVM does not support this and will throw a type error when the above code is executed:

```
Catchable fatal error: Argument 1 passed to add() must be an instance of float, int given in /home/vagrant/main.php on line 6
```

If there are any other big differences I've missed, or other scenarios I should enumerate here, please let me know in the comments. I would love to keep exploring!

## Wrapping It Up

While there are many features in Hack that PHP 7 does not support ([nullable](http://docs.hhvm.com/manual/en/hack.nullable.php), [mixed](http://docs.hhvm.com/manual/en/hack.annotations.mixedtypes.php) types, void return types, [collections](http://docs.hhvm.com/manual/en/hack.collections.php), [async](http://docs.hhvm.com/manual/en/hack.async.php), etc) I am excited by the safety and readibility PHP 7's new strict mode enables.

After writing a couple small programs in Hack I've realized that it's not types themselves that make writing Hack enjoyable: it's the tight feedback loop Hack creates between the machine and myself. [Integrating the Hack type checker into my editor](https://github.com/hhvm/vim-hack) means that my entire codebase is analyzed in a split second as soon as I save a file. This immediately surfaces any dumb, or subtle mistakes I made. I find myself writing code fearlessly: when I forget what a function returns, I just write code that calls it with what I think it returns. If I'm wrong, the type checker will tell me immediately. I can fix it quickly, and move on.

PHP has always facilitated a tight feedback loop between the machine and the developer. Save the file, reload the browser, repeat. Hack's type checker makes this even faster. I look forward to being able to build similar tooling on top of PHP 7's strict mode.