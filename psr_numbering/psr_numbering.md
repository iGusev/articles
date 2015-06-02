>A Quick Note on PSR Numbering
https://philsturgeon.uk/php/2015/05/05/psr7-numeric-workflow/


Последним выпущенным `PSR` от FIG был [PSR-4: Autoloader](http://www.php-fig.org/psr/psr-4/). Сейчас люди начинают узнавать о `PSR-7`, и сразу же начинается "lolphp", всем интересно куда же делись `PSR-5` и `PSR-6`. Я больше не являюсь частью FIG после голосования о смене главного контрибьютора PyroCMS, но я все еще могу сказать пару слов о PSR нумерации.

Сразу же стоит отметить, что это не похоже на [бесконечные дебаты "PHP6 vs PHP7"](https://philsturgeon.uk/php/2014/07/23/neverending-muppet-debate-of-php-6-v-php-7/), как может показаться на первый взгляд. Предвосхищая подобные случаи был создан специальный [Wokrflow-устав](https://github.com/php-fig/fig-standards/blob/master/bylaws/004-psr-workflow.md) для работы над черновиками будущих PSR.

Раньше работа над PSR выглядела примерно так:

* **Ссора:** Бл**ь, кто-то опять все поменял
* **Принятие:** Ой, вы были в отпуске? Ну... там приняли пару стандартов, и теперь наш будет просто `$new_id = end($psrs)->id + 1`

Теперь, с новым рабочим процессом, это выглядит так:

* **Пре-черновик:** Случайные наброски в интернете, всем наплевать
* **Черновик:** FIG обязательно поработает над этим, чтобы посмотреть что может получиться. Давайте дадим ему номер
* **Обсуждение:** Время показать наработки миру, _не следует_ вносить слишком большие изменения, в ином случае лучше вернуться к Черновику
* **Принятие:** Готово! Выдвинули на голосование. Что бы ни произошло, номер по-прежнему остается за проектом

Появилось несколько дополнительных этапов, ключевым отличием которых является присвоение номера **ДО** принятия черновика.

Поначалу это может показаться немного сложным, но именно так сделаны [`PEPs`](https://www.python.org/dev/peps/) в Python. Также как и в Python, `PSRs` имеют [индекс в git](https://github.com/php-fig/fig-standards/blob/master/index.md), также [отображаемый на сайте](http://www.php-fig.org/psr/), для пользователей.

Python выбрал такой подход, отлично, но _почему_ это работает?

Изначально FIG работал только над одним PSR одновременно. Конечно, `PSR-1` и `PSR-2` вышли одновременно, но только потому, что `PSR-2` откололся от `PSR-1`, сделав `PSR-1` немного менее спорным. Они не были предназначены для разных вещей в тот момент, когда зародились.

## Nicknames are weird

After PSR-3, and before anything else was being worked on, people referred to what is now PSR-4 as PSR-X or PSR-next. This was only meant to be a nickname and ended up sticking as a thing. Then people started saying things like PSR-C or PSR-Cache, and PSR-R was some alternative to PSR-X, and it got fucking mental. Bloggers and people on twitter thought those names were legit what was going to be used, and all hell broke loose when they found out it wasn’t PSR-X anymore.

## PSR Concurrency

Now, with people working on caching, HTTP messaging, phpdoc, etc., there is even more room for confusion. We can’t even shorten HTTP messaging because there have been discussions of a HTTP client in the past, so giving these all a single, stable number which can be used regardless of its status just made sense.

We could potentially have a “draft number” and an “accepted number” but what is the point? Who cares? All we needed was an index to keep track of what is what, avoid multiple conversations confusing which PSR is being talked about, and keep that number the same when and if things are done.

## But 0, 1, 2, 3, 4…7?!!?

So what? PSRs are not software, they are not SemVer, and they are not important. They are an arbitrary assignment which could have been A, B, C or Roman Numerals.

If people keep complaining about interruptions in the accepted auto-increment I’d be tempted to suggest switching the version numbers to sha1 hashes of the content like Git commits.

The identifier does not matter.

## But what if PSR-5 never comes out?!

Then PSR-5 will remain in Draft or whatever status it is in until somebody decides to have another go.

Even if PSR-1023482 just came out in the year 2050, PSR-5 could happily make its way to the stage. There is no semantic meaning to the order of these numbers, even though a lot of people inferred meaning from 0, 1, 2 being so closely related.

We have near unlimited integers to use for these things, so worrying about 5 being a missed opportunity is just not a valid concern, especially when it could come back later.

PSR-8 is a dead number. It is assigned to the silly April Fools joke from 2014: [Huggable Interface](https://github.com/php-fig/fig-standards/blob/master/proposed/psr-8-hug/psr-8-hug.md). PSR-8 will never be anything else. It will never be useful. It will _hopefully_ never be accepted.

## Fingers crossed for PSR-7

I am very impressed with the work and dedication to this PSR. It’s had a few people come and go during the process for various reasons, but an [Acceptance Vote](https://groups.google.com/forum/#!topic/php-fig/0baLqR6Rvcg) is underway and it is looking good.

Learn more about PSR-7 [from MWOP’s blog](http://mwop.net/blog/2015-01-08-on-http-middleware-and-psr-7.html), or from [MWOP’s interview on PHP Town Hall](http://phptownhall.com/blog/2015/02/02/episode-36-psr-7-the-world-of-tomorrow/).