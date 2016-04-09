>http://www.sitepoint.com/how-to-contribute-to-phps-documentation/
Contributing to PHP: How to Contribute to PHP’s Manual

# Contributing to PHP: Как внести свой вклад в документацию

В этом цикле из двух статей, мы ответим для себя на все вопросы о принятии участия в проекте PHP. Надеюсь, эта статья разъяснит все шаги, необходимые для тех, кто хочет стать более активным участником PHP-сообщества.

Первая часть покроет часть вопросов об участии в подготовке документации к языку, в том числе как запросить доступ к [php.net](http://php.net) и что делать, когда ваш аккаунт получит соответствующие права.

![PHP logo](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2016/04/1459870313PHP-logo.svg.png)

## Why Contribute to PHP?

Why should you consider contributing to PHP?

PHP is an open source project that relies upon the willingness of its community to invest their time into the project. The more people become involved, the more the community at large stands to benefit. Whether it’s improving the documentation around the language or contributing bug fixes or features to the core, the cumulative efforts of every developer quickly add up.

Becoming more involved with PHP will also help to take your knowledge of the language to the next level. Contributing to the documentation will give you a more thorough knowledge of the language, and contributing to the core will keep you up to date with any changes that are happening to it. Becoming a contributor will also enable you to ultimately request a [php.net](http://php.net) account, which will put you in a position to help decide what direction the language is heading in. It is therefore definitely a worthwhile thing to do, if you enjoy working with PHP.

## About PHP’s Documentation

The documentation is maintained in the [DocBook](http://www.docbook.org) XML format. Generally speaking, little knowledge of this format is required to be able to contribute to PHP’s documentation. It’s easy to pick up, so you can simply follow along with the XML syntax used in other files of the documentation.

The folder structure for the documentation looks as follows:  
![Folder structure overview](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2016/04/1459870981img_0.png)

The **doc-base** folder contains some tools for converting the XML-based documentation into other formats. You probably don’t need to concern yourself much with this folder, except when creating custom entities (typically used when adding external links to the docs).

The **en** folder is specific to the English documentation (other translations will follow their respective two letter country code names). This folder is the one you’ll predominantly be working in. The **reference** folder contains directories that each pertain to an extension. Each extension folder follows the convention of either having a **functions** folder (for procedural extensions) or folders named after the extension’s classes (for object-oriented extensions). Each extension folder also contains a few other files, including a **book.xml** file for the extension’s landing page and a **versions.xml** file for holding versioning information for when each function was introduced.

For more information on the documentation structure, see the _Files structure_ section of the [Manual sources structure](http://doc.php.net/tutorial/structure.php) page.

Also, while an on-going effort is being made to transition the documentation to Git, it is currently maintained using SVN. This means that you’ll need to install SVN and have a basic familiarity with it if you want to set the docs up locally (as seen later).

## A First-Time Contributor

If you’re a first-time contributor, then you’ll want to start off by using the [online documentation editor](https://edit.php.net). This provides a user interface to enable anyone to log in (using OAuth) and to submit simple patches to the documentation. It’s generally best to log in with the same account each time, so that your contributions remain under a single name only (making it easier to track them if you decide to apply for a [php.net](http://php.net) account later).

The online editor is almost always the starting point for new documentation contributors. By demonstrating the ability to submit patches and have them subsequently accepted, it shows competence and willingness to contribute.

As a first-time contributor, you’ll want to also familiarize yourself with [the style guidelines](http://doc.php.net/tutorial/style.php) of the documentation before submitting any patches.

### Example

Let’s resolve [bug #71716](https://bugs.php.net/bug.php?id=71716) from [bugs.php.net](https://bugs.php.net/). The report says that one of the demonstrative examples is incorrect, where the MongoDB `Client` class is namespaced incorrectly.

After verifying this (by running the connection script locally), we can fix this using the online editor:

<iframe width="560" height="315" src="https://www.youtube.com/embed/X9dRL7uHkjc?rel=0" allowfullscreen=""></iframe>

For more information on how to use the online editor, see the [wiki editor’s Getting Started page](https://wiki.php.net/doc/editor).

## PHP Docs Local Setup

The online editor, however, isn’t the nicest of ways to contribute to the documentation and it is also very limited in what it can do. It should therefore only be used for minor edits and serve as a first stepping stone to getting involved in the documentation side of the project.

If you would like to do anything outside of the editor’s capabilities (such as documenting a function or extension from scratch), or you are becoming a frequent contributor to the docs, then you’ll want to set up the docs locally and request a [php.net](http://php.net) account.

Setting up the PHP docs locally on your computer is a one-time inconvenience. It can be done with the following steps:

1. **Create a docs directory**. This will be used to hold the docs and docs-related stuff in one place:

```
mkdir phpdocs
cd phpdocs
```

The rest of the steps all assume you are in the **phpdocs** directory, unless stated otherwise.

2. **Clone the docs**. Use SVN to pull down a copy of the repo. In our case, we are looking to get the English manual, and so we specify the **doc-en** module (at the end):

```
svn checkout https://svn.php.net/repository/phpdoc/modules/doc-en
```

3. **Clone PhD**. [PhD](http://doc.php.net/phd/docs/) is a tool that renders the DocBook XML format into different output formats.

```
git clone http://git.php.net/repository/phd.git
```

4. **Clone the [php.net](http://php.net) website**. This will be used to view the documentation locally, so that you can see the changes as they would appear on the website before pushing them.

```
git clone http://git.php.net/repository/web/php.git web-php
rm -fR web-php/manual/en
ln -s rendered-docs/php-web web-php/manual/en
```

We need to remove the dummy docs located in **web-php/manual/en**, and then set up a symbolic link to here from the generated documentation files (generated via PhD) at **rendered-docs/php-web**.

5. **Setup SVN Keywords**. The DocBook files all have a `<!-- $Revision: 338832 $ -->` comment near the top of the file. We need to configure SVN to automatically update this value whenever a file changes by configuring its automatic properties. To do this, simply add the following line to `~/.subversion/config` (the automatic properties part is somewhere near the end):

```
*.xml = svn:eol-style=native;svn:keywords=Id Rev Revision Date LastChangedDate LastChangedRevision Author LastChangedBy HeadURL URL
```

The revision ID tracks file changes, which is used by docs translators to see which files need to be updated.

6. **Add workflow file**. This is optional, but you’ll find it very useful to have the commands to validate, build, and view the docs locally at hand. Paste the following into a file called **ref**:

```bash
# Validate the DocBook XML

php ~/phpdocs/doc-en/doc-base/configure.php

# Build the docs

php ~/phpdocs/phd/render.php --docbook ~/php-docs/doc-en/doc-base/.manual.xml --package PHP --format php --output ~/php-docs/rendered-docs

# Run the php.net website locally

cd ~/phpdocs/web-php
php -S 0.0.0.0:4000
```

The above assumes that your `phpdocs` folder is located in your home directory – if not, then update the paths accordingly.

And now you’re all set up!

## Docs Workflow

With everything set up, it’s time to take a look at the workflow for contributing to the docs locally. For this, let’s resolve [bug #71866](https://bugs.php.net/bug.php?id=71866).

We start by making sure the versioned repos are all up-to-date. This means performing an `svn up` in `doc-en/en` and `doc-en/doc-base`, and a `git pull` in **web-php** and **phd** (typically, you’ll find that only the first repo, **doc-en/en**, needs to be updated).

Next, we open up the `doc-en/en/reference/mbstring/functions/mb-detect-order.xml` file and rectify the function’s return value description as per the bug report information:

```
  <refsect1 role="returnvalues">
   &reftitle.returnvalues;
   <para>
-   &return.success;
+   When setting the encoding detection order, &true; is returned on success or &false; on failure.
+  </para>
+  <para>
+   When getting the encoding detection order, an ordered array of the encodings is returned.
   </para>
  </refsect1>
```

We then ensure that we haven’t broken the docs builds by running the docs validator:

```bash
php ~/phpdocs/doc-en/doc-base/configure.php
```

The docs should validate successfully, and as a result, we should see a picture of a nice ASCII cat.

Now, we can either view the changes locally or commit them. Typically, you will just want to commit the changes (unless you’ve made any drastic changes to confirm that they look fine), but we’ll do it this time for didactic reasons.

We build the documentation by running the PhD tool:

```bash
php ~/phpdocs/phd/render.php --docbook ~/php-docs/doc-en/doc-base/.manual.xml --package PHP --format php --output ~/php-docs/rendered-docs
```

Sometimes this stage fails for me the first time because PHP runs out of memory – just simply re-run the command and it should build fine.

Then change into the [php.net](http://php.net) website’s directory to start the local PHP server:

```bash
cd ~/phpdocs/web-php
php -S 0.0.0.0:4000
```

Visiting the localhost (on port 4000) and browsing to the `mb_detect_order()` function, we can see the changes made:

![Changes successfully applied](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2016/04/1459870969img_1.png)

Now that the changes have been made and we’re happy with them, we can commit them. The following parts assume that you have a [php.net](http://php.net) account. If that’s not the case, then you need not concern yourself with the rest of this section just yet (instead, you may wish to request a [php.net](http://php.net) account (see below).

```bash
cd ~/phpdocs/doc-en/en
svn ci -m "Resolve doc bug #71866" reference/mbstring/functions/mb-detect-order.xml
```

The commit message references the doc bug so that an automated comment on our behalf can be made on the bug report we’re resolving. You also won’t be able to view your changes right away on [php.net](http://php.net), but they should propagate to the servers within a few hours typically. We can then visit [bug #71866](https://bugs.php.net/bug.php?id=71866), and, assuming we’re logged into our [php.net](http://php.net) account, navigate to the “Developer” tab and close the bug report.

That’s the basic workflow of resolving documentation bugs. Quick and easy!

## Requesting a [php.net](http://php.net) account

Having set the docs up locally and seen the general workflow, you may then want to consider requesting a [php.net](http://php.net) account with docs karma. (Karma gives contributors differing privileges for various parts of the PHP project. In order to contribute directly to the documentation, docs karma is required on your [php.net](http://php.net) account.)

There are no concrete prerequisites that must be met prior to requesting a [php.net](http://php.net) docs account. Generally, though, accounts will only be granted to those who are looking to actively contribute to and maintain the documentation. Past contributions and current efforts are therefore looked at as evidence of competence and willingness to contribute when requesting an account.

To request an account, visit the [account request page](http://php.net/git-php.php) and read it carefully. Then, fill out the form, submit it, and then email the PHP docs mailing list (phpdoc@lists.php.net) stating why you would like karma, what your wiki account username is, and reference any past contributions you’ve made to the project. If your request is successful, then you’ll be granted a **@php.net** account.

## Documentation to-dos

Aside from fixing the numerous filed [bug reports on the documentation](https://bugs.php.net/search.php?limit=30&order_by=id&direction=DESC&cmd=display&status=Open&bug_type=Documentation+Problem), there are other areas of the manual that could be worked on too, including:

* [Translating the documentation](http://doc.php.net/tutorial/translating.php)
* Expanding upon the partially documented material (typically by adding examples) – the reflection extension is a prime example of this.
* Documenting the undocumented – this will involve digging into the PHP source code using the [lxr tool](http://lxr.php.net)
* General page tidy-ups – rephrasing parts (such as removing any “you”s), removing PHP 4-related material, merging useful comments into the manual itself, etc.

## General Tips

The following are some tips on contributing, some based on my experience, and others simply reiterating from above because they are important:

* **Follow the style guidelines**. Please make sure you’re following [the style guidelines](http://doc.php.net/tutorial/style.php), and when contributing, try to rectify documentation that doesn’t follow it. I generally perform at least a quick search for “I ” and “you” in the file I’m working on.

* **Check tangential things**. This generally refers to resolving bug reports – don’t just fix the bug, check that related things aren’t affected too. This helps to ensure that bugs are properly resolved. For example, whilst [bug #71638](https://bugs.php.net/bug.php?id=71638) only referenced the `stream_filter_append` function, its counterpart `stream_filter_prepend` also [needed to be rectified](https://github.com/salathe/phpdoc-en/commit/16b846d3377578d2b21b8d1187644045d0886836).

* **Be terse**. This applies to both writing and code examples. Poorly worded documentation and convoluted code examples make things more difficult to understand. Keep things short and simple.

* **Separate example code from its output**. Make sure the output of examples is not put with the examples themselves. This convention helps to keep the example clean and clearly shows its output. For example, when cleaning up the `token_get_all` function page, I had to [move a commented dump of a variable and an explanation out of the example](https://github.com/salathe/phpdoc-en/commit/7187e86632414c6ac7bd4fa41571fe95d45f827e).

* **Check the page order**. Ensure that the order of content on the page is correct. This creates consistency in the manual, which enables developers to reference it easier. Again, looking at [the cleaning up of the `token_get_all` function](https://github.com/salathe/phpdoc-en/commit/7187e86632414c6ac7bd4fa41571fe95d45f827e), we can see that the changelog section was initially at the bottom of the page, when it should have been above the examples section.

* **Remove references to PHP 4**. PHP 4 is long gone, and references to it are not only irrelevant now, but they convolute the documentation. Look out for any mentions of PHP 4 (I usually perform a quick grep for it in the file I’m working on) and remove any mentions of it.

*   **Properly version files**. When creating a new file in the documentation, ensure that its revision id is set to nothing:

```
<!-- $Revision$ -->
```

([Example](https://github.com/salathe/phpdoc-en/commit/9d9ced11c2c064b9d3ac8850572b562059f6237f#diff-5517c8f63eae32e2317189b1155186d2R2).)

Other than when creating new files or translating the documentation, you will not need to worry about the revision ID.

If you’re unsure about something, start by checking out the [docs FAQ](http://doc.php.net/tutorial/faq.php). If that doesn’t help, then simply send an email to the php-docs mailing list for support.

And just to reiterate, please check that the documentation build passes before committing any changes to the manual!

## Conclusion

We’ve seen two workflows in this article when contributing to PHP’s documentation. The first was using the [online editor](https://edit.php.net) to resolve [bug #71716](https://bugs.php.net/bug.php?id=71716), and the second was using a local setup of the docs to resolve [bug #71866](https://bugs.php.net/bug.php?id=71866). I hope this has served as a clear and pragmatic introduction on how to contribute to PHP’s documentation.

In part 2 of this series, I will be showing you how to get involved with PHP’s internals. This will involve looking at how to fix bugs in the core by taking a pragmatic approach again.

# contributing, documenation, Open Source, PHP, SVN