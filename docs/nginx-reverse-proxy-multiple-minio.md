# Nginx as a reverse proxy to multiple minio instances. [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/minio/minio?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

This document explains how to run three minio instances and use nginx to route the client requests to the right minio instance based on the request's ACCESS_KEY.

*Run Minio 1*:
```
minio server --address :9001 /export1
```
Let's assume access key of this minio instance is ACCESS_KEY_1

*Run Minio 2*:
```
minio server --address :9002 /export2
```
Let's assume access key of this minio instance is ACCESS_KEY_2

*Run Minio 3*:
```
minio server --address :9003 /export3
```
Let's assume access key of this minio instance is ACCESS_KEY_3

We have 3 instances of minio running on ports 9001, 9002 and 9003 with different access/secret key pairs.

Now run nginx with the following config in `/etc/nginx/sites-enabled`:
```
server {
    listen       80;
	location / {
	    proxy_set_header Host $http_host;
	    if ($http_authorization ~* "^AWS4-HMAC-SHA256 Credential=ACCESS_KEY_1") {
           # proxy the request to Minio-1
	       proxy_pass http://localhost:8001;
	    }
	    if ($http_authorization ~* "^AWS4-HMAC-SHA256 Credential=ACCESS_KEY_2") {
           # proxy the request to Minio-2
	       proxy_pass http://localhost:8002;
	    }
	    if ($http_authorization ~* "^AWS4-HMAC-SHA256 Credential=ACCESS_KEY_3") {
           # proxy the request to Minio-3
	       proxy_pass http://localhost:8003;
	    }
    }
}
```

Nginx runs on port 80 and reverse proxies the requests to Minio-1 or Minio-2 or Minio-3 depending on the ACCESS_KEY in the client request.

