# View all DB's

> `show dbs`

# View all Collections

> `show collections`

# Create DB

use \<db-name>
> `use users`

# Create Collection

### *__Lazy Creation__*:

> `db.user.insertOne(<document/json>)`
```
db.user.inserOne({name: "Jason", age: 25})
```
### *__Expilicit creation__*: 

> `db.createCollection(<collection-name>, <options>)`
```
db.createCollection("user")
```
# Basic CRUD operations

1. **CREATE**

    * Methods available: 
        * `insertOne` 
        * `insertMany`
        * `insert`
    
    * Commands syntax:
        * `db.<collection>.inserOne(<document>)`
        * `db.<collection>.insertMany(<array of documents>)`
        * `db.<collection>.insert(<document or array of documents>)`

    * Examples:
        * > Insert one document
            ```
                db.user.insertOne({name: "Jason", age: 25})
            ```
        *  > Insert many documents
            ```
                db.uset.insertMany(
                    [
                        {name: "James", age: 41}, 
                        {name: "Ritesh", age: 32}, 
                        {name: "Giorgio", age: 12}
                    ]
                )
            ```
2. **READ**

    * Methods available: 
        * `findOne`
        * `find`

    * Commands syntax:
        * `db.<collection>.findOne(<filter>)`
        * `db.<collection>.find()`
        * `db.<collection>.find(<filter>, <options>)`

    * Examples:
        * > To find the first document where `user["name"] == "James"`
            ```
                db.user.findOne({name: "James"})
            ```
        *  > To find all documents in the user collection
            ```
                db.user.find()
            ```
        *  > To find all docmunets where `user["name"] == "James"`
            ```
                db.user.find({name: "James"})
            ```
        *  > To find all documents where `user["age"] > 25`
            ```
                db.user.find({age: {$gt: 25}})
            ```
    * __PROJECTION__:
        * Projection is a way to further trim down what is displayed after a find command

        * Examples:
            * To find all documents where `user["age"] > 25` and only return the name and _id field
            ```
                db.user.find({age: {$gt: 25}}, {name: 1})
            ```
            * To find all documents where `user["age"] > 25` and only return the name field
            ```
                db.user.find({age: {$gt: 25}}, {name: 1, _id: 0})
            ```
3. **UPDATE**:

    * Methods available: 
        * `updateOne`
        * `updateMany`
        * `update (don't use)`
        * `replaceOne`

    * Commands syntax:
        * `db.<collection>.updateOne(<filter>, <new-data>)`
        * `db.<collection>.updateMany(<filter>, <new-data>)`
        * `db.<collection>.update(<filter>, <new-data>)`
        * `db.<collection>.replaceOne(<filter>, <new-data>)`
    
    * Examples:
        * > To update the first document where `user['name'] == 'James'` with `user['name'] = 'Jonathan'`
            ```
                db.user.updateOne({name: "James"}, {$set: {name: "Jonathan"}})
            ```
        *  > To update all documents where `user['name'] == 'James'` with `user['name'] = 'Jonathan'`
            ```
                db.user.updateMany({name: "James"}, {$set: {name: "Jonathan"}})
            ```
        *  > To update all documents where `user['name'] == 'James'` with `user['name'] = 'Jonathan'`. Don't use UPDATE !!
            ```
                db.user.update({name: "James"}, {$set: {name: "Jonathan"}})
            ```
        *  > To replace first documents where `user['name'] == 'James'` with document `{name: "Jonathan"}`. Don't use UPDATE !!
            ```
                db.user.update({name: "James"}, {name: "Jonathan"})
            ```
        *  > To replace first documents where `user['name'] == 'James'` with document `{name: "Jonathan"}`.
            ```
                db.user.replaceOne({name: "James"}, {name: "Jonathan"})
            ```
4. **DELETE**:

    * Methods available: 
        * `deleteOne`
        * `deleteMany`

    * Commands syntax:
        * `db.<collection>.deleteOne(<filter>)`
        * `db.<collection>.deleteMany(<filter>)`

    * Examples:
        * > To delete first document `user['name'] == James`
            ```
                db.user.deleteOne({name: "James"})
            ```
        *  > To delete all documents where `user['name'] == James`
            ```
                db.user.deleteMany({name: "James"})
            ```

# Accessing Embedded Document

> Sample `users` collection below : 
```
users = [
    {
        "name": "James",
        "age": 24
        "likes": ["books", "games"]
        "address": {
            "houseNo": 4,
            "city": "Kolkata"
        }
    },
    {
        "name": "Rio",
        "age": 56
        "likes": ["travelling", "cricket"],
        "address": {
            "houseNo": 2,
            "city": "Mumbai"
        }
    }
]
```
To access document with `houseNo == 4` under `address` field in `users` collection, we will use the below command :

> `db.users.find({"address.houseNo": 4})`

# lookup() in Aggregate Module

> Sample `users` & `books` collection below : 

```
users = [
    {"_id": 1, "name": "Jonathan"},
    {"_id": 2, "name": "Rio"},
    {"_id": 3, "name": "Livent"}
]

books = [
    {"name": "celetial empire", author: 1},
    {"name": "a day in the desert", author: 3}
]
```

To embedd `user` document in `books` collection under a new field "`creator`", we will use the below command : 

> `db.books.aggregate([{$lookup: {"from": "user", "localField": "author", "foreignField": "_id", "as": "creator"}}])`

# Schema Validation

> Sample `users` collection below : 

```
users = [
    {
        "name": "James",
        "age": 24,
        "likes": ["books", "games"],
        "address": {
            "houseNo": 4,
            "city": "Kolkata",
        },
    },
]
```

Suppose we want to make sure that every time a new document is inserted in `users` collection, the mentioned above fields are always present, we will use the below command : 

```
db.createCollection(
    "user", 
    {validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["name", "age", "likes", "address"],
            description: "json schema validation for user collection",
            properties: {
                "name": {
                    bsonType: "string",
                    description: "is required and is string"
                },
                "age": {
                    bsonType: "double",
                    description: "is required and is interger"
                },
                "likes": {
                    bsonType: "array",
                    minItems: 2,
                    uniqueItems: true,
                    description: "is required and is array"
                },
                "address":{
                    bsonType: "object",
                    required: ["houseNo", "city"],
                    properties: {
                        "houseNo": {
                            bsonType: "double",
                            description: "is required and is interger"
                        },
                        "city": {
                            bsonType: "string",
                            description: "is required and is string"
                        }
                    }
                }
            }
        }
    }
})
```
# Modifying Schema Validation

Below we changed the default **_validationAction_** from **_err_** to **_warn_**. We can also modify other field params here.

```
db.runCommand({
    collMod: "user", 
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["name", "age", "likes", "address"],
            description: "json schema validation for user collection",
            properties: {
                "name": {
                    bsonType: "string",
                    description: "is required and is string"
                },
                "age": {
                    bsonType: "double",
                    description: "is required and is interger"
                },
                "likes": {
                    bsonType: "array",
                    minItems: 2,
                    uniqueItems: true,
                    description: "is required and is array"
                },
                "address":{
                    bsonType: "object",
                    required: ["houseNo", "city"],
                    properties: {
                        "houseNo": {
                            bsonType: "double",
                            description: "is required and is interger"
                        },
                        "city": {
                            bsonType: "string",
                            description: "is required and is string"
                        }
                    }
                }
            }
        }
    },
    validationAction: "warn",
})
```
