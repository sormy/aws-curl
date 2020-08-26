# aws-curl

`aws-curl` is like `curl` but with automatic SIGV4 signing to simplify calling
AWS services without requirement to have Python to be installed.

## Prerequisites

- openssl (or libressl)
- curl
- coreutils

On macOS: `brew install coreutils`

## Installation

```
curl -s https://raw.githubusercontent.com/sormy/aws-curl/master/aws-curl -o /usr/local/bin/aws-curl
chmod +x /usr/local/bin/aws-curl
```

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

Another example, for API that requires POST with url encoded body:

```sh
aws-curl -X POST \
  -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
  -d 'Action=GetCallerIdentity&Version=2011-06-15' \
  https://sts.amazonaws.com
```

## How to get know syntax for API

1. Take a look on AWS documentation, sometimes it has explicit http example.

2. Run the same command using aws cli with `--debug`. Take a look how aws cli
   is performing the request, what body is used, what method is used, what
   url, what region and what headers. Take a look on `Content-Type`, it impacts
   on how request body should be encoded.

3. You can get more details by exploring service json metadata file. aws cli
   with `--debug` dumps what service file is loaded for specific command.
   You can explore it and find what body format is, what are all available
   options and what is the result.

## How it works

This is very thin wrapper around curl. Most of options are passed as it is to
curl with some exceptions (see below).

Wrapper recognizes these curl arguments:

- Last argument as `url` or service endpoint
- `-X` | `--request` as request METHOD (`GET` is default)
- `-H` | `--header` as request header
- `-d` | `--data` as request body

Wrapper recognizes these non-curl arguments:

- `--service` - AWS service name if can't be automatically detected from host
- `--region` - AWS region name if can't be automatically detected from host
  or if not provided in `AWS_DEFAULT_REGION` environment variable

Wrapper tries to auto detect AWS region name and AWS service name from service
endpoint host.

Assumed format for endpoint is `https://<service>.<region>.<domain>`
or `https://<service>.<domain>`, for example
`https://logs.us-east-1.amazonaws.com` and `https://sts.amazonaws.com`.

Wrapper implicitly adds headers `authorization`, `x-amz-date` and
`x-amz-security-token` (if needed), so don't need to pass them explicitly.

Wrapper reads these environment variables (as AWS credentials):

- `AWS_ACCESS_KEY_ID` - AWS public access key
- `AWS_SECRET_ACCESS_KEY` - AWS private secret key
- `AWS_SESSION_TOKEN` - temporary token received from STS or from EC2 metadata
- `AWS_DEFAULT_REGION` - in case if region is not provided in URL

## EC2 attached role

This repo includes `ec2-import-creds` that can import attached credentials
including access key, secret key, session token and region.

Just import from the shell as `source ec2-import-creds`.

## Platforms

Verified to work on Linux and macOS.

## License

MIT
