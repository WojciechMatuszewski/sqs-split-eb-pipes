# Splitting SQS message via EB Rules and EB Pipes - WIP

Based on [this article](https://medium.com/@pubudusj/split-messages-from-single-sqs-queue-into-multiple-sqs-queues-using-eventbridge-728809342352).

## Learnings

- SQS is an old service and, just like STS, responds with XML.

  - This is very vital to keep in mind as the direct integration between APIGW and SQS requires you to set correct headers.

  - The mapping template is also different from services working with JSON.

- TIL that for `AWS` or `AWS_PROXY` integrations, the `Uri` always starts with `arn:aws:apigateway`.

- Regarding the two different _integration types_.

  - **The `AWS_PROXY` is for AWS Lambda** – the APIGW will forward the request to AWS Lambda.
    - APIGW will **ignore any mapping templates for that integration**.

  - **The `AWS` is for any AWS service (could also be AWS Lambda)**.
    - This is where you must set the mapping templates and define extra parameters.
    - Also works with AWS Lambda, but you most likely want to use `AWS_PROXY` with that service (why bother with extra transforms?).

- Is it possible to only specify the permissions on the SQS level (resource), rather than to create an IAM role?
  - I think it is a best practice to have the permission as close to the resource as possible
