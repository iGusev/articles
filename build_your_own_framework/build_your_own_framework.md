>Build your own PHP Framework with Symfony Components
http://www.sitepoint.com/build-php-framework-symfony-components/

You’ve probably met Symfony in your PHP career – or have at least heard of it. What you may not know is that Symfony is, at its core, composed of separate libraries called _components_, which can be reused in any PHP application.

For example, the popular PHP framework [Laravel](http://laravel.com/) was developed using several Symfony components we will also be using in this tutorial. The next version of the popular CMS Drupal is also being built on top of some of the main Symfony components.

We’ll see how to build a minimal PHP framework using these components, and how they can interact to create a basic structure for any web application.

![](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/09/1412092488Fotolia_65666354_Subscription_Monthly_M.jpg)

**Note:** This tutorial won’t cover _every_ Symfony component and _every_ feature of each one. We’ll see only the main things that we need to build a minimal functional framework. If you want to go deeper into Symfony components, I encourage you to read their excellent [documentation](http://symfony.com/doc/current/components/index.html).

## Creating the project

We’ll start from scratch with a simple `index.php` file at the root of our project directory, and use [Composer](https://getcomposer.org/doc/00-intro.md) to install the dependencies.

For now, our file will only contain this simple piece of code:

<div>

<div id="highlighter_476291" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`switch``(``$_SERVER``[``'PATH_INFO'``]) {`</div>

<div class="line number2 index1 alt1">`case` `'/'``:`</div>

<div class="line number3 index2 alt2">`echo` `'This is the home page'``;`</div>

<div class="line number4 index3 alt1">`break``;`</div>

<div class="line number5 index4 alt2">`case` `'/about'``:`</div>

<div class="line number6 index5 alt1">`echo` `'This is the about page'``;`</div>

<div class="line number7 index6 alt2">`break``;  `</div>

<div class="line number8 index7 alt1">`default``:`</div>

<div class="line number9 index8 alt2">`echo` `'Not found!'``;`</div>

<div class="line number10 index9 alt1">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

This code just maps the requested URL (contained in `$_SERVER['PATH_INFO']`) to the right `echo` instruction. It’s a very, very primitive router.

## The HttpFoundation component

[HttpFoundation](http://symfony.com/doc/current/components/http_foundation/introduction.html) acts as a top-level layer for dealing with the HTTP flow. Its most important entrypoints are the two classes `Request` and `Response`.

`Request` allows us to deal with the HTTP request information such as the requested URI or the client headers, abstracting default PHP globals (`$_GET`, `$_POST`, etc.). `Response` is used to send back response HTTP headers and data to the client, instead of using `header` or `echo` as we would in “classic” PHP.

Install it using composer :

<div>

<div id="highlighter_743950" class="syntaxhighlighter  plain">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`php composer.phar require symfony/http-foundation 2.5.*`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

This will place the library into the `vendor` directory. Now put the following into the index.php file:

<div>

<div id="highlighter_409999" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`// Initializes the autoloader generated by composer`</div>

<div class="line number2 index1 alt1">`$loader` `=` `require` `'vendor/autoload.php'``;`</div>

<div class="line number3 index2 alt2">`$loader``->register();`</div>

<div class="line number5 index4 alt2">`use` `Symfony\Component\HttpFoundation\Request;`</div>

<div class="line number7 index6 alt2">`$request` `= Request::createFromGlobals();`</div>

<div class="line number9 index8 alt2">`switch``(``$request``->getPathInfo()) {`</div>

<div class="line number10 index9 alt1">`case` `'/'``:`</div>

<div class="line number11 index10 alt2">`echo` `'This is the home page'``;`</div>

<div class="line number12 index11 alt1">`break``;`</div>

<div class="line number13 index12 alt2">`case` `'/about'``:`</div>

<div class="line number14 index13 alt1">`echo` `'This is the about page'``;`</div>

<div class="line number15 index14 alt2">`break``;  `</div>

<div class="line number16 index15 alt1">`default``:`</div>

<div class="line number17 index16 alt2">`echo` `'Not found!'``;`</div>

<div class="line number18 index17 alt1">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

What we did here is pretty straightforward:

*   Create a `Request` instance using the `createFromGlobals` static method. Instead of creating an empty object, this method populates a `Request` object using the current request information.
*   Test the value returned by the `getPathInfo` method.

We can also replace the different `echo` commands by using a `Response` instance to hold our content, and `send` it to the client (which basically outputs the response headers and content).

<div>

<div id="highlighter_295589" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

<div class="line number20 index19 alt1">20</div>

<div class="line number21 index20 alt2">21</div>

<div class="line number22 index21 alt1">22</div>

<div class="line number23 index22 alt2">23</div>

<div class="line number24 index23 alt1">24</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`$loader` `=` `require` `'vendor/autoload.php'``;`</div>

<div class="line number2 index1 alt1">`$loader``->register();`</div>

<div class="line number4 index3 alt1">`use` `Symfony\Component\HttpFoundation\Request;`</div>

<div class="line number5 index4 alt2">`use` `Symfony\Component\HttpFoundation\Response;`</div>

<div class="line number7 index6 alt2">`$request` `= Request::createFromGlobals();`</div>

<div class="line number8 index7 alt1">`$response` `=` `new` `Response();`</div>

<div class="line number10 index9 alt1">`switch` `(``$request``->getPathInfo()) {`</div>

<div class="line number11 index10 alt2">`case` `'/'``:`</div>

<div class="line number12 index11 alt1">`$response``->setContent(``'This is the website home'``);`</div>

<div class="line number13 index12 alt2">`break``;`</div>

<div class="line number15 index14 alt2">`case` `'/about'``:`</div>

<div class="line number16 index15 alt1">`$response``->setContent(``'This is the about page'``);`</div>

<div class="line number17 index16 alt2">`break``;`</div>

<div class="line number19 index18 alt2">`default``:`</div>

<div class="line number20 index19 alt1">`$response``->setContent(``'Not found !'``);`</div>

<div class="line number21 index20 alt2">`$response``->setStatusCode(Response::HTTP_NOT_FOUND);`</div>

<div class="line number22 index21 alt1">`}`</div>

<div class="line number24 index23 alt1">`$response``->send();`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

## Use HttpKernel to wrap the framework core

<div>

<div id="highlighter_384193" class="syntaxhighlighter  plain">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`php composer.phar require symfony/http-kernel 2.5.*`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

For now, as simple as it is, the framework logic is still located in our front controller, the _index.php_ file. If we wanted to add more code, it would be better to wrap it into another class, which would become the “core” of our framework.

The [HttpKernel](http://symfony.com/doc/current/components/http_kernel/index.html) component was conceived with that goal in mind. It is intended to work with HttpFoundation to convert the Request instance to a Response one, and provides several classes for us to achieve this. The only one we will use, for the moment, is the `HttpKernelInterface` interface. This interface defines only one method: `handle`.

This method takes a `Request` instance as an argument, and is supposed to return a `Response`. So, each class implementing this interface is able to process a `Request` and return the appropriate `Response` object.

Let’s create the class `Core` of our framework that implements the `HttpKernelInterface`. Now create the `Core.php` file under the `lib/Framework` directory:

<div>

<div id="highlighter_951139" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

<div class="line number20 index19 alt1">20</div>

<div class="line number21 index20 alt2">21</div>

<div class="line number22 index21 alt1">22</div>

<div class="line number23 index22 alt2">23</div>

<div class="line number24 index23 alt1">24</div>

<div class="line number25 index24 alt2">25</div>

<div class="line number26 index25 alt1">26</div>

<div class="line number27 index26 alt2">27</div>

<div class="line number28 index27 alt1">28</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`<?php`</div>

<div class="line number3 index2 alt2">`namespace` `Framework;`</div>

<div class="line number5 index4 alt2">`use` `Symfony\Component\HttpFoundation\Request;`</div>

<div class="line number6 index5 alt1">`use` `Symfony\Component\HttpFoundation\Response;`</div>

<div class="line number7 index6 alt2">`use` `Symfony\Component\HttpKernel\HttpKernelInterface;`</div>

<div class="line number9 index8 alt2">`class` `Core` `implements` `HttpKernelInterface`</div>

<div class="line number10 index9 alt1">`{`</div>

<div class="line number11 index10 alt2">`public` `function` `handle(Request` `$request``,` `$type` `= HttpKernelInterface::MASTER_REQUEST,` `$catch` `= true)`</div>

<div class="line number12 index11 alt1">`{`</div>

<div class="line number13 index12 alt2">`switch` `(``$request``->getPathInfo()) {`</div>

<div class="line number14 index13 alt1">`case` `'/'``:`</div>

<div class="line number15 index14 alt2">`$response` `=` `new` `Response(``'This is the website home'``);`</div>

<div class="line number16 index15 alt1">`break``;`</div>

<div class="line number18 index17 alt1">`case` `'/about'``:`</div>

<div class="line number19 index18 alt2">`$response` `=` `new` `Response(``'This is the about page'``);`</div>

<div class="line number20 index19 alt1">`break``;`</div>

<div class="line number22 index21 alt1">`default``:`</div>

<div class="line number23 index22 alt2">`$response` `=` `new` `Response(``'Not found !'``, Response::HTTP_NOT_FOUND);`</div>

<div class="line number24 index23 alt1">`}`</div>

<div class="line number26 index25 alt1">`return` `$response``;`</div>

<div class="line number27 index26 alt2">`}`</div>

<div class="line number28 index27 alt1">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

**Note:** The `handle` method takes two more optional arguments: the request type, and a boolean indicating if the kernel should throw an exception in case of error. We won’t use them in this tutorial, but we need to implement the exact same method defined by `HttpKernelInterface`, otherwise PHP will throw an error.

The only thing we did here is move the existing code into the `handle` method. Now we can get rid of this code in `index.php` and use our freshly created class instead:

<div>

<div id="highlighter_909245" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`require` `'lib/Framework/Core.php'``;`</div>

<div class="line number3 index2 alt2">`$request` `= Request::createFromGlobals();`</div>

<div class="line number5 index4 alt2">`// Our framework is now handling itself the request`</div>

<div class="line number6 index5 alt1">`$app` `=` `new` `Framework\Core();`</div>

<div class="line number8 index7 alt1">`$response` `=` `$app``->handle(``$request``);`</div>

<div class="line number9 index8 alt2">`$response``->send();`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

## A better routing system

There is still a problem with our class: it is holding the routing logic of our application. If we wanted to add more URLs to match, we would have to modify the code inside our framework – which is clearly not a good idea. Moreover, this would mean adding a `case` block for each new route. No, we definitely don’t want to go down that dirty road.

The solution is to add a routing system to our framework. We can do this by creating a `map` method that binds a URI to a PHP callback that will be executed if the right URI is matched:

<div>

<div id="highlighter_871888" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

<div class="line number20 index19 alt1">20</div>

<div class="line number21 index20 alt2">21</div>

<div class="line number22 index21 alt1">22</div>

<div class="line number23 index22 alt2">23</div>

<div class="line number24 index23 alt1">24</div>

<div class="line number25 index24 alt2">25</div>

<div class="line number26 index25 alt1">26</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`class` `Core` `implements` `HttpKernelInterface`</div>

<div class="line number2 index1 alt1">`{`</div>

<div class="line number3 index2 alt2">`protected` `$routes` `=` `array``();`</div>

<div class="line number5 index4 alt2">`public` `function` `handle(Request` `$request``,` `$type` `= HttpKernelInterface::MASTER_REQUEST,` `$catch` `= true)`</div>

<div class="line number6 index5 alt1">`{`</div>

<div class="line number7 index6 alt2">`$path` `=` `$request``->getPathInfo();`</div>

<div class="line number9 index8 alt2">`// Does this URL match a route?`</div>

<div class="line number10 index9 alt1">`if` `(``array_key_exists``(``$path``,` `$this``->routes)) {`</div>

<div class="line number11 index10 alt2">`// execute the callback`</div>

<div class="line number12 index11 alt1">`$controller` `=` `$routes``[``$path``];`</div>

<div class="line number13 index12 alt2">`$response` `=` `$controller``();`</div>

<div class="line number14 index13 alt1">`}` `else` `{`</div>

<div class="line number15 index14 alt2">`// no route matched, this is a not found.`</div>

<div class="line number16 index15 alt1">`$response` `=` `new` `Response(``'Not found!'``, Response::HTTP_NOT_FOUND);`</div>

<div class="line number17 index16 alt2">`}`</div>

<div class="line number19 index18 alt2">`return` `$response``;`</div>

<div class="line number20 index19 alt1">`}`</div>

<div class="line number22 index21 alt1">`// Associates an URL with a callback function`</div>

<div class="line number23 index22 alt2">`public` `function` `map(``$path``,` `$controller``) {`</div>

<div class="line number24 index23 alt1">`$this``->routes[``$path``] =` `$controller``;`</div>

<div class="line number25 index24 alt2">`}`</div>

<div class="line number26 index25 alt1">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

Now application routes can be set directly in the front controller:

<div>

<div id="highlighter_376485" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`$app``->map(``'/'``,` `function` `() {`</div>

<div class="line number2 index1 alt1">`return` `new` `Response(``'This is the home page'``);`</div>

<div class="line number3 index2 alt2">`});`</div>

<div class="line number5 index4 alt2">`$app``->map(``'/about'``,` `function` `() {`</div>

<div class="line number6 index5 alt1">`return` `new` `Response(``'This is the about page'``);`</div>

<div class="line number7 index6 alt2">`});`</div>

<div class="line number9 index8 alt2">`$response` `=` `$app``->handle(``$request``);`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

This tiny routing system is working well, but it has major flaws: what if we wanted to match dynamic URLs that hold parameters? We could imagine a URL like `posts/:id` where `:id` is a variable parameter that could map to a post ID in a database.

We need a more flexible and powerful system: that’s why we’ll use the Symfony [Routing](http://symfony.com/doc/current/book/routing.html) component.

<div>

<div id="highlighter_176118" class="syntaxhighlighter  plain">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`php composer.phar require symfony/routing 2.5.*`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

Using the Routing component allows us to load `Route` objects into a `UrlMatcher` that will map the requested URI to a matching route. This `Route` object can contain any attributes that can help us execute the right part of the application. In our case, such an object will contain the PHP callback to execute if the route matches. Also, any dynamic parameters contained in the URL will be present in the route attributes.

In order to implement this, we need to do the following changes:

*   Replace the `routes` array with a `RouteCollection` instance to hold our routes.
*   Change the `map` method so it registers a `Route` instance into this collection.
*   Create a `UrlMatcher` instance and tell it how to match its routes against the requested URI by providing a context to it, using a `RequestContext` instance.

<div>

<div id="highlighter_492070" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

<div class="line number20 index19 alt1">20</div>

<div class="line number21 index20 alt2">21</div>

<div class="line number22 index21 alt1">22</div>

<div class="line number23 index22 alt2">23</div>

<div class="line number24 index23 alt1">24</div>

<div class="line number25 index24 alt2">25</div>

<div class="line number26 index25 alt1">26</div>

<div class="line number27 index26 alt2">27</div>

<div class="line number28 index27 alt1">28</div>

<div class="line number29 index28 alt2">29</div>

<div class="line number30 index29 alt1">30</div>

<div class="line number31 index30 alt2">31</div>

<div class="line number32 index31 alt1">32</div>

<div class="line number33 index32 alt2">33</div>

<div class="line number34 index33 alt1">34</div>

<div class="line number35 index34 alt2">35</div>

<div class="line number36 index35 alt1">36</div>

<div class="line number37 index36 alt2">37</div>

<div class="line number38 index37 alt1">38</div>

<div class="line number39 index38 alt2">39</div>

<div class="line number40 index39 alt1">40</div>

<div class="line number41 index40 alt2">41</div>

<div class="line number42 index41 alt1">42</div>

<div class="line number43 index42 alt2">43</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`use` `Symfony\Component\Routing\Matcher\UrlMatcher;`</div>

<div class="line number2 index1 alt1">`use` `Symfony\Component\Routing\RequestContext;`</div>

<div class="line number3 index2 alt2">`use` `Symfony\Component\Routing\RouteCollection;`</div>

<div class="line number4 index3 alt1">`use` `Symfony\Component\Routing\Route;`</div>

<div class="line number5 index4 alt2">`use`</div>

<div class="line number6 index5 alt1">`Symfony\Component\Routing\Exception\ResourceNotFoundException;`</div>

<div class="line number8 index7 alt1">`class` `Core` `implements` `HttpKernelInterface`</div>

<div class="line number9 index8 alt2">`{`</div>

<div class="line number10 index9 alt1">`/** @var RouteCollection */`</div>

<div class="line number11 index10 alt2">`protected` `$routes``;`</div>

<div class="line number13 index12 alt2">`public` `function` `__construct()`</div>

<div class="line number14 index13 alt1">`{`</div>

<div class="line number15 index14 alt2">`$this``->routes =` `new` `RouteCollection();`</div>

<div class="line number16 index15 alt1">`}`</div>

<div class="line number18 index17 alt1">`public` `function` `handle(Request` `$request``,` `$type` `= HttpKernelInterface::MASTER_REQUEST,` `$catch` `= true)`</div>

<div class="line number19 index18 alt2">`{`</div>

<div class="line number20 index19 alt1">`// create a context using the current request`</div>

<div class="line number21 index20 alt2">`$context` `=` `new` `RequestContext();`</div>

<div class="line number22 index21 alt1">`$context``->fromRequest(``$request``);`</div>

<div class="line number24 index23 alt1">`$matcher` `=` `new` `UrlMatcher(``$this``->routes,` `$context``);`</div>

<div class="line number26 index25 alt1">`try` `{`</div>

<div class="line number27 index26 alt2">`$attributes` `=` `$matcher``->match(``$request``->getPathInfo());`</div>

<div class="line number28 index27 alt1">`$controller` `=` `$attributes``[``'controller'``];`</div>

<div class="line number29 index28 alt2">`$response` `=` `$controller``();`</div>

<div class="line number30 index29 alt1">`}` `catch` `(ResourceNotFoundException` `$e``) {`</div>

<div class="line number31 index30 alt2">`$response` `=` `new` `Response(``'Not found!'``, Response::HTTP_NOT_FOUND);`</div>

<div class="line number32 index31 alt1">`}`</div>

<div class="line number34 index33 alt1">`return` `$response``;`</div>

<div class="line number35 index34 alt2">`}`</div>

<div class="line number37 index36 alt2">`public` `function` `map(``$path``,` `$controller``) {`</div>

<div class="line number38 index37 alt1">`$this``->routes->add(``$path``,` `new` `Route(`</div>

<div class="line number39 index38 alt2">`$path``,`</div>

<div class="line number40 index39 alt1">`array``(``'controller'` `=>` `$controller``)`</div>

<div class="line number41 index40 alt2">`));`</div>

<div class="line number42 index41 alt1">`}`</div>

<div class="line number43 index42 alt2">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

The `match` method tries to match the URL against a known route pattern, and returns the corresponding route attributes in case of success. Otherwise it throws a `ResourceNotFoundException` that we can catch to display a 404 page.

We can now take advantage of the Routing component to retrieve any URL parameters. After getting rid of the `controller` attribute, we can call our callback function by passing other parameters as arguments (using the `call_user_func_array` function):

<div>

<div id="highlighter_988147" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`try` `{`</div>

<div class="line number2 index1 alt1">`$attributes` `=` `$matcher``->match(``$request``->getPathInfo());`</div>

<div class="line number3 index2 alt2">`$controller` `=` `$attributes``[``'controller'``];`</div>

<div class="line number4 index3 alt1">`unset(``$attributes``[``'controller'``]);`</div>

<div class="line number5 index4 alt2">`$response` `= call_user_func_array(``$controller``,` `$attributes``);`</div>

<div class="line number6 index5 alt1">`}` `catch` `(ResourceNotFoundException` `$e``) {`</div>

<div class="line number7 index6 alt2">`$response` `=` `new` `Response(``'Not found!'``, Response::HTTP_NOT_FOUND);`</div>

<div class="line number8 index7 alt1">`}`</div>

<div class="line number10 index9 alt1">`return` `$response``;`</div>

<div class="line number11 index10 alt2">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

We can now easily handle dynamic URLs like this:

<div>

<div id="highlighter_370263" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`$app``->map(``'/hello/{name}'``,` `function` `(``$name``) {`</div>

<div class="line number2 index1 alt1">`return` `new` `Response(``'Hello '``.``$name``);`</div>

<div class="line number3 index2 alt2">`});`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

Note that this is very similar to what the Symfony full-stack framework is doing: we inject URL parameters into the right controller.

## Hooking into the framework

The Symfony framework also provides various way to hook into the request lifecycle and to change it. A good example is the security layer intercepting a request which attempts to load an URL between a firewall.

All of this is possible thanks to the [EventDispatcher](http://symfony.com/doc/current/components/event_dispatcher/introduction.html) component, which allows different components of an application to communicate implementing the [Observer](http://en.wikipedia.org/wiki/Observer_pattern) pattern.

<div>

<div id="highlighter_960791" class="syntaxhighlighter  plain">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`php composer.phar require symfony/event-dispatcher 2.5`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

At the core of it, there is the EventDispatcher class, which registers listeners of a particular event. When the dispatcher is notified of an event, all known listeners of this event are called. A listener can be any valid PHP callable function or method.

We can implement this in our framework by adding a property `dispatcher` that will hold an `EventDispatcher` instance, and an `on` method, to bind an event to a PHP callback. We’ll use the dispatcher to register the callback, and to fire the event later in the framework.

<div>

<div id="highlighter_53831" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

<div class="line number20 index19 alt1">20</div>

<div class="line number21 index20 alt2">21</div>

<div class="line number22 index21 alt1">22</div>

<div class="line number23 index22 alt2">23</div>

<div class="line number24 index23 alt1">24</div>

<div class="line number25 index24 alt2">25</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`use` `Symfony\Component\Routing\Matcher\UrlMatcher;`</div>

<div class="line number2 index1 alt1">`use` `Symfony\Component\Routing\RequestContext;`</div>

<div class="line number3 index2 alt2">`use` `Symfony\Component\Routing\RouteCollection;`</div>

<div class="line number4 index3 alt1">`use` `Symfony\Component\Routing\Route;`</div>

<div class="line number5 index4 alt2">`use` `Symfony\Component\Routing\Exception\ResourceNotFoundException;`</div>

<div class="line number6 index5 alt1">`use` `Symfony\Component\EventDispatcher\EventDispatcher`</div>

<div class="line number8 index7 alt1">`class` `Core` `implements` `HttpKernelInterface`</div>

<div class="line number9 index8 alt2">`{`</div>

<div class="line number10 index9 alt1">`/** @var RouteCollection */`</div>

<div class="line number11 index10 alt2">`protected` `$routes``;`</div>

<div class="line number13 index12 alt2">`public` `function` `__construct()`</div>

<div class="line number14 index13 alt1">`{`</div>

<div class="line number15 index14 alt2">`$this``->routes =` `new` `RouteCollection();`</div>

<div class="line number16 index15 alt1">`$this``->dispatcher =` `new` `EventDispatcher();`</div>

<div class="line number17 index16 alt2">`}`</div>

<div class="line number19 index18 alt2">`// ...`</div>

<div class="line number21 index20 alt2">`public` `function` `on(``$event``,` `$callback``)`</div>

<div class="line number22 index21 alt1">`{`</div>

<div class="line number23 index22 alt2">`$this``->dispatcher->addListener(``$event``,` `$callback``);`</div>

<div class="line number24 index23 alt1">`}`</div>

<div class="line number25 index24 alt2">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

We are now able to register listeners, which are just simple PHP callbacks. Let’s write now a `fire` method which will tell our dispatcher to notify all the listeners he knows when some event occurs.

<div>

<div id="highlighter_598345" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`public` `function` `fire(``$event``)`</div>

<div class="line number2 index1 alt1">`{`</div>

<div class="line number3 index2 alt2">`return` `$this``->dispatcher->dispatch(``$event``);`</div>

<div class="line number4 index3 alt1">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

In less than ten lines of code, we just added a nice event listener system to our framework, thanks to the EventDispatcher component.

The `dispatch` method also takes a second argument, which is the dispatched event object. Every event inherits from the generic `Event` class, and is used to hold any information related to it.

Let’s write a `RequestEvent` class, which will be immediately fired when a request is handled by the framework. Of course, this event must have access to the current request, using an attribute holding a `Request` instance.

<div>

<div id="highlighter_137212" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`namespace` `Framework\Event;`</div>

<div class="line number3 index2 alt2">`use` `Symfony\Component\HttpFoundation\Request;`</div>

<div class="line number4 index3 alt1">`use` `Symfony\Component\EventDispatcher\Event;`</div>

<div class="line number6 index5 alt1">`class` `RequestEvent` `extends` `Event`</div>

<div class="line number7 index6 alt2">`{`</div>

<div class="line number8 index7 alt1">`protected` `$request``;`</div>

<div class="line number10 index9 alt1">`public` `function` `setRequest(Request` `$request``)`</div>

<div class="line number11 index10 alt2">`{`</div>

<div class="line number12 index11 alt1">`$this``->request =` `$request``;`</div>

<div class="line number13 index12 alt2">`}`</div>

<div class="line number15 index14 alt2">`public` `function` `getRequest()`</div>

<div class="line number16 index15 alt1">`{`</div>

<div class="line number17 index16 alt2">`return` `$this``->request;`</div>

<div class="line number18 index17 alt1">`}`</div>

<div class="line number19 index18 alt2">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

We can now update the code in the `handle` method to fire a `RequestEvent` event to the dispatcher every time a request is received.

<div>

<div id="highlighter_7392" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`public` `function` `handle(Request` `$request``,` `$type` `= HttpKernelInterface::MASTER_REQUEST,` `$catch` `= true)`</div>

<div class="line number2 index1 alt1">`{`</div>

<div class="line number3 index2 alt2">`$event` `=` `new` `RequestEvent();`</div>

<div class="line number4 index3 alt1">`$event``->setRequest(``$request``);`</div>

<div class="line number6 index5 alt1">`$this``->dispatcher->dispatch(``'request'``,` `$event``);`</div>

<div class="line number7 index6 alt2">`// ...`</div>

<div class="line number8 index7 alt1">`}`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

This way, all called listeners will be able to access the `RequestEvent` object and also the current `Request`. For the moment, we wrote no such listener, but we could easily imagine one that would check if the requested URL has restricted access, before anything else happens.

<div>

<div id="highlighter_193893" class="syntaxhighlighter  php">

<table border="0" cellpadding="0" cellspacing="0">

<tbody>

<tr>

<td class="gutter">

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

</td>

<td class="code">

<div class="container">

<div class="line number1 index0 alt2">`$app``->on(``'request'``,` `function` `(RequestEvent` `$event``) {`</div>

<div class="line number2 index1 alt1">`// let's assume a proper check here`</div>

<div class="line number3 index2 alt2">`if` `(``'admin'` `==` `$event``->getRequest()->getPathInfo()) {`</div>

<div class="line number4 index3 alt1">`echo` `'Access Denied!'``;`</div>

<div class="line number5 index4 alt2">`exit``;`</div>

<div class="line number6 index5 alt1">`}`</div>

<div class="line number7 index6 alt2">`});`</div>

</div>

</td>

</tr>

</tbody>

</table>

</div>

</div>

This is a very basic security system, but you could imagine implementing anything you want, because we now have the ability to hook into the framework at any moment, which makes it much more scalable.

## Conclusion

You’ve seen, by reading this tutorial, that Symfony components are great standalone libraries. Moreover, they can interact together to build a framework that fits your needs. There are many more of them which are really interesting, like the [DependencyInjection](http://symfony.com/doc/current/components/dependency_injection/introduction.html) component or the [Security](http://symfony.com/doc/current/components/security/introduction.html) component.

Of course, full-stack frameworks such as Symfony itself or Laravel have pushed these components to their limits, to create the powerful tools we know today.

</section>