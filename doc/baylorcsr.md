➜  ca openssl x509 -req -CA ca-cert -CAkey ca-key -in genome.network.csr -out genome.network-signed.crt -days 365 -CAcreateserial -passin pass:mink58]boots
Signature ok
subject=/C=US/ST=Texas/L=Houston/O=BCM/OU=BRL/CN=genome.network
Getting CA Private Key
➜  ca openssl x509 -req -CA ca-cert -CAkey ca-key -in test.genome.network.csr -out test.genome.network-signed.crt -days 365 -CAcreateserial -passin pass:mink58]boots
Signature ok
subject=/C=US/ST=Texas/L=Houston/O=BCM/OU=BRL/CN=test.genome.network
Getting CA Private Key
- "User:CN=test.genome.network,OU=BRL,O=BCM,L=Houston,ST=Texas,C=US"
- "User:CN=genome.network,OU=BRL,O=BCM,L=Houston,ST=Texas,C=US"
