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
  - `lambda_scheduler_timer`: calls lambda_scheduler_tick on a regular interval. The faster this runs, the higher the resolution of our job scheduling.

1. SNS topic for executions
  - `lambda_scheduler_invocations`: listens for messages and dispatches to lambda_scheduler_invoke

1. Lambda jobs
  - `lambda_scheduler_tick` (event): determines what jobs need to be run on each tick, sends SNS message to run them with payload and job name

Pseudocode:
```
  retrieve all jobs from lambda_scheduler_event table
  for each job:
    check for last event

    if no last event:
      run event now

    if schedule + last event = event due:
      run event now

```

  - `lambda_scheduler_invoke` (function name, payload): host job to execute arbitrary Python with payload, mark job as executed in DynamoDB table

  Pseudocode:
  ```
    remove job details and payload from SNS event
    execute python method with payload
    capture any errors and log them
    re-throw error or publish it to SNS (optional)
  ```

## How to deploy

Use the CF template to install.

## How to configure

Use the CLI to configure jobs, schedules, and payloads.

## Frequently asked questions

1. _What scheduling algorithms are supported?_ Currently rate (eg. every 5 minutes) and periodic (eg. every 5th minute after the hour). Note that both of these will immediately execute all jobs if they have never been run, in contrast with typical scheduling implementations like crontab.

1. _How does this project handle different regions?_ We assume everything, including all possible jobs, are deployed in the same region as the caller, SNS topic, CloudWatch rule, etc.

1. _How does this project handle dynamically loading of code?_ We assume all Python methods that might be called on an interval are included in the zip file uploaded to Lambda. If the `lambda_scheduler_invoke` job can't see your Python code, it won't be invoked. We *do not* support invoking a separately-configured Lambda package from the API.

1. _Why don't you use AWS CloudWatch Events rules?_ See the background section for why. We wanted to provide more features than cloudwatch events provides.

1. _Are there other similar projects out there?_ Yes, check out [neyric/lambda-schedule](https://github.com/neyric/lambda-schedule), [paulmelnikow/node-aws-lambda-scheduler](https://github.com/paulmelnikow/node-aws-lambda-scheduler), and [milindparikh/diksha](https://github.com/milindparikh/diksha).

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
