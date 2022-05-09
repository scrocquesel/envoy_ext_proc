### Envoy External Processing Filter


Repro for https://github.com/envoyproxy/envoy/issues/21205

```bash
docker cp `docker create  envoyproxy/envoy-dev:latest`:/usr/local/bin/envoy .
```

Now start the external gRPC server

```bash
go run grpc_server.go
```

This will start the gRPC server which will receive the requests from envoy.  

```
./envoy -c server.yaml -l debug
```

```bash
$ curl -v -H "host: http.domain.com"  --resolve  http.domain.com:8080:127.0.0.1  http://http.domain.com:8080/gzip

* Added http.domain.com:8080:127.0.0.1 to DNS cache
* Hostname http.domain.com was found in DNS cache
*   Trying 127.0.0.1:8080...
* TCP_NODELAY set
* Connected to http.domain.com (127.0.0.1) port 8080 (#0)
> GET /gzip HTTP/1.1
> Host: http.domain.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< date: Mon, 09 May 2022 21:12:34 GMT
< content-type: application/json
< server: envoy
< access-control-allow-origin: *
< access-control-allow-credentials: true
< x-envoy-upstream-service-time: 358
< transfer-encoding: chunked
< 
^C
```

```plain
2022/05/09 23:12:34 Got stream:  -->  
2022/05/09 23:12:34 pb.ProcessingRequest_RequestHeaders &{headers:{headers:{key:":authority"  value:"http.domain.com"}  headers:{key:":path"  value:"/gzip"}  headers:{key:":method"  value:"GET"}  headers:{key:":scheme"  value:"http"}  headers:{key:"user-agent"  value:"curl/7.68.0"}  headers:{key:"accept"  value:"*/*"}  headers:{key:"x-forwarded-proto"  value:"http"}  headers:{key:"x-request-id"  value:"2e2388f4-47b4-4038-ab46-48f68aef50f0"}}  end_of_stream:true} 
```