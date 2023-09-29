# Beginners Guide: AWS DynamoDB

## Introduction

Even the dashboard for AWS (Amazon Web Services) can be daunting, let alone the documentation! Here, I decided to create a resource for juniors such as myself to help make life a little easier when making requests to a DynamoDB table. Note that we will be using IAM roles to grant your AWS resources the necessary permissions to access other AWS services securely. This approach eliminates the need to store AWS access keys directly in your application environment, reducing the risk of compromising your keys. In chronological order this tutorial will show you how to:

```markdown
1) Set up a DynamoDB table.
2) Create Lambda CRUD functions (plus code snippets).
3) Add relevant IAM roles, so that it does not carry out unintended actions (i.e., abuse). And a lot more.
4) Set up API Gateway so that the Lambda functions can communicate with your DynamoDB table.
5) Deploy the API to generate the Invoke URL needed for endpoints.
6) Use Postman/Thunder Client to make requests to this Invoke URL...
7) ...and thus, retrieve/add/update/delete data from/to your DynamoDB table.
8) (Eventually) make your frontend carry out CRUD functions...
9) ...and set up authorization via the likes of Firebase or OAuth.

```

## WARNING

It is vital you DO NOT push the following to GitHub (or anywhere else):

Your AWS Secret Access Key. This will grant all permissions across your entire AWS account -> abuse -> unwanted costs and/or compromise AWS resources and services.
Your Invoke URL - anyone would be able to make requests to your API.
<br>
Note: if you do not want to utilize Lambda functions and write your own code, make sure your keys are stored in your .env, and said .env is noted in your .gitignore.

## Getting Started
### Prerequisites
```markdown
- AWS Free acount
- API client/testing tools (Postman, Thunder Client, etc.)

```
### Step 1 - Set up DynamoDB Table
*to be populated (text and gifs)*
### Step 2 - Lambda CRUD Function
*Explanations and Code snippets to come*

### Security and Cost Concerns
