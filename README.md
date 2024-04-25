# Chat API using Ruby on Rails and with (ElasticSearch,Redis,MySQL)

Chatting API application in Ruby on Rails and Go

## Overview

The API is composed of two separate services:

- Chat API (`Rails`): Main service which provides most of the core management operations
  (create, update, get) of applications,
  (get) of chats,
  and (get, update) messages, using `MySQl` as DB, and `Redis` - `Sidekiq` as (cahce, queue)
  also supports searching through messages in chats using `ElasticSearch`.

## Starting Services

```bash
sudo docker-compose up #as mentioned on task PDF
```

Make sure that `docker` and `docker-compose` are installed, and `docker` running, also make sure that ports `3000` not in use.

## Using Services

Feel Free to use Postman Colection & Enviroment attached with this repo

### Chat API (Rails)

This service exposes these endpoints for operating on applications, chats and messages.

```bash
Verb  URI Pattern
----  -----------
# application endpoints
GET    /application(.:format)
POST   /application(.:format)
GET    /application/:access_token(.:format)
PATCH  /application/:access_token(.:format)
PUT    /application/:access_token(.:format)
DELETE /application/:access_token(.:format)

# chat endpoints
GET    /application/:application_access_token/chat(.:format)
POST   /application/:application_access_token/chat(.:format)
GET    /application/:application_access_token/chat/:number(.:format)
PATCH  /application/:application_access_token/chat/:number(.:format)
PUT    /application/:application_access_token/chat/:number(.:format)
DELETE /application/:application_access_token/chat/:number(.:format)

# messages endpoints
GET    /application/:application_access_token/chat/:chat_number/message(.:format)
POST   /application/:application_access_token/chat/:chat_number/message(.:format)
GET    /application/:application_access_token/chat/:chat_number/message/:number(.:format)
PATCH  /application/:application_access_token/chat/:chat_number/message/:number(.:format)
PUT    /application/:application_access_token/chat/:chat_number/message/:number(.:format)
DELETE /application/:application_access_token/chat/:chat_number/message/:number(.:format)

# message search
GET    /application/:application_access_token/chat/:chat_number/message/search(.:format)
```

## How This Works

1. **Application Creation:**

   - Clients create applications with a provided name, receiving a unique token for identification.

2. **Chat Creation:**

   - Within each application, sequential chats are created, each with a unique number starting from 1.

3. **Message Creation:**

   - Clients send messages within specific chats, with each message assigned a sequential number starting from 1.

4. **Security and Privacy:**

   - IDs of entities (applications, chats, messages) are concealed from clients. Identification is done via tokens and chat numbers.

5. **Message Searching:**

   - ElasticSearch facilitates efficient message searching within chats.

6. **Data Management:**

   - Tables track chat and message counts, updated regularly.

7. **Concurrency and Optimization:**

   - Handles concurrent requests, avoiding race conditions.
   - Optimizes queries with appropriate indices and queuing for efficient processing.

8. **Containerization:**

   - Docker containerizes the application for easy deployment.

9. **Implementation Details:**
   - Built with Ruby on Rails, following RESTful conventions.
   - Utilizes MySQL as the main datastore, with optional Redis integration.
