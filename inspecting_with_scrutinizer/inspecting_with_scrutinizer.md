>Inspecting PHP Code Quality with Scrutinizer
http://www.sitepoint.com/inspecting-php-code-quality-scrutinizer/

У нас уже было приличное количество руководств по качеству кода, тестированию, системам сбора билдов и так далее:

*   [PHP Quality Assurance with Jenkins](http://www.sitepoint.com/series/php-quality-assurance-with-jenkins/) [в 4-х частях]
*   [PHP and Continuous Integration with Travis CI](http://www.sitepoint.com/php-continuous-integration-travis-ci/)
*   [Visualize your Code Quality with PhpMetrics](http://www.sitepoint.com/visualize-codes-quality-phpmetrics/)
*   [Continuous Integration with PHP-CI](http://www.sitepoint.com/continuous-integration-php-ci/)

В этой статье мы взглянем на [Scrutinizer CI](https://scrutinizer-ci.com/) - инструмент для непрерывной интегракции, довольно дорогой и закрытый для коммерческих проектов, но очень удобный для публичных.

![](https://avatars1.githubusercontent.com/u/2988888?v=3)

## Scrutinizer vs/+ Travis

Scrutinizer performs many analyses that a compiler does in order to help you find potential bugs, security vulnerabilities, or violations of best practices. It also allows you to combine its results with those of some open-source tools like PHP Code Sniffer that we discussed in our 4-part series on Jenkins.

Where Travis is fully customizable, with various virtual environments running your code and letting you know of your build status, it doesn’t really have built-in support for quality assurance. Scrutinizer does, but it doesn’t run tests on public projects, and to power private ones, you need a paid plan.

Scrutinizer, therefore, cannot run PHPUnit for you, nor can it provide build status or code coverage. We can, however, configure Travis to send coverage reports _to_ Scrutinizer on every build. That way, whenever Travis makes a new build of your project, it automatically makes sure the Scrutinizer report is up to date, too.

## External Code Coverage

To get started with Scrutinizer, sign up for an account there. Then, connect your Github account with it and authorize access, so it can reach your repositories and verify you’re the owner. Finally, click the Add Repository button and add the one you want checked. Scrutinizer will automatically add a webhook to your repo, so that all events happening on it automatically trigger a scan process:

[![Github Scrutinizer Webhook](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429559175Screenshot-2015-04-20-21.45.53.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429559175Screenshot-2015-04-20-21.45.53.png)

I’ll assume you already know how to configure Travis. If not, please see [this post](http://www.sitepoint.com/php-continuous-integration-travis-ci/) and come back. At the end of our `.travis.yml` file, we need to add the following:

```
script:
  - phpunit --coverage-text --coverage-clover=coverage.clover
 
after_script:
  - wget https://scrutinizer-ci.com/ocular.phar
  - php ocular.phar code-coverage:upload --format=php-clover coverage.clover
```

This first instructs Travis on what to do in order to trigger a build process: run PHPUnit and produce a clover coverage report. After the build executes, Travis needs to download the `ocular.phar` helper from Scrutinizer, and uses this tool to send the coverage report over. This means that, if you prefer to bypass Travis altogether, you can use this tool from your own machine, too – just run the same commands locally.

## Configuration

When you add your project, Scrutinizer will automatically infer a configuration based on your project structure. It recognizes many common frameworks and CMSes like Symfony, Zend, Laravel, Drupal, Magento, or WordPress, and will automatically adjust its analysis to match that particular project. Scrutinizer also allows you to change its behavior through a config file. The [configuration](https://scrutinizer-ci.com/docs/configuration/best_practices) process is a bit too versatile for comfort, so I’ll try and simplify as best I can.

### Global / Repo

There are configurations that you save on the Scrutinizer website, into your account – _global_, and _repo_. **Global configuration** can be shared among repos (but doesn’t have to be – the word _global_ means it’s _globally available_, not _globally active_).

**Repo configuration** is configuration saved in a repo’s configuration section:

[![Configuration screen](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429556953Screenshot-2015-04-20-21.07.38.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429556953Screenshot-2015-04-20-21.07.38-1024x610.png)

At first, it’ll be empty, with some [default values implied](https://scrutinizer-ci.com/docs/guides/php/automated-code-reviews). To fine-tune it, one can just put in custom values as [per the docs](https://scrutinizer-ci.com/docs/configuration). In the case of [thephpleague/skeleton](https://github.com/thephpleague/skeleton), this is already beautifully defined, so you can rip off their configuration and shove it right in there:

```
filter:
    excluded_paths: [tests/*]
checks:
    php:
        code_rating: true
        remove_extra_empty_lines: true
        remove_php_closing_tag: true
        remove_trailing_whitespace: true
        fix_use_statements:
            remove_unused: true
            preserve_multiple: false
            preserve_blanklines: true
            order_alphabetically: true
        fix_php_opening_tag: true
        fix_linefeed: true
        fix_line_ending: true
        fix_identation_4spaces: true
        fix_doc_comments: true
tools:
    external_code_coverage:
        timeout: 600
        runs: 3
```

When a build process is triggered (and this will happen automatically due to the webhook), Travis will run the PHPUnit command and send over the coverage report when it’s done. A timeout of 600 seconds means that’s the maximum amount of time Scrutinizer will tolerate waiting for the code coverage report to arrive.

`runs` is useful for when you have your PHPUnit tests split up into several suites, each producing its own coverage report – specifying runs as X means the last X coverage reports sent to Scrutinizer will be _merged together and treated as one_. This is also useful when you’re testing for several environments on Travis – e.g. PHP 5.5, 5.6, and 7.0 – specifying a `runs` value of 3 will merge these reports. If `runs` is omitted, only the first coverage report Scrutinizer receives will be taken into account for code coverage and build success status.

### File Configuration

File based configuration is read from a `.scrutinizer.yml` file from your project’s root folder. It can contain the exact same values as all other configurations, but takes priority over them – i.e., values in the file will overwrite values in repo configuration, which will overwrite those in global configuration.

The [League Skeleton](https://github.com/thephpleague/skeleton) has one such file, and as such is ready for Scrutinizer testing by default. Feel free to use their file as an example on how to build your own.

### Local Configuration

Local configuration can be defined per-run, by clicking “Schedule Inspection”, which will allow one to punch in custom config values to be applied immediately before the run, and only on that one run:

![Schedule Inspection Custom Config](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429559929Screenshot-2015-04-20-21.58.37.png)

Local configuration is also one which you pass in via [API calls](https://scrutinizer-ci.com/docs/api/) as parameters.

This configuration overwrites all others before it while merging.

For a handy reference to these overwrites and the configuration hierarchy, as well as helpful hints for when to use which configuration type, see this [handy chart](https://scrutinizer-ci.com/docs/configuration/cascade).

## Reports

Now that configuration is out of the way, let’s inspect an actual project. For an example, I’ll use a past version of my [Diffbot client](https://github.com/Swader/diffbot-php-client) which we’ve built [in a previous series](http://www.sitepoint.com/series/how-to-build-your-own-php-package/). After adding it to Travis, configuring everything as mentioned above (using the identical configuration settings), the Scrutinizer build triggers.

### Dashboard

After completion, the project dashboard changes:

[![Project Dashboard](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429560674Screenshot-2015-04-20-16.22.54-1024x519.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429560674Screenshot-2015-04-20-16.22.54.png)

_The build process was considered failed originally due to a configuration misstep on my end, hence the error._

We can see that we have a very good rating in code quality, 100% test coverage, and 4 detected issues. Let’s see what those are.

### Issues

Clicking on either the “4 issues” link, or the issues icon in the left-hand side menu, we’re taken to the issues list:

[![Issues screen](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429560871Screenshot-2015-04-20-16.23.13-1024x523.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429560871Screenshot-2015-04-20-16.23.13.png)

Scrutinizer offers handy tags for issue categorization by type and severity. Oddly enough, the “Last Found” column reads “2 months ago” in spite of the project just having been added to Scrutinizer and the first inspection running just mere minutes ago. In some regard, this interface leaves something to be desired – it’d be nice, for example, if bugs were color coded per severity, and if there were an “expand” option to glance at the errors without going to a whole new screen for a minor issue.

Alright then, let’s see what the problem is. I chose to inspect the second entry: `Entity.php`:

[![Entity Inspection](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561341Screenshot-2015-04-20-16.24.36-1024x423.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561341Screenshot-2015-04-20-16.24.36.png)

Besides the fact that it would be nice to have the full file path printed at the top instead of just the file name, this interface is clean and very direct in pointing out one’s mistakes. In this particular case, it actually inspected the docblock of an interface’s method signature and warned me about not respecting it.

Indeed, once I changed this:

[![Fixed Code](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561768Screenshot-2015-04-20-16.31.01-1024x353.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561768Screenshot-2015-04-20-16.31.01.png)

to this:

[![Bugged Code](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561776Screenshot-2015-04-20-16.30.22-1024x334.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561776Screenshot-2015-04-20-16.30.22.png)

and fixed a similar `Api.php` bug as well, I committed, pushed, and the analysis was re-run, producing a positive output:

![Success Notification](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561868Screenshot-2015-04-20-16.47.25.png)

It even let me know via email:

[![Success Notification in Email](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561914Screenshot-2015-04-20-16.59.54-1024x888.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429561914Screenshot-2015-04-20-16.59.54.png)

The remaining two issues were unfixable due to an issue with [Scrutinizer’s analyzer](https://github.com/scrutinizer-ci/php-analyzer/issues/17). Due to their unfixable nature, ignoring these issues was the logical approach:

[![Hide Issues](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562740Screenshot-2015-04-20-22.44.23-1024x520.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562740Screenshot-2015-04-20-22.44.23.png)

This doesn’t really _resolve_ them, but it does hide them from the issues list so they don’t stand out in future inspections.

### Code

Clicking on the “Code” menu option, we’re taken to the code analysis screen which tells us about the quality of our classes.

[![Code Quality Screen](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562170Screenshot-2015-04-20-16.23.27-1024x654.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562170Screenshot-2015-04-20-16.23.27.png)

The `Product` class seems problematic. Let’s see what Scrutinizer is complaining about.

[![Product Class Analysis](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562268Screenshot-2015-04-20-16.27.12-1024x319.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562268Screenshot-2015-04-20-16.27.12.png)

Uh huh. So it’s the complexity that’s causing the problem? Hmm, let’s see “How to fix”.

[![How to Fix instructions](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562377Screenshot-2015-04-20-16.28.10-1024x417.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562377Screenshot-2015-04-20-16.28.10.png)

Now, while I absolutely love this approach of giving advice with followup links, there’s just nothing that can be done for this class. The file is just one big collection of getters, and there’s no way to extract a class, subclass, or interface out of it. Sadly, there’s no way to ignore just this one inspection like there is on issues.

There is another link on the “Code” screen, one perhaps less obvious: “Hot Spots”.

Hot spots displays the most “optimizable” areas of the code – those that Scrutinizer assumes would yield the greatest quality increase with the least amount of work:

[![Hot Spots Listed](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562924Screenshot-2015-04-20-16.54.11-1024x278.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429562924Screenshot-2015-04-20-16.54.11.png)

In my case, those were the aformentioned (unfixable) `Product` class, and two methods in the `Api` abstract class. Interesting. Let’s have a look at the `buildUrl` method’s problems.

[![BuildURL Method's problems](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429566880Screenshot-2015-04-20-16.54.45-1024x859.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429566880Screenshot-2015-04-20-16.54.45.png)

Hmm. Too many conditions? The `buildUrl` method takes various options from the API that calls it and builds a URL from them, so it’s only natural that it has “many” control structures. One could extract the loops into methods like `buildFieldString` and `buildOptionsString`, but what for? This is the only method that needs this string building, so introducing additional unnecessary overhead to fix this is not something I’m interested in.

The other method, `__construct` definitely leaves something to be desired. Let’s improve it somewhat and change it from this:

```php
public function __construct($url)
    {
        if (!is_string($url)) {
            throw new \InvalidArgumentException('URL param must be a string.');
        }
        $url = trim($url);
        if (strlen($url) < 4) {
            throw new \InvalidArgumentException('URL must be at least four characters in length');
        }
        if ($parts = parse_url($url)) {
            if (!isset($parts["scheme"])) {
                $url = "http://$url";
            }
        }
        $filtered_url = filter_var($url, FILTER_VALIDATE_URL);
        if (false === $filtered_url) {
            throw new \InvalidArgumentException('You provided an invalid URL: ' . $url);
        }
        $this->url = $filtered_url;
    }
```

to this:

```php
public function __construct($url)
{
    $url = trim((string)$url);
    if (strlen($url) < 4) {
        throw new \InvalidArgumentException(
            'URL must be a string of at least four characters in length'
        );
    }
 
    $url = (isset(parse_url($url)['scheme'])) ? $url : "http://$url";
 
    $filtered_url = filter_var($url, FILTER_VALIDATE_URL);
    if (!$filtered_url) {
        throw new \InvalidArgumentException(
            'You provided an invalid URL: ' . $url
        );
    }
 
    $this->url = $filtered_url;
}
```

Now if we trigger a rebuild by committing and pushing, we get this:

[![Dashboard shows improvements](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429568857Screenshot-2015-04-21-00.27.02-1024x266.png)](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/04/1429568857Screenshot-2015-04-21-00.27.02.png)

The quality has increased from a B rating to A for the `__construct` method – success!

## Inspections and Statistics

The two remaining screens are Inspections, which lists all the inspections done so far and their outcome, and Statistics and Trends, a dashboard of graphs displaying visual cues as to how the quality of your code progresses (or regresses) over time – I’ll leave those up to you to explore.

## Conclusion

Scrutinizer is a powerful tool in making sure your PHP code quality is top notch. It’s very easy to set up, and once it’s up and running requires minimal interaction to stay active and up to date. While the pricing tiers aren’t all too accessible to individuals and solo developers with private projects (starting at 50 Euro rather than 19 Euro as stated on the landing page), they’re very approachable to companies, and the free tier is more than enough for an open source project.

Have you tried Scrutinizer? Which code quality services do you use? Maybe their competition, Code Climate? Let us know.