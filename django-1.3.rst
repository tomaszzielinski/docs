========================
 Django 1.3, 1.4 tips&tricks
========================

settings.py
===========

* Either have a global, versioned *settings.py* file which imports a local (non-versioned) configuration::

    import settings_local

  which has a versioned template *settings_local.py.template*, or use the reverse approach - have a common settings file, e.g. *common_settings.py*
  and then a non-versioned settings.py which imports the common stuff. The latter seems to be the preferred way.

* Figure out project root using either::

      PROJECT_ROOT = os.path.dirname(os.path.realpath(__file__))

  or::

      PROJECT_ROOT = os.path.realpath(os.path.dirname(__file__))

  both forms seems to be actively used and they are pretty much equivalent.

* To get full file paths, use::

      os.path.join(PROJECT_ROOT, 'dir1', 'myfile.txt')

* .. index:: DEBUG, TEMPLATE_DEBUG

  You probably always want to have the detailed information about errors in templates. This is
  independent of the DEBUG setting::

      TEMPLATE_DEBUG = True

* .. index:: SESSION_COOKIE_PATH, SESSION_COOKIE_HTTPONLY

  You may want to use `HttpOnly <http://www.codinghorror.com/blog/2008/08/protecting-your-cookies-httponly.html>`_
  `cookies <http://stackoverflow.com/questions/3529695/how-do-i-set-httponly-cookie-in-django>`_::

      SESSION_COOKIE_PATH = '/; HttpOnly'
      SESSION_COOKIE_HTTPONLY = True

  .. versionchanged:: Django 1.4

      `SESSION_COOKIE_HTTPONLY` is True by default in Django 1.4+

* .. index:: USE_I18N, USE_L10N, LANGUAGES, gettext

  For multilingual sites use::

      USE_I18N = True

  You might also want::

      USE_L10N = True

  Language definitions::

      gettext = lambda s: s
      LANGUAGES = (
          ('sv', gettext('Swedish')),
          ('en', gettext('English')),
      )

* .. index:: TEMPLATE_DIRS

  If you use a global (per-project) template folder you need::

    TEMPLATE_DIRS = (os.path.join(PROJECT_ROOT, 'templates'),)


(De facto) standard add-ons
===========================

* `South migrations <http://south.aeracode.org/>`_ - you might want to use the following settings::

      SKIP_SOUTH_TESTS  = True,
      SOUTH_TESTS_MIGRATE  = False

  (`SKIP_SOUTH_TESTS <http://south.aeracode.org/docs/settings.html#skip-south-tests>`_,
  `SOUTH_TESTS_MIGRATE <http://south.aeracode.org/docs/settings.html#south-tests-migrate>`_)

* `Django Debug Toolbar <http://pypi.python.org/pypi/django-debug-toolbar/0.8.4>`_
  - make sure to configure it according to the `docs <http://pypi.python.org/pypi/django-debug-toolbar/0.8.4#installation>`_

* `Django Sentry <https://github.com/dcramer/django-sentry>`_ - the preferred way to catch exceptions and log messages.
  It has been split into Sentry and `Raven <https://github.com/dcramer/raven>`_ so now both are needed.
  Note that because Sentry/Raven are meant to replace Django's default mechanism and also to integrate deeply into the framework,
  some attention is needed during `configuration <http://raven.readthedocs.org/en/latest/config/django.html>`_.
  Also note that there were (still are?) unsolved problems like `this one <https://github.com/dcramer/django-sentry/issues/210>`_.
  But still, Sentry/Raven is probably one of the best such tools out there.


MySQL 5.x
=========

* Create the database using the following command::

      CREATE DATABASE CHARACTER SET UTF8;

* To convert an existing table with different encoding, use::

      ALTER TABLE tab CONVERT TO CHARACTER SET utf8 COLLATE utf8_unicode_ci;

  Note that ``CONVERT TO`` is critical to do the actual encoding conversion.

* Make sure your tables use the InnoDB engine. You can make sure that it is so by adding this line
  to your database configuration::

    'OPTIONS': {'init_command': 'SET storage_engine=INNODB',}

  `More <https://docs.djangoproject.com/en/1.3/ref/databases/#creating-your-tables>`_.
  Note that MySQL 5.5 (and probably 5.1) have already set InnoDB as the default engine).

* You can make the InnoDB engine the default one in my.cnf file (if you're on MySQL <= 5.0),
  and you don't even have to modify the global my.cnf but use a
  `custom config file <https://docs.djangoproject.com/en/1.3/ref/databases/#connecting-to-the-database>`_ for your Django project.

* `In-memory database for tests <http://tomislavsantek.iz.hr/2011/03/moving-mysql-databases-to-ramdisk-in-ubuntu-linux/>`_,
  and also `this <http://lists.mysql.com/mysql/147938>`_.
  Rewritten in a cleaner way::

      stop mysql
      mount -t tmpfs -o size=400M tmpfs /tmp/ramdisk/
      cp /var/lib/mysql /tmp/ramdisk/
      mount --bind /tmp/ramdisk/ /var/lib/mysql
      start mysql

* Speed tuning:

    * http://www.mysqlperformanceblog.com/2010/02/28/maximal-write-througput-in-mysql/
    * http://www.stereoplex.com/blog/speeding-up-django-unit-test-runs-with-mysql
    * http://www.stereoplex.com/blog/speeding-up-django-unit-test-runs-with-mysql
    * http://www.mysqlperformanceblog.com/2007/11/01/innodb-performance-optimization-basics/
    * http://www.mysqlperformanceblog.com/2007/11/03/choosing-innodb_buffer_pool_size/
    * http://www.mysqlperformanceblog.com/2006/09/29/what-to-tune-in-mysql-server-after-installation/
    * http://www.mysqlperformanceblog.com/2007/11/01/innodb-performance-optimization-basics/#comment-364739
    * Disable logging, slow-logging, binary log etc.

* Watch out for problems:

    * http://stackoverflow.com/questions/2235318/how-do-i-deal-with-this-race-condition-in-django/2235624#2235624
    * http://stackoverflow.com/questions/2221247/why-doesnt-this-loop-display-an-updated-object-count-every-five-seconds/2221400#2221400
    * http://www.no-ack.org/2010/07/mysql-transactions-and-django.html
    * http://www.no-ack.org/2011/05/broken-transaction-management-in-mysql.html
    * `QuerySet.get_or_create() <https://github.com/django/django/blob/2591fb8d4c0246f68b79554976c012039df75359/django/db/models/query.py#L427>`_
      is clumsy anyway


Forms
=====

Smart handling of forms in views (Credits go to `PyDanny&Co <http://speakerdeck.com/u/pydanny/p/advanced-django-forms-usage>`_).
Instead of this::

    def my_view(request):
        if request.method == 'POST':
            form = MyForm(request.POST)
            if form.is_valid():
                form.hooray()
                return HttpResponseRedirect('/success/')
        else:
            form = MyForm()
        return render_to_response('my_template.html', {'form': form})

do this::

    def my_view(request):
        form = MyForm(request.POST or None)
        if form.is_valid():
            form.hooray()
            return HttpResponseRedirect('/success/')
        return render_to_response('my_template.html', {'form': form})

The catch here is that ``form.is_valid()`` returns ``False`` for unbound forms.


Rarely-known (and/or undocumented) Django features
===================================================

* When converting *models.py* into a Python package, make sure that models there have ``app_label`` set in their Meta::

      class Meta:
          app_label = 'app-name'

  Without this trick Django won't see the models.
* `form.Form.has_changed() <https://github.com/django/django/blob/1.3.2/django/forms/forms.py#L316>`_
  - checks if form data is different than the initial data
* `django.utils.html.linebreaks(...) <https://github.com/django/django/blob/1.3.2/django/utils/html.py#L71>`_
  - converts newlines into ``\<p\>`` and ``\<br\>`` tags
* `django.utils.html.urlize(...) <https://github.com/django/django/blob/1.3.2/django/utils/html.py#L102>`_
  - safely converts URLs into clickable links. This is a hard task otherwise:

   #. http://stackoverflow.com/questions/37684/how-to-replace-plain-urls-with-links
   #. http://www.codinghorror.com/blog/2008/10/the-problem-with-urls.html
   #. http://www.ietf.org/rfc/rfc1738.txt
   #. http://www.codinghorror.com/blog/2008/08/protecting-your-cookies-httponly.html

* `model.Meta.order_with_respect_to <https://docs.djangoproject.com/en/1.3/ref/models/options/#order-with-respect-to>`_
  - adds an additional field to the model, purely for ordering purposes. The code behind this feature:

   #. https://github.com/django/django/blob/1.3.2/django/db/models/base.py#L227
   #. https://github.com/django/django/blob/1.3.2/django/db/models/base.py#L532
   #. https://github.com/django/django/blob/1.3.2/django/db/models/base.py#L603
   #. https://github.com/django/django/blob/1.3.2/django/db/models/base.py#L860
   #. https://github.com/django/django/blob/1.3.2/django/db/models/options.py#L114
   #. https://github.com/django/django/blob/1.3.2/django/db/models/fields/proxy.py

* Check `the difference <https://docs.djangoproject.com/en/1.3/topics/db/queries/#spanning-multi-valued-relationships>`_
  between ``Model.objects.filter(a__x=1, a__y=2)`` and ``Model.objects.filter(a__x=1).filter(a__y=2)``
* `A neat trick with aggregation and filtering <https://docs.djangoproject.com/en/1.3/topics/db/aggregation/#order-of-annotate-and-filter-clauses>`_
  - if ``.filter()`` precedes ``.annotate()`` then the annotation is applied only to the filtered elements.


REST, HTTP and Django
======================


URLs, application structure
---------------------------

* A good practice is to design your URL structure so that it more or less follows
  `the de facto standard convention <http://en.wikipedia.org/wiki/Representational_State_Transfer#RESTful_web_services>`_.
  Note that this is mostly about "ordnung", not about being RESTful. It's very hard, if not impossible,
  to write a RESTful service - and if you violate any of the REST principles, you're not RESTful anymore.
  So just accept that and follow whatever is reasonable.

* Still not convinced that REST is not what it appears to be (i.e. a way of naming URLs)? Check these resources (in random order):
  `S.O. thread #1 <http://stackoverflow.com/questions/973796/what-are-the-best-uses-of-rest-services>`_,
  `Roy Fielding's article <http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven>`_,
  `S.O. thread #2 <http://stackoverflow.com/questions/2001773/understanding-rest-verbs-error-codes-and-authentication>`_,
  `Example of RESTful web service design <http://www.peej.co.uk/articles/restfully-delicious.html>`_.

* Specifically, Django sessions are not RESTful so to speak (check these:
  `[1] <http://www.peej.co.uk/articles/no-sessions.html>`_,
  `[2] <http://tech.groups.yahoo.com/group/rest-discuss/message/3583>`_,
  `[3] <http://davidvancouvering.blogspot.com/2007/09/session-state-is-evil.html>`_).
  But they are great otherwise, so why not use them? Web development is not a purity contest!

* Still, adopting parts of the REST philosophy is a good idea. Some readings:
  `[1] <http://stackoverflow.com/questions/6433480/restful-actions-services-that-dont-correspond-to-an-entity>`_,
  `[2] <http://stackoverflow.com/questions/3408191/is-the-twitter-api-really-restful>`_,
  `[3] <http://stackoverflow.com/questions/969585/rest-url-design-multiple-resources-in-one-http-call>`_,
  `[4] <http://stackoverflow.com/questions/2173721/why-does-including-an-action-verb-in-the-uri-in-a-rest-implementation-violate-th>`_.

* ``Get lost, my website is RESTful!!!!!`` collapses if only it uses HTML forms. For illustration - let's imagine
  that we want to add books to a catalog. To create a new book resource you POST data to ``/books/`` collection.
  If there is any error, you can get one of the HTTP error codes. If the new book resource is created, you get #201 response.

  Now, that's not how it works in Django (or any other web framework)!
  In Django, if there is any form validation error, a normal (i.e. #200) response is returned,
  just with some additional HTML markup for presenting errors to the user.
  And even if the new book resource is created, a #302 redirect is returned.
  Moreover, you POST to the very same URL which you get the form from - and not to the ``/books/`` collection!

  Why do we have here such a big deviation from how it should look like in a RESTful case?

  The answer is simple - the HTML form is kind of a separate application, a user interface to the server-side service - in the old
  days it would just be a standalone program. It's simply a coincidence (or signum temporis) that now it's a part of
  the same web application.

  The moment we abandon the POST-REDIRECT-GET paradigm, and start POSTing forms to the backend using AJAX requests, we have a much cleaner separation
  of the user interface part and the underlying RESTful (or pseudo-RESTful) service.
  Only that the application is hooked to an URL in the same URL space..

  So what to do about that? Just treat forms as non-RESTful parts, separate applications that happen to live in the same house.
  Use a consistent URL naming for them, like ``/books/1/edit``, and don't think about them more.

* Some back up for what I've written above:
  `[1] <http://stackoverflow.com/questions/7259464/how-should-a-resource-edit-path-looks-like-on-a-restful-web-app>`_,
  `[2] <http://stackoverflow.com/questions/1711653/three-step-buyonline-the-restful-way>`_,
  `[3] <http://stackoverflow.com/questions/3432660/how-to-edit-a-resource>`_,
  `[4] <http://stackoverflow.com/questions/1657454/how-to-do-a-restful-request-for-an-edit-form>`_,
  `[5] <http://stackoverflow.com/questions/1269816/html-interface-to-restful-web-service-without-javascript>`_.

* Some more reading about "RESTful" URLs:
  `[1] <http://stackoverflow.com/questions/1827293/restful-urls-for-a-search-service-with-an-arbitrary-number-of-filtering-criteria>`_,
  `[2] <http://stackoverflow.com/questions/7272472/how-to-specify-a-range-of-data-or-multiple-entities-in-a-restful-web-service>`_.

* Which HTTP error codes to use? `Here's the answer <http://www.aisee.com/graph_of_the_month/http.png>`_.
  Ok ok, I know :-)

* But seriously, there are some rules that are worth following.

* ``HttpResponseBadRequest [400]`` seems to be a good choice when Django view is reached but request parameters are
  invalid.
  Here are some `good <http://stackoverflow.com/questions/5077871/what-is-the-proper-http-response-code-for-request-without-mandatory-fields>`_
  `discussions <http://stackoverflow.com/questions/4781187/http-400-bad-request-for-logical-error-not-malformed-request-syntax>`_
  `on <http://stackoverflow.com/questions/1364527/http-status-code-for-bad-data>`_ that.

* ``HttpResponseForbidden [403]`` seems like a good choice to indicate that authentication is needed
  in a situation when redirection to the login page doesn't make sense - e.g. for AJAX requests.
  Note that there is also 401 code, but it is meant to be used for the purposes of
  `HTTP authentication <http://en.wikipedia.org/wiki/Basic_access_authentication>`_,
  and not a custom one.
  (`A nice discussion <http://stackoverflow.com/questions/6113014/what-http-code-to-use-in-not-authenticated-and-not-authorized-cases>`_)


Django and HTTP caching for static assets
-----------------------------------------

* `Introduction to HTTP caching <http://www.mnot.net/cache_docs/>`_

* Use an asset manager. There is one shipped with Django 1.3+
  (`django.contrib.staticfiles <https://docs.djangoproject.com/en/1.3/howto/static-files/>`_) but it's not too powerful

  * Pick your favourite one from `django-pluggables <http://djangopackages.com/grids/g/asset-managers/>`_
  * A pretty great one is (was?) `django-mediagenerator <http://www.allbuttonspressed.com/projects/django-mediagenerator>`_
    (Hopefully someone will `maintain it <http://www.allbuttonspressed.com/goodbye#comment-372779409>`_)
  * Your picked assed manager should be able to:

        * Combine & minimize CSS and JS scripts, preferably using `YUI Compressor <http://developer.yahoo.com/yui/compressor/>`_ and/or
          `Google Closure Compiler <http://code.google.com/intl/pl-PL/closure/compiler/>`_
        * Version the assets, i.e. give them unique names like ``sitescripts.1fhdysjnry46.js`` - this is required to
          efficiently cache them
        * Now, you want your web server to serve the assets with one of these headers::

              Expires: (now + 1 year)
              Cache-Control: public, max-age=31536000

          plus this one::

              Last-Modified: {{ date }}

        * Thanks to the above headers, the browser caches the assets for up to one year - and in case it wants to check if an asset has changed,
          it sends a conditional request (using ``If-Modified-Since`` header) that makes it possible for the web
          server to reply with ``304 Not Modified`` status code.
        * `Perfect caching headers <http://www.allbuttonspressed.com/projects/django-mediagenerator#q-what-are-the-perfect-caching-headers>`_
        * `Even more, from Yahoo <http://developer.yahoo.com/performance/rules.html#expires>`_
        * In Apache one need to add something like this to the virtual host definition (after making sure that the
          relevant modules are loaded)::

              <Directory /my/project/dir/_generated_media>
                  ExpiresActive On
                  ExpiresDefault "access plus 1 year"
                  Header merge Cache-Control "public"
                  Header unset Etag
                  FileETag None
              </Directory>

        * That's basically all - for static assets there is no need to worry about things like proxy caches storing sensitive data etc.
        * Ah, one more thing - you probably want to have ``Keep-Alive`` on for static assets, but it's not that good for your Django application.
          So better think about some nginx. `Useful link <http://serverfault.com/questions/73812/should-i-activate-keepalive-in-apache2>`_

    * Btw do not get frustrated if the caching doesn't work when you refresh the page using F5.
      `That's a known issue <http://stackoverflow.com/questions/3934413/chrome-why-is-it-sending-if-modified-since-requests/3934694#3934694>`_.


HTTP caching for Django views
--------------------------------

* There's probably no single setup suitable for all your views (pages)
* So let me just give you a few links:

    * `Caching in IE9 <http://blogs.msdn.com/b/ie/archive/2010/07/14/caching-improvements-in-internet-explorer-9.aspx>`_
      Take a look at Vary-related issues, HTTPS caching, redirect caching etc..
      It's not trivial to set it all up properly.
    * `Controlling HTTP caching from Django <https://docs.djangoproject.com/en/1.3/topics/cache/#upstream-caches>`_
    * `django.utils.cache module <https://docs.djangoproject.com/en/1.3/ref/utils/#module-django.utils.cache>`_

* Because of all these things to consider, if you don't have enough manpower to handle it properly,
  I think that it's not that unreasonable to just disable HTTP caching using something like this (idea borrowed from Google Docs)::

      response['Cache-Control'] = 'no-cache, no-store, max-age=0, must-revalidate'
      response['Expires'] = 'Fri, 01 Jan 2010 00:00:00 GMT'

* Otherwise you would have to make sure that there's no leak of sensitive data, no old content is presented to users etc.
  (Btw using ``must-revalidate`` causes the back button in the browser to refresh (reload) the page when pressed.)


Useful links
------------

* `HTTP 1.1 - RFC 2616 <http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html>`_
* `Cache-Control summary <http://palisade.plynt.com/issues/2008Jul/cache-control-attributes/>`_


Other HTTP performance tips
----------------------------

* Read `Yahoo guidelines <http://developer.yahoo.com/performance/>`_
* Read `Google guidelines <http://code.google.com/intl/pl-PL/speed/articles/>`_
* Use `YSlow <http://developer.yahoo.com/yslow/>`_, `PageSpeed <http://code.google.com/intl/pl-PL/speed/page-speed/>`_ or
  even "Audits" tool from Chrome inspector to learn what are the bottlenecks of your site
* There are also other online: `Pingdom <http://tools.pingdom.com/fpt/>`_, `Redbot <http://redbot.org/>`_
* One thing that I think is interesting:
  `optimize the order of stylesheets and scripts <http://code.google.com/intl/pl-PL/speed/page-speed/docs/rtt.html#PutStylesBeforeScripts>`_
* Remember, `performance is a feature <http://www.codinghorror.com/blog/2011/06/performance-is-a-feature.html>`_!


Non-HTTP caching and Django
===========================

* Learn to use `the cache framework <https://docs.djangoproject.com/en/1.3/topics/cache/>`_
* Employ `template source caching <https://docs.djangoproject.com/en/dev/ref/templates/api/#loader-types>`_ - look for
  ``django.template.loaders.cached.Loader``
* Consider using `two-phased template rendering <http://www.holovaty.com/writing/django-two-phased-rendering/>`_
* Try `Redis <http://redis.io/, it's more powerful than `Memcached <http://memcached.org/>`_ and not slower.
  Even if you're not impressed by its `command set <http://redis.io/commands>`_ it has one major advantage over
  Memcached...
* ...which is the persistent storage. It's great not only because of being persistent, but also because it allows to decrease the chances
  of learning `dog piling <http://highscalability.com/strategy-break-memcache-dog-pile>`_ aka
  `thundering herd
  <http://books.google.pl/books?id=m-bDb87UWL0C&pg=PA357&lpg=PA357&dq=thundering+herd+memcache&source=bl&ots=VURP6rGOpI&sig=oa-uHNZpj5IATTg_P_eF7852iWY&hl=pl&ei=6lqwTvX-E9T54QT73dicAQ&sa=X&oi=book_result&ct=result&resnum=4&ved=0CD0Q6AEwAw#v=onepage&q=thundering%20herd%20memcache&f=false>`_
  problem in practice.
  If you can dump your cached data and reload it later, then server crashes or restarts don't hurt that much.
* A nice `Redis tutorial <http://simonwillison.net/static/2010/redis-tutorial/>`_
* Btw, the thundering herd problem is related also to the normal usage of the cache -
  check `django-newcache's README <https://github.com/ericflo/django-newcache/blob/master/README.txt#L79>`_.


Avoid Apache :)
===================
* Apache is a mature and stable piece of software...
* ...but it's also a complex one. It's not that hard to leave a security hole or misconfigure it:

      * MPM vs Prefork
      * mod_wsgi embedded vs daemon mode
      * Are you sure /etc/passwd is not exposed? I'm never sure :) Apache "thinks" in terms of files and folders
        so there might be a way (i.e. URL) to access sensitive data.
      * http://stackoverflow.com/questions/6248772/should-django-python-apps-be-stored-in-the-web-server-document-root/6249943#6249943
      * http://stackoverflow.com/questions/5021424/mod-wsgi-daemon-mode-wsgiapplicationgroup-and-python-interpreter-separation

* nginx is simpler and is the preferred server for static assets anyway.
* Btw use ``KeepAlive=0`` for wsgi apps (to not run out of connections) vs ``KeepAlive=1`` for static assets (to
  speed up serving them)
