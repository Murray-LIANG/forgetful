# Rate Limiting


## Simple Implementation

- git/playground/rate-limit
- git/playground/rate-limiting-service


## Open Source Projects 

There are some open source libraries which could help do the rate limiting.

### Doorman

https://github.com/youtube/doorman/tree/master/doc/loadtest

It is used in this scenario: there is a service (RPC, REST, HTTP, .etc) having limited capacity. And you have lots of clients sending requests to the service.

It starts a `doorman server`. And all the clients ask for limits before sending requests to the target.

Multiple instances of `doorman server` could be set up to avoid single point of failure, for example, we would run 3 replicas of `doorman server` in Kubernetes cluster and they would user `etcd` to elect a leader among themselves.

### Traefik

Traefik is a HTTP reverse proxy and load balancer which creates and updates the routers automatically.

You could configure a middleware named `rate limiter` in Traefik.

https://github.com/containous/traefik/blob/master/pkg/middlewares/ratelimiter/rate_limiter.go

Each request reaches the Traefik and goes through the rate limiter for throttling and then is forwarded to the real target.

