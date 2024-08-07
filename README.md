＇＇＇python
import json
import requests

def lambda_handler(event, context):
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