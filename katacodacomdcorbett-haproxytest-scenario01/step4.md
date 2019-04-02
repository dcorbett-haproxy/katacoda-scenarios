## Runtime API
The HAProxy Runtime API traces its origins back to our wishes to create a complete configuration and statistics API for HAProxy, whose commands would all take effect immediately, during runtime. One of our early features in this API was of course the ability to retrieve detailed real-time statistics. Also, unlike typical newer APIs which only support HTTP, the HAProxy Runtime API was always accessible through TCP and Unix sockets. That is why we sometimes still refer to it as the HAProxy `stats socket` or just "socket", and why the configuration directive for enabling the Runtime API bears the same name.


### show info
Sending `show info` to the Runtime API should give you information about the running HAProxy process.  Some useful information you can expect to find is the version, the maximum connection limits, the current connections established, as well as the current idle percentage.  It's extremely helpful when it comes determining how busy HAProxy may be or debugging an issue you may be encountering.

Run the below in the terminal.

`echo "show info" | socat - tcp-connect:172.18.0.2:9000`{{execute T1}} 


### show stat
Sending `show stat` to the Runtime API will provide a CSV output of statistics for all of the configured `frontend` and `backend` sections configured within HAProxy.  It also supports output in JSON which can be gathered using `show stat json`.

The statistics output contains more than 80 different metrics. To quickly convert it to a shorter and human readable output, we could use standard command line tools.

Run the below in the terminal.

`echo "show stat" | socat - tcp-connect:172.18.0.2:9000  | cut -d "," -f 1-2,5-10,34-36 | column -s, -t'`{{execute T1}}

### set server
Sending `set server` allows you to change multiple aspects about a specific server within an HAProxy `backend`.  Some of the items that can be changed are:

Below is just a brief description of the options available.  More detail can be found within the (HAProxy Documentation)[https://www.haproxy.com/documentation/hapee/1-9r1/onepage/management/#9.3]
* addr - Replace the current IP address of a server by the one provided.
* agent -  Force a server's agent to a new state. This can be useful to immediately switch a server's state regardless of some slow agent checks for example.
* agent-addr - Change addr for servers agent checks. Allows to migrate agent-checks to another address at runtime.
* agent-send - Change agent string sent to agent check target. Allows to update string while changing server address to keep those two matching.
* health - Force a server's health to a new state. This can be useful to immediately switch a server's state regardless of some slow health checks for example.
* check-port - Change the port used for health checking to <port>
* state - Force a server's administrative state to a new state. This can be useful to disable load balancing and/or any traffic to a server.
* weight - Change a server's weight to the value passed in argument. This is the exact equivalent of the "set weight" command below.
* fqdn - Change a server's FQDN to the value passed in argument. This requires the internal run-time DNS resolver to be configured and enabled for this server.

For this example we will disable one of the servers from receiving traffic.

Run the below in the terminal.

`echo "set server be_app/app1 maint" | socat - tcp-connect:172.18.0.2:9000`{{execute T1}}

The server **app1** should now be disabled.  You can confirm by sending this by navigating to the statistics page: [https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/haproxy-stats

You should see **app1** labeled in orange and under the "Status" column it should say "MAINT".

You can also use the below curl command to the terminal a few times.  You should no longer receive a response from it.

`curl localhost`{{execute T1}}
 
