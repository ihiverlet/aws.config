# aws.config

To configure only one endpoint : use env variable
Configure several s3 endpoints : create .aws/config and .aws/credentials files. One for the configuration, and the other for sensitive information.
To change the default path of these files : `AWS_CONFIG_FILE` or alike.
By default, the default profile will be used. To use another one, most of the time, one can use `AWS_PROFILE` env variable.

The config files will look like : 
`.aws/config`
```
[default]
region = us-east-1
output = json
endpoint_url = https://my-minio

[profile parseable]
region = us-east-1
output = json
endpoint_url = https://my-minio

```

Il existe une variante - todo: creuser les services
```
s3 =
    endpoint_url = https://my-minio.
```
The following can be important to make minio work. However it does not seem necessary.
```
config=Config(signature_version='s3v4')  # Important for MinIO
```


`.aws/credentials`
```
[default]
aws_access_key_id = default_access_key
aws_secret_access_key = default_secret_access_key
aws_session_token = default_token

[parseable]
aws_access_key_id = parseable_access_key
aws_secret_access_key = parseable_secret_acess_key
```

## Python

### boto3

```python
import boto3
from botocore.config import Config

# Load the profile if different from default
session = boto3.Session(profile_name='parseable')
s3_client = session.client(service_name='s3')

# Example: List objects in the bucket
response = s3_client.list_objects_v2(Bucket='inesh')
response

```

### s3fs

```
import s3fs

# Connect using default profile
fs = s3fs.S3FileSystem(
        client_kwargs={
            'endpoint_url': 'https://minio.lab.sspcloud.fr'
        }
)

fs = s3fs.S3FileSystem(profile="parseable")


# List files in a bucket
files = fs.ls('inesh')
print(files)

```
### pandas / arrow

Same as duckdb, need to wait for aws-sdk-cpp

```python
import pandas as pd
import os

os.environ["AWS_DEFAULT_PROFILE"] = "default"
os.environ["AWS_PROFILE"] = "default"  # one or the other ?

os.environ["AWS_ENDPOINT_URL"] = "https://minio.lab.sspcloud.fr"

df = pd.read_parquet('s3://inesh/demo/fd-logemt-2020.parquet', engine='pyarrow')

```

Workaround : specify storage option :

```
df = pd.read_csv("s3://inesh/demo/airports_fr.csv", storage_options=dict(profile='default'))

```

## R 

### paws
works fine

### aws.s3
too old, won't recognize endpoints and region

# duckdb cli

Need to wait for issue https://github.com/aws/aws-sdk-cpp/issues/2587 so that the endpoint could be recognize automatically.

```
CREATE OR REPLACE SECRET secret (
    TYPE s3,
    PROVIDER credential_chain,
    CHAIN config,
    PROFILE 'default',
    ENDPOINT "my-minio"
);
```

# Spark ?
