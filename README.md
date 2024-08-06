```python
import os
import json
import traceback
from kafka import KafkaConsumer
from kafka.errors import NoBrokersAvailable
import boto3

def invoke_update_cognito_lambda(username, email):
    client = boto3.client('lambda')
    payload = json.dumps({'username': username, 'email': email})
    
    try:
        response = client.invoke(
            FunctionName=os.getenv('UPDATE_COGNITO_FUNCTION_NAME'),
            InvocationType='Event',  # 异步调用
            Payload=payload
        )
        print(f"Invoked update_cognito_lambda for user {username}")
    except Exception as e:
        print(f"Failed to invoke update_cognito_lambda for user {username}: {e}")
        raise

def consume_message():
    kafka_brokers = os.getenv('KAFKA_BROKERS', 'your_vpc_endpoint1:9092,your_vpc_endpoint2:9092,your_vpc_endpoint3:9092').split(',')
    kafka_username = os.getenv('KAFKA_USERNAME', 'your_username')
    kafka_password = os.getenv('KAFKA_PASSWORD', 'your_password')
    topic = os.getenv('KAFKA_TOPIC', 'your_topic')

    print("Connecting to Kafka brokers:", kafka_brokers)
    
    try:
        consumer = KafkaConsumer(
            topic,
            bootstrap_servers=kafka_brokers,
            security_protocol="SASL_SSL",
            sasl_mechanism="PLAIN",
            sasl_plain_username=kafka_username,
            sasl_plain_password=kafka_password,
            auto_offset_reset='earliest',
            enable_auto_commit=False,
            group_id='your_consumer_group'
        )
        print("Connected to Kafka, consuming messages")

        for message in consumer:
            consumer.commit()
            data = message.value.decode('utf-8')
            print(f"Consumed message: {data}")
            try:
                username, email = data.split(',')
                username = username.strip()
                email = email.strip()
                if username and email:
                    invoke_update_cognito_lambda(username, email)
                else:
                    print(f"Invalid message format: {data}")
            except ValueError:
                print(f"Failed to parse message: {data}")
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


