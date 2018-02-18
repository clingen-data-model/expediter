### Generate an SSL certificate
The data exchange will support any X.509 compliant certificate (as tested so far). The following guide was used to generate the gene dosage example: https://docs.confluent.io/current/kafka/ssl.html#overview.

### Send signing request for certificate to Data Exchange admin (Tristan)

The signed certificate will identify your site to Kafka. We'll be self-signing the certificates, rather than using an external authority.

### Install Kafka client for language/programming environment

https://cwiki.apache.org/confluence/display/KAFKA/Clients

Most popular (and many unpopular) programming languages will have recent native Kafka clients. The most recent version of Kafka (0.11) is running on the Data Exchange, though it will support older clients. Clients will need to support at least version 0.9 of the Kafka server in order to support encryption and authentication. Of course it's recommended to use an actively maintained client that supports the current version (0.11) of the Kafka server.

### Configure client to send to data exchange

Configuration will vary by client. The client should be configured to use SSL and to authenticate via a signed certificate.

The host to connect to is: exchange.clinicalgenome.org:9093

Be sure to download and trust the certificate for the local certificate authority governing the Data Exchange

https://raw.githubusercontent.com/clingen-data-model/expediter/master/ca-cert


```clojure
;; src/garde-manger/kafka.clj
(ns garde-manger.kafka
  (:import java.util.Properties
           [org.apache.kafka.clients.producer KafkaProducer Producer ProducerRecord]))

(def client-properties
  {"bootstrap.servers" (System/getenv "DATA_EXCHANGE_HOST") ; exchange.clinicalgenome.org:9093
   "acks" "0"
   "key.serializer" "org.apache.kafka.common.serialization.StringSerializer"
   "value.serializer" "org.apache.kafka.common.serialization.StringSerializer"
   "security.protocol" "SSL"
   ;; Password protected keystores are specific to Java-based clients
   ;; Specify identity and trust certificates in a way appropriate to your client
   "ssl.truststore.location" "keys/garde.truststore.jks"
   "ssl.truststore.password" (System/getenv "GARDE_KEY_PASS")
   "ssl.keystore.location" "keys/garde.keystore.jks"
   "ssl.keystore.password" (System/getenv "GARDE_KEY_PASS")
   "ssl.key.password" (System/getenv "GARDE_KEY_PASS")})

;; Java Properties object defining configuration of Kafka client
(defn client-configuration 
  "Create client "
  []
  (let [props (new Properties)]
    (doseq [p client-properties]
      (.put props (p 0) (p 1)))
    props))

(defn producer
  []
  (let [props (client-configuration)]
    (new KafkaProducer props)))
    
;; src/garde-manger/gene_dosage.clj

(ns garde-manger.gene-dosage
  (:require [clj-http.client :as http]
            [cheshire.core :as json]
            [clojure.set :refer [rename-keys]]
            [garde-manger.kafka :as kafka])
  (:import java.util.Properties
           [org.apache.kafka.clients.producer KafkaProducer Producer ProducerRecord]))
           
(def kafka-topic "gene_dosage")

;; Ommitted code not related to data exchange

(defn push-messages
  "Push incoming messages to Kafka-based data exchange"
  [messages]
  ;; Create Kafka producer. Will leak TCP connection if not closed properly
  (with-open [p (kafka/producer)]
    (doseq [m messages]
      ;; Key of message should be the unique ID/iri of message, body
      ;; should be the JSON representation of the message
      (.send p (ProducerRecord. kafka-topic (:iri m) (json/generate-string m))))))
```

### Send to 'test' topic to validate configuration

All users are given read and write access to this topic

### Send curations to appropriate topic for processing by website

Topics that exist so far are:
* gene_dosage
* actionability
* gene_validity
