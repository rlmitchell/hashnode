# Using python-decouple for AWS credentials

### Requirements

You may have a situation where you cannot set environment variables which you could use for credentials or your code may be on a machine that does not have AWSCLI installed. In these situations you can keep environment variables with but not in your code.  The Python-Decouple package can help keep environment variables separate from your code.

### Install python-decouple

```bash 
$ pip install python-decouple
Collecting python-decouple
  Downloading python_decouple-3.6-py3-none-any.whl (9.9 kB)
Installing collected packages: python-decouple
Successfully installed python-decouple-3.6
$
```

### Using python-decouple with an AWS EC2 client class

First, create a .env file with your environment variables in it.

```bash
$ cat .env
my_aws_region='us-east-1'
my_aws_access_key_id='<abc>'
my_aws_secret_access_key='<xyz>'
$
```

Now you can use the config class to get values. 

```python
import boto3
from decouple import config

client = boto3.client('ec2', 
                       region_name=config('my_aws_region'), 
                       aws_access_key_id=config('my_aws_access_key_id'), 
                       aws_secret_access_key=config('my_aws_secret_access_key'))

# do something with client...
```

&nbsp;

Don't forget to exclude the .env file in your repo.

```bash
$ echo .env >>.gitignore
```

&nbsp;

### References:
- [pypi/python-decouple](https://pypi.org/project/python-decouple/)
