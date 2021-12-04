
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
