# CreateRecommender<a name="API_CreateRecommender"></a>

Creates a recommender with the recipe \(a Domain dataset group use case\) you specify\. You create recommenders for a Domain dataset group and specify the recommender's Amazon Resource Name \(ARN\) when you make a [GetRecommendations](https://docs.aws.amazon.com/personalize/latest/dg/API_RS_GetRecommendations.html) request\. 

 **Status** 

A recommender can be in one of the following states:
+ CREATE PENDING > CREATE IN\_PROGRESS > ACTIVE \-or\- CREATE FAILED
+ DELETE PENDING > DELETE IN\_PROGRESS

To get the recommender status, call [ DescribeRecommender ](API_DescribeRecommender.md)\.

**Note**  
Wait until the `status` of the recommender is `ACTIVE` before asking the recommender for recommendations\.

**Related APIs**
+  [ ListRecommenders ](API_ListRecommenders.md) 
+  [ DescribeRecommender ](API_DescribeRecommender.md) 
+  [ UpdateRecommender ](API_UpdateRecommender.md) 
+  [ DeleteRecommender ](API_DeleteRecommender.md) 

## Request Syntax<a name="API_CreateRecommender_RequestSyntax"></a>

```
{
   "datasetGroupArn": "string",
   "name": "string",
   "recipeArn": "string",
   "recommenderConfig": { 
      "itemExplorationConfig": { 
         "string" : "string" 
      }
   }
}
```

## Request Parameters<a name="API_CreateRecommender_RequestParameters"></a>

The request accepts the following data in JSON format\.

 ** [ datasetGroupArn ](#API_CreateRecommender_RequestSyntax) **   <a name="personalize-CreateRecommender-request-datasetGroupArn"></a>
The Amazon Resource Name \(ARN\) of the destination domain dataset group for the recommender\.  
Type: String  
Length Constraints: Maximum length of 256\.  
Pattern: `arn:([a-z\d-]+):personalize:.*:.*:.+`   
Required: Yes

 ** [ name ](#API_CreateRecommender_RequestSyntax) **   <a name="personalize-CreateRecommender-request-name"></a>
The name of the recommender\.  
Type: String  
Length Constraints: Minimum length of 1\. Maximum length of 63\.  
Pattern: `^[a-zA-Z0-9][a-zA-Z0-9\-_]*`   
Required: Yes

 ** [ recipeArn ](#API_CreateRecommender_RequestSyntax) **   <a name="personalize-CreateRecommender-request-recipeArn"></a>
The Amazon Resource Name \(ARN\) of the recipe that the recommender will use\. For a recommender, a recipe is a Domain dataset group use case\. Only Domain dataset group use cases can be used to create a recommender\. For information about use cases see [Choosing recommender use cases](https://docs.aws.amazon.com/personalize/latest/dg/domain-use-cases.html)\.   
Type: String  
Length Constraints: Maximum length of 256\.  
Pattern: `arn:([a-z\d-]+):personalize:.*:.*:.+`   
Required: Yes

 ** [ recommenderConfig ](#API_CreateRecommender_RequestSyntax) **   <a name="personalize-CreateRecommender-request-recommenderConfig"></a>
The configuration details of the recommender\.  
Type: [ RecommenderConfig ](API_RecommenderConfig.md) object  
Required: No

## Response Syntax<a name="API_CreateRecommender_ResponseSyntax"></a>

```
{
   "recommenderArn": "string"
}
```

## Response Elements<a name="API_CreateRecommender_ResponseElements"></a>

If the action is successful, the service sends back an HTTP 200 response\.

The following data is returned in JSON format by the service\.

 ** [ recommenderArn ](#API_CreateRecommender_ResponseSyntax) **   <a name="personalize-CreateRecommender-response-recommenderArn"></a>
The Amazon Resource Name \(ARN\) of the recommender\.  
Type: String  
Length Constraints: Maximum length of 256\.  
Pattern: `arn:([a-z\d-]+):personalize:.*:.*:.+` 

## Errors<a name="API_CreateRecommender_Errors"></a>

 ** InvalidInputException **   
Provide a valid value for the field or parameter\.  
HTTP Status Code: 400

 ** LimitExceededException **   
The limit on the number of requests per second has been exceeded\.  
HTTP Status Code: 400

 ** ResourceAlreadyExistsException **   
The specified resource already exists\.  
HTTP Status Code: 400

 ** ResourceNotFoundException **   
Could not find the specified resource\.  
HTTP Status Code: 400

## See Also<a name="API_CreateRecommender_SeeAlso"></a>

For more information about using this API in one of the language\-specific AWS SDKs, see the following:
+  [ AWS Command Line Interface](https://docs.aws.amazon.com/goto/aws-cli/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for \.NET](https://docs.aws.amazon.com/goto/DotNetSDKV3/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for C\+\+](https://docs.aws.amazon.com/goto/SdkForCpp/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for Go](https://docs.aws.amazon.com/goto/SdkForGoV1/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for Java V2](https://docs.aws.amazon.com/goto/SdkForJavaV2/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for JavaScript](https://docs.aws.amazon.com/goto/AWSJavaScriptSDK/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for PHP V3](https://docs.aws.amazon.com/goto/SdkForPHPV3/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for Python](https://docs.aws.amazon.com/goto/boto3/personalize-2018-05-22/CreateRecommender) 
+  [ AWS SDK for Ruby V3](https://docs.aws.amazon.com/goto/SdkForRubyV3/personalize-2018-05-22/CreateRecommender) 