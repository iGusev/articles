>Demystifying RegEx with Practical Examples
http://www.sitepoint.com/demystifying-regex-with-practical-examples/

A regular expression is a sequence of characters used for parsing and manipulating strings. They are often used to perform searches, replace substrings and validate string data. This article provides tips, tricks, resources and steps for going through intricate regular expressions.

![regular expression](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/09/1442490508regex.jpg "Regular Expression")

There are many books, articles, websites and the [PHP official documentation](http://php.net/manual/en/regexp.reference.meta.php) that explain regular expressions, so instead of writing another explanation I’d prefer to go straight to more practical examples. You can find a useful cheat sheet at [this link](https://github.com/niklongstone/regular-expression-cheat-sheet).

Along with a host of useful resources, there is also a conference video by Lea Verou at the bottom of this post – it’s a bit long, but it’s excellent in breaking down RegEx.

## How to build a good regex

Regular expressions are often used in the developer’s daily routine – log analysis, form submission validation, find and replace, and so on. That’s why every good developer should know how to use them, but what is the best practice to build a good regex?

### 1. Define a scenario

Using natural language to define the problem will give you a better idea of the approach to use. The words **could** and **must**, used in a definition, are useful to describe mandatory constraints or assertions.

Below is an example:

* The string must start with ‘h’ and finish with ‘o’ (e.g. hello, halo).
* The string could be wrapped in parentheses.

### 2. Develop a plan

After having a good definition of the problem, we can understand the kind of elements that are involved in our regular expression:

* What are the types of characters allowed (word, digit, new line, range, …)?
* How many times must a character appear (one or more, once, …)?
* Are there some constraints to follow (optionals, lookahead/behind, if-then-else, …)?

### 3. Implement/Test/Refactor

It’s very important to have a real-time test environment to test and improve your regular expression. There are websites like [regex101.com](https://regex101.com/), [regexr.com](http://www.regexr.com/) and [debuggex.com](https://www.debuggex.com/) that provide some of the best environments.

To improve the efficiency of the regex, you could try to answer some of these additional questions:

* Are the character classes correctly defined for the specific domain?
* Should I write more test strings to cover more use cases?
* Is it possible to find and isolate some problems and test them separately?
* Should I refactor my expression with subpatterns, groups, conditions, etc., to make it smaller, clearer and more flexible?

## Practical examples

The goal of the following examples is not to write an expression that will only solve the problem, but to write the most effective expression for the specific use cases, using important elements like character ranges, assertions, conditions, groups and so on.

### Matching a password

![password match](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/09/1442490514password.jpg "password match")

**Scenario:**

* 6 to 12 characters in length
* Must have at least one uppercase letter
* Must have at least one lower case letter
* Must have at least one digit
* Should contain other characters

**Pattern:**

`^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{6,12}$`

This expression is based on multiple positive lookahead `(?=(regex))`. The lookahead matches something followed by the declared `(regex)`. The order of the conditions doesn’t affect the result. Lookaround expressions are very useful when there are several conditions.  
We could also use the negative lookahead `(?!(regex))` to exclude some character ranges. For example, I could exclude the `%` with `(?!.*#)`.

Let’s explain each pattern of the above expression:

1. `^` asserts position at start of the string
2. `(?=.*[a-z])` positive lookahead, asserts that the regex `.*[a-z]` can be matched:
    * `.*` matches any character (except newline) between zero and unlimited times
    * `[a-z]` matches a single character in the range between a and z (case sensitive)
3. `(?=.*[A-Z])` positive lookahead, asserts that the regex `.*[A-Z]` can be matched:
    * `.*` matches any character (except newline) between zero and unlimited times
    * `[A-Z]` matches a single character between A and Z (case sensitive)
4. (?=.*\d) positive lookahead, asserts that the regex `*\d`can be matched:
    * `.*` matches any character (except newline) between zero and unlimited times
    * `\d` matches a digit [0-9]
5. `.{6,12}` matches any character (except newline) between 6 and 12 times
6. `<div class="ArticleCopy language-" asserts position at end of the string

### Matching URL

![URL match](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/09/1442490495url.jpg)

**Scenario:**

* Must start with `http` or `https` or `ftp` followed by `://`
* Must match a valid domain name
* Could contain a port specification (`http://www.sitepoint.com:80`)
* Could contain digit, letter, dots, hyphens, forward slashes, multiple times

**Pattern:**

`^(http|https|ftp):[\/]{2}([a-zA-Z0-9\-\.]+\.[a-zA-Z]{2,4})(:[0-9]+)?\/?([a-zA-Z0-9\-\._\?\,\'\/\\\+&amp;%\$#\=~]*)`

The first scenario is pretty easy to solve with `^(http|https|ftp):[\/]{2}`.  
To match the domain name we need to bear in mind that to be valid it can only contain letters, digits, hyphen and dots. In my example, I limited the number of characters after the punctuation from 2 to 4, but could be extended for new domains like `.rocks` or `.codes`. The domain name is matched by `([a-zA-Z0-9\-\.]+\.[a-zA-Z]{2,4})`.

The optional port specification is matched by the simple `(:[0-9]+)?`.

A URL can contain multiple slashes and multiple characters repeated many times (see [RFC3986](http://tools.ietf.org/html/rfc3986#section-2)), this is matched by using a range of characters in a group `([a-zA-Z0-9\-\._\?\,\'\/\\\+&amp;%\$#\=~]*)`.  
It’s really useful to match every important element with a group capture `()`, because it will return only the matches we need. Remember that certain characters need to be escaped with `\`.

Below, every single subpattern explained:

1. `^` asserts position at start of the string
2. capturing group `(http|https|ftp)`, captures `http` or `https` or `ftp`
3. `:` escaped character, matches the character `:` literally
4. `[\/]{2}` matches exactly 2 times the escaped character `/`
5. capturing group `([a-zA-Z0-9\-\.]+\.[a-zA-Z]{2,4})`:
  * `[a-zA-Z0-9\-\.]+` matches one and unlimited times character in the range between a and z, A and Z, 0 and 9, the character `-` literally and the character `.` literally
  * `\.` matches the character `.` literally
  * `[a-zA-Z]{2,4}` matches a single character between 2 and 4 times between a and z or A and Z (case sensitive)
6. capturing group `(:[0-9]+)?`:
  * quantifier `?` matches the group between zero or more times
  * `:` matches the character `:` literally
  * `[0-9]+` matches a single character between 0 and 9 one or more times
7. `\/?` matches the character `/` literally zero or one time
8. capturing group `([a-zA-Z0-9\-\._\?\,\'\/\\\+&amp;%\$#\=~]*)`:
    * `[a-zA-Z0-9\-\._\?\,\'\/\\\+&amp;%\$#\=~]*` matches between zero and unlimited times a single character in the range a-z, A-Z, 0-9, the characters: `-._?,'/\+&amp;%$#=~`.

### Matching HTML TAG

![HTML TAG match](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/09/1442490503tag.jpg "HTML TAG match")

**Scenario:**

* The start tag must begin with `<` followed by one or more characters and end with `>`
* The end tag must start with `</` followed by one or more characters and end with `>`
* We must match the content inside a TAG element

**Pattern:**

`<([\w]+).*>(.*?)<\/\1>`

Matching the start tag and the content inside it’s pretty easy with `<([\w]+).*>` and `(.*?)`, but in the pattern above I have added a useful thing: the reference to a capturing group.  
Every capturing group defined by parentheses `()` could be referred to using its position number, `(first)(second)(third)`, which will allow for further operations.  
The expression above could be explained as:

* Start with `<`
* Capture the tag name
* Followed by one or more chars
* Capture the content inside the tag
* The closing tag must be `</tag name captured before>`

Including only two capture groups in the expression, the tag name and the content, will return a very clear match, a list of tag names with related content.

Let’s dig a little deeper and explain the subpatterns:

1. `<` matches the character `<` literally
2. capturing group `([\w]+)` matches any word character `a-zA-Z0-9_` one or more times
3. `.*` matches any character (except newline) between zero or more times
4. `>` matches the character `>` literally
5. capturing group `(.*?)`, matches any character (except newline), zero and more times
6. `<` matches the characters `<` literally
7. `\/` matches the character `/` literally
8. `\1` matches the same text matched by the first capturing group: `([\w]+)`
9. `>` matches the characters `>` literally

### Matching duplicated words

![HTML TAG match](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/09/1442490495url.jpg)

**Scenario:**

* The words are space separated
* We must match every duplication – non-consecutive ones as well

**Pattern:**

`\b(\w+)\b(?=.*\1)`

This regular expression seems challenging but uses some of the concept previously shown.  
The pattern introduces the concept of word boundaries.

A word boundary `\b` mainly checks positions. It matches when a word character (i.e.: `abcDE`) is followed by a non-word character (Ie: `-~,!`).  
Below you can find some example uses of word boundary to make it clearer:  
– Given the phrase `Regular expressions are awesome`  
– The pattern `\bare\b` matches `are`  
– The pattern `\w{3}\b` could match the last three letters of the words: `lar, ion, are, ome`

The expression above could be explained as:

* Match every word character followed by a non-word character (in our case space)
* Check if the matched word is already present or not

Below you will find the explanation for each sub pattern:

1. `\b` word boundary
2. capturing group `([\w]+)` matches any word character `a-zA-Z0-9_`
3. `\b` word boundary
4. `(?=.*\1)` positive lookahead assert that the following can be matched:
    * `.*` matches any character (except newline)
    * `\1` matches same text as first capturing group

The expression will make more sense if we return all the matches instead of returning only the first one. See the PHP function [`preg_match_all`](http://php.net/manual/en/function.preg-match-all.php) for more information.

## Final thoughts

Regular expressions are double-edged swords. The more complexity is added, the more difficult it is to solve the problem. That’s why, sometimes, it’s hard to find a regular expression that will match all the cases, and it’s better to use several smaller regex instead.

Having a good scenario of the problem could be very helpful, and will allow you to start thinking of the character range, constraints, assertions, repetitions, optional values, etc. Paying more attention to group captures will make the matches useful for further processing. Feel free to improve the expressions in the examples, and let us know how you do!

## Useful resources

Below you can find further information and resources to help your regex skills grow.  
Feel free to add a comment to the article if you find something useful that isn’t listed.

### Lea Verou – /Reg(exp){2}lained/: Demystifying Regular Expressions

[https://www.youtube.com/watch?v=EkluES9Rvak](https://www.youtube.com/watch?v=EkluES9Rvak)

### PHP libraries

<table>

<thead>

<tr>

<th align="left">Name</th>

<th align="left">Description</th>

</tr>

</thead>

<tbody>

<tr>

<td align="left">[RegExpBuilder](https://github.com/gherkins/regexpbuilderphp)</td>

<td align="left">Creates regex using human-readable chains of methods</td>

</tr>

<tr>

<td align="left">[NooNooFluentRegex](https://github.com/tomgray15/NooNooFluentRegex)</td>

<td align="left">Builds Regex expressions using fluent setters and English language terms like above</td>

</tr>

<tr>

<td align="left">[Hoa\Regex](https://github.com/hoaproject/Regex)</td>

<td align="left">Provides tools to analyze regex and generate strings</td>

</tr>

<tr>

<td align="left">[Regex reverse](https://github.com/niklongstone/regex-reverse)</td>

<td align="left">Given a regular expression will generate a string</td>

</tr>

</tbody>

</table>

### Websites

<table>

<thead>

<tr>

<th align="left">URL</th>

<th align="left">Description</th>

</tr>

</thead>

<tbody>

<tr>

<td align="left">[regex101.com](https://regex101.com/)</td>

<td align="left">PCRE online regex tester</td>

</tr>

<tr>

<td align="left">[regextester.com](http://www.regextester.com/)</td>

<td align="left">PCRE online regex tester</td>

</tr>

<tr>

<td align="left">[rexv.org](http://www.rexv.org/)</td>

<td align="left">PCRE online regex tester</td>

</tr>

<tr>

<td align="left">[debuggex.com](https://www.debuggex.com/)</td>

<td align="left">Supports PCRE and provides a very useful visual regex debugger</td>

</tr>

<tr>

<td align="left">[regexper.com](http://regexper.com/)</td>

<td align="left">Javascript style regex, but useful for debug</td>

</tr>

<tr>

<td align="left">[phpliveregex.com](http://www.phpliveregex.com/)</td>

<td align="left">Online tester for preg functions</td>

</tr>

<tr>

<td align="left">[regxlib.com](http://www.regxlib.com/)</td>

<td align="left">Database of regular expressions ready to use</td>

</tr>

<tr>

<td align="left">[regular-expressions.info](http://www.regular-expressions.info/)</td>

<td align="left">Regex tutorials, books review, examples</td>

</tr>

</tbody>

</table>

### Books

<table>

<thead>

<tr>

<th align="left">Title</th>

<th align="left">Description</th>

<th align="left">Author</th>

<th align="left">Editor</th>

</tr>

</thead>

<tbody>

<tr>

<td align="left">Mastering Regular Expressions</td>

<td align="left">The must have regex book</td>

<td align="left">Jeffrey Friedl</td>

<td align="left">O’Reilly</td>

</tr>

<tr>

<td align="left">Regular Expression Pocket Reference</td>

<td align="left">Regular Expressions for Perl, Ruby, PHP, Python, C, Java and .NET</td>

<td align="left">Tony Stubblebine</td>

<td align="left">O’Reilly</td>

</tr>

</tbody>

</table>

</div>