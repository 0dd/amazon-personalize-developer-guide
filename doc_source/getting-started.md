# Getting started<a name="getting-started"></a>

 The following sections help you get started using Amazon Personalize with the Amazon Personalize console, AWS CLI, and AWS SDKs\. You can choose to get started with a Domain dataset group or a Custom dataset group: 
+  Domain dataset groups provide resources that are optimized for different use cases based on your domain\. To get started creating a Domain dataset group, complete the [Getting started prerequisites](gs-prerequisites.md) and then complete the tutorial in [Getting started with a Domain dataset group](getting-started-domain.md)\. 
+  Custom dataset groups allow you to create and configure custom resources\. To get started providing personalized movie recommendations for your users with a Custom dataset group and the User\-Personalization recipe, complete the [Getting started prerequisites](gs-prerequisites.md) and then start the tutorials in [Getting started with a Custom dataset group](getting-started-custom.md)\. 

When you finish the getting started exercise, to avoid incurring unnecessary charges, follow the steps in [Cleaning up resources](gs-cleanup.md) to delete the resources you created\. 

The tutorials use historical data that consists of 100,000 movie ratings on 9,700 movies from 600 users\.

**To simplify this guide:**
+ We rely on the fact that a user saw a movie and not on what they rated the movie\. This simplifies the preparation of the training data\.
+ We don't record live user interaction events\. For information on capturing user events, see [Recording events](recording-events.md)\.

**Topics**
+ [Getting started prerequisites](gs-prerequisites.md)
+ [Getting started with a Domain dataset group](getting-started-domain.md)
+ [Getting started with a Custom dataset group](getting-started-custom.md)
+ [Cleaning up resources](gs-cleanup.md)