#判断 collection 是否存在
```
   mongoose.connection.db.listCollections({name: queueName})
        .next(function (err, collinfo) {
            if (collinfo) {
                var queue = mongoDbQueue(db.dbConn, queueName);
                //添加到队列
                addQueue(queue, req.body, res);
            } else {
                //不存在则新建queue
                var deadQueue = mongoDbQueue(db.dbConn, 'dead-queue');
                var queue = mongoDbQueue(db.dbConn, queueName, {
                    visibility: 5,
                    delay: 0,
                    deadQueue: deadQueue
                });
                //添加到队列
                addQueue(queue, req.body, res);
            }
        });
```
