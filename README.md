# Lambda Application 
<img src="file:///Users/hyunbok.lee/Downloads/awslambda.svg" width="200" height="200" /


### 라이브러리

```
import json
import urllib.parse
import boto3
```

### 변수 
```
target_role = '특정 타겟 Iam Role' 
error_log_name = 'Error 문구'
source_email = 'AWS SES 등록 Email'
target_email = 'cloud-noti@kakaobank.com'

```

### 코드 구현
1. 타겟 s3에서 이벤트를 받기 때문에 Lambda_Handler 함수 구현 
```
def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name'] # Records 0번째 bucket 항목에 name 값 bucket 변수 저장 
    ob_key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')  
    data = s3.get_object(Bucket=bucket, Key=ob_key) 
    contents = data['Body'].read().decode('utf-8')
```
S3 이벤트 예제 
```
{
  "Records": [
    {
      "eventVersion": "2.0",
      "eventSource": "aws:s3",
      "awsRegion": "us-west-2",
      "eventTime": "1970-01-01T00:00:00.000Z",
      "eventName": "ObjectCreated:Put",
      "userIdentity": {
        "principalId": "EXAMPLE"
      },
      "requestParameters": {
        "sourceIPAddress": "127.0.0.1"
      },
      "responseElements": {
        "x-amz-request-id": "EXAMPLE123456789",
        "x-amz-id-2": "EXAMPLE123/5678abcdefghijklambdaisawesome/mnopqrstuvwxyzABCDEFGH"
      },
      "s3": {
        "s3SchemaVersion": "1.0",
        "configurationId": "testConfigRule",
        "bucket": {
          "name": "my-s3-bucket",
          "ownerIdentity": {
            "principalId": "EXAMPLE"
          },
          "arn": "arn:aws:s3:::example-bucket"
        },
        "object": {
          "key": "HappyFace.jpg",
          "size": 1024,
          "eTag": "0123456789abcdef0123456789abcdef",
          "sequencer": "0A1B2C3D4E5F678901"
        }
      }
    }
  ]
}
```

2. 비인가 데이터 검색 조건 및 SES 메일 전송 
  error_log_name: 'NoSuchBucketPolicy', target_role: '타겟 Iam Role Name'이 AND 조건으로 'contents'에 포함되면 
  AWS SES 메일 전송이 됩니다. 
  
```
    if error_log_name and target_role in contents:
        print("비인가 접근 탐지")
        ses_client = boto3.client('ses')
        CHARSET = "UTF-8"
        response = ses_client.send_email(
         Destination={
            "ToAddresses": [
                target_email,
            ],
        },
        Message={
            "Body": {
                "Text": {
                    "Charset": CHARSET,
                    "Data": "비정상 행위 발생",
                }
            },
            "Subject": {
                "Charset": CHARSET,
                "Data": "비인가 접근 탐지",
            },
        },
        Source=source_email,)
```
