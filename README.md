# AWS IAM Roles for Service Accounts (IRSA) for Confluent S3 Sink Kafka Connect Connector

## Context

Kafka connect [confluentinc/kafka-connect-storage-cloud/tree/master/kafka-connect-s3][0] is an open 
source connector in charge of landing data from Kafka to S3. It provides key features as multi-part 
uploads, highly configurable S3 partitioning, exactly once semantics, several formats as parquet and 
compression.

In regard to AWS authentication, it supports static credentials or assuming roles with some 
previously provided AWS credentials.

[AWS IAM roles for service accounts (IRSA)][5] is a recommended AWS approach for applications such
as Kubernetes to authenticate with AWS services without using static credentials.

This repository is focused on using Confluent S3 Sink current features to use IRSA the connectors.

## Implementation

The Confluent S3 Sink connector supports providing a custom AWS credentials provider class, 
which can be configured via connector properties. The class must implement both 
`com.amazonaws.auth.AWSCredentialsProvider` and`org.apache.kafka.common.Configurable`.

This provider is a wrapper around AWS’s native [WebIdentityTokenCredentialsProvider][1], 
similar to how [AwsAssumeRoleCredentialsProvider][2] is implemented. 
It enables configuring IRSA credentials directly via connector properties.

### References
- [S3SinkConnectorConfig.java][3]
- [AwsAssumeRoleCredentialsProvider.java][4]

## Connector Configuration

Add the following settings to the Confluent S3 Sink connector:
- `irsa.role.arn`: Role ARN to use when starting a session.
- `irsa.session.name`: Role session name to use when starting a session.
- `irsa.token.file:` Path to the web identity token file.


For example:
```json
{
  "name": "my-s3-sink",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "s3.credentials.provider.class": "io.confluent.connect.s3.auth.AwsWebIdentityTokenCredentialsProvider",
    "s3.credentials.provider.irsa.role.arn": "arn:aws:iam::123456689123:role/my-role",
    "s3.credentials.provider.irsa.session.name": "my--sink-connector-session",
    "s3.credentials.provider.irsa.token.file": "/var/run/secrets/kubernetes.io/serviceaccount/token",
    ...
  }
}
```

## Release
Run:
```bash
./gradlew clean jar
``` 
The jar is located in /lib/build/kafka-connect-s3-irsa.jar

## Deploy
Place the jar inside the connector's lib classpath, for example: 
`confluentinc-kafka-connect-avro-converter-7.8.0/lib`


[0]: https://github.com/confluentinc/kafka-connect-storage-cloud
[1]: https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/WebIdentityTokenCredentialsProvider.html
[2]: https://github.com/rodrigo-molina/kafka-connect-storage-cloud/blob/f3f928abccee1e21ac2126b7eddfc83bff860e81/kafka-connect-s3/src/main/java/io/confluent/connect/s3/auth/AwsAssumeRoleCredentialsProvider.java#L40
[3]: https://github.com/confluentinc/kafka-connect-storage-cloud/blob/19029ec7d1b7e557dd5b81946a60227619972c40/kafka-connect-s3/src/main/java/io/confluent/connect/s3/S3SinkConnectorConfig.java#L946
[4]: https://github.com/confluentinc/kafka-connect-storage-cloud/blob/19029ec7d1b7e557dd5b81946a60227619972c40/kafka-connect-s3/src/main/java/io/confluent/connect/s3/auth/AwsAssumeRoleCredentialsProvider.java
[5]: https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html
