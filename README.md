docpie
===============================================================================

Intro
-------------------------------------------------------------------------------

Isn't it brilliant how [python-docopt](https://github.com/docopt/docopt)
parses the `__doc__` and converts command line into python dict? `docpie` does
the similar work, but...

**`docpie` can do more!**

If you have never used `docpie` or `docopt`, try this. It can parse your
command line according to the `__doc__` string:

```python
# example.py
"""Naval Fate.

Usage:
  naval_fate.py ship new <name>...
  naval_fate.py ship <name> move <x> <y> [--speed=<kn>]
  naval_fate.py ship shoot <x> <y>
  naval_fate.py mine (set|remove) <x> <y> [--moored | --drifting]
  naval_fate.py (-h | --help)
  naval_fate.py --version

Options:
  -h --help     Show this screen.
  --version     Show version.
  --speed=<kn>  Speed in knots. [default: 10]
  --moored      Moored (anchored) mine.
  --drifting    Drifting mine.

"""
from docpie import docpie

argument = docpie(__doc__, version='Naval Fate 2.0')
print(argument)
```

Then try `$ python example.py ship Titanic move 1 2` or
`$ python example.py --help`, see what you get.

**`docpie` can do...**

`docpie` has some useful and handy features, e.g.

1.   it allow you to specify the program name.

    ```python
    '''Usage:
      myscript.py rocks
      $ python myscript.py rocks
      $ sudo python myscript.py rocks
    '''
    print(docpie(__doc__, name='myscript.py'))
    ```

2.  Different from `docpie`, `docpie` will handle `--` automatically by
    default, you do not need to write it in your "Usage:" anymore.
    (You can also trun off this feature)

    ```python
    '''Usage:
     prog <hello>
    '''
    from docpie import docpie
    print(docpie(__doc__))
    ```

    Then `$ python example.py test.py -- --world` will give you
    `{'--': True, '<hello>': '--world'}`

3.  Some issues in `docopt` have been solved in `dopie` (e.g.
    [#71](https://github.com/docopt/docopt/issues/71),
    [#282](https://github.com/docopt/docopt/issues/282),
    [#130](https://github.com/docopt/docopt/issues/130))

    ```python
    '''
    Usage:
     test.py [ --long-option ]
       -o <value>  (-a | -b)

    Options:
     --long-option    Some help.
     -a               Some help.
     -b               Some help.
     -o <value>       Some help.
    '''

    from docpie import docpie
    from docopt import docopt

    print('---- docopt ----')
    try:
       print(docopt(__doc__))
    except BaseException as e:
       print(e)

    print('---- docpie ----')
    try:
       print(docpie(__doc__))
    except BaseException as e:
       print(e)
    ```

    output:

    ```bash
    $ python test.py -o sth -a
    ---- docopt ----
    -o is specified ambiguously 2 times
    ---- docpie ----
    {'--': False,
     '--long-option': False,
     '-a': True,
     '-b': False,
     '-o': 'sth'}
    ```


Installation
-------------------------------------------------------------------------------

```bash
pip install git+git://github.com/TylerTemp/docpie.git
```

`docopt` has been tested with Python:

2.6.6, 2.6.9, 2.7, 2.7.10,

3.2, 3.3.0, 3.3.6, 3.4.0, 3.4.3,

pypy-2.0, pypy-2.6.0, pypy3-2.4.0


Basic Usage
-------------------------------------------------------------------------------

```python
from docpie import docpie
```

```python
docpie(doc, argv=None, help=True, version=None,
       stdopt=True, attachopt=True, attachvalue=True,
       auto2dashes=True, name=None, case_sensitive=False, extra={})
```

Note that it's strongly suggested that you pass keyword arguments instead of
positional arguments.

*   `doc` is the description of your program which `docpie` can parse. It's
    usually the `__doc__` string of your python script, but it can also be any
    string in corrent format. The format is given in next section. Here is an
    example:

    ```python
    """
    Usage: my_program.py [-hso FILE] [--quiet | --verbose] [INPUT ...]

    Options:
     -h --help    show this
     -s --sorted  sorted output
     -o FILE      specify output file [default: ./test.txt]
     --quiet      print less text
     --verbose    print more text
    """
    ```

*   `argv` (sequence) is the command line your program accepted and it should
    be a list or tuple. By default `docpie` will use `sys.argv` if you omit
    this argument when it's called.
*   `help` (bool, default `True`) tells `docpie` to handle `-h` & `--help`
    automatically. When it's set to `True`, `-h` will print "Usage" and "Option"
    section, then exit; `--help` will print the whole `doc`'s value and exit.
    set `help=False` if you want to handle it by yourself. Use `extra` (see
    below) or see `Docpie` if you only want to change `-h`/`--help` behavior.
*   `version` (any type, default `None`) specifies the version of your program.
    When it's not `None`, `docpie` will handle `-v`/`--version`, print this
    value, and exit. See `Docpie` if you want to customize it.
*   `stdopt` (bool, default `True`) when set `True`(default), long flag should
    only starts with `--`, e.g. `--help`, and short flag should be `-` followed
    by a letter. This is suggested to make it `True`. When set to `False`,
    `-flag` is also a long flag. Be careful if you need to turn it off.
*   `attachopt` (bool, default `True`) allow you to write/pass several short
    flag into one, e.g. `-abc` can mean `-a -b -c`. This only works when
    `stdopt=True`

    ```python
    from docpie import docpie
    print(docpie('''Usage: prog -abc''', ['prog', '-a', '-bc']))
    # {'--': False, '-a': True, '-b': True, '-c': True}
    ```

*   `attachvalue` (bool, default `True`) allow you to write short flag and its
    value together, e.g. `-abc` can mean `-a bc`. This only works when
    `stdopt=True`

    ```python
    '''
    Usage:
      prog [options]

    Options:
      -a <value>  -a expects one value
    '''
    from docpie import docpie
    print(docpie(__doc__, ['prog', '-abc']))
    # {'--': False, '-a': 'bc'}
    ```

*   `auto2dashes` (bool, default `True`) When it's set `True`, `docpie` will
    handle `--` (which means "end of command line flag", see
    [here](http://www.cyberciti.biz/faq/what-does-double-dash-mean-in-ssh-command/)
    )

    ```python
    from docpie import docpie
    print(docpie('Usage: prog <file>'), ['prog', '--', '--test'])
    # {'--': True, '<file>': '--test'}
    ```

*   `name` (str, default `None`) is the "name" of your program. In each of your
    "usage" the "name" will be ignored. By default `docpie` will ignore the
    first element of your "usage"
*   `case_sensitive` (bool, default `False`) specifies if it need case
    sensitive when matching "Usage:" and "Options:"
*   `extra` see ["Advanced Usage"](#advanced-usage) - ["Auto Handler"](#auto-handler)

the return value is a dictonary. Note if a flag has alias(e.g, `-h` & `--help`
has the same meaning, you can specify in "Options"), all the alias will also
in the result.

Format
-------------------------------------------------------------------------------

`docpie` is indent sensitive.

### Usage Format

"Usage" starts with `Usage:`(set `case_sensitive` to make it case
sensitive/insensitive), ends with a *visibly* empty line.

```python
"""
Usage: program.py

"""
```

You can write more than one usage patterns

```python
"""
Usage:
  program.py <from> <to>...
  program.py -s <source> <to>...

"""
```

When one usage pattern goes too long you can separate into several lines,
but the following lines need to indent more:

```python
"""
Usage:
    prog [--long-option-1] [--long-option-2]
         [--long-option-3] [--long-option-4]  # Good
    prog [--long-option-1] [--long-option-2]
      [--long-option-3] [--long-option-4]  # Works but not so good
    prog [--long-option-1] [--long-option-2]
    [--long-option-3] [--long-option-4]  # Not work. Need to indent more.

"""
```

Each pattern can consist of the following elements:

*   **&lt;arguments&gt;**, **ARGUMENTS**. Arguments are specified as either
    upper-case words, e.g. `my_program.py CONTENT-PATH` or words
    surrounded by angular brackets: `my_program.py <content-path>`.
*   **--options**. Short option starts with a dash(`-`), followed by a
    character(`a-z`, `A-Z` and `0-9`), e.g. `-f`. Long options starts with two
    dashes (`--`), followed by seveval characters(`a-z`, `A-Z`, `0-9` and `-`),
    e.g. `--flag`. When `stdopt` and `attachopt` are on, you can "stack"
    seveval of short option, e.g. `-oiv` can mean `-o -i -v`.

    The option can have value. e.g. `--input=FILE`, `-i FILE`, `-i<file>`.
    But it's important that you specify its argument in "Options"
*   **commands** are words that do *not* follow the described above. Note that
    `-` and `--` are also commands.


Use the following constructs to specify patterns:

*   **[ ]** (brackets) **optional** elements. Note the elements in brackets
    should either be all omitted or provided. e.g. `program.py [-ab]` will
    only match `-ab`, `-a -b` or ` `(empty argument)
*   **( )** (parens) **required** elements.  All elements that are *not*
    put in **[ ]** are also required, e.g.:
    `my_program.py --path=<path> <file>...` is the same as
    `my_program.py (--path=<path> <file>...)`.
*   **|** (pipe) **mutually exclusive** elements. Use **( )** or **[ ]**
    to group them, e.g `program.py (--left | --right)`.
    Note for `program.py (<a> | <b> | <c>)`, because there is no difference
    between arguments, this will be parsed as `program.py (<a>)`
    and `<b>`, `<c>` will be the alias of `<a>`

    ```python
    from docpie import docpie
    print(docpie('Usage: prog (<a> | <b>)', 'prog py'.split()))
    # {'--': False, '<a>': 'py', '<b>': 'py'}
    ```

*   **...** (ellipsis) **repeatable** elements. To specify that
    arbitrary number of repeating elements could be accepted, use
    ellipsis (`...`), e.g.  `my_program.py FILE ...` means one or
    more `FILE`-s are accepted.  If you want to accept zero or more
    elements, use brackets, e.g.: ``my_program.py [FILE ...]``. Ellipsis
    works as a unary operator on the expression to the left.
*   **[options]** (case sensitive) shortcut for any options.  You can
    use it if you want to specify that the usage pattern could be
    provided with any options defined below in the option-descriptions
    and do not want to enumerate them all in usage-pattern.

    Note that you can wirte `program.py [options]...`, but you can't break
    the format like `program.py [options...]`
    (in this case, `options` is a command)

If your pattern allows to match argument-less option (a flag) several
times:

```
Usage: my_program.py [-vvv | -vv | -v]
```

then number of occurrences of the option will be counted. I.e.
`args['-v']` will be `2` if program was invoked as `my_program
-vv`. Same works for commands.

Note that the `|` acts like `or` in python, which means if one elements
group matched, the following groups will be skipped. usage like
`program.py [-v | -vv | -vvv]` will not match `program.py -vv`, because the
first `-v` matches first part of `-vv`, and then nothing left to match the
rest argv, so it fails.

If your usage patterns allows to match same-named option with argument
or positional argument several times, the matched arguments will be
collected into a list:

```
Usage: program.py <file> <file> --path=<path>...

Options: --path=<path>...     the path you need
```

(Note you **must** specify "Options" here. See "Known Issue")

Then `program.py file1 file2 --path ./here ./there` will give you
`{'<file>': ['file1', 'file2'], '--path': ['./here', './there']}`

Also note that the `...` only has effect to `<path>`. You can also write in
this way:

```
Usage: program.py <file> <file> (--path=<path>)...

Options: --path=<path>     the path you need
```

Then it can match `program.py file1 file2 --path=./here --path=./there`
with the same result.


### Options Format

**Option descriptions** consist of a list of options that you put
below your usage patterns.

It is necessary to list option descriptions in order to specify:

*   synonymous short and long options,
*   if an option has an argument,
*   if option's argument has a default value.


"Options" starts with `Options:` (set `case_sensitive` to make it case
sensitive/insensitive). descriptions can followed it directly or
on the next line. If you have rest content, separate with an empty line.

e.g.

```python
"""
Usage: prog [options]

Options: -h"""
```

or

```python
"""
Usage: prog [options]

Options:
  -h, --help

Not part of Options.
"""
```

The rules in "Option" section are as follows:

*   To specify that option has an argument, put a word describing that
    argument after space (or equals "`=`" sign) as shown below. Follow
    either <angular-brackets> or UPPER-CASE convention for options'
    arguments.  You can use comma if you want to separate options. In
    the example below, both lines are valid, however you are recommended
    to stick to a single style.

    ```
    -o FILE --output=FILE       # without comma, with "=" sign
    -i <file>, --input <file>   # with comma, without "=" sing
    ```

    You can also give several synonymous
    (only suggested in the following situation)

    ```
    -?, -h, --help
    ```

*   the description of the option can be written in two ways:
    1) separate option and description with 2+ empty spaces.
    2) start at the next line but indent 2+ empty spaces more.

    ```
    -?, -h, --help  print help message. use
                    -h/-? for a short help and
                    --help for a long help. # Good. 2+ empty spaces
    -a, --all
        A long long long long long long long
        long long long long long description of
        -a & --all    # Good. New line & indent 2 more spaces
    ```
*   Use `[default: <your-default-value>]` at the end of the description
    if you need to provide a default value for an option. Note `docpie` has
    a very strict format of default: it must start with `[default: `(note
    the empty space after `:`), followed by your default value, then `]`
    and no more, even a following dot is not acceptale.

    ```
    --coefficient=K  The K coefficient [default: 2.95]  # '2.95'
    --output=FILE    Output file [default: ]            # empty string
    --directory=DIR  Some directory [default:  ]        # a space
    --input=FILE     Input file[default: sys.stdout].   # not work because of the dot
    ```
*   If the option is not repeatable, the value inside `[default: ...]`
    will be interpreted as string.  If it *is* repeatable, it will be
    splited into a list on whitespace:

    ```
    Usage: my_program.py [--repeatable=<arg> --repeatable=<arg>]
                         [--another-repeatable=<arg>]...
                         [--not-repeatable=<arg>]

    # will be ['./here', './there']
    --repeatable=<arg>          [default: ./here ./there]

    # will be ['./here']
    --another-repeatable=<arg>  [default: ./here]

    # will be './here ./there', because it is not repeatable
    --not-repeatable=<arg>      [default: ./here ./there]
    ```


Advanced Usage
-------------------------------------------------------------------------------

Normally the `docpie` is all you need, But you can do more tricks with `Docpie`

```python
from docpie import Docpie
```

### Basic Usage

when call

```python
from docpie import docpie
print(docpie(__doc__))
```

it's equal to:

```python
from docpie import Docpie
pie = Docpie(__doc__)
pie.docpie()
print(pie)
```

```python
Docpie.__init__(self, doc=None, **config)
```

`Docpie.__init__` accepts all arguments of `docpie` function except the `argv`.
Another difference is that all arguments in  `Docpie.__init__` except `doc` are
keyword-only arguments.


```python
Docpie.docpie(self, argv=None)
```

`Docpie.docpie` accepts `argv` which is the same `argv` in `docpie`


### Change Config

```python
Docpie.set_config(self, **config)
```

`set_config` allows you to change the argument after you initialized `Docpie`.
`config` is a dict that the key is the same of `**config` in `__init__`

```python
pie = Docpie(__doc__)
pie.set_config({'help': False})  # now Docpie will not handle `-h`/`--help`
pie.docpie()
```

### Auto handler

Docpie has an attribute called `extra`. `extra` is a dict, the key is an option
(str), and the value is a function. the function accepts two arguments, the
first will be the `Docpie` instance, the second is the the same of the key.

it may lookes like:

```python
{'-h': <function docpie.Docpie.short_help_handler>,
 '--help': <function docpie.Docpie.long_help_handler>,
 '-v': <function docpie.Docpie.version_handler>,
 '--version': <function docpie.Docpie.version_handler>,
}
```

When `version` is not `None`, Docpie will do the following things:

1.  set `Docpie.version` to this value
2.  add "-v", "--version" as the key of `Docpie.extra`, both values are
    `Docpie.version_handler`
3.  when call `Docpie.docpie`, `Docpie` check if whether the keys in
    `Docpie.extra` appears in `argv`.
4.  if find the key, to say `-v` for example, `Docpie` will check
    `Docpie.extra` and call `Docpie.extra["-v"](docpie, "-v")`,
    the first argument is the instance
5.  By default, `Docpie.version_handler(docpie, flag)` will print
    `Docpie.version`, and exit the program.

for `help=True`, `Docpie` will set `{'-h': Docpie.short_help_handler,
'--help': Docpie.long_help_handler}`.


When doing so, `Docpie` will not check your `Options` section. Which means even
you set `-h, -?, --help    print help message` in `Options` secion, `Docpie`
will not handle `-?` automatically. (TODO: make this happen next version)

You can change this by passing `extra` argument, e.g.

```python
"""
Example for Docpie!

Usage: example.py [options]

Options:
  -v, --obvious    print more infomation  # note the `-v` is taken
  --version        print version
  -h, -?, --help   print this infomation
  --moo            the Easter Eggs!

Have fun, my friend.
"""
from docpie import Docpie
import sys


def moo_handler(pie, flag):
    print("Alright you got me. I'm an Easter Eggs.\n"
          "You may use this program like this:\n")
    print("Usage:")
    print(pie.usage_text)
    print("")
    print("Options:")
    print("".join(pie.option_text.splitlines(True)[:-1]))
    sys.exit()    # Don't forget to exit

pie = Docpie(__doc__, version='0.0.1')
pie.set_config(
  extra={
    '--moo': moo_handler,  # set moo handler
    '-?': pie.short_help_handler,  # make it same as `-h`
    '-v': None,  # unset `-v`
  }
)

print(pie)
```

now try the following command:

```bash
example.py -v
example.py --version
example.py -h
example.py -?
example.py --help
example.py --moo
```

to customize your `extra`, the following attribute of `Docpie` may help:

*   `Docpie.version` is the version you set. (default `None`)
*   `Docpie.usage_text` is the usage section. ("Usage:" is not contained)
*   `Docpie.option_text` is the options section. ("Options:" is not contained)

### Serialization

`Docpie.need_pickle(self)` give you everything you need to pickle.
`Docpie.restore_pickle(value)` restore everything which already converted
back by pickle

`Docpie.convert_2_dict(self)` can convert `Docpie` instance into a dict so
you can JSONlizing it. Use `Docpie.convert_2_docpie(cls, dic)` to convert
back to `Docpie` instance.

Here is a full example of serialization and unserialization together with
`pickle`

In developing:

```python
"""
This is my cool script!

Usage: script.py [options] (--here|--there)

Options:
  --here
  --there
  -h, --help
  -v, --version

Have fun then.
"""

import json
try:
    import cPickle as pickle
except ImportError:    # py3 maybe
    import pickle
from docpie import Docpie


pie = Docpie(__doc__)

with open('myscript.docpie.pickle', 'wb') as pkf:
    pickle.dump(pie.need_pickle(), pkf)

# omit `encoding` if you're using python2
with open('myscript.docpie.json', 'w', encoding='utf-8') as jsf:
    json.dump(pie.convert_2_dict(), jsf)
```

In release:

```python
"""
This is my cool script!

Usage: script.py [options] (--here|--there)

Options:
  --here
  --there
  -h, --help
  -v, --version

Have fun then.
"""

import os
import json
try:
    import cPickle as pickle
except ImportError:    # py3 maybe
    import pickle
from docpie import Docpie

pie = None

if os.path.exists('myscript.docpie.pickle'):
    with open('myscript.docpie.pickle', 'rb') as pkf:
        try:
            pie = Docpie.restore_pickle(pickle.load(pkf))
        except BaseException:
            pass

if pie is None and os.path.exists('myscript.docpie.json'):
    # omit `encoding` if you're using python2
    with open('myscript.docpie.json', 'r', encoding='utf-8') as jsf:
        try:
            pie = Docpie.convert_2_docpie(json.load(jsf))
        except BaseException:
            pass

if pie is None:
    pie = Docpie(__doc__)

print(pie.docpie())
```


### preview

after you get your `Docpie` instance, call `.preview()` to have a quick view
that how does `Docpie` undertand your `doc`


Difference
-------------------------------------------------------------------------------

`docpie` is not `docopt`.

1.  `docpie` will trade element in `[]` (optional) as a whole, e.g

    ```python
    doc = '''Usage: prog [a a]...'''
    print(docpie(doc, 'prog a'))  # Exit
    print(docopt(doc, 'prog a a'))  # {'a': 2}
    ```

    Which is equal to `Usage: prog [(a a)]...` in `docopt`.

2.  In `docpie` if one mutually exclusive elements group matches, the rest
    groups will be skipped

    ```python
    print(docpie('Usage: prog [-vvv | -vv | -v]', 'prog -vvv'))  # {'-v': 3}
    print(docpie('Usage: prog [-v | -vv | -vvv]', 'prog -vvv'))  # Fail
    print(docopt('Usage: prog [-v | -vv | -vvv]', 'prog -vvv'))  # {'-v': 3}
    ```

3.  In `docpie` you can not "stack" option and value in this way
    even you specify it in "Options":

    ```python
    """Usage: prog -iFILE   # Not work in docpie

    Options: -i FILE
    """
    ```

    But you can do it in this way:

    ```python
    """Usage: prog -i<FILE>

    Options: -i <FILE>
    """
    ```

4. `docpie` use `Options:` to find the current "Option" section,
    however `docopt` will treat any line in `doc` starts with `-`
    (not counting spaces) as "Options"

5.  Subparsers are not supported currently.


Known Issue
-------------------------------------------------------------------------------

the following situation:

```
Usage: --long=<arg>
```

without announcing `Options` will match `--long sth` and `sth --long`. To
avoid, simply write an announcement in `Options`

```
Usage: --long=<arg>

Options:
 --long=<sth>    this flag requires a value
```

More features
-------------------------------------------------------------------------------

This feature can be expected in the future `docpie`

(Not support currently)

```
Usage: cp.py <source_file>... <target_directory>
```

And the "known issue" may also be solved in the future `docpie`


Developing
-------------------------------------------------------------------------------

execute `/test/test.py` to run the test

the logger name of `docpie` is `"docpie"`

`docpie` contains two developing tools: `bashlog` and `tracemore`. You can
do like:


```python
from docpie import docpie, Docpie, bashlog
from docpie.tracemore import get_exc_plus

logger = bashlog.stdoutlogger('docpie')  # You may init your logger in your way

try:
    docpie(doc)
except BaseException:
    logger.error(get_exc_plus())
```

the code in `bashlog.py` is taken from [tornado](https://github.com/tornadoweb/tornado),
and `tracemore.py` is from [python Cookbook](http://www.amazon.com/Python-Cookbook-Third-David-Beazley/dp/1449340377/ref=sr_1_1?ie=UTF8&qid=1440593849&sr=8-1&keywords=python+cookbook)

<!-- Use `pandoc --from=markdown --to=rst --output=README.rst README.md` to convert this into rst -->
