>Programming guidelines - Part 1: Reducing complexity
>https://www.ibuildings.nl/blog/2016/01/programming-guidelines-php-developers-part-1-reducing-complexity

## Введение в серию

PHP - это довольно свободный язык программирования. Он динамичный и снисходительный по отношению к программисту. Следовательно, Вам, как php-разработчику, потребуется много дисциплины, чтобы создавать код правильно.

За эти годы я прочитал множество книг по программированию и обсуждал стиль кодирования со многими другими разработчиками. Я уже и не вспомню какие правила приходят из каких книг или от каких людей, но эта статья (и последующие) отражает *мое* видение того, что является действительно полезными в создании перспективного, читабельного и достаточно хорошо понимаемого кода. Коллеги-разработчики смогут с уверенностью утверждать что он делает, быстро выявлять проблемы и легко использовать его в других проектах.

## Снижение сложности функции

Уменьшайте сложность тела метода или функции как можно больше. Меньшая сложность приводит к снижению психической нагрузки для тех, кто будет читать. Постарайтесь также уменьшить количество возможных разночтений и недоразумений, пишите код так, чтобы однозначно можно было понять как он работает и как его изменять.

### Уменьшение количества ветвлений в теле функции

Избавляйтесь от `if`, `elseif`, `else` и `switch` настолько, насколько это возможно. Каждый из них создает одну или более возможностей ветвления исполнения программы. Это сделает код более трудным для понимания или проверки, а также усложнит его тестируемость (поскольку каждая из этих ветвей должна быть покрыта тестом). Хорошее решение часто лежит прямо на поверхности, стоит лишь немного подумать.

#### Делегирование полномочий ("Сообщай, не спрашивай")

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

#### Маппинг

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

#### Типы

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

Следуя этому шаблон вы сильно повысите читатемость и понимаемость кода.

### Создание небольших логических частей

Если тело функции достаточно большое, трудно будет понять что происходит внутри. Отслеживание переменных, их типов, жизненного цикла, (вспомогательных) функций, которые в настоящее время вызываются и т.д. Уходит много усилий чтобы разобраться со всем этим. Когда функция маленькая, проще понять как она работает: принимает некоторый входной сигнал, преобразовывает его и возвращает результат. 

#### Вспомогательные функции

С помощью вышеописанных правил вы сократили количество ветвлений вашего кода, но его еще нельзя назвать хорошим. Разделите ваш код на более мелкие логические единицы. Сгруппируйте строки, отделив их несколькими пустыми линиями, так, чтобы они решали общую задачу. Затем повторяющийся код выделите в отдельные "вспомогательные" методы (Это правило также известно как один из шагов рефакторинга - "Extract method").

Вспомогательные методы, как правило, являются "приватными" и используются только объектами данного класса. Зачастую им не нужен доступ к переменным экземпляра объекта. В таком случае их следует делать "статическими". По моему опыту приватные (статические) вспомогательные методы часто эволюционируют в отдельные классы с публичными интерфейсами.

#### Промежуточные значения

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

Как вы можете видеть, мы в основном сочетаем функции внутри функций, тем самым давая понять что происходит в коде и создаем возможность внесения изменений. В целом, этот код является немного приближен к "open/closed" концепции, поскольку нам больше никогда не придется модифицировать код вспомогательных функций.

Так как многие алгоритмы требуют перебора коллекций, тем самым создавая новую коллекцию или приводя ее к одному значению, то имеет смысл делать коллекции "первого сорта" и привязывать поведение именно к ним:

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

Теперь еще легче сочетать функции друг с другом.

Снижение числа временных переменных, как правило, приводит к более лучшему дизайну, как в примере выше. Но все же вы не должны полностью отказываться от промежуточных значений. Иногда они пригождаются в целях документирования и способны подсказать результат выражения.

### Единый тип

Отслеживание переменных и их текущих значений довольно сложно. Оно становится еще труднее, когда неочевиден тип переменной, и совсем болезненно, если этот тип не соответсвует ожидаемому.

#### Массивы только с одним типом данных

Всякий раз, когда вы используете массив в качестве коллекции, убедитесь, что он содержит только один тип значений. Это позволит снизить сложность считывающего эти значения цикла:

```php
foreach ($collection as $value) {
    // no need to do any checks if we know the type of $value
}
```

Поддерживайте читаемость вашего кода, а также работу IDE, всегда указывайте type-hint коллекции:

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

Поскольку вы не можете быть абсолютно уверены, что `$value` - это именно экземпляр `DateTime`, вам необходимо добавить условие для проверки этого утверждения. Библиотека [beberlei/assert](https://github.com/beberlei/assert) позволит сделать это с легкостью:

```php
use Assert\Assertion
 
public function doSomething(array $collection) {
    Assertion::allIsInstanceOf($collection, \DateTime::class);
 
    ...
}
```

Если коллекция содержит значение, которое не является `DateTime`, то это приведет к выбросу `InvalidArgumentException`. Помимо соблюдения корректности входных значений, используя ее вы также уменьшаете сложность кода, так как вам не придется держать все проверки типов в голове.

#### Один тип возвращаемого значения

Всякий раз, когда функция может возвращать различные типы значений, значительно повышается сложность в клиентского кода:

```php
$result = someFunction();
if ($result === false) {
    ...
} elseif (is_int($result)) {
    ...
}
```

PHP не будет принуждать вас избавляться от возвращаемых типов "mixed". Но их использование приводит к большому количеству конструкций `if`/`else` для учета всех возможных исходов.

Возьмем более распространенным примером смешанных возвращаемых типов:

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

Эта функция будет возвращать либо объект `User` либо `null`, что является очень проблематичным, так как сейчас мы не сможем вызвать метод в любом месте, не проверив предварительно на самом ли деле объект является экземпляром `User`. Более того, PHP до версии 7 просто падал с фатальной ошибкой в случае взаимодействия с объектом другого типа.

Мы рассмотрим этот случай с `null` и методы решения данной проблемы в следующей статье.

### Читаемые выражения

Мы обсудили много вариантов для сокращения общей сложности функций. Есть еще несколько вещей, которые мы можем сделать для упрощения кода.

#### Скрытие сложной логики

Часто можно перенести сложное выражение условия в вспомогательный метод. Вместо:

```php
if (($a || $b) && $c) {
    ...
}
```

Вы можете написать что-то намного лучше:

```php
if (somethingIsTheCase($a, $b, $c)) {
    ...
}
```

Читателю теперь ясно, что исход решения зависит от `$a`, `$b` и `$c`, и имя функции сообщает какое решение принимается.

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