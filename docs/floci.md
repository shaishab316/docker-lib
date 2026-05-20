# Floci Cheatsheet

```bash
docker compose -f floci.yaml up -d
```

---

## 1. Environment Variables (App)

```bash
export AWS_ENDPOINT_URL=http://localhost:4566
export AWS_DEFAULT_REGION=us-east-1
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
```

Or in `.env`:
```env
AWS_ENDPOINT_URL=http://localhost:4566
AWS_DEFAULT_REGION=us-east-1
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test
```

---

## 2. Storage Modes

| Mode     | Description                                  |
|----------|----------------------------------------------|
| `memory` | Lost on restart — fastest, best for CI       |
| `hybrid` | Default — in-memory + disk flush every 5s    |
| `wal`    | Write-Ahead Log — maximum durability         |

---

## 3. AWS CLI Usage

```bash
# S3
aws s3 mb s3://my-bucket
aws s3 ls
aws s3 cp file.txt s3://my-bucket/
aws s3 rm s3://my-bucket/file.txt
aws s3 rb s3://my-bucket --force

# SQS
aws sqs create-queue --queue-name my-queue
aws sqs list-queues
aws sqs send-message --queue-url http://localhost:4566/000000000000/my-queue --message-body "Hello"
aws sqs receive-message --queue-url http://localhost:4566/000000000000/my-queue
aws sqs delete-queue --queue-url http://localhost:4566/000000000000/my-queue

# DynamoDB
aws dynamodb create-table \
  --table-name users \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
aws dynamodb list-tables
aws dynamodb put-item --table-name users --item '{"id":{"S":"1"},"name":{"S":"John"}}'
aws dynamodb get-item --table-name users --key '{"id":{"S":"1"}}'
aws dynamodb scan --table-name users
aws dynamodb delete-table --table-name users

# SNS
aws sns create-topic --name my-topic
aws sns list-topics
aws sns publish --topic-arn arn:aws:sns:us-east-1:000000000000:my-topic --message "Hello"

# Secrets Manager
aws secretsmanager create-secret --name mySecret --secret-string '{"key":"value"}'
aws secretsmanager get-secret-value --secret-id mySecret

# SSM Parameter Store
aws ssm put-parameter --name "/app/db-url" --value "postgres://..." --type String
aws ssm get-parameter --name "/app/db-url"
```

> All commands need `--endpoint-url http://localhost:4566` if `AWS_ENDPOINT_URL` is not set globally.

---

## 4. Node.js / TypeScript SDK

```bash
npm install @aws-sdk/client-s3 @aws-sdk/client-sqs @aws-sdk/client-dynamodb
```

```typescript
import { S3Client, CreateBucketCommand } from "@aws-sdk/client-s3";

const s3 = new S3Client({
  region: "us-east-1",
  endpoint: "http://localhost:4566",
  credentials: {
    accessKeyId: "test",
    secretAccessKey: "test",
  },
  forcePathStyle: true, // required for S3
});

await s3.send(new CreateBucketCommand({ Bucket: "my-bucket" }));
```

```typescript
import { SQSClient, SendMessageCommand } from "@aws-sdk/client-sqs";

const sqs = new SQSClient({
  region: "us-east-1",
  endpoint: "http://localhost:4566",
  credentials: { accessKeyId: "test", secretAccessKey: "test" },
});

await sqs.send(new SendMessageCommand({
  QueueUrl: "http://localhost:4566/000000000000/my-queue",
  MessageBody: "Hello",
}));
```

```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand, GetCommand } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({
  region: "us-east-1",
  endpoint: "http://localhost:4566",
  credentials: { accessKeyId: "test", secretAccessKey: "test" },
});

const ddb = DynamoDBDocumentClient.from(client);

await ddb.send(new PutCommand({
  TableName: "users",
  Item: { id: "1", name: "John" },
}));

const result = await ddb.send(new GetCommand({
  TableName: "users",
  Key: { id: "1" },
}));
```

---

## 5. NestJS Config Helper

```typescript
// aws.config.ts
import { S3Client } from "@aws-sdk/client-s3";
import { SQSClient } from "@aws-sdk/client-sqs";

const isLocal = process.env.NODE_ENV !== "production";

const awsConfig = {
  region: process.env.AWS_DEFAULT_REGION ?? "us-east-1",
  ...(isLocal && {
    endpoint: "http://localhost:4566",
    credentials: { accessKeyId: "test", secretAccessKey: "test" },
  }),
};

export const s3Client = new S3Client({ ...awsConfig, forcePathStyle: isLocal });
export const sqsClient = new SQSClient(awsConfig);
```

---

## 6. Supported Services

| Service           | Notes                          |
|-------------------|--------------------------------|
| S3                | REST XML compatible            |
| SQS               | FIFO + Standard                |
| DynamoDB          | Streams supported              |
| SNS               | Topics + subscriptions         |
| Lambda            | Docker-native execution        |
| API Gateway       | v1 + v2, WebSocket             |
| Cognito           | JWKS                           |
| ElastiCache       | IAM Auth                       |
| RDS               | PostgreSQL + MySQL             |
| Step Functions    | ASL                            |
| CloudFormation    | Stacks                         |
| EventBridge       | Rules + Scheduler              |
| KMS               | Sign + Verify                  |
| EKS               | Kubernetes                     |
| Secrets Manager   | —                              |
| SSM               | Parameter Store                |
| SES               | Email                          |
| Route 53          | DNS                            |
| Athena            | Query                          |
| Glue              | Data Catalog                   |
| Firehose          | S3 delivery                    |
| Bedrock Runtime   | Stub                           |

---

## 7. Health Check

```bash
curl http://localhost:4566/_floci/health
```

---

## 8. vs LocalStack

| Feature             | Floci          | LocalStack Community  |
|---------------------|----------------|-----------------------|
| Auth token required | Never          | Yes (March 2026+)     |
| Startup time        | ~24ms          | ~3,300ms              |
| Idle memory         | ~13 MiB        | ~143 MiB              |
| License             | MIT            | Apache (restricted)   |
| Port                | 4566           | 4566                  |
| Code change needed  | Zero           | Zero                  |