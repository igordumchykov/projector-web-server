# Web Server
Configuration that runs NGINX web server
- exposes ` GET /image/{imageName}` endpoint to get images stored on back end
- NGINX caches images that requested at least 2 times
- exposed `PURGE /image/{imageName}` endpoint for purging image from NGINX cache
- there is [lua script](./sripts.lua) that handles purge request: it calculates md5 hash, extracts 1:2 cache level folders, finds a file and removes from a cache
- cache is stored in a [separate folder](./cache) and mounted as a volume to NGNX in [docker-compose](./docker-compose.yml)

## Purge Approaches
To purge a file from a cache we can use the next approaches:
1. Using `ngx_cache_purge` module.

`ngx_cache_purge` is not included in official [NGINX](https://hub.docker.com/_/nginx) image. Therefore, NGINX docker
image should be built with ngx_cache_purge module.

2. Using [commercial version of NGNX](https://docs.nginx.com/nginx/)
3. Using custom lua scrip that handles purge request


# Run
```shell
docker-compose up -d
```

# Test

1. Get image and check it is missed in cache:

request:
```shell
curl -I -X GET http://localhost/image/dog.jpeg
```

response:
```
Server: nginx/1.10.2
Date: Wed, 19 Jul 2023 16:39:37 GMT
Content-Type: image/jpeg
Content-Length: 211690
Connection: keep-alive
Last-Modified: Tue, 18 Jul 2023 08:17:52 GMT
ETag: "64b64ab0-33aea"
Accept-Ranges: bytes
X-Cache-Status: MISS
X-Nginx-Cache-Head: httplocalhost/image/dog.jpeg
```

2. Get image and check it is missed in cache:

request:
```shell
curl -I -X GET http://localhost/image/dog.jpeg
```

response:
```
HTTP/1.1 200 OK
Server: nginx/1.10.2
Date: Wed, 19 Jul 2023 16:40:18 GMT
Content-Type: image/jpeg
Content-Length: 211690
Connection: keep-alive
Last-Modified: Tue, 18 Jul 2023 08:17:52 GMT
ETag: "64b64ab0-33aea"
X-Cache-Status: MISS
X-Nginx-Cache-Head: httplocalhost/image/dog.jpeg
```

3. Get image and check it exists in cache:

request:
```shell
curl -I -X GET http://localhost/image/dog.jpeg
```

response:
```
HTTP/1.1 200 OK
Server: nginx/1.10.2
Date: Wed, 19 Jul 2023 16:40:40 GMT
Content-Type: image/jpeg
Content-Length: 211690
Connection: keep-alive
Last-Modified: Tue, 18 Jul 2023 08:17:52 GMT
ETag: "64b64ab0-33aea"
X-Cache-Status: HIT
X-Nginx-Cache-Head: httplocalhost/image/dog.jpeg
```

4. Purge image from cache:

request:
```shell
curl -X PURGE http://localhost/image/dog.jpeg
```

response:
```
HTTP/1.1 200 OK
Server: nginx/1.10.2
Date: Wed, 19 Jul 2023 16:44:33 GMT
Content-Type: image/jpeg
Transfer-Encoding: chunked
Connection: keep-alive
X-Nginx-Cache-Head: http/image/dog.jpeg
```

5. Get image again:

request:
```shell
 curl -I -X GET http://localhost/image/dog.jpeg
```

response:
```
HTTP/1.1 200 OK
Server: nginx/1.10.2
Date: Wed, 19 Jul 2023 16:46:41 GMT
Content-Type: image/jpeg
Content-Length: 211690
Connection: keep-alive
Last-Modified: Tue, 18 Jul 2023 08:17:52 GMT
ETag: "64b64ab0-33aea"
X-Cache-Status: MISS
X-Nginx-Cache-Head: httplocalhost/image/dog.jpeg
```

6. Get image again:

request:
```shell
 curl -I -X GET http://localhost/image/dog.jpeg
```

response:
```
HTTP/1.1 200 OK
Server: nginx/1.10.2
Date: Wed, 19 Jul 2023 16:48:39 GMT
Content-Type: image/jpeg
Content-Length: 211690
Connection: keep-alive
Last-Modified: Tue, 18 Jul 2023 08:17:52 GMT
ETag: "64b64ab0-33aea"
X-Cache-Status: HIT
X-Nginx-Cache-Head: httplocalhost/image/dog.jpeg
```

# TODO
Take a look at [OpenResty](https://openresty.org/en/)
