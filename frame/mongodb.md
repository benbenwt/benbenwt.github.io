# 安装

下载后安装，建好log，db文件夹存储日志和数据库。
使用--logpath=。。。设置路径。

进入安装目录的bin目录，

cmd执行mongod.exe，启动mongo服务。

cmd执行mongo.exe，进入mongo环境。

# mongodb增删改查

use test
db.inventory.insert({})
db.inventory.find({})
db.inventory.updateone(<filter>,<update>,<options>)

db.集合名.find(),查看集合数据

例如
db.inventory.updateOne(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)

db.test.deleteMany({})

- mongodb driver增删改查

  - 查
    MongoClient mongoClient=new MongoClient();MongoDatabase db=mongoClient.getDatabase("test");MongoCollection collection=db.getCollection("inventory");Document doc=new Document("name","mongodb");Document doc1= (Document) collection.find().first();MongoCursor<Document> cursor=collection.find().iterator();while(cursor.hasNext())
    {
        System.out.println([cursor.next](http://cursor.next/)().toJson());
    }

    

  - 改
    To update at most a single document (may be 0 if none match the filter), use the [updateOne](http://api.mongodb.com/java/3.3/?com/mongodb/client/MongoCollection.html#updateOne-org.bson.conversions.Bson-org.bson.conversions.Bson-) method to specify the filter and the update document. Here we use the [Updates.set](http://mongodb.github.io/mongo-java-driver/3.3/builders/updates/#set) helper to update the first document that meets the filter i equals 10 and set the value of i to 110:

    collection.updateOne(eq("i", 10), set("i", 110));

    To update all documents matching the filter use the [updateMany](http://api.mongodb.com/java/3.3/?com/mongodb/async/client/MongoCollection.html#updateMany-org.bson.conversions.Bson-org.bson.conversions.Bson-) method. Here we use the [Updates.inc](http://mongodb.github.io/mongo-java-driver/3.3/builders/updates/#increment) helper to increment the value of i by 100 where i is less than 100.

    UpdateResult updateResult = collection.updateMany(lt("i", 100), inc("i", 100));
    System.out.println(updateResult.getModifiedCount());

  - 删
    collection.deleteOne(eq("i", 110));

  - 增
    List<Document> documents = new ArrayList<Document>();
    for (int i = 0; i < 100; i++) {    documents.add(new Document("i", i));}

    collection.insertMany(documents);
    ​