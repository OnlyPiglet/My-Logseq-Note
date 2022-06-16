- ## 单节点自测部署
	- 下载二进制包
	- 配置zookeeper
		- ```
		  tickTime=2000
		  dataDir=/root/kafka_2.10-0.10.2.0/zk/data
		  dataLogDir=/root/kafka_2.10-0.10.2.0/zk/logs
		  clientPort=2181
		  ```
	- 启动kafka-zooleeper
		- ``./zookeeper-server-start.sh -daemon ../config/zookeeper.properties``
	- 配置kafka
		- ```
		  broker.id=1
		  listeners=PLAINTEXT://0.0.0.0:9092
		  num.network.threads=3
		  num.io.threads=8
		  socket.send.buffer.bytes=102400
		  socket.receive.buffer.bytes=102400
		  socket.request.max.bytes=104857600
		  log.dirs=/root/kafka_2.10-0.10.2.0/logs
		  num.partitions=1
		  num.recovery.threads.per.data.dir=1
		  offsets.topic.replication.factor=1
		  transaction.state.log.replication.factor=1
		  transaction.state.log.min.isr=1
		  log.retention.hours=168
		  log.segment.bytes=1073
		  log.retention.check.interval.ms=300000
		  zookeeper.connect=1.116.69.14:2181
		  zookeeper.connection.timeout.ms=6000
		  group.initial.rebalance.delay.ms=0
		  ```
	- 启动kafka
		- ``./kafka-server-start.sh -daemon  ../config/server.properties``
	- 创建 topic
	- ``./kafka-topics.sh --create --topic="topic-monitor-collect" --zookeeper="127.0.0.1:2181" --partitions=2 --replication-factor=1``
	- 启动consumer
	- ``./kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic topic-monitor-collect``
	- 启动producer
	- ``./kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic topic-monitor-collect``
	-
-