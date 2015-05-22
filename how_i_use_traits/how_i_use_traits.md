>How I Use Traits
http://rosstuck.com/how-i-use-traits/

Недавно несколько человек просили меня рассказать об использовании трейтов в новом проекте, который я пишу. Примерно в тоже время, Рафаэль Домс показал мне его новую речь о сложных когнитивных процессах, которые мы не замечаем. Так как наш мозг - это большой мешок, перемешивающий все, в результате получился пост, который пытается охватить и то как я используют трейты, и то как я решаю где они нужны. 

## Воздействие vs Абстракция

Первое, что вы должны сделать - это пойти почитать пост [“Abstraction or Leverage” от Майкла Найгарда](http://thinkrelevance.com/blog/2013/04/04/abstraction-or-leverage). Это отличная статья.

Если же у вас мало времени, основная суть поста состоит в том, что части кода (функции, классы, методы и т.д.) могут предназначаться либо для абстракции, либо для воздействия. Разница в:

* **Абстракция** содержит высокоуровневый концептуальный код, позволяющий более лаконично работать с ним другому коду.
* **Воздействие** содержит код, изменения в котором влияют лишь на определенную часть.

Общей абстракцией будет паттерн [Репозиторий](http://habrahabr.ru/post/248505/): вы не знаете как объект хранится или где, вам все равно. Детали лежат _вне_ концепции Репозитория.

Общим воздействием будет что-то вроде базового класса конроллера в вашем фреймворке. Он многое не скрывает, просто добавляет некоторые классные фичи, облегчающие работу.

В вышеупомянутом посте говорится, что как абстракции, так и воздействия хороши. Абстракция - немного лучше, потому что она всегда дает вам возможность воздействовать, но воздействие не дает вам абстракции. Тем не менее, я хотел бы добавить, что хорошая абстракция является более трудозатратной в создании и не на всех уровнях возможна. Так что это компромис.

## Как это связано с трейтами?

Некоторые возможности языка лучше, чем другие в создании либо воздействия, либо абстракции. Интерфейсы, например, отлично помогают нам строить и применять абстракции.

Наследование, с другой стороны, великолепно в предоставлении воздействия. Оно позволяет нам переопределить части родительского кода без необходимости копировать или извлекать каждый метод, использовать код класса (но не обязательно абстрактного) несколько раз. Так, отвечая на первоначальный вопрос, когда же я могу использовать трейты?

Я использую трейты, когда я хочу создать воздействие, не абстракцию.

Иногда.

## Иногда?

Бенджамин Эберлей высказа хороший аргумент, что [трейты имеют в основном те же проблемы, что и статический доступ](http://www.whitewashing.de/2013/04/12/traits_are_static_access.html). Вы не можете заменить или переорпеделить их, они откровенно плохо поддаются тестированию.

Но все же статические методы полезны. Если у вас одна функция без состояния и вы не хотите заменить ее на другую реализацию, то нет ничего плохого в том, чтобы сделать ее статической. Именованные конструкторы (вы же редко хотите именно пустой объект) или получение массива/результата математических операций с хорошо определенными вводом/выводом, без состояния, детерминированные: все это вам интересно. Статическое состояние, а не методы, вот реальное зло.

Трейты имеют примерно те же ограничения, плюс они могут быть использовании только внутри класса. Они более глобальны, чем объект.

Это дает дополнительную особенность, хотя черты: они могут читать и писать внутреннее состояние класса они подмешивали. Это делает их более подходящими для некоторых поведение, чем статический метод будет.

Это дает трейтам дополнительную особенность: они могут работать (читать и писать) с внутренним состоянием класса, в который подмешаны. В некоторых случаях это делает их более подходящими чем статические методы

Например, я часто использую генерацию доменных событий в сущности:

```php
trait GeneratesDomainEvents
{
    private $events = [];

    protected function raise(DomainEvent $event)
    {
        $this->events[] = $event;
    }

    public function releaseEvents()
    {
        $pendingEvents = $this->events;
        $this->events = [];
        return $pendingEvents;
    }
}
```

[While we can refactor this to an abstraction](https://gist.github.com/rosstuck/09804eed5fb9020a1ff0), it’s still a nice example of how a trait can work with local object state in a way static methods can’t. We don’t want to expose the events array blindly or place it outside the object. We might not want to force another abstraction inside our model but we certainly don’t want to copy and paste this boiler plate everywhere. Plain as they are, traits help us sidestep these issues.

Other practical examples might be custom logging functions that dump several properties at once or common iteration/searching logic. Admittedly, we could solve these with a parent class but we’ll talk about that in a moment.

So traits are a solid fit here but that doesn’t make static methods useless. In fact, I prefer static methods for behavior that doesn’t need the object’s internal state since it’s always safer to not provide it. Static methods are also more sharply defined and don’t require a mock class to be tested.

Assertions are a good example of where I prefer static methods, despite seeing them commonly placed in traits. Still, I find `Assertion::positiveNumber($int)` gives me the aforementioned benefits and it’s easier to understand what it is (or isn’t) doing to the calling class.

If you do have an assertion you’re tempted to turn into a trait, I’d treat it as a code smell. Perhaps it needs several parameters you’re tired of giving it. Perhaps validating `$this->foo` relies on the value of `$this->bar`. In either of these cases, refactoring to a value object can be a better alternative. Remember, it’s best if leverage eventually gives way to abstraction.

So, to restate: I use traits when I want leverage that needs access to an object’s internal state.

## Parent Classes

Everything we’ve seen is also possible through inheritance. An `EventGeneratingEntity` would arguably be even better since [the events array would be truly private](http://3v4l.org/M80Zl). However, traits offer the possibility of multiple inheritance instead of a single base class. Aside from their feature set, is there a good heuristic for choosing?

All things being equal, I like to fallback on something akin to the “Is-A vs Has-A” rule. It’s not an exact fit because _traits are not composition_ but it’s a reasonable guideline.

In other words, use parent classes for functionality that’s intrinsic to what the object is. A parent class is good at communicating this to other developers: “Employee is a Person”. Just because we’re going for leverage doesn’t mean the code shouldn’t be communicative.

For other non-core functionality on an object (fancy logging, event generation, boiler-plate code, etc), then a trait is an appropriate tool. It doesn’t define the nature of the class, it’s a supporting feature or better yet, an implementation detail. Whatever you get from a trait is just in service of the main object’s goal: traits can’t even pass a type check, that’s how unimportant they are.

So, in the case of the event generation, I prefer the trait to a base `EventGeneratingEntity` because Event Generation is a supporting feature.

## Interfaces

I rarely (if ever) extend a class or create a trait without also creating an interface.

If you follow this rule closely, you’ll find that traits can complement the Interface Segregation Principle well. It’s easy to define an interface for a secondary concern and then ship a trait with a simple default implementation.

This allows the concrete class to implement its own version of the interface or stick with the trait’s default version for unimportant cases. When your choices are boilerplate or forcing a poor abstraction, traits can be a powerful ally.

Still, if you’ve only got one class implementing the interface in your code and you don’t expect anyone else to use that implementation, don’t bother with a trait. You’re probably not making your code any more maintainable or readable.

## When I do not use traits

To be quite honest, I don’t use traits very often, perhaps once every few months. The heuristic I’ve outlined (leverage requiring access to internal state) is extremely niche. If you’re running into it very often, you probably need to step back and reexamine your style of programming. There’s a good chance you’ve got tons of objects waiting to be extracted.

There’s a few places I don’t like to use traits due to style preferences:

*   If the code you’re sharing is just a couple of getters and setters, I wouldn’t bother. IDEs can be your leverage here and adding a trait will only add mental overhead.
*   Don’t use traits for dependency injection. That’s less to do with traits themselves and more to do with setter injection, which I’m rather opposed to.
*   I don’t like traits for mixing in large public APIs or big pieces of functionality. Skip the leverage step and moving directly to finding an abstraction.

Finally, remember that while traits do not offer abstraction and they are not composition, they can still have a place in your toolbox. They’re useful for providing leverage over small default implementations or duplicate code. Always be ready to refactor them to a better abstraction once the code smells pile up.
