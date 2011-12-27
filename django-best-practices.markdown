Django 1.3 best practices
=====================

## settings.py

* Either have a global, versioned *settings.py* file which imports a local (non-versioned) configuration:
    ```import settings_local```
    which has a versioned template *settings_local.py.template*, or use the reverse approach - have a common settings file, e.g. *common_settings.py*
    and then a non-versioned settings.py which import the common stuff. This seems to be the preferred way.

* ```PROJECT_ROOT = os.path.dirname(os.path.realpath(__file__))``` or
    ```PROJECT_ROOT = os.path.realpath(os.path.dirname(__file__))``` - both forms seems to be actively used and they are pretty much equivalent.

* To get full file paths, use ```os.path.join(PROJECT_ROOT, 'dir1', 'myfile.txt')```.

* ```TEMPLATE_DEBUG = True``` - you probably always want to have the detailed information about errors in templates. This is independent of DEBUG setting.

* You may want to use
    [HttpOnly](http://www.codinghorror.com/blog/2008/08/protecting-your-cookies-httponly.html)
    [cookies](http://stackoverflow.com/questions/3529695/how-do-i-set-httponly-cookie-in-django):

    ```
    SESSION_COOKIE_PATH = '/; HttpOnly'
    SESSION_COOKIE_HTTPONLY = True (In Django 1.3+?)
    ```

* For multilingual sites:

    ```USE_I18N = True```

    Possibly you want:

    ```USE_L10N = True```

    Language definitions:

    ```
    gettext = lambda s: s
    LANGUAGES = (
        ('sv', gettext('Swedish')),
        ('en', gettext('English')),
    )
    ```

* If you use a global (per-project) template folder:
    ```TEMPLATE_DIRS = (os.path.join(PROJECT_ROOT, 'templates'),)```

## (De facto) standard add-ons

* [South migrations](http://south.aeracode.org/) - you might want to use the following settings:
  [SKIP_SOUTH_TESTS](http://south.aeracode.org/docs/settings.html#skip-south-tests) = True,
  [SOUTH_TESTS_MIGRATE](http://south.aeracode.org/docs/settings.html#south-tests-migrate) = False

* [Django Debug Toolbar](http://pypi.python.org/pypi/django-debug-toolbar/0.8.4) - make sure to configure it according to [docs](http://pypi.python.org/pypi/django-debug-toolbar/0.8.4#installation)

* [Django Sentry](https://github.com/dcramer/django-sentry) - the preferred way to catch exceptions and log messages.
   It has been split into Sentry and [Raven](https://github.com/dcramer/raven) so both are needed.
   Note that because Sentry/Raven are meant to replace the default Django mechanism and also integrate deeply into the framework,
   some attention is needed during [configuration](http://raven.readthedocs.org/en/latest/config/django.html).
   Also note that there were (still are?) unsolved problems like [this one](https://github.com/dcramer/django-sentry/issues/210).
   But still, Sentry/Raven is probably the best such tool out there.

## MySQL 5.x

* Create the database using the following command: ```CREATE DATABASE CHARACTER SET UTF8;```

* To convert an existing table with different encoding, use:
    ```ALTER TABLE tab CONVERT TO CHARACTER SET utf8 COLLATE utf8_unicode_ci;```
    Note that <strong>CONVERT TO</strong> is critical to do the actual encoding conversion.

* Make sure you tables use the InnoDB engine.
    You can make sure that it is so by adding this line to your database configuration:
    ```'OPTIONS': {'init_command': 'SET storage_engine=INNODB',}```
    [[more]](https://docs.djangoproject.com/en/1.3/ref/databases/#creating-your-tables)

* You can make the InnoDB engine the default one in my.cnf file,
  and you don't even have to modify the global my.cnf but use a [custom config file](https://docs.djangoproject.com/en/1.3/ref/databases/#connecting-to-the-database) for your Django project.

## Forms

Smart handling of forms in views (Credits go to [PyDanny&Co](http://speakerdeck.com/u/pydanny/p/advanced-django-forms-usage)).
Instead of this:

```
def my_view(request):
    if request.method == 'POST':
        form = MyForm(request.POST)
        if form.is_valid():
            form.hooray()
            return HttpResponseRedirect('/success/')
    else:
        form = MyForm()
    return render_to_response('my_template.html', {'form': form})
```

Do this:

```
def my_view(request):
    form = MyForm(request.POST or None)
    if form.is_valid():
        form.hooray()
        return HttpResponseRedirect('/success/')
    return render_to_response('my_template.html', {'form': form})
```

The catch here is that *form.is_valid()* returns *False* for unbound forms.

## HTTP and REST

* A good practice is to design your URL structure so that it more or less follows
    [the de facto standard convention](http://en.wikipedia.org/wiki/Representational_State_Transfer#RESTful_web_services).
    Note that this is mostly about "ordnung", not about being RESTful. It's very hard, if not impossible,
    to write a RESTful website - and if you violate any of the REST principles, you're not RESTful anymore.
    So just accept that and follow whatever is reasonable.

* Still not convinced that REST is not what it appears to be (i.e. a way of naming URLs)? Check these resources (in random order):
    [S.O. thread #1](http://stackoverflow.com/questions/973796/what-are-the-best-uses-of-rest-services),
    [Roy Fielding's article](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven),
    [S.O. thread #2](http://stackoverflow.com/questions/2001773/understanding-rest-verbs-error-codes-and-authentication),
    [Example of RESTful web service design](http://www.peej.co.uk/articles/restfully-delicious.html).

* Specifically, Django sessions are not RESTful (check these:
    [1](http://www.peej.co.uk/articles/no-sessions.html),
    [2](http://tech.groups.yahoo.com/group/rest-discuss/message/3583),
    [3](http://davidvancouvering.blogspot.com/2007/09/session-state-is-evil.html)).
    But they are great otherwise, so why not use them? Web development is not a purity contest!

* Still, adopting parts of the REST philosophy is a good idea. Some readings:
    [1](http://stackoverflow.com/questions/6433480/restful-actions-services-that-dont-correspond-to-an-entity),
    [2](http://stackoverflow.com/questions/3408191/is-the-twitter-api-really-restful),
    [3](http://stackoverflow.com/questions/969585/rest-url-design-multiple-resources-in-one-http-call),
    [4](http://stackoverflow.com/questions/2173721/why-does-including-an-action-verb-in-the-uri-in-a-rest-implementation-violate-th)

* *"Get lost, my website is RESTful!!!!!"* collapses if only it uses HTML forms. For illustration - let's imagine adding books to a catalog.
    To create a new book resource you POST data to ```/books/``` collection. If there is any error, you can get one of the HTTP error codes.
    If the new book resource is created, you get #201 response.

    Now, that's not how it works in Django (or any other web framework)! In Django, if there is any form validation error, a normal (i.e. #200) response is returned,
    just with some additional HTML markup for presenting errors to the user. And even if the new book resource is created, a #302 redirect is returned.
    Moreover, you POST to the very same URL which you get the form from - and not to the ```/books/``` collection!

    Why do we have here such a big deviation from how it should look like in a RESTful case?

    The answer is simple - the HTML form is kind of a separate application, a user interface to the server-side service - in the old
    days it would just be a standalone program. It's simply a coincidence (or signum temporis) that it's a part of the same web application.

    The moment we abandon the POST-REDIRECT-GET paradigm, and start POSTing forms to the backend using AJAX requests, we have a much cleaner separation
    of the user interface part and the underlying REST (or pseudo-REST) service. Only that the application is hooked to an URL in the same URL space..

    So what to do about that? Just treat forms as non-RESTful parts, separate applications that happen to live in the same house.
    Use a consistent URL naming for them, like ```/books/1/edit``` and don't think about them more.

* Some backup for what I've written above:
    [1](http://stackoverflow.com/questions/7259464/how-should-a-resource-edit-path-looks-like-on-a-restful-web-app),
    [2](http://stackoverflow.com/questions/1711653/three-step-buyonline-the-restful-way),
    [3](http://stackoverflow.com/questions/3432660/how-to-edit-a-resource),
    [4](http://stackoverflow.com/questions/1657454/how-to-do-a-restful-request-for-an-edit-form),
    [5](http://stackoverflow.com/questions/1269816/html-interface-to-restful-web-service-without-javascript).

* Some more reading about "RESTful" URLs:
    [1](http://stackoverflow.com/questions/1827293/restful-urls-for-a-search-service-with-an-arbitrary-number-of-filtering-criteria),
    [2](http://stackoverflow.com/questions/7272472/how-to-specify-a-range-of-data-or-multiple-entities-in-a-restful-web-service).

* Which HTTP error codes to use? [Here's the answer](http://www.aisee.com/graph_of_the_month/http.png).
   Ok ok, I know :-)

* But seriously, there are some rules that are worth following.

* **HttpResponseBadRequest [400]** seems to be a good choice when Django view is reached but request parameters are invalid.
    Here are some [good](http://stackoverflow.com/questions/5077871/what-is-the-proper-http-response-code-for-request-without-mandatory-fields) [discussions](http://stackoverflow.com/questions/4781187/http-400-bad-request-for-logical-error-not-malformed-request-syntax)
    [on](http://stackoverflow.com/questions/1364527/http-status-code-for-bad-data) this</a>.

* **HttpResponseForbidden [403]** looks like a good choice to indicate that authentication is needed
    in a situation when redirection to the login page doesn't make sense - e.g. for AJAX requests.
    Note that there is also 401 code, but it is meant to be used for the purposes of [HTTP authentication](http://en.wikipedia.org/wiki/Basic_access_authentication),
    and not a custom one. ([A nice discussion](http://stackoverflow.com/questions/6113014/what-http-code-to-use-in-not-authenticated-and-not-authorized-cases) on this)


## Misc

When converting *models.py* into models package, make sure that models there have ```app_label``` set in their Meta:

```
class Meta:
    app_label = 'app-name'
```

Without this trick, Django won't see the models.
