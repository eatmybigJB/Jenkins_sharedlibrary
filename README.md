```python
import os
import json
import traceback
from kafka import KafkaConsumer
from kafka.errors import NoBrokersAvailable
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
            traceback.print_exc()
            raise

def consume_message():
    kafka_brokers = os.getenv('KAFKA_BROKERS', 'your_vpc_endpoint1:9092,your_vpc_endpoint2:9092,your_vpc_endpoint3:9092').split(',')
    username = os.getenv('KAFKA_USERNAME', 'your_username')
    password = os.getenv('KAFKA_PASSWORD', 'your_password')
    topic = os.getenv('KAFKA_TOPIC', 'your_topic')

    print("Connecting to Kafka brokers:", kafka_brokers)
    
    try:
        consumer = KafkaConsumer(
            topic,
            bootstrap_servers=kafka_brokers,
            security_protocol="SASL_SSL",
            sasl_mechanism="PLAIN",
            sasl_plain_username=username,
            sasl_plain_password=password,
            auto_offset_reset='earliest',
            enable_auto_commit=False,
            group_id='your_consumer_group'
        )
        print("Connected to Kafka, consuming messages")

        for message in consumer:
            consumer.commit()
            data = message.value.decode('utf-8')
            print(f"Consumed message: {data}")
            username, email = data.split(',')
            update_cognito_user(username, email)
            break  # 只消费一条消息后退出

        consumer.close()
        return {
            'statusCode': 200,
            'body': json.dumps('Successfully processed message from Kafka topic')
        }
    except NoBrokersAvailable as e:
        error_message = str(e)
        print(f"Failed to connect to Kafka cluster: {error_message}")
        traceback.print_exc()
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Failed to connect to Kafka cluster', 'details': error_message})
        }
    except Exception as e:
        error_message = str(e)
        print(f"An unexpected error occurred: {error_message}")
        traceback.print_exc()
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'An unexpected error occurred', 'details': error_message})
        }

def lambda_handler(event, context):
    print("Lambda function invoked")
    result = consume_message()
    print(f"Lambda function result: {result}")
    return result
