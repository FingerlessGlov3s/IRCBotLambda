# IRCBotLambda
Lambda and AWS API Gateway to get around Cloudflare issues on my host hosting SecurityFeed bot, getting RSS feeds.

Create Python3.8 Lambda Function, add python layer "arn:aws:lambda:eu-west-2:142628438157:layer:AWSLambda-Python-AWS-SDK:4".

Set Python runtime settings hander to "lambda_function.main_handler".

Python Code Example
```
from botocore.vendored import requests
import json

def main_handler(event, context):
   s = requests.Session()
   headers = {
      'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0'
   }
   response = s.get("https://linuxsecurity.com/linuxsecurity_advisories.xml", timeout=10,headers=headers)
   if response.status_code is not 200:
      return "error - non 200 HTTP code"
   return response.content
```

Save the lambda, we're now ready for the API Gateway.

Create a AWS API Gateway, create a GET resource which runs the lambda function we just created.

Go to Resource Policy, so we can whitelist the server that will call this API
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "arn:aws:execute-api:eu-west-2:926649409264:gcsdtsc4zg/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "51.xx.xx.xx"
                }
            }
        }
    ]
}
```

Go to Stages and create a stage named "json" or something like this, don't forget to enable throttling and set it to something sensible like 1 request per second.

Then it'll give you the invoke URL you may can use in the IRC bot configuration to get the json feed.
