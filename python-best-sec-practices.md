# Defenselayers Python Security Best Practices version 1.0
Aleksander P. Czarnowski, Defenselayers Sp z o.o.
Copyright 2020

## Switch to Python 3
Python 2 is no longer supported. Most of the packages have been already converted and now are compatible with Python 3. 

__Defenselayers secure container image:__
* Each of our python-based secure container images include recent Python 3.x interpreter with latest patches and fixes 
including binary dependencies.

## Manage your dependencies
Most of the best practices recommend creating separate _virtualenvs_ (for example: `python3 -m venv ./myenv`) to manage 
your dependencies, but in case of containers you should install only single microservice application into single 
container, therefore in most cases there is no need for dedicated _virtualenv_. 

> Always install packages using `pip` (`pip3` if you have installed both Python 2.x and 3.x).

> Generate project dependencies using `pip3 freeze > filename.txt` which lets you use `pip3 install -r filename.txt` 
> to install them later. Keep application dependencies and development dependencies in separate files, for example: 
> `requirements.txt` and `devel-requirements.txt`.

__Defenselayers secure container image:__
* Each of our python-based secure containers includes `pip3` by default
* Each of our python-based secure containers includes `pipenv ` by default
* Each of our python-based secure containers includes secure version of the relevant dependencies by default

## Do not use outdated packages
Never use outdated or vulnerable packages in your dependencies.

## Do not use assertions to control execution flow
Remember that `assert` is not executed in optimized (compiled to bytecode) form of python code. Therefore, do not use 
assertions outside your unit test files to control execution flow, e.g. as a replacement of `if` statement. 

## Validate and sanitize user input
Never trust _any_ user input data which always must be treated as __untrusted__ (in some literature called _tainted_). 
The input data can come from various sources including: network connection, database data, file data etc. 

> Always validate and sanitize user input. Crate __threat models__ of your application to better understand control and 
> data flows.
 
Some python packages are insecure by design when handling user controlled input. XML processing is a great 
example: [https://docs.python.org/3/library/xml.html#xml-vulnerabilities](https://docs.python.org/3/library/xml.html#xml-vulnerabilities). 

In result, this may affect XML-RPC handling, particularly `xmlrpc.server` module.

> Please note that Python 2 handles input data differently than Python 3. In some cases Python 3 does it more securely. 
> However even in case of Python 3 input validation is still obligation.

## Be careful with `import` and `reload` statements
Keep in mind that every `import` statement results in code execution. Be careful with what and when you are 
importing. Do not import nor reload package names that can be controller by an __attacker__/__user input__.

## Race condition and file I/O
Always use `with` statement when opening a file to avoid race conditions and other issues.

The correct syntax is:
```python
with open(filename, 'rt') as fin:
	fin.readlines()
```

## Handle exceptions correctly
Always keep crucial parts of your code inside `try` and `exception` blocks to handle exception gracefully. When 
catching exceptions do not leave `exception` name empty (thus catching all possible exceptions, not just particular 
ones) nor use single `pass` inside `exception` block unless this is intentional. 

## Serialization and deserialization
Avoid deserialization of data from untrusted sources. This mainly applies to usage of `pickle` and `cPickle` modules 
but also to other serialization frameworks. 

When parsing YAML documents always use `yaml.safe_load` to limit ability to construct an arbitrary Python object which
always should be considered dangerous action. 

## Temporary files
Handling temporary files in a secure way may be tricky to implement. Fortunately Python provides `tempfile` module to 
handle it and unless you really know what you are doing, you should use it. When using `tempfile` do not use deprecated
`tempfile.mktemp()`.
 
## Example of insecure calls and functions
Extreme caution must be taken when calling following functions, especially if their parameters may be controlled by 
an attacker (__user input__):

* `__import__`
* `eval`
* `exec`
* `import`
* `importlib.reload`
* `open`
* `os.popen`
* `os.spawn*`
* `os.system`
* `os.walk`
* `reload`
* `subprocess.run`

> The above list is not complete (and will never be), those are only examples.

## Protect secrets
Protect personal data, passwords and other sensitive or confidential data (including authentication data). For
example, encrypt passwords when stored in files or databases. When using encryption ensure that you use __strong__ 
encryption algorithms.   

__Defenselayers secure container image:__

* `deflayers-flask` secure container image is delivered with `bcrypt` package to enable strong password protection and data encryption. All required dependencies are already compiled and available to use out-of-the-box, meaning no additional action is required by the user.

## Do not use cgi module
Handling `CGI` is not trivial. Use other frameworks if possible. If you must use `cgi` module do not enable `cgitb` 
module: `cgit.enable()` as it would enable debug data.

> Some popular web frameworks like `Django` and `Flask` provide their own set of debugging aids. `DEBUG` variable is
> one example. Never enable debug mode in production environments.

__Defenselayers secure container image:__
* `deflayers-flask` and `deflayer-django` could be great alternatives for developing web apps which makes using `cgi` module not necessary.  

## DB-API
Unless you really need to, do not use `DB-API` interface directly. Use one of the available ORM frameworks instead. 
Most of the frameworks allow you to construct SQL queries in case ORM does not provide particular functionality by 
itself. 

When constructing SQL query strings be careful with __untrusted data__. Also construct SQL query string using `?` 
symbol like this:

```python
# secure SQL query string
symb = ('DL')
c.execute('SELECT * FROM items WHERE symbol=?', symb)
```

## Bandit is not the ultimate security tester
`Bandit` is a popular static analysis tool for python source code. However its capabilities of detecting 
even simple vulnerabilities and security issues in your code are very limited. Therefore, on the contrary to many other
best practices we do not recommend running `bandit` alone as it can provide a false sense of security. However it may 
be treated as an additional defensive measure.

> As a side note: running [PEP8](https://www.python.org/dev/peps/pep-0008/) checkers (like `pep8` plugin for `pytest`)
> is a good quality practice but can not replace code reviews and audits. Manual security code review, especially
> risk-based ones, are still providing in many cases a lot better results than simple static check provided by most
> freeware and commercial source code scanners.  
