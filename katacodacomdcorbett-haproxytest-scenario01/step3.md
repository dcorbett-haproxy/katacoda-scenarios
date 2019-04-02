## Statistics Page
HAProxy provides a verbose statistics page.  This should already be enabled and can be accessed from a separate browser window using the following URL:
https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/haproxy-stats

The HAProxy statitics page provides a wealthy of information about the running HAProxy process as well as its frontends and backends.

This same information can be gathered from the Runtime API which allows feeding this data into an orchestration system or integrating it with various monitoring tools.  We'll give an introduction to the Runtime API in the next section.
