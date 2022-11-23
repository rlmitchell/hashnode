# DIY Infrastructure Monitoring Part 2

My current organization uses [Let's Encrypt](https://letsencrypt.org/) certificates that expire at 90 days old.  Obviously, we need to know ahead of time when any one of them is going to expire.  Rather than maintaining yet another spreadsheet wouldn't it be nice to be notified of an expiration about 5 days out?  In this post we will automate the checking of SSL/TLS certificates to be informed of an upcoming expiration.  

### Requirements 
We need the pyOpenSSL package for the utility info class we are going to make.

```bash
~/pylib/infracheck$ cat requirements.txt 
requests==2.22.0
pyOpenSSL==22.1.0
~/pylib/infracheck$
```

Install the pip packages in the requirements.txt file with `pip install -r requirements.txt`

### The SSLInfo class

`~/pylib/infracheck/web/ssl_info.py`
```python
from datetime import datetime, timezone
import OpenSSL
import ssl
import socket


class SSLInfo:
    def __init__(self, hostname, port=443):
        self.hostname = hostname
        self.port = port

    def get_num_days_before_expired(self) -> int:
        """
        Get number of days before an TLS/SSL certificate of a domain expired
        """
        try:
            context = ssl.SSLContext()
            connection = socket.create_connection((self.hostname, self.port))
            sock = context.wrap_socket(connection, server_hostname = self.hostname)
            certificate = sock.getpeercert(True)
            pem_cert = ssl.DER_cert_to_PEM_cert(certificate)
            x509 = OpenSSL.crypto.load_certificate(OpenSSL.crypto.FILETYPE_PEM, pem_cert)
            cert_expires = datetime.strptime(x509.get_notAfter().decode('utf-8'), '%Y%m%d%H%M%S%z')
            sock.close()
            connection.close()
            return (cert_expires - datetime.now(timezone.utc)).days
        finally:
            sock.close()
            connection.close()
```


### Using the SSLInfo class
Let's use the SSLInfo class manually.  As we did with [part 1](https://rlmitchell.hashnode.dev/diy-infrastructure-monitoring-part-1),  we will use [github](https://github.com/) to validate the class. 

```bash
~$ python3 
Python 3.8.10 (default, Jun 22 2022, 20:18:18) 
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from infracheck.web.ssl_info import SSLInfo 
>>> SSLInfo('github.com').get_num_days_before_expired()
112
>>> 
```
We can see from the output that (today) github.com's SSL certificate will expire in 112 days. 



### Adding the infrastructure test
`~/infra-tests/test_github_certificate.py`

```python
from infracheck.web.ssl_info import SSLInfo
import unittest

class Test_Github_SSL_Certificate(unittest.TestCase):
    def test_github_certificate_expires_lessthan_5_days(self):
        self.assertTrue(SSLInfo('github.com').get_num_days_before_expired() > 5)
```

### Running the test
We've added the test to our infra-tests directory, now let's run it with our run script from [part 1](https://rlmitchell.hashnode.dev/diy-infrastructure-monitoring-part-1)).

```bash
~/infra-tests$ bash run.sh 
test_github_certificate_expires_lessthan_5_days (test_github_certificate.Test_Github_SSL_Certificate) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.097s

OK
~/infra-tests$ 
```

Now we simply add a test_XXX method for each certificate we want to be informed about. 