# aws-curl

`aws-curl` is like `curl` but with automatic SIGV4 signing to simplify calling
AWS services without requirement to have Python to be installed.

## Usage

Set AWS credentials as environment variables.

Find an API you want to call, for example CloudWatch Logs CreateLogGroup:
<https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_CreateLogGroup.html>

Documentation shows example request:

```
POST / HTTP/1.1
Host: logs.<region>.<domain>
X-Amz-Date: <DATE>
Authorization: AWS4-HMAC-SHA256 Credential=<Credential>, SignedHeaders=content-type;date;host;user-agent;x-amz-date;x-amz-target;x-amzn-requestid, Signature=<Signature>
User-Agent: <UserAgentString>
Accept: application/json
Content-Type: application/x-amz-json-1.1
Content-Length: <PayloadSizeBytes>
Connection: Keep-Alive
X-Amz-Target: Logs_20140328.CreateLogGroup
{
  "logGroupName": "my-log-group"
}
```

The `aws-curl` example to perform this request is:

```sh
aws-curl -X POST \
  -H "Content-Type: application/x-amz-json-1.1" \
  -H "x-amz-target: Logs_20140328.CreateLogGroup" \
  -d '{"logGroupName": "my-log-group"}' \
  "https://logs.us-east-1.amazonaws.com"
```

## How it works

This is very thin wrapper around curl. Most of options are passed as it is to
curl with some exceptions (see below).

Wrapper recognizes these curl arguments:

- Last argument as `url` or service endpoint
- `-X` | `--request` as request METHOD (`GET` is default)
- `-H` | `--header` as request header
- `-d` | `--data` as request body

Wrapper extracts AWS region name and AWS service name from service endpoint.

Assumed format for endpoint is `https://<service>.<region>.<domain>`,
for example `https://logs.us-east-1.amazonaws.com`.

Wrapper implicitly adds headers `authorization`, `x-amz-date` and
`x-amz-security-token` (if needed), so don't need to pass them explicitly.

## Custom credentials

Wrapper reads these environment variables (as AWS credentials):

- `AWS_ACCESS_KEY_ID` - AWS public access key
- `AWS_SECRET_ACCESS_KEY` - AWS private secret key
- `AWS_SESSION_TOKEN` - temporary token received from STS or from EC2 metadata

## EC2 attached role

Here is an example how to fetch credentials for attached to EC2 instance role:

```sh
# update role name to the one that is used on EC2 instance
role_name="cloudwatch-logs-all"
credentials="$(curl -m 1 -s http://169.254.169.254/latest/meta-data/iam/security-credentials/$role_name)"

get_key_value() {
  echo "$1" | grep "$2" | cut -d ':' -f 2 | cut -d '"' -f 2
}

if [ -n "$credentials" ]; then
    export AWS_ACCESS_KEY_ID=$(get_key_value "$credentials" "AccessKeyId")
    export AWS_SECRET_ACCESS_KEY=$(get_key_value "$credentials" "SecretAccessKey")
    export AWS_SESSION_TOKEN=$(get_key_value "$credentials" "Token")
fi
```

## Dependencies

- openssl (or libressl)
- curl
- coreutils

## Platforms

Verified to work on Linux and macOS.

## License

MIT
