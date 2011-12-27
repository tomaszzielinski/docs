# Speeding up your Django websites


## [Introduction to HTTP caching](http://www.mnot.net/cache_docs/)
        
## Django and HTTP caching for static assets
        
* Use an asset manager. There is one shipped with Django 1.3+ ([django.contrib.staticfiles](https://docs.djangoproject.com/en/1.3/howto/static-files/)) but it's not too powerful

    * Pick your favourite one from [django-pluggables](http://djangopackages.com/grids/g/asset-managers/)
    * A pretty great one is [django-mediagenerator](http://www.allbuttonspressed.com/projects/django-mediagenerator).
    (Hopefully someone will [maintain it](http://www.allbuttonspressed.com/goodbye#comment-372779409))
    * Your picked assed manager should be able to:
        * Combine & minimize CSS and JS scripts, preferably using [YUI Compressor](http://developer.yahoo.com/yui/compressor/) and/or
        [Google Closure Compiler](http://code.google.com/intl/pl-PL/closure/compiler/)
        * Version the assets, i.e. give them unique names like **sitescripts.1fhdysjnry46.js** - this is required to efficiently cache them
        * Now, you want your web server to serve the assets with one of these headers:
           * Expires: (now + 1 year)
           * Cache-Control: public, max-age=31536000
          plus this one:
           * Last-Modified: (date)
        * Thanks to the above headers, the browser caches the assets for up to one year - and in case it wants to check if an asset has changed,
          it sends a conditional request (using If-Modified-Since header) that makes it possible for the web server to reply with "304 Not Modified" status code.
        * [Perfect caching headers](http://www.allbuttonspressed.com/projects/django-mediagenerator#q-what-are-the-perfect-caching-headers)
        * [Even more, from Yahoo](http://developer.yahoo.com/performance/rules.html#expires)
        * In Apache one need to add something like this to the virtual host definition (after making sure that the relevant modules are loaded):
            ```
            <Directory /my/project/dir/_generated_media>
                ExpiresActive On
                ExpiresDefault "access plus 1 year"
                Header merge Cache-Control "public"
                Header unset Etag
                FileETag None
            </Directory>
            ```
        * That's basically all - for static assets there is no need to worry about things like proxy caches storing sensitive data etc.
        * Ah, one more thing - you probably want to have Keep-Alive on for static assets, but it's not that good for your Django application.
          So better think about some nginx. [[Useful link]](http://serverfault.com/questions/73812/should-i-activate-keepalive-in-apache2)
    * Btw do not get frustrated if the caching doesn't work when you refresh the page using F5.
        [That's a known issue](http://stackoverflow.com/questions/3934413/chrome-why-is-it-sending-if-modified-since-requests/3934694#3934694).

## HTTP caching for Django views
    * There's probably no single setup suitable for all your views (pages)
    * So let me just give you a few links:
        * [Caching in IE9](http://blogs.msdn.com/b/ie/archive/2010/07/14/caching-improvements-in-internet-explorer-9.aspx)
          Take a look at Vary-related issues, HTTPS caching, redirect caching etc..
          It's not trivial to set it all up properly.
        * [Controlling HTTP caching from Django](https://docs.djangoproject.com/en/1.3/topics/cache/#upstream-caches)
        * [django.utils.cache module](https://docs.djangoproject.com/en/1.3/ref/utils/#module-django.utils.cache)
    * Because of all these things to consider, if you don't have enough manpower to handle it properly,
      I think that it's not that unreasonable to just disable HTTP caching using something like this (idea borrowed from Google Docs):
      ```
      response['Cache-Control'] = 'no-cache, no-store, max-age=0, must-revalidate'
      response['Expires'] = 'Fri, 01 Jan 2010 00:00:00 GMT'
      ```
      Otherwise you would have to make sure that there's no leak of sensitive data, no old content is presented to users etc.
      (Btw using **must-revalidate** causes the back button in the browser to refresh (reload) the page when pressed.)

## Useful links
* [HTTP 1.1 - RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)
* [Cache-Control summary](http://palisade.plynt.com/issues/2008Jul/cache-control-attributes/)

## Other HTTP performance tips

* Read [Yahoo guidelines](http://developer.yahoo.com/performance/)
* Read [Google guidelines](http://code.google.com/intl/pl-PL/speed/articles/)
* Use [YSlow](http://developer.yahoo.com/yslow/), [PageSpeed](http://code.google.com/intl/pl-PL/speed/page-speed/) or even "Audits" tool from Chrome inspector to learn what are the bottlenecks of your site
* There are also other online: [Pingdom](http://tools.pingdom.com/fpt/), [Redbot](http://redbot.org/)
* One thing that I think is interesting: [optimize the order of stylesheets and scripts](http://code.google.com/intl/pl-PL/speed/page-speed/docs/rtt.html#PutStylesBeforeScripts)

## Remember, [performance is a feature](http://www.codinghorror.com/blog/2011/06/performance-is-a-feature.html)!

## Non-HTTP caching and Django

* Learn to use [the cache framework](https://docs.djangoproject.com/en/1.3/topics/cache/)
* Employ [template source caching](https://docs.djangoproject.com/en/dev/ref/templates/api/#loader-types) - look for **django.template.loaders.cached.Loader**
* Consider using [two-phased template rendering](http://www.holovaty.com/writing/django-two-phased-rendering/)
* Try [Redis](http://redis.io/), it's more powerful than [Memcached](http://memcached.org/) and not slower.
  Even if you're not impressed by its [command set](http://redis.io/commands) it has one major advantage over Memcached...
* ...which is the persistent storage. It's not only great because of being persistent, but also because it allows to decrease the chances
  to learn [dog piling](http://highscalability.com/strategy-break-memcache-dog-pile) aka [thundering herd](http://books.google.pl/books?id=m-bDb87UWL0C&pg=PA357&lpg=PA357&dq=thundering+herd+memcache&source=bl&ots=VURP6rGOpI&sig=oa-uHNZpj5IATTg_P_eF7852iWY&hl=pl&ei=6lqwTvX-E9T54QT73dicAQ&sa=X&oi=book_result&ct=result&resnum=4&ved=0CD0Q6AEwAw#v=onepage&q=thundering%20herd%20memcache&f=false) problem in practice.
  If you can dump you cache data and reload it late then server crashes or restarts don't hurt that much.
* A nice [Redis tutorial](http://simonwillison.net/static/2010/redis-tutorial/)
* Btw, the thundering herd problem is related also to the normal usage of the cache - check [django-newcache README](https://github.com/ericflo/django-newcache/blob/master/README.txt#L79)

