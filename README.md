# SAM HelloWorld Application 

## Requirements

* AWS CLI already configured with at least PowerUser permission
* [Java SE Development Kit 8 installed](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* [Docker installed](https://www.docker.com/community-edition)
* [Maven](https://maven.apache.org/install.html)
* [SAM CLI](https://github.com/awslabs/aws-sam-cli)
* [Python 3](https://docs.python.org/3/)

## Setup process

### Installing dependencies

We use `maven` to install our dependencies and package our application into a JAR file:

```bash
mvn package
```

### Local development

**Invoking function locally through local API Gateway**
1. Start DynamoDB Local in a Docker container. `docker run -p 8000:8000 amazon/dynamodb-local`
2. Start the SAM local API.
 - On a Mac/Linux: `sam local start-api`.

## Packaging and deployment

Firstly, we need a `S3 bucket` where we can upload our Lambda functions packaged as ZIP before we
deploy anything - If you don't have a S3 bucket to store code artifacts then this is a good time to
create one:

```bash
export BUCKET_NAME=my_cool_new_bucket
aws s3 mb s3://$BUCKET_NAME
```

Next, run the following command to package our Lambda function to S3:

```bash
sam package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket $BUCKET_NAME
```

Next, the following command will create a Cloudformation Stack and deploy your SAM resources.

```bash
sam deploy \
    --template-file packaged.yaml \
    --stack-name $Stack_name \
    --capabilities CAPABILITY_IAM
```

> **See [Serverless Application Model (SAM) HOWTO Guide](https://github.com/awslabs/serverless-application-model/blob/master/HOWTO.md) for more details in how to get started.**

After deployment is complete you can run the following command to retrieve the API Gateway Endpoint URL:

```bash
aws cloudformation describe-stacks \
    --stack-name sam-orderHandler \
    --query 'Stacks[].Outputs'
```

## Testing

### Running unit tests
We use `JUnit` for testing our code.
Unit tests in this sample package mock out the DynamoDBTableMapper class for Order objects.
Unit tests do not require connectivity to a DynamoDB endpoint. You can run unit tests with the
following command:

```bash
mvn test
```

### Running integration tests
Integration tests in this sample package do not mock out the DynamoDBTableMapper and use a real
AmazonDynamoDB client instance. Integration tests require connectivity to a DynamoDB endpoint, and
as such the POM starts DynamoDB Local from the Dockerhub repository for integration tests.

```bash
mvn verify
```

### Running end to end tests through the SAM CLI Local endpoint
Running the following end-to-end tests requires Python 3 and the `requests` pip
package to be installed. For these tests to succeed,
```bash
pip3 install requests
python3 src/test/resources/api_tests.py 3
```

The number that follows the test script name is the number of orders to create in the
test. For these tests to work, you must start DynamoDB Local (`docker run -p 8000:8000 amazon/dynamodb-local`)
and then start SAM Local with the appropriate CLI command (for example,
`sam local start-api --env-vars src/test/resources/test_environment_mac.json`).

# Appendix

## AWS CLI commands

AWS CLI commands to package, deploy and describe outputs defined within the cloudformation stack:

```bash
sam package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME

sam deploy \
    --template-file packaged.yaml \
    --stack-name sam-orderHandler \
    --capabilities CAPABILITY_IAM \
    --parameter-overrides MyParameterSample=MySampleValue

aws cloudformation describe-stacks \
    --stack-name sam-orderHandler --query 'Stacks[].Outputs'
```

## Bringing to the next level

Next, you can use the following resources to know more about beyond hello world samples and how others
structure their Serverless applications:

* [AWS Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/)
