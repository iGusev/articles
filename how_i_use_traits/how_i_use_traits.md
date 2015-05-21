>How I Use Traits
http://rosstuck.com/how-i-use-traits/

Недавно несколько человек просили меня рассказать об использовании трейтов в новом проекте, который я пишу. Примерно в тоже время, Рафаэль Домс показал мне его новую речь о сложных когнитивных процессах, которые мы не замечаем. Так как наш мозг - это большой мешок, перемешивающий все, в результате получился пост, который пытается охватить и то как я используют трейты, и то как я решаю где они нужны. 

## Leverage vs Abstraction

The first thing you should do is go read this blog post: [“Abstraction or Leverage” from Michael Nygard](http://thinkrelevance.com/blog/2013/04/04/abstraction-or-leverage). It’s an excellent article.

If you’re short on time, the relevant part is that chunks of code (functions, classes, methods, etc) can be split out for either abstraction or leverage. The difference is:

*   **Abstraction** gathers similar code behind a higher level concept that’s more concise for other code to work with.
*   **Leverage** gathers similar code together so you only have one place to change it.

A common abstraction would be a Repository: you don’t know how an object is being stored or where but you don’t care. The details are _behind_ the concept of a Repository.

Common leverage would be something like your framework’s Controller base class. It doesn’t hide much, it just adds some nifty shortcuts that are easier to work with.

As the original blogpost points out, both abstraction and leverage are good. Abstraction is slightly better because it always gives you leverage but leverage doesn’t give you abstraction. However, I would add that a good abstraction is more expensive to produce and isn’t possible at every level. So, it’s a trade off.

## What’s this have to do with traits?

Some language features are better than others at producing either Leverage or Abstraction. Interfaces, for example, are great at helping us build and enforce abstractions.

Inheritance, on the other hand, is great at providing leverage. It lets us override select parts of the parent code without having to copy it or extract every method to a reusable (but not necessarily abstracted) class. So, to answer the original question, when do I use traits?

I use traits when I want leverage, not abstraction.

Sometimes.

## Sometimes?

Benjamin Eberlei makes a good argument that [traits have basically the same problems as static access](http://www.whitewashing.de/2013/04/12/traits_are_static_access.html). You can’t exchange or override them and they’re lousy for testing.

Still, static methods are useful. If you’ve got a single function with no state and you wouldn’t want to exchange it for another implementation, there’s nothing wrong with a static method. Think about named constructors (you rarely want to mock your domain models) or array/math operations (well defined input/output, stateless, deterministic). It makes you wonder if static state, rather than methods, are the real evil.

Traits have roughly the same constraints, plus they can only be used when mixed into a class. They’re more macro than object.

This does gives traits an extra feature though: they can read and write the internal state of the class they’re mixed into. This makes them more suitable for some behavior than a static method would be.

An example I often use is generating domain events on an entity:

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
