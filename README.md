# aws-cloud


What is a CloudFormation Stack?

A stack is a collection of AWS resources that are created, updated, and deleted together.

Think like this:

Stack = one logical unit of infrastructure

Example:

1 stack → S3 bucket + IAM role + policy

Delete stack → everything goes away together

📌 Stack is the owner of the resources.


=====================================================================================================================

Template = Desired State

A CloudFormation template is a declaration of what you want.

You don’t say:

“Create EC2, then attach SG, then add route…”

You say:

“This is how my infrastructure should look”

CloudFormation figures out:

Order

Dependencies

Updates

📌 You define WHAT, CFN handles HOW.

=====================================================================================================================
CFN Manages the Lifecycle

CloudFormation controls the entire lifecycle:
Create
create-stack
→ Resources are created

Update
update-stack
→ Only changed parts are modified

Delete
delete-stack
→ Everything in the stack is removed

📌 This prevents manual mistakes.

=====================================================================================================================

Key Template Components (Must Know)
1️⃣ Parameters (Inputs)

Dynamic values you pass at runtime.

Example:

Parameters:
  Env:
    Type: String


📌 Same template → dev / prod / test

2️⃣ Resources (🔥 MOST IMPORTANT)

This is where actual AWS resources are defined.

Example:

Resources:
  MyBucket:
    Type: AWS::S3::Bucket


📌 No resources = no stack value.

3️⃣ Outputs (Exports)

Values returned after stack creation.

Example:

Outputs:
  BucketName:
    Value: !Ref MyBucket


📌 Used by:

Humans

Other stacks (cross-stack reference)

=====================================================================================================================

Golden Rule (Very Important)

🚫 Never create or modify stack resources manually

Why?

CFN will lose track
Leads to drift
Updates may fail or delete things unexpectedly

Correct way:
✅ Always update template
✅ Run update-stack

🧠 One-line Mental Model

CloudFormation = Git for infrastructure
Template → code
Stack → deployed version
Update → commit
Delete → cleanup

=====================================================================================================================
CloudFormation Template Structure (Must Know)

A CloudFormation template is just a YAML file that follows a fixed pattern.

Minimal valid structure
AWSTemplateFormatVersion: "2010-09-09"
Description: My first CloudFormation template

Resources:
  LogicalResourceName:
    Type: AWS::Service::Resource
    Properties:
      ...


📌 Resources is mandatory
Everything else is optional.

=====================================================================================================================
YAML Basics (Only what you need)

YAML is indentation-based.

Rules:

Spaces matter (use 2 spaces)

No tabs

Key: value format

Example:

Key:
  SubKey: value


❌ Wrong:

Key:
SubKey: value


=====================================================================================================================
Resources Section (Heart of CFN)

This is where AWS resources are defined.

Example: S3 Bucket
Resources:
  MyBucket:
    Type: AWS::S3::Bucket


Breakdown:

MyBucket → Logical ID (used inside template)

Type → AWS resource type

📌 Logical ID ≠ actual resource name

=====================================================================================================================
Intrinsic Functions (Only 3 you MUST know)

These are built-in CFN helpers.

1️⃣ !Ref → Get value

Used to reference:

Parameters

Resource IDs

Example:

BucketName: !Ref MyBucket


📌 Returns:

Bucket name

Instance ID

Parameter value


2️⃣ !GetAtt → Get attribute

Used when you need specific attributes.

Example:

BucketArn: !GetAtt MyBucket.Arn


📌 Use when !Ref is not enough.

3️⃣ !Sub → String substitution

Used to build names dynamically.

Example:

BucketName: !Sub "my-app-${Env}-bucket"


📌 Cleaner than joining strings manually.

=====================================================================================================================
Pattern to Remember (Don’t Memorize Syntax)

Every CFN resource follows this pattern:

Resources:
  LogicalName:
    Type: AWS::Service::Resource
    Properties:
      Property1: value
      Property2: value

=====================================================================================================================
Full s3.yaml Example (Type This Once)
AWSTemplateFormatVersion: "2010-09-09"
Description: S3 bucket example

Parameters:
  Env:
    Type: String
    Default: dev

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "my-demo-${Env}-bucket"

Outputs:
  BucketName:
    Value: !Ref MyBucket

=====================================================================================================================

STEP 1: Update stack → Add tags to S3
Edit templates/s3.yaml

Add Tags under the S3 bucket.

AWSTemplateFormatVersion: "2010-09-09"
Description: S3 bucket with tags

Parameters:
  Env:
    Type: String
    Default: dev

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "demo-s3-${Env}"
      Tags:
        - Key: Application
          Value: CloudFormationDemo
        - Key: Environment
          Value: !Ref Env

Outputs:
  BucketName:
    Value: !Ref MyBucket

📌 Only template change → no console changes

Update stack via CLI
aws cloudformation update-stack \
  --stack-name demo-s3 \
  --template-body file://templates/s3.yaml \
  --parameters ParameterKey=Env,ParameterValue=dev


✔️ Expected result:

Stack goes to UPDATE_IN_PROGRESS

Then UPDATE_COMPLETE

=====================================================================================================================

STEP 2: Describe stack (current state)
Basic stack info
aws cloudformation describe-stacks \
  --stack-name demo-s3

Clean, readable output (recommended)
aws cloudformation describe-stacks \
  --stack-name demo-s3 \
  --query "Stacks[0].StackStatus"


You should see:

"UPDATE_COMPLETE"

=====================================================================================================================
📜 STEP 3: Describe stack events (VERY IMPORTANT)

This shows what CFN actually did.

aws cloudformation describe-stack-events \
  --stack-name demo-s3


Look for:

UPDATE_IN_PROGRESS

UPDATE_COMPLETE

Resource type: AWS::S3::Bucket

📌 When something fails → error is always here

=====================================================================================================================

🔴 STEP 4: Delete stack (cleanup)
Delete stack
aws cloudformation delete-stack \
  --stack-name demo-s3

Watch delete events
aws cloudformation describe-stack-events \
  --stack-name demo-s3


Final state:

DELETE_COMPLETE


✔️ S3 bucket is deleted
✔️ Stack is gone
✔️ No orphan resources
=====================================================================================================================
🧠 COMMANDS YOU MUST REMEMBER (LOCK THESE 🔐)
Purpose	Command
Create stack	create-stack
Update stack	update-stack
Stack status	describe-stacks
Debug	describe-stack-events
Cleanup	delete-stack

If you know these → you know CloudFormation basics

=====================================================================================================================
🧱 STEP 1: Create parameterized s3.yaml

Edit or create:

templates/s3.yaml

AWSTemplateFormatVersion: "2010-09-09"
Description: Parameterized S3 bucket for multiple environments

Parameters:
  Env:
    Type: String
    Description: Deployment environment
    AllowedValues:
      - dev
      - prod
    Default: dev

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "my-app-${Env}-bucket"

Outputs:
  BucketName:
    Description: Name of the S3 bucket
    Value: !Ref MyBucket

Using parameter in resource
BucketName: !Sub "my-app-${Env}-bucket"


If:

Env = dev → my-app-dev-bucket

Env = prod → my-app-prod-bucket

📌 This is environment isolation

🟢 STEP 2: Create DEV stack

Since stack is deleted, create fresh.

aws cloudformation create-stack \
  --stack-name s3-dev-stack \
  --template-body file://templates/s3.yaml \
  --parameters ParameterKey=Env,ParameterValue=dev

Verify output
aws cloudformation describe-stacks \
  --stack-name s3-dev-stack \
  --query "Stacks[0].Outputs"

🟡 STEP 3: Create PROD stack (same template!)
aws cloudformation create-stack \
  --stack-name s3-prod-stack \
  --template-body file://templates/s3.yaml \
  --parameters ParameterKey=Env,ParameterValue=prod


✔️ Two stacks
✔️ Same template
✔️ Different resources

=====================================================================================================================
🎯 Goal

Instead of this 👇 (ugly & error-prone):

--parameters ParameterKey=Env,ParameterValue=dev


We want this 👇 (clean & reusable):

--parameters file://params/dev.json  

=====================================================================================================================

📁 STEP 1: Create params folder


In your repo root:

cloudformation/
├── templates/
│   └── s3.yaml
└── params/
    ├── dev.json
    └── prod.json

🧱 STEP 2: Create dev.json

params/dev.json

[
  {
    "ParameterKey": "Env",
    "ParameterValue": "dev"
  }
]

🧱 STEP 3: Create prod.json

params/prod.json

[
  {
    "ParameterKey": "Env",
    "ParameterValue": "prod"
  }
]
📌 JSON must be an array, not an object.

🟢 STEP 4: Create stack using parameters JSON
DEV stack
aws cloudformation create-stack \
  --stack-name s3-dev-stack \
  --template-body file://templates/s3.yaml \
  --parameters file://params/dev.json

PROD stack
aws cloudformation create-stack \
  --stack-name s3-prod-stack \
  --template-body file://templates/s3.yaml \
  --parameters file://params/prod.json


✔️ Same template
✔️ Different param files
✔️ Clean commands
=====================================================================================================================
🔹 What a Change Set REALLY is

A Change Set is a PREVIEW (dry run) of what CloudFormation would do if you apply a create or update.

Think of it as:

“Tell me what will change BEFORE you actually change it.”

Best Mental Model (remember this)
Action	What it does
create-stack	Creates resources immediately
update-stack	Updates resources immediately
create-change-set	Shows planned changes (no action yet)
execute-change-set	Actually applies those changes

📌 Change Set = Plan, not Apply

✅ Correct way to preview a NEW stack

Use create-change-set with type CREATE.

aws cloudformation create-change-set \
  --stack-name prod-v2-stack \
  --change-set-name prod-v2-preview \
  --change-set-type CREATE \
  --template-body file://templates/s3.yaml \
  --parameters file://params/prod.json


📌 Important:

No resources are created

Stack is NOT active yet

You only get a preview

🔍 Review the plan
aws cloudformation describe-change-set \
  --stack-name prod-v2-stack \
  --change-set-name prod-v2-preview


You’ll see:

All resources to be created

Their properties

Replacement info


🟢 Only after approval → execute
aws cloudformation execute-change-set \
  --stack-name prod-v2-stack \
  --change-set-name prod-v2-preview


Now:
✅ Stack is created
✅ Resources are deployed


=====================================================================================================================



=====================================================================================================================



=====================================================================================================================



=====================================================================================================================




=====================================================================================================================




=====================================================================================================================




=====================================================================================================================




=====================================================================================================================




=====================================================================================================================



=====================================================================================================================




=====================================================================================================================


=====================================================================================================================
