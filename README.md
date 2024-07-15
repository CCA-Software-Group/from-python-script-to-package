# from-python-script-to-package

Outgrowing your Python notebooks or simple scripts, but not sure where to go next? 

Ready to distribute your code to collaborators, or just package it for your own ease of installation on new computers?

We can help.

## Choose your own adventure! Does one of these sound relatable?

1. I have accumulated a lot of code in a single `.py` file (or notebook) and I want to reuse parts of it without copying and pasting every time
2. I have a folder full of `.py` files, but I want to keep things more organized so that I can share code between my projects
3. I have organized some of my code into a folder that I can import, but I'd still like to understand why I can `import numpy` in scripts anywhere but `import mypackage` only works in one folder
4. I want to make my code easier to install for collaborators (and on new computers), but I'm not ready to "publish" it
5. I've got things working for myself and my collaborators, but I'd like to share with the world

    * ...and there's some data I want to package with it
    * ...and I want to include extensions written in C (or other compiled code)
6. I've published a package, but it was kind of a lot of steps to release a new version, so it's a chore I'm avoiding

## 1. I have accumulated a lot of code in a single `.py` file (or notebook) and I want to reuse parts of it without copying and pasting every time

We all love Python's low barrier to entry. The simplest Python program is a text file, which you can run as soon as you save.

#### [`myscript.py`](chapter-1/00-starting-point/myscript.py)

```python
print('Hello')
```

Save and run, and:

```
$ python myscript.py
Hello
$
```

(This was a big deal when Python was new. If this sounds trivial, check out how C++ code turns into something you can run!)

The downside of this freedom is that your scripts can quickly become long and convoluted, so you pull things out into functions, like this contrived example:

#### [`myscript.py`](chapter-1/01-importable-function-before/myscript.py)
```python
def greet(greeting='Hello'):
    print(greeting)

greet()
```

You probably already use `import`, e.g. `import numpy as np`. Wouldn't it be nice to use `import` to get `greet` from `myscript`? Well, this very common case is easily supported by Python as long as the files are in the same folder.

#### [`myotherscript.py`](chapter-1/01-importable-function-before/myotherscript.py)
```python
import myscript
myscript.greet('Howdy')
```

But when you run it...

```
$ python myotherscript.py
Hello
Howdy
$
```

### What's going on?

When you do `python myotherscript.py`, Python runs your script from beginning to end, top to bottom. When it encounters an `import`, it puts your script on hold while it goes and finds the imported package, and runs *that* beginning to end, top to bottom.

What's at the bottom of `myscript2.py`? Yep, it's calling `greet()` there. That will run whenever it's imported in a new script, which is usually undesirable behavior.

This is common when scripts grow organically, where you end up with a mix of functions and "imperative" or "procedural" code that calls them when a script is run to accomplish some task.

### How do I stop that?

You have multiple options. 

**Option 1:** You can pull the procedural code, the call to `greet()` in `myscript.py` in this example, into a different file. For example,

#### [`myscript_functions.py`](chapter-1/02-importable-function-after-opt-1/myscript_functions.py)
```python
def greet(greeting='Hello'):
    print(greeting)
```

#### [`myscript.py`](chapter-1/02-importable-function-after-opt-1/myscript.py)
```python
from myscript_functions import greet
greet()
```

#### [`myotherscript.py`](chapter-1/02-importable-function-after-opt-1/myotherscript.py)

```python
from myscript_functions import greet
greet('Howdy')
```

Then you would run `python myscript.py`, and it would call the function as before...

```
$ python myscript.py
Hello
```

But `python myotherscript.py` would not call `greet()` twice (like it did in the previous example):

```
$ python myotherscript.py
Howdy
```

**Option 2:**

If you move all your procedural code to the end of the file, you can just make an indented block and use an `if __name__ == "__main__":` statement to ensure it only runs when you want it to. For example,

#### `myscript.py`
```python
def greet(greeting='Hello'):
    print(greeting)

if __name__ == "__main__":
    greet()
```


The `__name__` variable in the condition is "special", in that it's set automatically by Python itself. When the script is run directly, the `__name__` is always set to `"__main__"`.

The `myotherscript.py` file is unchanged from the previous example:

#### [`myotherscript.py`](chapter-1/03-importable-function-after-opt-2/myotherscript.py)
```python
import myscript
myscript.greet('Howdy')
```

Now, if you do `python myscript.py` you get a greeting, but if you do `import myscript` you will not:

```
$ python myscript.py
Hello
$ python
>>> import myscript
>>>
```

That change makes `myotherscript.py` work the way you probably wanted all along:

```
$ python myotherscript.py
Howdy
```

**But wait, what if I develop new functionality in `myotherscript.py` and want to import from _it_ some day? Won't it have the same problem?**

Yep! Read on for some more information on code organization.

### Which option is the best one?

If you have some procedural code in a module with importable functions, it's generally a good idea to have procedural code behind a conditional (`if __name__ == "__main__":`). On the other hand, mixing procedural code and your helper functions can be confusing.

If you can, separating your scripts into a library of functionality to import, and another file that uses that library, will help keep your code organized as it grows.

So, in this hypothetical example, if we wanted to add a function called `farewell()` to `myotherscript.py`:

#### [`myotherscript.py`](chapter-1/04-code-organization-before/myotherscript.py)
```python
import myscript

def farewell(message='See ya.'):
    print(message)

myscript.greet('Howdy')
farewell()
```

the better option for later reuse of the code is to put that function in a "library" module that both `myscript.py` and `myotherscript.py` import:

#### [`pleasantries.py`](chapter-1/05-code-organization-after/pleasantries.py)
```python
def greet(greeting='Hello'):
    print(greeting)

def farewell(message='See ya.'):
    print(message)
```

#### [`myscript.py`](chapter-1/05-code-organization-after/myscript.py)
```python
from pleasantries import greet

greet()
```

#### [`myotherscript.py`](chapter-1/05-code-organization-after/myotherscript.py)
```python
from pleasantries import greet, farewell

greet('Howdy')
farewell()
```

A common pattern for when you want to keep your options open, (code-organization-wise) as your project grows is to write the procedural code in a `main` function, and then include a block at the end that calls `main` when the script is run directly.

If you've written C/C++ before, this probably [sounds familiar](https://en.wikipedia.org/wiki/C_syntax#Global_structure).

Obviously, this is overkill for this trivial example, and makes things much more verbose, but here's what that would look like.

#### [`pleasantries.py`](chapter-1/06-code-organization-main-funcs/pleasantries.py)
```python
def greet(greeting='Hello'):
    print(greeting)

def farewell(message='See ya.'):
    print(message)
```

#### [`myscript.py`](chapter-1/06-code-organization-main-funcs/myscript.py)
```python
from pleasantries import greet

def main():
    greet()

if __name__ == "__main__":
    main()
```

#### [`myotherscript.py`](chapter-1/06-code-organization-main-funcs/myotherscript.py)
```python
from pleasantries import greet, farewell

def main():
    greet('Howdy')
    farewell()

if __name__ == "__main__":
    main()
```

### What about Jupyter notebooks?

Jupyter notebooks are not well-suited to holding your reusable code, but can easily _use_ the code by importing it, as long as they are located in the same folder as the module. (For more complex cases, where you want to have your library of functionality available regardless of where your notebook is saved, read on.)

## 2. I have a folder full of `.py` files, but I want to keep things more organized so that I can share code between my projects

So far, our code is organized in a "flat hierarchy" (a contradiction in terms, of course). All the functionality and all the scripts using said functionality live in the same folder, importing as needed.

#### [`myotherscript.py`](chapter-2/00-starting-point/myotherscript.py)
```python
from pleasantries import greet, farewell

greet('Howdy')
farewell()
```

#### [`pleasantries.py`](chapter-2/00-starting-point/pleasantries.py)
```python
def greet(greeting='Hello'):
    print(greeting)

def farewell(message='See ya.'):
    print(message)
```

For the sake of illustration I've added another library of functions in `arithmetic.py`:

#### [`arithmetic.py`](chapter-2/00-starting-point/arithmetic.py)
```python
def add(a, b):
    return a + b

def sub(a, b):
    return a - b
```

And I've updated `myscript.py` to import from it:

#### [`myscript.py`](chapter-2/00-starting-point/myscript.py)
```python
from pleasantries import greet
from arithmetic import add

greet()
print("Result", add(40, 7))
```

Since they're part of the same project, we might want to organize `pleasantries.py` and `arithmetic.py` in some way that groups them. More importantly, by organizing them under a top-level "package" name (like `numpy`, or `astropy`, for example) we avoid the potential for name "collisions".

### What is a name collision?

Consider the code in [chapter-2/01-bad-name-collision-before/](chapter-2/01-bad-name-collision-before):

#### [`example.py`](chapter-2/01-bad-name-collision/example.py)
```python
import math
def mult(a, b):
    return a * b
print("Pi is approximately", math.pi)
print("2 * Pi is approximately", mult(2, math.pi))
```

This code uses the [`math` standard library module](https://docs.python.org/3/library/math.html) that comes with Python to retrieve the value of pi.

We want to share our `mult()` function between scripts, so we move it out to a new file as discussed in the last chapter. We call it `math.py`.

#### [`math.py`]()
```python
def mult(a, b):
    return a * b
```

#### [`example.py`]()
```python
import math
from math import mult

print("Pi is approximately", math.pi)
print("2 * Pi is approximately", mult(2, math.pi))
```

This example makes the problem somewhat obvious, but let's try running the script anyway:

```
$ python example.py
Traceback (most recent call last):
  File "from-python-script-to-package/chapter-2/02-bad-name-collision/example.py", line 4, in <module>
    print("Pi is approximately", math.pi)
                                 ^^^^^^^
AttributeError: module 'math' has no attribute 'pi'
```

The core of the problem is that `import math` is **ambiguous** when there is a `math.py` or a `math/` folder in your project. So, there is no error importing `mult` from `math`, because it picks up the `math.py` in the same folder as `example.py`. But when you need functionality from the `math` standard library module, it's not there, because the two modules have "collided".

Python has an [extensive standard library](https://docs.python.org/3/library/index.html), so many names are taken... can we just never use names like `math` or `queue` or `random` as Python module names?

The solution is grouping your modules as a **package**. Python distinguishes between "modules" that are single Python files and "packages" that are collections of modules. Both of them are used with the `import` statement, and the term "package" has a bunch of other mostly-compatible definitions—some of which we'll see later—so you'll sometimes hear folders of modules called modules themselves... the terminology is a bit of a mess.

We can resolve the issue in this script by creating a package folder called `myproject` and plopping our new `math.py` in there. We also [should create an `__init__.py` file](https://stackoverflow.com/questions/37139786/is-init-py-not-required-for-packages-in-python-3-3), which can be empty.

See the result in [`chapter-2/03-bad-name-collision-fixed`](chapter-2/03-bad-name-collision-fixed), or in the diagram below:

```
chapter-2/03-bad-name-collision-fixed
├── example.py
└── myproject
    ├── __init__.py
    └── math.py
```

We also need to update `example.py` with the new name.

#### [`example.py`]()
```python
import math
from myproject.math import mult

print("Pi is approximately", math.pi)
print("2 * Pi is approximately", mult(2, math.pi))
```

The collision is gone! Grouping related modules into a package is the first step in making your code easier to maintain and organize.

## 3. I have organized some of my code into a folder that I can import, but I'd still like to understand why I can `import numpy` in scripts anywhere but `import mypackage` only works in one folder

## 4. I want to make my code easier to install for collaborators (and on new computers), but I'm not ready to "publish" it

## 5. I've got things working for myself and my collaborators, but I'd like to share with the world

### 5.a. ...and there's some data I want to package with it

### 5.b. ...and I want to include extensions written in C (or other compiled code)

## 6. I've published a package, but it was kind of a lot of steps to release a new version, so it's a chore I'm avoiding