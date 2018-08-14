java连接kafka，实现消息产生和接收
==============================

# 一. 启动相关服务

## 1.启动Zookeeper服务

## 2.启动kafka相关服务

# 二.代码演示

## 1. 生产者

```java


import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;
 
import java.util.Properties;
 

public class KafkaProducerExample {
 
    public void produceMessage()
    {
        Properties props = getConfig();
        Producer<String, String> producer = new KafkaProducer<String, String>(props);
        String topic="slavetest",key,value;
        for (int i = 0; i < 1000; i++) {
            key = "key"+i;
            value="value"+i;
            System.out.println("TOPIC: slavetest;发送KEY："+key+";Value:"+value);
            producer.send(new ProducerRecord<String, String>(topic, key,value));
            try {
                Thread.sleep(1000);
            }
            catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
 
        producer.close();
    }
 
    // config
    public Properties getConfig()
    {
        Properties props = new Properties();
        props.put("bootstrap.servers", "10.211.55.3:9092");
        props.put("acks", "all");
        props.put("retries", 0);
        props.put("batch.size", 16384);
        props.put("linger.ms", 1);
        props.put("buffer.memory", 33554432);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        return props;
    }
 
    public static void main(String[] args)
    {
        KafkaProducerExample example = new KafkaProducerExample();
        example.produceMessage();
    }
}


```

> 运行代码即可在kafka consumer看到消息。

## 2. 消费者

```java
package bigdata.kafka;
 
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.errors.WakeupException;
 
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicBoolean;
 
public class KafkaConsumerExample {
 
    //config
    public static Properties getConfig()
    {
        Properties props = new Properties();
        props.put("bootstrap.servers", "10.211.55.3:9092");
        props.put("group.id", "testGroup");
        props.put("enable.auto.commit", "true");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
 
        return props;
    }
 
    public void consumeMessage()
    {
        // launch 3 threads to consume
        int numConsumers = 1;
        final String topic = "slavetest";
        final ExecutorService executor = Executors.newFixedThreadPool(numConsumers);
        final List<KafkaConsumerRunner> consumers = new ArrayList<KafkaConsumerRunner>();
        for (int i = 0; i < numConsumers; i++) {
            KafkaConsumerRunner consumer = new KafkaConsumerRunner(topic);
            consumers.add(consumer);
            executor.submit(consumer);
        }
 
        Runtime.getRuntime().addShutdownHook(new Thread()
        {
            @Override
            public void run()
            {
                for (KafkaConsumerRunner consumer : consumers) {
                    consumer.shutdown();
                }
                executor.shutdown();
                try {
                    executor.awaitTermination(5000, TimeUnit.MILLISECONDS);
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
    }
 
    // Thread to consume kafka data
    public static class KafkaConsumerRunner
            implements Runnable
    {
        private final AtomicBoolean closed = new AtomicBoolean(false);
        private final KafkaConsumer<String, String> consumer;
        private final String topic;
 
        public KafkaConsumerRunner(String topic)
        {
            Properties props = getConfig();
            consumer = new KafkaConsumer<String, String>(props);
            this.topic = topic;
        }
 
        public void handleRecord(ConsumerRecord record)
        {
            System.out.println("name: " + Thread.currentThread().getName() + " ; topic: " + record.topic() + " ; offset" + record.offset() + " ; key: " + record.key() + " ; value: " + record.value());
        }
 
        public void run()
        {
            try {
                // subscribe 订阅｀topic
                consumer.subscribe(Arrays.asList(topic));
                while (!closed.get()) {
                    //read data
                    ConsumerRecords<String, String> records = consumer.poll(10000);
                    // Handle new records
                    for (ConsumerRecord<String, String> record : records) {
                        handleRecord(record);
                    }
                }
            }
            catch (WakeupException e) {
                // Ignore exception if closing
                e.printStackTrace();
                if (!closed.get()) {
                    throw e;
                }
            }
            finally {
                consumer.close();
            }
        }
 
        // Shutdown hook which can be called from a separate thread
        public void shutdown()
        {
            closed.set(true);
            consumer.wakeup();
        }
    }
 
    public static void main(String[] args)
    {
        KafkaConsumerExample example = new KafkaConsumerExample();
        example.consumeMessage();
    }
}


```
