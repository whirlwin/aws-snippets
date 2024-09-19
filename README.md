# aws-snippets
Useful collection of scripts for working with AWS through the AWS CLI

## Searching through WAF logs
If you prefer to easily get and query WAF logs, you can use `jq`, instead of configuring Athena which is difficult, expensive and error-prone.

```shell
Set all required variables in the shell, adjust accordingly.
accountId=123456789
albName=myAlb
filterDate=2024/09/13
wafRule=waf_rule_123
region=eu-north-1

First download all WAF logs
aws s3 cp s3://${myAlb}/AWSLogs/${accountId}/WAFLogs/${region}/${waf_rule_123}/${filterDate} . --recursive

# Clean up from previous runs, if any
rm -rf ./**/*.log

# Uncompress the log files
gunzip ./**/*.gz

# Select all blocked requests
cat ./**/*.log | jq -c '. | select(."action" == "BLOCK")' | jq '.'

# You can tailor the above query further to narrow down or filter for logs using jq or other tools.

# E.g. showing the number of requests for a given URL path:
cat ./**/*.log | jq -c '. | select(."action" == "BLOCK")' | jq '.httpRequest.uri' | sort | uniq -c

# E.g. showing blocked requests from a particular country with a specific URL path:
cat ./*/.log | jq -c '. | select(."action" == "BLOCK")' | jq 'select(."httpRequest".country == "US")' | jq 'select(.httpRequest.uri == "/login/v-1")' | less
```
