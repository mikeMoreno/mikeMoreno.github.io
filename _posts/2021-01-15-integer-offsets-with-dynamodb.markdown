Recently my company decided to use [DynamoDB](https://aws.amazon.com/dynamodb/) for its data storage needs. DynamoDB is a [NoSQL](https://en.wikipedia.org/wiki/NoSQL) database and lacks many of the features that one would find in a relational database. One such feature is integer-based offsets.


DynamoDB provides no way to do what would otherwise be a simple query in SQL Server:



```
SELECT *
FROM MyTable
ORDER BY column_name
OFFSET 25 ROWS
FETCH NEXT 25 ROWS ONLY
```


Paging is possible in DynamoDB, but instead of using integers, it uses values.


What this means is when DynamoDB returns the results of your query, it also returns a `LastEvaluatedKey`, which you then send back as 
the `ExclusiveStartKey` the next time you reissue your query. This process continues until the LastEvaluatedKey 
returned is null. At that point, there are no more items in the database that match your query. 
Along with your query, you send a limit. The limit specifies the maximum number of items for DynamoDB to return. 
Every time you reissue your query, DynamoDB will return at most the number of items specified in your limit, *but it may return less*.


This is all well and good if you can get away with value-based paging, but it was a problem for us because we needed 
to make DynamoDB work with integer-based offsets for reasons.


We did this by adding a layer of abstraction that converts an integer-based paging scheme to DynamoDB's value-based paging scheme. 
The layer sits between DynamoDB and our code that needs to query data.


The way it works is we use the concept of a "cursor". We create a cursor for each query that is sent to DynamoDB, 
and this cursor keeps track of the last place a particular query left off in the database, in other words, the LastEvaluatedKey.
You can think of a cursor like a bookmark.


When a user sends a query, along with a limit and offset, we either have a cursor stored for that query or we don't.


If we don't have a cursor for the query, we check if the offset is 0, or something greater than 0.


If the offset is 0, we send the user's query to DynamoDB with the user's limit. As mentioned previously, 
DynamoDB may return less than the given limit, even though there are more items to return, so we may have to send the 
query multiple times until the limit is reached, using DynamoDB's value-based paging. From the user's perspective, 
only a single query is sent. It's also possible that the user's limit may exceed the number of items that match the 
query. Once we've satisfied the user's limit, or retrieved as many items as we can, we set a cursor for this query 
with the `LastEvaluatedKey` and return the results.


If the offset is greater than 0, we value-page through items DynamoDB returns until we've reached the user's offset. 
All of the items we page through before the offset we discard. We then reissue the query until we reach the user's limit.
We then set a cursor at that location with the LastEvaluatedKey and return the results as before.


When the user sends a query that we have a cursor for, we simply set the cursor's `LastEvaluatedKey` to the query's 
`ExclusiveStartKey` and query DynamoDB until we reach the user's limit, set another cursor with the new LastEvaluatedKey 
and return the results.


The cursor is just another DynamoDB item. The Partition Key of the cursor is a hash made up of the user's query and 
the limit + offset values. We call the limit + offset the cursor's index. Attributes on the cursor store the 
LastEvaluatedKey, if one was returned.


This scheme is completely transparent to the user, who never needs to know about DynamoDB's value-based paging.
