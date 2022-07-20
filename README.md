# Lambda Application 


## Code 구성 

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
