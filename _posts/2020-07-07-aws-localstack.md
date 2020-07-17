---
title: "Local development with AWS on LocalStack"
categories: [craft]
date: 2020-06-30 06:00:00 +1100
modified: 2020-06-30 06:00:00 +1100
author: pratikdas
excerpt: "Develop and test your applications using AWS services locally with LocalStack."
image:
  auto: 0072-aws
---
In the early days of development using AWS, we prefer to focus on writing application code instead of spending time on setting up the environment for accessing AWS services. While building our applications, we access various AWS services  - for example, upload files in S3, store some data in DynamoDB, send messages to SQS, write event handlers with lambda functions, etc.  

**Setting up a development environment for using these services requires creating different infrastructure resources coupled with giving the right level of access following the principles of least privilege.** To avoid getting bogged down by these mundane tasks, we use [LocalStack](https://github.com/localstack/localstack) to develop and unit test our applications with mock implementations of these services. 

Simply put, LocalStack is an open-source test framework for developing applications with mock AWS services. It provides a testing environment on our local machine with the same APIs as real AWS. We switch to the real AWS services only in the integration environment and beyond.

{% include github-project.html url="https://github.com/thombergs/code-examples//tree/master/aws/localstack" %}

## Why Use LocalStack?
The method of temporarily using dummy (or mock, fake, proxy) objects in place of actual ones is a time-tested way of running unit tests for applications having external dependencies. Most appropriately, these dummies are called [test doubles](http://xunitpatterns.com/Test%20Double.html). 

Some methods to create test doubles are:

1. Manual - Creating and using mock objects with frameworks like Mockito during unit testing.
2. DIY (Do It Yourself) - Using a homegrown solution deployed as a service running in a separate process, or embedded in our code.

LocalStack gives a good alternative to both these approaches.**We will implement test doubles of our AWS services with LocalStack.** With LocalStack, we can:

1. run our applications without connecting to AWS.
2. avoid the complexity of AWS configuration and focus on development.
3. run tests in our CI/CD pipeline. 
4. configure and test error scenarios.

## How To Use LocalStack 

**LocalStack is a Python application designed to run as an HTTP request processor while listening on specific ports.** 


Our usage of LocalStack is centered around two tasks:  

1. Running LocalStack.
2. Overriding the AWS endpoint URL with the URL of LocalStack.

### Running LocalStack

LocalStack is run inside a Docker container. We run LocalStack either as a Python application using our local Python installation or by directly using it's Docker image. 

#### Running LocalStack With Python

We first install the LocalStack package using pip:

```
pip install localstack
```
We then start localstack with the "start" command as shown below:
```
localstack start
```
This will start LocalStack inside a Docker container.

#### Run LocalStack With Docker
We can also run LocalStack directly as a Docker image either with the Docker run command or docker-compose.

For running with docker, we first pull the image from the docker hub and use the docker run command. We need to allocate a minimum of 4GB  memory for the container.

We start LocalStack with docker-compose using the command:
```
TMPDIR=/private$TMPDIR docker-compose up
```
The part `TMPDIR=/private$TMPDIR` is required only in MacOS. We take the base version of `docker-compose.yml` from the [github repository of LocalStack](https://github.com/localstack/localstack/blob/master/docker-compose.yml) and customize it as shown in the next section or run it without changes if we prefer to use the default configuration.

#### Customize LocalStack
The default behavior of LocalStack is to spin up all the [core services](https://github.com/localstack/localstack#overview) with each of them listening on port 4566. We can override this behavior of LocalStack by setting a few environment variables. The default port 4566 can be overridden by setting the environment variable EDGE_PORT. We can also configure LocalStack to spin a limited set of services by setting a comma-separated list of service names as value for the environment variable SERVICES as shown below:

```
version: '2.1'

services:
  localstack:
    container_name: "${LOCALSTACK_DOCKER_NAME-localstack_main}"
    image: localstack/localstack
    ports:
      - "4566-4599:4566-4599"
      - "${PORT_WEB_UI-8080}:${PORT_WEB_UI-8080}"
    environment:
      - SERVICES=s3,dynamodb

```

In this `docker-compose.yml`, we set the environment variable `SERVICES` to the name of the services we want to use in our application (`S3` and `DynamoDB`). There is a quirky way to set the commonly used services used for serverless applications by setting SERVICES environment variable to "serverless".


#### Connect With LocalStack By Overriding AWS Endpoints
We access AWS services from the CLI or our applications using the AWS SDK (Software Development Kit).

The AWS SDK and CLI are an integral part of our toolset for building applications with AWS services. The SDK provides client libraries in all the popular programming languages like Java, Node js, or Python for accessing various AWS services. 

Both the AWS SDK and the CLI provide an option of overriding the URL of the AWS API. We usually use this to specify the URL of our proxy server when connecting to AWS services from behind a corporate proxy server. We will use this same feature in our local environment for connecting to LocalStack.

We do this in the AWS CLI using commands like this:
```
aws --endpointurl http://localhost:4956 kinesis list-streams
```

Executing this command will send the requests to the URL of LocalStack specified as the value of the endpoint URL command line parameter (localhost on port 4956) instead of the real AWS endpoint.

We adopt a similar approach when using the SDK.
 
```java
URI endpointOverride = new URI("http://localhost:4566");
S3Client s3 = S3Client.builder()
  .endpointOverride(endpointOverride )  // Overriding the endpoint
  .region(region)
  .build();
```

Here, we have overridden the AWS endpoint of S3 by providing the value of the URL of LocalStack as the value of the endpointOverride method in the [S3ClientBuilder](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/s3/S3ClientBuilder.html) class.



## Common Usage Patterns 

#### Running CLI Commands Against LocalStack

**We start by creating a fake profile in the AWS CLI** so that we can later use the AWS CLI for invoking the services provided by LocalStack:

```
aws configure --profile localstack
```

Here we create a profile named `localstack`. Any other name can be provided. 
Provide any value you like for AWS Access Key and Secret Access Key and a valid AWS region like `us-east-1`, but do not leave them blank. 

Unlike AWS, LocalStack does not validate these credentials but complains if no profile is set. So far, it is just like any other AWS profile which we will use to work with LocalStack.

With our profile created, **we proceed to execute the AWS CLI commands** by passing an additional parameter for overriding the endpoint URL:
```
aws s3 --endpoint-url http://localhost:4566 create-bucket io.pratik.mybucket
```
**This command created an S3 bucket in LocalStack.**

We can also execute a regular CloudFormation template that describes multiple AWS resources:

```
aws cloudformation create-stack \
  --endpoint-url http://localhost:4566 \
  --stack-name samplestack \
  --template-body file://sample.yaml \
  --profile localstack
```

#### Running JUnit Tests Against LocalStack
If we want to run tests against the AWS APIs, we can do this from within a JUnit test.

**At the start of a test, we start LocalStack as a Docker container on a random port and after all tests have finished execution we stop it again:** 

```java
@ExtendWith(LocalstackDockerExtension.class)
@LocalstackDockerProperties(services = { "s3", "sqs" })
class AwsServiceClientTest {

    private static final Logger logger = Logger.getLogger(AwsServiceClient.class.getName());

    private static final Region region = Region.US_EAST_1;
    private static final String bucketName = "io.pratik.poc";

    private AwsServiceClient awsServiceClient = null;

    @BeforeEach
    void setUp() throws Exception {
        String endpoint = Localstack.INSTANCE.getEndpointS3();
        awsServiceClient = new AwsServiceClient(endpoint);
        createBucket();
    }

    @Test
    void testStoreInS3() {
        logger.info("Executing test...");
        awsServiceClient.storeInS3("image1");
        assertTrue(keyExistsInBucket(), "Object created");
    }
```

The code snippet is a JUnit Jupiter test used to test a java class to store an object in an S3 bucket.

#### Configuring a Spring Boot Application using [S3](https://aws.amazon.com/s3/) and [DynamoDB](https://aws.amazon.com/dynamodb/) to use LocalStack

We will create a simple customer registration application using the popular [Spring Boot](https://spring.io/projects/spring-boot) framework. Our application will have an API that will take a first name, last name, email, mobile, and a profile picture. This API will save the record in DynamoDB, and store the profile picture in an S3 bucket. 

We will start by creating a Spring Boot REST API using [https://start.spring.io](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.1.RELEASE&packaging=jar&jvmVersion=14&groupId=io.pratik&artifactId=customerregistration&name=customerregistration&description=Spring%20Boot%20with%20Dynamodb%20and%20S3%20to%20demonstrate%20LocalStack&packageName=io.pratik.customerregistration&dependencies=lombok,web) with minimum dependencies for web and Lombok. 


Next, we add dependencies to our pom.xml.

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>dynamodb</artifactId>
</dependency>
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
</dependency>
<dependency>
    <groupId>cloud.localstack</groupId>
    <artifactId>localstack-utils</artifactId>
    <version>0.2.1</version>
    <scope>test</scope>
</dependency>
```
Here we have added dependencies for the two AWS services - DynamoDB and S3. We also add a test scoped dependency on localstack to start the localstack container when the JUnit test starts and stops the container at the end of our test run.  

After that, we create the controller class containing the endpoint and two service classes for invoking the S3 and DynamoDB services.
We use the default profile for real AWS services and create an additional profile named "local" for testing with LocalStack (mock AWS services). The LocalStack URL is configured in application-local.properties as shown here. 

```application-local.properties
aws.local.endpoint=http://localhost:4566
```

Let us now take a look at our CustomerProfileStore class. 

```java
@Service
public class CustomerProfileStore {
    private static final String TABLE_NAME = "entities";
    private static final Region region = Region.US_EAST_1;
    
    private final String awsEndpoint;
    
    public CustomerProfileStore(@Value("${aws.local.endpoint:#{null}}") String awsEndpoint) {
        super();
        this.awsEndpoint = awsEndpoint;
    }

    private DynamoDbClient getDdbClient() {
        DynamoDbClient dynamoDB = null;;
        try {
            DynamoDbClientBuilder builder = DynamoDbClient.builder();
            // awsLocalEndpoint is set only in local environments
            if(awsEndpoint != null) {
                // override aws endpoint with localstack URL in dev environment
                builder.endpointOverride(new URI(awsEndpoint));
            }
            dynamoDB = builder.region(region).build();
        }catch(URISyntaxException ex) {
            log.error("Invalid url {}",awsEndpoint);
            throw new IllegalStateException("Invalid url "+awsEndpoint,ex);
        }
        return dynamoDB;
    }
    
```
We inject the URL of LocalStack using the variable `awsEndpoint`. The value is set only when we run our application using the local profile, else it has the default value null. 

In the method `getDdbClient()`, we pass this variable to the `endpointOverride()` method in the DynamoDbClientBuilder class only if the variable awsLocalEndpoint has a value which is the case when using the local profile.

I created the AWS resources - S3 Bucket and DynamoDB table using a [cloudformation template](https://github.com/thombergs/code-examples//tree/master/aws/localstack/cloudformation/sample.yaml). I prefer this approach instead of creating the resources individually from the console. It allows me to create and clean up all the resources with a single command at the end of the exercise following IAC principles.

#### Running the Spring Boot application

We start LocalStack with docker-compose using the command  as explained earlier in the section on running LocalStack with docker:

Next, We will create our resources with the CloudFormation service:

```
aws cloudformation create-stack \
  --endpoint-url http://localhost:4566 \
  --stack-name samplestack \
  --template-body file://sample.yaml \
  --profile localstack
```
Here we define the S3 bucket and DynamoDB table in a CloudFormation Template file - sample.yaml.
After creating our resources, we will run our Spring Boot application with the spring profile named "local".
```
java -Dspring.profiles.active=local -jar target/customerregistration-1.0.jar
```
I have set 8085 as the port for my application. I tested my API by sending the request using curl. You can also use Postman or any other REST client.

```
curl -X POST 
     -H "Content-Type: application/json" 
     -d '{"firstName":"Peter","lastName":"Parker", 
          "email":"peter.parker@fox.com", "phone":"476576576", 
          "photo":"rtruytiutiuyo"
         }' 
       http://localhost:8085/customers/
```

Finally, we run our Spring Boot app connected to the real AWS services by switching to the default profile.


## Summary

We saw how to use LocalStack for testing the integration of our application with AWS services locally. Localstack also has an [enterprise version](https://localstack.cloud/#pricing) available with more services and features. I hope this will help you to feel empowered and have more fun while working with AWS services during development. Ultimately leading to higher productivity, shorter development cycles, and lower AWS cloud bills.

You can refer to all the source code used in the article in my [Github repository](https://github.com/thombergs/code-examples/tree/master/aws/localstack).