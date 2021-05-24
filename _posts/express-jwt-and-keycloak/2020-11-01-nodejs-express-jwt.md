---
title: "Express-jwt and Keycloak: how I did not use official Keycloak library"
description: "I had task of adding security to microservices system using JWT..."
date: 2020-11-01 00:00:00 +00:00
modified: 2020-11-01 00:00:00 +00:00
tags: [node,springboot,jwt,keycloak]
---

# Problem

We have many microservices that run on multiple deployments. I wanted to add security by using Keycloak with the help of JWT. 

# Solution

One of the earliest solution was to use [Keycloak Js Adapter](https://github.com/keycloak/keycloak-nodejs-connect). Yet, Keycloak JS adapter requires following:

```js
var keycloakConfig = {
    clientId: 'nodejs-microservice',
    bearerOnly: true,
    serverUrl: 'http://localhost:8080/auth',
    realm: 'Demo-Realm',
    credentials: {
        secret: '62c99f7c-da55-48fb-ae4e-a27f132546b7'
    }
};
```
which seems cumbersome way of doing this.

---

I thought there must be more simple way, I just wanted to **validate requests**. 

That's why I liked [Spring Boot approach](https://www.appsdeveloperblog.com/oauth2-resource-server-and-keycloak/) which is:
- include package
  
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

- add one line config

```yaml
spring.security.oauth2.resourceserver.jwt.issuer-uri = http://localhost:8080/auth/realms/appsdeveloperblog
```

At start, it fetches makes request to `issuer-uri` which has response like this

```json
{
  "realm": "appsdeveloperblog",
  "public_key": "...",
  "token-service": "http://localhost:8080/auth/appsdeveloperblog/master/protocol/openid-connect",
  "account-service": "http://localhost:8080/realms/appsdeveloperblog/account",
  "tokens-not-before": 0
}
```
and stores `public_key` which is used to **validate JWT tokens**. It doesn't make request each time to verify JWT.
As result, any request is validated and working out of box.


---

So I wanted to replicate this on NodeJS.

I started with [express-jwt](https://github.com/auth0/express-jwt) and simple example was like this

```js
var jwt = require('express-jwt');

app.get('/protected',
  jwt({ secret: 'shhhhhhared-secret', algorithms: ['HS256'] }),
  function(req, res) {
    if (!req.user.admin) return res.sendStatus(401);
    res.sendStatus(200);
  });

//Or with public key, shortened

var publicKey = fs.readFileSync('/path/to/public.pub');
jwt({ secret: publicKey, algorithms: ['RS256'] });

```

However it was problem for us to provide public key because 
- we have multiple deployments
- each deployment has its own Keycloak. 

We couldn't maintain this so I decided to implement like in Spring Boot.

With the help `sync-request` package:

```js
const res = request('GET', 'http://localhost:8080/auth/realms/appsdeveloperblog');
const response = JSON.parse(res.getBody().toString());
const publicKey = `-----BEGIN PUBLIC KEY-----\r\n${response.public_key}\r\n-----END PUBLIC KEY-----`;

app.use(jwt({ secret: publicKey, algorithms: ['RS256'] }));
```
I achieved on-start fetch of public key without cumbersome settings on NodeJS.


---

> [Article on dev.to](https://dev.to/nurgasemetey/how-to-fetch-public-key-for-express-jwt-on-start-from-openid-on-nodejs-4hjc)
> 
> [Article on Medium](https://nurgasemetey.medium.com/problem-728af2b5dbf3)