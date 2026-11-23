# API Rate Limiter Using Python and MongoDB

## Overview

This project focuses on developing an API rate-limiting system using Python, FastAPI, and MongoDB. It manages incoming client requests and prevents excessive API usage by restricting the number of requests allowed over time.

## Algorithm

Token Bucket Algorithm

## Data Storage

MongoDB Collection: `rate_limits`

Each client is represented by a document that stores:
- Client identifier (API key or IP address)
- Current available tokens
- Timestamp of the last token update
- Document expiration time

## System Configuration

- Maximum bucket capacity: 5 tokens
- Token refill rate: 1 token per second
- Database: MongoDB
- Framework: FastAPI
- Programming Language: Python

## Working Mechanism

When a client sends a request, the system identifies them through the `X-API-Key` header. If the header is unavailable, the client's IP address is used as the identifier.

A token bucket is maintained separately for each client. Every incoming request requires one available token, while tokens are replenished automatically at a rate of one per second, without exceeding the bucket's maximum capacity.

Before processing a request, the system calculates the tokens accumulated since the previous update. If sufficient tokens are available, the request is permitted and one token is deducted. If the bucket does not contain enough tokens, the request is denied with a `429 Too Many Requests` response.

MongoDB performs these updates atomically to avoid inconsistencies when multiple requests arrive simultaneously. Expired client records are automatically deleted using a TTL index.

## Key Features

- Individual request limits for different clients
- Separate token buckets based on API keys or IP addresses
- Automatic token regeneration over time
- Configurable bucket size and refill speed
- MongoDB integration for storing client rate-limit data
- Atomic updates to support concurrent requests
- HTTP 429 error handling for excessive requests
- Retry-After header to indicate the waiting period
- Rate-limit response headers
- Database connectivity verification through a health endpoint

## Prerequisites

- Python 3
- MongoDB 4.2 or newer
- FastAPI
- Uvicorn
- Motor
- PyMongo

## Installation

Install the required Python dependencies:

```bash
pip install -r requirements.txt
```

Start a MongoDB instance using Docker:

```bash
docker run -d -p 27017:27017 --name mongo mongo:7
```

## Running the Project

Launch the application using Uvicorn:

```bash
uvicorn main:app --reload
```

Once started, the application runs locally at:

`http://localhost:8000`

## Testing

Multiple requests can be sent to the API using the following command:

```bash
for i in $(seq 1 8); do
    curl -i localhost:8000/api/hello
done
```

Initially, the bucket contains 5 tokens, allowing the first five requests to pass. Additional requests are rejected with a `429 Too Many Requests` response until enough tokens are replenished.

Clients can also be tested independently by providing different API keys through the `X-API-Key` header.

## Available Endpoints

| Endpoint | HTTP Method | Purpose |
|---|---|---|
| `/api/hello` | GET | Processes requests subject to rate limiting |
| `/health` | GET | Verifies application and database connectivity |

## Limitations

- MongoDB must be running for the rate limiter to function.
- Default rate-limit values are configurable but remain common to all clients.
- Client identification depends on the API key or IP address.
- The project implements rate limiting without additional authentication or authorization.
- Inactive client records are removed through MongoDB's TTL cleanup mechanism.

## Conclusion

The project provides a practical implementation of API rate limiting using the Token Bucket algorithm. By combining FastAPI with MongoDB, it manages request traffic, supports independent client tracking, and handles simultaneous requests through atomic database operations. Automatic token replenishment also allows occasional bursts of requests while maintaining control over overall API usage.
