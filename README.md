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
- 1.0) Under the DynamoDB dashboard, head to 'Tables' on the left hand side. 
- 1.1) Click the orange 'Create table'.
- 1.2) Enter a name for the table and the Partition key. Make note of these two names - they will be
needed for your Lambda Function later.
- 1.3) You can leave everything under 'Default settings'. Everything can be changed later (except
secondary indexes).
- 1.4) Click the orange 'Create table' button at the bottom.
```
### Step 2 - Lambda CRUD Function
```markdown
- 2.0) Navigate to Lambda. Search for it in the search bar or find it under 'Services' (top left, next to
the search bar).
- 2.1) Select 'Author from scratch' and leave all the default options.
- 2.2) Hit the orange 'Create function' button.
- 2.3) Copy and paste the below code into the 'Code' section. Check the comments for clues on how to make
it fit your table.
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
- 2.4) Locate the blue 'Test' button and click the dropdown. Select 'Configure test event'.
- 2.5) Under 'Configure test event', select 'Create new event' and name the event (e.g., testEvent1).
- 2.6) Here is example code for the event JSON - this is what will be delivered to the DynamoDB table:
```

```javascript
{
  "task": "SUCCESS"
}

```

```markdown
- 2.7) Press 'Deploy' and then 'Test'.
- 2.8) Expect a 500 error. Scroll along the error message, which should mention 'AccessDeniedException'.
```

### Step 3 - IAM Role
```markdown
- 3.0) Go to the 'Configuration' section (three right from 'Code'). Now click 'Permissions' on the left-hand
side (still under 'Configuration').
- 3.1) This will take you to 'Execution role'. Under 'Role name', click the only link below (should be the
name of the Lambda function, followed by '-role-*random characters*').
- 3.2) You'll have been taken to the 'Roles' section (IAM -> Roles). Under 'Permissions policies', you will
see one policy that has been attached for you. Click the 'Add permissions' dropdown and click
'Attach policies'.
- 3.3) Search for 'AmazonDynamoDBFullAccess'. We will use this *temporarily* to establish a connection.
Click 'Add permissions'.
- 3.4) Under 'Permissions policies', you should now have two - the full access policy (Type: 'Customer
managed') and the one automatically assigned (Type: AWS managed).
```

### Step 4 - Populating the DynamoDB table via Lambda
```markdown
- 4.0) Go back to the Lambda dashboard and click 'Test' again.
- 4.1) You should recieve the 200 code - a successful outcome. Navigate to the DynamoDB dashboard to see the
new entry.
- 4.2) DynamoDB -> Explore Items -> the name of your table. You should see a random number under the Partition
key (String) - which is called whatever you named it earlier - and 'SUCCESS' under Task, assuming you used the
JSON from Step 2.
```
If you still get the 500 error, check what the error might be. Incorrectly spelling what you named the partition key, for example, would return an error mentioning "ValidationException" and "One or more parameter values were invalid: Missing the key PARTITION_KEY_NAME in the item".
<br>
<br>
This highlights the usefulness of utilising 'error' on line 25 of the Lambda code.

### Step 5 - Creating strict IAM permissions
```markdown
- 5.0) Under the DynamoDB dashboard: Update settings -> Overview. Find 'General information' and expand
'Additonal info' below. Copy the ARN (Amazon Resource Name).
- 5.1) Go back to the IAM dashboard and navigate to the 'Policies' section.
- 5.2) Click 'Create policy'. Search for DynamoDB under 'Select a service'.
- 5.3) Now search for 'PutItem' under actions and tick the box.
- 5.4) Finally, click on the 'Resources' section and click 'Specific'. Then click 'Add ARNs'.
- 5.5) Select 'This Account' and enter/select your region and the name of your table. Paste the ARN
where required.
- 5.6) Under 'Review and create'. Give it a name similar but distinguiable from the name of your
Lambda Function. Click 'Create policy'.
- 5.7) Go back to IAM -> Roles. Find a role that resembles the name of your Lambda Function and click
the blue text. Select the AmazonDynamoDBFullAccess
policy and remove it.
- 5.8) On the same page, click 'Add permissions' dropdown and click 'Attach policies. Find the
policy you created in Step 5.7 and select 'Add permissions'.
- 5.9) You should now have 2 policies under 'Permission policies' -
'AWSLambdaBasicExecutionRole-*random characters*' (which was made for you when you created your Lambda
Functions) and the one you created in Step 5.7.
```

### Step 6 - Test the strict permissions
```markdown
6.0) Head back to Lambda and hit 'Test' again. You should recieve the 200 statusCode message again,
"Task created successfully".
6.1) Navigate to the DynamoDB table, under explore items. You should now have two 'SUCCESS' entries
under 'Task' (assuming you only clicked 'Test' in Lambda twice).
```
With this, your Lambda code has successfully added a new entry to your DynamoDB table, but this time with a 'PutItem' only permission. You can now repeat the process, with a NEW Lambda function (one for each CRUD operation), with code similar to what was provided in Step 2.

### Security and Cost Concerns
