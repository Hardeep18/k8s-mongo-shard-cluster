## Set up Sharding using Docker Containers

### Config servers
Start config servers (3 member replica set)
```
kubectl apply -f mongo_config.yaml -n database
```
Initiate replica set
```
mongo mongodb://mongocfg1:27017
```
```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "mongocfg1:27017" },
      { _id : 1, host : "mongocfg2:27017" },
      { _id : 2, host : "mongocfg3:27017" }
    ]
  }
)

rs.status()
```

### Shard 1 servers
Start shard 1 servers (3 member replicas set)
```
kubectl apply -f mongo_sh_1.yaml -n database
```
Initiate replica set
```
mongo mongodb://mongosh1-1:27017
```
```
rs.initiate(
  {
    _id: "rs1",
    members: [
      { _id : 0, host : "mongosh1-1:27017" },
      { _id : 1, host : "mongosh1-2:27017" },
      { _id : 2, host : "mongosh1-3:27017" }
    ]
  }
)

rs.status()
```

### Shard 2 and 3 servers
Start shard 1 servers (3 member replicas set)
```
kubectl apply -f mongo_sh_2.yaml -n database
```
Initiate replica set
```
mongo mongodb://mongosh2-1:27017
```
```
rs.initiate(
  {
    _id: "rs2",
    members: [
      { _id : 0, host : "mongosh2-1:27017" },
      { _id : 1, host : "mongosh2-2:27017" },
      { _id : 2, host : "mongosh2-3:27017" }
    ]
  }
)

rs.status()
```

### Shard 3 servers
Start shard 1 servers (3 member replicas set)
```
kubectl apply -f mongo_sh_3.yaml -n database
```
Initiate replica set
```
mongo mongodb://mongosh3-1:27017
```
```
rs.initiate(
  {
    _id: "rs3",
    members: [
      { _id : 0, host : "mongosh3-1:27017" },
      { _id : 1, host : "mongosh3-2:27017" },
      { _id : 2, host : "mongosh3-3:27017" }
    ]
  }
)

rs.status()
```





### Mongos Router
Start mongos query router
```
kubectl apply -f mongos.yaml -n database
```

### Add shard to the cluster
Connect to mongos
```
mongo mongodb://mongocfg1:27017
```
Add shard
```
mongos> sh.addShard("rs1/mongosh1-1:27017,mongosh1-2:27017,mongosh1-3:27017")
mongos> sh.status()
```

## Adding another shard
### Shard 2 servers
Start shard 2 servers (3 member replicas set)
```
docker-compose -f shard2/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://mongocfg1:27017
```
```
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "mongocfg1:27017" },
      { _id : 1, host : "mongocfg1:27017" },
      { _id : 2, host : "mongocfg1:27017" }
    ]
  }
)

rs.status()
```
### Add shard to the cluster
Connect to mongos
```
mongo mongodb://mongocfg1:27017
```
Add shard
```
mongos> sh.addShard("rs2/mongosh2-1:27017,mongosh2-2:27017,mongosh2-3:27017")
mongos> sh.status()
```

```
mongos> sh.addShard("rs3/mongosh3-1:27017,mongosh3-2:27017,mongosh3-3:27017")
mongos> sh.status()
```
