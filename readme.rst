=======
withdir
=======

The ``withdir`` Python module provides three classes:

* ``Directory``
* ``NewDirectory``
* ``TransientDirectory``

all intended for us in ``with`` clauses.

``Directory`` changes into the named directory, and at the end of the ``with``
clause changes back to the original location. It also sets the ``PWD``
environment variable, which some programs use (in preference to the actual
current directory), mainly to get around problems with soft links.

``NewDirectory`` first creates the requested directory (or, if none is
requested, a temporary directory), and then acts like ``Directory``.

``TransientDirectory`` acts like ``NewDirectory``, but will delete the new
directory at the end of the ``with`` clause. This can be very useful for
testing (and there's an option not to delete the directory if the ``with``
clause is exited because of an exception).

Two examples from some of the muddle_ testing code::

  with TransientDirectory(keep_on_error=True):
      banner('TEST SIMPLE BUILD (GIT)')
      test_git_simple_build()

and::

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



_muddle:: http://code.google.com/p/muddle/

.. vim: set filetype=rst tabstop=8 softtabstop=2 shiftwidth=2 expandtab:
