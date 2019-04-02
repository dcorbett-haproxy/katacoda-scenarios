There are four essential sections to an HAProxy configuration file. They are `global`, `defaults`, `frontend`, and `backend`. These four sections define how the server as a whole performs, what your default settings are, and how client requests are received and routed to your backend servers.


## Format
A section begins when a keyword like `global` or `defaults` is encountered and is comprised of all of the lines that follow until you reach another section keyword. Blank lines and indentation are ignored. So, the global section continues until you get to, say, a defaults keyword on its own line. 

## Global
At the top of your HAProxy configuration file is the `global` section, identified by the word `global` on its own line. Settings under `global` define process-wide security and performance tunings that affect HAProxy at a low level.

<pre class="file" data-filename="haproxy.cfg" data-target="replace">global
    log stdout local0
    log stdout local1 notice
    user haproxy
    group haproxy
    stats socket 172.18.0.2:9000 user haproxy group haproxy mode 660 level admin
</pre>

Let's go over how these settings work

### log
The `log` setting ensures that warnings emitted during startup and issues that arise during runtime get logged to syslog. It also logs requests as they come through. You can target the traditional UNIX socket where Syslog or journald, listen, /dev/log, specify a remote rsyslog server, or using cloud-native way which is logging to **stdout** so that log data is preserved externally to your load balancing server. Set a Syslog facility, which is typically **local0**, which is a facility categorized for custom use. Note that in order to read the logs, you will need to configure any of the syslog daemons, or journald, to write them to a file.

### user / group
The `user` and `group` lines tell HAProxy to drop privileges after initialization. Linux requires processes to be root in order to listen on ports below 1024. You’ll also typically want your TLS private keys to be readable only by root as well. Without defining a user and group to continue the process as, HAProxy will keep root privileges, which is a bad practice. Be aware that HAProxy itself does not create the user and group and so they should be created beforehand.

### stats socket
The `stats` socket line enables the Runtime API, which you can use to dynamically disable servers and health checks, change the load balancing weights of servers, and pull other useful levers.  This can be specified as a unix socket (/var/run/haproxy.sock) or over TCP using (IP:port).

## Defaults
As your configuration grows, using a `defaults` section will help reduce duplication. Its settings apply to all of the `frontend` and `backend` sections that come after it. You’re still free to override those settings within the sections that follow.

You also aren’t limited to having just one `defaults`. Subsequent defaults sections will override those that came before and reset all options to their default values.

So, you might decide to configure a defaults section that contains all of your TCP settings and then place your TCP-only `frontend` and `backend` sections after it. Then, place all of your HTTP settings in another `defaults` section and follow it with your HTTP frontend and backend sections.

<pre class="file" data-filename="haproxy.cfg" data-target="append">defaults
    mode http
    log global
    option httplog
    timeout connect 5s
    timeout client 5s
    timeout server 5s
</pre>

### mode
The `mode` setting defines whether HAProxy operates as a simple TCP proxy or if it’s able to inspect incoming traffic’s higher-level HTTP messages. The alternative to specifying `mode http` is to use `mode tcp`, which operates at the faster, but less-aware, level. If most of your `frontend` and `backend` sections would use the same mode, it makes sense to specify it in the `defaults` section to avoid repetition.

### log global
The `log global` setting is a way of telling each subsequent `frontend` to use the `log` setting that you defined in the `global` section. This isn’t required for logging, as new `log` lines can be added here or in each `frontend`. However, in most cases wherein only one syslog server is used, this is common.

### option httplog
The `option httplog` setting, or more rarely `option tcplog`, tells HAProxy to use a more verbose log format when sending messages to Syslog. You will generally prefer `option httplog` over `option tcplog` in your `defaults` section because when HAProxy encounters a `frontend` that uses `mode tcp`, it will emit a warning and downgrade it to `option tcplog` anyway.

If neither is specified, then the connect log format is used, which has very few details other than the client and backend IP addresses and ports. Another option is to define a custom log format with the `log-format` setting, in which case `option httplog` and `option tcplog` aren’t necessary.

### timeout connect / timeout client / timeout server
The `timeout connect` setting configures the time that HAProxy will wait for a TCP connection to a backend server to be established. The “s” suffix denotes seconds. Without any suffix, the time is assumed to be in milliseconds. The `timeout client` setting measures inactivity during periods that we would expect the client to be speaking, or in other words sending TCP segments. The `timeout server` setting measures inactivity when we’d expect the backend server to be speaking. When a timeout expires, the connection is closed. Having sensible timeouts reduces the risk of deadlocked processes tying up a connections that could otherwise be reused.

When operating HAProxy in TCP mode, which is set with `mode tcp`, `timeout server` should be the same as `timeout client`. That’s because HAProxy doesn’t know which side is supposed to be speaking and, since both apply all the time, having different values makes confusion more likely.

## Frontend
When you place HAProxy as a reverse proxy in front of your backend servers, a `frontend` section defines the IP addresses and ports that clients can connect to. You may add as many `frontend` sections as needed for exposing various websites to the Internet. Each `frontend` keyword is followed by a label, such as **fe_main**, to differentiate it from others.

Consider the following example:
<pre class="file" data-filename="haproxy.cfg" data-target="append">frontend fe_main 
    bind :80
    use_backend be_stats if { path_beg /haproxy-stats } 
    default_backend be_app 
</pre>

### bind
A `bind` setting assigns a listener to a given IP address and port. The IP can be omitted to `bind` to all IP addresses on the server and a port can be a single port, a range, or a comma-delimited list.

### use_backend
The `use_backend` setting chooses a backend pool of servers to respond to incoming requests if a given condition is true. It is followed by an ACL statement, such as `if path_beg /haproxy-stats`, that allows HAProxy to select a specific backend based on some criteria, such as checking if the path begins with /haproxy-stats.

### default_backend
The `default_backend` setting is found in nearly every `frontend` and gives the name of a `backend` to send traffic to if a `use_backend` rule doesn’t send it elsewhere first. If a request isn’t routed by a `use_backend` or `default_backend` directive, HAProxy will return a 503 Service Unavailable error.

## Backend
A `backend` section defines a group of servers that will be load balanced and assigned to handle requests. You’ll add a label of your choice to each `backend`, such as web_servers. It’s generally, pretty straightforward and you won’t often need many settings here.

### balance
The `balance` setting controls how HAProxy will select the server to respond to the request if no persistence method overrides that selection. A persistence method might be to always send a particular client to the same server based on a cookie. Common load balancing values include `roundrobin`, which just picks the next server and starts over at the top of the list again, and `leastconn`, where HAProxy selects the server with the fewest active sessions.  If `balance` is not defined it defaults to `roundrobin`.

<pre class="file" data-filename="haproxy.cfg" data-target="append">backend be_app
    balance roundrobin 
    server app1 172.18.0.3:80 check
    server app2 172.18.0.4:80 check

backend be_stats
  stats uri /haproxy-stats
</pre>

### server
The `server` setting is the heart of the `backend`. Its first argument is a name, followed by the IP address and port of the backend server. You can specify a domain name instead of an IP address. In that case, it will be resolved at startup or, if you add a `resolvers` argument, it will be updated during runtime. If the DNS entry contains an SRV record, the port and weight will be filled in from it too. If the port isn’t specified, then HAProxy will use the same port that the client connected on, which is useful for randomly used ports such as for active-mode FTP. 

The `check` argument has tells HAProxy to check the health of the application server periodically.  By default it does a basic TCP check but it can be configured using `option httpchk` to send an HTTP request.

