>PSR-7 Accepted!
https://mwop.net/blog/2015-05-18-psr-7-accepted.html

Рад сообщить, что по состоянию на 06:00 18 мая 2015 года [http://www.php-fig.org/psr/psr-7](http://www.php-fig.org/psr/psr-7) PSR-7 (HTTP Message Interfaces) был принят!

## Дорога к PSR-7

Путь к PSR-7 был долгим и извилистым. Все началось летом 2012 года в качестве проекта предложения по работе с HTTP клиентами за авторством Бенджамина Эберлея _([Benjamin Eberlei](http://www.whitewashing.de/))_, в течение обсуждения которого другие участники предложили, что, возможно, небольшой стандарт на http-сообщения - вот что позволило бы решить все проблемы.

В этот момент Крис Уилкинсон _([Chris Wilkinson](https://github.com/thewilkybarkid))_ подхватил эстафету и создал формальный черновик, который и стал в последствии PSR-7. В нем он разработал оригинальный интерфейс для работы HTTP сообщений. Во время своего пребывания на посту редактора он также начал составление сопутствующего предложения по URI, но оно так никогда и не стало PSR. PSR-7 полировался в течение значительного срока, особенно часть об аспекте сохранения заголовка как части сообщения и имел `MessageInterface`, описывающий общие черты между двумя типами сообщений с описанием отличий запроса и ответа.

В определенный момент (я точно не знаю, до сих пор не нашел никакого формального анонса в архивах группы) эстафету передали Майклу Доулингу _([Michael Dowling](http://mtdowling.com/))_. Большим вкладом стало его предложение моделирования тела сообщения в виде потока. Это было довольно спорным внедрением, которое летом 2014го вытекло в [большой пост](http://mtdowling.com/blog/2014/07/03/a-case-for-higher-level-php-streams/) с подробным описанием зачем и почему было принято такое решение. По-моему, это потрясающе! Я проверил, каждый язык имеет превосходную HTTP абстракцию, моделирующую тело сообщения в виде потока: это позволяет работать асинхронно (что увы, довольно долго не поддерживалось в PHP), и также гарантирует, что большие сообщения не съедят всю доступную память. Сам PHP имеет модель сообщений как стримов (`php://input` и выходной буфер), поэтому именно этот путь является самым естественным.

Вскоре после написания этого поста в блоге, я сильно заигрался с [node.js](https://nodejs.org) и был поражен тем, как работают различные модули в плане обработки http. Это в значительной степени обусловлено тем, что в node.js по-умолчанию хорошо разработанная абстракция протокола http. Побочным эффектом наличия абстракции является возможность использовать одинаковую работу с http практически везде.

Я решил портировать [Connect](https://github.com/senchalabs/connect), промежуточную библиотеку, которая поставлялается с [Express](http://expressjs.com/), возможно самой распространенной библиотекой в node.js. Но делая это я столкнулся с непреодолимым препятствием: http в абстракции.

Конечно, каждый фреймворк имееет HTTP абстракцию. Я даже контрибьютил в один, предназначенный для Zend Framework. Но мои мысли были следующими: я хочу разобраться, как и многие PHP-разработчики, в этих понятиях, узнать как это работает, но мой выбор конкретной http абстракции мог повлиять на дальнейнее развитие проекта. И тогда я вспомнил, что видел пост  Михаэля о PSR-7 и потоках. Тогда-то и пришло понимание - "мне нужно нацелиться на PSR-7!"

Проблема заключалась в отсутствии имплементации.

So, I put together an implementation over a weekend, and went to the list with it. About two days after Michael posted saying he was going to abandon PSR-7 due to time constraints.

After a few weeks of discussion and heavy thinking, I decided to pick it up and try to move it forward. [Phil Sturgeon](https://philsturgeon.uk/) and [Beau Simensen](https://beau.io) agreed to continue as sponsor and coordinator, respectively, and so my tenure as editor began.

This has been an interesting journey for me. When I started, there were still debates about using streams; I had to quickly ramp up on the spec and Michael's arguments, and see if I agreed with them (I did), and, better, if I could defend them (I could).

I discovered there was another aspect I felt needed to be addressed though: the messages worked great for client-side aspects, but fell short for server-side: users were left to parse the URI for query string arguments, and to parse the body manually, and headers for cookies, and... well, this is where the "mostly" came in with Node: you end up having to do a lot of stuff yourself, or rely on a microframework for that stuff. I felt we could do better. Thus, the `ServerRequestInterface` was born, providing access to the data we take for granted in PHP's superglobals, but also providing some necessary abstraction for populating that data, as PHP sometimes does a poor job of it (e.g., you do not get a populated `$_POST` for non-POST requests or for POST requests without specific media types).

December 2014 came and went, and Phil resigned from FIG, leaving me in need of an additional sponsor. [Paul M Jones](http://paul-m-jones.com) graciously stepped up.

The server request additions had a fair bit of back and forth, but gained consensus in the end. Except that a certain amount of feedback concerned the fact that these were value objects, but mutable. So, in January, I took a deep dive into understanding value objects and immutability, and applied the concepts to the specification. We ended up with something that's very robust, without side effects, and which eliminates whole categories of potential problems due to changing state. This, too, was quite controversial, until folks saw actual real-world examples.

I also introduced a `UriInterface` in order to abstract the URI components. I and others discovered that we often needed to parse the URI to get at things like the scheme, path, host, etc., and that this was tedious, repetitious, and sometimes error-prone. Introducing a URI abstraction solves this. I tried to do so in such a way that we can be forward-compatible with a later, formal URI proposal, and borrowed heavily from work Chris Wilkinson had done earlier.

At this point, we decided to put it up for a vote. Initial results were quite positive, and it looked like we had a shoo-in. Except sometime during the second week, we got a lot of people reviewing the proposal for the first time. There's nothing like imminent acceptance to raise the interest of developers, and a number of flaws were found. Key among them were the fact that we weren't doing a great job of detailing the relationship between the URI and the Host header, nor were file uploads being handled particularly well.

So, we cancelled the vote around 24 hours before acceptance, and went back to draft stage. [Bernard Schussek](http://webmozarts.com/) provided some great justifications for an abstraction around file uploads which we ended up incorporating, and we ironed out the URI/host relationships in a much simpler fashion than we had previously.

And that took us to a second vote, which puts us where we are today: with a new PSR now accepted!

## Спасибо

Я, как редактор и в значительной степени публичное лицо, не могу не отметить, что это был результат многолетней работы множества людей. И особенно я хочу поблагодарить:

* _([Larry Garfield](http://wwww.garfieldtech.com/))_, который протестировал PSR-7 на некоторых проектах, и позволил получить фидбек о перспективах использования.
* _([Evert Pot](http://evertpot.com))_, который в конечном итоге проголосовал против принятия стандарта, но предоставил фантастические отзывы, принимая участие в обсуждении на протяжении всей жизни черновика. Его помощь была неоценима.
* Phil, Beau, и Paul, которые читали мои тирады, полные разочарования и неуверенности в себе на протяжении последних нескольких месяцев!