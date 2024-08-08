好的，为了在这个Lambda函数的基础上增加测试端口联通性的方法，并从test event中获取调用的方法，我们可以进行以下调整：

1. **修改Lambda函数代码**：
   - 添加一个新的方法来测试端口联通性。
   - 根据test event中的参数决定调用哪个方法。

```python
import json
import requests
import socket

def lambda_handler(event, context):
    action = event.get('action')
    if action == 'http_request':
        return handle_http_request(event)
    elif action == 'port_check':
        return handle_port_check(event)
    elif action == 'resolve_hostname':
        return handle_resolve_hostname(event)
    else:
        return {
            'statusCode': 400,
            'body': json.dumps('Invalid action specified')
        }

def handle_http_request(event):
    url = event.get('url')
    if not url:
        return {
            'statusCode': 400,
            'body': json.dumps('URL not provided')
        }

    try:
        response = requests.get(url)
        return {
            'statusCode': response.status_code,
            'headers': dict(response.headers),
            'body': response.text
        }
    except requests.exceptions.RequestException as e:
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e),
                'type': type(e).__name__
            })
        }

def handle_port_check(event):
    host = event.get('host')
    port = event.get('port')
    if not host or not port:
        return {
            'statusCode': 400,
            'body': json.dumps('Host or port not provided')
        }

    try:
        with socket.create_connection((host, port), timeout=10) as sock:
            return {
                'statusCode': 200,
                'body': json.dumps('Port is open')
            }
    except (socket.timeout, socket.error) as e:
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e),
                'type': type(e).__name__
            })
        }

def handle_resolve_hostname(event):
    hostname = event.get('hostname')
    if not hostname:
        return {
            'statusCode': 400,
            'body': json.dumps('Hostname not provided')
        }

    try:
        ip_address = socket.gethostbyname(hostname)
        return {
            'statusCode': 200,
            'body': json.dumps({'hostname': hostname, 'ip_address': ip_address})
        }
    except socket.error as e:
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e),
                'type': type(e).__name__
            })
        }
