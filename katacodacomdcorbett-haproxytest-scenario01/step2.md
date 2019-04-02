## Starting HAProxy
`docker run -d -p 80:80 -v /root/haproxy.cfg:/etc/haproxy/haproxy.cfg haproxytech/haproxy-ubuntu`{{execute T1}}

## Start Application Servers
`docker run -d katacoda/docker-http-server; docker run -d katacoda/docker-http-server`{{execute T1}}

## Test Results
`curl localhost`{{execute T1}}

## Use Runtime API
`echo "show info" | socat - tcp-connect:172.18.0.2:9000`{{execute T1}} 
