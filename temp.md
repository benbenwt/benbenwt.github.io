[TOC]
### Flink connector，java对象传输到kafka序列化与反序列化
>当dwd层完成计算后，需要将数据写入kafka，然后再由dws从kafka读出该数据，进一步统计。在使用java编写flink程序时，这些数据就是用java对象表示和操作的。所以Flink如何将java对象数据写入kafka，以及如何读出，需要我们自己实现序列化和反序列化方法。通过实现org.apache.flink.api.common.serialization.SerializationSchema接口，我们可以定义自己的序列化方法;
>主要修改addSink和addSource指定的Schema即可。

```
#dwd层，即需要序列化方法将对象写入kafka的应用

package com.ti.dwd;

import com.ti.domain.SecurityInfo;
import com.ti.functions.map.ParseJsonMapFunction;
import com.ti.utils.serializer.FlinkSecurityInfoSerializationSchema;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;

import java.util.Properties;

public class ParseJson {
    public static void main(String[] args) throws Exception {
//        从kafka创建流，元素为json str。
        StreamExecutionEnvironment env=StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        Properties properties=new Properties();
        properties.setProperty("bootstrap.servers","localhost:9092");
        properties.setProperty("group.id","dwd_ParseJson");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset","latest");

        DataStream<String> kafkaStream=env.addSource(new FlinkKafkaConsumer<String>("ods_json",new SimpleStringSchema(),properties));
        kafkaStream.print("2");

        DataStream<SecurityInfo> infoStream=kafkaStream.map(new ParseJsonMapFunction());

        Properties properties1=new Properties();
        properties1.setProperty("bootstrap.servers","localhost:9092");

        infoStream.addSink(new FlinkKafkaProducer<SecurityInfo>("dwd_ParseJson",new FlinkSecurityInfoSerializationSchema<SecurityInfo>(),properties1));
        env.execute("ParseJson");
    }
}

#dws层，即需要反序列化方法将kafka的字节流转为对象的应用
package com.ti.dws;

import com.ti.domain.SecurityInfo;
import com.ti.functions.map.ParseJsonMapFunction;
import com.ti.utils.serializer.FlinkSecurityInfoDeserializationSchema;
import com.ti.utils.serializer.FlinkSecurityInfoSerializationSchema;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;

import java.util.Properties;

public class LocationDayPeriod {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env=StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        Properties properties=new Properties();
        properties.setProperty("bootstrap.servers","localhost:9092");
        properties.setProperty("groupid","dws_LocationDayPeriod");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset","latest");

        DataStream<SecurityInfo> infoStream=env.addSource(new FlinkKafkaConsumer<SecurityInfo>("dwd_ParseJson",new FlinkSecurityInfoDeserializationSchema<SecurityInfo>(SecurityInfo.class), properties));
        StreamTableEnvironment tableEnv=StreamTableEnvironment.create(env);
        Table infoTable=tableEnv.fromDataStream(infoStream);
        infoTable.printSchema();
        tableEnv.toChangelogStream(infoTable).print("infoTable:");
        Table countLocationTable=tableEnv.sqlQuery("select location,sampletime,count(location) from "+infoTable+" group by location,sampletime");
        Table countTypeTable=tableEnv.sqlQuery("select type,sampletime,count(location) from "+infoTable+" group by type,sampletime");
        Table countArchTable=tableEnv.sqlQuery("select architecture,sampletime,count(location) from "+infoTable+" group by architecture,sampletime");
        tableEnv.toChangelogStream(countLocationTable).print("countLocationTable: ");
        tableEnv.toChangelogStream(countTypeTable).print("countTypeTable:");
        tableEnv.toChangelogStream(countArchTable).print("countArchTable:");
//  所有数据中，每天分布了多少个。
        env.execute("locationDayPeriod");
    }
}

```

--
```
#序列化 Flink schema
package com.ti.utils.serializer;

import org.apache.flink.api.common.serialization.DeserializationSchema;
import org.apache.flink.api.common.serialization.SerializationSchema;

public class FlinkSecurityInfoSerializationSchema<T> implements SerializationSchema<T> {

    @Override
    public byte[] serialize(T element) {
        return SerializeUtil.serialize(element);
    }
}

```
--

```
#反序列化  Flink schema
package com.ti.utils.serializer;

import org.apache.flink.api.common.serialization.DeserializationSchema;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.api.java.typeutils.TypeExtractor;

import java.io.IOException;

public class FlinkSecurityInfoDeserializationSchema<T> implements DeserializationSchema<T> {
    private Class<T> clazz;
    public FlinkSecurityInfoDeserializationSchema(Class<T> clazz){
        this.clazz=clazz;
    }

    @Override
    public T deserialize(byte[] message) throws IOException {
        return (T)SerializeUtil.deserialize(message,clazz);
    }

    @Override
    public boolean isEndOfStream(T nextElement) {
        return false;
    }

    @Override
    public TypeInformation<T> getProducedType() {
        return TypeExtractor.getForClass(clazz);
    }
}

```

--
```
#实现具体功能的util

package com.ti.utils.serializer;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class SerializeUtil {

    public static byte[] serialize(Object object) {
        ObjectOutputStream oos = null;
        ByteArrayOutputStream baos = null;
        try {
            // 序列化
            baos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(baos);
            oos.writeObject(object);
            byte[] bytes = baos.toByteArray();
            return bytes;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    @SuppressWarnings("unchecked")
    public static <T> Object deserialize(byte[] bytes,Class<T> className) {
        ByteArrayInputStream bais = null;
        T tmpObject = null;
        try {
            // 反序列化
            bais = new ByteArrayInputStream(bytes);
            ObjectInputStream ois = new ObjectInputStream(bais);
            tmpObject = (T)ois.readObject();
            return tmpObject;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
