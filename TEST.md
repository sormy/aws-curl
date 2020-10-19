Below is the list of basic integ tests:

```shell
export AWS_ACCESS_KEY_ID=$(aws configure get default.aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get default.aws_secret_access_key)

aws-curl --request POST \
  --header "Content-Type: application/x-amz-json-1.1" \
  --header "x-amz-target: Logs_20140328.CreateLogGroup" \
  --header "Accept: application/json" \
  --data '{"logGroupName": "my-log-group"}' \
  "https://logs.us-east-1.amazonaws.com"

aws-curl --request POST \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data "Action=GetCallerIdentity&Version=2011-06-15" \
  --region "us-east-1" \
  "https://sts.amazonaws.com"

aws-curl --request PUT \
  --data "test" \
  --region us-east-1 \
  "https://s3.amazonaws.com/sormy/test.txt"

echo "test" > test.txt
aws-curl --request PUT \
  --data "@test.txt" \
  --region us-east-1 \
  "https://s3.amazonaws.com/sormy/test.txt"

aws-curl --request GET \
  --region us-east-1 \
  "https://s3.amazonaws.com/sormy/test.txt"

aws-curl --request DELETE \
  --region us-east-1 \
  "https://s3.amazonaws.com/sormy/test.txt"
```
