# Resume-Builder-Counter

# Resume-Builder-Counter

To build a visitor counter, i have created a Serverless Stack. 
Here is the architecture: 
                           S3 (Frontend) → JavaScript (Fetch API) → API Gateway → Lambda (Python/Node) → DynamoDB (Database)
         
          
          Architecture
 
          S3 (Resume Website)
          ↓
          JavaScript (Fetch)
          ↓
          API Gateway (HTTP API + CORS)
          ↓
          Lambda
          ↓
          DynamoDB

          
Steps :
         Step 1 : 
                1. Go to DynamoDB in the AWS Console.
                2. Click Create Table.
                3. Table Name: VisitorCount
                4. Partition Key: id (Type: String).
                5. Click Create.
                6. Once created, click Explore Items → Create Item.
                        o  id: visitors (as a string)
                        o  Add a new attribute: count (Type: Number) and set it to 0.


         Step 2: Create the Lambda Function
                This is the "brain" that will talk to the database.

                    1. Go to Lambda → Create Function.
                    2. Name: ResumeCounterFunction. Runtime: Python 3.x.
                    3. Under Permissions, AWS will create a basic role. We will edit this in a moment.
                    4. The Code: Paste this into the lambda_function.py tab:

                            import json
                            import boto3

                            dynamodb = boto3.resource('dynamodb')
                            table = dynamodb.Table('VisitorCount')

                            def lambda_handler(event, context):

                            response = table.update_item(
                            Key={'id': 'visitors'},
                            UpdateExpression='ADD #c :val',
                            ExpressionAttributeNames={'#c': 'count'},
                            ExpressionAttributeValues={':val': 1},
                            ReturnValues="UPDATED_NEW"
                            )

                            new_count = response['Attributes']['count']
                            
                            return {
                            "statusCode": 200,
                            "headers": {
                            "Access-Control-Allow-Origin": "*",
                            "Access-Control-Allow-Headers": "Content-Type",
                            "Access-Control-Allow-Methods": "OPTIONS,GET"
                            },
                            "body": json.dumps({"count": int(new_count)})
                            }



         Step 3: Grant Permissions (IAM)
                By default, Lambda isn't allowed to touch DynamoDB.

                    1. In the Lambda console, go to Configuration → Permissions.
                    2. Click the link under Role Name.
                    3. Click Add Permissions → Create inline policy.
                    4. Choose Service: DynamoDB.
                    5. Actions: GetItem, UpdateItem.
                    6. Resources: Specific -> Add the ARN of your VisitorCount table.
                    7. Review and Save.


         Step 4: Create the API Gateway
                This gives your JavaScript a URL to call.

                    1. Go to API Gateway → Create API.
                    2. Select HTTP API (easier and cheaper than REST).
                    3. Click Add Integration → Select Lambda. Pick your ResumeCounterFunction.
                    4. API Name: ResumeAPI.
                    5. Under Configure Routes, set the method to GET and resource path to /visit.
                    6. Crucial Step: Once the API is created, go to the CORS tab (left sidebar).
                        o Click Configure.
                        o Access-Control-Allow-Origin: * (or your S3 bucket URL).

                    Example:
                    Access-Control-Allow-Origin: http://onkar-resume-builder.s3-website.ap-south-1.amazonaws.com
                    Access-Control-Allow-Headers: Content-Type
                    Access-Control-Allow-Methods: GET,POST,OPTIONS
                        o Access-Control-Allow-Methods: GET.
                        o Click Save.

                    7. Get your URL: Look for the Invoke URL (e.g., https://random-id.execute-api.us-east-1.amazonaws.com/visit).


            Step 5: Update your Website (S3)
                    Add this JavaScript to your index.html file on your S3 resume.


