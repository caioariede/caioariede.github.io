+++
date = "2015-12-15T14:31:05-02:00"
draft = false
title = "Intercepting function calls in Python scripts"

+++

This post is about [pyintercept](https://github.com/caioariede/pyintercept). A
tool that I created to intercept functions in Python scripts without its source
code. It patches the bytecode before executing it.

# A piece of history

When I started to develop [Codenizer](http://codenizer.co/), the first
challenging feature I faced was the dependency tracking. The tracking part is
almost trivial, but extracting dependencies without actually running
anything, like `python setup.py install` is hard.

I could install it in an isolated environment like Docker, than check the
installed libraries by doing something like `pip freeze` but I wanted to avoid
the hassle of compiling heavy-weight stuff just to know its dependencies.

# Installation

    pip install pyintercept

# Usage

This is an example on how to use pyintercept to get the
[Wagtail](http://github.com/torchbox/wagtail) dependencies.

```
$ git clone https://github.com/torchbox/wagtail.git # Wagtail 1.2
$ cd wagtail
$ python -m pyintercept setup.py setuptools.setup --args=install --handler=pyintercept.pdb
```

Where:

- `setuptools.setup` is the dotted path to the function will want to intercept
- `--args=install` is used to pass arguments to the script (`setup.py`)
- `--handler=pyintercept.pdb` is used to set the handler you want. There are
some predefined handlers: `json`, `pdb`, `pickle` and `print`.

Will drop you in a pdb console:

```
(Pdb) l
1  	def pdb(origfn, *args, **kwargs):
2  	    import pdb; pdb.set_trace()
3  ->       return origfn(*args, **kwargs)
```

```
(Pdb) p origfn
<function setup at 0x1020ed9b0>
```

```
(Pdb) from pprint import pprint
(Pdb) pprint(kwargs['install_requires'])
['Django>=1.7.1,<1.10',
'django-compressor>=1.4',
'django-modelcluster>=1.0',
'django-taggit>=0.17.5',
'django-treebeard==3.0',
'djangorestframework>=3.1.3',
'Pillow>=2.6.1',
'beautifulsoup4>=4.3.2',
'html5lib>=0.999,<1',
'Unidecode>=0.04.14',
'Willow>=0.2.2,<0.3']
```

We just intercepted the call to `setuptools.setup` without modifying any source
code. The `origfn` argument contains the original function. The `*args` and
`**kwargs` contains the arguments that would be passed to the original
function.

## Advanced usage

### Defining a custom handler

You can also, of course, write your own handlers. Let's write a custom handler
that prints out to stdout all requirements in JSON format.

Let's call it `amazing.py`:

```
def handler(*args, **kwargs):
    import json
    print(json.dumps(kwargs.get('install_requires')))
```

```
python -m pyintercept setup.py setuptools.setup --args=install --handler=amazing.handler
["Django>=1.7.1,<1.10", "django-compressor>=1.4", "django-modelcluster>=1.0", "django-taggit>=0.17.5", "django-treebeard==3.0", "djangorestframework>=3.1.3", "Pillow>=2.6.1", "beautifulsoup4>=4.3.2", "html5lib>=0.999,<1", "Unidecode>=0.04.14", "Willow>=0.2.2,<0.3"]
```

It's important to notice the `import` statement inside the handler function.
You have to ensure that all the required stuff to make your handler work must
be defined within it, otherwise it will not be injected and will cause errors.
