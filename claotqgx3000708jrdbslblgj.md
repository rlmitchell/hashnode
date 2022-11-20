# DIY Infrastructure Monitoring Part 1

I've spent many years in the IT/DevOps world and one thing I've learned is when you have a production issue it is of paramount importance to find the cause of the issue as fast as possible.  A lot of organizations subscribe to paid monitoring services like [Datadog](https://www.datadoghq.com/) or [Honeycomb](https://www.honeycomb.io/) which is perfectly fine for day to day operations, but may not identify the cause of an intricate issue immediately.   Suppose you are just monitoring that containers are running, that wouldn't identify that a particular endpoint served by a particular container was serving bad data. 

In this series we will create a Python library and unit tests that interrogate infrastructure and applications.  The tests can be run to check an entire tech stack as thoroughly as needed.  It is not meant as a full data metrics replacement, but rather a customized way to check every needed aspect of your infrastructure and applications rapidly.  


### Setup
On your control machine, verify your python and pip versions.  
```bash
$ python3 --version && pip --version 
Python 3.8.10
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
$ 
```

If Python3 and/or pip is not installed you can install it with apt on Ubuntu:
```
$ sudo apt -y update && sudo apt install python3 python3-pip 
```

We need a directory location for our library.  Let's make the ~/pylib/infracheck directory:
```bash
$ mkdir -p ~/pylib/infracheck
```

For our library we will need a requirements.txt file for pip packages.  The first class in the library will use the requests package so let's setup our requirements file and install it. 
```bash
~/pylib/infracheck$ cat requirements.txt 
requests==2.22.0
~/pylib/infracheck$
~/pylib/infracheck$ pip install -r requirements.txt 
```

Next we need a place to put and run our tests from:
```bash
$ mkdir infra-tests 
```

To run our tests we will need to include our pylib directory in the PYTHONPATH.  Let's setup a runner script that will set the PYTHONPATH and run all the tests it finds.
```bash
~/infra-tests$ cat run.sh 
#!/bin/bash

export PYTHONPATH=$PYTHONPATH:~/pylib
/usr/bin/python3 -m unittest discover -v 
~/infra-tests$ 
```

Finally run the script to verify it works
```bash
~/infra-tests$ bash run.sh 

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
~/infra-tests$
```

### Endpoint data retrieval class
The first thing we want to check is that our endpoint returns the correct data.  By checking for a string in the data that is returned we verify that the site is up and is returning correct data.  For this example I will use my [github](https://github.com/rlmitchell) page as the target endpoint.

This basic class issues a get request to a URL.  It has a method that looks for a given string in the response data returning True if the string exists.

`~/pylib/infracheck/web/endpoint.py`:
```python
import requests

class Endpoint:
    def __init__(self, url):
        self.response = requests.get(url) 
    
    def text_in_response(self, text):
        return text in self.response.text 
```

Let's verify the code by using the class in a pass and a fail condition.  I know that the text 'Norman, OK' should be returned from my github page.

Here is the class working correctly with a True condition for text_in_response() and with a False condition for text_in_response()

```bash
~/infra-tests$ export PYTHONPATH=~/pylib 
~/infra-tests$ python3 
Python 3.8.10 (default, Jun 22 2022, 20:18:18) 
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from infracheck.web.endpoint import Endpoint
>>> endpoint = Endpoint('https://github.com/rlmitchell')
>>> endpoint.text_in_response('Norman, OK') 
True
>>> endpoint.text_in_response('Norman, TX') 
False
>>> quit()
~/infra-tests$ 
```

Now that we know our logic works we can setup a test that will check the site on-demand.

### Endpoint data test
Here we have two tests in a test class.  Unittest will discover an invoke functions that start with 'test'.  The Python file also needs to be named with 'test' at the beginning of the filename.

`test_github_account.py`:
```python
from infracheck.web.endpoint import Endpoint
import unittest

class Test_Github_Account_Endpoints(unittest.TestCase):
    def test_github_account_location_correct(self):
        self.assertTrue(Endpoint('https://www.github.com/rlmitchell').text_in_response('Norman, OK'))

    def test_github_account_linkedin_url_correct(self):
        self.assertTrue(Endpoint('https://www.github.com/rlmitchell').text_in_response('https://www.linkedin.com/in/rob-mitchell-a5b0575/'))
```

### Run the test
```bash
~/infra-tests$ bash run.sh 
test_github_account_linkedin_url_correct (test_github_account.Test_Github_Account_Endpoints) ... ok
test_github_account_location_correct (test_github_account.Test_Github_Account_Endpoints) ... ok

----------------------------------------------------------------------
Ran 2 tests in 1.102s

OK
~/infra-tests$
```
Note that the unittest module detected and ran the tests founds in `test_github_account.py` without the methods having to be called directly. 

&nbsp;

Now you can programmatically check endpoints for correct data easily.  As a DevOps engineer you can test/check all the endpoints in your infrastructure quickly with minimal manual effort.



