>Comparing the PHP 7 and Hack Type Systems
>http://www.dmiller.io/blog/2015/4/26/comparing-the-php7-and-hack-type-systems


One of the exciting things about PHP 7, aside from the incredible performance improvements, is the introduction of [scalar type hinting](https://wiki.php.net/rfc/scalar_type_hints_v5) coupled with an optional "strict" mode. When reading the RFC I noticed that PHP 7 code written with type hinting begins to look a lot like [Hack](http://hacklang.org/). I wanted to find out if you could execute the same code in PHP 7 and Hack, and what the differences in execution might be. Here's what I found out.

## Установка

Получите следующий результат:

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

Для того, чтобы выполнять PHP-код в HHVM, не изменяя открывающих тегов в исходных файлах исполняйте `hhvm` с флагом `-vEval.EnableHipHopSyntax=true`.

## Некоторые примеры

Рассмотрим простой пример:

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

Выполнение этого в PHP 7 вернет:

```bash
Fatal error: Argument 1 passed to myLog() must be of the type string, integer given, called in /home/vagrant/basic/main.php on line 9 and defined in /home/vagrant/basic/main.php on line 4
```

Выглядит хорошо! PHP7 правильно говорит нам, что мы передаем целое число (`$a + $b`) в фунцию, которая ожидает строку, и выдает соответствующее сообщение об ошибке. Посмотрим, что скажет HHVM:


```bash
Catchable fatal error: Argument 1 passed to myLog() must be an instance of string, int given in /home/vagrant/basic/main.php on line 6
```

Появилась пара различий:

* HHVM называет это "catchable" фатальной ошибкой. Это интересно, так как в [RFC](https://wiki.php.net/_export/code/rfc/scalar_type_hints_v5?codeblock=9) сказано, что ошибка фактически должна совпадать с HHVM.
* HHVM говорит, что ошибка в строке 6, а PHP, что это произошло в строке 9. В подобных случаях я бы предпочел PHP подход, нам показывается и где функция была некорректно вызвана и где определена.


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

PHP с радостью выполняет код. Hack же возвращает ошибку:

```bash
/home/vagrant/nullable/main.php:4:16,21: Please add a ?, this argument can be null (Typing[4065])
```

Hack не позволяет нам иметь дефолтный аргумет со значением null, т.к. смешивает понятие необязательный аргумент с обязательным аргументом, который позволяет иметь дефолтное значение (Подробнее об этом читайте в книге _[Hack and HHVM](http://shop.oreilly.com/product/0636920037194.do)_). Язык предлагает вам сделать аргумент `nullable`:

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

Давайте попробуем что-нибудь посложее. Что произойдет, если мы смешаем типизации в PHP? Обратите внимание, что определение strict-режима в верхней части файла не имеет никакого эффекта в HHVM.

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

Для `logger.php` включился strict-режим, но PHP позволяет передать int в него из nonstrict-файла. HHVM в подобном случае выбрасывает исключение. Что произойдет, если мы переведем `add.php` в режим строгой типизации:

```bash
Fatal error: Argument 1 passed to myLog() must be of the type string, integer given, called in /home/vagrant/separate_files_mixed/add.php on line 5 and defined in /home/vagrant/separate_files_mixed/logger.php on line 4
```

Так-то лучше. Strict-режим действует только в тех файлах, где он указан, даже если декларирующий функцию файл подразумевает иное. А что произойдет, если мы вызовем non-strict функцию, из strict-функции? Для реализации я поставил следующие значения для файлов:

```
logger.php - non-strict
add.php - strict
```

```bash
Fatal error: Argument 1 passed to myLog() must be of the type string, integer given, called in /home/vagrant/separate_files_mixed/add.php on line 5 and defined in /home/vagrant/separate_files_mixed/logger.php on line 3
```

Получается, что функция является строго типизированной, если она вызывается из функции, которая объявлена в strict-файле. Впрочем это влияет только на прямые вызовы в этом файле. Если мы объявим `main.php` строго типизированным, PHP радостно вернет нам 4, несмотря на несоответствие типов, которые мы передаем в `log()`.

В Hack соотношение обратное. Если HHVM выполяет main.php в нестрогом режиме, и логгер [написан на Hack](https://gist.github.com/jazzdan/fe0648a6848dadda5039) (c `hh` тегом в верхней части файла), мы все равно получим ошибку типа несмотря на то, что вызываемый файл не написан на Hack.

```bash
Catchable fatal error: Argument 1 passed to myLog() must be an instance of string, int given in /home/vagrant/separate_files_mixed/logger.php on line 5
```

Другим интересным отличием между системами типизирования Hack и PHP является аннотация float. Возьмем пример:

```
<?php
declare(strict_types=1);

function add(float $a, float $b): float {
    return $a + $b;
}

echo add(1, 2);
```

При выполнении в PHp вернется `3`, хотя мы передаем `int` в том месте, где анотировали `float` и несмотря на то, что режим строгой типизации включен. Причина заключается в том, что в PHP7 поддерживается расширенное преобразование примитивов _([Widening primative conversion](http://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.2))_ при включенном строгом режиме. Это означает, что параметры анотированные как `float` могут иметь значение `int` [в тех случаях](https://wiki.php.net/rfc/scalar_type_hints_v5#int_-_float_conversion_isn_t_lossless), когда возможно безопасное преобразование (почти всегда). HHVM не поддерживает подобное поведение и выбрасывает ошибку типов при исполнении приведенного выше кода:

```
Catchable fatal error: Argument 1 passed to add() must be an instance of float, int given in /home/vagrant/main.php on line 6
```

Если есть еще какие-то другие различия, которые я упустил, пожалуйста, дайте знать в комментариях ниже. Я бы очень хотел глубже изучить эту тему.

## Wrapping It Up

While there are many features in Hack that PHP 7 does not support ([nullable](http://docs.hhvm.com/manual/en/hack.nullable.php), [mixed](http://docs.hhvm.com/manual/en/hack.annotations.mixedtypes.php) types, void return types, [collections](http://docs.hhvm.com/manual/en/hack.collections.php), [async](http://docs.hhvm.com/manual/en/hack.async.php), etc) I am excited by the safety and readibility PHP 7's new strict mode enables.

After writing a couple small programs in Hack I've realized that it's not types themselves that make writing Hack enjoyable: it's the tight feedback loop Hack creates between the machine and myself. [Integrating the Hack type checker into my editor](https://github.com/hhvm/vim-hack) means that my entire codebase is analyzed in a split second as soon as I save a file. This immediately surfaces any dumb, or subtle mistakes I made. I find myself writing code fearlessly: when I forget what a function returns, I just write code that calls it with what I think it returns. If I'm wrong, the type checker will tell me immediately. I can fix it quickly, and move on.

PHP has always facilitated a tight feedback loop between the machine and the developer. Save the file, reload the browser, repeat. Hack's type checker makes this even faster. I look forward to being able to build similar tooling on top of PHP 7's strict mode.