# aws-curl

`aws-curl` is like `curl` but with automatic SIGV4 signing to simplify calling
AWS services without requirement to have AWS CLI and Python to be installed.

The script is pure shell script designed for embedded and lightweight linux
distributions, docker images etc.

This utility also takes much less RAM than aws cli, so nano ec2 instances are
not dying with "Out of memory" when trying to download from s3 as aws cli does.

## Prerequisites

Dependencies:

- openssl (or libressl)
- curl
- GNU or similar coreutils (date/od/paste)
- GNU or similar sed

GNU coreutils/sed can be installed on macOS using Homebrew:
`brew install coreutils gsed`

On OpenWRT you might need to install at least `coreutils-od` and
`coreutils-paste` to get it working.

## Installation

```
curl -s https://raw.githubusercontent.com/sormy/aws-curl/master/aws-curl -o /usr/local/bin/aws-curl
chmod +x /usr/local/bin/aws-curl
```

## AWS credentials

Before you can use the application, you need to set environment variables with
AWS credentials.

Set AWS credentials and region using standard AWS CLI environment variables:

- `AWS_ACCESS_KEY_ID` - AWS public access key
- `AWS_SECRET_ACCESS_KEY` - AWS private secret key
- `AWS_SESSION_TOKEN` - temporary token received from STS or from EC2 metadata
- `AWS_DEFAULT_REGION` - AWS default region, in case if region is not provided
  in URL or as command line argument `--region`.

You can read more about AWS CLI environment variables here:
<https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html>

NOTE: Wrapper doesn't read configuration files generated by AWS CLI and reads
credentials only from environment variables.

## Usage

Application has the same syntax of command line arguments as `curl`, it just
automatically adds mandatory headers to make AWS to accept the request.

AWS provides documentation for all APIs, sometimes with explicit curl usage
examples. Find an API documentation you want to call and follow documentation to
construct valid command line request.

### Example 1: CloudWatchLogs.CreateLogGroup

Documentation:
<https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_CreateLogGroup.html>

Reference example http request:

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

The corresponding `aws-curl` request is:

```sh
aws-curl --request POST \
  --header "Content-Type: application/x-amz-json-1.1" \
  --header "x-amz-target: Logs_20140328.CreateLogGroup" \
  --header "Accept: application/json" \
  --data '{"logGroupName": "my-log-group"}' \
  "https://logs.us-east-1.amazonaws.com"
```

### Example 2: STS.GetCallerIdentity

Documentation:
<https://docs.aws.amazon.com/STS/latest/APIReference/API_GetCallerIdentity.html>

Reference example http request:

```
POST / HTTP/1.1
Host: sts.amazonaws.com
Accept-Encoding: identity
Content-Length: 32
Content-Type: application/x-www-form-urlencoded
Authorization: AWS4-HMAC-SHA256 Credential=AKIAI44QH8DHBEXAMPLE/20160126/us-east-1/sts/aws4_request,
        SignedHeaders=host;user-agent;x-amz-date,
        Signature=1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
X-Amz-Date: 20160126T215751Z
User-Agent: aws-cli/1.10.0 Python/2.7.3 Linux/3.13.0-76-generic botocore/1.3.22

Action=GetCallerIdentity&Version=2011-06-15
```

The corresponding `aws-curl` request is:

```sh
aws-curl --request POST \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data "Action=GetCallerIdentity" \
  --data "Version=2011-06-15" \
  --region "us-east-1" \
  "https://sts.amazonaws.com"
```

NOTE: Region can't be detected from URL, so it should be explicitly provided as
argument or as `AWS_DEFAULT_REGION` env variable.

NOTE: This API has xml response format by default, pass
`Accept: application/json` header to change response format.

### Example 3: S3

Download file from s3 to local file:

```sh
aws-curl --request GET \
  --output "my-file.txt" \
  --region "us-east-1" \
  "https://s3.amazonaws.com/my-bucket/my-file.txt"
```

Download file from s3 and print to stdout:

```sh
aws-curl --request GET \
  --region "us-east-1" \
  "https://s3.amazonaws.com/my-bucket/my-file.txt"
```

Upload local file to s3.

```sh
aws-curl --request PUT \
  --data "@my-file.txt" \
  --region "us-east-1" \
  "https://s3.amazonaws.com/my-bucket/my-file.txt"
```

Upload buffer to s3.

```sh
aws-curl --request PUT \
  --data "my content" \
  --region "us-east-1" \
  "https://s3.amazonaws.com/my-bucket/my-file.txt"
```

Delete file from s3:

```sh
aws-curl --request DELETE \
  --region "us-east-1" \
  "https://s3.amazonaws.com/sormy/test.txt"
```

## Example 4: EC2

Create AMI image:

```sh
aws-curl --request POST \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data "Action=CreateImage" \
  --data "Version=2016-11-15" \
  --data "InstanceId=i-something" \
  --data "Name=My Image Name" \
  --data "Description=My Image Description" \
  --data "NoReboot=true" \
  --data "BlockDeviceMapping.1.DeviceName=/dev/xvdb" \
  --data "BlockDeviceMapping.1.NoDevice=1" \
  --data "DryRun=true" \
  --region "us-east-1" \
  "https://ec2.amazonaws.com"
```

Terminate EC2 instance:

```sh
aws-curl --request POST \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data "Action=TerminateInstances" \
  --data "Version=2016-11-15" \
  --data "InstanceId.1=i-something" \
  --data "DryRun=true" \
  --region "us-east-1" \
  "https://ec2.amazonaws.com"
```

### Command line arguments

`aws-curl` is very thin wrapper around curl. Most of options are passed as it is
to `curl` with some exceptions (see below).

Wrapper recognizes these `curl` arguments:

- `--url` as explicit request URL (but can be provided only once)
- `https://*.amazonaws.com` somewhere in argument list as request URL
- Last argument as `url` if not explicitly provided using `--url` and if there
  is no argument that looks like AWS service endpoint `https://*.amazonaws.com`
- `-X` | `--request` as request METHOD (`GET` is default)
- `-H` | `--header` as request header
- `-d` | `--data` as request body (but passed to curl as `--data-binary`)
- `-V` | `--version` - shows version of `aws-curl` and `curl`
- `-h` | `--help` - shows help for `aws-curl` and `curl`
- `-v` | `--verbose` - dumps debug details (in addition to `curl` debug details)

Wrapper recognizes these non-curl arguments:

- `--service` - AWS service name, if can't be automatically detected from host
- `--region` - AWS region name, if can't be automatically detected from host or
  if not explicitly provided in `AWS_DEFAULT_REGION` environment variable
- `--ec2-creds` - use attached to EC2 credentials (instance role)

### Response format

APIs for different services have different default response format. Sometimes it
is json, sometimes xml. For most APIs you could enforce json output format by
adding header `Accept: application/json` and xml output format by adding header
`Accept: application/xml`.

## Automatically computed headers

These headers are automatically handled by `aws-curl`:

- `Host` - `curl` computes it automatically based on URL
- `x-amz-date` - generated based on current date/time
- `x-amz-content-sha256` - added automatically if request is to s3 service
- `x-amz-security-token` - added automatically if corresponding env variable is
  set
- `Authorization` - automaticaly generated based on request and AWS credentials
- `Content-Length` - `curl` computes it automatically
- `Connection` - `curl` inserts it if needed

Some headers provided by default by curl are unset by aws-curl:

- `User-Agent` - doesn't have any impact on response
- `Accept` - optional, but can be used to enforce response format (xml or json)
- `Content-Type` - can be required or not depending on API

## Service/region autodetection

Wrapper tries to auto detect AWS region name and AWS service name from service
endpoint host.

Assumed format for endpoint is `https://<service>.<region>.<domain>` or
`https://<service>.<domain>`, for example `https://logs.us-east-1.amazonaws.com`
and `https://sts.amazonaws.com`.

Read more about AWS service endpoints:
<https://docs.aws.amazon.com/general/latest/gr/rande.html>

Wrapper implicitly adds headers `authorization`, `x-amz-date`,
`x-amz-content-sha256` and `x-amz-security-token` (if needed), so don't need to
pass them explicitly.

## How to get know syntax for API

### AWS documentation

Take a look on AWS documentation, in most cases it has explicit http request
example.

The documentation is available here: <https://docs.aws.amazon.com>

### AWS CLI

Download and install AWS CLI: <https://aws.amazon.com/cli/>

Run the same command using aws cli with `--debug`. Take a look how AWS CLI is
performing the request, what body is used, what method is used, what url, what
region and what headers. Take a look on `Content-Type`, it impacts on how
request body should be encoded.

You can get more details by exploring service json metadata file. AWS CLI with
`--debug` dumps what service file is loaded for specific command. You can
explore it and find what body format is, what are all available options and what
is the result.

## EC2 attached role

This repo includes `ec2-import-creds` that can import attached credentials
including access key, secret key, session token and region.

Just import from the shell as `source ec2-import-creds`.

Or you can use `--ec2-creds` options of `aws-cli` to get the same effect, but
importing credentials once in beginning is faster than importing for every
`aws-curl` invocation.

## Platforms

The script has been tested on bash in posix mode on macOS and Linux. It should
work on other shells and OS as well. If not, please cut a ticket.

## License

MIT
