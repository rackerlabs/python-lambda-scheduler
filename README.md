# python_lambda_scheduler
Flexible scheduling of AWS Lambda functions with arbitrary payloads

## Background

While [CloudWatch Events rules](https://aws.amazon.com/blogs/aws/new-cloudwatch-events-track-and-respond-to-changes-to-your-aws-resources/) do support rate and crontab schedule types, we
wanted a few additional features, such as:
- support [additional schedule types](https://en.wikipedia.org/wiki/Scheduling_%28computing%29#Scheduling_disciplines) (e.g. deadline, non-periodic)
- exceed the AWS [CloudWatch Events limits](http://docs.aws.amazon.com/AmazonCloudWatch/latest/events/cloudwatch_limits_cwe.html)
- single AWS CloudWatch event rule instead of one per schedule
- immediate execution for crontab (rate) expressions that have never run (e.g. for EC2 snapshots)

## Design

1. DynamoDB table
   - `lambda_scheduler_event`: Key(function name, schedule, payload), last_run

1. CloudWatch Event rule
  - `lambda_scheduler_timer`: calls lambda_scheduler_tick on a regular interval

1. SNS topic for executions
  - `lambda_scheduler_invocations`: listens for messages and dispatches to lambda_scheduler_invoke

1. Lambda jobs
  - `lambda_scheduler_tick` (event): determines what jobs need to be run on each tick, sends SNS message to run them with payload and job name
  - `lambda_scheduler_invoke` (function name, payload): host job to execute arbitrary Python with payload, mark job as executed in DynamoDB table

## How to deploy

Use the CF template to install.

## How to configure

Use the CLI to configure jobs, schedules, and payloads.

## Frequently asked questions

1. _Why don't you use AWS CloudWatch Events rules?_ See the background section for why. We wanted to provide more features than cloudwatch events provides.

2. _Are there other similar projects out there?_ Yes, check out [neyric/lambda-schedule](https://github.com/neyric/lambda-schedule), [paulmelnikow/node-aws-lambda-scheduler](https://github.com/paulmelnikow/node-aws-lambda-scheduler), and [milindparikh/diksha](https://github.com/milindparikh/diksha).


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
