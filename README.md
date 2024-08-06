```python
import os
import json
import boto3
from botocore.exceptions import ClientError

def update_cognito_user(username, email):
    client = boto3.client('cognito-idp')
    user_pool_id = os.getenv('COGNITO_USER_POOL_ID')
    
    print(f"Updating Cognito user pool {user_pool_id} for user {username}")

    try:
        response = client.admin_get_user(
            UserPoolId=user_pool_id,
            Username=username
        )
        print(f"User {username} exists, updating email to {email}")
        # 用户已存在，更新用户属性
        client.admin_update_user_attributes(
            UserPoolId=user_pool_id,
            Username=username,
            UserAttributes=[
                {'Name': 'email', 'Value': email},
                {'Name': 'email_verified', 'Value': 'true'}
            ]
        )
        print(f"Updated user {username} with email {email}")
    except ClientError as e:
        if e.response['Error']['Code'] == 'UserNotFoundException':
            print(f"User {username} not found, creating new user with email {email}")
            # 用户不存在，创建新用户
            client.admin_create_user(
                UserPoolId=user_pool_id,
                Username=username,
                UserAttributes=[
                    {'Name': 'email', 'Value': email},
                    {'Name': 'email_verified', 'Value': 'true'}
                ],
                MessageAction='SUPPRESS'  # 不发送欢迎邮件
            )
            print(f"Created user {username} with email {email}")
        else:
            print(f"ClientError: {e}")
            raise

def lambda_handler(event, context):
    try:
        username = event['username']
        email = event['email']
        update_cognito_user(username, email)
        return {
            'statusCode': 200,
            'body': json.dumps(f'Successfully processed user {username}')
        }
    except Exception as e:
        print(f"Error processing user {event['username']}: {e}")
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error processing user {event["username"]}: {e}')
        }

