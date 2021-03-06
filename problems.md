Problems and solutions
----------------------

### App Engine/browsers

1.  The `update` endpoint which (re)initializes the SQL database was known to not be safe for concurrent calls. But it
    also started to fail with referential integrity errors after a few sequential invocations.
    
    **Cause** The endpoint was (temporarily) implemented as a HTTP GET-request. It turned out that Chrome - for unknown
    reason - already sent a request when the suggested link (previously visited) was *focused* in the URL bar (without
    showing it in the "Network" tab of the developer tools) and then again when it was actually requested. There were
    thus actually two concurrent requests after all. Furthermore, because of the short time between the requests, the
    App Engine (even in developer mode) would spin up an extra instance which meant that the two update request would be
    handled in different processes that couldn't even agree on sharing a mutex.
    
    **Solution** Calling the endpoint from a HTML form (and making it a proper POST) solved the problem. Making the
    endpoint thread-safe was added as a TODO item.

2.  Could not see standard output (in development mode).
    
    **Cause/solution** For logging a request, one should use the logger methods on an `appengine.Context`. Since there
    is no request in the `init`-function, we can use the `log` package to print the logs. This was done by adding a
    custom `Logger` interface (which the context automatically implements) and add an `InitLogger` implementation. Since
    this will not work on the App Engine, a recorder was added to save the log output and error for later retrieval.

3.  Deployed application could not find database (with various, cryptic errors).
   
    **Cause** The Cloud SQL database was created at the wrong regional location.
    
    **Solution** Deleting it and creating a new one in `europe-west1` solved the problem. Also, "data source name" might
    have been wrong. The/a correct format is `root@cloudsql([project-ID]:europe-west1:[DB-instance-ID])/locations`.

4.  Could not access log output from deployed application. A few of the *debug* logs would show up in the online logger
    tool, but only a few and only if the request didn't time out (which happened a lot - see below).
    
    **Cause** Unknown.
    
    **Solution** Use the log recorder mentioned above and dump it into the HTML.

5.  SQL queries were hopelessly slow. From the dumped log, it appeared that we were only doing 2-4 queries (both
    insertion and select) per second. This caused timeouts because the app couldn't update the database in within the
    60 sec time limit enforced by App Engine.
    
    **Cause** Unknown, but might just be inherent to the Cloud-based architecture. It does appear like the app and
    database might still be in different data centers (despite the fact that recreating the database solved the problem
    above), but it can only be changed by also recreating the app, which isn't necessary at this point.
    
    **Solution** Bulking insertions together (at the cost of slightly increased code complexity) improved performance
    tremendously. The same is the case with selections, and all operations now complete in a at most a few seconds.

6.  After performing an update, a problem happened in IE (11) where links referred URLs containing obsolete movie
    indices.
    
    **Cause** Since no cache headers were present, IE (apparently unlike Firefox and Chrome) choose to cache the data
    containing the IDs. This leads to the observed behavior because the IDs are renewed on updates.
    
    **Solution** Setting the header `Expires: 0` seems to have solved the problem (note that App Engine converts this
    into more robust headers). Ensuring that updates don't break URLs was added as a TODO item.

### Go

1.  The officially recommended tool for package management in Go is `go get`, which has the perplexingly obvious
    shortcoming of not referring to versioned libraries (for ensuring that builds are reproducible and uses stable
    libraries). It also turns one's workspace into a mess unless annoying work-arounds are applied.
    
    **Solution** The [Glide tool](https://glide.sh/) allows one to keep dependency information in a file (`glide.yaml`)
    along the lines of Maven (for Java), npm (node.js), etc. Other package management tools for Go exist as well.
