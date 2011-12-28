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

