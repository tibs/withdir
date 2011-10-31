=======
withdir
=======

Useful tools for changing into a directory
==========================================

Tony Ibbs, October 2011

https://github.com/tibs/withdir

-----

Three classes
=============
1. Directory
2. NewDirectory
3. TransientDirectory

-----

Directory
=========
Simple: change to a directory for the duration of a ``with`` clause:

.. sourcecode:: python

  >>> import os
  >>> from withdir import Directory
  >>> with Directory('~') as d:
  ...     print d.where
  ... 
  ++ pushd to /Users/tibs
  /Users/tibs

----

It does what it says on the tin
===============================
Just to show it does change directory:

.. sourcecode:: python

  >>> print os.getcwd()
  /Users/tibs/sw/withdir
  >>> with Directory('~'):
  ...     print os.getcwd()
  ... 
  ++ pushd to /Users/tibs
  /Users/tibs

(we don't *have* to use the "``as d``" part of the ``with`` statement).

----

It sets $PWD
============
.. sourcecode:: python

  >>> with Directory('~') as d:
  ...     print os.environ['PWD']
  ... 
  ++ pushd to /Users/tibs
  /Users/tibs

This can be useful when working in soft-linked directories with commands (like
``muddle``) that use ``$PWD`` to decide where the user *thinks* they are.

----

For instance
============
.. sourcecode:: bash

  tibs:withdir$ pushd ~/tmp
  ~/tmp ~/sw/withdir
  tibs:tmp$ mkdir fred
  tibs:tmp$ ln -s fred jim
  tibs:tmp$ cd jim
  tibs:jim$ pwd
  /Users/tibs/tmp/jim
  tibs:jim$ python

.. sourcecode:: python

  Python 2.6.5 (r265:79359, Mar 24 2010, 01:32:55) 
  [GCC 4.0.1 (Apple Inc. build 5493)] on darwin
  Type "help", "copyright", "credits" or "license" for more information.
  >>> import os
  >>> os.getcwd()
  '/Users/tibs/tmp/fred'
  >>> os.environ['PWD']
  '/Users/tibs/tmp/jim'
  >>> 

.. sourcecode:: bash

  tibs:jim$ popd
  ~/sw/withdir

-----

Variations on Directory
=======================
You don't need to show the directory changed to:

.. sourcecode:: python

  >>> with Directory('~', show_pushd=False) as d:
  ...     print d.where
  ... 
  /Users/tibs

or you can show the directory changed to *and* that changed back to:

.. sourcecode:: python

  >>> with Directory('~', show_pushd=True, show_popd=True) as d:
  ...     print d.where
  ... 
  ++ pushd to /Users/tibs
  /Users/tibs
  ++ popd to  /Users/tibs/sw/withdir

and you don't *have* to set PWD:

.. sourcecode:: python

  >>> print os.environ['PWD']
  /Users/tibs/sw/withdir
  >>> with Directory('~', show_pushd=False, set_PWD=False) as d:
  ...     print os.environ['PWD']
  ... 
  /Users/tibs/sw/withdir

----

Exceptions need not change back
===============================
.. sourcecode:: python

  >>> with Directory('~', stay_on_error=True) as d:
  ...     print d.where
  ...     1/0
  ... 
  ++ pushd to /Users/tibs
  /Users/tibs
  Traceback (most recent call last):
    File "<stdin>", line 3, in <module>
  ZeroDivisionError: integer division or modulo by zero
  >>> os.getcwd()
  '/Users/tibs'
  >>> os.environ['PWD']
  '/Users/tibs'


...but then we need to change back for ourselves:

.. sourcecode:: python

  >>> os.chdir('/Users/tibs/sw/withdir')
  >>> os.getcwd()
  '/Users/tibs/sw/withdir'

although:

.. sourcecode:: python

  >>> os.environ['PWD']
  '/Users/tibs'

----

NewDirectory
============
As you might expect:

.. sourcecode:: python

  >>> from withdir import NewDirectory
  >>> with NewDirectory('new'):
  ...     print os.getcwd()
  ... 
  ++ mkdir    /Users/tibs/sw/withdir/new
  ++ pushd to /Users/tibs/sw/withdir/new
  /Users/tibs/sw/withdir/new

but also:

.. sourcecode:: python

  >>> with NewDirectory() as d:
  ...     print os.getcwd()
  ...     print d.where
  ... 
  ++ mkdir    /var/folders/4k/4kt1W+tGE5K+xz1rlYdtS++++TM/-Tmp-/tmpYgSPqa
  ++ pushd to /var/folders/4k/4kt1W+tGE5K+xz1rlYdtS++++TM/-Tmp-/tmpYgSPqa
  /private/var/folders/4k/4kt1W+tGE5K+xz1rlYdtS++++TM/-Tmp-/tmpYgSPqa
  /var/folders/4k/4kt1W+tGE5K+xz1rlYdtS++++TM/-Tmp-/tmpYgSPqa

----

Extra options
=============
Of course, ``NewDirectory`` inherits from ``Directory``, but we also have a
new argument:

.. sourcecode:: python

  >>> with NewDirectory(show_pushd=False, show_dirops=False) as d:
  ...     print d.where
  ... 
  /var/folders/4k/4kt1W+tGE5K+xz1rlYdtS++++TM/-Tmp-/tmp9zCkaR

which stops the ``++ mkdir`` printout.

----

An obvious error condition
==========================
.. sourcecode:: python

  >>> with NewDirectory('new'):
  ...     print os.getcwd()
  ... 
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "withdir.py", line 156, in __init__
      raise GiveUp('Directory %s already exists'%where)
  withdir.GiveUp: Directory /Users/tibs/sw/withdir/new already exists

----

TransientDirectory
==================
Easy enough to guess:

.. sourcecode:: python

  >>> from withdir import TransientDirectory
  >>> with TransientDirectory('temp') as d:
  ...     print os.getcwd()
  ...     a = d.where
  ... 
  ++ mkdir    /Users/tibs/sw/withdir/temp
  ++ pushd to /Users/tibs/sw/withdir/temp
  /Users/tibs/sw/withdir/temp
  ++ rmtree   /Users/tibs/sw/withdir/temp
  >>> print a
  /Users/tibs/sw/withdir/temp
  >>> os.path.exists(a)
  False

----

Extra options
=============
Of course, ``TransientDirectory`` inherits from ``NewDirectory``, but it too
has an extra argument:

.. sourcecode:: python

  >>> with TransientDirectory('temp', keep_on_error=True) as d:
  ...     print os.getcwd()
  ...     a = d.where
  ...     1/0
  ... 
  ++ mkdir    /Users/tibs/sw/withdir/temp
  ++ pushd to /Users/tibs/sw/withdir/temp
  /Users/tibs/sw/withdir/temp
  Traceback (most recent call last):
    File "<stdin>", line 4, in <module>
  ZeroDivisionError: integer division or modulo by zero
  >>> print a
  /Users/tibs/sw/withdir/temp
  >>> print os.getcwd()
  /Users/tibs/sw/withdir
  >>> os.path.exists(a)
  True

----

Nesting
=======
Nesting ``with`` clauses can be fun, as in some of the ``muddle``
test code:

.. sourcecode:: python

  def make_repos_with_subdomain(root_dir):
      """Create git repositories for our subdomain tests.
      """
      repo = os.path.join(root_dir, 'repo')
      with NewDirectory('repo'):
          with NewDirectory('main'):
              with NewDirectory('builds') as d:
                  make_build_desc(d.where, TOPLEVEL_BUILD_DESC.format(repo=repo))
              with NewDirectory('main_co') as d:
                  make_standard_checkout(d.where, 'main1', 'main')
              with NewDirectory('first_co') as d:
                  make_standard_checkout(d.where, 'first', 'first')
              with NewDirectory('second_co') as d:
                  make_standard_checkout(d.where, 'second', 'second')

----

One more thing
==============
It turns out to be a pain typing ``os.path.join(d.where, ...)``
within these ``with`` clauses, so we provide a ``join`` method:

.. sourcecode:: python

  with Directory('build') as d:
      # Things get built in their subdomains, but we're deploying at top level
      check_files([d.join('obj', 'main_pkg', 'x86', 'main1'),
                   d.join('install', 'x86', 'main1'),
                   d.join('domains', 'subdomain1', 'obj', 'main_pkg', 'x86', 'subdomain1'),
                   d.join('domains', 'subdomain1', 'install', 'x86', 'subdomain1'),
                   d.join('deploy', 'everything', 'main1'),
                   d.join('deploy', 'everything', 'sub1', 'subdomain1'),
                  ])

      # The top level build has its own stuff
      with Directory(d.join('.muddle', 'tags')) as t:
          check_files([t.join('checkout', 'builds', 'checked_out'),
                       t.join('checkout', 'main_co', 'checked_out'),
                       t.join('package', 'main_pkg', 'x86-built'),
                       t.join('package', 'main_pkg', 'x86-configured'),
                       t.join('package', 'main_pkg', 'x86-installed'),
                       t.join('package', 'main_pkg', 'x86-postinstalled'),
                       t.join('package', 'main_pkg', 'x86-preconfig'),
                       t.join('deployment', 'everything', 'deployed'),
                      ])



.. vim: set filetype=rst tabstop=8 softtabstop=2 shiftwidth=2 expandtab:
