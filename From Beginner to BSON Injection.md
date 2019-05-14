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
	* Can contain the typical types: strings(code), ints (numbers), etc.
  * NoSQLi can also contain query objects.

## SQL Injection: Fundamental Thinking

* Understanding the thought process behind SQLi will help us understand NoSQLi.

```javascript
let sqlStatement = `SELECT * FROM accounts WHERE username = '${username_value}' AND password = '${password_value}'`

model.sequelize.query(sqlStatement)

```

* When `sqlStatement` is passwd to `query()` as a string, what functionality does this inhibit?
	* `query()` has no way to scope `username_value` to `username`
	* Problems?
		* As `username_value` isn't scoped to `username` it can affect the meaning of other items that come after it.

##  SQL VS NOSQL INJECTION

```text
let username_value = "admin' --"
let password_value = "i dont matter"
let sqlStatement = `SELECT * FROM accounts WHERE username='${username_value}' AND password= '${password_value}'`
model.sequelize.query(sqlStatement)

Resulting SQL
	* SELECT * FROM accounts WHERE username = 'admin' -- `AND password = 'i dont matter'`

Typical MongoDb Equivalent

	```javascript
		db.accounts.find({username: username_value, password: password_value})

	```

```

* `username_value` is scoped to `username`
	* Is this injectable ?
	* Visit the MongoDb Official Docs.

## MONGOS NoSQLi Response
* MongoDB represents queries as BSON objects (Binary JSON), basically JSON gets serialized in the Binary form called BSON.

* Typically client libraries provide a convenient, injection free, process to build these objects. Consider the following C++ example:
```C++
// db.accounts.find({username: username_value});

BSONobj my_query = BSON("username" < username_value);
auto_ptr<dbclientcursor> cursor = c.query("accounts, my_query");
```

* As a client program assembles a query in MongoDB, it builds a BSON object, not a string. Thus *traditional* SQL Injection attacks are not a problem.
	* client program = client library

* If `my_query` contained special characters, for example `,`, `:`, and `{`, the query wouldn't match any documents. Cause they don't have any meaning to the JSON objects. For example, users cannot hijack a query and convert it to a delete.
	* `let username_value = "admin' --"`
		* Special characters were used to alter the meaning of the SQL query.

## Mongos NoSQLi Response 
```javascript

db.accounts.find({username: username_value, password: password_value});
```

* Mongo's statement about the lack of injection vulnerabilities assumes the input will be passed in a certain way.
	* What is the input assumption ?
		* String.

## BSON INJECTION

```javascript

{"username": "admin"}
..snip..
\x02				// 0x02 = type String
username\x00		// field name
\x06\x00\x00\x00admin\x00 // field value
\x00 				// 0x00 = type EOO ('end of object')

```
	
* How could a string potentially exploit this BSON  object?
	* Insert a BSON special character/delimiter: 0x00
		* Similar idea to the `'` within `${username_value}`
	* Insert BSON directly
		* Nested BSON object.
	* Insert hex/binary directly
	* Insert garbage that isn't BSON and cause a DoS.
	

## BSON-RUBY INJECTION: BACKGROUND
* BSON-Ruby Background
	* `Mongoid` is an Ruby ODM (Object-Document-Mapper) for MongoDB.
		* Leveraged a lower-level adapter called Moped
			* Moped leveraged the BSON-Ruby library.
* Ruby Regex Background
	* \A and \z match the start and end of the string.
	* ^ and $ match the start/end of a line.
		* $ matches a /n
			* Other languages this matches the end of a string.
* Issue in the wild with bson-ruby.

```ruby
# Determine if the provided string is legal object id (hex string)

def legal?(string)
	string.to_s =~ /^[0-9a-f]{24}$/i ? true : false
end

```

### How can this hex check be exploited ?
```ruby
b = ((defined?(Moped::BSON) ? Moped::BSON : BSON)::ObjectId)
raise "Dos!" if b.legal? "a"*24+"\n"
raise "Injection!" if b.legal? "aaaaaaaaaaaaaaaaaaaaaaa\na"
```

*Takeaways*
	*Contrary to what organizations say, injection is always a risk when you take into account all contexts that a query is evaluated in.
