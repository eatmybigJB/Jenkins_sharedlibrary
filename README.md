```python
import os
import json
import traceback
from kafka import KafkaProducer

def lambda_handler(event, context):
    # 从环境变量中读取配置
    kafka_brokers = os.getenv('KAFKA_BROKERS', 'your_vpc_endpoint1:9092,your_vpc_endpoint2:9092,your_vpc_endpoint3:9092').split(',')
    username = os.getenv('KAFKA_USERNAME', 'your_username')
    password = os.getenv('KAFKA_PASSWORD', 'your_password')
    topic = os.getenv('KAFKA_TOPIC', 'your_topic')
    message = os.getenv('KAFKA_MESSAGE', 'Default message')

    # 尝试连接到Kafka集群并发送消息
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

