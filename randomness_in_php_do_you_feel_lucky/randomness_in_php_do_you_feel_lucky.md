>Randomness in PHP – Do You Feel Lucky?
http://www.sitepoint.com/randomness-php-feel-lucky/

![Криптографическая рандомизация в PHP](https://habrastorage.org/files/dd7/176/96c/dd717696cc1a4dc69497bb7d06734f94.jpg "Криптографическая рандомизация в PHP")

В этой статье мы проанализируем проблемы, относящиеся к генерации случайных чисел, используемых в криптографии. PHP5 не обеспечивает простой механизм генерации криптостойких случайных чисел, в то время как PHP7 решает эту проблему путем введения [CSPRNG](https://ru.wikipedia.org/wiki/Криптографически_стойкий_генератор_псевдослучайных_чисел)-функций.

## Что такое CSPRNG?

Цитируя [википедию](https://ru.wikipedia.org/wiki/Криптографически_стойкий_генератор_псевдослучайных_чисел), криптографически стойкий генератор псевдослучайных чисел (англ. Cryptographically secure pseudorandom number generator, CSPRNG) - это генератор псевдослучайных чисел с определёнными свойствами, позволяющими использовать его в криптографии.

CSPRNG в основном используется для следующих целей:

*	Генерация ключей (в том числе, генерация public/private ключей)
*	Создание случайных паролей для аккаунтов пользователей
*	Системы шифрования

Главным аспектом сохранения высокого уровня безопасности является высокое качество случайности.

## CSPRNG в PHP7

PHP7 вводит две новых функции, которые могут быть использованы для CSPRNG: [`random_bytes`](http://php.net/manual/en/function.random-bytes.php) и [`random_int`](http://php.net/manual/en/function.random-int.php).

Функция `random_bytes` возвращает строку и принимает в качестве входных параметров `int`, задающий длину (в байтах) возвращаемого значения:

```php
$bytes = random_bytes('10');
var_dump(bin2hex($bytes));
//possible ouput: string(20) "7dfab0af960d359388e6"  
```

`random_int` возвращает целое число в заданном диапазоне:

```php
var_dump(random_int(1, 100));
//possible output: 27
```

## За кадром

Источники случайности вышеперечисленных функций отличаются в зависимости от среды:

*	В Windows всегда будет использоваться `CryptGenRandom();`
*	На других платформах - при условии доступности будет задействована [`arc4random_buf()`](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man3/arc4random.3) (верно в случае BSD-производных систем или систем с `libbsd`).
*   В случае недоступности вышеобозначенного, в Linux будет использоваться системный [`getrandom(2)`](http://man7.org/linux/man-pages/man2/getrandom.2.html).
*	Если все это терпит неудачу, в качестве финальной попытки PHP попробует задействовать `/dev/urandom`.
*	При невозможности использовать эти источники будет выброшена ошибка.

## Простой тест

Хорошая система генерации случайных чисел определяется "качеством" генераций. Чтобы его проверить часто используется набор статистических тестов, позволяющих, не вникая в сложную тему статистики, сравнить известное эталонное поведение с результатом генератора и помочь в оценке его качества.

Один из самых простых тестов - игра в кости. Предполагаемая вероятность выпадения шестерки при одной кости - один к шести, в то же время, если я брошу три кости 100 раз, то ожидаемые выпадения 1, 2 и 3х шестерок примерно такие:

*   0 шестерок = 57.9 раз
*   1 шестерка = 34.7 раз
*   2 шестерки = 6.9 раз
*   3 шестерки = 0.5 раз

Вот код для воспроизведения броска костей 1 000 000 раз:

```php
$times = 1000000;
$result = [];
for ($i=0; $i < $times; $i++) {
    $dieRoll = array(6 => 0); //initializes just the six counting to zero
    $dieRoll[roll()] += 1; //first die
    $dieRoll[roll()] += 1; //second die
    $dieRoll[roll()] += 1; //third die
    $result[$dieRoll[6]] += 1; //counts the sixes
}
function roll() {
    return random_int(1,6);
}
var_dump($result);
```

Прогонка кода в PHP7 c использованием `random_int` и простого `rand` выдаст следующие результаты:

| Шестерки | Ожидаемый результат | random_int | rand   |
|----------|---------------------|------------|--------|
| 0        | 579000              | 579430     | 578179 |
| 1        | 347000              | 346927     | 347620 |
| 2        | 69000               | 68985      | 69586  |
| 3        | 5000                | 4658       | 4615   |

Для лучшего сравнения `rand` и `random_int` построим график результатов, применяя формулу: `результат PHP` - `ожидаемый результат` / `sqrt(ожидаемый результат)`.

График будет выглядеть так:

![график test random](https://habrastorage.org/files/942/335/eef/942335eef1ae4f09935722986e17d66a.png "график test random")  
(чем ближе к нулю, тем лучше)

Даже несмотря на плохой результат с тремя шестерками и всю простоту теста, мы видим явное превосходство `random_int` над `rand`.

## А что насчет PHP5?

По умолчанию, PHP5 не предусматривает каких-либо сильных псевдо-генераторов случайных чисел. Но на самом деле есть несколько вариантов, таких как [`openssl_random_pseudo_bytes()`](http://php.net/manual/en/function.openssl-random-pseudo-bytes.php), [`mcrypt_create_iv()`](http://php.net/manual/en/function.mcrypt-create-iv.php) или непосредственное использование [`/dev/random`](https://en.wikipedia.org/wiki//dev/random) или [`/dev/urandom`](https://en.wikipedia.org/wiki//dev/random) с `fread()`. Есть также такие библиотеки, как [RandomLib](https://github.com/ircmaxell/RandomLib) или [libsodium](https://pecl.php.net/package/libsodium).

Если вы хотите начать использовать хороший генератор случайных чисел и в то же время пока еще не готовы к переходу на PHP7, вы можете использовать библитеку [`random_compat`](https://github.com/paragonie/random_compat) от Paragon Initiative Enterprises. Она позволяет использовать `random_bytes()` и `random_int()` в PHP 5.х проектах.

Библиотеку можно установить через [Composer](https://getcomposer.org/):

```php
composer require paragonie/random_compat

require 'vendor/autoload.php';
$string = random_bytes(32);
var_dump(bin2hex($string));
// string(64) "8757a27ce421b3b9363b7825104f8bc8cf27c4c3036573e5f0d4a91ad2aaec6f"
$int = random_int(0,255);
var_dump($int);
// int(81)
```

По сравнению с PHP7, `random_compat` использует несколько другие приоритеты:

1.	`fread()` `/dev/urandom` если доступно
2.	`mcrypt_create_iv($bytes, MCRYPT_CREATE_IV)`
3.	`COM('CAPICOM.Utilities.1')->GetRandom()`
4.	`openssl_random_pseudo_bytes()`

Дополнительную информацию о том почему используется именно этот порядок вы можете прочитать в [документации](https://github.com/paragonie/random_compat/blob/master/ERRATA.md).

Пример генерации пароля с использованием библиотеки:

```php
$passwordChar = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
$passwordLength = 8;
$max = strlen($passwordChar) - 1;
$password = '';
for ($i = 0; $i < $passwordLength; ++$i) {
    $password .= $passwordChar[random_int(0, $max)];
}
echo $password;
//possible output: 7rgG8GHu
```

## Краткий итог

Вы всегда должны применять криптографически стойкие генераторы псевдослучайных чисел, и `random_compat` является хорошим решением для этого.

Если же вам необходим надежный источник случайных данных, то посмотрите в сторону `random_int` и `random_bytes`.

### Ссылки по теме

*	[Die Hard Test](https://en.wikipedia.org/wiki/Diehard_tests)
*	[Chi-square test with dice example](http://bit.ly/1Mrptf5)
*	[Kolmogorov-Smirnov Test](https://en.wikipedia.org/wiki/Kolmogorov-Smirnov_test)
*	[Spectral Test](http://random.mat.sbg.ac.at/tests/theory/spectral/)
*	[RaBiGeTe test suite](http://cristianopi.altervista.org/RaBiGeTe_MT)
*	[Random Number Generation In PHP (2011)](http://cristianopi.altervista.org/RaBiGeTe_MT)
*	[Testing RNG part 1](http://ubm.io/1Ot46vL) [and 2](http://ubm.io/1VNzh3N)