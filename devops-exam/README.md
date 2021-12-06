
### Elastic Beanstalk
- 每個環境都具備指向負載平衡器的 CNAME (URL)。本環境即具備 URL (如 myapp.us-west-2.elasticbeanstalk.com)。 可透過 CNAME 來將 `myapp.us-west-2.elasticbeanstalk.com` 成 `www.yourweb.com`.
  - 直接API
  - 保留的config
  - `.ebxtensions`
  - default vaule. 


### API Gateway Basic Integration Request with lambda.

![截圖 2021-12-03 下午5 33 16](https://user-images.githubusercontent.com/46012524/144579708-7b8b3162-609f-405f-b6cb-cb5cc01942eb.png)

#### API Gateway Settings
*Integration Request*
- Mapping Templates
```bash
##  See http://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html
##  This template will pass through all parameters including path, querystring, header, stage variables, and context through to the integration endpoint via the body/payload
#set($allParams = $input.params())
{
"body-json" : { "lanuch": $input.json('$.lanuch'), "count" : $input.json('$.count')  },
"params" : {
#foreach($type in $allParams.keySet())
    #set($params = $allParams.get($type))
"$type" : {
    #foreach($paramName in $params.keySet())
    "$paramName" : "$util.escapeJavaScript($params.get($paramName))"
        #if($foreach.hasNext),#end
    #end
}
    #if($foreach.hasNext),#end
#end
},
"stage-variables" : {
#foreach($key in $stageVariables.keySet())
"$key" : "$util.escapeJavaScript($stageVariables.get($key))"
    #if($foreach.hasNext),#end
#end
},
"context" : {
    "account-id" : "$context.identity.accountId",
    "api-id" : "$context.apiId",
    "api-key" : "$context.identity.apiKey",
    "authorizer-principal-id" : "$context.authorizer.principalId",
    "caller" : "$context.identity.caller",
    "cognito-authentication-provider" : "$context.identity.cognitoAuthenticationProvider",
    "cognito-authentication-type" : "$context.identity.cognitoAuthenticationType",
    "cognito-identity-id" : "$context.identity.cognitoIdentityId",
    "cognito-identity-pool-id" : "$context.identity.cognitoIdentityPoolId",
    "http-method" : "$context.httpMethod",
    "stage" : "$context.stage",
    "source-ip" : "$context.identity.sourceIp",
    "user" : "$context.identity.user",
    "user-agent" : "$context.identity.userAgent",
    "user-arn" : "$context.identity.userArn",
    "request-id" : "$context.requestId",
    "resource-id" : "$context.resourceId",
    "resource-path" : "$context.resourcePath"
    }
}
  ```

#### Lambda Code

```python
import json

def lambda_handler(event, context):
    
    body = event.get('body-json')
    
    new_body = ''
    
    if body.get('lanuch') == '':
        body['lanuch'] = 'English'
        new_body = body
    else:
        new_body = body
    
    
    return {
        'statusCode': 200,
        'body': json.dumps(new_body)
    }

```


### Code Deploy Lifecycle Event Hooks in CodeDeploy
![image](https://user-images.githubusercontent.com/46012524/144706571-a9c4f7e3-c47d-43af-a4b9-327b3daf0734.png)


### How Amazon EC2 Auto Scaling works with CodeDeploy
In order for CodeDeploy to deploy your application revision to new EC2 instances during an Auto Scaling scale-out event, CodeDeploy uses an Auto Scaling lifecycle hook. The lifecycle hook notifies CodeDeploy that an Auto Scaling scale-out event is in progress and that CodeDeploy must deploy a revision to the scaled-out instances.


#### After CodeDeploy adds the lifecycle hook, how is it used?
![image](https://user-images.githubusercontent.com/46012524/144707006-3d2110c6-315a-4d09-a298-80de423742fd.png)

#### After the lifecycle hook is installed, it is used during scale-out events. A scale-out event unfolds as follows:

1. The Auto Scaling service (or simply, Auto Scaling) determines that a scale-out event needs to occur, and contacts the EC2 service to launch a new EC2 instance.

2. The EC2 service launches a new EC2 instance. The instance moves into the `Pending` state, and then into the `Pending:Wait` state.

3. During `Pending:Wait`, Auto Scaling runs all the lifecycle hooks attached to the Auto Scaling group, including the lifecycle hook created by CodeDeploy.

4. The lifecycle hook sends a notification to the Amazon SQS queue that is polled by CodeDeploy.

5. Upon receiving the notification, CodeDeploy parses the message, performs some validation, and starts to deploy your application to the new EC2 instance using the last successful revision.

6. While the deployment is running, CodeDeploy sends heartbeats every five minutes to Auto Scaling to let it know that the instance is still being worked on.

7. So far, the EC2 instance is still in the `Pending:Wait` state.

8. When the deployment completes, CodeDeploy indicates to Auto Scaling to either `CONTINUE` or `ABANDON` the EC2 launch process, depending on whether the deployment succeeded or failed.
    - If CodeDeploy indicates `CONTINUE`, Auto Scaling continues the launch process, either waiting for other hooks to complete, or putting the instance into the `Pending:Proceed` and then the `InService` state.

    - If CodeDeploy indicates `ABANDON`, Auto Scaling terminates the EC2 instance, and restarts the launch procedure if needed to meet the desired number of instances, as defined in the Auto Scaling Desired Capacity setting.



### AWS Elastic Beanstal's model?
- Applications have many environments, environments have many deployments.

### Dynamodb
Q. What are the Global Secondary Key features of DynamoDB? 
A. The partition key and sort key can be different from the table. (分區鍵和排序鍵可以與表不同。)

Q. What is true of the Local Secondary Key attributes in DynamoDB?
A. Only the sort key can be different from the table. (與基表具有相同分區鍵但排序鍵不同的索引)

Local Secondary Indexes (本地二級索引)只能通過 Query API 查詢

#### DynamoDB TTL
- DynamoDB 生存時間 (TTL) 啟用每個項目的時間戳，以確定何時不再需要項目。
- 在指定時間戳的日期和時間之後，DynamoDB 從表中刪除項目，而不會消耗任何寫入吞吐量。
- 從表中刪除的 ite 也以與 DeleteItem 操作相同的方式從任何本地二級索引和全局二級索引中刪除。
- DynamoDB Stream tracks 追蹤的是系統刪除的動作，不是一般的刪除動作。
- TTL 擅長用於，儲存 item 後過不久就不需要使用的 item。
   - 在應用程序中一年不活動後刪除用戶或傳感器數據。
   - 通過 DynamoDB Streams 和 AWS Lambda 將過期項目存檔到 S3 數據湖。 
   - 根據合同或監管義務，將敏感數據保留一段時間。 以上三點都適合使用 DynamoDB TTL。
   - Cloud Watch Event 沒辦法檢查 DynamoDB Event。
Q. 一家制造公司将所有遥测数据存储在 Amazon DynamoDB 表中。在写入数据后，这些项目不会发生变化。为了降低存储成本，公司必须长期归档所有数据。不过，必须为应用程序提供 60 天以内的数据。
DevOps 工程师应采取哪些组合步骤以最经济高效地将数据移动到归档存储？ （选择三项。）

A. Table enable TTL, 分配一個包含 60天時間戳的屬性，將該屬性設定為 TTL，Table enable DynamoDB Streams ，創建 Lambda Function 輪詢 DynamoDB Streams，將紀錄寫進 Kinesis Data Firehose stream flow，創建 Kinesis Data Firehose stream ，將Data 寫入S3 bucket 中，並設定S3 生命週期policy，以便在第零天時將data 歸檔到 S3 Glacier Deep Archice。


### Service Qutoas and Organizations 集成
但必須先啟用受信任的訪問，透過Cloud watch alarm and sns 進行告警。

### Cloudformation
Q. How should you get near-real-time CloudFormation stack status updates in a continuous delivery system?
A. Use NotificationARNs.member.N when making a CreateStack call to push stack events into SNS in nearly real-time.

Q. Which stack state in AWS CloudFormation rejects UpdateStack calls?
A. When a stack is in the UPDATE_ROLLBACK_FAILED state, you can continue rolling it back to return it to a working state (to
UPDATE_ROLLBACK_COMPLETE). You cannot update a stack that is in the UPDATE_ROLLBACK_FAILED state. However, if you can continue to roll it back, you can return the stack to its original settings and try to update it again.

Q. Which of the following is not an AWS CloudFormation Pseudo Parameter?
A. This is the complete list of Pseudo Parameters: AWS::AccountId, AWS::NotificationARNs, AWS::NoValue, AWS::Region, AWS::StackId, AWS::StackName.


### IAM
Q. A user is creating an IAM user policy. Which of the following alternatives corresponds to the policy's valid version?
A. "Version":"2012-10-17" is now, 2008-10-17 is earilier version.

### AWS OpsWorks
Q. Which of the following is not an instance type that may be allocated in a stack tier when considering AWS OpsWorks?
A. Spot instances


### SQS
Q. By default, how long are messages retained on a SQS queue?
A. The SQS message retention period is configurable and can be set anywhere from 1 minute to 2 weeks. The default is 4 days and once the message retention limit is reached your messages will be automatically deleted. The option for longer message retention provides greater flexibility to allow for longer intervals between message production and consumption.

### AWS Inspector
Q. Which command initiates the evaluation process?
A. aws inspector start-assessment-run --assessment-template-arn<template-arn>
