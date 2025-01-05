# Queue membership manager
Created: 2024-11-20

## Problems
Message brokers now mostly provide a feature called ordering key:
- kafka partition key
- gcp pub/sub ordering key
- natJetstream partitioned consumer group: https://github.com/nats-io/nats-server/issues/2043 
- rabbitmq routing key with + rabbitmq_consistent_hash_exchange plugin 
- SQS message group ID https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-key-terms.html
This feature allow the processing of messages belong to the same key happen sequentially and "usually" by the same instance, denoted by consumer affinity

The consumer affinities by message brokers:
- kafka: strict
- gcp pub/sub: best-effort
- sqs fifo: unknown
- natJetstream: strict
- rabbitmq: best effort

This "usually" shall further described as "sticky consumption". Because of this sticky locality, it provides a lot of benefit if the app has some logic related to aggregation.

For example an app to calculate the total birth count distribution of America group by distrinct over time and do some expensive calculation based on this count (Assuming that calculating all the count for each district is very expensive an cannot be maintained at database level)
Where each consumer will receive a payload like
- baby_id
- district

And the partition/ordering key is distrinct, meaning the application which is processing for a district can just store the count of every district  in-memory and increment it everytime it receives new message for this distrinct

initial vendor1_cache_key: 0
T0: thread1 receive vendor1
check cache freshness: incrby(key,1) = 0 => own = true
....
T100: thread1 receive vendor1
check cache freshness: key(vendor1) = 0  => own = true
update in-mem cache
T100+10ms thread2 receive vendor1 
check cache freshness: incrby(key,1) = 1 => own = true
T100+20ms thread2 receive vendor1
check cache freshness: incrby(key,1) = 2 => own = FALSE
in validate in-memory cache

Because "usually"
consumer-like application often leverage data locality provided by sticky
During message-processing,A lot of consumer-type application maintain some in-mem cache in every instance of the app. I.e a consumer may does multiple things:
- receive msg of entity/item
- insert item into primary storage
- update item in cache
- leverage data in cache to do extra logic

	



	There is one problem, if the messages belonging to the same item/entity goes to different instance of app, it is meaningless to maintain such in-mem data 

## References
1. 
tags: 