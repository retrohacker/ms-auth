Auth Microservice
====

[![Greenkeeper badge](https://badges.greenkeeper.io/retrohacker/ms-auth.svg)](https://greenkeeper.io/)

A simple microservice that handles all authentication

# Structure

### `auth`

Contains all code related to oauth providers

### `db`

Contains all postgresql specific code

### `glue`

Glue code wiring auth endpoints to the db

### `models`

Defines the structure of data within the app. Also contains the postgresql definitions of that data.

### `session`

All infomration for connecting to redis.

# Dependencies

We highly recommend using this with a tool such as `docker-compose` for local development.

## Environment Variables

There are two variables that need to be set:

* `APP_HOST`: The host that Google OAuth services expects your app to redirect to. For local dev, check out http://localtest.me or update `/etc/hosts`
* `APP_ROUTE`: The endpoint this authentication service will register to. For example `/auth` will register `/auth/google`.
* `APP_REDIRECT`: What endpoint should a completed OAuth handshake send the client to?
* `APP_PORT`: What port should the service bind to? (Defaults to 3000)


Here is an example `app.env` environment file for use with `docker-compose`:

```
APP_HOST=http://localtest.me:3000
APP_ROUTE=/auth
APP_REDIRECT=/
APP_PORT=3000
```

## `creds.json`

Defines the Google OAuth params, if you don't have these check out console.developers.google.com, setup a project, and go to APIs & auth -> Credentials. Here is an example file:

```
{
  "google" : {
    "id":"",
    "secret":""
  }
}
```

## Postgresql

Can be provided using the `postgres` docker container. Here is an example `app.env` environment file for use with `docker-compose`:

```
PGHOST=db
PGPORT=5432
PGDATABASE=postgres
PGUSER=postgres
PGPASSWORD=pgpassword
```

## Redis

Can be provided using the `redis` docker container. Here is an example `app.env` environment file for use with `docker-compose`:

```
REDIS_PORT=6379
REDIS_URL=session
REDIS_SECRET=redispassword
```

## Proxy

You will probably need a proxy layer that routes traffic to your authentication microservice. NGinx is great for this, and you can use the `nginx` docker container. Here is an example `nginx.conf` file for use when 

```
worker_processes 4;
worker_rlimit_nofile 8192;

user      www-data;
pid       /var/run/nginx.pid;

events {
    multi_accept on;
    worker_connections 4096;
}

http {
  server {
    listen 80;
    location /auth {
      proxy_pass http://auth:3000;
    }
  }
}
```

Where `auth` resolves to the ip address of your authentication service (using `iptables`, `/etc/hosts`, or `docker` magic)
