## TALK SCOPE
```text
* What is NoSQL Injection (NoSQLi) ?
* How does NoSQLi compare to SQLi
* Evaulate MOngoDb's claim that "traditional SQL injection attacks are not a problem" in MongoDb
* Evalaute how MondoDb can be exploited through BSON Injection.
* Understand the execution contexts that queries are evaluated in (and how can be exploited).

```

## What is NoSQL Injection (NoSQLi) ?
* Introduced when developers create dynamic database queries that includes user supplied input.
  * Untrusted input
  * Can contain the typical types : strings (code), ints (numbers), etc.
    * NoSQLi can also contain query objects for injection.
   
