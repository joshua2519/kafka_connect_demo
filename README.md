啟動docker

```
docker-compose up
```

確認 Kafka connect topic與 status
```
docker exec -it kafka-connect-demo-connect-1-1 /kafka/bin/kafka-topics.sh --bootstrap-server kafka:19092 -list

curl http://localhost:8083/

```
部署source connector
```
curl -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @mysql_source_connector.json

curl http://localhost:8084/connectors/inventory-connector/status
```
確認資料有寫入kafka topics

```
docker exec -it kafka-connect-demo-connect-1-1 /kafka/bin/kafka-topics.sh --bootstrap-server kafka:19092 --list
docker exec -it kafka-connect-demo-connect-1-1 /kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka:19092 --from-beginning --topic dbserver1.inventory.products
```

手動新增MySQL資料
```
docker exec -it mysql mysql -uroot -pdebezium -e 'select * from inventory.products' 

docker exec -it mysql mysql -uroot -pdebezium -e 'insert into inventory.products (id,name,description,weight) values (110,"test","demo",99)' 

docker exec -it mysql mysql -uroot -pdebezium -e 'insert into inventory.products (id,name,description,weight) values (111,"opensource","for you",999)' 
```

關閉 connect worker
確認目前task是在哪台work，關閉後稍等幾秒，task會指派到另一個worker
```
curl http://localhost:8084/connectors/inventory-connector/status

docker stop kafka-connect-demo-connect-1-1

docker stop kafka-connect-demo-connect-2-1

curl http://localhost:8084/connectors/inventory-connector/status 
curl http://localhost:8083/connectors/inventory-connector/status 


```

刪除 connector
```
curl -X DELETE http://localhost:8084/connectors/inventory-connector

```

部署 sink connector與查詢資料是否有回寫MySQL
```
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8084/connectors/ -d @jdbc-sink.json

curl http://localhost:8084/connectors/jdbc-sink/status

docker exec -it mysql mysql -uroot -pdebezium -e 'select * from inventory.dbserver1.inventory.products' 

```

更新 sink connector
```
curl -i -X PUT -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8084/connectors/jdbc-sink/config -d @jdbc-sink-put.json
docker exec -it mysql mysql -uroot -pdebezium -e 'select * from inventory.dbserver1.inventory.customers' 

```

刪除docker
```
docker-compose down

```

