Logging serves:

- Debugging
- Server operations
- Security/intrusion
- Auditing
- Business analytics
- Marketing

####Debugging
- Possible to log stack trace with `console.trace()`
- You can log time to completion:
```javascript
console.time('a timer');
// code here e.g. a request and 
// time to create the response
console.timeEnd('a timer');
```

####Server Operations
Good idea to collect all logs in one log aggregation service, e.g. ELK

####Security
- Who was involved
- What was compromised
- When it was compromised
- How it was compromised
- Which systems were involved
- Attack vectors (intruder path and sequence of events)

Taking the system offline preserves much of the evidence.

For virtual machines, you can take snapshots. Launch an offline instance, isolate the compromised VM from the internal network and lure the intruder in.

####Auditing
Every auth or authorisation attempt should be logged in systems which require for certain policies to be verified.

####Business analytics
- Viral factor (K-factor): `k = i x c`, where i = number of invites sent out by customers before they go inactive, and c = invite conversion rate. k should be greater than 1. k = 1 means your service audience is constant.
- Churn rate: percentage of users who stop using the service from one month to the next
- Monthly reccurring revenue: The amount of revenue earned per customer per month.
- Customer acquisition cost: must link conversions to campaign spending
- Customer lifetime value: how much a customer spend during the lifetime of his activity

####Logging checklist
- Requests from clients (GET, POST, etc.)
- Errors
- Failure alerts
- Conversion goals (sales, social-media shares, etc.)
- Authorizations (user requesting a protected resource)
- Feature activations (moving between pages and views, etc.)
- Performance metrics
- Server response time
- Time to page-ready event
- Time to page-rendered event

####Logging request
Install `bunyan` package from npm. Code on Logging Request section can be used as is or install the `bunyan-request-logger` package which stitches all this code together.

You need to create a custom serialiser and use this to create request and error logging middleware, before your application. Also, a *pixel tracking* middleware should go before your application.

For error handling, pass in the `server` object to the `errorHandler` middleware (`express-error-handler` package) so it can gracefully shutdown the server.

####Sample Log Output
Everything is JSON. ELK should be just fine.

####Logging Service Alerts
Alerts, service start and stop should be logged.

####Profiling and instrumentation
Use newRelic and AppDynamics among others. There are npm packages implementing those services APIs.

####Logging client-side events
Pixel-tracking should do fine. Look it up at the Logging Requests section of this chapter.

####Misc
Splunk, Kissmetrics and Mixapel are fine tools to connect your logging mechanisms.