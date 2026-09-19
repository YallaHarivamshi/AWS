# AWS 3-Tier Web Application with Redis Caching

## Project Overview

This project demonstrates a **3-tier web application deployed on AWS EC2**.

The application uses:

* **React** for the frontend
* **Nginx** as the web server and reverse proxy
* **Node.js** for the backend API
* **Redis (ElastiCache)** for caching
* **Amazon RDS MySQL** for database storage

Redis is used to reduce repeated database queries and improve application performance.

## Architecture

```text
                    USER
                      |
                      v
                 +---------+
                 |  Nginx  |
                 | Port 80 |
                 +----+----+
                      |
                      v
              +---------------+
              | React Frontend|
              +-------+-------+
                      |
                 /api/products
                      |
                      v
              +---------------+
              | Node.js API   |
              |   Port 3000   |
              +-------+-------+
                      |
                 Check Cache
                      |
              +-------+-------+
              |               |
          Cache Hit       Cache Miss
              |               |
              |               v
              |        +-------------+
              |        | RDS MySQL   |
              |        | productsdb  |
              |        +------+------+
              |               |
              |          Store Result
              |               |
              |               v
              |        +-------------+
              +------->|    Redis    |
                       | ElastiCache |
                       +-------------+
```

## Request Flow

```text
User opens website
        ↓
Nginx serves React frontend
        ↓
React requests /api/products
        ↓
Nginx forwards request to Node.js
        ↓
Node.js checks Redis
        ↓
Cache HIT → Return cached data
        ↓
Cache MISS → Query RDS MySQL
        ↓
Store database result in Redis
        ↓
Return data to React
        ↓
React displays products
```

## AWS Services Used

| Service           | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| EC2               | Hosts the application                            |
| Nginx             | Serves frontend and reverse proxies API requests |
| RDS MySQL         | Stores product data                              |
| ElastiCache Redis | Caches frequently requested data                 |
| IAM               | Provides AWS permissions                         |

## Application Components

### React

React is used to build the frontend user interface.

It displays the products received from the backend API.

### Nginx

Nginx acts as the public-facing web server and reverse proxy.

```text
User
 ↓
Nginx
 ├── React Frontend
 └── /api/ → Node.js
```

### Node.js

Node.js runs the backend API.

The backend:

* Receives API requests
* Checks Redis cache
* Queries RDS when required
* Returns product data to React

### Redis

Redis is used as a caching layer.

```text
Request
   ↓
Redis
   ↓
Cache HIT → Return data
```

If the data is not available:

```text
Request
   ↓
Redis
   ↓
Cache MISS
   ↓
RDS MySQL
   ↓
Store result in Redis
   ↓
Return data
```

### RDS MySQL

Amazon RDS MySQL is used as the persistent database.

Example database:

```text
productsdb
```

Example table:

```text
products
```

Sample data includes:

```text
Laptop    1000
Phone      500
Keyboard   100
```

## Cache Testing

The backend was tested using the `/products` API.

First request:

```text
CACHE MISS
```

The application gets the data from RDS and stores it in Redis.

Second request:

```text
CACHE HIT
```

The application gets the data from Redis instead of querying RDS again.

## Process Manager

**PM2** is used to keep the Node.js backend running continuously.

```text
Node.js
   ↓
PM2
   ↓
Backend keeps running
```

PM2 also provides automatic startup after a server reboot.

## Technologies Used

```text
AWS
Amazon EC2
Amazon RDS MySQL
Amazon ElastiCache Redis
Nginx
Node.js
Express.js
React.js
Axios
PM2
Linux
```

## Key Learning

Through this project, I learned:

* How to deploy a React frontend on EC2
* How to create a Node.js backend API
* How to use Nginx as a reverse proxy
* How to connect Node.js with RDS MySQL
* How to use Redis for caching
* Difference between cache hit and cache miss
* How PM2 keeps Node.js applications running
* How multiple AWS services work together in a 3-tier application

## Final Architecture

```text
             React Frontend
                    |
                    v
                 Nginx
                    |
                    v
              Node.js API
                    |
              +-----+-----+
              |           |
              v           v
            Redis       RDS MySQL
         (Cache)       (Database)
```

### Project Result

A complete AWS-based 3-tier application was deployed where **React provides the frontend, Nginx handles web traffic, Node.js provides the backend API, Redis improves performance through caching, and RDS MySQL stores persistent data**.
