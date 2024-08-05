```python
import json
import traceback
from kafka import KafkaConsumer
from kafka.errors import NoBrokersAvailable

def lambda_handler(event, context):
    # Kafka配置
    kafka_brokers = [
        'your_vpc_endpoint1:9092',
        'your_vpc_endpoint2:9092',
        'your_vpc_endpoint3:9092'
    ]
    username = 'your_username'
    password = 'your_password'
    
    # 尝试连接到Kafka集群
    try:
        consumer = KafkaConsumer(
            bootstrap_servers=kafka_brokers,
            security_protocol="SASL_SSL",
            sasl_mechanism="PLAIN",
            sasl_plain_username=username,
            sasl_plain_password=password
        )
        consumer.close()
        return {
            'statusCode': 200,
            'body': json.dumps('Successfully connected to Kafka cluster')
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
