---
layout: post
title:  "Learning from Requests Source Code"
date:   2016-06-07 16:16:01 -0600
categories: python
---



Requests (https://github.com/kennethreitz/requests) is a popular http request handling package. Its code is supposed to be well written and pythonic. So I decided to learn from it and improve my Python programming. Here is something that I learned or re-learned.

## How to organize project files?

- **docs**: the documents is written in .rst format supported by Python `docutils` package.
- **requests**: it has the main code of Requests. It is a package. But it also includes other packages in the sub-folder of `packages`.
- **tests**: it is a package with empty `__init__.py`. They use `pytest` package for testing.
- .gitignore: a lot of projects have this
- AUTHORS.rst
- CONTRIBUTING.md
- HISTORY.rst: version notes
- LICENSE
- MANIFEST.in
- Makefile: it includes script to run tests and install required dependencies.
- NOTICE
- README.rst
- requirements-to-freeze.txt
- requirements.txt: it has version requirements for dependents
- setup.py: it uses `setuptools` package. `setuptools`: Easily download, build, install, upgrade, and uninstall Python packages.

**Lesson Learned**
- Use package `setuptools` to install your package
- You can maintain a list of required packages and use `setuptools` to install them all
- `.gitignore` is good to have

## How to organize classes?

- adapters.py: this adapter is an analogy to network adatper.
    - `class BaseAdapter`
    - `class HTTPAdapter`: This is a class used to create http connection and send prepared http requests.
- api.py: it contains a set of functions that are exposed as the api of the requests package. These functions include `request()`, `get()`, `options()`, `head()`, `post()`, `put()`, ... For example:
    - `put(url, data=None, **kwargs)`: sends a PUT request
    - `get(url, params=None, **kwargs)`: sends a GET request
    - `request(method, url, **kwargs)` construct and send a request.
- auth.py: authentication stuff. I don't understand it yet.
    - `class AuthBase`
    - `class HTTPBasicAuth`
    - `class HTTPProxyAuth`
    - `class HTTPDigestAuth`
- cacert.pem: contains certificates.
- certs.py: it is a standalone file that prints out the path of `cacert.pem` file.
- compat.py: for compatibility issue. Mostly between python2 and python3. For example, it defines `bytes = str` for python2 and `bytes = bytes` for python3.
- cookies.py: cookie is essentially a dictionary that holds information for a website. 
    - MockRequest: they need these Mock classes because `cookielib`(third party) does not know how to use Request class (it only knows how to use `urllib2.Request`). These mock classes are actually adatpers (design pattern).
    - `class MockResponse`
    - `class RequestsCookieJar`
    - `class create_cooke()`
    - ...
- exceptions.py: This module contains the set of Requests' exceptions. Most of the definitions are empty. Examples of exceptions include `InvalidURL`, `RetryError`, `ConnectionError`.
- hooks.py: hooks (callable objects) are provided by user. Hooks will be called by `dispatch_hook()` after receiving the response of request. 
    - `default_hooks()`
    - `dispath_hook()`
- `__init__.py`: expose package APIs. For example: `from .api import request, get, head, post, patch, put, delete, options`
- models.py: it has primary classes for Requests. They are mainly used by session.py
    - class Request
    - class PreparedRequest
    - class Response
- packages: this directory has packages `chardet` and `urllib3`. `packages/__init__.py` simply do `from . import urllib3` and `from . import chardet`.
- sessions.py: it provides `class Session` to maintain settings (proxies, cookies, auth) across requests. 
- status_codes.py: it is a dictionary of http status code and explanations, such as 404.
- structures.py: it has two data structures, `CaseInsensitiveDict` and `LookupDict`. LookupDict is a subclass of dict. It overrides `get()` and `__getitem__()` to retrive value from `__dict__`.
- utils.py: a bunch of helper functions.

**Lesson Learned**
- Separate code to files if they are not logically in the same module. For example, don't put `class Session` and `class HTTPAdapter` in the same file
    - Physically separating code help you metally separate them and clarify the relationship
    - Physically cleaning code help you conduct stress-free thinking
- Files with a few lines is fine. `compat.py` is short and it is fine.
- package names are all in lower case and wit no dash or underscore.

## Makefile: how to install required Python packages?

```makefile
init:
	pip install -r requirements.txt
```

This command reads requirements.txt and try to install the required Python packages. pip reads the requirements file. 

## Makefile: how to show test coverage?

```makefile
coverage:
	py.test --verbose --cov-report term --cov=requests tests
```
Show coverage report. It shows how much code is covered by the tests. (So you know how good your tests are.)
https://pypi.python.org/pypi/pytest-cov

## Makefile: how to export test results to xml?

```
ci: init
	py.test --junitxml=junit.xml
```

It creates test results in XML, which can be visualized later. https://github.com/kyrus/python-junit-xml

## How to write setup.py

This file uses `setuptools` package to implement package distribution, i.e. making `.egg` cross-platform file, gathers all dependant packages in the place where `setup.py` lives. 

```python
with open('requests/__init__.py', 'r') as fd:
    version = re.search(r'^__version__\s*=\s*[\'"]([^\'"]*)[\'"]',
                        fd.read(), re.MULTILINE).group(1)
```

The code searches the whole file for the version. Note the usage of `re.MULTILINE`.


## How to run tests

To be able to run the tests, you need to 
```text
sudo make init
make test
```

`make test` will turn to `py.test tests`.

To run a specific test, do `py.test -k test_path_is_not_double_encoded`.

`tests/__init__.py` is empty. But it is required to make it a 'package'.

## test_requests.py: what are the good practices of importing?

Classes and functions are imported as needed. 

You can import package and functions in the package at the same time. For example:

```python
import requests
from requests.adapters import HTTPAdapter
```

`HTTPAdatper` is a class.

## What does `from .compat import StringIO, u` mean?

The `.` is a shortcut that tells it search in current package before rest of the `PYTHONPATH`. So, if a same-named module Recipe exists somewhere else in your `PYTHONPATH`, it won't be loaded.

http://stackoverflow.com/questions/22511792/python-from-dotpackage-import-syntax


## compat.py: how to handle package compatibility?

```python
try:
    import StringIO
except ImportError:
    import io as StringIO
```

This file is intended for package compatibility. The code above imports a proper package to the namespace of `compat` as `StringIO`. So later another file can simply do `from .compat import StringIO` to get a proper package of `StringIO`. Doing so in a file instead of as a function is to avoid multiple instances of importing.

## How handle unicode in python2 and python3 with compatiblly?

```python
if is_py3:
    def u(s):
        return s
else:
    def u(s):
        return s.decode('unicode-escape')
```

Define different implementation of a function based on Python version. The function is used to decode unicode in non-python3, for example: `u('http://www.example.com/üniçø∂é')`.

## conftest.py: how to use py.test fixture?
```python
def prepare_url(value):
    # Issue #1483: Make sure the URL always has a trailing slash
    httpbin_url = value.url.rstrip('/') + '/'

    def inner(*suffix):
        return urljoin(httpbin_url, '/'.join(suffix))

    return inner


@pytest.fixture
def httpbin(httpbin):
    return prepare_url(httpbin)

```

In conftest.py, `httpbin()` is in fact an override of fixture `httpbin` in `pytest-httpbin` package. The proof that pytest depends on pytest-httpin is 

```text
jun@jun-VirtualBox:~/workdir/requests$ greppy pytest-httpbin
./setup.py:test_requirements = ['pytest>=2.8.0', 'pytest-httpbin==0.0.7', 'pytest-cov']
```

Here is an introduction of the `pytest-httpbin` package: 

```text
httpbin is an amazing web service for testing HTTP libraries. It has several great endpoints that can test pretty much everything you need in a HTTP library. The only problem is: maybe you don't want to wait for your tests to travel across the Internet and back to make assertions against a remote web service (speed), and maybe you want to work offline (convenience).

Enter pytest-httpbin. Pytest-httpbin creates a pytest fixture that is dependency-injected into your tests. It automatically starts up a HTTP server in a separate thread running httpbin and provides your test with the URL in the fixture. Check out this example:
```

https://github.com/kevin1024/pytest-httpbin

The new `httpbin` fixture is a function that takes suffix and form a url to the httpbin address. For example: in test, if you do 

```lasso
def test_httpbin(httpbin):
    print httpbin("hello", "world")
```

It will output  http://127.0.0.1:43559/hello/world


## test_requests.py: how to pass parameters to test functions?
```python
    @pytest.mark.parametrize(
        'exception, url', (
            (MissingSchema, 'hiwpefhipowhefopw'),
            (InvalidSchema, 'localhost:3128'),
            (InvalidSchema, 'localhost.localdomain:3128/'),
            (InvalidSchema, '10.122.1.1:3128/'),
            (InvalidURL, 'http://'),
        ))
    def test_invalid_url(self, exception, url):
        with pytest.raises(exception):
            requests.get(url)
```

`@pytest.mark.parametrize` is a way of passing in parameter values to `test_invalid_url()`. So `test_invalid_url(MissingSchema, '
'hiwpefhipowhefopw'), test_invalid_url(InvalidSchema,  'localhost:3128'),...` will be called. 


## how to always do true division? (1/3=0.3333, not 0)
Use `from __future__ import division`. Division will always return approximation (instead of flooring). For example: `4/2=2.0`.

https://www.python.org/dev/peps/pep-0238/


## sessions.py: how does the module uses factory pattern？
This file has a factory that returns an instance of `Session()` class. The `session()` function and `Session()` class are exposed as first-level interface of the package by `__init__.py`: 

```python
from .sessions import session, Session`
```

## How to define functions in base class?

Base class' functions raise `NotImplementedError` instead of being abstract.  This is less strict as it you can inherit without implementing an 'abstract' function. 

## How to define interface/abstraction?

`requests/adapters.py` defines `class BaseAdapter` and then `class HTTPAdapter(BaseAdapter)`. It could have been only `class HTTPAdapter(object)`. But declaring `class BaseAdatper` allows people to implement their own adapter, such as `class MyCaseHTTPAdapter(BaseAdatper)`, which will work with the rest of Requests code. `class BaseAdatper` is the interface/abstraction that other parts of the code depends on. 

## How api of a package is exposed?

Requests defines the main APIs in `api.py`, and then expose them in `requests/__init__.py` by 

```python
from .api import request, get, head, post, patch, put, delete, options
```

Also in `__init__.py`, it exposes exceptions, utils, status_codes and some more, by

```python
from .models import Request, Response, PreparedRequest
from .sessions import session, Session
from .status_codes import codes
from .exceptions import (
    RequestException, Timeout, URLRequired,
    TooManyRedirects, HTTPError, ConnectionError,
    FileModeWarning, ConnectTimeout, ReadTimeout
)
```

## How to manage exceptions of a class?

Exception class does not need to be fancy. You can simply subclass an existing exception. For example, in `cookies.py`, it defines an exception by subclassing `RuntimeError`. This exception class contains nothing. It serves as a unique ID so the program can identify which exception is throwed.

```python
# Note that they don't put a 'pass' in the definition
class CookieConflictError(RuntimeError):
    """There are two cookies that meet the criteria specified in the cookie jar.
    Use .get and .set and include domain and path args in order to be more specific."""
```

There are plenty of empty exception classes defined in `exceptions.py`.

https://github.com/junhe/requests/blob/jun.comments/requests/exceptions.py

## What's a value that dict user will never use?

If you have a dictionary and you want to 'return None if you cannot find the key', you may do:

```python
val = mydict.get('key3', None)
if val is None:
   print 'key3 does not exist'
```

This is WRONG, because user may have key-value pair 'key3':None. The correct way is 

```python
_Null = object()
val = mydict.get('key3', _Null)
if val is _Null:
   print 'key3 does not exist'
```

Or you can use `KeyError` exception.


## How to log in python?

Use `logging` package.

https://github.com/junhe/requests/blob/jun.comments/requests/packages/urllib3/poolmanager.py#L34

## What if I have a data class (almost has no methods) that I need to define?

Simply use `namedtuple`, or subclass `namedtuple`. Example: https://github.com/junhe/requests/blob/jun.comments/requests/packages/urllib3/_collections.py



## Why use `is None` instead of `== None`?
> Comparisons to singletons like None should always be done with `is` or `is not` , never the equality operators.

> Also, beware of writing `if x` when you really mean if x is not None -- e.g. when testing whether a variable or argument that defaults to None was set to some other value. The other value might have a type (such as a container) that could be false in a boolean context!

> 'is' compares if they are the same object, '==' compares if they have the same value and '==' can be overrided by '__eq__()'

More:
http://stackoverflow.com/questions/7816363/if-a-vs-if-a-is-not-none
http://stackoverflow.com/questions/3257919/is-none-vs-none
http://jaredgrubb.blogspot.com/2009/04/python-is-none-vs-none.html

## What if I don't want to write a lot of `&&`?

Don't forget you have `all()`.

Also `any()`.

## How to check if a object is a (general) dictionary?

`isinstance(myobj, Mapping)`

## How to delete some keys whose values are None?

```python
none_keys = [k for (k, v) in merged_setting.items() if v is None]
for key in none_keys:
    del merged_setting[key]
```

Also correct (because `.items()` returns a list of (key, value), not a iterator:

```python
for k, v in merged_setting.items():
    if v is None:
        del merged_setting[key]
```

You can also create new dictionary.

## How to subclass a mapping class?

First, it is easier to store the data as a member variable (such as 'self._store' here), instead of storing them in `self`. If you store it to `self`, it is quite easy to incur infinite recursive calls.

Second, the arguments in `__init__(self, data=None, **kwargs)` allows initializations like `CaseInsensitiveDict({1:1, 2:2})`, and `CaseInsensitiveDict(key1='v1', key2='v2')`.

```python
class CaseInsensitiveDict(collections.MutableMapping):
    def __init__(self, data=None, **kwargs):
        """
        This is a regular constructor of dict. data can be mapping 
        or an iterable. kwargs will become k-v pairs in the dict.
        """
        self._store = dict()
        if data is None:
            data = {}
        self.update(data, **kwargs)
```

## What does the 'b' prefix do in Python 2 and 3?

```python
# Prefix 'b' marks byte literal
# >>> a = b'3f'
# >>> type(a)  # Python 2.7
# <type 'str'>
# >>> type(a)  # Python 3.4
# <class 'bytes'>
```

## How to use lambda function?

```python
# equal to:
# def get_proxy(k):
#   return os.environ.get(k) or os.environ.get(k.upper())
get_proxy = lambda k: os.environ.get(k) or os.environ.get(k.upper()) 
```

Also note that the `or` make the code shorter here.

