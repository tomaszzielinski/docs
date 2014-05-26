===================
Python 2.x rarities
===================


Slicing, extended slicing, *Ellipsis* - ``a[i:j:step], a[i:j, k:l], a[..., i:j]``
=================================================================================

More:
`[1] <http://docs.python.org/release/2.7/library/functions.html#slice>`_,
`[2] <http://stackoverflow.com/questions/118370/how-do-you-use-the-ellipsis-slicing-syntax-in-python>`_,
`[3] <http://stackoverflow.com/questions/772124/what-does-the-python-ellipsis-object-do>`_.

    >>> class C(object):
    ... def __getitem__(self, sli):
    ... print sli

    >>> c = C()
    >>> c[2, 1:3, 1:4:6, ..., 4:, :6, :, ::-1]
    (2, slice(1, 3, None), slice(1, 4, 6), Ellipsis, slice(4, None, None), slice(None, 6, None), slice(None, None, None), slice(None, None, -1))

NotImplemented
==============

`Special value which can be returned by the "rich comparison" special methods (__eq__(), __lt__(), and friends),
to indicate that the comparison is not implemented with respect to the other type. <http://docs.python.org/release/2.7/library/constants.html#NotImplemented>`_.

`*NotImplemented* and reflected operands <http://stackoverflow
.com/questions/101268/hidden-features-of-python/3693838#3693838>`_.


iter(obj, sentinel)
===================

`The iter(callable, until_value) function repeatedly calls callable and yields its result until until_value is
returned <http://stackoverflow.com/questions/101268/hidden-features-of-python/102202#102202>`_.

Example: ``for line in iter(f.read(), '\n'): ...``


Rot13 source encoding
=====================

http://stackoverflow.com/questions/101268/hidden-features-of-python/1024693#1024693


`Negative \*round()\* <http://stackoverflow.com/questions/101268/hidden-features-of-python/373949#373949>`_
===========================================================================================================

Negative precision affects digits in front of the decimal point::

    >>> str(round(1234.5678, -2))
    '1200.0'
    >>> str(round(1234.5678, 2))
    '1234.57'

Reversing a string or a list (well, a sequence)
===============================================

It's is as simple as making a copy of it with negative increment: ``sequence[::-1]`` - which is equivalent to
``sequence[-1::-1]``  (see: `Extended slices <http://docs.python.org/release/2.3.5/whatsnew/section-slices.html>`_).


