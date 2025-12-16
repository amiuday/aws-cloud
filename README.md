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



=====================================================================================================================



=====================================================================================================================


=====================================================================================================================


=====================================================================================================================


=====================================================================================================================


=====================================================================================================================
