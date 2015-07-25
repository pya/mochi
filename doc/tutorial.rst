Tutorial
========

Installation
------------

You need to have Python 3 installed. PyPy3 also works.

Install Mochi with::

    pip install mochi

It is highly recommended to install install the Mochi kernel.
This gives you Mochi support in an `IPython Notebook`_.

Install the Mochi kernel **after**  you installed Mochi with::

    pip install mochi-kernel

The kernel dose not work with work with PyPy3 yet.
Currently (July 2015) the currently PyPy3 release is 3.2 but the
Notebook requires at least Python 3.3.

First Steps
-----------

Interactive Mode (REPL)
+++++++++++++++++++++++

After you installed Mochi, you can type ``mochi`` at the command line to
invoke the interactive prompt a.k.a. REPL::

    $ mochi
    >>> 1 + 1
    2
    >>> def add(a, b):
    ...     a + b  # no return!
    ...
    <function add at 0x1025a0598>
    >>> add(3, 4)
    7
    >>> exit()

Writing Mochi Modules
+++++++++++++++++++++

Very similar to Python, you can write modules. Open a file, named it
``mymod.mochi`` and add this content:

.. literalinclude:: code/mymod.mochi
   :language: python


Now you can run the program from the command line::

    $ mochi mymod.mochi
    5

Using Mochi from an IPython/Jupyter Notebook
++++++++++++++++++++++++++++++++++++++++++++

You can use an `IPython Notebook`_ to work with Mochi.
You need to install the kernel (see above). Start a new Notebook
server with ``ipython notebook`` and select "Mochi" from the drop-down menu
"New". The support is basic so far. Three will be more useful features such
magic function, tab completion, and help via pre/postfix with ``?``.

Compiling Mochi to Python Bytecode
++++++++++++++++++++++++++++++++++

You can compile Mochi modules to Python 3 bytecode.
This allows to use these modules in Python.

Compile your module with::

    mochi -pyc mymod.mochi -o mymod.pyc

Now there should be file ``mymod.pyc``.

Start Python 3 and import the just created module:

$ python

::

    Python 3.4.3 ...
    >>> import mymod
    5
    >>> mymod.add(10, 20)
    30

Converting Mochi to Python Source Code
++++++++++++++++++++++++++++++++++++++

**Experimental feature**

You can also convert Mochi source code into Python source::

$ mochi -py mymod.mochi -o mymod.py

This results in this Python code:

.. literalinclude:: code/mymod.py
   :language: python

This feature is still experimental and needs to be improved.


Working with Functions
----------------------

Python-style Functions
++++++++++++++++++++++

You can define function very similarity as in Python:

.. literalinclude:: code/mymod.mochi
   :language: python

Note there is no return. The last active expression in a function is the
return value.

.. literalinclude:: code/func_py_style.mochi
   :language: python
   :lines: 1-14

prints::

    limit 10
    value 7 7
    value -7 0
    value -17 10

Haskell-style Functions with Pattern Matching
+++++++++++++++++++++++++++++++++++++++++++++

Mochi offers another way to define a function.
This is very similar to the way functions are defined in Haskell.
Pattern matching helps us to something different depending on the
arguments. For example, we can do something different depending on
the number of arguments:

.. literalinclude:: code/func_haskell_style.mochi
   :language: python

prints::

    no args
    one arg: 10
    two args: 10, 20
    three args: 10, 20, 30
    5 args

We can make up names for the arguments.
If the function gets only one argument, which we choose to name ``x`` it
returns ``one arg``. For two arguments named ``x`` and ``y`` it returns
``two args`` and ``three args `` for three arguments ``x``, ``y``, and ``z``.
The ``&`` catches any number of arguments.
There it should always be the last pattern.
For example, moving this ``&`` pattern to the first position:

.. literalinclude:: code/unreachable_patterns.mochi
   :language: python

results in this being printed::

    no args
    1 args
    2 args
    3 args
    5 args

The same holds true for a duplicate pattern:

.. literalinclude:: code/duplicate_pattern.mochi
   :language: python
prints::

    match 1
    match 1
    match 1

The pattern ``a, b`` will never be reached because it is the same
as the pattern ``x, y``.

.. _IPython Notebook: http://ipython.org/notebook.html
