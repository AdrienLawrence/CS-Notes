# Overview

- An Application Program Interface, or an 'API', is a liason between the client-side frontend of an application, and the server-side backend of an application

- It works like this: 
    User -> API -> Server, 
    and then:
    Server -> API -> User

- APIs exist for a variety of reasons, including abstraction, security, and efficiency

- They give users what they need, and nothing they don't. This limits server usage and prevents vulnerable data from being present to the user

- APIs also act as an outlet for various other frontend plugs to plug into. They allow a server to work with various user applications

- They work with requests and responses. A user sends a request to an API, who sends a request to a server. Then the server sends a response to the API, who sends the response to the user

- APIs usually speak in a language called HTTP (Hypertext Transfer Protocol), which is the shared language the client and server can both speak. Every API request is sent with this protocol. 

Note: HTTPS is the current adaptation of HTTP, with more encryption and end-to-end security measures

- Every API request specifies two things:

    A URL (Uniform Resource Locator): Where the requested resource is

    and an HTTP Method: What you plan to do with the resource

The four main HTTP methods are as follows:

- GET: Retrieves data from a server without changing resources

- POST: Creates a new resource

- PUT/PATCH: Modifies an existing resource. PUT modifies the entire resource, PATCH modifies part of it

- DELETE: Deletes an existing resource

The URL identifies the specific resource (data object) you want to interact with

- Ex. 
    '/users' - A collection of all users

    '/users/123' A specific user with the ID '123'

Example HTTP requests using URLs: 

    'GET /users' - Retrieves a list of all users
    
    'POST /users' - Creates a new user (addition to users list)

    'GET /users/123' - Retrieves the user with the ID '123'

    'PUT /users/123' - Updates the user with the ID '123'

    'DELETE /users/123' - Deletes the user with the ID '123'

- The combination of an HTTP method with a URL noun is the foundation of RESTful API design

# HTTP Requests and Responses

- A full HTTP request consist of three parts and multiple lines

Part One: The request line

- This section is the main request line, as seen in 'Overview'

- It is typically a one-line command

- It uses the structure 'METHOD /path HTTP/Version' 

- Ex. ('GET /users HTTP/1.1')

Part Two: The headers

- This section contains Key-Value pairs that provide request metadata

- Example pairs include:
    Host: The server 
    (Ex. 'www.google.com')

    Content-Type: The data format sent by the client
    (Ex. 'application/json')

    Authorization: Bearer 'token' 
    (Ex. 'Bearer A1B2C3D45...')

    User-Agent: Client software info 
    (Ex. 'Chrome/71.0')

    Content-Length: The length of the requested content in bytes 
    (Ex. '1024')

    Status: The HTTP Status Code 
    (Ex. '200 OK')

    Accept-Language: Clients preferred human language
    (Ex. 'en-US,en;q=0.9')

    Accept: The data format received by the client, ';q=x' after a data type is how strongly preferred a format is, where x is a number less than 1, which is the default max preference
    (Ex. 'application/json')
    
    Connection: Controls where the connection remains open after request
    (Ex. 'keep-alive')

Part Three: The body

- This part is optional, only required for requests which send client data to the server (POST, PUT, PATCH)

- It follows an empty line after the headers

- Formatting depends on content type. For a simple content type like plain text the body is simply text

- The most common content type is JSON (JavaScript Object Notation) which is human-readable language used to transmit structured data

- JSON bodies are formatted like this: 

    {
        "firstName": "Jane",
        "lastName": "Doe",
        "email": "jane.doe@example.com"
    }

- JSON is built with key-value pairs, which is also what HTTP headers use

- If 'Content-Type: multipart/form-data' (multiple bodies of different content types), then bodies are formatted like this:

    --123456789
    Content-Disposition: form-data; name="firstName"

    John
    --123456789
    Content-Disposition: form-data; name="photo"; filename="profile.jpg"
    Content-Type: image/jpeg

- For each content type, bodies are broken up by new headers and the same empty line between header and body. Each body needs a 'Content-Disposition:' header

- Full GET Request Example:

    GET /docs/index.html HTTP/1.1
    Host: www.docs.google.com
    User-Agent: Chrome/123.0.0.0
    Accept: text/html,application/xhtml+xml,application,xml;q=0.9,image/avif,
    Accept-Language: en-US,en;q=0.9
    Connection: keep-alive

- Full POST Request Example: 

    POST /submit-form HTTP/1.1
    Host: www.example.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 30

    name=John+Doe&email=john%40example.com

- Full PUT Request Example:

    PUT /users/123 HTTP/1.1
    Host: api.example.com
    Content-Type: application/json
    Content-Length: 45

    {"name": "Jane Doe", "email": "jane@example.com"}

- Full DELETE Request Example:

    DELETE /users/123 HTTP/1.1
    Host: api.example.com

- A full HTTP response consist of roughly the same three parts, but the request line is a response line formatted like this: 

    'HTTP/Version Status_Code Status_Text'

    Ex. 'HTTP/1.1 200 OK' or 'HTTP/1.1 404 Not Found'

- POST requests are typically the sole "non-idempotent" request types. This means identical requests (spamming, bots, etc.) can result in different responses for each duplicate request. This is because POST is the only request which can produce different outcomes if repeated, usually creating duplicate resources or performing duplicate actions

- GET, PUT, and DELETE requests can only work ONE way. There is only one way to GET or DELETE a resource, and replacing a resource with itself in duplicate PUT requests doesn't do anything since its identical data

- Some APIs support an 'Idempotency-Key:' header which acts as an ID for POST requests, such that identical POST requests will share the same ID and not be accepted by the server

- Rate Limiting also prevents duplicate requests, which protects against abuse or accidental spam

Status Code Format: 

- 1xx (Informational): Request recieved, continuing process

- 2xx (Success): The action was accepted by the server

- 3xx (Redirection): Further action required to complete request

- 4xx (Client Error): The request contains bad syntax or cannot be fulfilled

- 5xx (Server Error): The server failed to fulfill a valid request

Common Status Codes:

- 200 OK: Standard success response for GET requests

- 201 Created: New resource sucessfully created (POST)

- 204 No Content: Successful, but no content to send back (DELETE)

- 400 Bad Request: Invalid syntax, server could not understand

- 401 Unauthorized: The client is missing or has invalid credentials

- 403 Forbidden: The client does not have permission to access the resource

- 404 Not Found: The server cannot find the requested resource

- 429 Too Many Requests: The user has been rate limited

- 500 Internal Server Error: Generic error for a server encountering an unexpected condition

- 502 Bad Gateway: The server acting as a gateway got an invalid response from an upstream server

- 503 Service Unavailable: The server cannot handle the request (maintenance or overload)

# Authentication and Authorization

- Authentication (AuthN): the verification of identity ("Who are you?")

- Authorization (AuthZ): the verification of permissions ("What are you allowed to do?")

API Keys:

- API Keys: A common, simple method of authenticating administrative clients. It is a randomly generated string of characters issued by the API provider

- API Key Format: 
    In request header: 'X-API-Key: -insert API Key-'

- Query Parameter: 'https://api.example.com/data?api_key=your_key'

- The server checks the API Key for its existence, status, and scope

- If checks pass for the request, the server returns a '200 OK' response 

- If checks fail (invalid or not permissed), the server returns a '401 Unauthorized' or '403 Forbidden' error response

- An API Key is not a password, it is a token. This means it is generated, not user-written. 

Note: Never embed an API Key in Client-side code, this defeats the purpose. They must be stored on server-side

Bearer Tokens (JWT): 

- A bearer token is a token of permission given to a client after credentials have been validated. It grants access to the API simply through possession

- A bearer token is generated through the server and given to the client, where it is incorporated in further request headers

- The server still verifies the token for every request. If it is invalid, a '401 Unauthorized' error response occurs

- Bearer tokens are typically JWT (JSON Web Token) format, which are structured like this:

    'header.payload.signature'

- The header is the token type and signing algorithm

- The payload includes user identity, such as permissions or roles

- The signature is used to verify the sender

- Because bearer tokens are self-contained, they can be used by anyone, meaning a leaker bearer token can lead to someone unauthorized gaining authorized access

- A Username/Password combination gives a user a bearer token for the remainder of their session, which is why you do not have to login every time you do something after an initial login

OAuth 2.0: 

- The core mechanism of OAuth 2.0 is delegating authorization to another platform with reputable measures in place. This allows you to validate yourself through a different platform and receive permissions for the application you're using

- These are all of the 'Sign in with:' options prevalent across web applications

- 'Sign in with Google', 'Sign in with Facebook', and 'Sign in with Microsoft' are some of the most common examples

- This centralizes your identity to a single account (Usually a Google account), allowing for easier access of resources and control for the user

- The 'Sign in with:' option is typically referred to as SSO (Single Sign On), where you can use one account to sign in to many unrelated websites 

- SSO is safe due to your SSO data being private to web applications. They can only see that you used SSO successfully. Your password is only visible to the SSO provider, meaning the web app never sees your password. Instead the web app receives an access token and some basic profile information

Key Roles in OAuth 2.0:

- Resource Owner: The user (you)

- Client: The application of usage (The website you're on)

- Resource Server: The API that holds your data (Google Cloud Storage)

- Authorization Server: The server that handles authentication and tokens (Google Login)

Note: The Resource Server and Authorization Server are typically held by the same entity (Google, Facebook, etc.)

Grant Types (Flows): 

- Authorization Code Grant: Used for web apps with a backend server. This is 'Sign in With:' flow

- Implicit Grant (Deprecated): A former flow for client-side apps. Insecure

- Authorization Code with PKCE: The modern, secure flow for mobile apps and single page web apps. Adds cryptographs

- Client Credential Grants: For machine-to-machine communication, where the SSO provider verifies your web app's identity 

- Resource Owner Password Credentials Grant: A flow that uses a username and password, discouraged due to web app vulnerabilities

# REST

- REST stands for Representational State Transfer, which is a set of principles and constraints that are used to design an API, not a standard or protocol

- An API that adheres to REST is considered a RESTful API

- RESTful APIs usually use HTTP as their communication protocol because HTTP has similar principles

The Five Architectural Constraints:

1. Client-Server Architecture 

    - Separation of Concerns - The client and server are independent of each other

    - The client is responsible for the user interface

    - The server is responsible for data storage, business logic, and security

    - They are scaled and developed independently

2. Statelessness

    - The server DOES NOT store any client context between requests, which means every request is independent of each other, not maintaining a single state which evolves with more context

    - The server cannot rely on stored 'session' information (The server cannot remember any authentication info)

    - Authentication and authorization must be done with every request using a token (JWT) header, as opposed to only using the token on one request and it being remembered by the API for future requests

3. Cacheability

    - Responses from the server MUST define themselves as cacheable or non-cacheable

    - This allows clients to store and reuse responses for identical future requests, because the request does not have to be fully processed, the client can simply 'remember' the response without going back to the server

    - This improves performance and reduces server load

    - HTTP headers such as 'Cache-Control: max-age=3600' denote cacheability

4. Uniform Interface

    - A RESTful system is resource-based, meaning everything is a resource identified by a URI (Ex. '/users', '/users/123', '/orders/456/items')

    - Manipulation through Representation: Clients interact with resources through representations (JSON, XML, etc.), such as receiving a JSON representation of a resource and updating it by sending back a modified JSON representation

    - Each request/response must describe the representation type so it may be processed correctly. This is done with HTTP methods, status codes, and media types (Ex. 'Content-Type: application/json')

    - HATEOAS (Hypermedia as the Engine of Application State) - The API should guide clients, with easy hyperlink navigation to possible actions instead of hardcoded URL's, similar to navigation links or related page links on modern web applications

    - Ex. 
    ```JSON
    {
        "id": 123,
        "name": "Jane Doe",
        "email": "jane@example.com",
        "_links": {
            "self": { "href": "/users/123" },
            "update": { "href": "/users/123", "method": "PUT" },
            "delete": { "href": "/users/123", "method": "DELETE" }
        }
    }
    ```
    The "_links" element adds actions for the client to take next

5. Layered System

    - The architecture is composed of multiple hierarchical layers (Ex. security >> load balancers >> caches >> API gateways)

    - The client is ignorant of which layer it is connected to, whether an end server or an intermediary, which improves scalability and security

    - This also allows intermediaries (inner-layers) to address cross-cutting concerns (authentication at a gateway, caching at a proxy, etc.) without presenting themselves to the client

- Some APIs follow four constraints and fail to fully implement HATEOAS. While they are commonly referred to as 'RESTful', these are actually 'REST-like' as HATEOAS is explicitly a required constraint by the founder of REST, Roy Fielding, who developed REST in his 2000 dissertation

- The Richardson Maturity Model (RMM) measures how RESTful an API really is. The levels are as shown: 

    Level 0: One endpoint (the same single API reference for every request), runs functions instead of using HTTP methods on resources, functions are called with POST, limiting use of other HTTP methods

    Level 1: Multiple endpoints, now models resources but still uses POST every time to call functions to interact with these resources

    Level 2: Utilizes HTTP methods, status codes, and four of the main REST constraints, omitting HATEOAS (REST-like APIs)

    Level 3: Fully RESTful, embracing every core constraint including HATEOAS, which includes links to possible next actions, making the API self-explanatory and discoverable

# GraphQL

- GraphQL is a query language for APIs. Its fundamental difference is moving from 'imperative' to 'declarative' data fetching. It is built for HTTP/1.1 but can also be used with HTTP/2

- REST by comparison is mostly imperative and Server-driven. Clients details an explicit query and REST structure allows the API to respond with what the query brings ("Do this")

- GraphQL is declarative and Client-driven, meaning clients details the specific data the query SHOULD bring and allows the API to determine how to get it ("Get me this")

- Requests using GraphQL are typically POST requests with a query in the body

- Ex. 
    Request:
    ```GraphQL
    POST /graphql
    Content-Type: application/json
    Accept: application/json

    {
        "query": "{ user(id: 1) {
            name
            email
        } }"
    }
    ```
    Response:
    ```JSON
    {
        "data": {
            "user": {
                "name": "Jane Doe",
                "email": "jane@example.com"
            }
        }
    }
    ```

- GraphQL APIs expose a single endpoint (typically '/graphql')

- APIs using GraphQL are built on a schema that defines the types of data available and their relationships

- GraphQL is harder to cache with standard HTTP caching, and it can also lead to complex queries making it more efficient and flexible for the client than REST, but more straining for the server

- GraphQL also uses mutations to write data and subscriptions for real-time updates via websocket

- A good way to think about it is that REST uses a single request, with the logic in the request, while GraphQL uses an outer (POST) request to reference the API, and an inner request with the embedded logic

# gRPC

- gRPC stands for Google Remote Procedure Call. It's a high performance API framework built on HTTP/2

- gRPC is method-oriented, as opposed to resource-oriented like REST

- RPC-First: the API has designated services and methods clients can call remotely

- Data is in compact binary form (protobuf) and transported over HTTP/2. Because of this, gRPC is smaller, faster, and more efficient than HTTP/1.1 based APIs like REST and GraphQL

- The API is defined in a .proto file which lists all possible services and methods

- Client calls methods like normal functions, the framework handles serialization and transport

- Ex. 'response = user_service.GetUser(UserRequest(id=1))'

- Messages and services are strongly typed by the .proto schema, meaning they must be written in a valid way or the code will throw compile-time errors 

There are four call patterns gRPC supports:

1. Unary: Single request -> Single response (like a normal function)

2. Server Streaming: Single request -> Stream of responses

3. Client Streaming: Stream of requests -> Single response

4. Bidirectional Streaming: Stream of requests -> Stream of responses

- Interceptors and middleware can hook into calls for logging, authentication, retries, etc.

- Browsers do not support raw gRPC. gRPC-Web or proxies are required. gRPC is mainly for server-to-server communication for large entities


# Exponential Backoff

- Exponential backoff is a retry strategy used for requests that fail due to rate limits, network errors, server overload, or other reasons

- The crux of exponential backoff is that retries for failed requests do not happen immediately or at a fixed interval. Instead the retry happens after a time period relative exponentially to the number of retries that have already taken place

- Ex. 'wait_time = base_delay * (2 ^ retry_attempt)'

- After an initial wait time, the wait time grows exponentially for each new attempt, as opposed to linearly ('base_delay * (2 * retry_attempt)' instead)

- Maximum delays are often set as well as randomization variables (Ex. 'wait_time = random(0, ...)') to prevent many delayed retries from random users happen concurrently

- Exponential backoff is used for three main reasons:
    
    - One, it reduces server load.

    - Two, it increases success probability for failures due to transient issues that need time to resolve

    - Three, it prevents network congestion for distributed systems or microservices