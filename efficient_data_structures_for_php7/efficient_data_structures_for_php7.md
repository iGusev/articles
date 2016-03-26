>Efficient data structures for PHP 7
https://medium.com/@rtheunissen/efficient-data-structures-for-php-7-9dda7af674cd#.y3w5bzg8d

### Эффективной структуры данных для PHP 7

PHP имеет всего одну структуру данных для управления всем. `array` - это сложная, гибкая, гибридная структура данных, сочетающая поведение `list` и `linked map`. Но мы используем его для всего, потому что PHP использует **прагматичный подход**: предельно правильный, здравый и реалистичный способ, исходящий из практических, а не теоретических рассуждений. `array` позволяет делать работу, хотя о нем и так много рассказывают на лекциях по информатике. Но, к сожалению, с гибкостью приходит и сложность.

Последний релиз PHP вызвал большое оживление в сообществе. Мы не могли дождаться того, чтобы начать использовать [новые возможности](http://php.net/manual/en/migration70.new-features.php) и почувствовать вкус [~2х прироста производительности](https://www.reddit.com/r/PHP/comments/3q2brz/how_is_php_7_twice_as_fast/). Одна из причин, почему это случилось - [структура `array` была переработана](https://nikic.github.io/2014/12/22/PHPs-new-hashtable-implementation.html). Но она все также придерживается принципа "оптимизировано для всего; оптимизировано для ничего", еще не все идеально, есть возможности для совершенствования.

> "А что насчет [структур данных SPL](http://php.net/manual/en/spl.datastructures.php)?"

К сожалению... они ужасны. Они предлагали _некоторые_ преимущества дл PHP7, но сейчас мы дошли до точки, когда использование SPL не имеет практического смысла.

> "Почему мы не можем просто поправить и улучшить их?"

Да, мы могли бы, но я считаю, что их дизайн и реализация настолько бедны, что лучше бы заменить их чем-то более современным.

>**_"SPL data structures are horribly designed."_**
>
>— _Anthony Ferrara_

<cut />

* * *

Введение: `php-ds`, расширение для PHP 7, добавляющее структуры данных. Этот пост кратко охватывает поведение, производительность и преимущества каждой из них. Также в конце вы найдете список ответов на ожидаемые вопросы.

**Github**: [https://github.com/**php-ds**](https://github.com/php-ds)

**Пространство имен:** `Ds\`

**Интерфейсы:** `Collection`, `Sequence`, `Hashable`

**Классы:** `Vector`, `Deque`, `Stack`, `Queue`, `PriorityQueue`, `Map`, `Set`

* * *

## Коллекция (Collection)

`Collection` - это базовый интерфейс, охватывающий общую функциональность, такую как `foreach`, `echo`, `count`, `print_r`, `var_dump`, `serialize`, `json_encode`, и `clone`.

## Последовательность (Sequence)

`Sequence` описывает поведение значений, организованных в единый, линейный размер. В некоторых языках он назвается `List` _(список)_. Он подобен `array`, который использует инкрементальные ключи, за исключением некоторых особенностей:

* Значения **всегда** должны быть индексированы как `[0, 1, 2, …, size - 1]`.
* Извлечение или добавление приводит к обновление индекса всех последовательных значений.
* Поддерживает доступ к занчениям только из индекса `[0, size - 1]`

### **Варианты использования**

* Везде, где вы бы хотели использовать `array` как список (без ключей).
* Более эффективная альтернатива `SplDoublyLinkedList` и `SplFixedArray`.

## Вектор (Vector)

`Vector` преставляет собой `Sequence`, объединяющую значения в непрерывный буфер, увеличивающийся и уменьшающийся автоматически. Это наиболее эффективная последовательная структура, поскольку индекс элемента является прямым отражением его индекса в буфере, и увеличение вектора никак не повлияет на производительность.

<iframe src="https://player.vimeo.com/video/154438958" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

### **Сильные стороны**

* Очень маленькое потребление памяти
* Очень быстрые итерации
* `get`, `set`, `push` и `pop` имеют сложность `O(1)`

### **Недостатки**

* `insert`, `remove`, `shift` and `unshift` имеют сложность `O(n)`

> Структурой номер один в Photoshop были Вектора.
>
>— **Sean Parent**, [CppCon 2015](https://youtu.be/sWgDk-o-6ZE?t=21m52s)

### Двусвязная очередь (Deque)

`Deque` (произносится как "deck") - это последовательность значений, объединенных в непрерывный буфер, увеличивающийся и уменьшающийся автоматически. Название является общепринятым сокращением от _"double-ended queue"_. Используется внутри `Ds\Queue`.

Два указателя используется для отслеживания головы и хвоста. Наличие указателей позволяет изменять конец и начало буфера без необходимости перемещать другие элементы для освобождения места. Это делает `shift` и `unshift` настолько быстрым, что даже `Vector` не может конкурировать с этим.

Доступ к значению по индексу требует вычисления соответствующей позиции в буфере: `((head + position) % capacity)`.

<iframe src="https://player.vimeo.com/video/154438012" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

#### **Сильные стороны**

* Очень маленькое потребление памяти
* `get`, `set`, `push`, `pop`, `shift` и `unshift имеют сложность `O(1)`

#### **Недостатки**

* `insert`, `remove` имеют сложность `O(n)`
* Емкость буфера должна иметь степень двойки (`2ⁿ`)

Следующий бенчмарк показывает общее затраченное время и память, используемую для операции `push` 2ⁿ случайных чисел. `array`, `Ds\Vector` и `Ds\Deque` отрабатывают быстро, но `SplDoublyLinkedList` стабильно показывает результат **более чем в 2 раза хуже**.

`SplDoublyLinkedList` выделяет память для каждого значения по отдельности, поэтому и происходит ожидаемый рост по памяти. `array` и `Ds\Deque` при своей реализации выделяют память порционно для поддержания достаточного объема для 2ⁿ элементов. `Ds\Vector` имеет фактор роста 1.5, что влечет за собой увеличение количества выделений памяти, но меньший расход в целом.

![](https://cdn-images-1.medium.com/max/1600/1*BZVzcscdpcUg8SZmvUEjQQ.gif)

![](https://cdn-images-1.medium.com/max/1600/1*FHxbwYbZ75l_pSEvWmNCig.gif)

Следующий бенчмарк показывает время, затраченное на `unshift` **единственного элемента** в последовательности значений размером 2ⁿ. Время, требующееся на установку значений не учитывается.

На графике видно, что `array_unshift` имеет сложность `O(n)`: всякий раз, когда объем выборки удваивается, растет и время, необходимое для `unshift`. Это объясняется тем, что каждый числовой показатель в диапазоне `[1, size - 1]` должен быть обновлен.

![](https://cdn-images-1.medium.com/max/1600/1*7lF6nsm9MlvpHZ-IzFCLdw.gif)

Но и `Ds\Vector::unshift` также `O(n)`, так почему же он намного быстрее? Имейте ввиду, что `array` храник каждое значение в `bucket` вместе с его ключем и хэшем. Поэтому приходится проверять каждый элемент и обновлять хэш, если индекс является числовым. На самом деле `array_unshift` выделяет новый массив для этого и заменяет старый, когда все значения скопированы.

В векторе же индекс значения - это прямое отображение его индекса в буфере, поэтому все, что нам нужно сделать - сдвинуть каждое значение в диапазоне [1, size - 1] вправо на одну позицию. Делается это при помощи всего одной операции `memmove`.

`Ds\Deque` и `_SplDoublyLinkedList_` в свою очередь очень быстры, потому что на время для `unshift` значения не влияет размер выборки, т.е. его сложность будет `O(1)`.

Следующий тест показывает сколько памяти используется при 2ⁿ `pop` операций. Другими словами при изменении размера от 2ⁿ до нуля

Интересно тут то, что `array` всегда держит выделенную память, даже если его размер существенно уменьшается. `Ds\Vector` and `Ds\Deque` позволяют в два раза уменьшить выделяемые ресурсы, если их размер падает ниже четверти своего текущего потенциала. `SplDoublyLinkedList` освобождает память после каждого удаления из выборки, поэтому мы можем наблюдать линейное снижение.

![](https://cdn-images-1.medium.com/max/1600/1*i8dnM0yH9dHlhL-RuK2MAg.gif)

### Стек _(Stack)_

[Стек](https://ru.wikipedia.org/wiki/%D0%A1%D1%82%D0%B5%D0%BA) - является коллекцией, организованной по принципу _"последним пришёл — первым вышел"_ или _"LIFO" (last in — first out)_, позволяющей получить доступ только к значению на вершине структуры. Вы можете думать о нем как об [оружейном магазине](https://upload.wikimedia.org/wikipedia/commons/6/6c/Single_row_box_magazine.svg) с динамической емкостью.

`Ds\Stack` использует внутри себя `Ds\Vector`.

`SplStack` наследуется от SplDoublyLinkedList, поэтому производительность будет эквивалентна сравнению `Ds\Vector` to `SplDoublyLinkedList` из предыдущих тестов. Посмотрим на время, необходимое для выполнения 2ⁿ `pop`-операций, изменения размера от 2ⁿ до нуля.

![](https://cdn-images-1.medium.com/max/1600/1*yKfo29kCIPVkqFETXmVGRg.gif)

### Очередь _(Queue)_

[Очередь](https://ru.wikipedia.org/wiki/Очередь_(программирование)) - тип данных с парадигмой доступа к элементам _"первый пришел — первый вышел" ("FIFO", "First In — First Out")_. Такая коллекция позволяет получить доступ к элементам в порядке их добавления. Ее название говорит само за себя, представьте себе структуру как линию людей, стоящих в очереди на кассу в магазине.

`Ds\Queue` использует внутри себя `Ds\Deque`. `SplQueue` наследуется от `SplDoublyLinkedList`, поэтому производительность будет эквивалентна сравнению `Ds\Deque` с `SplDoublyLinkedList`, показанному в предыдущем бенчмарке.

### Очередь с приоритетом _(PriorityQueue)_

Очередь с приоритетом очень похожа на простую очередь. Элементы помещаются в очередь с указанным приоритетом и значение с наивысшим приоритетом всегда будет в передней части. Прямой перебор очереди с приоритетом очень деструктивен, это будет последовательный вызов операций `pop`, что является очень затратной операцией.

Реализация очереди с приоритетом использует _max-heap_.

Принцип "первый пришел — первый вышел" сохраняется для значений с одинаковым приоритетом, так что группа значений с равным приоритетом можно рассматривать как обычную очередь.

А что же с производительностью? Следующий бенчмарк показывает время и память, требующиеся для операции `push` 2ⁿ случайных чисел со случайным приоритетом в очередь. Те же случайные числа будут использоваться для каждого из тестов. В тесте для `Queue` также генерируется случайный приоритет, хотя он и не используется.

Это, наверное, самый значимый из всех бенчмарков. `Ds\PriorityQueue` работает **более чем в два раза быстрее** чем `SplPriorityQueue` и использует только **5%** от его памяти - это в *20 раз более эффективное решение по памяти*.

Но как? Как может получиться настолько большая разница, когда `SplPriorityQueue` использует аналогичную внутреннюю структуру? Все сводится к тому, как хранятся значения в паре с приоритетом. `SplPriorityQueue` позволяет использовать любой тип значения для использования в качестве переменной, это приводит к тому, что в каждой паре приоритет занимает **32 байта**.

`Ds\PriorityQueue` поддерживает только целочисленные приоритеты, поэтому каждой паре выделяется **24 байта**. Но это все еще недостаточная разница для объяснения результата.

Если вы посмотрите на [исходный код `SplPriorityQueue::insert`](http://lxr.php.net/xref/PHP_7_0/ext/spl/spl_heap.c#629), то заметите, что он **инициализирует массив** для **хранения пары значение-приоритет**.

Т.к. массив имеет минимальную емкость 8, то для каждой пары на самом деле выделяется `zval + HashTable + 8 * (Bucket + hash) + 2 * zend_string + (8 + 16) byte string payloads` _=_ `16 + 56 + 36 * 8 + 2 * 24 + 8 + 16` _=_ _**432 байта**_ (64 бит).

> “Так… почему же массив?”

`SplPriorityQueue` использует ту же внутреннюю структуру `SplMaxHeap`, которая требует от значения быть типом `zval`. Очевидный (но неэффективный) способ создания `zval`-пары, т.к. `zval` сам используется как `array`.

![](https://cdn-images-1.medium.com/max/1600/1*G-jWxdZtPo7iWMCuiXKNgg.gif)

![](https://cdn-images-1.medium.com/max/1600/1*FvRoA1Nh6N2V6nwAgC6VRg.gif)

### Hashable

An interface which **allows objects to be used as keys**. It’s an alternative to _spl_object_hash,_ which determines an object’s hash based on its _handle:_ this means that two objects that are considered equal by an implicit definition would not treated as equal because they are not the same instance.

_Hashable_ introduces only two methods: **hash** and **equals**. Many other languages support this natively, like Java’s _hashCode_ and _equals,_ or Python’s ___hash___ and ___eq__._ There have been a few RFC’s to add this to PHP but none of them have been accepted.

All structures that honour this interface will fall back to _spl_object_hash_ if an object key does not implement _Hashable._

Data structures that honour the _Hashable_ interface_:_ **_Map_ **and **_Set_**

### Map

A _Map_ is a sequential collection of key-value pairs, almost identical to an _array_ used in a similar context. **Keys can be any type**, but must be unique. Values are replaced if added to the map using the same key.

Like an _array_, insertion order is preserved.

#### **Strengths**

*   Performance and memory efficiency is almost **identical to an _array_**.
*   Automatically frees allocated memory when its size drops low enough.
*   Keys and values can be any type, including objects.
*   Honours the _Hashable_ interface.
*   _put, get, remove,_ and _containsKey_ are _O(1)_

#### **Weaknesses**

*   Can’t be converted to an _array_ when objects are used as keys.
*   Can’t access values by index (position).

The following benchmarks show that the performance and memory efficiency is identical between an _array_ and a _Ds\Map._ However, an _array_ will always hold on to allocated memory, where a _Ds\Map_ will truncate memory when its size drops below a quarter of its capacity.

![](https://cdn-images-1.medium.com/max/1600/1*sLlnRyldnLfeGqLR1pWcaA.gif)

![](https://cdn-images-1.medium.com/max/1600/1*6cF3pbT_4DQfeqi12gEHZg.gif)

### Set

A _Set_ is a collection of **unique values**. The textbook definition of a _set_ will say that values are unordered unless an implementation specifies otherwise. Using Java as an example, _java.util.Set_ is an interface with two primary implementations: _HashSet_ and _TreeSet. HashSet_ provides _O(1) add_ and _remove_, and _TreeSet_ ensures a sorted set but _O(log n)_ _add_ and _remove._

_Set_ uses the same internal structure as a _Map_, which is based on the same structure as an _array_. This means that a _Set_ can be sorted in _O(n * log(n))_ timewhenever it needs to be, just like a _Map_ and an _array_.

<iframe src="https://player.vimeo.com/video/154441519" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

#### **Strengths**

* _add, remove,_ and _contains_ are _O(1)_
* Honours the _Hashable_ interface.
* Supports **any type of value** (_SplObjectStorage_ only supports objects).
* Bitwise equivalents for _intersection, difference, union,_ and _exclusive or._

#### **Weaknesses**

* Doesn’t support _push, pop, insert, shift,_ or _unshift._
* _get_ is _O(n)_ if there are deleted values before the index, _O(1)_ otherwise.

The following benchmark shows the time taken to add 2ⁿ new instances of _stdClass._ It shows that _Ds\Set_ is **slightly faster** than _SplObjectStorage,_ and uses about **half the memory**.

![](https://cdn-images-1.medium.com/max/1600/1*5ihSMBaqzpzMS-xVDL6nNA.gif)

![](https://cdn-images-1.medium.com/max/1600/1*L5lzvjIKpm-gsW53qbNRpA.gif)

A common way to create an _array_ of unique values is to use _array_unique,_ which creates a new _array_ containing only unique values. An important thing to keep in mind here is that **values in an array are not indexed**, so _in_array_ is a linear search, _O(n)._ Because _array_unique_ deals with values instead of keys, each membership test is a linear search, resulting in _O(n_²_)_ time complexity and _O(n)_ memory complexity.

![](https://cdn-images-1.medium.com/max/1600/1*jL3VMkV-JisdSqyuMaSMaQ.gif)

### Responses to expected questions and opinions

> Are there tests?

Right now there are ~**2600 tests**. It’s possible that some of the tests are redundant but I’d rather indirectly test the same thing twice than not at all.

> Documentation? API reference?

At the time of this writing there is no complete documentation, but there will be proper _docbook_ documentation with the first stable release.

There are however some [**well-documented stub files**](https://github.com/php-ds/ds/tree/master/php/include).

> Can we see how the benchmarks were configured? Are there more of them?

You can find a complete list of configurable benchmarks in the dedicated benchmark repository: [php-ds/benchmarks](https://github.com/php-ds/benchmarks)

All featured benchmarks were created using a default build of **_PHP 7.0.3_ **on a **2015 Macbook Pro**. Results will vary between versions and platforms.

> Why are Stack_,_ Queue_,_ Set_, and_ Mapnot interfaces_?_

I don’t believe that any of them have an alternative implementation worth including. Introducing 3 interfaces and 7 classes is a good balance between pragmatism and specialisation.

> When should I use a Dequerather than a Vector?

If you know **for sure** that you won’t be using **_shift_** and **_unshift_**, use _Vector_. You can use _Sequence_ as a typehint to accept either.

> Why are all the classes **final**_?_

The design of the _php-ds_API enforces [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance).

The SPL structures are a good example of how inheritance can be misused, eg. _SplStack_ extends _SplDoublyLinkedList_ which supports random access by index, _shift_ and _unshift — _so it’s not technically a [_Stack_](https://en.wikipedia.org/wiki/Stack_%28abstract_data_type%29)_._

The Java Collections Framework also has a few interesting cases where inheritance causes ambiguity. An _ArrayDeque_ has three methods for appending a value: _add, addLast,_ and _push._ This is not exactly a bad thing, because _ArrayDeque_ implements _Deque_ and _Queue,_ which is why it must implement _addLast_ and _push._ However, having three methods that do the same thing causes confusion and inconsistency.

The old _java.util.Stack_ extends _java.util.Vector,_ andstates that “a more complete and consistent set of LIFO stack operations is provided by the Deque interface and its implementations”, but the Deque interface includes methods like _addFirst_ and _remove(x),_ which shouldn't be part of a _stack_ API.

> Just because these structures don’t extend each other doesn't mean that we shouldn't be allowed to either.

That’s actually a fair point, but I still believe that composition is more appropriate for data structures. They are designed to be self-contained, much like an _array_. You can’t extend an _array_, so we design our own APIs around it by using an internal _array_ property to store the actual data.

Inheritance would also introduce unnecessary internal complexity.

> Why is there also a **ds**class in the global namespace?

It provides an alternative construction syntax:

![](https://cdn-images-1.medium.com/max/1600/1*iD41TKleXh32cgVkz18x7w.gif)

> Why no linked list?

_LinkedList_ actually came first because it seemed like a good place to start. I decided to remove it when I realised it wouldn’t be able to compete with _Vector_ or _Deque_ in any situation. The two primary reasons to support that are **allocation overhead** and **locality of reference_._**

A _linked list_ has to either allocate or free a _node_ whenever a value is added or removed. A _node_ also has two pointers (in the case of a doubly linked list) to reference another _node_ that comes before, and one that comes after. Both _Vector_ and _Deque_ allocate a buffer of memory in advance, so there’s no need to allocate and free as often. They also don’t need additional pointers to know what value comes before or after another, so there’s less overhead.

> Would a linked list have lower peak memory because there’s no buffer?

Only when the collection is very small. The upper bound of a _Vector_’s memory usage is (1.5 * (_size -_ 1)) * _zval_ bytes, with a minimum of 10 * _zval_. A _doubly linked list_ would use (_size_ * (_zval_ + 8 + 8)). So a _linked list_ uses less memory than a _Vector_ if its size is less than 6.

> Okay… so a linked list uses more memory. But why is it slow?

The nodes of a _linked list_ have bad **_spatial locality_**_._ This means that the physical memory location of a node might be far away from its adjacent nodes. Iterating through a _linked list_ therefore jumps around in memory instead of utilizing the CPU cache. This is where both _Vector_ and _Deque_ have a significant advantage: values are physically right next to each other.

> “Discontiguous data structures are the root of all performance evil. Specifically, please say no to linked lists.”

> “There is almost nothing more harmful you can do to the performance of an actual modern microprocessor than to use a linked list data structure.”

> — Chandler Carruth ([CppCon 2014](https://youtu.be/fHNmRkzxHWs?t=34m42s))





> PHP is a web development language — performance is not important.

**Performance should not be your top priority**. Code should be consistent, maintainable, robust, predictable, safe, and easy to understand. But that’s not to say that performance is _“not important”_.

We spend a lot of time trying to reduce the size of our assets, benchmarking frameworks, and coming up with pointless micro-optimisations:

*   [print vs echo, which one is faster?](http://fabien.potencier.org/print-vs-echo-which-one-is-faster.html)
*   [The PHP Ternary Operator: Fast or not?](http://fabien.potencier.org/the-php-ternary-operator-fast-or-not.html)
*   [The PHP Benchmark: setting the record straight](http://www.phpbench.com/)
*   [Disproving the Single Quotes Performance Myth](https://nikic.github.io/2012/01/09/Disproving-the-Single-Quotes-Performance-Myth.html)

The ~2x performance increase that came with PHP 7 had us all desperately eager to try it out. It’s arguably one of the most-mentioned benefits of switching from PHP 5.

Efficient code reduces the load on our servers, reduces the response time of our APIs and web pages, and reduces the runtime of our development tools. **Performance is important**, but maintainable code comes first.


💬 **Discuss**: [Twitter](https://twitter.com/rudi_theunissen), [Reddit](https://www.reddit.com/r/PHP/comments/44qsco/efficient_data_structures_for_php_7/), [Room 11](http://chat.stackoverflow.com/rooms/11/php)

🔎 **Explore**: [github.com/php-ds](https://github.com/php-ds)

📊 **Benchmarks:** [github.com/php-ds/benchmarks](https://github.com/php-ds/benchmarks)