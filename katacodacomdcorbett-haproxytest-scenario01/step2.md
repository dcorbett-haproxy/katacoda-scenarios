## Start Application Servers
To get started we will want to start some application servers that HAProxy will route traffic to.

Run the following command within the terminal.

`docker run -d katacoda/docker-http-server; docker run -d katacoda/docker-http-server`{{execute T1}}

## Starting HAProxy
Next we will want to start up HAProxy.

Run the following command within the terminal.
`docker run -d -p 80:80 -v /root/haproxy.cfg:/etc/haproxy/haproxy.cfg haproxytech/haproxy-ubuntu`{{execute T1}}

## Test Results
Finally, we can test the results.  You can view this in your browser at the following URL:
https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/ 

Or optionally, using curl by running the below command in the terminal. 
`curl localhost`{{execute T1}}


You should see the response:
"This request was processed by host: <docker container id>"

