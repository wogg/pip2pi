``pip2pi`` builds a PyPI-compatible package repository from ``pip`` requirements
================================================================================

PyPI can go down, package maintainers can remove old tarballs, and downloading
tarballs can take a long time. ``pip2pi`` helps to alleviate these problems by
making it blindingly simple to maintain a PyPI-compatible repository of packages
your software depends on.


Status
------

These tools were developed to be used internally, and they appear to work for
me. A quick glance at the code will make it obvious that they are far from
robust (ex, they probably won't work on Windows and they make a few calls to
shell commands that could be implemented in Python)... But they should work,
and they shouldn't eat your data or steal private keys or anything.


Requirements
------------

0. ``pip``
1. A ``requirements.txt`` file for your project (optional, but useful)
2. An HTTP server (optional, but useful)


Setup
-----

Install ``pip2pi``::

    $ pip install pip2pi

And create the directory which will contain the tarballs of required packages,
preferably somewhere under your web server's document root::

    $ mkdir /var/www/packages/


Mirroring Packages
------------------

To mirror a package and all of its requirements, use ``pip2tgz``::

    $ pip2tgz packages/ foo==1.2
    ...
    $ ls packages/
    foo-1.2.tar.gz
    bar-0.8.tar.gz

Note that ``pip2tgz`` passes package arguments directly to ``pip``, so packages
can be specified in any format that ``pip`` recognizes::

    $ cat requirements.txt
    foo==1.2
    http://example.com/baz-0.3.tar.gz
    $ pip2tgz packages/ -r requirements.txt bam-2.3/
    ...
    $ ls packages/
    foo-1.2.tar.gz
    bar-0.8.tar.gz
    baz-0.3.tar.gz
    bam-2.3.tar.gz


Building a Package Index
------------------------

A directory full of ``.tar.gz`` files can be turned into PyPI-compatible
"simple" package index using the ``dir2pi`` command::

    $ ls packages/
    bar-0.8.tar.gz
    baz-0.3.tar.gz
    foo-1.2.tar.gz
    $ dir2pi packages/
    $ find packages/
    packages/
    packages/bar-0.8.tar.gz
    packages/baz-0.3.tar.gz
    packages/foo-1.2.tar.gz
    packages/simple
    packages/simple/bar
    packages/simple/bar/bar-0.8.tar.gz
    packages/simple/baz
    packages/simple/baz/baz-0.3.tar.gz
    packages/simple/foo
    packages/simple/foo/foo-1.2.tar.gz


But that's a lot of work...
---------------------------

If running two commands seems like too much work... Take heart! The ``pip2pi``
command will run both of them for you... **And** it will use ``rsync`` to copy
the new packages and index to a remote host! ::

    $ pip2pi example.com:/var/www/packages/ foo==1.2
    ...
    $ curl -I http://example.com/packages/simple/foo/foo-1.2.tar.gz | head -n1
    HTTP/1.1 200 OK


But that's still too much work...
.................................

Take heart! Your shell's ``alias`` command can help. Add an alias like this to
your shell's runtime configuration file (hint: ``~/.bashrc`` or similar)::

    alias pip2acmeco="pip2pi dev.acmeco.com:/var/www/packages/"

Now updating your package index will be as simple as::

    $ pip2acmeco foo==1.2 -r bar/requirements.txt


Using Your New Package Index
----------------------------

To use the new package index, pass the ``--index-url=`` argument to ``pip``::

    $ pip install --index-url=http://example.com/packages/simple/ foo

Or, once it has been mirrored, prefix you ``requirements.txt`` with
``--index-url=...``::

    $ cat requirements.txt
    --index-url=http://example.com/packages/simple/
    foo==1.2


Without a web server
--------------------

You can use your package index offline, too::

    $ pip install --index-url=file:///var/www/packages/simple foo==1.2


Some Tips
---------

When installing packages from source via ``python setup.py install``
or ``python setup.py install``, you may need to create a
``setup.cfg``, which points to your package index.
Here are some examples for an offline package index
in your Windows, Linux, or Mac file system::
    
    [easy_install]
    # Windows
    # index_url = file:///C:/pip2pi/simple/

    # Linux
    # index_url = file:///home/myusername/.pip2pi/simple/

    # Mac
    index_url = file:///Users/myusername/.pip2pi/simple/
    
Note the triple ``///` after ``file:`` -- two for the protocol,
the third for the root of the local file system.


Keywords
========

* Mirror PyPI
* Offline PyPI
* Create offline PyPI mirror
