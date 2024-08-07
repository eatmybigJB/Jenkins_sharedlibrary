好的，为了详细地返回信息，包括在非200状态码情况下的错误原因，我们可以修改Lambda函数的代码。修改后的代码会返回HTTP状态码、响应体以及请求的详细信息（例如Headers）。以下是更新后的代码和步骤。

### 1. 编写Lambda函数代码
创建或修改`lambda_function.py`文件，包含以下代码：

```python
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
```

### 2. 创建依赖包
由于Lambda函数中使用了`requests`库，因此需要打包依赖库。使用以下步骤创建依赖包并打包：

```sh
mkdir package
pip install requests -t package/
cd package
zip -r ../lambda_function.zip .
cd ..
zip -g lambda_function.zip lambda_function.py
```

### 3. 部署Lambda函数
使用AWS CLI来创建或更新这个Lambda函数：

```sh
aws lambda create-function --function-name myLambdaFunction \
    --zip-file fileb://lambda_function.zip --handler lambda_function.lambda_handler \
    --runtime python3.8 --role arn:aws:iam::<your-account-id>:role/<your-lambda-execution-role>
```

如果Lambda函数已经存在，可以使用以下命令更新函数代码：

```sh
aws lambda update-function-code --function-name myLambdaFunction --zip-file fileb://lambda_function.zip
```

### 4. 创建测试事件
在AWS Lambda控制台中，创建一个测试事件，例如：

```json
{
    "url": "https://www.example.com"
}
```

### 5. 测试Lambda函数
你可以在AWS Lambda控制台中测试该函数，或者使用以下CLI命令进行测试：

```sh
aws lambda invoke --function-name myLambdaFunction --payload '{"url": "https://www.example.com"}' response.json
cat response.json
```

### 解释代码
- `lambda_handler` 函数接收事件和上下文参数。
- 从事件中提取URL，如果没有提供URL，则返回400错误。
- 使用 `requests.get` 请求URL，如果请求成功，返回状态码、响应头和响应体。
- 如果发生异常，返回500错误以及错误的详细信息（错误类型和错误消息）。

这样，无论HTTP请求成功与否，Lambda函数都能够详细地返回请求的结果和错误信息。