import socket
import json

def check_connectivity(host, port):
    try:
        with socket.create_connection((host, port), timeout=10):
            return True
    except (socket.timeout, socket.error):
        return False

def lambda_handler(event, context):
    host = event.get('host', 'example.com')
    port = event.get('port', 80)  # 默认端口设置为 80
    
    is_connected = check_connectivity(host, port)
    
    if is_connected:
        response = {
            'statusCode': 200,
            'body': json.dumps({
                'host': host,
                'port': port,
                'message': 'Successfully connected'
            })
        }
    else:
        response = {
            'statusCode': 400,
            'body': json.dumps({
                'host': host,
                'port': port,
                'message': 'Failed to connect'
            })
        }
    
    return response
