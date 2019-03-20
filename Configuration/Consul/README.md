# Service Configuration
Using Consul for Central Microservice Configuration

## Add key/value

### From dashboard
We can add configuration values from the Consul web UI (dashboard) under the `key/value` tab.

### From Consul CLI
We can add configuration using the Consul CLI:

``` bash
$ consul kv put foo bar
Success! Data written to: foo
```

and we can see in the Consul console logs the following message:

```bash 
consul_1  |     2019/03/20 11:28:43 [DEBUG] http: Request PUT /v1/kv/foo (457.4µs) from=172.19.0.1:60150
```

### From REST API

Request:
```
PUT http://localhost:8500/v1/kv/gin-web/message
Body: Hello Gin Framework.
```

Response:
``` 
Status: 200 OK
Body: true
```

## Retrieve key/value 

### From Consul CLI
``` bash 
$ consul kv get foo
bar
```

### From REST API

Request:
```
GET http://localhost:8500/v1/kv/?keys=
```

Response:
``` 
[
    "foo",
    "gin-web/message"
]
```

#### Display all configuration for gin-web path
Includes the value base64 encoded.

Request:
```
GET http://localhost:8500/v1/kv/gin-web?recurse=
```

Response:
``` 
[
    {
        "LockIndex": 0,
        "Key": "gin-web/message",
        "Flags": 0,
        "Value": "SGVsbG8gR2luIEZyYW1ld29yay4=",
        "CreateIndex": 81,
        "ModifyIndex": 81
    }
]
```

#### Display raw value

Request:
```
GET http://localhost:8500/v1/kv/gin-web/message?raw=true
```

Response:
``` 
Hello Gin Framework.
```

## Update key/value 

### From REST API

Request:
```
PUT http://localhost:8500/v1/kv/gin-web/message
Hello Gin Framework (Updated).
```

Response:
``` 
Status: 200 OK
Body: true
```



## Delete key/value 

### From REST API

Request:
```
DELETE http://localhost:8500/v1/kv/foo
```

Response:
``` 
Status: 200 OK
Body: true
```

