Python 2.x type system, metaclasses and more
============================================


## General information

+ Basic fact: *EVERYTHING IS AN OBJECT*

+ **Object** is an instance of a **class**, which is called its type: ``type(x) is x.__class__`` /always True/

+ Each&every **class** object inherits directly or indirectly from root base **class** ``object``

+ Thus each&every **object** (i.e. class instance) is a direct or indirect instance of ``object`` class: ``isinstance(x, object) is True`` /always/

+ **Classes** are also **objects**, therefore they also are instances of (other) classes (called metaclasses)
  *[My own idea: objects can be mentally split into (*) "plain" objects and (*) class objects (kind of plain objects with additional class stuff attached to them) ]*

+ Because every **object**, including **class** object, has its **class** ``x.__class__``, and that **class** has its own **class** ``x.__class__.__class__``, the chain would be infinite.
  As a solution, there is a **class** named ``type`` which is its own type, i.e. ``type.__class__ is type`` - that ``type`` class works as type of types (sth like "the ultimate type")


## Built-in types

+ For most built-in types the following relationships occur:

```
type(1) is int; int.__bases__ == (object,); type(int) is type; int.__class__ is type
type(1.0) is float; float.__bases__ == (object,); type(float) is type; float.__class__ is type
type(Ellipsis) is ellipsis; ellipsis.__bases__ == (object,); type(ellipsis) is type; # note that `ellipsis` is not recognized as a literal, but the relationships would uphold if it would be
type(lambda:1) is function; function.__bases__ == (object,); type(function) is type; # same as with `ellipsis` - `function` is not a literal
```

As for strings, it's the same after taking into account one minor detail:

```
type('text') is str; str.__bases__ == (basestring,); basestring.__bases__ == (object,); type(str) is type; type(basestring) is type;
str.__class__ is type; basestring.__class__ is type;
type(u'text') is unicode; unicode.__bases__ == (basestring,); type(unicode) is type
```

There is one edge case on the top of class hierarchy: ```type``` inherits from ```object```
(which is the root base class for all other classes; doesn't inherit from anything else), while ```object``` is instance of ```type```:

```
object.__bases__ == ()            # object is a root base class
type.__bases__ == (object,)       # object is a root base class, so type has to inherit from it
object.__class__ is type          # object is an instance of the type of all types, i.e. type
type.__class__ is type            # type is a type of itself
isinstance(type, object) is True  # type class object is an instance of object class
isinstance(object, type) is True  # object class object is an instance of type which is a descendant of object class
```

## Prerequisites for the subsequent sections

["For new-style classes, implicit invocations of special methods are only guaranteed to work correctly if defined on an object's type, not in the object's instance dictionary."](http://docs.python.org/release/2.7/reference/datamodel.html#special-method-lookup-for-new-style-classes)
In other words, ```C()``` resolves to ```C.__class__.__call__(C)``` and not to ```C.__call__()```. The latter ```__call__``` method is injected into the created ```C``` instance.

```
>>> type.__call__(int)
0
>>> type.__call__(int, 1)
1

>>> int.__new__(int)
0
>>> int.__new__(int, 1)
1
```


## Object creation a.k.a. class instantiation

