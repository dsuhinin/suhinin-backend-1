# Test Task Service.

# Topics
* [Common Info](#common-info)
* [Prerequisites](#Prerequisites)
* [Set up and run service](#set-up-and-run-service)
* [Environment Variables](#environment-variables)
* [Endpoints](#endpoints) 
  * [Auth Endpoints](#server-endpoints)
    * [POST /auth/signin](#post-authsignin)
    * [POST /auth/signup](#post-authsignup)
    * [GET /auth/signout](#get-authsignout)
  * [Member Endpoints](#member-endpoints)
    * [GET /members](#get-members)
  * [Service Endpoints](#service-endpoints)
    * [POST /service/status](#get-serviceinfo)
    * [GET /service/info](#get-servicestatus)
* [Appendix A. Response codes](#appendix-a-response-codes)
* [Appendix B. Request headers](#appendix-b-request-headers)

# Common Info
- I've spent around 4 hours to build everything from the scratch, the `backend` part.
- 1 hour I've spent to cover service little bit with `unit` and `integration` tests and prepare everything in Docker. 
  Test coeverage not 100% but `integration` tests cover each endpoint `happy path`
- aproximately 1 hour I've spent for `frontend` part to recall it little bit for myself and build UI. It isn't ideal unfortunately.
- Added additional endpoints for `health` checks so service is ready for `Kubernetes` integration.
- Integrated DI container. It helps to decouple all things and make services fully testable and well extandable.
- Decided to use `Makefile` to simplify some commands to `run`, `test`, `lint` code.
- Integrated `golangci-lint` to fully lint service code. If you will some errors when you run `make lint` then I think 
  this tool has been updated and it is possible to see new linter errors.
- Divided code to several layers like: `transport`, `controller`, `validator`, `dao`, etc.

# Prerequisites
- Golang
- MySQL
- Docker
- Make

# Set up and run service

- clone the repository from GitHub.
```
$ git clone https://github.com/dsuhinin/suhinin-backend-1.git
```
- go to the project folder root.
- run `make docker_build_image service_start` to build and run everything in Docker.
- open in a browser `http://localhost:8081`

- run `make lint` to run linter over the code.
- run `make go_test_unit` to run `unit` tests.
- run `make docker_build_image service_test` to run `integration` tests.

# Environment Variables

| Name          		| Required | Example		| Description	| 
| ----------------------------- | --------- | ----------------- | ------------- |		 
| JWT_KEY       		| Y 	    | A8F5F167F44F  	| The Key to sign JWT tokens.|
| MYSQL_USER     		| Y         | root 		| Database user.|
| MYSQL_PASS 			| Y         | password    	| Database password.|
| MYSQL_HOST 			| Y         | localhost 	| Database host.|
| MYSQL_PORT 			| Y         | 3306 		| Database port.|
| MYSQL_DB_NAME 		| Y         | database_name 	| Database name.|
| SERVER_ADDRESS 		| N         | 127.0.0.1:8080 	| Server address and port where service is listening for incoming requests. If not provided then `127.0.0.1:8080` will be used.|
| LOG_LEVEL 			| Y         | debug 		| Logger level. Possible values are: debug, info, warn, error.|
| CORS_ENABLED 			| N         | true 		| Enables CORS.|
| CORS_ENABLED_DEBUG 	| N         | true 		| Enables CORS debug info.|
| CORS_ALLOWED_ORIGINS 	| N         | http://localhost:8081 		| Provides allowed origings.|
| CORS_ALLOWED_METHODS 	| N         | POST,PUT,PATCH,DELETE 		| Provides allowed http methods.|
| CORS_ALLOWED_HEADERS 	| N         | Content-Type,Authorization 	| Provides allowed http headers.|

# Endpoints
# Auth Endpoints
## POST /auth/signin
Endpoint provides `signin` functionality.

**Request info**

```
HTTP Request method    POST
Request URL            https://localhost:8080/auth/signin
```

**Request body**

```
{
	"email": "user.email@gmail.com",
	"password": "password"
}
```

**Response body**

```
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MTM3NDgxNDAsImp0aSI6IjEiLCJpc3MiOiJzdWhpbmluLmRtaXRyaXlAZ21haWwuY29tIn0.9HUQFb4MlcQVHvXH6XiJ82FhQ3JylmXFHKHL52VuIj4"
}
```


## POST /auth/signup
Endpoint provides `signup` functionality.

**Request info**

```
HTTP Request method    POST
Request URL            https://localhost:8080/auth/signup
```

**Request body**

```
{
    "email": "user.email@gmail.com",
    "password": "password",
    "confirm_password": "password"  
}
```

**Response body**

```
{}
```

## GET /auth/signout
Endpoint provides `signout` functionality.

**Request info**

```
HTTP Request method    GET
Request URL            https://localhost:8080/auth/signout
Headers		       Authorization Bearer token
```

**Request body**

```
{}
```
**Response body**

```
{}
```

# Member Endpoints
## GET /members
Endpoint to get `members` data.

**Request info**

```
HTTP Request method    GET
Request URL            https://localhost:8080/members
Headers		       Authorization Bearer token
```
**Note**
For more information about `Authorization` header [Appendix B. Request headers](#appendix-b-request-headers)

**Request body**

```
{
  "text": "this text could be accessible only in case of success authentication and goes from the service"
}
```

**Response body**

```
{}
```

# Service Endpoints
## GET /service/info
The endpoint to get service info.

**Request info**

```
HTTP Request method    GET
Request URL            https://localhost:8080/service/info
```
**Request body**

```
{}
```

**Response body**

```
{
  "dependencies": {
    "mysql": {
      "status": 200,
      "latency": 0.002757021
    }
  }
}
```
**Note**
Endpoint give the full picture about `service` current state. Endpoint mostly needs for Kubernetes readiness probe.

## GET /service/status
The endpoint to get service status.

**Request info**

```
HTTP Request method    GET
Request URL            https://localhost:8080/service/status
```

**Request body**

```
{}
```

**Response body**
```
{}
```
**Note**
Endpoint give the full picture about `service` current state. Endpoint mostly needs for Kubernetes liveness probe.

# Appendix A. Response codes

### HTTP error codes

Application uses standard HTTP response codes:
```
200 - Success
201 - Created
400 - Request error
401 - Authentication error
404 - Entity not found
500 - Internal Server error
```

Additional information about the error is returned as JSON-object like:
```
{
    "code": "{error-code}",
    "message": "{error-message}"
}
```
# Appendix B. Request headers
HTTP header required for API calls:
- `Authorization` : `Bearer token` - authorization based on `JWT` token.
