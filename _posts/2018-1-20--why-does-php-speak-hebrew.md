---
layout:     post
title:      Why Does PHP Speak Hebrew? 
Date:       2018-1-20 14:44:00
author:     Yury Voloshin
summary:    The mysterious PAAMAYIM NEKUDOTAYIM error message in PHP
categories: PHP
---
This is my first post on a PHP topic. I'll talk about something very unusual that I saw while working on a PHP script. At first it seems like a minor find about the behaviour of PHP, but on digging further it turned out to be a small window into the history and politics of PHP. While checking whether a variable is empty before using it in a function, I did this:

```php
if (isset($a)) {
 // do something
}
```

Then I wanted to see what happens when $a is null:

```php
if (isset(null)) {
 // do something
}
```

This gave me an error message that I've never seen before:
`Parse error:  syntax error, unexpected ')', expecting T_PAAMAYIM_NEKUDOTAYIM`. The location of the error was in the line with `if (empty(null))`.

Same message came up when I used `empty(null)` instead of `isset(null)`.

This error message seemed like gibberish, but having studied Hebrew for a few months back in high school, I could tell that this gibberish looked suspiciously like Hebrew! A quick search showed that this was indeed Hebrew. PAAMAYIM_NEKUDOTAYIM means "double colon" (`::`) in Hebrew. This begs two questions. First, why was PHP looking for a double colon? And second, why was the error message in Hebrew?

The first question seems easier to answer. In the [PHP manual](http://php.net/manual/en/function.empty.php), the `empty()` function is described like this:
`bool empty ( mixed $var )`
 In other words, this function requires a variable as a parameter. Since `null` was not prefaced with a `$`, PHP interpreted it as a class name and continued to look for a variable. Since a variable that's prefaced by a class name has to be an abstract variable in that class, PHP looked for `::`, the static separator. 

The question of why the message is in Hebrew opens a little window into the history of PHP. It turns out that the compilers for PHP 3 and PHP 4 were [developed in Israel by Andi Gutmans and Zeev Suraski](https://en.wikipedia.org/wiki/Zend_Technologies). The PHP 4 compiler was called [Zend Engine](https://en.wikipedia.org/wiki/Zend_Engine) and increasingly more advanced versions of it went on to power all versions of PHP since 4. The Hebrew name of the double colon was inserted as an [easter egg](https://en.wikipedia.org/wiki/Easter_egg_(media)) by the developers as a reminder of PHP's origin. This has been a controversial topic  in the PHP community, as described in detail on [Phil Sturgeon's blog](https://philsturgeon.uk/php/2013/09/09/t-paamayim-nekudotayim-v-sanity/). On one hand, the "conservative" faction wanted to keep the Hebrew name to preserve the history of PHP. On the other hand, the "progressive" faction was sympathetic with the annoyed and puzzled developers googling "paamayim nekudotayim", and keenly aware of the hundreds (or thousands?) of hours of time collectively wasted on such googling. 

As with most issues related to politics, change happened slowly. It turned out that I was using PHP 5.3, which was the last version of PHP where this error stated "expecting T_PAAMAYIM_NEKUDOTAYIM". In PHP 5.4, the error text was "expecting :: (T_PAAMAYIM_NEKUDOTAYIM)". The mention of the double colon was helpful, even though the mysterious paamayim nekudotayim was still there. In PHP 5.5, the error changed to "Cannot use isset() on the result of an expression (you can use 'null !== expression' instead)". This is a uch better error message. Now the developer does not have to puzzle out why PHP is lookingfor a double colon. In PHP 7, the error message changes again to "Cannot use 'null' as class name as it is reserved". 
