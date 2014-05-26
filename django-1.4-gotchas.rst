==================
Django 1.4 gotchas
==================

* Password hasing makes unit tests `very slow <http://www.reddit.com/r/django/comments/seq59/are_other_people_experiencing_test_slowdown_in_14/>`_.
  The solution is to switch back to MD5 hashing during when running tests::

      if sys.argv[1] == 'test':
          PASSWORD_HASHERS = ('django.contrib.auth.hashers.MD5PasswordHasher',)

