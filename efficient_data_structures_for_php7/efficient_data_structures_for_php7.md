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

### Vector

A _Vector_ is a _Sequence_ of values in a contiguous buffer that grows and shrinks automatically. It’s the most efficient sequential structure because a value’s index is a direct mapping to its index in the buffer, and the growth factor isn't bound to a specific multiple or exponent.

<iframe src="https://player.vimeo.com/video/154438958" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

#### **Strengths**

* Very low memory usage
* Very fast iteration
* _get_, _set, push_ and _pop_ are _O(1)_

#### **Weaknesses**

* _insert, remove, shift,_ and _unshift_ are _O(n)_

> The number one data structure used in Photoshop was Vectors.” — Sean Parent, [CppCon 2015](https://youtu.be/sWgDk-o-6ZE?t=21m52s)

### Deque

A _Deque_ (pronounced _“deck”_)is a _Sequence_ of values in a contiguous buffer that grows and shrinks automatically. The name is a common abbreviation of “_double-ended queue”_ and is used internally by _Ds\Queue._

Two pointers are used to keep track of a head and a tail. The pointers can “wrap around” the end of the buffer, which avoids the need to move other values around to make room. This makes _shift_ and _unshift_ very fast — something a _Vector_ can’t compete with.

Accessing a value by index requires a translation between the index and its corresponding position in the buffer: _((head + position) % capacity)._

<iframe src="https://player.vimeo.com/video/154438012" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

#### **Strengths**

* Low memory usage
* _get,_ _set_, _push, pop, shift,_ and _unshift_ are all _O(1)_

#### **Weaknesses**

*   _insert, remove_ are _O(n)_
*   Buffer capacity must be a power of 2.

The following benchmark shows the total time taken and memory used to _push_ 2ⁿrandom integers. _PHP array_, _Ds\Vector_ and _Ds\Deque_ are all fast, but _SplDoublyLinkedList is_ consistently**more than 2x slower**.

_SplDoublyLinkedList_ allocates memory for each value individually, so linear memory growth is expected. Both an _array_ and _Ds\Deque_ have a 2.0 growth factor to maintain a 2ⁿ capacity. _Ds\Vector_ has a growth factor of 1.5, which results in more allocations but lower memory usage overall.

![](https://cdn-images-1.medium.com/max/1600/1*BZVzcscdpcUg8SZmvUEjQQ.gif)

![](https://cdn-images-1.medium.com/max/1600/1*FHxbwYbZ75l_pSEvWmNCig.gif)

The following benchmark shows the time taken to _unshift_ **a single value** into a sequence of 2ⁿ values. The time it takes to set up the sample is not included in the benchmark.

It shows that _array_unshift_ is _O(n)_. Every time the sample size doubles, so does the time it takes to _unshift._ This makes sense, because every numerical index in the range _[1, size - 1]_ has to be updated.

![](https://cdn-images-1.medium.com/max/1600/1*7lF6nsm9MlvpHZ-IzFCLdw.gif)

But _Ds\Vector::unshift_ is also _O(n)_, so why is it so much faster? Keep in mind that an _array_ stores each value in a _bucket_, along with its key and hash. So we have to inspect every bucket and update its hash if the index is numeric. Internally, _array_unshift_ actuallyallocates a brand new array to do this, and replaces the old one when all the values have been copied over.

The index of a value in a _Vector_ is a direct mapping to its index in the buffer, so all we need to do is move every value in the range _[1, size - 1]_ to the right by one position. Internally, this is done using a single _memmove_ operation_._

Both _Ds\Deque_ and _SplDoublyLinkedList_ are very fast, because the time it takes to _unshift_ a value is not affected by the sample size, ie. _O(1)_

The following benchmark shows how memory usage is affected by 2ⁿ _pop_ operations, or from a size of 2ⁿ to zero.

What’s interesting here is that an _array_ always holds on to allocated memory, even if its size decreases substantially. _Ds\Vector_ and _Ds\Deque_ will halve their allocated capacity if their size drops below a quarter of their current capacity. _SplDoublyLinkedList_ will free each individual value’s memory, which is why we can see a linear decline.

![](https://cdn-images-1.medium.com/max/1600/1*i8dnM0yH9dHlhL-RuK2MAg.gif)

### Stack

A [_Stack_](https://en.wikipedia.org/wiki/Stack_%28abstract_data_type%29)is a _“last in, first out”_ or “_LIFO_” collection that only allows access to the value at the top of the structure and iterates in that order, destructively. You can think of it as a [firearm magazine](https://upload.wikimedia.org/wikipedia/commons/6/6c/Single_row_box_magazine.svg) with dynamic capacity.

_Ds\Stack_ uses a _Ds\Vector_ internally.

_SplStack_ extends _SplDoublyLinkedList_, so a performance comparison would be equivalent to comparing _Ds\Vector_ to _SplDoublyLinkedList_,as seen in the previous benchmarks. The following benchmark shows the time taken to perform 2ⁿ _pop_ operations, or from a size of 2ⁿ to zero.

![](https://cdn-images-1.medium.com/max/1600/1*yKfo29kCIPVkqFETXmVGRg.gif)

### Queue

A [_Queue_](https://en.wikipedia.org/wiki/Queue_%28abstract_data_type%29)is a _“first in, first out”_ or _“FIFO_”collection that only allows access to the value at the front of the queue and iterates in that order, destructively. You can think of it as a line of people queuing up for a ride at a theme park.

_Ds\Queue_ uses a _Ds\Deque_ internally. _SplQueue_ extends _SplDoublyLinkedList_, so a performance comparison would be equivalent to comparing _Ds\Deque_ to _SplDoublyLinkedList_,as seen in the previous benchmarks_._

### PriorityQueue

A _PriorityQueue_ is very similar to a _Queue._ Values are pushed into the queue with an assigned priority, and the value with the highest priority will always be at the front of the queue. Iterating over a _PriorityQueue_ is destructive, equivalent to successive _pop_ operations until the queue is empty.

Implemented using a _max heap._

**_First in, first out_ ordering is preserved for values with the same priority**, so multiple values with the same priority will behave exactly like a _Queue._ On the other hand, _SplPriorityQueue_ will remove values in arbitrary order.

The following benchmark shows the time taken and memory used to _push_ 2ⁿ random integers with a random priority into the queue. The same random numbers are used for each benchmark, and the _Queue_ benchmarkalso generates a random priority even though it doesn't use it for anything.

This is probably the most significant of all the benchmarks… _Ds\PriorityQueue_ is **more than twice as fast** as an _SplPriorityQueue,_ and uses only **5%** of its memory. That’s **20 times more memory efficient**.

But _how?_ How can the difference be that much when _SplPriorityQueue_ also uses a similar internal structure_?_ It all comes down to how a value is paired with a priority. _SplPriorityQueue_ allows any type of value to be used as a priority, which means that each priority pair takes up **32 bytes**.

_Ds\PriorityQueue_ only supports integer priorities, so each pair only allocates **24 bytes**. But that’s not nearly a big enough difference to explain the result.

If you take a look at the [source](http://lxr.php.net/xref/PHP_7_0/ext/spl/spl_heap.c#629) for _SplPriorityQueue::insert,_ you will notice that it actually **allocates an array** **to store the pair**.

Because an array has a minimum capacity of 8, each pair actually allocates _zval + HashTable +_ 8 * (_Bucket + hash_) + 2 * _zend_string_ + (8 + 16) byte string payloads _=_ 16 + 56 + 36 * 8 + 2 * 24 + 8 + 16 = **432 bytes**_(64 bit)._

> “So… why an array?”

_SplPriorityQueue_ uses the same internal structure as _SplMaxHeap,_ which requires that a value must be a _zval._ An obvious (but inefficient) way to create a _zval_ pair that is also a _zval_ itself is to use an _array_.

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