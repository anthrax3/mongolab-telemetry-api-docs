# Alert Definitions CRUD

Using the Telemetry Service, it is possible to create and maintain [**alert definitions**](#alert-definitions) : criteria around a metric
value that you specify for a particular deployment.  (For brevity, we will sometimes use "alert" as a synonym for "alert
definition" here.)

For each active alert definition, the Telemetry system will monitor the relevant metric values continuously, and will
generate notifications whenever that value series goes outside the specified criteria for the resource(s) in that deployment.

The receivers of these notifications are defined by [**notification channels**](#notification-channels), which describe email addresses, PagerDuty services, or other points of contact that would be interested in hearing about these alerts.  These channels can be reused by multiple alerts, and each alert may be configured to send notifications to any number of notification channels.

## Alert Definitions

This section describes the API for programmatic control of alert definitions via the RESTful HTTPS endpoint at the following URI :

**<center>https://telemetry-api.mongolab.com/v0/alerts</center>**

Using the standard HTTP request verbs, this endpoint affords the Creation, Retrieval, Update, and Deletion ("CRUD") operations for Alert Definition objects.  Each is described in detail below.

### Create a new alert definition

To create a new alert definition, **POST** a JSON document describing it.

#### request

```
POST https://telemetry-api.mongolab.com/v0/alerts
```

#### body

```
{
    tags: { 
        TAG_NAME : TAG_VALUE ,
        ... 
    },
    deployment: DEPLOYMENT_ID,
    filter: FILTER_ID,
    notificationChannel: CHANNEL_ID | [ CHANNEL_ID, ... ] ,
    condition: { 
        metric: METRIC_ID,
        max: NUMBER,
        min: NUMBER,
        minCapacitanceInMinutes: NUMBER
    },
}
```

* ```tags```: *object (optional)* — User-defined free-form fields.  Tag names must be alphanumeric, may contain hyphens ('-') or underscores ('_'), and must not start with a number.  A tag value can be any string.  A tag's name and value must each comprise fewer than 1024 characters.
* ```deployment```: *string* — Specifies the ID of the root deployment resource (replica set cluster or sharded cluster) to which this alert definition is attached. All servers belonging to this deployment (subject to ```filter```, if given) will produce alert events based on the given ```condition```. 
* ```filter```: *string (optional)* — By default the alert will apply to the deployment and all its constituent resources. If the alert
only applies to a subset of those resources, a filter may be specified to narrow this scope. The set of available filters currently comprises:

    | `filter:` value | nodes selected |
    | --------------- | -------------- |
    | `"SERVER_ROLE_MONGOD_PRIMARY"` | Primary data node(s) |
    | `"SERVER_ROLE_MONGOD_SECONDARY"` | Secondary data node(s) |
    | `"SERVER_ROLE_MONGOD_DATA"` | Any data node |
    | `"SERVER_ROLE_CONFIG_MONGOD"` | Config servers (in a sharded cluster) |
    | `"SERVER_ROLE_MONGOS"` | Mongos routers (for a sharded cluster) |


* ```notificationChannel```: *string | string[] (optional)* - the id(s) of the notification channel(s) to which alerts will be sent. If this is not specified, all channels belonging to the API Key's account will receive notifications for this alert.  See the [Notification Channels](#notification-channels) section below for information on managing the channels in your account.
* ```condition```: *object* — Describes the conditions under which the alert should be triggered. For metric alerts, the alerting
condition is specified using these fields in a nested structure value:
    * ```metric```: *string* — Specifies the unique ID of the metric to whose values these thresholds will be applied. (See the [metrics glossary](metrics-glossary.md) for some common values of this field.)
    * ```max```: *number (optional)* — the largest "ok" value the metric may take on; if missing or null, no maximum is enforced.
    * ```min```: *number (optional)* — the smallest "ok" value the metric may take on; if missing or null, no minimum is enforced.
    * Note that while both `min` and `max` are optional, at least one of them must be supplied to form a valid condition.
    * ```minCapacitanceInMinutes```: *number (optional)* - the minimum amount of time the metric value must remain continuously outside the set thresholds before an alert is triggered, or remain inside them before a triggered alert may be automatically cleared. The smallest permitted value is 3 minutes, which is also the default if this is not specified.

#### response

If successful, the response will contain the new alert definition that was created, including an inlined reference to the specified deployment resource, an indication of the notification channel(s) that will be used, and the generated `_id` value if applicable.



### Update an existing alert definition

To modify a previously defined alert definition, **PUT** a new JSON document describing it, appending the alert's unique ID as the URI's final path component.

#### request

```
PUT https://telemetry-api.mongolab.com/v0/alerts/:id
```

#### body

The JSON document in the request body is grammatically identical to the body described for the **POST** endpoint above.  However, two fields are handled specially during an update operation:

The `_id` field in the request body is optional; however, if included, it must match the id that appears in the URL, or else a client error response code will be returned.  

The `deployment` field in the body is optional but, if given, must match the deployment information already present in the stored alert definition. That is, an update via **PUT** is not permitted to alter the deployment to which the alert is attached.

#### response

If successful, the response will contain the entire updated alert definition, including the proper `_id` field.  If the specified
alert definition does not exist, a 404 status will be returned.



### Retrieve an alert definition

To retrieve an alert definition using its unique ID, use a **GET** request, appending the alert's unique ID as the URI's final path component.


#### request 

```
GET https://telemetry-api.mongolab.com/v0/alerts/:id
```

#### response

If such an alert exists, it will be included in the response.  If not, a 404 status will be returned.



### Retrieve a group of alert definitions

To retrieve a set of alert definitions matching zero or more constraining query parameters, use a **GET** request with a query string specifying those parameters.

#### request

```
GET https://telemetry-api.mongolab.com/v0/alerts?QUERY
```

#### query parameters

All query parameters are optional.

* ```deployment=DEPLOYMENT_ID```: Find alerts that apply to the given deployment ID. If this parameter is
omitted, alerts for all deployments (permitted under acess control rules) will be returned.
* ```tags={"TAG_NAME":"TAG_VALUE", ...}```: Find alerts with the given tag(s).  If multiple tags are specified, only
alerts that have _all_ indicated tag values will be returned.



### Delete an alert definition

To delete an alert definition, use a **DELETE** request, appending the alert's unique ID as the URI's final path component.

#### request

```
DELETE https://telemetry-api.mongolab.com/v0/alerts/:id
```

#### response 

If no such alert exists, a 404 status will be returned.



## Notification Channels

This section describes the API for programmatic control of notification channels via the RESTful HTTPS endpoint at the following URI :

**<center>https://telemetry-api.mongolab.com/v0/notification-channels</center>**

Using the standard HTTP request verbs, this endpoint affords the Creation, Retrieval, Update, and Deletion ("CRUD") operations for Notification Channel objects.  Each is described in detail below.


### Create a new notificaion channel

To create a new notification channel, **POST** a JSON document describing it.

#### request

```
POST https://telemetry-api.mongolab.com/v0/notification-channels
```

#### body

```
{
    _type: TYPE,
    name: STRING,
    [ <type-specific fields> ]
}
```

* ```_type```: (required) One of ```EmailNotificationChannel```, ```AccountEmailNotificationChannel```,
 ```PagerDutyNotificationChannel```, or ```HipChatNotificationChannel```
* ```name```: (required) A string description of this channel

Other fields will depend on the value of ```_type```:

##### EmailNotificationChannel

Use this channel to send simple email notifications when an alert is opened or closed.

* ```email```: (required) Email address to send notifications to

Example:

```
{
    _type: "EmailNotificationChannel,
    name: "Ops Team",
    email: "ops@mycompany.com"
}
```

##### AccountEmailNotificationChannel

Use this channel to send email notifications to a designated contact in your MongoLab account when an alert is
opened or closed.  Any changes you make to your MongoLab account contacts will take effect immediately.

* ```recipient```: (required) Account contact to which to send email notifications.  Possible values include:
```"technicalContact"```, ```"adminUser"```, ```"emergencyContact"```, or ```"billingContact"```

Example:

```
{
    _type: "AccountEmailNotificationChannel,
    name: "Technical Contact",
    recipient: "technicalContact"
}
```

##### PagerDutyNotificationChannel

Use this channel to create PagerDuty incidents whenever an alert is triggered.  Any incidents created when an alert
is triggered will also automatically be resolved when the alert closes.

* ```serviceKey```: (required) PagerDuty service API key (32-digit hexadecimal string)

Example:

```
{
    _type: "PagerDutyNotificationChannel,
    name: "Emergency Pager",
    serviceKey: "1234567890abcdef1234567890abcdef"
}
```

##### HipChatNotificationChannel

Use this channel to send messages to a HipChat room whenever an alert is opened or closed.

* ```room```: (required) HipChat room ID to send notifications to (integer)
* ```authToken```: (required) HipChat auth token to use for authentication (30-digit hexadecimal string).  HipChat group
admins may create these tokens on the [API Tokens](https://www.hipchat.com/admin/api) page.

Example:

```
{
    _type: "HipChatNotificationChannel,
    name: "Developer Chat Room",
    room: "123456",
    authToken: "1234567890abcdef1234567890abcd"
}
```

#### response

If successful, the response will contain the new notification channel that was created, including a newly generated **_id** value that can be used in alert definitions as well as other notification channel API endpoints.

Example:

```
{
    _id: "c-1234567890abcdef12345678",
    _type: "EmailNotificationChannel,
    name: "Ops Team",
    email: "ops@mycompany.com"
}
```


### Update an existing notification channel

To modify a previously defined notification channel, **PUT** a new JSON document describing it, appending the channel's unique ID as the URI's final path component.

#### request

```
PUT https://telemetry-api.mongolab.com/v0/notification-channels/:id
```

#### body

the JSON document in the request body is grammatically identical to the body described for the **POST**
endpoint above.  However, any `_id` field in the request body is **ignored** in favor of the id that appears in the URL.  

#### response

If successful, the response will contain the entire updated notification channel, including the proper `_id` field.
If the specified channel does not exist, a 404 status will be returned.


### Retrieve a notification channel

To retrieve a notification channel using its unique ID, use a **GET** request, appending the channel's unique ID as the URI's final path component.

#### request 

```
GET https://telemetry-api.mongolab.com/v0/notification-channels/:id
```

#### response

If such a channel exists, it will be included in the response.  If not, a 404 status will be returned.


### Retrieve a group of notification channels

To retrieve a set of notification channels matching zero or more constraining query parameters, use a **GET** request with a query string specifying those parameters.

#### request

```
GET https://telemetry-api.mongolab.com/v0/notification-channels?QUERY
```

#### query parameters

All query parameters are optional.

* ```_type=TYPE```: Find notification channels with the given ```_type```.
* ```name=STRING```: Find notification channels with the given ```name```.
* ```email=STRING```: Find EmailNotificationChannels with the given email.
* ```recipient=STRING```: Find AccountEmailNotificationChannels with the given recipient.
* ```room=STRING```: Find HipChatNotificationChannels with the given room ID.

NOTE: querying by HipChat authToken or PagerDuty serviceKey are currently not supported.

### Delete a notification channel

To delete an notification channel, use a **DELETE** request, appending the channel's unique ID as the URI's final path component.

#### request

```
DELETE https://telemetry-api.mongolab.com/v0/notification-channels/:id
```

#### response 

If no such alert exists, a 404 status will be returned.

