# Beginners Guide: AWS DynamoDB (Node.js)

## Introduction

Even the dashboard for AWS (Amazon Web Services) can be daunting, let alone the documentation! Here, I decided to create a resource for juniors such as myself to help make life a little easier when making requests to a DynamoDB table. Note that we will be using IAM roles to grant your AWS resources the necessary permissions to access other AWS services securely. This approach eliminates the need to store AWS access keys directly in your application environment (code or config files), reducing the risk of compromising your keys. In chronological order this tutorial will show you how to:

```markdown
1) Set up a DynamoDB table.
2) Create Lambda CRUD functions (plus code snippets).
3) Add relevant IAM roles, so that it does not carry out unintended actions (i.e., abuse). And a lot more.
4) Set up API Gateway so that the Lambda functions can communicate with your DynamoDB table.
5) Set up API Gateway settings such as 'Throttling' (limiting the rate requests can be made)
6) Deploy the API to generate the Invoke URL needed for endpoints.
7) Use Postman/Thunder Client to make requests to this Invoke URL...
8) ...and thus, retrieve/add/update/delete data from/to your DynamoDB table.
9) (Eventually) make your frontend carry out CRUD functions...
10) ...and set up authorization via the likes of Firebase or OAuth.

```
<br>
EDIT: AWS Lambda has recently added a the 'Function URL' option under 'Configuration' (Lambda dashboard). This URL can be used instead of API Gateway.

## WARNING

It is vital you DO NOT push the following to GitHub (or anywhere else):

- Your AWS Secret Access Key. This Key grants all permissions across your *entire* AWS account. Your resources would be at risk and AWS would charge you if you breach the free tier.
- Your Invoke URL - anyone would be able to make requests to your API. The tutorial will set up API Gateway settings to bolster security (and eventually authorisation and/or authentication).
<br>
Note: if you do not want to utilize Lambda functions and write your own code, make sure your keys are stored in your .env, and said .env is added to your .gitignore.

## Getting Started
### Prerequisites
```markdown
- AWS Free acount
- API client/testing tools (Postman, Thunder Client, etc.)

```
### Step 1 - Set up DynamoDB Table
```markdown
- 1.1) Under the DynamoDB dashboard, head to 'Tables' on the left hand side. 
- 1.2) Click the orange 'Create table'.
- 1.3) Enter a name for the table and the Partition key. Make note of these two names - they will be needed for your
Lambda Function later.
- 1.4) You can leave everything under 'Default settings'. Everything can be changed later (except secondary indexes).
- 1.5) Click the orange 'Create table' button at the bottom.
```
### Step 2 - Lambda CRUD Function
```markdown
- 2.1) Navigate to Lambda. Search for it in the search bar or find it under 'Services' (top left, next to the search bar).
- 2.2) Select 'Author from scratch' and leave all the default options.
- 2.3) Hit the orange 'Create function' button.
- 2.4) Copy and paste the below code into the Code section. Check the comments for clues on how to make it fit your table.
```
<br>
Example Lambda function (Create new item in DynamoDB table):

```javascript
import { DynamoDBClient, PutItemCommand } from "@aws-sdk/client-dynamodb";

const dynamoDbClient = new DynamoDBClient({ region: "YOUR_REGION" }); // e.g., "eu-west-1"

export const handler = async (event) => {
  const task = event.task; // Imagine this app is for a To-do list, with tasks

  const params = {
    TableName: "DYNAMODB_TABLE_NAME",
    Item: {
      TableEntryId: { S: `${Date.now()}` }, // TableEntryId = the PARTITION KEY
      TableEntry: { S: entry },
    },
  };

  try {
    await dynamoDbClient.send(new PutItemCommand(params));
    return {
      statusCode: 200,
      body: JSON.stringify({ message: "Table entry inserted successfully" }),
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ message: "Table entry insertion failed", error }), // Notice the use of error
    };
  }
};
```

```markdown
- 2.5) Locate the blue 'Test' button and click the dropdown. Select 'Configure test event'.
- 2.6) Under 'Configure test event', select 'Create new event' and name the event (e.g., testEvent1).
- 2.7) Here is example code for the event JSON - this is what will be delivered to the DynamoDB table:
```

```javascript
{
  "task": "SUCCESS"
}

```


### Security and Cost Concerns
