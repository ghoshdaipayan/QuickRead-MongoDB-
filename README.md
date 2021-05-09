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
```javascript
db.user.inserOne({name: "Jason", age: 25})
```
### *__Expilicit creation__*: 

> `db.createCollection(<collection-name>, <options>)`
```javascript
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
        * Insert one document
            ```javascript
                db.user.insertOne({name: "Jason", age: 25})
            ```
        * Insert many documents
            ```javascript
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
        * To find the first document where **_user["name"] == "James"_**
            ```javascript
                db.user.findOne({name: "James"})
            ```
        * To find all documents in the **_user_** collection
            ```javascript
                db.user.find()
            ```
        * To find all docmunets where **_user["name"] == "James"_**
            ```javascript
                db.user.find({name: "James"})
            ```
        * To find all documents where **_user["age"] > 25_**
            ```javascript
                db.user.find({age: {$gt: 25}})
            ```
    * PROJECTION :
        * Projection is a way to further trim down what is displayed after a find command

        * Examples:
            * To find all documents where **_user["age"] > 25_** and only return the name and _id field
            ```javascript
                db.user.find({age: {$gt: 25}}, {name: 1})
            ```
            * To find all documents where **_user["age"] > 25_** and only return the name field
            ```javascript
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
        * To update the first document where **_user['name'] == 'James'_** with **_user['name'] = 'Jonathan'_**
            ```javascript
                db.user.updateOne({name: "James"}, {$set: {name: "Jonathan"}})
            ```
        * To update all documents where **_user['name'] == 'James'_** with **_user['name'] = 'Jonathan'_**
            ```javascript
                db.user.updateMany({name: "James"}, {$set: {name: "Jonathan"}})
            ```
        * To update all documents where **_user['name'] == 'James'_** with **_user['name'] = 'Jonathan'_**. Don't use UPDATE !!
            ```javascript
                db.user.update({name: "James"}, {$set: {name: "Jonathan"}})
            ```
        * To replace first documents where **_user['name'] == 'James'_** with document **_{name: "Jonathan"}_**. Don't use UPDATE !!
            ```javascript
                db.user.update({name: "James"}, {name: "Jonathan"})
            ```
        * To replace first documents where **_user['name'] == 'James'_** with document **_{name: "Jonathan"}_**.
            ```javascript
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
        * To delete first document **_user['name'] == James_**
            ```javascript
                db.user.deleteOne({name: "James"})
            ```
        * To delete all documents where **_user['name'] == James_**
            ```javascript
                db.user.deleteMany({name: "James"})
            ```

# Accessing Embedded Document

Sample **_users_** collection below : 
```javascript
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
To access document with **_houseNo == 4_** under **_address_** field in **_users_** collection, we will use the below command :

> `db.users.find({"address.houseNo": 4})`

# lookup() in Aggregate Module

Sample **_users_** & **_books_** collection below : 

```javascript
users = [
    {"_id": 1, "name": "Jonathan"},
    {"_id": 2, "name": "Rio"},
    {"_id": 3, "name": "Livent"}
]

books = [
    {"name": "celetial empire", "author": 1},
    {"name": "a day in the desert", "author": 3}
]
```

To embedd **_user_** document in **_books_** collection under a new field **_creator_**, we will use the below command : 

> `db.books.aggregate([{$lookup: {"from": "user", "localField": "author", "foreignField": "_id", "as": "creator"}}])`

# Schema Validation

Sample **_users_** collection below : 

```javascript
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

Suppose we want to make sure that every time a new document is inserted in **_users_** collection, the mentioned above fields are always present, we will use the below command : 

```javascript
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

* Below we changed the default **_validationAction_** from **_err_** to **_warn_**. Changing to **_warn_** will allow MongoDB to accept the insertion/update even if schema validation fails. It will simply log an warning message in the log file.

* We can also modify other field params here.

```javascript
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


# Operators

1. Query and Projection Operators
2. Update Operators
3. Aggregation Pipeline Stages
4. Aggregation Pipleline Operators

### **Query Selectors**

* **_Comparision_**
    1. > **$eq**\
       > `db.users.find({age: {$eq: 40}})` or `db.users.find({age: 40})`
    2. > **$gt**\
       > `db.users.find({age: {$gt: 40}})`
    3. > **$gte**\
       > `db.users.find({age: {$gte: 40}})`
    4. > **$in**\
       > `db.users.find({age: {$in: [40, 50]}})`
    5. > **$lt**\
       > `db.users.find({age: {$lt: 40}})`
    6. > **$lte**\
       > `db.users.find({age: {$lte: 40}})`
    7. > **$ne**\
       > `db.users.find({age: {$ne: 40}})`
    8. > **$nin**\
       > `db.users.find({age: {$nin: [40, 50]}})`

* **_Logical_**
    1. > **$and**\
       > `db.users.find({$and: [{salary: {$gt: 10000}}, {age: 40}]})`
    2. > **$nor**\
       > `db.users.find({$nor: [{salary: {$gt: 10000}}, {age: 40}]})`
    3. > **$not**\
       > `db.users.find({$not: {salary: {$gt: 10000}}})`
    4. > **$or**\
       > `db.users.find({$or: [{salary: {$gt: 10000}}, {age: 40}]})`

* **_Element_**
    1. > **$exist**\
       > `db.users.find(name: {$exist: true})` or `db.users.find(name: {$exist: true, $eq: "John"})`
    2. > **$type**\
       > `db.users.find(name: {$type: "string"})` or `db.users.find(name: {$type: "string", $eq: "John"})`

* **_Evaluation_**
    1. > **$expr**

        Expression is bit more complex and we will try to understand it using two examples

        * Example 1 : Finding all users whose salary is greater than expense
          > `db.users.find({$expr: {$gt: ["$salary", "$expense"]}})`
          
        * Example 2: Using $cond (if, else, then)
            ```javascript
            db.users.find(
                {
                    $expr: {
                        $gt: [
                            {
                                $cond: {
                                    if: {$gt: ["$salary", 15000]},
                                    then: {$subtract: ["$salary", 5000]},
                                    else: "$salary"
                                }
                            },
                            "$expense"
                        ]
                    }
                }
            )
            ```
    2. > **$jsonSchema**\
       > See schema validation section

    3. > **$mod**\
       > `db.users.find(age: {$mod: [2, 0]})`

    4. > **$regex**\
       > `db.users.find(name: {$regex: /^Jhon/, $options: "i"})` or `db.users.find(name: {$regex: /^Jhon/})`
      
       Valid Options: 
        * i
        * m
        * x
        * s

    5. > **$text**\
       > **_Will check this later_**

    6. > ~~**$where**~~\
       > **_Don't Use_**

* **_Array_**

    1. > **$all**\
       > `db.users.find({hobbies: {$all: ["sports", "drawing", "hiking"]}})`
    2. > **$elemMatch**
    
        **Syntax** : `{<field>: {$elemMatch: {<query1>, <query2>, ... }}}`
       > `db.scores.find({results: {$elemMatch: {$gte: 80, $lt: 85}}})` or ``
    3. > **$size**\
       > `db.users.find({hobbies: {$size: 3}})`

> ### _Will see **Geospatial**, **Bitwise**, **Projection Operators** later on_


# INSERT Operation


### **importing data from a json file**

> `mongoimport <file-name.json> -d <db-name> -c <collection-name> --jsonArray --drop`

* **--jsonArray** : Specifies that the document contains an array of json objects

* **--drop** : Specifies to drop the collection if a collection by same name exists. If not provided, imported data will be appended to the existing collection

### **ordered** insertion

By default, insert operation in MongoDB is ordered. So, if an array of documents are tried to be added at the same time and if there is error inserting one document in that, then all documents following that error document will also not be added.

We can change that behaviour using **ordered** option.

> `db.user.inserMany([{_id: 1, name: 'John', age: 45}, {_id: 2, name: 'Ray', age: 25}], {ordered: false})`

### **writeConcern**

There is a **journal** file in MongoDB which queues all the task that needs to be done before it is actually done on the DB. 

We can make sure that journal file has been written for our operation using the below command.

> `db.user.insertOne({_id: 3, name: 'Tomas', age: 55}, {writeConsern: {w: 1, j: true, wtimeout: 200}})`

# READ operation

**_See Query and Projection Operators in [OPERATORS](#operators)_**

