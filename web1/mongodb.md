# 安装

下载后安装，建好log，db文件夹存储日志和数据库。
使用--logpath=。。。设置路径。

进入安装目录的bin目录，

cmd执行mongod.exe，启动mongo服务端。

cmd执行mongo.exe，进入mongo客户端环境。

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

# porblem

### springboot mongo封装document

>解决，提供了类似driver的接口，例如getcollection，iterator等。

springboot默认封装，而我查询的功能只需要单纯的数据交互。

解决:springboot不适合这种单一的业务，使用php，nodejs等。

对于可能要修改数据的业务，可使用spirngboot，将bson映射到对象，再进行操作。

但是，拥有stix2对象，由于对象数据类型变动，仍然无法灵活的映射到对象进行更改。no，stix2官方提供了映射到对象的方法了，由json到对象。但是无法作为mongo的实体类啊。

>百度相关：spring-data-mongo将mongodb的查询数据转换成了实体类，实体类又被responsebody转换成了JSON字符串，这真是够折腾的。
>
>在数据表达上面，MongoDB虽然使用的是JSON，但是由于JSON的一些先天不足（如类型欠缺），数据库没有办法在底层使用它，所以MongoDB中存储的实际是BSON，这是一种有利于存储和传输但是人不可读的二进制格式，从MongoDB得到的结果实际上是BSON。无论是BSON还是JSON，都不是Java原生支持的类型，所以它们在Java中都有对应的封装类。BSON对应的是BsonDocument，JSON类型并没有在驱动程序中体现，因为驱动给出的是Document类型，这个稍后再讲。从整体流程上看，驱动在发出请求之后得到的是一个BSON文档，可以理解为一堆二进制数据，这是没法直接用的，所以通过反序列化变成BsonDocument，这个类型就有一些有意思的API了，比如toJson。理论上从这里就可以直接变成JSON返回回去了，但是实际使用中，我们只有很少的情况是需要JSON字符串作为结果的，更多的情况下，我们是需要把文档中的值取出来用。为了这个目的，所有的查询方法返回的都是Document，它相当于一个字典或者哈希表，通常这就是我们使用的基础类型，同样它也是支持toJson方法的。下面才说到SpringData，它比驱动位于更上层，实际上是调用驱动然后在这个基础上实现了更多上层功能，比如映射成POJO就是其中之一，剩下的诸如各种CRUD方法。如前面所说，站在前端的角度可能JSON对你是最有用的，大部分时候我们需要的并不是把一个对象变成JSON，而是要根据业务逻辑方便地查询和修改其中的值。这也是SpringData的意义所在。如果你的唯一目的就是变成JSON，那么根本就用不上SpringData，直接用驱动就好了，确实效率是更好的，并且驱动其实本身更接近Mongo一些，SpringData为了在不同数据库之间形成共同的语义，做了一些自己的封装，有时候并不是十分容易理解。说了这么多，不知道有没有加深你对Mongo驱动和SpringData在定位上的理解，希望有所帮助。

### 运行后连接断开

解决：无密码即可连接，将yml中用户名和密码删除。

### 密码问题

默认无密码，通过ip+port即可访问。



### platform临时代码

```
package com.ti.platform_provider_stix8001.mapper;



import static com.mongodb.client.model.Filters.*;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Repository;



import org.bson.Document;

import javax.sound.midi.Soundbank;
import java.io.IOException;

@Repository
public class TestMapper {

    @Autowired
    private MongoTemplate mongoTemplate;


    public void find1()
    {
        System.out.println(mongoTemplate);

        MongoCollection<Document> collection=mongoTemplate.getCollection("inventory");
        System.out.println(collection);
        System.out.println(collection.find());
        MongoCursor<Document> cursor=collection.find().iterator();
        while(cursor.hasNext())
        {
            System.out.println(cursor.next());
        }
        Integer sum= Math.toIntExact(collection.countDocuments(eq("type", "bundle")));
        System.out.println(sum);
    }

    public void testHbase() throws IOException {
        Configuration configuration=HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hbaseserver");
        configuration.set("hbase.zookeeper.property.clientPort", "2181");
        Connection connection= ConnectionFactory.createConnection(configuration);
        System.out.println(connection.toString());
    }
}
```