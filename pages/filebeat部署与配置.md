- # 部署
	- 下载rpm包 https://www.elastic.co/
	  https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.1.2-x86_64.rpm
	- rpm -ivh 安装
	- 设置监听目录 与 kafka 导出设置
	- ```yaml
	  filebeat.inputs:
	  - type: log
	    enabled: true
	    encoding: "plain"
	    paths:
	      - /root/test_logs/monitor-collect.log
	  
	  processors:
	    - decode_json_fields:
	        fields: ["message"]
	        process_array: false
	        max_depth: 1
	        target: ""
	        overwrite_keys: false
	        add_error_key: true
	  
	  output.kafka:
	    version: "0.10.2.0"
	    hosts: ["127.0.0.1:9092"]
	    topic: "topic-monitor-collect"
	    partition.round_robin:
	      reachable_only: false
	    required_acks: 1
	    compression: gzip
	    max_message_bytes: 100000
	    codec.format:
	      string: '%{[Msg]}'
	  ```
	- 启动filebeat
		- ``filebeat -e -c ./filebeat.yaml``
		- 原始数据 为
		- ``{"Time":"1650510681.524","Level":"info","Msg":{"event":"PutExpo","view":"V001","stand_id":"sd001","put_id":"p002","client_ip":"221.228.242.101","device_id":"D002","server_time":1650425379000,"site_id":"s001","distinct_id":"U002","trigger_time":"1650425378000","project":"PJ1647005747001"}}``
		- 转发到 kafka 中为
		- ``{"client_ip":"221.228.242.101","device_id":"D002","distinct_id":"U002","event":"PutExpo","project":"PJ1647005747001","put_id":"p002","server_time":1650425379000,"site_id":"s001","stand_id":"sd001","trigger_time":"1650425378000","view":"V001"}``