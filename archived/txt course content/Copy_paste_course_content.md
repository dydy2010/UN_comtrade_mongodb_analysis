Start / NoSQL Lab / Lectures / Lecture 5 / MongoDB / Exercises / MongoDB Data-Modeling
Data Model Design
DBs / MongoDBData Modeling
Page 200
Embedded Data Models
Embedded data models allow applications to store related pieces of information in the same database record. As a result, applications may need to issue fewer queries and updates to complete common operations.

In general, use embedded data models when:
you have "contains" relationships between entities.
you have one-to-many relationships between entities. In these relationships the "many" or child documents always appear with or are viewed in the context of the "one" or parent documents.
embedding provides better performance for read operations, as well as the ability to request and retrieve related data in a single database operation.
Embedded data models make it possible to update related data in a single atomic write operation.
Documents in MongoDB must be smaller than the maximum BSON document size (= 16 MB)
Normalized Data Models
Normalized data models describe relationships using references between documents.

In general, use normalized data models when:
when embedding would result in duplication of data but would not provide sufficient read performance advantages to outweigh the implications of the duplication.
to represent more complex many-to-many relationships.
to model large hierarchical data sets (like tree structures).

References
docs.mongodb.com - Data Model Design



Start / NoSQL Lab / Lectures / Lecture 5 / MongoDB / Exercises
Restore Database (mongorestore)
DBs / MongoDBTools / MongoDB Database ToolsExercise
Page 194
HINT: Open Command Line or Terminal to execute these commands. Change directory to...

cd nosql_lab\lecture_5\data
Help
mongorestore --help
--archive=<filename>		restore dump from the specified archive file.
--drop				drop each collection before import
--gzip				decompress gzipped input
--dryRun			view summary without importing anything. recommended with verbosity
--nsFrom=<namespace-pattern>	rename matching namespaces, must have matching nsTo
--nsTo=<namespace-pattern>	rename matched namespaces, must have matching nsFrom
Connection Options
mongorestore --host=localhost:27017 ...
mongorestore --uri=mongodb+srv://dbAdmin:dbPassword@cluster0.ypbr6t4.mongodb.net ...
Restore Database
mongorestore --gzip --archive=sample_analytics.gz --drop ^
--host=localhost:27017
preparing collections to restore from
reading metadata for sample_analytics.accounts from archive 'sample_analytics.gz'
reading metadata for sample_analytics.transactions from archive 'sample_analytics.gz'
reading metadata for sample_analytics.customers from archive 'sample_analytics.gz'
restoring sample_analytics.accounts from archive 'sample_analytics.gz'
finished restoring sample_analytics.accounts (1746 documents, 0 failures)
restoring sample_analytics.customers from archive 'sample_analytics.gz'
finished restoring sample_analytics.customers (500 documents, 0 failures)
restoring sample_analytics.transactions from archive 'sample_analytics.gz'
finished restoring sample_analytics.transactions (1746 documents, 0 failures)
no indexes to restore for collection sample_analytics.transactions
no indexes to restore for collection sample_analytics.accounts
no indexes to restore for collection sample_analytics.customers
3992 document(s) restored successfully. 0 document(s) failed to restore.
Restore & Rename Database
mongorestore --gzip --archive=sample_analytics.accounts.gz ^
--nsFrom=sample_analytics.* --nsTo=sample_analytics2.* ^
--host=localhost:27017
preparing collections to restore from
reading metadata for sample_analytics2.accounts from archive 'sample_analytics.accounts.gz'
restoring sample_analytics2.accounts from archive 'sample_analytics.accounts.gz'
finished restoring sample_analytics2.accounts (1746 documents, 0 failures)
no indexes to restore for collection sample_analytics2.accounts
1746 document(s) restored successfully. 0 document(s) failed to restore.


References
docs.mongodb.com - mongorestore

Start / NoSQL Lab / Lectures / Lecture 5 / MongoDB / Exercises
Updating Collection Structure
DBs / MongoDBTools / PyCharmTools / MongoDB ShellData Modeling
Page 209
HINT: Open MongoDB Shell to execute the following commands.

Option 1: Update many via set
See /nosql_lab/lecture_6/exercises/pycharm/93-string-to-int-02.py (Lecture 5 - Exercises)

db.str_to_int.update_many({}, [{"$set": {"nr": {"$toInt": "$nr"}}}])
Option 2: Adding fields via aggregate and $out
See /nosql_lab/lecture_5/homework/pycharm/google_stock_data-exercise.py (Lecture 5 - Exercises)

db.stock_data_raw.aggregate([
    {$addFields: {
        "Date": {
            $dateFromString: {"dateString": "$Date_String"}
        },
    }},
    {$out: "stock_data_clean"}
])
Option 3: Adding nested structures via db.runCommand() in mongosh or db.command() in pymongo
You can find the scripts as JS files in the following foder /nosql_lab/lecture_5/exercises/mongosh/ (Lecture 5 - Exercises)

// init data -> 100-add-nested-structures.js
var db = db.getSiblingDB('unflatten');

db.grades.deleteMany({});

// flat structure (except arrays)
db.grades.insertMany( [
  {
    _id: 1,
    student: "Maya",
    homework: [ 10, 5, 10 ],
    quiz: [ 10, 8 ],
    extraCredit: 0
  },
  {
    _id: 2,
    student: "Ryan",
    homework: [ 5, 6, 5 ],
    quiz: [ 8, 8 ],
    extraCredit: 8
  }
] );


// add nested totals
db.runCommand(
   {
      update: "grades",
      updates: [
         {
           q: { },
           u: [
             {
               $set: {
                 "totals.homework": { $sum: "$homework" } ,
                 "totals.quiz": { $sum: "$quiz" }
               }
             },
             {
               $set: {
                 "totals.finalScore": { $add: [ "$totals.homework", "$totals.quiz", "$extraCredit" ] } }
             }
           ],
           multi: true
         }
      ],
      ordered: false,
      writeConcern: { w: "majority", wtimeout: 5000 }
   }
);

db.grades.find({});
[
  {
    _id: 1,
    student: 'Maya',
    homework: [ 10, 5, 10 ],
    quiz: [ 10, 8 ],
    extraCredit: 0,
    totals: { 
        homework: 25,
        quiz: 18,
        finalScore: 43
    }
  },
  {
    _id: 2,
    student: 'Ryan',
    homework: [ 5, 6, 5 ],
    quiz: [ 8, 8 ],
    extraCredit: 8,
    totals: {
        homework: 16,
        quiz: 16,
        finalScore: 40
    }
  }
]

References
docs.mongodb.com - Update
pymongo - command

Start / NoSQL Lab / Lectures / Lecture 5 / MongoDB / Exercises
Export Data (mongoexport)
DBs / MongoDBTools / MongoDB Database ToolsExercise
Page 190
We will use "mongoexport" in this exercise, which is part of the MongoDB Databse Tools.

HINT: Open Command Line or Terminal to execute these commands. Change directory to...

cd nosql_lab\lecture_5\data
Help
mongoexport --help
--host=<hostname>	mongodb host to connect to (e.g. localhost)
--uri=<mongodb-uri>	mongodb uri connection string (e.g. Atlas-URI)
--type=<type>		the output format, either json or csv
--out=<filename>	output file; if not specified, stdout is used
--jsonArray		output to a JSON array rather than one object per line
--pretty		output JSON formatted to be human-readable
--noHeaderLine		export CSV data without a list of field names at the first line
--jsonFormat=<type>	the extended JSON format to output, either canonical or relaxed (default: relaxed)
Connection Options
mongoexport --host=localhost:27017 ...
mongoexport --uri=mongodb+srv://dbAdmin:dbPassword@cluster0.ypbr6t4.mongodb.net ...
Export JSON
mongoexport --db=sample_training --collection=zips ^
--type=json --out=zips.json ^
"mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net"
File: nosql_lab/lecture_5/data/zips.json
{"_id":{"$oid":"5c8eccc1caa187d17ca6ed16"},"city":"ALPINE","zip":"35014","loc":{"y":33.331165,"x":86.208934},"pop":3062,"state":"AL"}
{"_id":{"$oid":"5c8eccc1caa187d17ca6ed17"},"city":"BESSEMER","zip":"35020","loc":{"y":33.409002,"x":86.947547},"pop":40549,"state":"AL"}
{"_id":{"$oid":"5c8eccc1caa187d17ca6ed18"},"city":"ACMAR","zip":"35004","loc":{"y":33.584132,"x":86.51557},"pop":6055,"state":"AL"}
...
Export CSV
mongoexport --db=sample_training --collection=zips ^
--type=csv --fields=_id,city,zip,loc,pop,state --out=zips.csv ^
--uri=mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
File: nosql_lab/lecture_5/data/zips.csv
_id,city,zip,loc,pop,state
ObjectId(5c8eccc1caa187d17ca6ed16),ALPINE,35014,"{""y"":33.331165,""x"":86.208934}",3062,AL
ObjectId(5c8eccc1caa187d17ca6ed17),BESSEMER,35020,"{""y"":33.409002,""x"":86.947547}",40549,AL
ObjectId(5c8eccc1caa187d17ca6ed18),ACMAR,35004,"{""y"":33.584132,""x"":86.51557}",6055,AL
...
HINT: Column "loc" contains a nested JSON object.

Export Filtered
mongoexport --db=sample_training --collection=zips ^
--type=json --query="{\"state\":\"CA\"}" --out=zips_filtered.json ^
--uri=mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
File: nosql_lab/lecture_5/data/zips_filtered.json
{"_id":{"$oid":"5c8eccc1caa187d17ca6f3a4"},"city":"HAZARD","zip":"90063","loc":{"y":34.044017,"x":118.185432},"pop":61123,"state":"CA"}
{"_id":{"$oid":"5c8eccc1caa187d17ca6f3b3"},"city":"MANHATTAN BEACH","zip":"90266","loc":{"y":33.889594,"x":118.399562},"pop":31984,"state":"CA"}
{"_id":{"$oid":"5c8eccc1caa187d17ca6f3b8"},"city":"DOWNEY","zip":"90241","loc":{"y":33.941612,"x":118.13062},"pop":34348,"state":"CA"}
...
Export as Array & pretty
mongoexport --db=sample_training --collection=zips ^
--type=json --query="{\"state\":\"CA\"}" /jsonArray /pretty --out=zips_array.json ^
--uri=mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
File: nosql_lab/lecture_5/data/zips_array.json
[{
	"_id": {
		"$oid": "5c8eccc1caa187d17ca6f3a4"
	},
	"city": "HAZARD",
	"zip": "90063",
	"loc": {
		"y": 34.044017,
		"x": 118.185432
	},
	"pop": 61123,
	"state": "CA"
},
{
	"_id": {
		"$oid": "5c8eccc1caa187d17ca6f3b3"
	},
	"city": "MANHATTAN BEACH",
	"zip": "90266",
	"loc": {
		"y": 33.889594,
		"x": 118.399562
	},
	"pop": 31984,
	"state": "CA"
},
...
]
Export in Canonical Format
mongoexport --db=sample_training --collection=zips ^
--type=json --query="{\"state\":\"CA\"}" --jsonFormat=canonical --out=zips_canonical.json ^
--uri=mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
File: nosql_lab/lecture_5/data/zips_canonical.json
{"_id":{"$oid":"5c8eccc1caa187d17ca6f3a4"},"city":"HAZARD","zip":"90063","loc":{"y":{"$numberDouble":"34.044017"},"x":{"$numberDouble":"118.185432"}},"pop":{"$numberInt":"61123"},"state":"CA"}
{"_id":{"$oid":"5c8eccc1caa187d17ca6f3b3"},"city":"MANHATTAN BEACH","zip":"90266","loc":{"y":{"$numberDouble":"33.889594"},"x":{"$numberDouble":"118.399562"}},"pop":{"$numberInt":"31984"},"state":"CA"}
{"_id":{"$oid":"5c8eccc1caa187d17ca6f3b8"},"city":"DOWNEY","zip":"90241","loc":{"y":{"$numberDouble":"33.941612"},"x":{"$numberDouble":"118.13062"}},"pop":{"$numberInt":"34348"},"state":"CA"}
...


References
docs.mongodb.com - mongoexport

Start / NoSQL Lab / Lectures / Lecture 5 / MongoDB / Exercises
Import Data (mongoimport)
DBs / MongoDBTools / MongoDB Database ToolsExercise
Page 191
HINT: Open Command Line or Terminal to execute these commands. Change directory to...

cd nosql_lab\lecture_5\data
Help
mongoimport --help
--host=<hostname>		mongodb host to connect to (e.g. localhost)
--uri=<mongodb-uri>		mongodb uri connection string (e.g. Atlas-URI)
--collection=<collection-name>	collection to use
--drop				Drops collection before inserting
--jsonArray			treat input source as a JSONarray
Connection Options
mongoimport --host=localhost:27017 ...
mongoimport --uri=mongodb+srv://dbAdmin:dbPassword@cluster0.ypbr6t4.mongodb.net ...
Import JSON
mongoimport --host=localhost:27017 --db=imp_testing ^
--drop --collection=zips --file=zips.json
connected to: mongodb://localhost:27017/
dropping: imp_testing.zips
29470 document(s) imported successfully. 0 document(s) failed to import.
Import JSON (array)
mongoimport --host=localhost:27017 --db=imp_testing ^
--drop --collection=zips_array --jsonArray --file=zips_array.json
connected to: mongodb://localhost:27017/
dropping: imp_testing.zips_array
1523 document(s) imported successfully. 0 document(s) failed to import.
Import CSV
mongoimport --host=localhost:27017 --db=imp_testing ^
--drop --collection=zips_csv --type=csv --headerline --file=zips.csv
connected to: mongodb://localhost:27017/
dropping: imp_testing.zips_csv
29470 document(s) imported successfully. 0 document(s) failed to import.
HINT: The nested JSON object "loc" was imported as string. Use JSON for nested Documents!


Result
_id,city,zip,loc,pop,state
ObjectId(5c8eccc1caa187d17ca6ed21),CALERA,35040,"{""y"":33.1098,""x"":86.755987}",4675,AL
ObjectId(5c8eccc1caa187d17ca6ed17),BESSEMER,35020,"{""y"":33.409002,""x"":86.947547}",40549,AL
ObjectId(5c8eccc1caa187d17ca6ed20),CENTREVILLE,35042,"{""y"":32.950324,""x"":87.11924}",4902,AL
ObjectId(5c8eccc1caa187d17ca6ed27),COOSA PINES,35044,"{""y"":33.266928,""x"":86.337622}",7985,AL
...
Fix
db.zips_csv.updateMany(
  { loc: { $type: "string" } },
  [
    {
      $set: {
        loc: {
          $function: {
            body: "function(loc){ return JSON.parse(loc) }",
            args: ["$loc"],
            lang: "js"
          }
        }
      }
    }
  ]
)


References
docs.mongodb.com - mongoimport

Start / NoSQL Lab / Lectures / Lecture 5 / MongoDB / Exercises
Dump Database (mongodump)
DBs / MongoDBTools / MongoDB Database ToolsExercise
Page 193
HINT: Open Command Line or Terminal to execute these commands. Change directory to...

cd nosql_lab\lecture_5\data
Help
mongodump --help
--db=<database-name>	database to use
--collection=<collection-name>	collection to use
--archive=<file-path>	dump as an archive to the specified path. If flag is specified without a value, archive is written to stdout
--gzip			compress archive or collection output with Gzip
Connection Options
mongodump --host=localhost:27017 ...
mongodump --uri=mongodb+srv://dbAdmin:dbPassword@cluster0.ypbr6t4.mongodb.net ...
Dump Database
mongodump --db=sample_analytics --gzip --archive=sample_analytics.gz ^
--uri=mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
writing sample_analytics.accounts to archive 'sample_analytics.gz'
writing sample_analytics.customers to archive 'sample_analytics.gz'
writing sample_analytics.transactions to archive 'sample_analytics.gz'
done dumping sample_analytics.accounts (1746 documents)
done dumping sample_analytics.customers (500 documents)
done dumping sample_analytics.transactions (1746 documents)
Dump Collection
mongodump --db=sample_analytics --collection=accounts --gzip --archive=sample_analytics.accounts.gz ^
--uri=mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
writing sample_analytics.accounts to archive 'sample_analytics.accounts.gz'
done dumping sample_analytics.accounts (1746 documents)


References
docs.mongodb.com - mongodump
Previous PageNext Page

Start / NoSQL Lab / Lectures / Lecture 4 / NoSQL
BASE semantics of NoSQL databases
NoSQLRDBMS
Page 155
Eventual consistency is a consistency model used in distributed computing to achieve high availability that informally guarantees that, if no new updates are made to a given data item, eventually all accesses to that item will return the last updated value.

Eventually-consistent services are often classified as providing BASE (Basically Available, Soft state, Eventual consistency) semantics, in contrast to traditional ACID (Atomicity, Consistency, Isolation, Durability) guarantees.

These are the rough definitions of each term in BASE:

B-asically A-vailable: basic reading and writing operations are available as much as possible (using all nodes of a database cluster), but without any kind of consistency guarantees (the write may not persist after conflicts are reconciled, the read may not get the latest write)
S-oft state: without consistency guarantees, after some amount of time, we only have some probability of knowing the state, since it may not yet have converged
E-ventually consistent: If the system is functioning and we wait long enough after any given set of inputs, we will eventually be able to know what the state of the database is, and so any further reads will be consistent with our expectations
ACID

Pros	Cons
They scale out horizontally and work with unstructured and semi-structured data. CP Type DBs support ACID transactional consistency.
Schema-free or Schema-on-read options.
High availability.
While many NoSQL databases are open source and so “free”, there are often considerable training, setup, and developments costs. There are also numerous commercial products available.
Weaker or eventual consistency (BASE) instead of ACID.
Limited support for joins.
Data is denormalized, requiring mass updates (i.e. product name change).
Does not have built-in data integrity (must do in code).

References
dataversity.net/review-pros-cons-different-databases-relational-versus-non-relational
Previous PageNext Page
© 2026 by Revolytics AG	


Start / NoSQL Lab / Lectures / Lecture 4 / MongoDB / Exercises
MongoDB Shell (mongosh)
DBs / MongoDBTools / MongoDB ShellExercise
MongoDB Shell (mongosh)
DBs / MongoDBTools / MongoDB ShellExercise
Page 162
HINT: Open Command Line or Terminal to execute these commands.

Help
mongosh --help
$ mongosh [options] [db address] [file names (ending in .js or .mongodb)]
...
Connection to Cluster
mongosh "mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net"
Current Mongosh Log ID: 6131e167a77c2c70a0d06dfd
Connecting to:          mongodb+srv://@cluster0.ypbr6t4.mongodb.net/
Using MongoDB:          4.4.8
Using Mongosh:          1.0.5
Connection to Single Server (localhost)
mongosh --host localhost:27017
Current Mongosh Log ID: 6131e2d9b373e8c50e0fd873
Connecting to:          mongodb://localhost:27017/?directConnection=true&serverSelectionTimeoutMS=2000
Using MongoDB:          5.0.2
Using Mongosh:          1.0.5
Execute Commands In A Script
mongosh --quiet < script.js
Exit the shell
exit

HINT: Following commands are used after connecting to server.

Show DBs
show dbs
sample_airbnb       55.1 MB
sample_analytics    9.94 MB
sample_geospatial   1.57 MB
sample_mflix        50.8 MB
sample_restaurants  6.23 MB
sample_supplies      991 kB
sample_training     51.1 MB
sample_weatherdata   2.7 MB
test_db              700 kB
admin                377 kB
local               6.39 GB
Show Current Database
db
sample_airbnb
Create Or Switch Database
use test_db
switched to db test_db
Delete Database
db.dropDatabase()
{ ok: 1, dropped: 'test_db' }
Create Collection
db.createCollection('posts')
switched to db test_db
Show Collections
show collections
posts
Delete Collection
db.posts.drop()
true
Insert Row
db.posts.insertOne({
  title: 'Post One',
  body: 'Body of post one',
  category: 'News',
  tags: ['news', 'events'],
  user: {
    name: 'John Doe',
    status: 'author'
  },
  date: Date()
})
{
  acknowledged: true,
  insertedIds: { '0': ObjectId("6131e81db10976d1fa495549") }
}
Insert Multiple Rows
db.posts.insertMany([
  {
    title: 'Post Two',
    body: 'Body of post two',
    category: 'Technology',
    date: Date()
  },
  {
    title: 'Post Three',
    body: 'Body of post three',
    category: 'News',
    date: Date()
  },
  {
    title: 'Post Four',
    body: 'Body of post three',
    category: 'Entertainment',
    date: Date()
  }
])
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("6131e8cdb10976d1fa49554a"),
    '1': ObjectId("6131e8cdb10976d1fa49554b"),
    '2': ObjectId("6131e8cdb10976d1fa49554c")
  }
}
Get All Rows
db.posts.find()
Sort Rows (ascending)
db.posts.find().sort({ title: 1 })
Sort Rows (descending)
db.posts.find().sort({ title: -1 })
Count Rows
db.posts.find().count()
4
Filter and Count Rows
db.posts.find({ category: 'News' }).count()
2
Limit Rows
db.posts.find().limit(2)
Foreach
Results can be iterated over with Javascript.

db.posts.find().forEach(function(doc) {
  console.log("Blog Post: " + doc.title)
})
Blog Post: Post One
Blog Post: Post Two
Blog Post: Post Three
Blog Post: Post Four
Find One Row
db.posts.findOne({ category: 'News' })
{
  _id: ObjectId("6131e81db10976d1fa495549"),
  title: 'Post One',
  body: 'Body of post one',
  category: 'News',
  tags: [ 'news', 'events' ],
  user: { name: 'John Doe', status: 'author' },
  date: 'Fri Sep 03 2021 11:17:17 GMT+0200 (Central European Summer Time)'
}
Find Specific Fields
db.posts.find({ title: 'Post One' }, {
  title: 1,
  author: 1
})
[ { _id: ObjectId("6131e81db10976d1fa495549"), title: 'Post One' } ]
Update Row
db.posts.replaceOne(
{ title: 'Post Two' },
{ 
    title: 'Post Two',
    body: 'New body for post 2',
    date: Date()
},
{ upsert: true })
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
Update Fields
db.posts.updateOne(
{ title: 'Post Two' },
{ 
  $set: {
    title: 'Post Two',
    body: 'New body for post 2',
    likes: 1,
    date: Date()
  }
},
{ upsert: true })
Increment Field ($inc)
db.posts.updateOne({ title: 'Post Two' },
{
  $inc: {
    likes: 5
  }
})
Increment Field ($inc)
db.posts.updateOne({ title: 'Post Two' },
{
  $rename: {
    likes: 'views'
  }
})
Delete Row
db.posts.deleteOne({ title: 'Post Two' })
Increment Field ($inc)
db.posts.updateOne({ title: 'Post One' },
{
  $set: {
    comments: [
      {
        body: 'Comment One',
        user: 'Mary Williams',
        date: Date()
      },
      {
        body: 'Comment Two',
        user: 'Harry White',
        date: Date()
      }
    ]
  }
})
Find By Element in Array ($elemMatch)
db.posts.find({
  comments: {
     $elemMatch: {
       user: 'Mary Williams'
       }
    }
  }
)
Text Search
Only works when there is a text index created for the field.

db.posts.find({
  $text: {
    $search: "\"Post O\""
    }
})
Add Index
db.posts.createIndex({ title: 'text' })
Greater & Less Than
db.posts.find({ views: { $gt: 2 } })
db.posts.find({ views: { $gte: 7 } })
db.posts.find({ views: { $lt: 7 } })
db.posts.find({ views: { $lte: 7 } })
Aggregation Pipelines
use sample_mflix;
db.movies.aggregate([
  // First Stage
  { $project: { _id: 0, genres: 1, imdb: 1, title: 1 } },
  // Second Stage
  { $unwind: "$genres" },
  // Third Stage
  { $group:
    { _id: "$genres",
      averageGenreRating: { $avg: "$imdb.rating" }
    }
  },
   // Fourth Stage
   { $sort: { averageGenreRating: -1 } }
] )

References
docs.mongodb.com - mongosh Usage
github.com/bradtraversy - MongoDB Cheat Sheet (outdated commands, but good overview)

Start / NoSQL Lab / Lectures / Lecture 3 / Introduction / Solutions
PyCharm: MySQL - Chinook Queries Solution
DBs / MySQLTools / PyCharmExerciseHomeworkSolution
Page 102
File: /nosql_lab/lecture_2/homework/pycharm/100-chinook-solution.py (Lecture 2 - Exercises)

#!/usr/bin/env python3
# © 2026 by Revolytics AG, Oliver Staubli
# Data from: https://github.com/lerocha/chinook-database/blob/master/ChinookDatabase/DataSources/Chinook_MySql.sql

import json
import pymysql


class App:

    DB_CONFIG_FILE = '../../config/db_settings_local.json'
    DB_NAME = 'chinook'

    def __init__(self):
        self.connection = None
        self.cursor = None

    def run(self):

        self.init_cursor()
        self.exercise_01()
        self.exercise_02()
        self.exercise_03()

    def init_cursor(self):
        print(f'- init_cursor')
        config = json.load(open(self.DB_CONFIG_FILE, 'r'))
        config['database'] = self.DB_NAME
        self.connection = pymysql.connect(
            host=config['host'],
            port=config['port'],
            database=config['database'],
            user=config['user'],
            password=config['password']
        )
        self.cursor = self.connection.cursor(pymysql.cursors.DictCursor)


    def exercise_01(self):
        """
        Find top 5 countries by number of customers.
        Solution output should be:
        {'Country': 'USA', 'cnt': 13}
        {'Country': 'Canada', 'cnt': 8}
        {'Country': 'Brazil', 'cnt': 5}
        {'Country': 'France', 'cnt': 5}
        {'Country': 'Germany', 'cnt': 4}
        """
        print(f'- exercise_01', "~" * 50)

        sql = """
            SELECT Country, COUNT(Country) AS cnt
            FROM Customer
            GROUP BY Country
            ORDER BY COUNT(Country) DESC
            LIMIT 5;
        """

        self.cursor.execute(sql)
        for row in self.cursor:
            print(row)

    def exercise_02(self):
        """
        Find highest invoice.
        Solution output should be:
        {'InvoiceId': 404, 'MaxInvoice': Decimal('25.86')}
        """
        print(f'- exercise_02', "~" * 50)

        sql = """
                SELECT
                InvoiceId,
                SUM(UnitPrice * Quantity) AS MaxInvoice
                FROM InvoiceLine
                GROUP BY  InvoiceId
                ORDER BY MaxInvoice DESC
                LIMIT 1;
        """
        self.cursor.execute(sql)
        for row in self.cursor:
            print(row)


    def exercise_03(self):
        """
        Find how many tracks were purchased per media type.
        Solution output should be:
        {'Media_Type': 'MPEG audio file', 'Times_Purchased': Decimal('1976')}
        {'Media_Type': 'Protected AAC audio file', 'Times_Purchased': Decimal('146')}
        {'Media_Type': 'Protected MPEG-4 video file', 'Times_Purchased': Decimal('111')}
        {'Media_Type': 'Purchased AAC audio file', 'Times_Purchased': Decimal('4')}
        {'Media_Type': 'AAC audio file', 'Times_Purchased': Decimal('3')}
        """
        print(f'- exercise_03', "~" * 50)

        sql = """
            SELECT
                m.Name AS "Media_Type", 
                SUM(il.Quantity) AS "Times_Purchased"
            FROM Track t
                JOIN MediaType    m ON m.MediaTypeId = t.MediaTypeId
                JOIN InvoiceLine il ON il.TrackId = t.Trackid
            GROUP BY m.Name
            ORDER BY "Times_Purchased" DESC
            LIMIT 10;
        """
        self.cursor.execute(sql)
        for row in self.cursor:
            print(row)





if __name__ == '__main__':
    app = App()
    app.run()

Start / NoSQL Lab / Lectures / Lecture 3 / Introduction / Solutions
PlantUML: Chinook UML Modeling Solution
Tools / PlantUMLData ModelingExerciseHomeworkSolution
Page 103
Open DBeaver and connect to the Chinook -> MySQL import from File: /nosql_lab/lecture_2/data/chinook.sql (Lecture 2 - Exercises)
Use DBeaver to inspect the generated ER Diagram (Tables / View Tables / ER Diagram) of the Chinook database.
Extend the diagram below by 3 more entities following the structure of Chinook database in DBeaver.
Use this partial ER Diagram on plantuml.com as starting point.
Solution

Start / NoSQL Lab / Lectures / Lecture 3
JSON
JSON
Page 106

JavaScript Object Notation (JSON) is an open-standard file format that uses human-readable text to transmit data objects consisting of attribute–value pairs and array data types (or any other serializable value). It is a very common data format.

JSON is a language-independent data format. It was derived from JavaScript, but many modern programming languages include code to generate and parse JSON-format data.

JSON Example
{
    "title" : "Love Actually",
    "year" : 2003,
    "released": true,
    "actors" : [ 
        "Bill Nighy", 
        "Gregor Fisher", 
        "Rory MacGregor", 
        "Colin Firth"
    ],
    "imdb" : {
        "id" : "tt0314331",
        "rating" : 7.7,
        "votes" : 306036
    }, 
    "plot": null
}
Definition
On json.org you can find definitions like these ones for "object" and "value":

JSON
JSON
Data types
String
Number
Object (JSON)
Array
Boolean
Null
Canonical Form
RFC 7159 defines JSON as “a text format for the serialization of structured data”, but allows many distinct serializations to describe the same data. Such human-friendly flexibility can hinder machine treatment of JSON text, particularly when it is used as input for cryptographic hash functions that are expected to yield identical results for logically equivalent input (as is the case in computation of digital signatures). This specification defines a unique canonical form for every JSON value, the result being safe for comparison (in that logically equivalent structured data are guaranteed to have the same canonical form). See JSON Canonical Form for details.

Canonical JSON Example
Keys are sorted. No whitespaces except in value strings.

{"actors":["Bill Nighy","Gregor Fisher","Rory MacGregor","Colin Firth"],"imdb":{"id":"tt0314331","rating":7.7,"votes":306036},"plot":null,"released":true,"title":"Love Actually","year":2003}
Useful Links
Chrome Extension "JSON Formatter" to visualize raw JSON as nested tree.
HINT: This extension is handy for quickly reviewing JSON files for your personal project.
Enable "Allow access to file URLs" in the extension in order to visualize local files as well.
Online JSON Formatter & Validator

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB
Introduction
DBs / MongoDB
Page 108
MongoDB
MongoDB is a cross-platform document-oriented database program. Classified as a NoSQL database program, MongoDB uses JSON-like documents with schema. MongoDB is developed by MongoDB Inc. and licensed under the Server Side Public License (SSPL).

What is MongoDB
MongoDB stores data in flexible, JSON-like documents, meaning fields can vary from document to document and data structure can be changed over time.
The document model maps to the objects in your application code, making data easy to work with.
Ad hoc queries, indexing, and real time aggregation provide powerful ways to access and analyze your data.
MongoDB is a distributed database at its core, so high availability, horizontal scaling, and geographic distribution are built in and easy to use.
MongoDB is free to use.
DB-Engines Ranking
MongoDB usually is ranked as the most popular document based database:

SQL
Check current ranking

MongoDB Key Features
Availability: Deliver globally resilient apps through sophisticated replication and self-healing recovery
Scalability: Grow horizontally through native sharding
Workload Isolation: Run operational and analytical workloads in the same cluster
Other Features worth mentioning
Document based NoSQL Database
Stored in BSON format
Presented in JSON
Autogenerated _id indexed unique field (aka primary key)
No predefined schema
GridFS for binary data
Built in Replication (high availability)
Great for Aggregation
Hadoop & Spark integration (Big Data)
Open Source -> Server Side Public License
Who is using MongoDB
Introduction
See full list of Who Uses MongoDB.

Top Real-World Use Cases
Content Management Systems (CMS): Stores diverse or nested content types like blogs, articles, and user comments where metadata frequently changes.
E-Commerce & Product Catalogs: Manages dynamic product attributes, user profiles, shopping carts, and real-time inventory catalogues.
Artificial Intelligence & Vector Search: Powers semantic search, Retrieval-Augmented Generation (RAG), and stateful AI agent memory by storing high-dimensional vector embeddings next to operational data.
Single View & Customer Analytics: Aggregates multi-source data streams to provide a unified, real-time single view of customer activity.
Internet of Things (IoT) & Time-Series Data: Ingests and analyzes high-volume temporal metrics from physical devices using specialized time-series collections.
Gaming Backends: Tracks fast-moving player profiles, global leaderboards, and concurrent session states at scale.
When to Choose MongoDB
Your data is semi-structured or lacks a rigid, predefined schema.
You need horizontal scalability (sharding) to handle high-velocity write/read traffic.
Application requirements evolve quickly, requiring rapid, iterative database design.


RDBMS	MongoDB
Service/Server	Service/Server
Database	Database
Table	Collection
Row	Document
Column	Field
Value	Value
Primary Key	_id
Table Join	Embedded Document or $lookup

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB
Setup Your MongoDB Cluster
DBs / MongoDBTools / MongoDB AtlasInstallationExercise
Page 111
You will set up and connect to a free MonogDB cluster in the cloud on MongoDB Atlas.

1. Create an Atlas Account
Use this Registration-Form, sign-up and verify your email-address.

Completed
2. Deploy a Free Cluster
Continue from the verification Email link to set up your free cluster. Or start with login in in to atlas.mongodb.com and click "Databases" and then the green "Create" button.

HINT: In step 4 select "AWS" + "Frankfurt" for lowest latency (or whatever is closest to your location).

HINT Don't use "?" in your password!

Completed
3. Add Your Connection IP Address to IP Access List
Follow these steps below to whitelist your IP or any IP ("0.0.0.0/0"):

Setup Your MongoDB Cluster
Setup Your MongoDB Cluster
HINT: It's not best practice for security reasons, but you can whitelist all IP Addresses for cluster by adding "0.0.0.0/0" as IP Address.

Completed
4. Insert Sampledata in Your Cluster
Follow these to insert the MongoDB sample datasets in Your Cluster:

Setup Your MongoDB Cluster
Setup Your MongoDB Cluster
HINT Select "Select All" datasets and continue.

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB / MongoDB GUIs
MongDB Compass
DBs / MongoDBTools / MongoDB Compass
Page 113
MongDB Compass is the official GUI for MongoDB to:

Visually explore your data.
Run ad hoc queries in seconds
Interact with your data with full CRUD functionality.
View and optimize your query performance.
MongDB Compass (GUI)
Mongo DB Compass Demo
Connect to a MongoDB cluster
Compass Home
Collections and Views
Natural Language (ChatGPT)
Aggregation Pipeline Builder
Import and Export Data

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB / MongoDB GUIs / MongDB Compass
Connect to a MongoDB cluster
DBs / MongoDBTools / MongoDB Compass
Page 114
For this demo we will use my MongoDB cluster with a user having read only rights.
Open MongoDB Compass with the following connection string:

mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
MongDB Compass (GUI)

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB / MongoDB GUIs / MongDB Compass
Compass Home
DBs / MongoDBTools / MongoDB Compass
Page 115
Once connected to a MongoDB instance, you can reach the Compass Home screen by clicking the Home icon in the upper left corner.

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB / MongoDB GUIs / MongDB Compass
Natural Language (ChatGPT)
DBs / MongoDBTools / MongoDB Compass
Page 117
Find Documents
Find documents using natural language.

Collections and Views
Collections and Views
Aggregation Pipeline Builder
Build your aggregation pipeline using natural language.

Collections and Views
Collections and Views

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB / MongoDB GUIs / MongDB Compass
Import and Export Data
DBs / MongoDBTools / MongoDB Compass
Page 119
Create Database and Collection
Create a database and an empty collection first to import your data:

Collections and Views

Import Data
You can manually import JSON or CSV files:
Files: /nosql_lab/lecture_3/data/import_test_collection.json/csv (Lecture 3 - Exercises)

IMPORTANT: The CSV file is expected to have a header line with field names for comma separated position of the document lines that follow. When importing a JSON file, the source file should have a separate JSON document on each line and it should not be in the JSON array format with beginning and ending square brackets.

Collections and Views
Collections and Views
Collections and Views

Export
Same options for exporting JSON or CSV files:

Collections and Views
Collections and Views
Collections and Views

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB / MongoDB CLIs
Overview
DBs / MongoDBTools / MongoDB ShellTools / MongoDB Database Tools
Page 122
MongoDB Shell
The MongoDB Shell, mongosh, is a fully functional JavaScript and Node.js environment for interacting with MongoDB deployments.

mongosh	Test queries and operations directly with your database.
HINT: Command "mongosh" was introduced in MongoDB Version 5.0, before the command "mongo" was used.

MongoDB Database Tools
The MongoDB Database Tools are a collection of command-line utilities for working with a MongoDB deployment. The Database Tools include the following binaries:

Binary Import / Export
mongodump	Creates a binary export of the contents of a mongod database.
mongorestore	Restores data from a mongodump database dump into a mongod or mongos
bsondump	Converts BSON dump files into JSON.
Data Import / Export
mongoimport	Imports content from an Extended JSON, CSV, or TSV export file.
mongoexport	Produces a JSON or CSV export of data stored in a mongod instance.
Diagnostic Tools
mongostat	Provides a quick overview of the status of a currently running mongod or mongos instance.
mongotop	Provides an overview of the time a mongod instance spends reading and writing data.
GridFS Tools
mongofiles	Supports manipulating files stored in your MongoDB instance in GridFS objects.

Start / NoSQL Lab / Lectures / Lecture 3 / MongoDB
MongoDB Setup & Connections
DBs / MongoDBTools / MongoDB CompassTools / MongoDB ShellTools / MongoDB AtlasInstallation
Page 124
There are many options to setup MongoDB server...

1. On your Laptop/PC locally
You could install the free MongoDB Community Server locally on your Laptop/PC.
Completed
To connect to your local MongoDB Server you would use "localhost" in the connection string:
MongoDB Shell:
mongosh --host localhost:27017
show dbs
exit
MongoDB Compass:
MongoDB Backends
Python pymongo:
client = pymongo.MongoClient("localhost:27017")
client.list_database_names()
2. On a server via network
You could install the free MongoDB Community Server on a server .
To connect to a MongoDB Server you would use the servers IP (e.g. "172.19.42.3") in the connection string:
MongoDB Shell:
mongosh --host 172.19.42.3:27017
show dbs
exit
MongoDB Compass:
MongoDB Backends
Python pymongo:
client = pymongo.MongoClient("172.19.42.3:27017")
client.list_database_names()
3. On Infrastructure as a Service (IaaS)
You would use virtual machines running in the cloud that are not managed by the provider - also known as "the hard way".

AWS:
https://aws.amazon.com/ec2/
https://aws.amazon.com/ec2/instance-types/
https://aws.amazon.com/ec2/instance-types/t2/
https://aws.amazon.com/free/
Azure:
https://azure.microsoft.com/de-de/pricing/details/virtual-machines/series/
https://azure.microsoft.com/de-de/pricing/details/virtual-machines/linux/#a-series
4. On Software as a Service (SaaS)
Just rent the service, not the servers - also known as "easy peasy lemon squeezy".

MongoDB as a Service:
https://www.mongodb.com/cloud/atlas
https://aws.amazon.com/documentdb
https://azure.microsoft.com/de-de/services/cosmos-db/
To connect to your MongoDB cluster in the cloud you would use "mongodb+srv://..." in the connection string:
MongoDB Shell:
mongosh mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net
show dbs
exit
MongoDB Compass:
MongoDB Backends
Python pymongo:
client = pymongo.MongoClient("mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net")
client.list_database_names()

We will use MongoDB Atlas (SaaS) in this course. You will Setup Your MongoDB Cluster as homework.

4.1 MongoDB Atlas
Known Connection Issues
If you try to connect from Python via pymongo to an Atlas cluster and suddenly get the following SSL error message:

pymongo.errors.ServerSelectionTimeoutError: cluster0-shard-00-00.9tnmh.mongodb.net:27017: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: certificate has expired
See error explanation.

On Windows
Download https://letsencrypt.org/certs/lets-encrypt-r3.pem
rename file .pem to .cer
double click and install
On MacOs
Follow this guide.

Alternatively
An alternative "quick-fix" is to use Python package "certifi" when creating the MongoClient with the additional "tlsCAFile" parameter like this:

import pymongo
import certifi

client = pymongo.MongoClient("mongodb+srv://dbUser:dbPassword@cluster0.ypbr6t4.mongodb.net", tlsCAFile=certifi.where())

Start / NoSQL Lab / Lectures / Lecture 2 / Data Modeling
Introduction
Data Modeling
Page 57
What is Data Modelling?
Data modeling (or data modelling) is the process of creating a data model for the data to be stored in a database. This data model is a conceptual representation of Data objects, the associations between different data objects, and the rules. Data modeling helps in the visual representation of data and enforces business rules, regulatory compliances, and government policies on the data. Data Models ensure consistency in naming conventions, default values, semantics, security while ensuring quality of the data.

Introduction
References
guru99.com/data-modelling-conceptual-logical.htm

Start / NoSQL Lab / Lectures / Lecture 2 / Data Modeling
Entity Relationship Diagram (ERD)
Data Modeling
Page 58
An entity relationship diagram (ERD) shows the relationships of entity sets stored in a database. An entity in this context is an object, a component of data. An entity set is a collection of similar entities. These entities can have attributes that define its properties.

An example of a entity relationship diagram:

Entity Relationship Diagram (ERD)
By defining the entities, their attributes, and showing the relationships between them, an ER diagram illustrates the logical structure of databases.

ER diagrams are used to sketch out the design of a database.

Why you should care
ERD as Input: If you are tasked to analyse data stored in a relational database, an ER diagram helps you getting an overview of what data you are dealing with and how it is structured.
ERD as Output: If you are designing a new (relational) database schema to persist your data, a ER diagram helps you to document and communicate your data structure.
Common Entity Relationship Diagram Symbols
An ER diagram is a means of visualizing how the information a system produces is related. There are five main components of an ERD:

Entities, which are represented by rectangles. An entity is an object or concept about which you want to store information.Entity - ERD Symbol
Actions, which are represented by diamond shapes, show how two entities share information in the database.Action - ERD SymbolIn some cases, entities can be self-linked. For example, employees can supervise other employees.
Self-linked Action - ERD Symbol
Attributes, which are represented by ovals. A key attribute is the unique, distinguishing characteristic of the entity. For example, an employee's social security number might be the employee's key attribute.
Attribute - ERD Symbol
Connecting lines, solid lines that connect attributes to show the relationships of entities in the diagram.
Cardinality specifies how many instances of an entity relate to one instance of another entity. Ordinality is also closely linked to cardinality. While cardinality specifies the occurrences of a relationship, ordinality describes the relationship as either mandatory or optional. In other words, cardinality specifies the maximum number of relationships and ordinality specifies the absolute minimum number of relationships.
Cardinality - ERD Symbol
Cardinality Notations
There are many notation styles that express cardinality.

Information Engineering Style
Information Engineering Style Cardinality - ERD
Chen Style
Chen Style Cardinality - ERD
Bachman Style
Bachman Style Cardinality - ERD
Martin Style
Martin Style Cardinality - ERD
Relationships Examples
We start with the one-to-one relations event though these aren't the most common when modeling an information solution.

examples a-c
A more common set of relations are the one-to-many relations.

examples d-g
Finally, entities may be related using many-to-many relations.

examples h-j
Many-to-Many relations are useful primerely in early stage, analysis, modeling. A many-to-many relation can always be transformed into two one-to-many relations by modeling the relation itself as yet another entity.

example k
Physical data model
Physical ERD represents the actual design blueprint of a relational database. A physical data model elaborates on the logical data model by assigning each column with type, length, nullable, etc. Since a physical ERD represents how data should be structured and related in a specific DBMS it is important to consider the convention and restriction of the actual database system in which the database will be created. Make sure the column types are supported by the DBMS and reserved words are not used in naming entities and columns.

Physical data model


References
smartdraw.com/entity-relationship-diagram
risetobloome.com/relationships
visual-paradigm.com/guide/data-modeling

Start / NoSQL Lab / Lectures / Lecture 2 / Data Modeling
Class Diagram (in UML)
Data Modeling
Page 59
The UML Class diagram is a graphical notation used to construct and visualize object oriented systems. A class diagram in the Unified Modeling Language (UML) is a type of static structure diagram that describes the structure of a system by showing the system's:

classes,
their attributes,
operations (or methods),
and the relationships among objects.
An example of an UML class diagram:

Class Diagram (in UML)
Difference Between Class Diagram (UML) and Entity Relationship Diagram (ERD)
ER diagrams represent the abstract representation of the data model, while class diagrams represent the static structure and behavior of the proposed system.

Why you should care
Class Diagrams are very similar to Entity Relationship Diagrams, but better suited for visualizing Document Databases structures like with MongoDB.

Full Specification
You don't need to read the Full Specification (PDF with 796 pages) to understand the main concepts.

Class Notation
A class represent a concept which encapsulates state (attributes) and behavior (operations). Each attribute has a type. Each operation has a signature. The class name is the only mandatory information.

Class Notation
Class Name:
The name of the class appears in the first partition.
Class Attributes:
Attributes are shown in the second partition.
The attribute type is shown after the colon.
Attributes map onto member variables (data members) in code.
Class Operations (Methods):
Operations are shown in the third partition. They are services the class provides.
The return type of a method is shown after the colon at the end of the method signature.
The return type of method parameters are shown after the colon following the parameter name. Operations map onto class methods in code.
Class Diagram (in UML)
Relationships between classes
A class may be involved in one or more relationships with other classes. A relationship can be one of the following types:

Class Diagram (in UML)
Association
Associations are relationships between classes in a UML Class Diagram. They are represented by a solid line between classes. Associations are typically named using a verb or verb phrase which reflects the real world problem domain.

Cardinality
Cardinality is expressed in terms of:

one to one
one to many
many to many
Class Diagram (in UML)
Aggregation
A special type of association: It represents a "part of" relationship, e.g. car model engine ENG01 is part of a car model CM01, as the engine, ENG01, may be also part of a different car model.

Class Diagram (in UML)
Composition
A special type of aggregation where parts are destroyed when the whole is destroyed, e.g. a university and its departments.

Class Diagram (in UML)


References
visual-paradigm.com/guide/uml-unified-modeling-language
wikipedia.org/wiki/Class_diagram

Start / NoSQL Lab / Lectures / Lecture 2 / Data Modeling
Data Modeling Examples (Chinook)
Tools / PlantUMLData Modeling
Page 60
Chinook Database
File: /nosql_lab/lecture_2/data/chinook.sql (Lecture 2 - Exercises)

The Chinook data model represents a digital media store, including tables for artists, albums, media tracks, invoices and customers.

Media-related data was created using real data from an Apple iTunes library.
Customer and employee information was created using fictitious names and addresses that can be located on Google maps, and other well formatted data (phone, fax, email, etc.)
Sales information was auto generated using random data for a four year period.
The Chinook sample database includes:

11 tables
A variety of indexes, primary and foreign key constraints
Over 15,000 rows of data
For details, here's the entity relationship diagram of the Chinook data model.

Chinook Examples
PlantUML
PlantUML is a webservices/component that allows to quickly write:

Class diagram
Entity Relationship diagram (IE/ER)
Component diagram
Sequence diagram
...
Chinook ER Diagram in crow's foot notation (partially)
Edit diagram on plantuml.com

Chinook UML Class Diagram (partially)
Edit diagram on plantuml.com

Formatting examples

skinparam default {
  FontName           Mono
  FontSize           16
  FontStyle          bold
  FontColor          #022
}

skinparam title {
  FontName           Arial
  FontSize           30
  FontStyle          bold
  FontColor          #000
}

skinparam class {
  FontName           Arial
  FontSize           24
  FontStyle          bold
  FontColor          #060
  BackgroundColor    #efe
  BorderColor        #000
  ArrowColor         #000
}
Other Webservices Data Modeling Tools
https://yuml.me
https://www.diagrams.net
https://www.yworks.com/yed-live
https://www.websequencediagrams.com
https://creately.com
https://start.jhipster.tech/jdl-studio
https://mermaid-js.github.io/mermaid-live-editor
https://mermaid-js.github.io/mermaid
Other Software for Data Modeling
wikipedia.org - List of Unified Modeling Language tools


References
yugabyte.com - Chinook sample database
sqlitetutorial.net - sqlite-sample-database

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1
About The MongoDB Manual
Page 221
The MongoDB Manual
The MongoDB Manual (Version 8.0) introduces key concepts in MongoDB, presents the query language, and provides operational and administrative considerations and procedures as well as a comprehensive reference section.

We will go through the "most important" sections together, with some "deep-dives" where deemed appropriate.


References
docs.mongodb.com - MongoDB Manual
Previous Page

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / Introduction to MongoDB
Databases and Collections
DBs / MongoDB
Page 223
Revisit MongoDB vs RDBMS.
MongoDB stores BSON documents in collections; the collections in databases.
If a database does not exist, MongoDB creates the database when you first store data for that database.
Ditto for collections.
use myDB
// Does not create the database "myDB".

db.myCollection.insertOne( { x: 1 } )
// Creates db "myDB" and collection "myCollection" with one document.
There are restricitions on DB names depending on the oprating system.
Database names are "case insensitive", meaning database names cannot differ only by the case of the characters.
Collection can be created explicitly, as an empty collection, but with optional validation rules. with
db.createCollection('myCollection2')

References
docs.mongodb.com - Databases and Collections

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / Introduction to MongoDB
Documents
DBs / MongoDBData Modeling
Page 224
MongoDB stores data records as BSON documents. BSON is a binary representation of JSON documents, though it contains more data types than JSON.
{
   field1: value1,
   field2: value2,
   field3: value3,
   ...
   fieldN: valueN
}
Field Names:
Field names are strings
The field name _id is reserved for use as a primary key; its value must be unique in the collection, is immutable, and may be of any type other than an array.
Top-level field names cannot start with the dollar sign ($) character.
The _id Field:
By default, MongoDB creates a unique index on the _id field during the creation of a collection.
The _id field is always the first field in the documents. If the server receives a document that does not have the _id field first, then the server will move the field to the beginning.
Dot Notation:
MongoDB uses the dot notation to access the elements of an array and to access the fields of an embedded document.

References
docs.mongodb.com - Documents

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / Introduction to MongoDB
BSON
DBs / MongoDBJSON
Page 225
BSON is a binary serialization format used to store documents and make remote procedure calls in MongoDB.

Origins
BSON is a computer data interchange format. The name "BSON" is based on the term JSON and stands for "Binary JSON".

It is a binary form for representing simple or complex data structures including associative arrays (also known as name-value pairs), integer indexed arrays, and a suite of fundamental scalar types. BSON originated in 2009 at MongoDB Inc. Several scalar data types are of specific interest to MongoDB and the format is used both as a data storage and network transfer format for the MongoDB database, but it can be used independently outside of MongoDB.

Data Types
An important differentiator to JSON is that BSON contains types not present in JSON (e.g. datetime and byte array) and offers type-strict handling for several numeric types instead of a universal "number" type.

JSON Types	BSON Types (short list)
String	String
Boolean	Boolean
Object	Object
Array	Array
Null	Null
Number	
Double
32 bit Integer
64 bit Integer
Decimal128
Date
Timestamp (internal only)
ObjectId
Document Size Limit
The maximum BSON document size is 16 megabytes.


References
wikipedia.org - BSON
docs.mongodb.com - BSON Types
bsonspec.org

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / Introduction to MongoDB
Extended JSON
DBs / MongoDBJSON
Page 226
JSON can only directly represent a subset of the types supported by BSON. To preserve type information, MongoDB adds the following extensions to the JSON format.

Relaxed Mode
A string format that emphasizes readability and interoperability at the expense of type preservation. That is, conversion from relaxed format to BSON can lose type information.
{
  "my_date": {
    "$date": "2020-10-28T12:44:51.122Z"
  }
}
Canonical Mode
A string format that emphasizes type preservation at the expense of readability and interoperability. That is, conversion from canonical to BSON will generally preserve type information except in certain specific cases.
{"my_date":{"$date":{"$numberLong":"1603889091122"}}}

References
docs.mongodb.com - MongoDB Extended JSON (v2)

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / Introduction to MongoDB
JSON Export Tests
DBs / MongoDBTools / MongoDB Database ToolsExerciseJSON
Page 227
We try to export data inserted with inline BSON types and without. NumberInt, Long, ISODate, Timestamp, new Date, ObjectId are built in JavaScript functions available in the MongoDB Shell.

Insert Test Data
mongosh  myTestDataTypesDB
db.myDataTypesOkay.insertOne(
{
    "hello_string" : "Hello World",
    "hello_boolean" : true,
    "hello_null" : null,
    "hello_array" : [
        "jane",
        24.42
    ],
    "hello_object" : {
        "age" : 42,
        "name" : "john"
    },
    "hello_double_a" : 42,
    "hello_double_b" : 42.0,
    "hello_double_c" : Double(42.0),
    "hello_int32_a" : NumberInt(1234),
    "hello_int32_max" : NumberInt(2147483647),
    "hello_int64" : Long(42),
    "hello_date_a" : ISODate("2020-01-30T12:00:00.000Z"),
    "hello_date_b" : ISODate("2020-01-30T12:00:00.000+01:00"),
    "hello_date_c" : new Date(),
    "hello_timestamp" : Timestamp(1580428800, 1),
    "hello_objectid":  ObjectId("5d70eeecbc0a86f79cc65c34")
}
)
Export In Canonical Mode
BSON types are represented via special key names (prepended with $) and results in valid JSON.

mongoexport --db myTestDataTypesDB --collection myDataTypesOkay --jsonFormat canonical --pretty --out=myDataTypesOkay.json
{
	"_id": {
		"$oid": "6140bf9b17053cf8010ded2d"
	},
	"hello_string": "Hello World",
	"hello_boolean": true,
	"hello_null": null,
	"hello_array": [
		"jane",
		{
			"$numberDouble": "24.42"
		}
	],
	"hello_object": {
		"age": {
			"$numberInt": "42"
		},
		"name": "john"
	},
	"hello_double_a": {
		"$numberInt": "42"
	},
	"hello_double_b": {
		"$numberInt": "42"
	},
	"hello_double_c": {
		"$numberDouble": "42.0"
	},
	"hello_int32_a": {
		"$numberInt": "1234"
	},
	"hello_int32_max": {
		"$numberInt": "2147483647"
	},
	"hello_int64": {
		"$numberLong": "42"
	},
	"hello_date_a": {
		"$date": {
			"$numberLong": "1580385600000"
		}
	},
	"hello_date_b": {
		"$date": {
			"$numberLong": "1580382000000"
		}
	},
	"hello_date_c": {
		"$date": {
			"$numberLong": "1631633307595"
		}
	},
	"hello_timestamp": {
		"$timestamp": {
			"t": 1580428800,
			"i": 1
		}
	},
	"hello_objectid": {
		"$oid": "5d70eeecbc0a86f79cc65c34"
	}
}
Export In Relaxed Mode
mongoexport --db myTestDataTypesDB --collection myDataTypesOkay --jsonFormat relaxed --pretty --out=myDataTypesFail.json
{
	"_id": {
		"$oid": "6140bf9b17053cf8010ded2d"
	},
	"hello_string": "Hello World",
	"hello_boolean": true,
	"hello_null": null,
	"hello_array": [
		"jane",
		24.42
	],
	"hello_object": {
		"age": 42,
		"name": "john"
	},
	"hello_double_a": 42,
	"hello_double_b": 42,
	"hello_double_c": 42.0,
	"hello_int32_a": 1234,
	"hello_int32_max": 2147483647,
	"hello_int64": 42,
	"hello_date_a": {
		"$date": "2020-01-30T12:00:00Z"
	},
	"hello_date_b": {
		"$date": "2020-01-30T11:00:00Z"
	},
	"hello_date_c": {
		"$date": "2021-09-14T15:28:27.595Z"
	},
	"hello_timestamp": {
		"$timestamp": {
			"t": 1580428800,
			"i": 1
		}
	},
	"hello_objectid": {
		"$oid": "5d70eeecbc0a86f79cc65c34"
	}
}
Export In Canonical Mode & Import
mongoexport --db myTestDataTypesDB --collection myDataTypesOkay --jsonFormat canonical --pretty --out=myDataTypesOkay.json | ^
mongoimport --db myTestDataTypesDB --collection myDataTypesOkay2 --drop --file=myDataTypesOkay.json
Documents field types of collection myDataTypesOkay and myDataTypesOkay2 are preserved.

Export In Relaxed Mode & Import
mongoexport --db myTestDataTypesDB --collection myDataTypesOkay --jsonFormat relaxed --pretty --out=myDataTypesFail.json | ^
mongoimport --db myTestDataTypesDB --collection myDataTypesFail --drop --file=myDataTypesFail.json
Documents field types of collection myDataTypesOkay and myDataTypesFail differ! Type of field "hello_int64" is now "int32".

Takeaway
Hint: Export in canonical mode to be "type safe".
Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1
CRUD Operations
DBs / MongoDB
Page 229
CRUD = create, read, update, delete.

Overview
Create Operations
Create or insert operations add new documents to a collection.
If the collection does not currently exist, insert operations will create the collection.
db.collection.insert
db.collection.insertOne
db.collection.insertMany
Read Operations
Read operations retrieves documents from a collection; i.e. queries a collection for documents.
db.collection.find
db.collection.findOne
Update Operations
Update operations modify existing documents in a collection
db.collection.updateOne
db.collection.updateMany
db.collection.replaceOne
Delete Operations
Delete operations remove documents from a collection.
db.collection.deleteOne
db.collection.deleteMany
Exercises
Insert
Query Documents
Query on Embedded/Nested Documents
Query an Array
Query an Array of Embedded Documents
Project Fields to Return from Query
Query for Null or Missing Fields
Update Documents
SQL to MongoDB Mapping Chart
Text Search
Geospatial Queries
Vector Search Queries
Usecases from Students Personal Projects

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Insert
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 230
db.collection.insert inserts a single document or multiple documents into a collection.

In MongoDB, each document stored in a collection requires a unique _id field that acts as a primary key. If an inserted document omits the _id field, the MongoDB driver automatically generates an ObjectId for the _id field.

Insert a Single Document
mongosh: db.collection.insertOne()

File: /nosql_lab/lecture_6/exercises/mongosh/01-insertOne-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.insertOne(
   { item: "canvas", qty: 100,
     tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" }
   }
);

db.inventory.find( { item: "canvas" } );
pymongo: db.collection.insert_one()

File: /nosql_lab/lecture_6/exercises/pycharm/01-insertOne-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

id = db.inventory.insert_one(
  {"item": "canvas",
   "qty": 100,
   "tags": ["cotton"],
   "size": {"h": 28, "w": 35.5, "uom": "cm"}}
)

cursor = db.inventory.find({"item": "canvas"})
pprint(list(cursor)[:3])
Insert Multiple Documents
mongosh: db.collection.insertMany()

File: /nosql_lab/lecture_6/exercises/mongosh/02-insertMany-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.insertMany([
   { item: "journal", qty: 25,
     tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85,
     tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25,
     tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])

db.inventory.find();
pymongo: db.collection.insert_many()

File: /nosql_lab/lecture_6/exercises/pycharm/02-insertMany-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.insert_many([
  {"item": "journal",
   "qty": 25,
   "tags": ["blank", "red"],
   "size": {"h": 14, "w": 21, "uom": "cm"}},
  {"item": "mat",
   "qty": 85,
   "tags": ["gray"],
   "size": {"h": 27.9, "w": 35.5, "uom": "cm"}},
  {"item": "mousepad",
   "qty": 25,
   "tags": ["gel", "blue"],
   "size": {"h": 19, "w": 22.85, "uom": "cm"}}
])

cursor = db.inventory.find()
pprint(list(cursor)[:3])

References
docs.mongodb.com - Insert Documents

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Query Documents
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 231
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/10-query-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.deleteMany({});

db.inventory.insertMany([
  { item: "journal", qty: 25,
    size: { h: 14, w: 21, uom: "cm" }, status: "A" },
  { item: "notebook", qty: 50,
    size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
  { item: "paper", qty: 100,
    size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
  { item: "planner", qty: 75,
    size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
  { item: "postcard", qty: 45,
    size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);

db.inventory.find();
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/10-query-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([
  {"item": "journal",
   "qty": 25,
   "size": {"h": 14, "w": 21, "uom": "cm"},
   "status": "A"},
  {"item": "notebook",
   "qty": 50,
   "size": {"h": 8.5, "w": 11, "uom": "in"},
   "status": "A"},
  {"item": "paper",
   "qty": 100,
   "size": {"h": 8.5, "w": 11, "uom": "in"},
   "status": "D"},
  {"item": "planner",
   "qty": 75, "size": {"h": 22.85, "w": 30, "uom": "cm"},
   "status": "D"},
  {"item": "postcard",
   "qty": 45,
   "size": {"h": 10, "w": 15.25, "uom": "cm"},
   "status": "A"}
])
Queries
mongosh: db.collection.find()

File: /nosql_lab/lecture_6/exercises/mongosh/11-query-find-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

// SELECT * FROM inventory
db.inventory.find();

// SELECT * FROM inventory WHERE status = "D"
db.inventory.find( { status: "D" } )

// SELECT * FROM inventory WHERE status in ("A", "D")
db.inventory.find( { status: { $in: [ "A", "D" ] } } )

// SELECT * FROM inventory WHERE status = "A" AND qty < 30
db.inventory.find( { status: "A", qty: { $lt: 30 } } )

// SELECT * FROM inventory WHERE status = "A" OR qty < 30
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )

// SELECT * FROM inventory WHERE status = "A"
//                           AND ( qty < 30 OR item LIKE "p%")
db.inventory.find( {
  status: "A",
  $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
pymongo: db.collection.find()

File: /nosql_lab/lecture_6/exercises/pycharm/11-query-find-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy


# 1. SELECT * FROM inventory
print("1.", "~"*30)
for d in db.inventory.find(): pprint(d)
print()

# 2. SELECT * FROM inventory WHERE status = "D"
print("2.", "~"*30)
for d in db.inventory.find({"status": "D"}):
  pprint(d)
print()

# 3. SELECT * FROM inventory WHERE status in ("A", "D")
print("3.", "~"*30)
for d in db.inventory.find({"status": {"$in": ["A", "D"]}}):
  pprint(d)
print()

# 4. SELECT * FROM inventory WHERE status = "A" AND qty < 30
print("4.", "~"*30)
for d in db.inventory.find({"status": "A", "qty": {"$lt": 30}}):
  pprint(d)
print()

# 5. SELECT * FROM inventory WHERE status = "A" OR qty < 30
print("5.", "~"*30)
for d in db.inventory.find(
    {"$or": [{"status": "A"}, {"qty": {"$lt": 30}}]}):
  pprint(d)
print()

# 6. SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")
print("6.", "~"*30)
for d in db.inventory.find({
    "status": "A",
    "$or": [{"qty": {"$lt": 30}}, {"item": {"$regex": "^p"}}]}):
  pprint(d)
print()

References
docs.mongodb.com - Query Documents

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Query on Embedded/Nested Documents
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 232
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/12-query-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.deleteMany({});

db.inventory.insertMany( [
  { item: "journal", qty: 25
    , size: { h: 14, w: 21, uom: "cm" }, status: "A" },
  { item: "notebook", qty: 50
    , size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
  { item: "paper", qty: 100
    , size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
  { item: "planner", qty: 75
    , size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
  { item: "postcard", qty: 45
    , size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/12-query-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([
   {'item': "journal", 'qty': 25,
    'size': {'h': 14, 'w': 21, 'uom': "cm"}, 'status': "A"},
   {'item': "notebook", 'qty': 50,
    'size': {'h': 8.5, 'w': 11, 'uom': "in"}, 'status': "A"},
   {'item': "paper", 'qty': 100,
    'size': {'h': 8.5, 'w': 11, 'uom': "in"}, 'status': "D"},
   {'item': "planner", 'qty': 75,
    'size': {'h': 22.85, 'w': 30, 'uom': "cm"}, 'status': "D"},
   {'item': "postcard", 'qty': 45,
    'size': {'h': 10, 'w': 15.25, 'uom': "cm"}, 'status': "A"}
])
Queries
mongosh: Match an Embedded/Nested Document

File: /nosql_lab/lecture_6/exercises/mongosh/13-query-find-embedded-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } );
db.inventory.find( { size: { w: 21, h: 14, uom: "cm" } } );

db.inventory.find( { "size.uom": "in" } );
db.inventory.find( { "size.h": { $lt: 15 } } );
db.inventory.find( { "size.h": { $lt: 15 },
                     "size.uom": "in", status: "D" } );
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/13-query-find-embedded-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

print("1.", "~"*30)
# matching 3 attributes of size
for d in db.inventory.find({"size": {"h": 14, "w": 21, "uom": "cm"}}):
    pprint(d)
print()

print("2.", "~"*30)
# no match w/h switched -> AND operation
for d in db.inventory.find({"size": {"w": 21, "h": 14, "uom": "cm"}}):
    pprint(d)
print()

print("3.", "~"*30)
# match nested structures with dot operator
for d in db.inventory.find({"size.uom": "in"}):
    pprint(d)
print()

print("4.", "~"*30)
# compare nested structures with dot operator
for d in db.inventory.find({"size.h": {"$lt": 15}}):
    pprint(d)
print()

print("5.", "~"*30)
# match on various levels
for d in db.inventory.find({"size.h": {"$lt": 15}, "size.uom": "in", "status": "D"}):
    pprint(d)
print()

References
docs.mongodb.com - Query on Embedded/Nested Documents

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Query an Array
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 233
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/14-query-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.deleteMany({});

db.inventory.insertMany([
  { item: "journal", qty: 25
    , tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
  { item: "notebook", qty: 50
    , tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
  { item: "paper", qty: 100
    , tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
  { item: "planner", qty: 75
    , tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
  { item: "postcard", qty: 45
    , tags: ["blue"], dim_cm: [ 10, 15.25 ] }
]);
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/14-query-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([
  {"item": "journal",
   "qty": 25,
   "tags": ["blank", "red"],
   "dim_cm": [14, 21]},
  {"item": "notebook",
   "qty": 50,
   "tags": ["red", "blank"],
   "dim_cm": [14, 21]},
  {"item": "paper",
   "qty": 100,
   "tags": ["red", "blank", "plain"],
   "dim_cm": [14, 21]},
  {"item": "planner",
   "qty": 75,
   "tags": ["blank", "red"],
   "dim_cm": [22.85, 30]},
  {"item": "postcard",
   "qty": 45,
   "tags": ["blue"],
   "dim_cm": [10, 15.25]}
])
Queries
mongosh: Match an Array

File: /nosql_lab/lecture_6/exercises/mongosh/15-query-find-array-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.find( { tags: "red" } );

db.inventory.find( { tags: ["red", "blank"] } );

db.inventory.find( { tags: { $all: ["red", "blank"] } } );

db.inventory.find( { dim_cm: { $gt: 25 } } );

db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } );

db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } );

db.inventory.find( { "dim_cm.1": { $gt: 25 } } );

db.inventory.find( { "tags": { $size: 3 } } );
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/15-query-find-array-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

print("1.", "~"*30)
# finds any document containing red in tags
for d in db.inventory.find({"tags": "red"}):
  pprint(d)
print()

print("2.", "~"*30)
# finds exact match with ordered list (but not if order does not match or there are more elements)
for d in db.inventory.find({"tags": ["red", "blank"]}):
  pprint(d)
print()

print("3.", "~"*30)
# finds any match with all tags in tags
for d in db.inventory.find({"tags": {"$all": ["red", "blank"]}}):
  pprint(d)
print()

print("4.", "~"*30)
# finds document if ANY of the list items match rule
for d in  db.inventory.find({"dim_cm": {"$gt": 25}}):
  pprint(d)
print()

print("5.", "~"*30)
# finds document if ANY of the list items match rules -> not what you expect! "," = OR operator in lists
for d in db.inventory.find({"dim_cm": {"$gt": 15, "$lt": 20}}):
  pprint(d)
print()

print("6.", "~"*30)
# finds document if at least one of the list items match ALL rules -> $elemMatch = AND operator
for d in  db.inventory.find({"dim_cm": {"$elemMatch": {"$gt": 22, "$lt": 30}}}):
  pprint(d)
print()

print("7.", "~"*30)
# finds document if rule applies to second element
for d in  db.inventory.find({"dim_cm.1": {"$gt": 25}}):
  pprint(d)
print()

print("8.", "~"*30)
# finds document with size of list == 3
for d in  db.inventory.find({"tags": {"$size": 3}}):
  pprint(d)
print()

References
docs.mongodb.com - Query an Array

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Query an Array of Embedded Documents
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 234
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/16-query-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.deleteMany({});


db.inventory.insertMany( [
  { item: "journal",
    instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
  { item: "notebook",
    instock: [ { warehouse: "C", qty: 5 } ] },
  { item: "paper",
    instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
  { item: "planner",
    instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
  { item: "postcard",
    instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/16-query-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([
  {"item": "journal",
   "instock": [
     {"warehouse": "A", "qty": 5},
     {"warehouse": "C", "qty": 15}]},
  {"item": "notebook",
   "instock": [
     {"warehouse": "C", "qty": 5}]},
  {"item": "paper",
   "instock": [
     {"warehouse": "A", "qty": 60},
     {"warehouse": "B", "qty": 15}]},
  {"item": "planner",
   "instock": [
     {"warehouse": "A", "qty": 40},
     {"warehouse": "B", "qty": 5}]},
  {"item": "postcard",
   "instock": [
     {"warehouse": "B", "qty": 15},
     {"warehouse": "C", "qty": 35}]},
])
Queries
mongosh: Query for a Document Nested in an Array

File: /nosql_lab/lecture_6/exercises/mongosh/17-query-find-array-of-embedded-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } );

db.inventory.find( { "instock": { qty: 5, warehouse: "A" } } );

db.inventory.find( { 'instock.qty': { $lte: 20 } } );

db.inventory.find( { 'instock.0.qty': { $lte: 20 } } );

db.inventory.find( { "instock":
                     { $elemMatch: { qty: 5, warehouse: "A" } } } );

db.inventory.find( { "instock":
                     { $elemMatch: { qty: { $gt: 10, $lte: 20 } } } } );

db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 } } );

db.inventory.find( { "instock.qty": 5, "instock.warehouse": "A" } );
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/17-query-find-array-of-embedded-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

print("1.", "~"*30)
# finds exact match
for d in db.inventory.find({"instock": {"warehouse": "A", "qty": 5}}):
    pprint(d)
print()

print("2.", "~"*30)
# finds no match as embeded document does not match order
for d in db.inventory.find({"instock": {"qty": 5, "warehouse": "A"}}):
    pprint(d)
print()

print("3.", "~"*30)
# finds documents with any match for instock list qty
for d in db.inventory.find({'instock.qty': {"$lte": 20}}):
    pprint(d)
print()

print("4.", "~"*30)
# finds documents with match for FIRST instock list qty
for d in db.inventory.find({'instock.0.qty': {"$lte": 20}}):
    pprint(d)
print()

print("5.", "~"*30)
# finds document if at least one of the list items match ALL rules
for d in db.inventory.find({"instock": {"$elemMatch": {"qty": 5, "warehouse": "A"}}}):
    pprint(d)
print()

print("6.", "~"*30)
# same, finds document if at least one of the list items match ALL rules
for d in db.inventory.find({"instock": {"$elemMatch": {"qty": {"$gt": 10, "$lte": 20}}}}):
    pprint(d)
print()

print("7.", "~"*30)
# finds document if any list items match one of the rules
for d in db.inventory.find({"instock.qty": {"$gt": 10, "$lte": 20}}):
    pprint(d)
print()

print("8.", "~"*30)
# same, finds document if any list items match one of the rules
for d in db.inventory.find({"instock.qty": 5, "instock.warehouse": "A"}):
    pprint(d)
print()

References
docs.mongodb.com - Query an Array of Embedded Documents

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Project Fields to Return from Query
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 236
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/18-query-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.deleteMany({});

db.inventory.insertMany( [
  { item: "journal", status: "A",
    size: { h: 14, w: 21, uom: "cm" },
    instock: [ { warehouse: "A", qty: 5 } ] },
  { item: "notebook", status: "A",
    size: { h: 8.5, w: 11, uom: "in" },
    instock: [ { warehouse: "C", qty: 5 } ] },
  { item: "paper", status: "D",
    size: { h: 8.5, w: 11, uom: "in" },
    instock: [ { warehouse: "A", qty: 60 } ] },
  { item: "planner", status: "D",
    size: { h: 22.85, w: 30, uom: "cm" },
    instock: [ { warehouse: "A", qty: 40 } ] },
  { item: "postcard", status: "A",
    size: { h: 10, w: 15.25, uom: "cm" },
    instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/18-query-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([
  {"item": "journal",
   "status": "A",
   "size": {"h": 14, "w": 21, "uom": "cm"},
   "instock": [{"warehouse": "A", "qty": 5}]},
  {"item": "notebook",
   "status": "A",
   "size": {"h": 8.5, "w": 11, "uom": "in"},
   "instock": [{"warehouse": "C", "qty": 5}]},
  {"item": "paper",
   "status": "D",
   "size": {"h": 8.5, "w": 11, "uom": "in"},
   "instock": [{"warehouse": "A", "qty": 60}]},
  {"item": "planner",
   "status": "D",
   "size": {"h": 22.85, "w": 30, "uom": "cm"},
   "instock": [{"warehouse": "A", "qty": 40}]},
  {"item": "postcard",
   "status": "A",
   "size": {"h": 10, "w": 15.25, "uom": "cm"},
   "instock": [
     {"warehouse": "B", "qty": 15},
     {"warehouse": "C", "qty": 35}]}
])
Queries
mongosh: Return All Fields in Matching Documents

File: /nosql_lab/lecture_6/exercises/mongosh/19-query-find-project-fields-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

// SELECT * from inventory WHERE status = "A"
db.inventory.find( { status: "A" } )

// SELECT _id, item, status from inventory WHERE status = "A"
db.inventory.find( { status: "A" }, { item: 1, status: 1 } )

// SELECT item, status from inventory WHERE status = "A"
db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0 } )

db.inventory.find( { status: "A" }, { status: 0, instock: 0 } )

db.inventory.find(
  { status: "A" },
  { item: 1, status: 1, "size.uom": 1 }
)

db.inventory.find(
  { status: "A" },
  { "size.uom": 0 }
)

db.inventory.find( { status: "A" },
                   { item: 1, status: 1, "instock.qty": 1 } )

db.inventory.find( { status: "A" },
                   { item: 1, status: 1, instock: { $slice: -1 } } )

db.inventory.find( { status: "A" },
                   { item: 1, status: 1, instock: { $elemMatch: { warehouse: "C" } } } )

pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/19-query-find-project-fields-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy


print("1.", "~"*30)
# SELECT * from inventory WHERE status = "A"
for d in db.inventory.find({"status": "A"}):
  pprint(d)
print()

print("2.", "~"*30)
# SELECT _id, item, status from inventory WHERE status = "A"
for d in db.inventory.find({"status": "A"}, {"item": 1, "status": 1}):
  pprint(d)
print()

print("3.", "~"*30)
# SELECT item, status from inventory WHERE status = "A"
for d in db.inventory.find({"status": "A"}, {"item": 1, "status": 1, "_id": 0}):
  pprint(d)
print()

print("4.", "~"*30)
# project all, minus the ones set to 0
for d in db.inventory.find({"status": "A"}, {"status": 0, "instock": 0}):
  pprint(d)
print()

print("5.", "~"*30)
# project all set to 1, plus _id
for d in db.inventory.find({"status": "A"}, {"item": 1, "status": 1, "size.uom": 1}):
  pprint(d)
print()

print("6.", "~"*30)
# project all, minus nested field set to 0
for d in db.inventory.find({"status": "A"}, {"size.uom": 0}):
  pprint(d)
print()

print("7.", "~"*30)
# project all set to 1, including nested subfield set to 1
for d in db.inventory.find(
    {"status": "A"}, {"item": 1, "status": 1, "instock.qty": 1}):
  pprint(d)
print()

print("8.", "~"*30)
# project all set to 1, including FIRST element of instock subfield
for d in db.inventory.find(
    {"status": "A"},
    {"item": 1, "status": 1, "instock": {"$slice": 1}}):
  pprint(d)
print()

print("9", "~"*30)
# project all set to 1, including LAST element of instock subfield
for d in db.inventory.find(
    {"status": "A"},
    {"item": 1, "status": 1, "instock": {"$slice": -1}}):
  pprint(d)
print()

print("10", "~"*30)
# project all set to 1, filter instock array
for d in db.inventory.find(
    {"status": "A"},
    {"item": 1, "status": 1, "instock": {"$elemMatch": { "warehouse": "C" }}}):
  pprint(d)
print()



References
docs.mongodb.com - Project Fields to Return from Query

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Query for Null or Missing Fields
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 237
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/20-query-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.deleteMany({});

db.inventory.insertMany([
  { _id: 1, item: null },
  { _id: 2 }
])
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/20-query-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([{"_id": 1, "item": None},
                          {"_id": 2}])
Queries
mongosh: Equality Filter

File: /nosql_lab/lecture_6/exercises/mongosh/21-query-find-missing-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.find( { item: null } );
db.inventory.find( { item : { $type: 10 } } );
db.inventory.find( { item : { $exists: false } } );
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/21-query-find-missing-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

print("1.", "~"*30)
# find document where field value is None
for d in db.inventory.find({"item": None}):
  pprint(d)
print()

print("2.", "~"*30)
# find document where field value is type 10 == None
# MongoDB tpyes: https://docs.mongodb.com/manual/reference/operator/query/type/
for d in db.inventory.find({"item": {"$type": 10}}):
  pprint(d)
print()

print("3.", "~"*30)
# find document where filed name is missing
for d in db.inventory.find({"item": {"$exists": False}}):
  pprint(d)
print()

References
docs.mongodb.com - Query for Null or Missing Fields

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Query for Null or Missing Fields
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 237
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/20-query-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.deleteMany({});

db.inventory.insertMany([
  { _id: 1, item: null },
  { _id: 2 }
])
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/20-query-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([{"_id": 1, "item": None},
                          {"_id": 2}])
Queries
mongosh: Equality Filter

File: /nosql_lab/lecture_6/exercises/mongosh/21-query-find-missing-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.find( { item: null } );
db.inventory.find( { item : { $type: 10 } } );
db.inventory.find( { item : { $exists: false } } );
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/21-query-find-missing-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

print("1.", "~"*30)
# find document where field value is None
for d in db.inventory.find({"item": None}):
  pprint(d)
print()

print("2.", "~"*30)
# find document where field value is type 10 == None
# MongoDB tpyes: https://docs.mongodb.com/manual/reference/operator/query/type/
for d in db.inventory.find({"item": {"$type": 10}}):
  pprint(d)
print()

print("3.", "~"*30)
# find document where filed name is missing
for d in db.inventory.find({"item": {"$exists": False}}):
  pprint(d)
print()

References
docs.mongodb.com - Query for Null or Missing Fields

y( [
  { item: "canvas", qty: 100
    , size: { h: 28, w: 35.5, uom: "cm" }, status: "A" },
  { item: "journal", qty: 25
    , size: { h: 14, w: 21, uom: "cm" }, status: "A" },
  { item: "mat", qty: 85
    , size: { h: 27.9, w: 35.5, uom: "cm" }, status: "A" },
  { item: "mousepad", qty: 25
    , size: { h: 19, w: 22.85, uom: "cm" }, status: "P" },
  { item: "notebook", qty: 50
    , size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
  { item: "paper", qty: 100
    , size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
  { item: "planner", qty: 75
    , size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
  { item: "postcard", qty: 45
    , size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
  { item: "sketchbook", qty: 80
    , size: { h: 14, w: 21, uom: "cm" }, status: "A" },
  { item: "sketch pad", qty: 95
    , size: { h: 22.85, w: 30.5, uom: "cm" }, status: "A" }
] );
pymongo:

File: /nosql_lab/lecture_6/exercises/pycharm/30-update-populate-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([
  {"item": "canvas",
   "qty": 100,
   "size": {"h": 28, "w": 35.5, "uom": "cm"},
   "status": "A"},
  {"item": "journal",
   "qty": 25,
   "size": {"h": 14, "w": 21, "uom": "cm"},
   "status": "A"},
  {"item": "mat",
   "qty": 85,
   "size": {"h": 27.9, "w": 35.5, "uom": "cm"},
   "status": "A"},
  {"item": "mousepad",
   "qty": 25,
   "size": {"h": 19, "w": 22.85, "uom": "cm"},
   "status": "P"},
  {"item": "notebook",
   "qty": 50,
   "size": {"h": 8.5, "w": 11, "uom": "in"},
   "status": "P"},
  {"item": "paper",
   "qty": 100,
   "size": {"h": 8.5, "w": 11, "uom": "in"},
   "status": "D"},
  {"item": "planner",
   "qty": 75,
   "size": {"h": 22.85, "w": 30, "uom": "cm"},
   "status": "D"},
  {"item": "postcard",
   "qty": 45,
   "size": {"h": 10, "w": 15.25, "uom": "cm"},
   "status": "A"},
  {"item": "sketchbook",
   "qty": 80,
   "size": {"h": 14, "w": 21, "uom": "cm"},
   "status": "A"},
  {"item": "sketch pad",
   "qty": 95,
   "size": {"h": 22.85, "w": 30.5, "uom": "cm"},
   "status": "A"}
])
Queries
mongosh:
db.collection.updateOne()
db.collection.updateMany()
db.collection.replaceOne()

File: /nosql_lab/lecture_6/exercises/mongosh/31-update-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.inventory.updateOne(
  { item: "paper" },
  {
    $set: { "size.uom": "cm", status: "P" },
    $currentDate: { lastModified: true }
  }
)

db.inventory.updateMany(
  { "qty": { $lt: 50 } },
  {
    $set: { "size.uom": "in", status: "P" },
    $currentDate: { lastModified: true }
  }
)

db.inventory.replaceOne(
  { item: "paper" },
  { item: "paper",
    instock: [ { warehouse: "A", qty: 60 },
               { warehouse: "B", qty: 40 } ] }
)
pymongo:
db.collection.update_one()
db.collection.update_many()
db.collection.replace_one()

File: /nosql_lab/lecture_6/exercises/pycharm/31-update-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

print("1.", "~"*30)
# update one with multiple fields
for d in db.inventory.find({"item": "paper"}):
  pprint(d)
print("~~~ update_one ~~~")
db.inventory.update_one(
  {"item": "paper"},
  {"$set": {"size.uom": "cm", "status": "P"},
   "$currentDate": {"lastModified": True}}
)
for d in db.inventory.find({"item": "paper"}):
  pprint(d)
print()


print("2.", "~"*30)
# update many with multiple fields
for d in db.inventory.find({"qty": {"$lt": 50}}):
  pprint(d)
print("~~~ update_many ~~~")
db.inventory.update_many(
  {"qty": {"$lt": 50}},
  {"$set": {"size.uom": "in", "status": "P"},
   "$currentDate": {"lastModified": True}}
)
for d in db.inventory.find({"qty": {"$lt": 50}}):
  pprint(d)
print()

print("3.", "~"*30)
# replace one document with new document
for d in db.inventory.find({"item": "paper"}):
  pprint(d)
print("~~~ replace_one ~~~")
db.inventory.replace_one(
  {"item": "paper"},
  {"item": "paper",
   "instock": [
     {"warehouse": "A", "qty": 60},
     {"warehouse": "B", "qty": 40}]}
)
for d in db.inventory.find({"item": "paper"}):
  pprint(d)
print()

References
docs.mongodb.com - Update Documents
Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
SQL to MongoDB Mapping Chart
DBs / MongoDBTools / MongoDB ShellSQLExercise
Page 239
Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/60-sql-mapping-populate-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.people.deleteMany({});

db.people.insertOne(
  {
    _id: ObjectId("509a8fb2f3f4948bd2f983a0"),
    user_id: "abc123",
    age: 55,
    status: 'A'
  }
)
CREATE
mongosh: Create and Alter

File: /nosql_lab/lecture_6/exercises/mongosh/61-sql-mapping-create-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

// CREATE TABLE people
db.createCollection("people")


// CREATE TABLE people (
//     id MEDIUMINT NOT NULL AUTO_INCREMENT,
//     user_id Varchar(30),
//     age Number,
//     status char(1),
//     PRIMARY KEY (id)
// )
db.people.insertOne( {
  user_id: "abc123",
  age: 55,
  status: "A"
} )


// ALTER TABLE people ADD join_date DATETIME
db.people.updateMany(
  { },
  { $set: { join_date: new Date() } }
)


// ALTER TABLE people DROP COLUMN join_date
db.people.updateMany(
  { },
  { $unset: { "join_date": "" } }
)


// CREATE INDEX idx_user_id_asc ON people(user_id)
db.people.createIndex( { user_id: 1 } )


// CREATE INDEX idx_user_id_asc_age_desc
//   ON people(user_id, age DESC)
db.people.createIndex( { user_id: 1, age: -1 } )
INSERT
mongosh: Insert

File: /nosql_lab/lecture_6/exercises/mongosh/62-sql-mapping-insert-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

// INSERT INTO people(user_id,
//                   age,
//                   status)
// VALUES ("bcd001",
//         45,
//         "A")
db.people.insertOne(
   { user_id: "bcd001", age: 45, status: "A" }
)
SELECT
mongosh: Select

File: /nosql_lab/lecture_6/exercises/mongosh/63-sql-mapping-select-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

// SELECT * FROM people
db.people.find()

// SELECT id, user_id, status FROM people
db.people.find(
    { },
    { user_id: 1, status: 1 }
)

// SELECT user_id, status FROM people
db.people.find(
    { },
    { user_id: 1, status: 1, _id: 0 }
)

// SELECT * FROM people WHERE status = "A"
db.people.find(
    { status: "A" }
)

// SELECT user_id, status FROM people WHERE status = "A"
db.people.find(
    { status: "A" },
    { user_id: 1, status: 1, _id: 0 }
)

// SELECT * FROM people WHERE status != "A"
db.people.find(
    { status: { $ne: "A" } }
)

// SELECT * FROM people WHERE status = "A" AND age = 50
db.people.find(
    { status: "A",
      age: 50 }
)

// SELECT * FROM people WHERE status = "A" OR age = 50
db.people.find(
    { $or: [ { status: "A" } , { age: 50 } ] }
)

// SELECT * FROM people WHERE age > 25
db.people.find(
    { age: { $gt: 25 } }
)

// SELECT * FROM people WHERE age < 25
db.people.find(
   { age: { $lt: 25 } }
)

// SELECT * FROM people WHERE age > 25 AND age <= 50
db.people.find(
   { age: { $gt: 25, $lte: 50 } }
)

// SELECT * FROM people WHERE user_id like "%bc%"
db.people.find( { user_id: /bc/ } )
// -or-
db.people.find( { user_id: { $regex: /bc/ } } )

// SELECT * FROM people WHERE user_id like "bc%"
db.people.find( { user_id: /^bc/ } )
// -or-
db.people.find( { user_id: { $regex: /^bc/ } } )

// SELECT * FROM people WHERE status = "A" ORDER BY user_id ASC
db.people.find( { status: "A" } ).sort( { user_id: 1 } )

// SELECT * FROM people WHERE status = "A" ORDER BY user_id DESC
db.people.find( { status: "A" } ).sort( { user_id: -1 } )

// SELECT COUNT(*) FROM people
db.people.count()
// or
db.people.find().count()

// SELECT COUNT(user_id) FROM people
db.people.count( { user_id: { $exists: true } } )
// or
db.people.find( { user_id: { $exists: true } } ).count()

// SELECT COUNT(*) FROM people WHERE age > 30
db.people.count( { age: { $gt: 30 } } )
// or
db.people.find( { age: { $gt: 30 } } ).count()

// SELECT status FROM people
// GROUP BY status;
// --- or ---
// SELECT DISTINCT(status) FROM people
db.people.aggregate( [ { $group : { _id : "$status" } } ] )
// or, for distinct value sets that do not exceed the BSON size limit
db.people.distinct( "status" )

// SELECT * FROM people LIMIT 1
db.people.findOne()
// or
db.people.find().limit(1)

// SELECT * FROM people LIMIT 5 SKIP 10
db.people.find().limit(5).skip(10)

// EXPLAIN SELECT * FROM people WHERE status = "A"
db.people.find( { status: "A" } ).explain()
UPDATE
mongosh: Update Records

File: /nosql_lab/lecture_6/exercises/mongosh/64-sql-mapping-update-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

// UPDATE people SET status = "C" WHERE age > 25
db.people.updateMany(
  { age: { $gt: 25 } },
  { $set: { status: "C" } }
)

// UPDATE people SET age = age + 3 WHERE status = "A"
db.people.updateMany(
  { status: "A" } ,
  { $inc: { age: 3 } }
)
DELETE
mongosh: Delete Records

File: /nosql_lab/lecture_6/exercises/mongosh/65-sql-mapping-delete-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

// DELETE FROM people WHERE status = "D"
db.people.deleteMany( { status: "D" } )

// DELETE FROM people
db.people.deleteMany({})

// DROP TABLE people
db.people.drop()

References
docs.mongodb.com - SQL to MongoDB Mapping Chart
docs.mongodb.com - STerminology and Concepts
Previous PageNext Page
© 2026 by Revolytics AG

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Text Search
DBs / MongoDBTools / MongoDB ShellExercise
Page 241
MongoDB supports query operations that perform a text search of string content. To perform text search, MongoDB uses a text index and the $text operator.

Initialize Data
mongosh:

File: /nosql_lab/lecture_6/exercises/mongosh/70-text-search-population-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.stores.deleteMany({});

db.stores.insert(
   [
     { _id: 1, name: "Java Hut", description: "Coffee and cakes" },
     { _id: 2, name: "Burger Buns", description: "Gourmet hamburgers" },
     { _id: 3, name: "Coffee Shop", description: "Just coffee" },
     { _id: 4, name: "Clothes Clothes Clothes",
       description: "Discount clothing" },
     { _id: 5, name: "Java Shopping", description: "Indonesian goods" }
   ]
);
Searches
mongosh: $text

File: /nosql_lab/lecture_6/exercises/mongosh/71-text-search-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.stores.createIndex( { name: "text", description: "text" } );

db.stores.find( { $text: { $search: "java coffee shop" } } );

db.stores.find( { $text: { $search: "\"coffee shop\"" } } );

db.stores.find( { $text: { $search: '"coffee shop"' } } );

db.stores.find( { $text: { $search: "java shop -coffee" } } );

db.stores.find(
  { $text: { $search: "java coffee shop" } },
  { score: { $meta: "textScore" }
  }).sort( { score: { $meta: "textScore" } } );

References
docs.mongodb.com - Text Search

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations
Geospatial Queries
DBs / MongoDBTools / MongoDB ShellExercise
Page 242
MongoDB supports query operations on geospatial data. This section introduces MongoDB's geospatial features.

Find Restaurants
GeoJSON Objects - Point
mongosh: GeoJSON Objects - Point

location: {
  type: "Point",
  coordinates: [-73.856077, 40.848447]
}
Queries
mongosh: Geospatial Query Operators

File: /nosql_lab/lecture_6/exercises/mongosh/80-GeoJSON-01.js (Lecture 6 - Exercises)

db = db.getSiblingDB('MyMongoManual');

db.places.insert( {
  name: "Central Park",
  location: { type: "Point", coordinates: [ -73.97, 40.77 ] },
  category: "Parks"
});

db.places.insert( {
  name: "Sara D. Roosevelt Park",
  location: { type: "Point", coordinates: [ -73.9928, 40.7193 ] },
  category: "Parks"
});

db.places.insert( {
  name: "Polo Grounds",
  location: { type: "Point", coordinates: [ -73.9375, 40.8303 ] },
  category: "Stadiums"
});

db.places.createIndex( { location: "2dsphere" } );

db.places.find();

db.places.find(
  {
    location:
    { $near:
      {
        $geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },
        $minDistance: 1000,
        $maxDistance: 5000
      }
    }
  }
);

db.places.aggregate( [
  {
    $geoNear: {
      near: { type: "Point", coordinates: [ -73.9667, 40.78 ] },
      spherical: true,
      query: { category: "Parks" },
      distanceField: "calcDistance"
    }
  }
]);
HINT mongosh and pymongo handle index creation differently:
mongosh (Javascript): db.places.createIndex( { location: "2dsphere" } );
pymongo (Python): db.places.create_index([("location", pymongo.GEOSPHERE)])

Advanced Geospatial Queries
Find Restaurants

References
docs.mongodb.com - Geospatial Queries

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations / Usecases from Students Personal Projects
Remove Duplicates
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 246
File: /nosql_lab/lecture_6/exercises/pycharm/91-remove-duplicates-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy


db.inventory.delete_many({})

db.inventory.insert_many([
    {'key': 'a', 'nr': 1},
    {'key': 'a', 'nr': 2},  # duplicate on "key"
    {'key': 'b', 'nr': 1},
    {'key': 'b', 'nr': 2},  # duplicate on "key"
    {'key': 'b', 'nr': 3},  # duplicate on "key"
    {'key': 'c', 'nr': 1},
])

print("1", "~"*30)
for d in db.inventory.find():
    print(d)
print()

pipeline = [
 {
     "$group": {
         "_id": {"key": "$key"},
         "duplicates": { "$addToSet": "$_id" } ,
         "count": { "$sum" : 1 }
     }
 },
 {
     "$match": {
         "count": { "$gt": 1 }
     }
 }
]

print("2", "~"*30)
for d in db.inventory.aggregate(pipeline):
    print(d)
    to_be_deleted = d["duplicates"][1:]  # all exept the first item should be deleted
    db.inventory.delete_many({
        "_id": {"$in": to_be_deleted}
    })
print()

print("3", "~"*30)
for d in db.inventory.find():
    print(d)
print()
Previous PageNext Page
© 2026 by Revolytics AG

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations / Usecases from Students Personal Projects
String To Int and Sort
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 247
File: /nosql_lab/lecture_6/exercises/pycharm/92-string-to-int-sort-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy


db.str_to_int.delete_many({})

db.str_to_int.insert_many([
    {'nr': '12'},
    {'nr': '2'},
    {'nr': '100'}
])

print("1", "~"*30)
for d in db.str_to_int.find({}).sort([("nr",1)]):
    print(d)

pipeline = [
 {
     "$addFields": {
         "nr_str": "$nr",
         "nr_int": {"$toInt": "$nr"},
     }
 },
 {
     "$sort": {
         "nr_int": 1
     }
 }
]

print("2", "~"*30)
for d in db.str_to_int.aggregate(pipeline):
    print(d)

Previous PageNext Page
© 2026 by Revolytics AG	

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations / Usecases from Students Personal Projects
String To Int and Update
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 248
File: /nosql_lab/lecture_6/exercises/pycharm/93-string-to-int-update-01.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.str_to_int.drop()
db.str_to_int.insert_many([
    {'nr': '1'},
    {'nr': '2'},
    {'nr': '3'}
])
print("1", "~"*30)
for d in db.str_to_int.find({}):
    print(d)

db.str_to_int.update_many({}, [{"$set": {"nr": {"$toInt": "$nr"}}}])

print("2", "~"*30)
for d in db.str_to_int.find({}):
    print(d)

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations / Usecases from Students Personal Projects
arrayToObject
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 249
Wildcards in the dot-notation like "currencies.*.name" are not possible in mongodb. But here's are workaround:

File: /nosql_lab/lecture_6/exercises/pycharm/94-arrayToObject.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many( [
   { "_id": 1, "item": "ABC1", "dimensions": [ [ "l", 25 ] , [ "w",  10 ], ["uom", "cm"] ] },
   { "_id": 2, "item": "ABC2", "dimensions": [ [ "l", 50 ], [ "w",  25 ], [ "uom", "cm" ] ] },
   { "_id": 3, "item": "ABC3", "dimensions": [ [ "l", 25 ], [ "l",  "cm" ], [ "l", 50 ] ] }
] )
# student needed the following structure:
target = [
    {"_id": 1, "item": "ABC1", "dimensions": {"l": 25, "w": 10, "uom": "cm"}},
    {"_id": 2, "item": "ABC2", "dimensions": {"l": 50, "w": 25, "uom": "cm"}},
    {"_id": 3, "item": "ABC3", "dimensions": {"l": 50}},
]

print("1", "~"*30)
for d in db.inventory.find():
  print(d)
print()

print("2", "~"*30)
project1 = {
   "$project": {
       "item": 1,
       "dimensions": {"$arrayToObject": "$dimensions"},
   }
}
for d in db.inventory.aggregate([project1]):
  print(d)
print()
Previous PageNext Page
Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations / Usecases from Students Personal Projects
objectToArray
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 250
Wildcards in the dot-notation like "currencies.*.name" are not possible in mongodb. But here's are workaround:

File: /nosql_lab/lecture_6/exercises/pycharm/95-objectToArray.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy

db.inventory.delete_many({})

db.inventory.insert_many([
    {'currencies': {'EUR': {'name': 'Euro', 'symbol': '€'}}},
    {'currencies': {'CHF': {'name': 'Chf', 'symbol': 'CHF'}}},
    {'currencies': {'EUR': {'name': 'Euro', 'symbol': '€'}, 'CHF': {'name': 'Chf', 'symbol': 'CHF'}}}
])
# student needed the following structure:
target = [
    {'currencies': ['Euro']},
    {'currencies': ['Chf']},
    {'currencies': ['Euro', 'Chf']},
]

print("1", "~"*30)
for d in db.inventory.find():
  print(d)
print()

print("2", "~"*30)
project1 = {
   "$project": {
       "currencies": {"$objectToArray": "$currencies"}
   }
}
for d in db.inventory.aggregate([project1]):
  print(d)
print()

print("3", "~"*30)
project2 = {
   "$project": {
       "currencies": "$currencies.v.name"
   }
}
for d in db.inventory.aggregate([project1, project2]):
  print(d)
print()
© 2026 by Revolytics AG

Start / NoSQL Lab / Lectures / Lecture 6 / MongoDB / MongoDB Manual - Part 1 / CRUD Operations / Usecases from Students Personal Projects
Group And Push
DBs / MongoDBTools / PyCharmTools / MongoDB ShellExercise
Page 251
Wildcards in the dot-notation like "currencies.*.name" are not possible in mongodb. But here's are workaround:

File: /nosql_lab/lecture_6/exercises/pycharm/96-group_by_push.py" (Lecture 6 - Exercises)

#!/usr/bin/env python3
import pymongo
from pprint import pprint

client = pymongo.MongoClient("mongodb://localhost")
db = client.MyMongoManualPy


db.inventory.delete_many({})

db.inventory.insert_many([
    {'nr': 1, 'lang': 'de', 'verse': 'v1-de'},
    {'nr': 2, 'lang': 'de', 'verse': 'v2-de'},
    {'nr': 3, 'lang': 'de', 'verse': 'v3-de'},
    {'nr': 1, 'lang': 'en', 'verse': 'v1-en'},
    {'nr': 2, 'lang': 'en', 'verse': 'v2-en'},
    {'nr': 3, 'lang': 'en', 'verse': 'v3-en'},
    {'nr': 4, 'lang': 'en', 'verse': 'v4-en'},
])

# student needed th following structure:
target = [
    {'nr': 1, 'lang_de': 'v1-de', 'lang_en': 'v1-en'},
    {'nr': 2, 'lang_de': 'v2-de', 'lang_en': 'v2-en'},
    {'nr': 3, 'lang_de': 'v3-de', 'lang_en': 'v3-en'},
    {'nr': 4, 'lang_de': None, 'lang_en': 'v4-en'},
]

print("1", "~"*30)
for d in db.inventory.find():
    print(d)
print()

print("2", "~"*30)
group1 = {
        '$group': {
            '_id': {
                'nr': '$nr',
                'lang': '$lang'
            },
            'verses': {
                '$push': '$verse'
            }
        }
    }
pipeline = [group1]
for d in db.inventory.aggregate(pipeline):
    print(d)
print()

print("3", "~"*30)
group2 = {
        '$group': {
            '_id': '$_id.nr',
            'lang_de': {
                '$push': {
                    '$cond': [
                        {
                            '$eq': [
                                '$_id.lang', 'de'
                            ]
                        }, '$verses', '$$REMOVE'
                    ]
                }
            },
            'lang_en': {
                '$push': {
                    '$cond': [
                        {
                            '$eq': [
                                '$_id.lang', 'en'
                            ]
                        }, '$verses', '$$REMOVE'
                    ]
                }
            }
        }
    }
pipeline.append(group2)
for d in db.inventory.aggregate(pipeline):
    print(d)
print()

print("4", "~"*30)
addField1 = {
        '$addFields': {
            'lang_de': {
                '$arrayElemAt': [
                    '$lang_de', 0
                ]
            },
            'lang_en': {
                '$arrayElemAt': [
                    '$lang_en', 0
                ]
            }
        }
    }
pipeline.append(addField1)
for d in db.inventory.aggregate(pipeline):
    print(d)
print()

print("5", "~"*30)
addField2 = {
        '$addFields': {
            'lang_de': {
                '$arrayElemAt': [
                    '$lang_de', 0
                ]
            },
            'lang_en': {
                '$arrayElemAt': [
                    '$lang_en', 0
                ]
            }
        }
    }
pipeline.append(addField2)
pipeline.append({
        '$project' : {
            '_id': 0,
            'nr': '$_id',
            'lang_de': 1,
            'lang_en': 1,
        }
    })
pipeline.append({
        '$sort' : {
            'nr': 1
        }
    })
for d in db.inventory.aggregate(pipeline):
    print(d)
print()

Start / NoSQL Lab / Lectures / Lecture 5 / MongoDB / Exercises / MongoDB Data-Modeling / Examples and Patterns
Tree Structures
DBs / MongoDBData ModelingExercise
Page 206
Model Tree Structures.

MongoDB allows various ways to use tree data structures to model large hierarchical or nested data relationships.

Tree
HINT: Open MongoDB Shell to execute the following commands.

You can find the scripts as JS files in the following foder /nosql_lab/lecture_5/exercises/mongosh/ (Lecture 5 - Exercises)

Parent References
Tree
Tree
// init data -> 200-trees-01.js
var db = db.getSiblingDB('MyMongoManual');

db.categories_parent.deleteMany({});

db.categories_parent.insertMany( [
   { _id: "Books", parent: null },
   { _id: "Programming", parent: "Books" },
   { _id: "Languages", parent: "Programming" },
   { _id: "Databases", parent: "Programming" },
   { _id: "MongoDB", parent: "Databases" },
   { _id: "dbm", parent: "Databases" }
] );

db.categories_parent.createIndex( { parent: 1 } );


// queries -> 200-trees-02.js
var db = db.getSiblingDB('MyMongoManual');

// find node MongoDB
var p = db.categories_parent.findOne( { _id: "MongoDB" } );
console.log(p);

// find children of Databases
var cursor = db.categories_parent.find( { parent: "Databases" } );
cursor.forEach(function(doc){console.log(doc)});
{ _id: 'MongoDB', parent: 'Databases' }

{ _id: 'MongoDB', parent: 'Databases' }
{ _id: 'dbm', parent: 'Databases' }
HINT: We can't find ALL descendants of e.g. Programming. We can't find list of ancestors of e.g. MongoDB.

Child References
Tree
Tree
// init data -> 210-trees-01.js
var db = db.getSiblingDB('MyMongoManual');

db.categories_child.deleteMany({});

db.categories_child.insertMany(
  [
    { _id: "Books", children: [ "Programming" ] },
    { _id: "Programming", children: [ "Databases", "Languages" ] },
    { _id: "Databases", children: [ "MongoDB", "dbm" ] },
    { _id: "Languages", children: [] },
    { _id: "MongoDB", children: [] },
    { _id: "dbm", children: [] }
  ]
);

db.categories_child.createIndex( { children: 1 } );


// queries -> 210-trees-02.js
var db = db.getSiblingDB('MyMongoManual');

// find node Databases
var c = db.categories_child.findOne( { _id: "Databases" } );
console.log(c);

// find parent with child MongoDB
var cursor = db.categories_child.find( { children: "MongoDB" } );
cursor.forEach(function(doc){console.log(doc)});
{ _id: 'Databases', children: [ 'MongoDB', 'dbm' ] }

{ _id: 'Databases', children: [ 'MongoDB', 'dbm' ] }
HINT: We can't find ALL descendants of e.g. Programming. We can't find list of ancestors of e.g. MongoDB.

Array of Ancestors
Tree
Tree
// init data -> 220-trees-01.js
var db = db.getSiblingDB('MyMongoManual');

db.categories_ancestors.deleteMany({});

db.categories_ancestors.insertMany(
  [
    { _id: "Books",
        ancestors: [ ]},
    { _id: "Programming",
        ancestors: [ "Books" ]},
    { _id: "Languages",
        ancestors: [ "Books", "Programming" ]},
    { _id: "Databases",
        ancestors: [ "Books", "Programming" ]},
    { _id: "MongoDB",
        ancestors: [ "Books", "Programming", "Databases" ]},
    { _id: "dbm",
        ancestors: [ "Books", "Programming", "Databases" ]}
  ]
);

db.categories_ancestors.createIndex( { ancestors: 1 } );



// queries -> 220-trees-02.js
var db = db.getSiblingDB('MyMongoManual');

// find node MongoDB
var a = db.categories_ancestors.findOne( { _id: "MongoDB" } );
console.log(a);

// find all children of Programming
var cursor = db.categories_ancestors.find( { ancestors: "Programming" } );
cursor.forEach(function(doc){console.log(doc)});
{ _id: 'MongoDB', ancestors: [ 'Books', 'Programming', 'Databases' ] }

{ _id: 'Languages', ancestors: [ 'Books', 'Programming' ] }
{ _id: 'Databases', ancestors: [ 'Books', 'Programming' ] }
{ _id: 'MongoDB', ancestors: [ 'Books', 'Programming', 'Databases' ] }
{ _id: 'dbm', ancestors: [ 'Books', 'Programming', 'Databases' ] }
HINT: We can't find list of ancestors of e.g. MongoDB.

Materialized Path
Tree
Tree
// init data -> 230-trees-01.js
db = db.getSiblingDB('MyMongoManual');
db.categories_path.deleteMany({});
db.categories_path.insertMany(
  [
      { _id: "Books", path: null }
    , { _id: "Programming", path: ",Books," }
    , { _id: "Databases", path: ",Books,Programming," }
    , { _id: "Languages", path: ",Books,Programming," }
    , { _id: "MongoDB", path: ",Books,Programming,Databases," }
    , { _id: "dbm", path: ",Books,Programming,Databases," }
  ]
);

db.categories_path.createIndex( { path: 1 } );


// queries -> 230-trees-02.js
var db = db.getSiblingDB('MyMongoManual');

// sort nodes by name (lexicographical order)
var cursor = db.categories_path.find().sort( { path: 1 } );
cursor.forEach(function(doc){console.log(doc)});

// find children of Programming node
var cursor = db.categories_path.find( { path: /,Programming,/ } );
cursor.forEach(function(doc){console.log(doc)});

// find children of Books node
var cursor = db.categories_path.find( { path: /^,Books,/ } );
cursor.forEach(function(doc){console.log(doc)});
{ _id: 'Books', path: null }
{ _id: 'Programming', path: ',Books,' }
{ _id: 'Databases', path: ',Books,Programming,' }
{ _id: 'Languages', path: ',Books,Programming,' }
{ _id: 'MongoDB', path: ',Books,Programming,Databases,' }
{ _id: 'dbm', path: ',Books,Programming,Databases,' }

{ _id: 'Databases', path: ',Books,Programming,' }
{ _id: 'Languages', path: ',Books,Programming,' }
{ _id: 'MongoDB', path: ',Books,Programming,Databases,' }
{ _id: 'dbm', path: ',Books,Programming,Databases,' }

{ _id: 'Programming', path: ',Books,' }
{ _id: 'Databases', path: ',Books,Programming,' }
{ _id: 'Languages', path: ',Books,Programming,' }
{ _id: 'MongoDB', path: ',Books,Programming,Databases,' }
{ _id: 'dbm', path: ',Books,Programming,Databases,' }
HINT: We can't find list of ancestors of e.g. MongoDB.

Nested Sets
Tree
Tree
// init data -> 240-trees-01.js
var db = db.getSiblingDB('MyMongoManual');

db.categories_sets.deleteMany({});

db.categories_sets.insertMany(
  [
      { _id: "Books", left: 1, right: 12 }
    , { _id: "Programming", left: 2, right: 11 }
    , { _id: "Languages", left: 3, right: 4 }
    , { _id: "Databases", left: 5, right: 10 }
    , { _id: "MongoDB", left: 6, right: 7 }
    , { _id: "dbm", left: 8, right: 9 }
  ]
);


// queries -> 240-trees-02.js
var db = db.getSiblingDB('MyMongoManual');

var databaseCategory = db.categories_sets.findOne({ _id: "Databases" } );
console.log(databaseCategory);

// find children of Databases
var cursor = db.categories_sets.find(
  {   left: { $gt: databaseCategory.left },
      right: { $lt: databaseCategory.right }
  } );
cursor.forEach(function(doc){console.log(doc)});

// find parents of Databases
var cursor = db.categories_sets.find(
  {   left: { $lt: databaseCategory.left },
      right: { $gt: databaseCategory.right }
  } );
cursor.forEach(function(doc){console.log(doc)});

// sort all nodes (preorder tree traversal) https://www.geeksforgeeks.org/tree-traversals-inorder-preorder-and-postorder/
var cursor = db.categories_sets.find({}).sort({left: 1});
cursor.forEach(function(doc){console.log(doc)});
{ _id: 'Databases', left: 5, right: 10 }

{ _id: 'MongoDB', left: 6, right: 7 }
{ _id: 'dbm', left: 8, right: 9 }

{ _id: 'Books', left: 1, right: 12 }
{ _id: 'Programming', left: 2, right: 11 }

{ _id: 'Books', left: 1, right: 12 }
{ _id: 'Programming', left: 2, right: 11 }
{ _id: 'Languages', left: 3, right: 4 }
{ _id: 'Databases', left: 5, right: 10 }
{ _id: 'MongoDB', left: 6, right: 7 }
{ _id: 'dbm', left: 8, right: 9 }

References
docs.mongodb.com - Model Tree Structures
sqlservercentr