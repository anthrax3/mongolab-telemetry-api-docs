# Alert Definitions CRUD

Using the Telemetry Service, it is possible to create and maintain **alert definitions** : criteria around a metric
value that you specify for a particular deployment.  (For brevity, we will sometimes use "alert" as a synonym for "alert
definition" here.)

For each active alert definition, the Telemetry system will monitor the relevant metric values continuously, and will
generate notifications whenever that value series goes outside the specified criteria for the resource(s) in that deployment.

This document describes the API for programmatic control of these alert definitions via the RESTful HTTPS endpoint at the following URI :

**<center>https://telemetry-api.mongolab.com/v0/alerts</center>**

Using the standard HTTP request verbs, this endpoint affords the Creation, Retrieval, Update, and Deletion ("CRUD") operations for Alert Definition objects.  Each is described in detail below.



## Create a new alert definition

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
        min: NUMBER
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


* ```notificationChannel```: *string | string[] (optional)* - the name(s) of the notification channel(s) to which alerts will be sent. If this is not specified, all channels belonging to the API Key's account will receive notifications for this alert.
* ```condition```: *object* — Describes the conditions under which the alert should be triggered. For metric alerts, the alerting
condition is specified using these fields in a nested structure value:
    * ```metric```: *string* — Specifies the unique ID of the metric to whose values these thresholds will be applied. (See the [metrics glossary](metrics-glossary.md) for some common values of this field.)
    * ```max```: *number (optional)* — the largest "ok" value the metric may take on; if missing or null, no maximum is enforced.
    * ```min```: *number (optional)* — the smallest "ok" value the metric may take on; if missing or null, no minimum is enforced.
    * Note that while both `min` and `max` are optional, at least one of them must be supplied to form a valid condition.

#### response

If successful, the response will contain the new alert definition that was created, including an inlined reference to the specified deployment resource, an indication of the notification channel(s) that will be used, and the generated `_id` value if applicable.



## Update an existing alert definition

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



## Retrieve an alert definition

To retrieve an alert definition using its unique ID, use a **GET** request, appending the alert's unique ID as the URI's final path component.


#### request 

```
GET https://telemetry-api.mongolab.com/v0/alerts/:id
```

#### response

If such an alert exists, it will be included in the response.  If not, a 404 status will be returned.



## Retrieve a group of alert definitions

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



## Delete an alert definition

To delete an alert definition, use a **DELETE** request, appending the alert's unique ID as the URI's final path component.

#### request

```
DELETE https://telemetry-api.mongolab.com/v0/alerts/:id
```

#### response 

If no such alert exists, a 404 status will be returned.
