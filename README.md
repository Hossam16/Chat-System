# Chat API using Ruby on Rails and Golang with (ElasticSearch,Redis,MySQL)

Chatting API application in Ruby on Rails and Go

## Overview
The API is composed of two separate services:
- Chat API (`Rails`): Main service which provides most of the core management operations 
(create, update, get) of applications, 
(get) of chats, 
and (get, update) messages, using `MySQl` as DB, and `Redis` - `Sidekiq` as (cahce, queue) 
also supports searching through messages in chats using `ElasticSearch`.
- Chat/Message Creation API (`Golang`): A complementary service to chat API that is responsible for creating chats and messages.

## Starting Services
> **Note**
>
> Just when cloning this repo care about LF files format not converted to CRLF
>
> file list
>
>   Chat_API_System_Rails_GO\go-chat-creation-api\entrypoint.sh
>
>   Chat_API_System_Rails_GO\go-chat-creation-api\wait-for-it.sh
>
>   Chat_API_System_Rails_GO\chat-api\entrypoint.sh
>
>   Chat_API_System_Rails_GO\chat-api\wait-for-it.sh

```bash
sudo docker-compose down && sudo docker-compose build && sudo docker-compose up
```
Make sure that `docker` and `docker-compose` are installed with `dockerd` running, also make sure that ports `3000` and ports `8080` are available for the services to run on.

## Using Services
Feel Free to use Postman Colection & Enviroment attached with this repo
### Chat API (Rails)
This service exposes these endpoints for operating on applications, chats and messages.

```
Verb  URI Pattern
----  -----------

GET   /applications/
POST  /applications?name={name}
GET   /applications/{access_token}
PUT   /applications/{access_token}?name={name}

GET   /applications/{access_token}/chats
GET   /applications/{access_token}/chats/{chat_number}

GET   /applications/{access_token}/chats/{chat_number}/messages
GET   /applications/{access_token}/chats/{chat_number}/messages/{message_number}
GET   /applications/{access_token}/chats/{chat_number}/messages/search?keyword={keyword}
PUT   /applications/{access_token}/chats/{chat_number}/messages/{message_number}?body={message_body}
```
#### Examples

##### Creating a new application
```bash
$ curl -X POST 'http://localhost:3000/applications?name=app'

# output
{
  "name": "app",
  "access_token": "fPrv7vr57dkUsP4KfZ4BdSmt",
  "created_at": "2019-11-11T19:14:51.589Z",
  "updated_at": "2019-11-11T19:14:51.589Z",
  "chat_count": 0
}
```

##### Getting applications
```bash
$ curl -X GET 'http://localhost:3000/applications'

# output
{
  [
    {
      "name": "app",
      "access_token": "fPrv7vr57dkUsP4KfZ4BdSmt",
      "created_at": "2019-11-11T19:14:51.589Z",
      "updated_at": "2019-11-11T19:14:51.589Z",
      "chat_count": 0
    },
    {
      "name": "app2",
      "access_token": "edxUGBopTJQpBpkeWLbdrccR",
      "created_at": "2019-11-11T19:14:51.589Z",
      "updated_at": "2019-11-11T19:14:51.589Z",
      "chat_count": 0
    }
  ]
}
```

##### Getting application
```bash
$ curl -X GET 'http://localhost:3000/applications/{fPrv7vr57dkUsP4KfZ4BdSmt}'

# output
{
   "name": "app",
   "access_token": "fPrv7vr57dkUsP4KfZ4BdSmt",
   "created_at": "2019-11-11T19:14:51.589Z",
   "updated_at": "2019-11-11T19:14:51.589Z",
   "chat_count": 0
}
```
##### Updating a application
```bash
$ curl -X PUT 'http://localhost:3000/applications/fPrv7vr57dkUsP4KfZ4BdSmt?name=app updated'

# output
{
  "name": "app updated",
  "access_token": "fPrv7vr57dkUsP4KfZ4BdSmt",
  "created_at": "2019-11-11T19:14:51.589Z",
  "updated_at": "2019-11-11T19:14:51.589Z",
  "chat_count": 0
}
```


##### Getting messages
```bash
$ curl -X GET 'http://localhost:3000/applications/fPrv7vr57dkUsP4KfZ4BdSmt/chats/1/messages'

# output
[
  {
    "number": 1,
    "body": "Rails stuff",
    "created_at": "2019-11-11T19:18:50.279Z",
    "updated_at": "2019-11-11T19:18:50.279Z"
  },
  {
    "number": 2,
    "body": "Stuff with Rails and Go and stuff with some other stuff",
    "created_at": "2019-11-11T19:18:52.351Z",
    "updated_at": "2019-11-11T19:18:52.351Z"
  }
]

```
##### Searching chats
```bash
$ curl -X GET 'http://localhost:3000/applications/fPrv7vr57dkUsP4KfZ4BdSmt/chats/1/messages/search?keyword=Go'

# output
[
  {
    "number": 2,
    "body": "Stuff with Rails and Go and stuff with some other stuff",
    "created_at": "2019-11-11T19:18:52.351Z",
    "updated_at": "2019-11-11T19:18:52.351Z"
  }
]
```

### Chat/Message Creation API (Golang)
This service handles creation of chats and messages.

```
Verb  URI Pattern
----  -----------

POST  /applications/{access_token}/chats/
POST  /applications/{access_token}/chats/{chat_number}/messages?body={message_body}
```
#### Examples
##### Creating a new chat
```bash
$ curl -X POST 'http://localhost:8080/applications/fPrv7vr57dkUsP4KfZ4BdSmt/chats'

# output
{
  "number": 1,
  "access_token": "fPrv7vr57dkUsP4KfZ4BdSmt"
}
```

##### Sending a new message
```bash
$ curl --data '{"body": "Rails stuff"}' -X POST 'http://localhost:8080/applications/fPrv7vr57dkUsP4KfZ4BdSmt/chats/1/messages'

# output
{
  "number":1,
  "chat_number":1,
  "access_token":"fPrv7vr57dkUsP4KfZ4BdSmt"
 }
 ```
 
 ## How This Works
 Chat/Message creation API uses `Redis` for two purposes, first of which is to cache and determine the next chat number
 and message number to respond to the user with, second of which is to queue jobs to `Sidekiq` workers hosted at the main Rails API. These `Sidekiq`
 workers are responsible of handling requests to create chats and messages in the background, to allow for better scaling in case a huge number of requests is received.
 
 Sending a POST request to create a message follows these procedures:
 - Golang Chat/Message creation API first receives this request, gets the application token and the chat number from the request URI.
 - It then generates a key that refers to this application token/chat number combination
 - It then tries to get the next number using this key from `Redis` store, if it exists then it atomically gets and increments the value,
 and if it's not, then it sends a request to the main Rails API to get the current message count.
 - After getting the next number, the API queues a job to create this message in `Sidekiq`, and responds to the user with the number it created.
 - When a `Sidekiq` worker pick up a message creation job, it then adds this message to the messages table, updating a [`counter_cache`](https://guides.rubyonrails.org/association_basics.html) automatically.
 - Race conditions are handled in both sides:
     - A race condition may occur in the message creation side when two concurrent requests that use the same chat are unable to find a key/value pair in `Redis`
 at the same time, when this happens they both send a request to the main Rails API and set the count in store with the same value, leading to two responses with the same number,
 to handle this [`redis-lock`](https://github.com/bsm/redis-lock) is used to avoid this issue, requests basically "lock" the key/value pair using
 this key combination and release this lock after writing to the store.
     - Race conditions are handled at the main Rails side using `uniqueness` validations on both chat number and message number.
     
Used `elasticsearch` to support searching through messages.

