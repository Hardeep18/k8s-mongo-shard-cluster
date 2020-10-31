## Set up Sharding using Docker Containers

### Config servers
Start config servers (3 member replica set)
```
docker-compose -f config-server/docker-compose.yaml up -d
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
docker-compose -f shard1/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://mongosh1-1:27017
```
```
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "mongosh1-1:27017" },
      { _id : 1, host : "mongosh1-2:27017" },
      { _id : 2, host : "mongosh1-3:27017" }
    ]
  }
)

rs.status()
```

### Mongos Router
Start mongos query router
```
docker-compose -f mongos/docker-compose.yaml up -d
```

### Add shard to the cluster
Connect to mongos
```
mongo mongodb://mongocfg1:60000
```
Add shard
```
mongos> sh.addShard("shard1rs/mongocfg1:27017,mongocfg1:27017,mongocfg1:27017")
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
mongo mongodb://mongocfg1:50004
```
```
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "mongocfg1:50004" },
      { _id : 1, host : "mongocfg1:50005" },
      { _id : 2, host : "mongocfg1:50006" }
    ]
  }
)

rs.status()
```
### Add shard to the cluster
Connect to mongos
```
mongo mongodb://mongocfg1:60000
```
Add shard
```
mongos> sh.addShard("shard2rs/mongocfg1:50004,mongocfg1:50005,mongocfg1:50006")
mongos> sh.status()
```
