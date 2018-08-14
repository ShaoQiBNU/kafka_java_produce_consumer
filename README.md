java连接kafka，实现消息产生和接收
==============================

# 一. 启动相关服务

## 1.启动Zookeeper服务

## 2.启动kafka相关服务

# 二.代码演示（非常简单，写一个KafkaProducer）

```java

import java.util.Properties;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

public class SendDataToKafka {
	public static void main(String[] args) {
		SendDataToKafka sendDataToKafka = new SendDataToKafka();
		sendDataToKafka.send("mysqltest", "", "this is a test data too");
	}

public void send(String topic,String key,String data){
	Properties props = new Properties();
	props.put("bootstrap.servers", "127.0.0.1:9092");
	props.put("acks", "all");
	props.put("retries", 0);
	props.put("batch.size", 16384);
	props.put("linger.ms", 1);
	props.put("buffer.memory", 33554432);
	props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

	KafkaProducer<String, String> producer = new KafkaProducer<String,String>(props);

	for(int i=1;i<2;i++){
		try {
			Thread.sleep(100);
		} 
		catch (InterruptedException e) {
			e.printStackTrace();
		}
	    producer.send(new ProducerRecord<String, String>(topic, ""+i, data));
    }

    producer.close();
    
    }

}

```

> 运行代码即可在kafka consumer看到消息.
