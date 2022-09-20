# Google-Workspace-APIs
Demonstration of the different Google Workspace APIs and their methods.

Here I will create a compilation of most of the methods available for most (if not all) the Google Workspace APIs, this  is for demonstration purposes and to provide a simple source for code snippets.

I will try to provide as much detail as possible for every method such as prerequisites and possible use cases.

## Alert Center API

The Alert Center API retrieves all the information related to events in the organization. Using this API facilitates fetching notifications about potential issues or risks within the organization.

Calling this API with a service account configured with domain-wide delegation results in the following entry under the OAuth Log events with the details of the action (Application ID: "{Service account unique ID}", Application name: "{App name | service account email address}", Event: "Grant", Description: "{User name} authorized access to {App name | service account email address} for https://www.googleapis.com/auth/apps.alerts scopes". Note: the 'App name' is the name of the application set in the OAuth consent screen of the project containing the service account.

The alerts fetched through this API will be the same as the ones visible in the admin console, meaning that Admins can only view (and manage through APIs) alerts related only to rules they have admin console privileges for, as mentioned in this Help Center article https://support.google.com/a/answer/9105276#privileges. Trying to fetch details or take action upon an inaccessible alert will return the following error message:
**<HttpError 403 when requesting https://alertcenter.googleapis.com/v1beta1/alerts/{alertId}?alt=json returned "The caller does not have permission". Details: "The caller does not have permission">**

This API can be called using a service account with domain-wide delegation and a service account using a custom role assigned to it. 

***getSettings*** method returns the current settings and creates an entry in the Admin Audit logs with information of the event (Event: "Customer Settings Retrieved", Description: "Alert center customer settings retrieved"). 

***updateSettings*** method returns the new settings after the update is applied and an entry is created in the Admin Audit logs with the details of that event (Event: "Customer Settings Retrieved", Description: "Alert center customer settings retrieved").

Although the **notifications** property is an array currently it is not possible to set multiple **cloudPubsubTopic** objects. Trying to make the API call with more than one **cloudPubsubTopic** object returns: **<HttpError 400 when requesting https://alertcenter.googleapis.com/v1beta1/settings?alt=json returned "Request contains an invalid argument.". Details: "Request contains an invalid argument.">**

Sample getSettings & updateSettings output:

no configurations set
```
{ }
```

configurations are set
```
{
  "notifications":[
     {
        "cloudPubsubTopic":{
           "topicName":"projects/projectID/topics/AlertCenterAPITest",
           "payloadFormat":"JSON"
        }
     }
  ]
}
```

More details about the structure of the Pub/Sub notification is available at
https://developers.google.com/admin-sdk/alertcenter/guides/notifications

This is a redacted output sample from the Cloud Pub/Sub API projects.subscriptions.pull method (https://cloud.google.com/pubsub/docs/reference/rest/v1/projects.subscriptions/pull):

```
{
  "receivedMessages": [
    {
      "ackId": "some_ack_id",
      "message": {
        "data": {AttributesInBase64},
        "attributes": {
          "alertcenter_payload_format": "JSON",
          "alertcenter_create_time": {events.get.createTime},
          "alertcenter_end_time": {events.get.endTime},
          "alertcenter_resource_type": "ALERT",
          "alertcenter_update_time": {events.get.createTime},
          "alertcenter_type": {events.get.type},
          "alertcenter_resource_status": "CREATED",
          "alertcenter_start_time": {events.get.startTime},
          "alertcenter_source": {events.get.source}
        },
        "messageId": "some_message_id",
        "publishTime": "YYYY-MM-DDTHH:MM:SS.000000Z"
      }
    }
  ]
}
```


_**alerts.list**_ method returns a list of notification alerts in the Admin console. Each alert may be a different type and will have different properties. See possible types at https://developers.google.com/admin-sdk/alertcenter/reference/alert-types. Calling this method creates an entry in the Admin Audit logs with the details of that action (Event: "Alert Center Viewed", Description: "Alert center details of alert {alertId},{alertId},...,{alertId},{alertId} viewed").


**_alerts.get_** method returns the specified alert with an alert ID, the alert ID Admin console can be fetched by going to https://admin.google.com/ac/ac/alert/details?{alertId}. Each alert may be a different type and will have different properties. See possible types at https://developers.google.com/admin-sdk/alertcenter/reference/alert-types. Calling this method creates an entry in the Admin Audit logs with the details of that action (Event: "Alert Center Viewed", Description: "Alert center details of alert {alertId} viewed").

Sample **alerts.get** output:
```
{
   "customerId":"00aabbcc",
   "alertId":"aaaabbbb-1111-2222-3333-cccceeeeffff",
   "createTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
   "startTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
   "endTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
   "type":"{alertType}",
   "source":"{alertSource}",
   "data":{
      "@type":"type.googleapis.com/google.apps.alertcenter.type.{alertType}",
      "field#"...
   },
   **"deleted":true**,
   "metadata":{
      "customerId":"00aabbcc",
      "alertId":"aaaabbbb-1111-2222-3333-cccceeeeffff",
      "status":"NOT_STARTED",
      "updateTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
      "severity":"HIGH",
      "etag":"someEtagValue"
   },
   "updateTime":"2022-07-09T21:24:52.611178Z",
   "etag":"_xLXTBB3FJQ="
}
```

The **"deleted":true** property is only available for alerts affected by the alerts.delete method.

Screenshot of the Admin console Alert Center after an alert type ssoProfileCreatedEvent is fetched:
![image](https://user-images.githubusercontent.com/9749627/191214054-bea65cfe-eaef-4116-9d78-634a27cbe6c1.png)


_**alerts.getMetadata**_ method returns only the metadata field of a specified alert with an alert ID. Calling this method creates an entry in the Admin Audit logs with the details of that action (Event: "Alert Metadata Retrieved", Description: "Alert center metadata was retrieved for alert {alertId}").

Sample **getMetadata** output:
```
{
   "customerId":"00aabbcc",
   "alertId":"aaaabbbb-1111-2222-3333-cccceeeeffff",
   "status":"NOT_STARTED",
   "updateTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
   "severity":"HIGH",
   "etag":"someEtagValue"
}
```


**_alerts.delete_** method sets the **"deleted"** property to **true** for an alert with a specific alert ID. Calling this method creates an entry in the Admin Audit logs with the details of that action (Event: "Alert Deleted", Description: "Alert center alert {alertId} deleted"). Calling this method on an alert pending deletion creates a new entry in the Admin Audit logs. Trying to delete an inaccessible alert returns a 403 "The caller does not have permission" error.


_**alerts.undelete**_ method removes the **"deleted":true** property for an alert with a specific alert ID. Calling this method does NOT create an entry in the Admin Audit logs.

Screenshot of the Admin console Alert Center after an alert is deleted and restored.

![image](https://user-images.githubusercontent.com/9749627/191211880-cb7f45a3-994c-4634-8c20-e006f9adc733.png)


_**alerts.batchDelete**_ method sets the **"deleted"** property to **true** for a batch of alerts with a specific alert ID. Calling this method creates an entry in the Admin Audit logs with the details of that action (Event: "Batch Delete Operation Performed", Description: "Alert center batch delete operation performed on {alertId},{alertId},{alertId}") regardless if the action was successful or not. Calling this method on an alert pending deletion creates a new entry in the Admin Audit logs. Trying to delete an inaccessible alert returns a 403 "The caller does not have permission" error. 


_**alerts.batchUndelete**_ method removes the **"deleted":true** property for an alert with a specific alert ID. Calling this method does NOT create an entry in the Admin Audit logs.

Sample **batchDelete** & **batchUndelete** output (for a call with two accessible alertIds and one inaccessible alertId):
```
{
   "successAlertIds":[
      "{alertId}",
      "{alertId}"
   ],
   "failedAlertStatus":{
      "{alertId}":{
         "code":7,
         "message":"PERMISSION_DENIED"
      }
   }
}
```


_**alerts.feedback.list**_ method returns a list of feedback for a notification alert in the Admin console Alert Center. Calling this method creates an entry in the Admin Audit logs with the details of that action (Event: "Listed Feedback of an Alert", Description: "Alert center listed feedback ids {alertFeedbackId},{alertFeedbackId},...,{alertFeedbackId},{alertFeedbackId} of alert {alertId}").

Sample alerts.feedback.list output:
```
[

   {
      "customerId":"00aabbcc",
      "alertId":"{alertId}",
      "feedbackId":"{alertFeedbackId}",
      "createTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
      "type":"VERY_USEFUL",
      "email":"{adminEmailAddress}"
   },
   {
      "customerId":"00aabbcc",
      "alertId":"{alertId}",
      "feedbackId":"{alertFeedbackId}",
      "createTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
      "type":"NOT_USEFUL",
      "email":"{adminEmailAddress}"
   },
   {
      "customerId":"00aabbcc",
      "alertId":"{alertId}",
      "feedbackId":"{alertFeedbackId}",
      "createTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
      "email":"{adminEmailAddress}"
   }
]
```

The **"type"** property is only available for alert feedback created with an enum type different to ALERT_FEEDBACK_TYPE_UNSPECIFIED, other values will show the property.


_**alerts.feedback.create**_ method returns the created feedback for a specified alert with an alert ID. Calling this method creates an entry in the Admin Audit logs with the details of that action (Event: "Feedback Written", Description: "Alert center feedback {alertFeedbackType} written for alert {alertId}  (alert_feedback_id: {alertFeedbackId})").

Sample **alerts.feedback.create** output:
```
{
   "customerId":"00aabbcc",
   "alertId":"{alertId}",
   "feedbackId":"{alertFeedbackId}",
   "createTime":"YYYY-MM-DDTHH:MM:SS.000000Z",
   "type":"VERY_USEFUL",
   "email":"{adminEmailAddress}"
}
```
