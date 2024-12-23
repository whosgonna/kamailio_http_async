## Kamailio's HTTP\_ASYNC\_CLIENT module vs ASYNC module

There seems to be conflation between Kamailio's `HTTP_ASYNC_CLIENT` module
and the ASYNC module.  The apparent behavior of the `HTTP_ASYNC_CLIENT` is to
create an event-loop where the current SIP transaction is suspended with the
sending of the http request (via libcurl) and then resume the transaction when
the HTTP reply is received.

From observation, this does not simply "hand off the transaction" to another
pool of workers which then accumulate load.



### Project description

This docker compose project consists of three containers:

- `sipp` - This can be used as a UAC to load test the kamailio config.  It
includes a scenario file that will expect a 404 response from the kamailio
server
- `kamailio` - A basic kamailio proxy config that can be set to send either
normal `http_client` requests or `http_async_client` requests to the web
server. On receiving a reply from the web server a `404 Not Found` is
replied to the uac. By default, the kamailio container will start up using
the http async queries
- `www` - A container using the [deminy/delayed-http-response](https://registry.hub.docker.com/r/deminy/delayed-http-response)
image which will take a GET request and wait the specified amount of time
before sending an OK response.


### Further details on the Kamailio server
Kamailio is started with a child process count of `2` via the startup command
in the docker-compose.yml file.

The number of workers for `http_async_client` is set  to `1`, and it's
connection timeout parameter is set to 1.5 seconds. This is to allow for a
longer response time from the web server
```
modparam("http_async_client", "workers", 1);
modparam("http_async_client", "connection_timeout", 1500)
```

The URL to which the request is sent requests that the receving web service
wait 1 second before replying.  This is done in the request path:
```
$var(sleep) = "1.00";
$var(req)   = "http://www/sleep/" + $var(sleep);
```

For further details on the web service's sleep settngs, see that container's
documentation on docker hub.


### Setup

Start all containers and watch Kamailio's logs:
```
docker compose up -d
docker compose logs -f kamailio
```



### Testing
From the root of the project, the `sipp` container can be used for loadtesting:
```
docker compose exec sipp sipp -sf uac_expect_300.xml -m 18000 -r 300 kamailio
```

This will send a total of 18,000 "calls" (`-m 18000`) at a rate of 300
requests per second (`-r 300`).  While using the `http_async_query()`
function Kamailio should have no trouble keeping up with the requests.


To toggle the server to use the `http_client` module's `http_client_query()`
function, set the pseudo variable `$shv(http_async)` to `0`:
```
docker compose exec kamailio kamcmd pv.shvSet http_async int 0
```

Re-running the load test will result in heavily blocked and dropped SIP
requests.
