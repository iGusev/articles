>Mastering Composer – Tips and Tricks
http://www.sitepoint.com/mastering-composer-tips-tricks/


Composer has revolutionized package management in PHP. It upped the reusability game and helped PHP developers all over the world generate framework agnostic, fully shareable code. But few people ever go beyond the basics, so this post will cover some useful tips and tricks.

![](https://getcomposer.org/img/logo-composer-transparent.png)

## Global

Although it’s clearly defined in the documentation, Composer can (and in most cases should) be installed globally. Global installation means that instead of typing out

```bash
php composer.phar somecommand
```

you can just type out

```bash
composer somecommand
```

in any project whatsoever. This makes starting new projects with, for example, the `create-project` command dead easy in any location on your filesystem.

To install Composer globally, follow [these instructions](https://getcomposer.org/doc/00-intro.md#globally).

## Installing packages the right way

When reading tutorials or README files of projects, many will say something like:

```
Just add the following to your composer.json file:
 
{
    "require": {
        "myproject": "someversion"
    }
}
```

But this has several downsides. One, the copy-pasting may introduce some errors. Two, for a newbie, figuring out where to place the code if you already have an extensive `composer.json` file in your project can be tedious and also introduce errors. Finally, many people will be encountering Composer for the first time and in a command line, so covering all the use cases in which they may find themselves isn’t feasible (do they have a GUI text editor or are they on the command line? If it’s the latter, do they have a text editor installed, and if so which? Do you explain the editing procedure or just leave it? What if the file doesn’t exist in their projects? Should you cover the creation of the file, too?).

The best way to add a new requirement to a `composer.json` file is with the `require command`:

```bash
composer require somepackage/somepackage:someversion
```

This adds everything that’s needed into the file, bypassing all manual intervention.

If you need to add packages to `require-dev`, add the `--dev` option, like so:

```bash
composer require phpunit/phpunit --dev
```

The `require` command supports adding several packages at once, just separate them with a space.

## Lock Files

The `composer.lock` file saves the list of currently installed packages, so that when another person clones your project at a date when the dependencies may have been updated, they still get the old versions installed. This helps make sure everyone who grabs your project has the exact same package environment as you did when the project was developed, avoiding any bugs that may have been created due to version updates.

`composer.lock` should [almost always](https://blog.engineyard.com/2014/composer-its-all-about-the-lock-file) be committed to version control. [Maybe](https://philsturgeon.uk/php/2014/11/04/composer-its-almost-always-about-the-lock-file/).

`composer.lock` also contains the hash of the `composer.json` file, so if you update just the project author, or some contact info, or a description, you’ll get a warning about the lock file not matching the `json` file – when that’s the case, running `composer update --lock` will help things, updating only the `lock` file and not touching anything else.

## Version flags

When defining [package versions](https://getcomposer.org/doc/01-basic-usage.md#package-versions), one can use exact matches (`1.2.3`), ranges with operators (`<1.2.3`), combinations of operators (`>1.2.3 <1.3`), best available (`1.2.*`), tilde (`~1.2.3`) and caret (`^1.2.3`).

The latter two might warrant further explanation:

*   tilde (`~1.2.3`) will go up to version `1.3` (not included), because in [semantic versioning](http://semver.org) that’s when new features get introduced. Tilde fetches the highest known stable minor version. As the docs say, we can consider it as **only** the last digit specified being allowed to change.

*   caret (`^1.2.3`) means “only be careful of breaking changes”, and will thus go up to version `2.0`. According to semver, that’s when breaking changes are introduced, so `1.3`,`1.4` and `1.9` are fine, while `2.0` is not.

Unless you know you need a specific version, I recommend always using the `~1.2.3` format – it’s your safest bet.

## Configuration and Global Configuration

The default values are not fixed in stone. See the full `config` [reference](https://getcomposer.org/doc/04-schema.md#config) for details.

For example, by specifying:

```
{
    "config": {
        "optimize-autoloader": true
    }
}
```

you force Composer to optimize the classmap after every installation/update, or in other words, whenever the autoload file is being generated. This is a little bit slower than generating the default autoloader, and slows down as the project grows.

Another useful option might be the `cache-files-maxsize` – in enormous projects like eZ Publish or Symfony, the cache might get full pretty fast. Increasing the size would keep Composer fast longer.

Note that configuration can be set globally, too, so it’s consistent across projects. See [here](https://getcomposer.org/doc/03-cli.md#config) for how. For example, to add the cache size setting to our global configuration, we either edit `~/.composer/config.json` or execute:

```bash
composer config --global cache-files-maxsize "2048MiB"
```

## Profile and Verbose

You can add a `--profile` flag to any command you execute on the command line with Composer, and it’ll produce not only a final output like this:

```bash
[174.6MB/54.70s] Memory usage: 174.58MB (peak: 513.47MB), time: 54.7s
```

but also prefix each line it outputs with the exact total duration of the command’s execution so far, plus the memory usage:

```bash
#[175.9MB/54.64s] Installing assets for Sensio\Bundle\DistributionBundle into web/bundles/sensiodistribution
```


I use this command often to identify the bottleneck packages and to observe how the stats improve or degrade on [different versions of PHP](https://www.sitepoint.com/php7-resource-recap/).

Likewise, the `--verbose` flag will make sure Composer outputs more information with each operation it performs, helping you understand _exactly_ what’s going on. Some people have even aliased their `composer` command to include `composer --verbose --profile` by default.

## Custom Sources

Sometimes, you just want to install from a Github repo if your project isn’t yet on Packagist. Maybe it’s under development, maybe it’s locally hosted, who knows. To do that, see [our guide](http://www.sitepoint.com/quick-tip-composer-github-develop-packages-interactively/).

Likewise, if you have your own version of a popular project that another part of your project depends on, you can use custom sources in combination with inline aliasing to fake the version constraint like Matthieu Napoli [did here](http://mnapoli.fr/overriding-dependencies-with-composer/).

## Speeding up Composer

As per this excellent trick by [Mark Van Eijk](http://markvaneijk.com/use-hhvm-to-speed-up-composer), you can speed up Composer’s execution by making it run on HHVM.

Another way is forcing it to use `--prefer-dist` which downloads a stable, packaged version of a project rather than cloning it from the version control system it’s on (much slower). This is on by default, though, so you shouldn’t need to specify it on stable projects. If you want to download the sources, use the `--prefer-source` flag. More info about this in the options of the `install` command [here](https://getcomposer.org/doc/03-cli.md#install).

## Making your Composer project lighter

If you’re someone who develops Composer-friendly projects, you might want to do your part, too. Based on [this Reddit thread](https://www.reddit.com/r/PHP/comments/2jzp6k/i_dont_need_your_tests_in_my_production/), you can use a `.gitattributes` file to ignore some of the files and folders during packaging for the `--prefer-dist` mode above.

```bash
/docs               export-ignore
/tests              export-ignore
/.gitattributes     export-ignore
/.gitignore         export-ignore
/.travis.yml        export-ignore
/phpunit.xml        export-ignore
```

How does this work? When you upload a project to Github, it automatically makes available the “Download zip” button which you can use to download an archive of your project. What’s more, Packagist uses these auto-generated archives to pull in the `--prefer-dist` dependencies, and then unarchives them once downloaded (much faster than cloning). If you thus ignore your tests, docs and other logically irrelevant files by listing them in `.gitattributes`, the archives won’t contain them, becoming much, much lighter.

Naturally, people who **want** to debug your library or run its tests should then specify the `--prefer-source` flag.

The [PhpLeague](http://thephpleague.com/) has adopted this approach and included it in their [Package skeleton](https://github.com/thephpleague/skeleton/blob/master/.gitattributes), so any project based on that is automatically “dist friendly”.

## Show

If you ever forget what version of PHP or its extensions you’re running, or need a list of all the projects (and their descriptions) that you’ve installed inside the current project and their versions, you can use the `show` command with the `--platform` (short `-p`) and `--installed` (short `-i`) flags respectively:

```bash
$ composer show --installed
 
behat/behat                            v3.0.15            #Scenario-oriented BDD framework for PHP 5.3
behat/gherkin                          v4.3.0             #Gherkin DSL parser for PHP 5.3
behat/mink                             v1.5.0             #Web acceptance testing framework for PHP 5.3
behat/mink-browserkit-driver           v1.1.0             #Symfony2 BrowserKit driver for Mink framework
behat/mink-extension                   v2.0.1             #Mink extension for Behat
behat/mink-goutte-driver               v1.0.9             #Goutte driver for Mink framework
behat/mink-sahi-driver                 v1.1.0             #Sahi.JS driver for Mink framework
behat/mink-selenium2-driver            v1.1.1             #Selenium2 (WebDriver) driver for Mink framework
behat/sahi-client                      dev-master ce7bfa7 #Sahi.js client for PHP 5.3
behat/symfony2-extension               v2.0.0             #Symfony2 framework extension for Behat
behat/transliterator                   v1.0.1             String transliterator
components/bootstrap                   3.3.2              #The most popular front-end framework for developing responsive, mobile first projects on the web.
components/jquery                      2.1.3              jQuery JavaScript Library
doctrine/annotations                   v1.2.4             Docblock Annotations Parser
doctrine/cache                         v1.4.1             #Caching library offering an object-oriented API for many cache backends
doctrine/collections                   v1.3.0             Collections Abstraction library
doctrine/common                        v2.5.0             #Common Library for Doctrine projects
doctrine/dbal                          v2.5.1             Database Abstraction Layer
doctrine/doctrine-bundle               v1.4.0             Symfony DoctrineBundle
doctrine/doctrine-cache-bundle         v1.0.1             #Symfony2 Bundle for Doctrine Cache
doctrine/inflector                     v1.0.1             Common String Manipulations with regard to casing and singular/plural rules.
doctrine/instantiator                  1.0.4              #A small, lightweight utility to instantiate objects in PHP without invoking their constructors
doctrine/lexer                         v1.0.1             #Base library for a lexer that can be used in Top-Down, Recursive Descent Parsers.
egulias/listeners-debug-command-bundle 1.9.1              Symfony 2 console command to debug listeners
ezsystems/behatbundle                  dev-master bd95e1b #Behat bundle for help testing eZ Bundles and projects
ezsystems/comments-bundle              dev-master 8f95bc7 #Commenting system for eZ Publish
ezsystems/demobundle                   dev-master c13fb0b #Demo bundle for eZ Publish Platform
ezsystems/demobundle-data              v0.1.0             #Data for ezsystems/demobundle
ezsystems/ezpublish-kernel             dev-master 3d6e48d eZ Publish API and kernel. This is the heart of eZ Publish 5.
ezsystems/platform-ui-assets-bundle    v0.5.0             #External assets dependencies for PlatformUIBundle
ezsystems/platform-ui-bundle           dev-master 4d0442d eZ Platform UI Bundle
ezsystems/privacy-cookie-bundle        v0.1               Privacy cookie banner integration bundle into eZ Publish/eZ Platform
fabpot/goutte                          v1.0.7             A simple PHP Web Scraper
friendsofsymfony/http-cache            1.3.1              Tools to manage cache invalidation
friendsofsymfony/http-cache-bundle     1.2.1              Set path based HTTP cache headers and send invalidation requests to your HTTP cache
guzzle/guzzle                          v3.9.3             #PHP HTTP client. This library is deprecated in favor of https://packagist.org/packages/guzzlehttp/guzzle
hautelook/templated-uri-bundle         2.0.0              Symfony2 Bundle that provides a RFC-6570 compatible router and URL Generator.
hautelook/templated-uri-router         2.0.1              Symfony2 RFC-6570 compatible router and URL Generator
imagine/imagine                        0.6.2              #Image processing for PHP 5.3
incenteev/composer-parameter-handler   v2.1.0             Composer script handling your ignored parameter file
instaclick/php-webdriver               1.0.17             #PHP WebDriver for Selenium 2
jdorn/sql-formatter                    v1.2.17            a PHP SQL highlighting library
knplabs/knp-menu                       v1.1.2             An object oriented menu library
knplabs/knp-menu-bundle                v1.1.2             This bundle provides an integration of the KnpMenu library
kriswallsmith/assetic                  v1.2.1             #Asset Management for PHP
kriswallsmith/buzz                     v0.13              Lightweight HTTP client
league/flysystem                       0.5.12             Many filesystems, one API.
liip/imagine-bundle                    1.2.6              #This Bundle assists in imagine manipulation using the imagine library
monolog/monolog                        1.13.1             Sends your logs to files, sockets, inboxes, databases and various web services
nelmio/cors-bundle                     1.3.3              #Adds CORS (Cross-Origin Resource Sharing) headers support in your Symfony2 application
ocramius/proxy-manager                 0.5.2              A library providing utilities to generate, instantiate and generally operate with Object Proxies
oneup/flysystem-bundle                 v0.4.2             Integrates Flysystem filesystem abstraction library to your Symfony2 project.
pagerfanta/pagerfanta                  v1.0.3             #Pagination for PHP 5.3
phpdocumentor/reflection-docblock      2.0.4              
phpspec/prophecy                       v1.4.1             #Highly opinionated mocking framework for PHP 5.3+
phpunit/php-code-coverage              2.0.16             #Library that provides collection, processing, and rendering functionality for PHP code coverage information.
phpunit/php-file-iterator              1.4.0              FilterIterator implementation that filters files based on a list of suffixes.
phpunit/php-text-template              1.2.0              Simple template engine.
phpunit/php-timer                      1.0.5              #Utility class for timing
phpunit/php-token-stream               1.4.1              Wrapper around PHPs tokenizer extension.
phpunit/phpunit                        4.6.4              The PHP Unit Testing framework.
phpunit/phpunit-mock-objects           2.3.1              #Mock Object library for PHPUnit
psr/log                                1.0.0              #Common interface for logging libraries
qafoo/rmf                              1.0.0              Very simple VC framework which makes it easy to build HTTP applications / REST webservices
sebastian/comparator                   1.1.1              #Provides the functionality to compare PHP values for equality
sebastian/diff                         1.3.0              Diff implementation
sebastian/environment                  1.2.2              Provides functionality to handle HHVM/PHP environments
sebastian/exporter                     1.2.0              #Provides the functionality to export PHP variables for visualization
sebastian/global-state                 1.0.0              Snapshotting of global state
sebastian/recursion-context            1.0.0              Provides functionality to recursively process PHP variables
sebastian/version                      1.0.5              Library that helps with managing the version number of Git-hosted PHP projects
sensio/distribution-bundle             v3.0.21            #Base bundle for Symfony Distributions
sensio/framework-extra-bundle          v3.0.7             This bundle provides a way to configure your controllers with annotations
sensio/generator-bundle                v2.5.3             #This bundle generates code for you
sensiolabs/security-checker            v2.0.2             #A security checker for your composer.lock
swiftmailer/swiftmailer                v5.4.0             Swiftmailer, free feature-rich PHP mailer
symfony-cmf/routing                    1.3.0              #Extends the Symfony2 routing component for dynamic routes and chaining several routers
symfony/assetic-bundle                 v2.6.1             Integrates Assetic into Symfony2
symfony/monolog-bundle                 v2.7.1             Symfony MonologBundle
symfony/swiftmailer-bundle             v2.3.8             Symfony SwiftmailerBundle
symfony/symfony                        v2.6.6             The Symfony PHP framework
tedivm/stash                           v0.12.3            The place to keep your cache.
tedivm/stash-bundle                    v0.4.2             Incorporates the Stash caching library into Symfony.
twig/extensions                        v1.2.0             #Common additional features for Twig that do not directly belong in core
twig/twig                              v1.18.1            #Twig, the flexible, fast, and secure template language for PHP
white-october/pagerfanta-bundle        v1.0.2             Bundle to use Pagerfanta with Symfony2
whiteoctober/breadcrumbs-bundle        1.0.2              #A small breadcrumbs bundle for Symfony2
zendframework/zend-code                2.2.10             provides facilities to generate arbitrary code using an object oriented interface
zendframework/zend-eventmanager        2.2.10             
zendframework/zend-stdlib              2.2.10             
zetacomponents/base                    1.9                The Base package provides the basic infrastructure that all packages rely on. Therefore every component relies on this package.
zetacomponents/feed                    1.4                #This component handles parsing and creating RSS1, RSS2 and ATOM feeds, with support for different feed modules (dc, content, creativeCommons, geo, iTunes).
zetacomponents/mail                    1.8.1              #The component allows you construct and/or parse Mail messages conforming to  the mail standard. It has support for attachments, multipart messages and HTML  mail. It also interfaces with SMTP to send mail or IMAP, P...
zetacomponents/system-information      1.1                Provides access to common system variables, such as CPU type and speed, and the available amount of memory.
```

## Dry Runs

To just see if an installation of new requirements would go well, you can use the `--dry-run` flag with Composer’s `install` and `update` command. This will throw all the potential problems at you, without actually causing them – no changes will really be made. Excellent for testing big requirement and setup changes before actually committing to them.

```bash
composer update --dry-run --profile --verbose
```

## Create Project

Last but not least, we must mention the [`create-project` command](https://getcomposer.org/doc/03-cli.md#create-project), applicable to anything and everything.

Create project takes a package name as the argument, then clones the package and executes `composer install` inside it. This is fantastic for bootstrapping projects – no more finding out the exact Github URL of the package you want, then cloning, then manually going into the folder and executing `install`.

Major projects such as Symfony and Laravel use this approach to bootstrap a skeleton application, and many others are jumping on board.

With Laravel, for example, it’s used like this:

```bash
composer create-project laravel/laravel --prefer-dist --profile --verbose
```

The `create-project` command also accepts two parameters. The first is the path into which to install. If omitted, the project’s name is used. The second is the version. If omitted, the latest version is used.

## Conclusion

Hope this list of tips and tricks has been helpful! If we missed some, do tell us and we’ll update the post! And remember – if you forget about some of the commands or switches, just check out the [cheatsheet](http://composer.json.jolicode.com/). Happy Composing!
