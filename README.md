```python
import json
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
    except NoBrokersAvailable:
        return {
            'statusCode': 500,
            'body': json.dumps('Failed to connect to Kafka cluster')
        }

