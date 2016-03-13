>Regular expressions documented in PHP
http://nyamsprod.com/blog/2015/regular-expressions-documented-in-php/

Tips to easily document complex regular expressions in PHP


One of the most challenging aspect of using regular expressions is documenting them. More than often you end up with a complex expression that even you as a creator have a hard time documenting. While reading [sitepoint recent article around regular expressions](http://www.sitepoint.com/demystifying-regex-with-practical-examples/) I was intrigued that this article did not feature any tips on how to document them.  So let’s take a very complex regular expression and see how we could improve its documentation.<span id="more-2488"></span>

For the purpose of this article we will look at [RFC3986 regular expression](http://tools.ietf.org/html/rfc3986#appendix-B) used to parse any given URI into its main parts. When adapted to be used in PHP you end up with the following code

```php
$uri = 'http://www.ics.uci.edu/pub/ietf/uri/#Related';
$regexp = ',^(([^:/?\#]+):)?(//([^/?\#]*))?([^?\#]*)(\?([^\#]*))?(\#(.*))?,';
preg_match($regexp, $uri, $matches);
var_dump($matches);

// array(10) {
//    [0]=> string(44) "http://www.ics.uci.edu/pub/ietf/uri/#Related"
//    [1]=> string(5) "http:"
//    [2]=> string(4) "http"
//    [3]=> string(17) "//www.ics.uci.edu"
//    [4]=> string(15) "www.ics.uci.edu"
//    [5]=> string(14) "/pub/ietf/uri/"
//    [6]=> string(0) ""
//    [7]=> string(0) ""
//    [8]=> string(8) "#Related"
//    [9]=> string(7) "Related"
// }
```

The regular expression works as expected but unless you have a very good memory there’s no way you’ll keep in mind which resulting index represents the URI path component. One way to overcome this problem is to add comments before the regular expression where each part is explained. But since its a comment you have no guarantee that someone else won’t override them at any point in the future. Thus, we need a better solution.

Thanks to the regular expression mechanism provided in PHP, since PHP 5.2, [you can named subpatterns in your regular expression](http://be2.php.net/manual/en/regexp.reference.subpatterns.php). Using this feature we improve the regular expression readability as well as its resulting array as shown below:

```php
$uri = 'http://www.ics.uci.edu/pub/ietf/uri/#Related';
$regexp = ',^((?<scheme>[^:/?\#]+):)?(?<authority>//([^/?\#]*))?(?<path>[^?\#]*)(?<query>\?([^\#]*))?(?<fragment>\#(.*))?,';
preg_match($regexp, $uri, $matches);
var_dump($matches);
// array(15) { 
//     [0]=> string(44) "http://www.ics.uci.edu/pub/ietf/uri/#Related"
//     [1]=> string(5) "http:"
//     ["scheme"]=> string(4) "http"
//     [2]=> string(4) "http"
//     ["authority"]=> string(17) "//www.ics.uci.edu"
//     [3]=> string(17) "//www.ics.uci.edu"
//     [4]=> string(15) "www.ics.uci.edu"
//     ["path"]=> string(14) "/pub/ietf/uri/"
//     [5]=> string(14) "/pub/ietf/uri/"
//     ["query"]=> string(0) ""
//     [6]=> string(0) ""
//     [7]=> string(0) ""
//     ["fragment"]=> string(8) "#Related"
//     [8]=> string(8) "#Related"
//     [9]=> string(7) "Related"
// }
```

Now, along with the previous numeric indexes, the returned array also includes named indexes for each selected subpattern. By assigning meaningful names to your named subpatterns you automatically document your regular expression. In our example, we use URI parts names to directly flag which pattern represents which URI part. The only drawback is that our regular expression becomes longer.

According to the [php documentation website](http://be2.php.net/manual/en/regexp.reference.internal-options.php) we can rely on the `x` PHP regular pattern modifier to help use better format the regular expression.

> If the `x` modifier is set, whitespace data characters in the pattern are totally ignored except when escaped or inside a character class, and characters between an unescaped # outside a character class and the next newline character, inclusive, are also ignored.

In other words, this modifier can be used to:

*   Format the regular expression on multiple lines.
*   Add inline comments to further document the regular expression.

So by combining named subpattern and regular pattern modifier, the regular expression ends up being:

```php
$uri = 'http://www.ics.uci.edu/pub/ietf/uri/#Related';
$regexp = ',^
        ((?<scheme>[^:/?\#]+):)?      # URI scheme component
        (?<authority>//([^/?\#]*))?   # URI authority part
        (?<path>[^?\#]*)              # URI path component
        (?<query>\?([^\#]*))?         # URI query component
        (?<fragment>\#(.*))?          # URI fragment component
        ,x';      
preg_match($regexp, $uri, $matches);
var_dump($matches);
// array(15) { 
//     [0]=> string(44) "http://www.ics.uci.edu/pub/ietf/uri/#Related"
//     [1]=> string(5) "http:"
//     ["scheme"]=> string(4) "http"
//     [2]=> string(4) "http"
//     ["authority"]=> string(17) "//www.ics.uci.edu"
//     [3]=> string(17) "//www.ics.uci.edu"
//     [4]=> string(15) "www.ics.uci.edu"
//     ["path"]=> string(14) "/pub/ietf/uri/"
//     [5]=> string(14) "/pub/ietf/uri/"
//     ["query"]=> string(0) ""
//     [6]=> string(0) ""
//     [7]=> string(0) ""
//     ["fragment"]=> string(8) "#Related"
//     [8]=> string(8) "#Related"
//     [9]=> string(7) "Related"
// }
```

As you can see without resorting to docblocks we have a relatively complex regular expression break down into simple parts and self documented. The matches are preserved and self documented to help developers understand the regular expression they wrote years ago.
