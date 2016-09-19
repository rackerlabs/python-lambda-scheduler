# python-lambda-scheduler
Flexible scheduling of AWS Lambda functions with arbitrary payloads

## Background

While [CloudWatch Events rules](https://aws.amazon.com/blogs/aws/new-cloudwatch-events-track-and-respond-to-changes-to-your-aws-resources/) do support rate and crontab schedule types, we
wanted a few additional features, such as:
- support additional schedule types (e.g. deadline, non-periodic)
- exceed the AWS [CloudWatch Events limits](http://docs.aws.amazon.com/AmazonCloudWatch/latest/events/cloudwatch_limits_cwe.html)
- single AWS CloudWatch event rule to tick scheduler
- immediate execution for crontab (rate) expressions that have never run (e.g. for EC2 snapshots)

## Frequently asked questions

1. _Why don't you use AWS CloudWatch Events rules?_ See the background section for why. We wanted to provide more features than cloudwatch events provides. This also opens the door to [more interesting scheduling algorithms](https://en.wikipedia.org/wiki/Scheduling_%28computing%29#Scheduling_disciplines).

2. ...

## License

Copyright 2016 Rackspace Hosting

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
