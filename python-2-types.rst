============================================
Python 2.x type system, metaclasses and more
============================================

General information
===================

+ Basic fact: *EVERYTHING IS AN OBJECT*
+ **Object** is an instance of a **class**, which is called its type: ``type(x) is x.__class__`` /always True/
+ Each&every **class** object inherits directly or indirectly from root base **class** ``object``
+ Thus each&every **object** (i.e. class instance) is a direct or indirect instance of ``object`` class: 
  ``isinstance(x, object) is True`` /always/
+ **Classes** are also **objects**, therefore they also are instances of (other) classes (called metaclasses)
  *[My own idea: objects can be mentally split into (*) "plain" objects and (*) class objects (kind of plain objects with additional class stuff attached to them) ]*
+ Because every **object**, including **class** object, has its **class** ``x.__class__``, and that **class** has its own **class** ``x.__class__.__class__``, the chain would be infinite.
  As a solution, there is a **class** named ``type`` which is its own type, i.e. ``type.__class__ is type`` - that ``type`` class works as type of types (sth like "the ultimate type")


Built-in types
==============

For most built-in types the following relationships occur::

      type(1) is int; int.__bases__ == (object,); type(int) is type; int.__class__ is type
      type(1.0) is float; float.__bases__ == (object,); type(float) is type; float.__class__ is type
      type(Ellipsis) is ellipsis; ellipsis.__bases__ == (object,); type(ellipsis) is type; # note that `ellipsis` is not recognized as a literal, but the relationships would uphold if it would be
      type(lambda:1) is function; function.__bases__ == (object,); type(function) is type; # same as with `ellipsis` - `function` is not a literal

As for strings, it's the same after taking into account one minor detail::

    type('text') is str; str.__bases__ == (basestring,); basestring.__bases__ == (object,); type(str) is type; type(basestring) is type;
    str.__class__ is type; basestring.__class__ is type;
    type(u'text') is unicode; unicode.__bases__ == (basestring,); type(unicode) is type

There is one edge case on the top of class hierarchy: ``type`` inherits from ``object``
(which is the root base class for all other classes; doesn't inherit from anything else), 
while ``object`` is instance of ``type``::

    object.__bases__ == ()            # object is a root base class
    type.__bases__ == (object,)       # object is a root base class, so type has to inherit from it
    object.__class__ is type          # object is an instance of the type of all types, i.e. type
    type.__class__ is type            # type is a type of itself
    isinstance(type, object) is True  # type class object is an instance of object class
    isinstance(object, type) is True  # object class object is an instance of type which is a descendant of object class


Prerequisites for the subsequent sections
=========================================


`"For new-style classes, implicit invocations of special methods are only guaranteed to work correctly if defined on 
an object's type, not in the object's instance dictionary." <http://docs.python.org/release/2.7/reference/datamodel.html#special-method-lookup-for-new-style-classes>`_
In other words, ``C()`` resolves to ``C.__class__.__call__(C)`` and not to ``C.__call__()``. 
The latter ``__call__`` method is injected into the created ``C`` instance.

    >>> type.__call__(int)
    0
    >>> type.__call__(int, 1)
    1
    
    >>> int.__new__(int)
    0
    >>> int.__new__(int, 1)
    1


Object creation a.k.a. class instantiation
==========================================

To create an object of class C one use: ``c = C(...)``.

``C(...)`` is a syntatic sugar for ``C.__class__.__call__(...)``, ??? which is a method call on class object C,
a method which is taken from class of C class (unless called explicitely as ``c.__call__()`` ???, more on this
`here <http://docs.python.org/release/2.7/reference/datamodel.html#special-method-lookup-for-new-style-classes>`_)
i.e. ``C.__class__``, i.e. metaclass, i.e. often the built-in ``type`` class (uff!)::

    def __call__(self, *kargs, **kwargs):
        obj = self.__new__(self, *kargs, **kwargs)
        obj.__init__(*kargs, **kwargs)
        return obj

``self.__new__()`` is a static method meant to create an instance of a class passed to it as a first parameter.
It's often taken from ``object`` base class, but can be overriden in given class, to customize the creation of class instances.
More on ``__new__()`` is `here <http://www.python.org/download/releases/2.2.3/descrintro/#__new__>`_
and `here <http://docs.python.org/release/2.7/reference/datamodel.html#object.__new__>`_.

Subsequently, ``self.__init__()`` takes the class instance object and initializes it.


Special case of object creation: class declaration a.k.a. metaclass instantiation
=================================================================================

The following declaration::

    class C(object):
        a = 1

is nothing more than just a syntatic sugar for: ``C = C.__metaclass__('C', (object,), {'a': 1})``
where ``__metaclass__`` is determined according to `this <http://www.python.org/download/releases/2.2.3/descrintro/#metaclasses>`_.
and very often it resolves to the built-in ``type`` class, therefore the above can often be rewritten as: ``C = type('C', (object,), {'a': 1})``.

``type('C', (object,), {'a': 1})`` is a syntatic sugar for ``type.__class__.__call__('C', (object,), {'a': 1})``
(which can be simplified to ``type.__call__('C', (object,), {'a': 1})`` because ``type.__class__ is type`` is always true)
and this is resolved like a standard object creation described in the previous section.


A more complex example of "class + metaclass + instantiation" hell
==================================================================

This::

    class MetaC(type):
        def __new__(cls, *kargs, **kwargs):   # static method, called by type.__call__() to create MetaC instance, i.e. C class object
            print 'MetaC.__new__:', cls, kargs, kwargs
            return type.__new__(cls, *kargs, **kwargs)  # this is *most probably* inherited from `object` class

        def __init__(self, *kargs, **kwargs):   # instance method, called to initialize MetaC instance, i.e. C class object
            print 'MetaC().__init__:', self, kargs, kwargs

    class C(object):   # equivalent to: C = MetaC('C', (object,), {'__metaclass__': MetaC})
            __metaclass__ = MetaC

gives in the interactive shell::

    MetaC.__new__: <class '__main__.MetaC'> ('C', (<type 'object'>,), {'__module__': '__main__', '__metaclass__': <class '__main__.MetaC'>}) {}

    MetaC().__init__: <class '__main__.C'> ('C', (<type 'object'>,), {'__module__': '__main__', '__metaclass__': <class '__main__.MetaC'>}) {}


Another - even more complex - example of "class + metaclass + instantiation" hell
=================================================================================

This::

    class MetaC(type):  # equivalent to: MetaC = MetaC('MetaC', (type,), {'__metaclass__': MetaC})
        __metaclass__ = MetaC  # MetaC is own metaclass!

        def __call__(cls, *kargs, **kwargs):
            print 'MetaC.__call__:', cls, kargs, kwargs
            return type.__call__(cls, *kargs, **kwargs)

        def __new__(cls, *kargs, **kwargs): # this is *most probably* inherited from `object` class
            print 'MetaC.__new__:', cls, kargs, kwargs
            return type.__new__(cls, *kargs, **kwargs)

        def __init__(self, *kargs, **kwargs):
            print 'MetaC().__init__:', self, kargs, kwargs

gives in the interactive shell::

    MetaC.__call__: <class '__main__.MetaC'> ('MetaC', (<type 'type'>,), {'__call__': <function __call__ at 0x9413224>, '__module__': '__main__', '__metaclass__': <class '__main__.MetaC'>, '__new__': <function __new__ at 0x94132cc>, '__init__': <function __init__ at 0x9413304>}) {}

    MetaC.__new__: <class '__main__.MetaC'> ('MetaC', (<type 'type'>,), {'__call__': <function __call__ at 0x9413224>, '__module__': '__main__', '__metaclass__': <class '__main__.MetaC'>, '__new__': <function __new__ at 0x94132cc>, '__init__': <function __init__ at 0x9413304>}) {}

    MetaC().__init__: <class '__main__.MetaC'> ('MetaC', (<type 'type'>,), {'__call__': <function __call__ at 0x9413224>, '__module__': '__main__', '__metaclass__': <class '__main__.MetaC'>, '__new__': <function __new__ at 0x94132cc>, '__init__': <function __init__ at 0x9413304>}) {}


Further reading
===============

* http://python.org/doc/newstyle/
* http://docs.python.org/reference/datamodel.html, especially http://docs.python.org/reference/datamodel.html#customizing-class-creation
* http://stackoverflow.com/questions/395982/metaclass-new-cls-and-super-can-someone-explain-the-mechanism-exa/396109
* http://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python, http://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python/6581949#6581949
* http://stackoverflow.com/questions/3798835/understanding-get-and-set-and-python-descriptors
* http://docs.python.org/reference/datamodel.html#implementing-descriptors
* http://docs.python.org/howto/descriptor.html#invoking-descriptors
* http://docs.python.org/reference/datamodel.html#special-method-lookup-for-new-style-classes
* http://docs.python.org/reference/datamodel.html#more-attribute-access-for-new-style-classes
* https://groups.google.com/forum/#!topic/secrets-of-the-framework-creators/UTCMHguEhKs
* http://users.rcn.com/python/download/Descriptor.htm
* Python descriptors/descriptor protocol: http://users.rcn.com/python/download/Descriptor.htm, http://docs.python.org/howto/descriptor.html, http://martyalchin.com/2007/nov/23/python-descriptors-part-1-of-2/
* Descriptors vs bound/unbound methods: http://stackoverflow.com/questions/1015307/python-bind-an-unbound-method, http://stackoverflow.com/questions/114214/class-method-differences-in-python-bound-unbound-and-static/114289#114289, http://stackoverflow.com/questions/114214/class-method-differences-in-python-bound-unbound-and-static/114289#114289