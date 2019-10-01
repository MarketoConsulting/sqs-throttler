# sqs-throttler
SQS setup to accept messages from API Gateway and retry calls to avoid concurrency error

### Required Values (Fill these in during setup):

| Item            | Value   |
| ------------------------- |
| aws user        | <value> |
| sqs name        | <value> |
| sqs url         | <value> |
| sqs arn         | <value> |
| sqs policy arn  | <value> |
| sqs policy name | <value> |
| sqs role arn    | <value> |
| sqs role name   | <value> |

aws secret key id = 
aws secret access key = 

## SQS Queue Setup:
1. Enter and select SQS in main AWS console

2. Create new queue in SQS console
- A. Add Queue Name
- B. Select 'Standard Queue'
- C. Configure queue
  - i. This step can be left alone if you do not have specifics, it is populated with default values
- D. Click 'Create Queue'
- E. You are kicked back out to the SQS console, in the bottom pane you will have an SQS Name, URL, and ARN. Write these down.

## Create SQS Role
1. Enter and select IAM in main AWS console

2. In left column, select 'Policies'

3. Top left of right pane, select 'Create Policy'
- A. In policy picker/editor, click the JSON table and add the following JSON:
  - {"Version":"2012-10-17","Statement":[{"Effect":"Allow","Resource":["YOUR-SQS-ARN"],"Action":["sqs:SendMessage","sqs:ReceiveMessage"]}]}
- B. Enter the SQS ARN from the previous step in the above JSON blob
- C. Click 'Review Policy'
  - i. NOTE: Any combination of spaces and tabs for indenting, or leading spaces at the top of the blob will cause the save to error our.  It is best practice to minify your JSON policy blob before entering it into the AWS policy editor
- D. You will be kicked to a 'Review Policy' screen, enter a policy name and description. Write the name down and save/create the policy
- E. You will be kicked back out to the main policy console, search for you policy and click into it
  - i. write down the policy ARN
  
4. In the absolute left column, click on 'Roles'

5. Click 'Create Role' to create a new role
- A. Select AWS as the type of trusted entity
- B. Select 'API Gateway' as the service that the role will use, your use case will default to 'API Gateway', click 'Next: Permissions' to continue
- C. Click through 'Next: Tags'
- D. After clicking 'Next: Review', give the policy a name and write this down and click 'Create Role'
- E. You will be kicked back out to the IAM console, click back into the role you created
- i. Attach your sqs policy by clicking 'Attach Policies' and searching for the role you created

## API Gateway Setup:
1. create new API

2. Create new Method (POST)
- A. Integration type = AWS Service
- B. Select your region
- C. AWS Service = Simple Queue Service
- D. HTTP Method = POST
- E. Action Type = Path Override
  - i. Path Override = <userid>/<sqs queue name> (eg 123456789123/demoqueue)
- F. Execution Role = <sqs role arn> (eg arn:aws:iam::123456789123:role/apigateway-sqs-demoqueue)	
- G. Content Handling = Passthrough
	
3. You will be kicked back out to the Method Execution diagram, click on the Integration Request step and add the following:
- A. HTTP Headers
  - i. Name = Content-Type
  - ii. Mapped From = 'application/x-www-form-urlencoded'
    - a. include the single quotes
- B. Mapping Templates
  - i. Request Body Pass Through = Never
  - ii. Content-Type = application/json
    - a. click on new Mapping Template created when entering 'Content-Type value' and scroll down
    - b. add the following template without quotes: Action=SendMessage&MessageBody=$util.urlEncode("$input.body")
    - c. save
