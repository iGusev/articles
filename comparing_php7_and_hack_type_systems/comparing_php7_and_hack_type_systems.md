>Comparing the PHP 7 and Hack Type Systems
>http://www.dmiller.io/blog/2015/4/26/comparing-the-php7-and-hack-type-systems

![](https://habrastorage.org/files/add/f40/35f/addf4035fbbc476bb5d14f1ecab3644a.png)

Одной из интересных вещей в PHP7, кроме невероятной производительности, является введение [скалярного type-hinting'а](https://wiki.php.net/rfc/scalar_type_hints_v5) в сочетании с опциональным "strict" режимом. При чтении RFC я заметил, что PHP код в примерах выглядит очень похожим на [Hack](http://hacklang.org/). Что если выполнить один и тот же код и в PHP7 и в Hack? Какая разница между ними? Вот что я узнал.
<habracut/>

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

Для того, чтобы не изменяя открывающих тегов в исходных файлах выполнять PHP-код в HHVM, исполняйте `hhvm` с флагом `-vEval.EnableHipHopSyntax=true`.

## Некоторые примеры

Рассмотрим простой код:

```
<?php
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

Его выполнение в PHP7 вернет:

```bash
Fatal error: Argument 1 passed to myLog() must be of the type string, integer given, called in /home/vagrant/basic/main.php on line 9 and defined in /home/vagrant/basic/main.php on line 4
```

Выглядит хорошо! PHP7 правильно говорит, что мы передаем целое число (`$a + $b`) в функцию, которая ожидает строку, и выдает соответствующее сообщение об ошибке. Посмотрим, что скажет HHVM:

```bash
Catchable fatal error: Argument 1 passed to myLog() must be an instance of string, int given in /home/vagrant/basic/main.php on line 6
```

Появилась пара различий:

* HHVM называет это "catchable" фатальной ошибкой. Интересно, ведь в [RFC](https://wiki.php.net/_export/code/rfc/scalar_type_hints_v5?codeblock=9) сказано, что ошибка фактически должна совпадать с HHVM.
* HHVM сообщает, что ошибка в строке 6, а PHP, что проблема произошла в строке 9. В подобных случаях я бы предпочел PHP подход, нам показывается и где функция была некорректно вызвана, и где определена.


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

PHP с радостью исполняет код. Hack же возвращает ошибку:

```bash
/home/vagrant/nullable/main.php:4:16,21: Please add a ?, this argument can be null (Typing[4065])
```

Hack не позволяет нам иметь дефолтный аргумет со значением null, т.к. не смешивает понятия "необязательный аргумент" с "обязательный аргументом, который позволяет иметь дефолтное значение" (Подробнее об этом читайте в книге _[Hack and HHVM](http://shop.oreilly.com/product/0636920037194.do)_). Язык предлагает вам сделать аргумент `nullable`:

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

Давайте попробуем что-нибудь посложнее. Что произойдет, если мы смешаем типизации в PHP? Обратите внимание, что определение strict-режима в верхней части файла не имеет никакого эффекта в HHVM.

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

Получается, что функция является строго типизированной, если она вызывается из функции, которая объявлена в файле с соответствующим заголовком. Впрочем, это влияет только на прямые вызовы. Если мы объявим `main.php` строго типизированным, PHP радостно вернет нам 4, несмотря на несоответствие типов, которые мы передаем в `log()`.

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

При выполнении в PHP вернется `3`, хотя мы передаем `int` в том месте, где аннотировали `float` и несмотря на то, что режим строгой типизации включен. Причина заключается в том, что в PHP7 поддерживается расширяющее примитивное преобразование _([Widening primative conversion](http://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.2))_ при включенном строгом режиме. Это означает, что параметры аннотированные как `float` могут иметь значение `int` [в тех случаях](https://wiki.php.net/rfc/scalar_type_hints_v5#int_-_float_conversion_isn_t_lossless), когда возможно безопасное преобразование (почти всегда). HHVM не поддерживает подобное поведение и выбрасывает ошибку типов при исполнении приведенного выше кода:

```
Catchable fatal error: Argument 1 passed to add() must be an instance of float, int given in /home/vagrant/main.php on line 6
```

Если есть еще какие-то другие различия, которые я упустил, пожалуйста, дайте знать в комментариях ниже. Я бы очень хотел глубже изучить эту тему.

## Заключение

В то время, как Hack поддерживает множество фишек, PHP7 не поддерживает типы [nullable](http://docs.hhvm.com/manual/en/hack.nullable.php), [mixed](http://docs.hhvm.com/manual/en/hack.annotations.mixedtypes.php), void возвращаемые значения, [коллекции](http://docs.hhvm.com/manual/en/hack.collections.php), [async](http://docs.hhvm.com/manual/en/hack.async.php) и т. д. Но все же меня сильно радует безопасность и читаемость, которая достигается новым strict-режимом в PHP7.

После написания пары небольших проектов на Hack я понял, что не сами типы делают Hack приятным, а та самая обратная связь, которую создает язык между разработчиком и машиной. [Наличие интеграции проверки типов для Hack в моем редакторе](https://github.com/hhvm/vim-hack) означает, что вся кодовая база анализируется за доли секунды после того, как я сохраню файл. Сразу же отсекаются глупые или неочевидные ошибки, которые я допустил. Я все чаще стал ловить себя на том, что не задумываюсь о возвращаемых значениях функций, просто пишу код по наитию, предполагая очевидные варианты. Если будет ошибка, редактор немедленно сообщит мне. Пара небольших правок и можно двигаться дальше.

PHP также всегда способствовал тесной обратной связи между машиной и разработчиком. Сохраните файл, обновите страницу в браузере, повторите до достижения результата. Быстро и удобно. Но проверка типов в Hack делает это даже быстрее. Я с нетерпением жду появления аналогичного функционала в IDE/редакторах для строгой типизации в новом PHP.