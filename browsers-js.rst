========================
 Browsers, HTML5 & JavaScript
========================

The hashbang hell
=====================

* http://danwebb.net/2011/5/28/it-is-about-the-hashbangs
* http://isolani.co.uk/blog/javascript/BreakingTheWebWithHashBangs
* http://webmasters.stackexchange.com/questions/32472/pros-cons-of-hash-navigation-from-seo-perspective

HTML5
==========

I've spent some time looking for the best explanations of different aspects of HTML5.
Here are my findings.

General
-----------

* http://mathiasbynens.be/notes/html5-levels
* http://html5doctor.com/avoiding-common-html5-mistakes/

Outlining
---------

* `New document outlines <http://html5doctor.com/outlines/>`_ -
  `sectioning flowchart <http://html5doctor.com/downloads/h5d-sectioning-flowchart.png>`_
  (`source <http://html5doctor.com/happy-1st-birthday-us/>`_)
* http://html5doctor.com/the-section-element/
* http://html5doctor.com/the-article-element/
* `Sections and outline <https://developer.mozilla.org/en-US/docs/Sections_and_Outlines_of_an_HTML5_document>`_
* `When to use sections <http://www.impressivewebs.com/html5-section/>`_
* http://stackoverflow.com/questions/8734350/html5-structure-article-section-and-div-usage
* http://stackoverflow.com/questions/6947489/html5-appropriate-use-of-article-tag


Headings
---------

* In general it seems that <header> tag is optional it's only meant to wrap a single <h1> tag.
  <h1> tag sort of implies <header> around it.
* http://html5doctor.com/the-header-element/ - http://html5doctor.com/the-header-element/#comment-5769
* http://stackoverflow.com/questions/7712871/difference-between-heading-inside-section-or-before-it-in-html5
* http://stackoverflow.com/questions/7796367/why-does-the-html5-header-element-require-a-h-tag
* http://stackoverflow.com/questions/4837269/html5-using-header-or-footer-tag-twice
* http://stackoverflow.com/questions/9663559/html5-section-headings
* http://www.w3.org/TR/html5/the-header-element.html#the-header-element
* http://www.w3.org/TR/html5/the-h1-h2-h3-h4-h5-and-h6-elements.html#the-h1-h2-h3-h4-h5-and-h6-elements
* http://www.w3.org/TR/html5/the-hgroup-element.html#the-hgroup-element
* http://www.w3.org/TR/html5/content-models.html#heading-content-0 (note no <header> tag!)
* http://www.w3.org/TR/html5/headings-and-sections.html#headings-and-sections





Browsers' bfcache
==========

* Firefox has so called `bfcache ("Back-Forward Cache") <https://developer.mozilla.org/en-US/docs/Using_Firefox_1.5_caching>`_
  that keeps the state of the whole page, including JavaScript context, and restores it when user presses the Back
  button. This is separate from the in-browse page (HTTP) cache which stores only the initial page data,
  as sent by the server. More on this
  `here <http://stackoverflow.com/questions/1195440/ajax-back-button-and-dom-updates>`_,
  `here <http://code.google.com/p/chromium/issues/detail?id=2879>`_.
* `Example of how bfcache works <http://www.twmagic.com/misc/cache.html>`_.
* `Bfcache in Opera <http://www.opera.com/support/kb/view/827/>`_.
* `Bfcache in WebKit I <http://www.webkit.org/blog/427/webkit-page-cache-i-the-basics/>`_.
* `Bfcache in WebKit II <http://www.webkit.org/blog/516/webkit-page-cache-ii-the-unload-event/>`_.
*

JavaScript
==========
* JS has some `evil parts <http://wtfjs.com/>`_, use `CoffeeScript <http://coffeescript.org/>`_
  (also protects from RSI ;))


Single page apps / mobile apps
====================================

jQuery Mobile
-------------
* https://github.com/jquery/jquery-mobile/issues/1571#issuecomment-1602190
* http://stackoverflow.com/questions/9829224/jquery-mobile-301-redirect-issues/11880230#11880230


KnockoutJS
----------
* http://stackoverflow.com/questions/7488208/am-i-over-using-the-knockout-mapping-plugin-by-always-using-it-to-do-my-viewmode
* http://stackoverflow.com/questions/7499133/mapping-deeply-hierarchical-objects-to-custom-classes-using-knockout-mapping-plu

CORS (Cross-Origin Resource Sharing)
-------------------------------------
* http://enable-cors.org/
* https://developer.mozilla.org/en-US/docs/HTTP_access_control
* http://www.html5rocks.com/en/tutorials/cors/
* https://github.com/elevenbasetwo/django-cors/blob/master/cors/middleware.py

API design
----------
http://toastdriven.com/blog/2012/sep/04/djangocon-2012-slides-api-design-tips/