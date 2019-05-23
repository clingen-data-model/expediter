# ClinGen Streaming Platform configuration

This playbook contains the configuration files for the ClinGen Streaming platform. The configuration is set up as an Ansible playbook. Two of the roles (common, kafka) are used for installing Kafka on a new host, or updating it on an existing host. These roles will run on any Debian based system of relatively recent vintage (Debian 8 or higher). The last role, kafka_roles, contains the topics and permissions necessary for all ClinGen roles.

Authentication of clients is accomplished via PKCS certificates, signed by a private CA not included in this repository.

# Running the streaming platform on a new host

It is possible to configure the platform to run on a new host by adding the host configuration to inventory/, following the example of the other configured hosts. It is also necessary to add a keystore and truststore with a signed certificate for the host, at roles/kafka/files/<hostname>.server.[truststore|keystore].jks.

# Adding topics

The first task in roles/kafka_roles/roles/main.yml lists the topics that exist on the server. Simply append a new topic name to this list to add to the server.

# Granting permissions on new topics

Tasks granting read and/or write permissions on the various topics are also included in roles/kafka_roles/roles/main.yml When the distinguished name of the entity you're granting permissions to is known, add the name to the list beneath the topic with the appropriate permissions. Run the playbook to add the appropriate permissions to the server. Maintaining permissions in this way ensures permissions for all topics are documented outside the server and stored in version control.

# Granting permissions on groups

Every certificate should generally have a unique group name. These groups are listed at the end of roles/kafka_roles/roles/main.yml . When a new identity is established, create a group and list it in this file, following the pattern for the other listed groups.

# Notes

Be sure Java Cryptography Extensions are enabled on any system interacting with keystores. Will not work otherwise.
