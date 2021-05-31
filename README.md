# View all DB's

```javascript
    show dbs
```

# View all Collections

```javascript
    show collections
```

# Create DB

```javascript
    use users
```

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
        * `db.<collection>.insertOne(<document>)`
        * `db.<collection>.insertMany(<array of documents>)`
        * `db.<collection>.insert(<document or array of documents>)`

    * Examples:
        * Insert one document
            ```javascript
                db.user.insertOne({name: "Jason", age: 25})
            ```
        * Insert many documents
            ```javascript
                db.user.insertMany(
                    [
                        {name: "James", age: 41}, 
                        {name: "Ritesh", age: 32}, 
                        {name: "Giorgio", age: 12}
                    ]
                )
            ```

        * Save v/s Insert
            ```javascript
                // save() inserts a new document if _id is provided and its not present already & if _id is present, then it updates the current document 
                db.user.save({_id: 1, name: "James"})
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
    
    * remove()
        * _remove_ does the same thing as that of _delete_ and _deleteOne_, the difference is the return from remove and delete

        * Example:
            ```javascript
            db.user.remove({name: "James"})
            db.user.remove({name: "James"}, {justOne: true})
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
    1. > **$eq**
        ```javascript
            db.users.find({age: {$eq: 40}})` or `db.users.find({age: 40})
        ```
    2. > **$gt**
        ```javascript
            db.users.find({age: {$gt: 40}})
        ```
    3. > **$gte**
        ```javascript
            db.users.find({age: {$gte: 40}})
        ```
    4. > **$in**
        ```javascript
            db.users.find({age: {$in: [40, 50]}})
        ```
    5. > **$lt**
        ```javascript
            db.users.find({age: {$lt: 40}})
        ```
    6. > **$lte**
        ```javascript
            db.users.find({age: {$lte: 40}})
        ```
    7. > **$ne**
        ```javascript
            db.users.find({age: {$ne: 40}})
        ```
    8. > **$nin**
        ```javascript
            db.users.find({age: {$nin: [40, 50]}})
        ```

* **_Logical_**
    1. > **$and**
        ```javascript
            db.users.find({$and: [{salary: {$gt: 10000}}, {age: 40}]})
        ```
    2. > **$nor**
        ```javascript
            db.users.find({$nor: [{salary: {$gt: 10000}}, {age: 40}]})
        ```
    3. > **$not**
        ```javascript
            db.users.find({$not: {salary: {$gt: 10000}}})
        ```
    4. > **$or**
        ```javascript
            db.users.find({$or: [{salary: {$gt: 10000}}, {age: 40}]})
        ```

* **_Element_**
    1. > **$exist**
       ```javascript
            db.users.find(name: {$exist: true})
            db.users.find(name: {$exist: true, $eq: "John"})
       ```
    2. > **$type**

        ```javascript
            db.users.find(name: {$type: "string"})
            db.users.find(name: {$type: "string", $eq: "John"})
        ```

* **_Evaluation_**
    1. > **$expr**

        Expression is bit more complex and we will try to understand it using two examples

        * **_Example 1_** : Finding all users whose salary is greater than expense
            ```javascript
                db.users.find({$expr: {$gt: ["$salary", "$expense"]}})
            ```
          
        * **_Example 2_**: Using $cond (if, else, then)
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
       > See [schema validation](#Schema-Validation) section

    3. > **$mod**
        ```javascript
            db.users.find(age: {$mod: [2, 0]})
        ```
            

    4. > **$regex**
        ```javascript
            db.users.find(name: {$regex: /^Jhon/, $options: "i"})
        ```
        or
        ```javascript
            db.users.find(name: {$regex: /^Jhon/})
        ```
     
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

    1. > **$all**
        ```javascript
            db.users.find({hobbies: {$all: ["sports", "drawing", "hiking"]}})
        ```
    2. > **$elemMatch**

        **Syntax** : `{<field>: {$elemMatch: {<query1>, <query2>, ... }}}`
        ```javascript
            db.scores.find({results: {$elemMatch: {$gte: 80, $lt: 85}}})
        ```
       
    3. > **$size**
        ```javascript
            db.users.find({hobbies: {$size: 3}})
        ```
* **_Cursors_**
    
    1. > **$sort**\
     1 is for ascending and -1 is for descending

        ```javascript
            db.users.find().sort({hobbies: 1})  
        ```

    2. > **$skip**
        ```javascript
            db.users.find().skip(10)
        ```

    3. > **$limit**
        ```javascript
            db.users.find().limit(10)
        ```
    
    4. > Difference between **count()** and **size()**\
       > **count :** Returns the total items in a cursor\
       > **size :** Returns the total items after skip and limit has been applied on a cursor

* **_Projection_**

    1. **Display only name and exclude id & rest**
        ```javascript
            db.users.find({}, {name: 1, _id: 0})
        ```

    2. > **For embedded document**
        ```javascript
            db.users.find({}, {"network.id": 1})
        ```

    3. > **For embedded array**
        ```javascript
            // Example 1: with $
            db.users.find({genres: "action"}, {"genres.$": 1})
            
            // Example 2: with $elemMatch
            db.users.find({genres: "action"}, {{$elemMatch: {$eq: "horror"}}})
            db.users.find({}, {name: 1, _id: 0, {$elemMatch: {$eq: "horror"}}})

            // Example 3: with $slice
            db.users.find({genres: "action"}, {genres: {$slice: 2}}) //skip first 2

            db.users.find({genres: "action"}, {genres: {$slice: [2, 3]}}) //skip first 2 & display rest 3
        ```
    
* ### _Will see **Geospatial**, **Bitwise** later on_


# INSERT Operation


### **importing data from a json file**

> `mongoimport <file-name.json> -d <db-name> -c <collection-name> --jsonArray --drop --maintainInsertionOrder`

* **--jsonArray** : Specifies that the document contains an array of json objects

* **--drop** : Specifies to drop the collection if a collection by same name exists. If not provided, imported data will be appended to the existing collection

* **--maintainInsertionOrder**: Maintains the order of the json array while importing

### **ordered** insertion

By default, insert operation in MongoDB is ordered. So, if an array of documents are tried to be added at the same time and if there is error inserting one document in that, then all documents following that error document will also not be added.

We can change that behaviour using **ordered** option.

```javascript
db.user.inserMany([{_id: 1, name: 'John', age: 45}, {_id: 2, name: 'Ray', age: 25}], {ordered: false})
```

### **writeConcern**

There is a **journal** file in MongoDB which queues all the task that needs to be done before it is actually done on the DB. 

We can make sure that journal file has been written for our operation using the below command.


```javascript
db.user.insertOne({_id: 3, name: 'Tomas', age: 55}, {writeConsern: {w: 1, j: true, wtimeout: 200}})
```

# READ operation

**_See Query and Projection Operators in [OPERATORS](#operators)_**

# UPDATE operation

1. **$set**
    ```javascript    
    db.users.updateMany({name: 'Jason'}, {$set: {age: 40}})

    // use upsert option to add a new document if there isn't one already
    db.users.updateMany({name: 'Jason'}, {$set: {age: 40}}, {upsert: true})
    ```

2. **$inc**
    ```javascript
    // increments age by 1    
    db.users.updateMany({name: 'Jason'}, {$inc: {age: 1}})
    ```

3. **$min**
    ```javascript
    // sets the value of age to 35 if the current value is greater than 35
    db.users.updateMany({name: 'Jason'}, {$min: {age: 35}})
    ```

4. **$max**
    ```javascript
    // sets the value of age to 55 if the current value is less than 55
    db.users.updateMany({name: 'Jason'}, {$max: {age: 55}})
    ```

5. **$mul**
    ```javascript
    // multiplies 1.10 to current age value and sets it
    db.users.updateMany({name: 'Jason'}, {$mul: {age: 1.10}})
    ```

6. **$rename**
    ```javascript
    db.users.updateMany({name: 'Jason'}, {$rename: {age: "totalAge"}})
    ```

7. **$unset**
    ```javascript
    // removes age field
    db.users.updateMany({name: 'Jason'}, {$unset: {age: 1}})
    ```

8. **$**
    ```javascript
    // overwrite first matching document with new fields
    db.users.updateMany({"hobbies.frequency": {$gte: 3}}, {$set: {"hobbies.$": {title: "Sports", frequency: 5}}})
    ```

9. **$.**
    ```javascript
    // adds a new field in the matching filter
    db.users.updateMany({"hobbies.frequency": {$gte: 3}}, {$set: {"hobbies.$.goodFrequency": true}})
    ```

10. **$[] **
    ```javascript
    // adds a new field in all array items
    db.users.updateMany({"hobbies.frequency": {$gte: 3}}, {$set: {"hobbies.$[].goodFrequency": true}})
    ```

11. **$[\<identifier\>], arrayFilters**
    ```javascript
    // adds a new field in the filtered array items
    db.users.updateMany({"hobbies.frequency": {$gte: 3}}, {$set: {"hobbies.$[el].goodFrequency": true}}, {arrayFilters: [{"el.frequency": {$gte: 3}}]})
    ```

12. **$push**
    ```javascript
    // adds item to the array
    db.users.updateMany({name: "Chris"}, {$push: {hobbies: "Music"}})
    ```


13. **$pop**
    ```javascript
    // removes first item from the array
    db.users.updateMany({name: "Chris"}, {$pop: {hobbies: -1}})

    // removes last item from the array
    db.users.updateMany({name: "Chris"}, {$pop: {hobbies: 1}})
    ```


14. **$pull**
    ```javascript
    // syntax
    { $pull: { <field1>: <value|condition>, <field2>: <value|condition>, ... } }
    ```
    ```javascript
    // example collection
    {
        _id: 1,
        fruits: [ "apples", "pears", "oranges", "grapes", "bananas" ],
        vegetables: [ "carrots", "celery", "squash", "carrots" ]
    }
    {
        _id: 2,
        fruits: [ "plums", "kiwis", "oranges", "bananas", "apples" ],
        vegetables: [ "broccoli", "zucchini", "carrots", "onions" ]
    }

    // pull can remove items from list when a condition matches
    db.stores.updateMany({}, {$pull: {fruits: { $in: [ "apples", "oranges" ] }, vegetables: "carrots"}}, {multi: true})
    ```

15. **_Modifiers_**
    * **$each :** Can be used with $push or $addToSet
        ```javascript
        //$addToSet
        db.inventory.updateOne({_id: 2 }, {$addToSet: {tags: {$each: ["camera", "electronics", "accessories"]}}})
        
        //$push
        db.inventory.updateOne({_id: 2 }, {$push: {tags: {$each: ["camera", "electronics", "accessories"]}}})
        ```
    * **$sort :** To be used with each
        ```javascript       
        db.inventory.updateOne({_id: 2 }, {$push: {tags: {$each: ["camera", "electronics", "accessories"], $sort: -1}}})
        ```
    * **$slice :** To be used with each
        ```javascript       
        db.inventory.updateOne({_id: 2 }, {$push: {tags: {$each: ["camera", "electronics", "accessories"], $slice: 1}}})
        ```
    * **$position :** To be used with each
         ```javascript       
        db.inventory.updateOne({_id: 2 }, {$push: {tags: {$each: ["camera", "electronics", "accessories"], $position: 1}}})
        ```

# DELETE operation

**_See Delete section in [Basic CRUD operations](#basic-crud-operations)_**

# EXPLAIN

* explain() lets us view how mongodb has planned our query
* **Syntax :** `db.<collection>.explain(<verbosity-mode>).find()`
* Verbosity-Mode :
    1. _**queryPlanner** (default)_
    2. _**executionStats**_
    3. _**allPlansExecution**_

# INDEXES

* Notes:
    1. Used to speed up quries but can also slow it down aswell as the write operation.
    2. By `default` there is always a index that is created on the _id field.
    3. **COLLSCAN** (_Collection Scan_) : Searching through the collection (default if no idexes present).
    4. **IXSCAN** (_Index Scan_) : Seaching using the index.
    5. We should use indexes for sorting also as MongoDB has a threshold of 32MB to in-memory data.


* View Index:
    ```javascript
    // returns list of all index keys and names
    db.users.getIndexes()

    // returns list of all indexe keys
    db.users.getIndexKeys()
    ```
* Drop Index:
    ```javascript
    // Using index key: db.users.dropIndex({<index-key>})
    db.users.dropIndex({"dob.age": 1})

    // Using index name: db.users.dropIndex(<index-name>)
    db.users.dropIndex("dob.age_1")
    ```

* Types of indexes: 
    1. **Single Field Index**
        ```javascript
        // Example: index on age field in ascending order
        db.users.createIndex({"age": 1})

        // Example: index on age field in descending order
        db.users.createIndex({"age": -1})
        ```
    2. **Compound Index**
        ```javascript
        db.users.createIndex({"age": 1, "gender": 1})
        ```
    3. **Configure Index**
        * _Unique Index_
            ```javascript
            // unique index can be used to make sure that a field is unique (other way is jsonValidation)
            db.users.createIndex({email: 1}, {unique: true})
            ```
        * _Partial Index_
             ```javascript
            // create index on age filed but for only males
            db.users.createIndex({"age": 1}, {partialFilterExpression: {gender: "male"}})

            // create index on age filed but for only people who are 60 or above
            db.users.createIndex({"age": 1}, {partialFilterExpression: {age: {$gte: 60}}})     
            ```
        * _Unique with Partial Index_
            ```javascript
            // create index on age filed but for only males
            db.users.createIndex({"email": 1}, {unique: true, partialFilterExpression: {email: {$exists: true}}})

            // create index on age filed but for only people who are 60 or above
            db.users.createIndex({"age": 1}, {partialFilterExpression: {age: {$gte: 60}}})
            ```

    4. **TTL Index** 
        * TTL index can be used to destroy a collection/document after a specified amount of time
        * Good for session related collection
        * Can be created only on _date time field_ & on _single field indexes_
        * 
            ```javascript
            db.session.insertOne({data: "Some data", createdAt: new Date()})
            {
                data: "Some data",
                createdAt: ISODate("2020-05-29T07:05:02.242Z")
            }
            // below creates a TTL index
            db.session.createIndex({createdAt: 1}, {expireAfterSeconds: 10})
            ```

    5. **Covered Index & Queries**
        * A covered query is a scenario where MongoDB doesn't have to scan the collection and can just return result based on the index.

        * Example:
        ```javascript
        // Imagine a collection which has a name field and a index like below exists
        db.users.createIndex({name: 1})

        // Now if we do a query on name field and using projection return only name, then such a scenario will be called as a covered query/index
        db.users.find({name: 'John'}, {_id: 0, name: 1})
        ```
    6. **Winning Plan**
        * Mongo db uses winning plan to find out which index to use
        * It lets the chooses plan to run for 100 first doc and which ever returns the result fastest is used.
        * This winning plan is cached in memory for this exact query
        * The cache is cleared after **1000 writes** or **Update/Drop/Create Index** or **Restart MongoDB**

    7. **Multikey Indexes**
        * When a index is created on a array field, then mongoDB creates a multi-key index
        ```javascript
        // example collection
        {
            "_id" : 1,
            "name" : "Max",
            "hobbies" : [
                    "Cooking",
                    "Sports"
            ],
            "addresses" : [
                    {
                            "street" : "Second St."
                    },
                    {
                            "street" : "Main St."
                    }
            ]
        }

        // below index will be an multi-key index
        db.users.createIndex({hobbies: 1})
        db.users.createIndex({addresses: 1})
        db.users.createIndex({"addresses.street": 1})
        ```
        * Compoud Index with on two array field is not possible
        * Compound Index witha a single field and an array field is possible

    8. ### **Text Index**
        * Text index is another type of Multikey Index used to search text
        * _Example :_
            ```javascript
            db.store.createIndex({description: "text"})

            // to search, we use the below query
            db.store.find({$text: {$search: "book"}})
            ```
        * _Combined Text Indexes_ : We can't have more than one text index on a collection and thus we can use _combined text collection_
            ```javascript
            db.store.createIndex({description: "text", title: "text"})
            ```
        * _Exclude words from Text Search_
            ```javascript
            db.store.createIndex({$text: {$search: "awesome -red"}})    
            ```
        * _Text Score & Sorting via Text Score_
            1. To see the text score:
                ```javascript
                db.store.find({$text: {$search: "awesome red"}}, {score: {$meta: "textScore"}})
                ```
            2. To sort via text score:
                ```javascript
                db.store.find({$text: {$search: "awesome red"}}).sort({score: {$meta: "textScore"}})
                ```
            3. Set Score Weight
                ```javascript
                    db.strore.createIndex({title: "text", description: "text"}, {weights: {text: 1, description: 10}})
                ```
        * _Set Default Language_
            ```javascript
                db.strore.createIndex({title: "text", description: "text"}, {defaultLanguage: "german"})
            ```
    9. **Create Index in Background** 
        ```javascript
            // doesn't block other operations on DB while insetion is happening
            db.store.createIndex({description: "text"}, {background: true})
        ```
