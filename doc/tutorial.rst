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
Currently (July 2015) the latest PyPy3 release supports Python 3.2 but the
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

You can define functions very similarity as in Python:

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
``two args`` and ``three args`` for three arguments ``x``, ``y``, and ``z``.
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

The same holds true for a duplicated pattern:

.. literalinclude:: code/duplicate_pattern.mochi
   :language: python

prints::

    match 1
    match 1
    match 1

The pattern ``a, b`` will never be reached because it is the same
as the pattern ``x, y``.

Patterns can also match types:

.. literalinclude:: code/match_types.mochi
   :language: python

prints::

    10 abc int str
    10.5 3 float int
    a b unknown
    pvector([1, 2, 3]) pvector

The builtin types ``int``, ``float``, and ``str`` before ``x`` and ``y``
make the patterns more specific. Only arguments with the specified type
match. Again, ``&z`` matches everything. Besides the builtin simple types,
Mochi has other types such as ``PVector`` that is essentially an immutable
list.


You can also match other data types. For example, the Python module
numbers_ contains the abstract base class ``Number``. You cannot an instance
of such class but can use it for pattern matching for any kind of number:

.. literalinclude:: code/match_number.mochi
   :language: python

prints::

    1 number
    1.5 number
    a unknown


Going Recursive
+++++++++++++++

While recursion is allowed in Python, the number of recursions is limited and
it is generally not encouraged. Mochi has tail call elimination.
It allows a functions to call itself without a limit.
Internally, Mochi translates the recursion into a Python loop.

There is already a built-in function ``sum``. If we want to implement our
own, we can do this using pattern matching and recursion:

.. literalinclude:: code/recursive_sum.mochi
   :language: python

Since it is bad practices to override built-in names, we choose ``sum_``
as the name for our function.
When we run it with ``$ mochi recursive_sum.mochi`` it prints::

    60.5
    0

There are two cases. If we get a list with content, we split it
into the first element and the rest with ``[head, &tail]:``.
The ``tail`` part can be empty. In this case we add the first element
``head``  to the sum of the rest with the recursive call ``sum_(tail)``.
If we get an empty list ``[]`` we return an zero. This is the base case.

This version cannot be called on more than 1000 numbers because it triggers
the recursion limit. The line ``head + sum_(tail)`` prevents the tail
call elimination, which circumvents the recursion limit.
Making some changes that make the recursive function call the only
expression after a matching pattern allows more recursions:

.. literalinclude:: code/recursive_sum_tail.mochi
   :language: python
   :lines: 1-12

The output is::

    60.5
    0
    49995000

We introduce and accumulator ``acc`` that carries th temporary sum from
on function call to the next. This is in contrast to ``head + sum_(tail)``,
which needs to "come back" after the call to ``sum_()`` to add the result
to ``head``.

The first case ``[head, &tail]`` matches a call with just a list
and calls ``sum_`` with ``head`` as first and ``tail`` as second
argument. The next cases matches this call: ``acc, [head, &tail]``
because it expects a call with two arguments and a list with at least
one member. It call itself adding the ``head`` to the accumulator
``acc``:  ``sum_(head + acc, tail)``. This keeps going until the
list is empty and the pattern ``acc, []`` matches returning the accumulator
``acc``. The base case is the same before returning a zero if the
argument is an empty list ``[]: 0``.

.. note:: Since the pattern ``acc, [head, &tail]: sum_(head + acc, tail)``
    matches for nearly all recursive calls and the other patterns match only
    for one call, it makes sense to make this pattern the first one.

.. _IPython Notebook: http://ipython.org/notebook.html

.. _numbers: https://docs.python.org/3.5/library/numbers.html
