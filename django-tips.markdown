---
title: Django tips
---

# Rarely-known (and/or undocumented) Django features

* [form.Form.has_changed()](https://code.djangoproject.com/browser/django/tags/releases/1.3/django/forms/forms.py#L316) - checks if the form data is different that the initial data
* [django.utils.html.linebreaks(...)](https://code.djangoproject.com/browser/django/tags/releases/1.3.1/django/utils/html.py#L71) - converts newlines into \<p\> and \<br\> tags
* [django.utils.html.urlize(...)](https://code.djangoproject.com/browser/django/tags/releases/1.3.1/django/utils/html.py#L102) - safely converts URLs into clickable links.
  This is a hard task otherwise:
    [1](http://stackoverflow.com/questions/37684/how-to-replace-plain-urls-with-links),
    [2](http://www.codinghorror.com/blog/2008/10/the-problem-with-urls.html),
    [3](http://www.ietf.org/rfc/rfc1738.txt),
    [4](http://www.codinghorror.com/blog/2008/08/protecting-your-cookies-httponly.html).
* [model.Meta.order_with_respect_to](https://docs.djangoproject.com/en/dev/ref/models/options/#order-with-respect-to)
* [The difference](https://docs.djangoproject.com/en/1.3/topics/db/queries/#spanning-multi-valued-relationships) between ```Model.objects.filter(a__x=1, a__y=2)``` and ```Model.objects.filter(a__x=1).filter(a__y=2)```
