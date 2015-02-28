---
layout: post
title:  "Using Composer in a Legacy Project"
date:   2015-02-28 18:00:32
---

I recently upgraded a legacy PHP project to use Composer. This legacy project already had an autoloader, but wasn’t conforming to any standards. I needed to be able to use some of the amazing PHP packages out there without doing awkward manual downloads and installations. Composer is the answer, but how to ensure a seamless transition?

## The Scene
The legacy project already had an autoloader that loads a few classes. It looked a little bit like this:

{% highlight php %}
spl_autoload_register(function ($class) {
    include $class . '.php';
});
{% endhighlight %}

These were all contained in a `Libs` directory. All of the views were contained in a `htdocs` directory. It looked a little something like this:

{% highlight bash %}
/Libs/
  /Autoloader.php
  /Other Classes…
/htdocs/
{% endhighlight %}

This set up works nicely. But what we really want is to be able to browse [Packagist](https://packagist.org) and easily install some of the open source tools available.

## Installing Composer
[Composer](https://getcomposer.org) is pretty much the standard package manager for PHP.

Composer is super simple to install. [It's all explained here on Composer's site so I don't need to repeat it](https://getcomposer.org/doc/00-intro.md).

This is how my folder structure was looking after the Composer install:

{% highlight bash %}
/Libs/
  /Autoloader.php
  /Other Classes…
/htdocs/
/vendors/
/composer.json
{% endhighlight %}

## Autoloading Legacy Libs
Composer is usually used to autoload packages and classes conforming to PSR-0 or PSR-4 standards. None of the code within the legacy `Libs` directory complies with either standard. They don't even use Namespaces. So how can we use Composer's autoloading abilities on them? We use [Composer's classmap](https://getcomposer.org/doc/04-schema.md#classmap) option.

So in our case it looks like this inside the `composer.json`:

{% highlight json %}
{
    "autoload": {
        "classmap": ["Libs"]
    }
}
{% endhighlight %}

We only have one folder to autoload, but you can include multiple folders in the classmap array.

## Using Composer's autoload in our views
The last step is to make sure we can cleanly include the Composer autoload in our views. The goal is to include something like this in each view:

{% highlight php %}
include 'autoload.php'
{% endhighlight %}

This is nice and simple, but our views are likely it be in lots of different directories. How can we use that same file reference from any view?

We update the `include_path` in our apache server config.

In our case, the `include_path` already points directly to our `Libs` directory. This allowed us to include our previous autoloader from any view. The previous autoloader is also not being completely replaced in one go. We are going to have to run them both side by side while we work on updating every view to use the new Composer autoload. In this case we need to include two directories in our `include_path`. One for the original `Autoloader.php` located in the `Libs` directory and one for the Composer `autoload.php` located in the `vendor` directory.

What we end up with is an `include_path` that looks like this:

{% highlight php %}
php_value include_path "vhosts/site_name/Libs:vhosts/site_name/vendor"
{% endhighlight %}

Give apache a quick restart and you are good to go!

Now just add the following to the top of your views:

{% highlight php %}
include 'autoload.php'
{% endhighlight %}

Next find a PHP package you like. I'm quite partial to [Carbon for date handling](https://packagist.org/packages/nesbot/carbon).

And now you can run

{% highlight bash %}
composer require nesbot/carbon
{% endhighlight %}

from the command line to install the package!

Inside your view you can just do something like:

{% highlight php %}
use Carbon\Carbon;
.
.
.
$now = Carbon::now();
{% endhighlight %}

Any questions, feel free to get in touch [@ACyphus](https://twitter.com/acyphus).
