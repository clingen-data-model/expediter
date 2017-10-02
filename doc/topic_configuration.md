```
# Create topics
ubuntu@exchange:/etc/kafka$ kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
Created topic "test".
ubuntu@exchange:/etc/kafka$ kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic gene_dosage
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic "gene_dosage".
ubuntu@exchange:/etc/kafka$ kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic gene_validity
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic "gene_validity".
ubuntu@exchange:/etc/kafka$ kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic actionability
Created topic "actionability".
# Grant localhost ability to read itself (needed to make authorizer work)
kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:ANONYMOUS --allow-host 127.0.0.1 --operation ClusterAction --cluster
# Grant garde-manger user read/write on topics
kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal "User:CN=garde-manger.clinicalgenome.org,OU=Unknown,O=Unknown,L=New York,ST=New York,C=US" --operation Read --operation Write --topic test
# Grant garde-manger read access on garde-manger group
kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal "User:CN=garde-manger.clinicalgenome.org,OU=Unknown,O=Unknown,L=New York,ST=New York,C=US" --operation Read --group garde-manger
Grant serveur read access on topics

```
