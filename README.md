# mongolab-telemetry-api-docs

Documents the API for the MongoLab Telemetry Service, 
accessible at https://telemetry-api.mongolab.com/v0


### Client authentication : `Telemetry-API-Key` request header

The Telemetry API will only honor authenticated client requests. 

To satisfy the authentication requirement, each HTTPS request must include an acceptable API key as the value of a specific request header field : `Telemetry-API-Key`.

(The Telemetry API is currently in a limited, invite-only Beta release.  To request a Telemetry API key, please contact [support@mongolab.com](mailto:support@mongolab.com).)

The server will verify the validity of the API key string found in this header value, confirm it exists in the set of currently authorized keys, and further verify that the authorization in question permits access to any deployment resources involved in the request or (where applicable) its response.



## Alert Definitions

Create and maintain rules for alerting on exceptional metric values.  
See [Alert Definitions CRUD](alerts.md).
