# Recording events<a name="recording-events"></a>

 With both Domain dataset group and Custom dataset group, Amazon Personalize can make recommendations based on real\-time *[event](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html#event)* data only, historical event data only \(see [Importing bulk records](bulk-data-import.md)\), or a mixture of both\. Record events in real\-time so Amazon Personalize can learn from your user’s most recent activity and update recommendations as they use your application\. This keeps your interactions data fresh and improves the relevance of Amazon Personalize recommendations\.

 You can record real\-time events using the AWS SDKs, AWS Amplify or AWS Command Line Interface \(AWS CLI\)\. When you record events, Amazon Personalize appends the event data to the Interactions dataset in your dataset group\. 

**Note**  
AWS Amplify includes a JavaScript library for recording events from web client applications, and a library for recording events in server code\. For more information, see [Amplify \- analytics](https://aws-amplify.github.io/docs/js/analytics)

**Topics**
+ [Requirements for recording events and training a model](#recording-events-requirements)
+ [How real\-time events influence recommendations](#recorded-events-influence-recommendations)
+ [Creating an event tracker](#event-get-tracker)
+ [PutEvents operation](#event-record-api)
+ [Recording impressions data](#putevents-including-impressions-data)
+ [Event metrics](#event-metrics)
+ [Events and solutions](#event-solutions)
+ [Sample Jupyter notebook](#recording-events-sample-notebook)

## Requirements for recording events and training a model<a name="recording-events-requirements"></a>

To record events, you need the following:
+ A dataset group that includes an `Interactions` dataset, which can be empty\. If you went through the [Getting started](getting-started.md) guide, you can use the same dataset group and dataset that you created\. For information on creating a dataset group and a dataset, see [Preparing and importing data](data-prep.md)\.
+ An event tracker
+ A call to the [ PutEvents ](API_UBS_PutEvents.md) operation\.

You can start out with an empty Interactions dataset and, when you have recorded enough data, train the model using only new recorded events\. The minimum data requirements to train a model are:
+ 1000 records of combined interaction data \(after filtering by `eventType` and `eventValueThreshold`, if provided\)
+ 25 unique users with at least 2 interactions each

## How real\-time events influence recommendations<a name="recorded-events-influence-recommendations"></a>

 Once you create a campaign \(for custom solutions\) or a recommender \(for Domain dataset group\), Amazon Personalize automatically uses new recorded event data for existing items \(items you included in the data you used to train the latest model or create the latest recommender\) when generating recommendations for a user\. This does not require retraining the model \(unless you are using the SIMS or Popularity\-Count recipes for custom solutions\)\. 

Instead, Amazon Personalize adds the new recorded event data to the user's history\. Amazon Personalize then uses the modified data when generating recommendations for the user \(and this user only\)\.
+  For recorded events for *new items* \(items you did not include in the data you used to train the model\), if you created recommenders for Top picks for you and Recommended for you use cases \(for Domain dataset group\) or if you trained your model \(solution version\) with User\-Personalization, Amazon Personalize automatically updates the model every two hours\. After each update completes, the new items influence recommendations\. For information about auto updates for the User\-Personalization recipe, see [User\-Personalization recipe](native-recipe-new-item-USER_PERSONALIZATION.md)\. 

   For any other domain use case, Amazon Personalize will automatically train new models for your recommenders every 7 days, starting from the recommender creation date\. For any other custom recipe, you must re\-train the model for the new records to influence recommendations\. Amazon Personalize stores recorded events for new items and, once you create a new solution version \(train a new model\), this new data will influence Amazon Personalize recommendations for the user\. 
+  For recorded events for *new users* \(users that were not included in the data you used to create recommenders or solution versions\), recommendations will initially be for popular items only\. Recommendations will be more relevant as you record more events for the user\. Amazon Personalize stores the new user data and will include the user when Amazon Personalize updates your recommender or when you manually train a new solution version\. 

   For new, anonymous users \(users without a userId\), Amazon Personalize uses the `sessionId` you pass in the [ PutEvents ](API_UBS_PutEvents.md) operation to associate events with the user before they log in\. This creates a continuous event history that includes events that occurred when the user was anonymous\. 

## Creating an event tracker<a name="event-get-tracker"></a>

An *[event tracker](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html#event-tracker)* specifies a destination dataset group for new event data\. You create an event tracker with the Amazon Personalize console or the [ CreateEventTracker ](API_CreateEventTracker.md) API operation\. You pass as a parameter the Amazon Resource Name \(ARN\) of the dataset group that contains the target Interactions dataset\. For instructions on creating an event tracker using the Amazon Personalize console, see [Creating an event tracker \(console\)](importing-interactions.md#event-tracker-console)\. 

 An event tracker includes a *tracking ID*, which you pass as a parameter when you use the [PutEvents](https://docs.aws.amazon.com/personalize/latest/dg/API_UBS_PutEvents.html) operation\. Amazon Personalize then appends the new event data to the Interactions dataset of the dataset group you specify in your event tracker\. 

**Note**  
You can create only one event tracker for a dataset group\.

------
#### [ Python ]

```
import boto3

personalize = boto3.client('personalize')

response = personalize.create_event_tracker(
    name='MovieClickTracker',
    datasetGroupArn='arn:aws:personalize:us-west-2:acct-id:dataset-group/MovieClickGroup'
)
print(response['eventTrackerArn'])
print(response['trackingId'])
```

The event tracker ARN and tracking ID display, for example:

```
{
    "eventTrackerArn": "arn:aws:personalize:us-west-2:acct-id:event-tracker/MovieClickTracker",
    "trackingId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

------
#### [ AWS CLI ]

```
aws personalize create-event-tracker \
    --name MovieClickTracker \
    --dataset-group-arn arn:aws:personalize:us-west-2:acct-id:dataset-group/MovieClickGroup
```

The event tracker ARN and tracking ID display, for example:

```
{
    "eventTrackerArn": "arn:aws:personalize:us-west-2:acct-id:event-tracker/MovieClickTracker",
    "trackingId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

------
#### [ SDK for Java 2\.x ]

```
public static String createEventTracker(PersonalizeClient personalizeClient, 
                                      String eventTrackerName, 
                                      String datasetGroupArn) {
        
    String eventTrackerId = null;
    String eventTrackerArn = null;
    long maxTime = 3 * 60 * 60; 
    long waitInMilliseconds = 30 * 1000;
    String status;
    
    try {
        CreateEventTrackerRequest createEventTrackerRequest = CreateEventTrackerRequest.builder()
            .name(eventTrackerName)
            .datasetGroupArn(datasetGroupArn)
            .build();
       
        CreateEventTrackerResponse createEventTrackerResponse = 
            personalizeClient.createEventTracker(createEventTrackerRequest);
        
        eventTrackerArn = createEventTrackerResponse.eventTrackerArn();
        eventTrackerId = createEventTrackerResponse.trackingId();
        
        System.out.println("Event tracker ARN: " + eventTrackerArn);
        System.out.println("Event tracker ID: " + eventTrackerId);

        maxTime = Instant.now().getEpochSecond() + maxTime;

        DescribeEventTrackerRequest describeRequest = DescribeEventTrackerRequest.builder()
            .eventTrackerArn(eventTrackerArn)
            .build();
        
        while (Instant.now().getEpochSecond() < maxTime) {

            status = personalizeClient.describeEventTracker(describeRequest).eventTracker().status();
            System.out.println("EventTracker status: " + status);

            if (status.equals("ACTIVE") || status.equals("CREATE FAILED")) {
                break;
            }
            try {
                Thread.sleep(waitInMilliseconds);
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
        return eventTrackerId;
    }
    catch (PersonalizeException e){
        System.out.println(e.awsErrorDetails().errorMessage());
        System.exit(1);
    }
    return eventTrackerId;
}
```

------

## PutEvents operation<a name="event-record-api"></a>

To record events, you call the [ PutEvents ](API_UBS_PutEvents.md) operation\. The following example shows a `PutEvents` operation that passes one event that contains the minimum required information\. The corresponding Interactions schema is shown, along with an example row from the Interactions dataset\.

Your application generates the `sessionId` when a user first visits your website or uses your application\. For example, you could generate a universally unique identifier \(UUID\) and use this value for the `sessionId`\. Amazon Personalize uses the `sessionId` to associate events with the user before they log in \(is anonymous\)\. After the user logs in and you send an event including the `userId`, Amazon Personalize associates the previously anonymous historical event data with their `userId` by matching the `sessionId`\. This creates a continuous event history that includes events that occurred when the user was anonymous\. We recommend having at most one user mapped to one `sessionID`\. 

The event list is an array of [ Event ](API_UBS_Event.md) objects\. An `eventType` is required for each event, but in this example, `eventType` data is not used in training because it is not included in the schema\. You can provide a placeholder value to satisfy the requirement\. 

The `userId`, `itemId`, and `sentAt` parameters map to the USER\_ID, ITEM\_ID, and TIMESTAMP fields of a corresponding historical `Interactions` dataset\. For more information, see [Datasets and schemas](how-it-works-dataset-schema.md)\.

**Corresponding interactions schema**

```
Interactions schema: USER_ID, ITEM_ID, TIMESTAMP
Interactions dataset: user123, item-xyz, 1543631760
```

**Code example**

------
#### [ Python ]

```
import boto3

personalize_events = boto3.client(service_name='personalize-events')

personalize_events.put_events(
    trackingId = 'tracking_id',
    userId= 'USER_ID',
    sessionId = 'session_id',
    eventList = [{
        'sentAt': TIMESTAMP,
        'eventType': 'eventTypePlaceholder',
        'itemId': 'ITEM_ID'
        }]
)
```

------
#### [ AWS CLI ]

```
aws personalize-events put-events \
    --tracking-id tracking_id \
    --user-id USER_ID \
    --session-id session_id \
    --event-list '[{
        "sentAt": "TIMESTAMP",
        "eventType": "eventTypePlaceholder",
        "itemId": "ITEM_ID"
      }]'
```

------
#### [ SDK for Java 2\.x ]

```
public static void putEvents(PersonalizeEventsClient personalizeEventsClient, 
                            String trackingId, 
                            String sessionId, 
                            String userId, 
                            String itemId) {
    
    try { 
        Event event = Event.builder()
            .sentAt(Instant.ofEpochMilli(System.currentTimeMillis() + 10 * 60 * 1000))
            .itemId(itemId)
            .eventType("typePlaceholder")
            .build();

        PutEventsRequest putEventsRequest = PutEventsRequest.builder()
            .trackingId(trackingId)
            .userId(userId)
            .sessionId(sessionId)
            .eventList(event)
            .build();

        int responseCode = personalizeEventsClient.putEvents(putEventsRequest)
            .sdkHttpResponse()
            .statusCode();
        System.out.println("Response code: " + responseCode);

        } catch (PersonalizeEventsException e) {
            System.out.println(e.awsErrorDetails().errorMessage());
        }
}
```

------

After this example, you would proceed to train a model using only the required properties\.

The next example shows how to submit data that would train on the event value\. It also demonstrates the passing of multiple events of different types \('like' and 'rating'\)\. In this case, you must specify the event type to train on in the [ CreateSolution ](API_CreateSolution.md) operation \(see **Events and Solutions** below\)\. The example also shows the recording of an extra property, `numRatings`, that is used as metadata by certain recipes\.

```
Interactions schema: USER_ID, ITEM_ID, TIMESTAMP, EVENT_TYPE, EVENT_VALUE, NUM_RATINGS
Interactions dataset: user123, movie_xyz, 1543531139, rating, 5, 12
                      user321, choc-ghana, 1543531760, like, 4
                      user111, choc-fake, 1543557118, like, 3
```

------
#### [ Python ]

```
import boto3
import json

personalize_events = boto3.client(service_name='personalize-events')

personalize_events.put_events(
    trackingId = 'tracking_id',
    userId= 'user555',
    sessionId = 'session1',
    eventList = [{
        'eventId': 'event1',
        'sentAt': '1553631760',
        'eventType': 'like',
        'properties': json.dumps({
            'itemId': 'choc-panama',
            'eventValue': 4,
            'numRatings': 0    
            })
        }, {
        'eventId': 'event2',
        'sentAt': '1553631782',
        'eventType': 'rating',
        'properties': json.dumps({
            'itemId': 'movie_ten',
            'eventValue': 3,
            'numRatings': 13
            })
        }]
)
```

------
#### [ AWS CLI ]

```
aws personalize-events put-events \
    --tracking-id tracking_id \
    --user-id user555 \
    --session-id session1 \
    --event-list '[{
        "eventId": "event1",
        "sentAt": "1553631760",
        "eventType": "like",
        "properties": "{\"itemId\": \"choc-panama\", \"eventValue\": \"true\"}"
      }, {
        "eventId": "event2",
        "sentAt": "1553631782",
        "eventType": "rating",
        "properties": "{\"itemId\": \"movie_ten\", \"eventValue\": \"4\", \"numRatings\": \"13\"}"
      }]'
```

------
#### [ SDK for Java 2\.x ]

```
public static void putMultipleEvents(PersonalizeEventsClient personalizeEventsClient, 
                            String trackingId, 
                            String sessionId, 
                            String userId, 
                            String event1Type, 
                            Float event1Value, 
                            String event1ItemId,
                            int event1NumRatings,
                            String event2Type, 
                            Float event2Value, 
                            String event2ItemId,
                            int event2NumRatings) {  
                                                    
    ArrayList<Event> eventList = new ArrayList<Event>();
                        
    try {
        Event event1 = Event.builder()
            .eventType(event1Type)
            .sentAt(Instant.ofEpochMilli(System.currentTimeMillis() + 10 * 60 * 1000))
            .itemId(event1ItemId)
            .eventValue(event1Value)
            .properties("{\"numRatings\": "+ event1NumRatings +"}")
            .build(); 

        eventList.add(event1); 

        Event event2 = Event.builder()
            .eventType(event2Type)
            .sentAt(Instant.ofEpochMilli(System.currentTimeMillis() + 10 * 60 * 1000))
            .itemId(event2ItemId)
            .eventValue(event2Value)
            .properties("{\"numRatings\": "+ event2NumRatings +"}")
            .build();

        eventList.add(event2);

        PutEventsRequest putEventsRequest = PutEventsRequest.builder()
            .trackingId(trackingId)
            .userId(userId)
            .sessionId(sessionId)
            .eventList(eventList)
            .build();

        int responseCode = personalizeEventsClient.putEvents(putEventsRequest)
            .sdkHttpResponse()
            .statusCode();
            
        System.out.println("Response code: " + responseCode);

    } catch (PersonalizeEventsException e) {
        System.out.println(e.awsErrorDetails().errorMessage());
    }
}
```

------

**Note**  
The properties keys use camel case names that match the fields in the Interactions schema\. For example, if the field 'NUM\_RATINGS' is defined in the Interactions schema, the property key should be `numRatings`\.

## Recording impressions data<a name="putevents-including-impressions-data"></a>

If you use the [User\-Personalization](native-recipe-new-item-USER_PERSONALIZATION.md) recipe or add the IMPRESSIONS field to your schema for a dataset in a Domain dataset group, you can record impressions data in your PutEvents operation\. Impressions are lists of items that were visible to a user when they interacted with \(for example, clicked or watched\) a particular item\. Amazon Personalize uses impressions data to guide exploration, where recommendations include items with less interactions data or relevance\. For information on the *implicit* and *explicit* impressions Amazon Personalize can model, see [Impressions data](interactions-datasets.md#interactions-impressions-data)\. 

**Important**  
If you provide conflicting implicit and explicit impression data in your `PutEvents` requests, Amazon Personalize uses the explicit impressions by default\.

To record the Amazon Personalize recommendations you show your user as impressions data, include the `recommendationId` in your [ PutEvents ](API_UBS_PutEvents.md) request and Amazon Personalize derives the implicit impressions based on your recommendation data\.

To manually record impressions data for an event, list the impressions in the [ PutEvents ](API_UBS_PutEvents.md) command's `impression` input parameter\. The following code sample shows how to include a `recommendationId` and an `impression` in a PutEvents operation with either the SDK for Python \(Boto3\) or the SDK for Java 2\.x\. If you include both, Amazon Personalize uses the explicit impressions by default\.

------
#### [ SDK for Python \(Boto3\) ]

```
import boto3

personalize_events = boto3.client(service_name='personalize-events')

personalize_events.put_events(
    trackingId = 'tracking_id',
    userId= 'userId',
    sessionId = 'sessionId',
    eventList = [{
        'eventId': 'event1',
        'eventType': 'rating',
        'sentAt': 1553631760,
        'itemId': 'item id',
        'recommendationId': 'recommendation id',
        'impression': ['itemId1', 'itemId2', 'itemId3']
        }]
)
```

------
#### [ SDK for Java 2\.x ]

Use the following `putEvents` method to record an event with impressions data and a recommendationId\. For the impressions parameter, pass the list of itemIds as an ArrayList\.

```
public static void putEvents(PersonalizeEventsClient personalizeEventsClient, 
                                String trackingId, 
                                String sessionId, 
                                String userId, 
                                String eventType, 
                                Float eventValue, 
                                String itemId,
                                ArrayList<String> impressions,
                                String recommendationId) {

    try { 
        Event event = Event.builder()
            .eventType(eventType)
            .sentAt(Instant.ofEpochMilli(System.currentTimeMillis() + 10 * 60 * 1000))
            .itemId(itemId)
            .eventValue(eventValue)
            .impression(impressions)
            .recommendationId(recommendationId)
            .build();

        PutEventsRequest putEventsRequest = PutEventsRequest.builder()
            .trackingId(trackingId)
            .userId(userId)
            .sessionId(sessionId)
            .eventList(event)
            .build();

        int responseCode = personalizeEventsClient.putEvents(putEventsRequest)
            .sdkHttpResponse()
            .statusCode();
        System.out.println("Response code: " + responseCode);

    } catch (PersonalizeEventsException e) {
        System.out.println(e.awsErrorDetails().errorMessage());
    }
}
```

------

## Event metrics<a name="event-metrics"></a>

To monitor the type and number of events sent to Amazon Personalize, use Amazon CloudWatch metrics\. For more information, see [Monitoring Amazon Personalize](personalize-monitoring.md)\. 

## Events and solutions<a name="event-solutions"></a>

When training a model that uses event data, two parameters of the [ CreateSolution ](API_CreateSolution.md) operation are relevant\. The `eventType` parameter must be specified when multiple event types are recorded\. The `eventType` indicates which type of event Amazon Personalize uses as the label for model training\.

The `eventValueThreshold` parameter of the `SolutionConfig` object creates an event filter\. When this parameter is specified, only events with a value greater than or equal to the threshold are used for training the model\. You must specify the event type when using `eventValueThreshold`\.

## Sample Jupyter notebook<a name="recording-events-sample-notebook"></a>

For a sample Jupyter notebook that shows how to use Amazon Personalize to react to real\-time behavior of users using an event tracker and the [ PutEvents ](API_UBS_PutEvents.md) operation, see [2\.View\_Campaign\_And\_Interactions\.ipynb](https://github.com/aws-samples/amazon-personalize-samples/blob/master/getting_started/notebooks/2.View_Campaign_And_Interactions.ipynb) in the **getting\_started** folder of the [amazon\-personalize\-samples](https://github.com/aws-samples/amazon-personalize-samples) GitHub repository\. 