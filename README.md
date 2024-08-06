```python
import os
import json
import traceback
from kafka import KafkaProducer, KafkaConsumer
from kafka.errors import NoBrokersAvailable

def produce_message(kafka_brokers, username, password, topic, message):
    try:
        producer = KafkaProducer(
            bootstrap_servers=kafka_brokers,
            security_protocol="SASL_SSL",
            sasl_mechanism="PLAIN",
            sasl_plain_username=username,
            sasl_plain_password=password,
            value_serializer=lambda v: json.dumps(v).encode('utf-8')
        )
        
        producer.send(topic, {'message': message})
        producer.flush()
        producer.close()
        print(f"Successfully sent message to Kafka topic {topic}")
        return {
            'statusCode': 200,
            'body': json.dumps(f'Successfully sent message to Kafka topic {topic}')
        }
    except Exception as e:
        error_message = str(e)
        print(f"Failed to send message to Kafka topic: {error_message}")
        traceback.print_exc()
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Failed to send message to Kafka topic', 'details': error_message})
        }

def consume_message(kafka_brokers, username, password, topic):
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

        for message in consumer:
            consumer.commit()
            consumer.close()
            print(f"Consumed message: {message.value.decode('utf-8')}")
            return {
                'statusCode': 200,
                'body': json.dumps(f'Consumed message: {message.value.decode("utf-8")}')
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
    # 从环境变量中读取配置
    kafka_brokers = os.getenv('KAFKA_BROKERS', 'your_vpc_endpoint1:9092,your_vpc_endpoint2:9092,your_vpc_endpoint3:9092').split(',')
    username = os.getenv('KAFKA_USERNAME', 'your_username')
    password = os.getenv('KAFKA_PASSWORD', 'your_password')
    topic = os.getenv('KAFKA_TOPIC', 'your_topic')
    message = os.getenv('KAFKA_MESSAGE', 'Default message')
    operation = os.getenv('KAFKA_OPERATION', 'produce')  # 默认操作为 produce

    if operation == 'produce':
        return produce_message(kafka_brokers, username, password, topic, message)
    elif operation == 'consume':
        return consume_message(kafka_brokers, username, password, topic)
    else:
        return {
            'statusCode': 400,
            'body': json.dumps('Invalid operation. Please set KAFKA_OPERATION to either "produce" or "consume".')
        }
