# Splitting SQS message via EB Rules and EB Pipes - WIP

Based on [this article](https://medium.com/@pubudusj/split-messages-from-single-sqs-queue-into-multiple-sqs-queues-using-eventbridge-728809342352).

## Learnings

- SQS is an old service and, just like STS, responds with XML.

  - This is very vital to keep in mind as the direct integration between APIGW, and SQS requires you to set correct headers.

  - The mapping template is also different from services working with JSON.

- TIL that for `AWS` or `AWS_PROXY` integrations, the `Uri` always starts with `arn:aws:apigateway`.

- Regarding the two different _integration types_.

  - **The `AWS_PROXY` is for AWS Lambda** – the APIGW will forward the request to AWS Lambda.

    - APIGW will **ignore any mapping templates for that integration**.

  - **The `AWS` is for any AWS service (could also be AWS Lambda)**.

    - This is where you must set the mapping templates and define extra parameters.

    - Also works with AWS Lambda, but you most likely want to use `AWS_PROXY` with that service (why bother with extra transforms?).

- Is it possible to only specify the permissions on the SQS level (resource) rather than to create an IAM role?

  - **It is possible, but the options to 'scope down' the resource are limited**.

    - Everywhere I look, the permission is set on APIGW execution role and not on the SQS level. **I think that having permission on the resource is better, as if you delete the resource, you also delete the permission**.

      - I was able to make it work by **specifying the ARN of the APIGW role**. I could not set the ARN of the method as CloudFormation would reject such format.

    - Side note: having the SQS policy be a separate resource makes my life much easier. Without it, I would have first to deploy the SQS queue and then be able to provide the permission.

      - Creating the `AWS::SQS::QueuePolicy` took a lot of time.

  - Like in the case of APIGW <-> SQS integration, SQS <-> Pipes also favours having the IAM permissions on the Pipe role, rather than the queue.

- YAML is a weird language. With most of the values, I can omit the quotes, but **for some, like the `"'application/x-www-form-urlencoded'"` I had to use double quotes.

  - Maybe it is a quirk of the service?

- The **Pipes DLQ configuration is a bit weird**.

  - The documentation mentions different behaviors related to the DLQ based on the source.

    > A pipe inherits dead-letter queue (DLQ) behavior from the source. If the source Amazon SQS queue has a configured DLQ, messages are automatically delivered there after the configured number of attempts. For stream sources, such as DynamoDB and Kinesis streams, you can configure a DLQ for the pipe and route events.

  - [The CFN documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-pipes-pipe.html) does not reference the `DeadLetterConfig` property, which **is also visible in the documentation (left panel)**.

    - I [found an article that shows how to configure it for DynamoDB streams](https://awstip.com/is-eventbridge-pipes-the-missing-piece-in-your-event-driven-puzzle-part-ii-a06fdddfb0fa). Weird.

- I find it a bit weird that one could specify the `Arn` of the role for EB on the _bus-resource_ level and also on the _target-resource_ level.

  - For sending the data to CW Logs, I had to specify it at the _bus-resource_ level. Weird

- I **always forget that you must name your CW log group with a prefix of `/aws/events` for the EventBridge to deliver logs to that CW log group**.

  - I have no idea why. I'm configuring this setup at least for the fourth time now, and every time I forget and every time I never find anything in the docs.
