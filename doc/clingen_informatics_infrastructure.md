# Components

Note that this only covers components built by the Harvard/Geisinger team, not the parts built by Baylor or Stanford.

## Expediter

### Description

Ansible playbook and roles for installing and configuring Apache Kafka for the Data Exchange on a host (eventually a cluster of hosts).

### Implementation

Ansible used for configuration of Kafka. Secrets necessary for exchange (TLS private certs) are stored in encrypted form in this repository.

### Todos

#### Monitoring

Currently there is no notification system for alerting administrators if the Kafka service fails. The preferred approach would be to use systemd to do this directly. If this is not possible or straightforward, monit would be the next tool to investigate.

#### Testing

Currently there is no automated testing of the Data Exchange configuration. The standard testbed for Ansible roles seems to be a Vagrant based VM. Approach would be to study popular playbooks on Ansible Galaxy for test configurations then replicate existing proven approaches.

#### Automate creation of data exchange topics

While the installation and configuration of Kafka itself is automatic, topics were created manually. This is probably automatable.

#### Automate ACL configuration of data exchange topics

Same for ACLs on the topics. There may be room to develop an Ansible module for Kafka if one does not already exist.

#### Refactor roles and vars according to Ansible best practicces

I've learned a bit more about how best to use Ansible since starting, would be good to apply that to clean up and modularize the code in this repository. Once this is done it should be possible to create a few different playbooks to simply update topics/acls, vs redeploying the whole data exchange...

## Garde-Manger

### Description

Clojure program for reading data from external sources that do not feed data directly into Data Exchange (currently Gene Dosage JIRA, eventually possibly ClinVar) and feeding it into the Data Exchange.

### Implementation

Written in Clojure and desigend to run as a persistent daemon on a host that can communicate with the Data Exchange. Authenticates to data exchange using SSL Cert (included in encrypted form in the repository).

### Todos

#### Testing

There is no test Jira instance; it feels a little excessive to stand one up. At a minimum should test mock input from Jira into data exchange format. Should also test roundtrip of messages posted to test topic on data exchange.

#### Monitoring

As with Kafka itself, should have monitoring and notifications. Same approach is preferred, use systemd, then explore monit.

#### Align message format with interpretation model

The current output format is based on where I thought the puck was moving at the time. It works well enough, largely because I'm only using a portion of the data included in it on the website, but it would be good to align with the interpretation model as I start grabbing more.

#### Read ClinVar and convert to interpretation model format

The intention for the kb is to base all interpretations on the abstraction developed by the DMWG. If we intend to represent ClinVar data in the ClinGen search, the next logical step is to convert ClinVar interpretaitons into the DMWG standard for interpretations.

## Serveur

### Description

Clojure program for reading information from the data exchange and piping it into the knowledge base.

### Implementation

Similar to garde-manger; written in Clojure and intended to be run as a daemon on a host that can read from the data exchange and write to the Neo4j database backed for the knowledge base. As with garde-manger, authenticates to data exchange via SSL-cert; Neo4j via username password.

### Todos

#### Testing

Should at a minimum have code that tests reading from the data exchange and writing into Neo4j. This is a very small program generally, testing need not necessarily go much beyond that.

#### Monitoring

Has similar monitoring requirements to garde-manger

#### Generalize import from RDF and document schema

It would be wonderful to have a module that could read any document in RDF (which data from JSON-LD is capable of becoming). Another person has created just this (discovered by Scott): https://jesusbarrasa.wordpress.com/2016/06/07/importing-rdf-data-into-neo4j/
I am not sure whether this module meets all the requirements, but it is definitely worth investigating. It would be great if it were also possible to document the schema, possibly by adding to an OWL ontology.

## Clingen-Knowledge

### Description

Web application for search and presentation for ClinGen generated data.

### Implementation

A Ruby on Rails web application with a Neo4j backend.

### Todos

#### Testing

There are some tests being run for the kb, but the test suite ought to be expanded.

#### Monitoring

As with the other tools, some sort of systemd monitoring, +/- an external service polling for availability on the website.

#### Deployment

Deployment of new code to the webserver has been scripted, (but could probably be updated with [Ansistrano](https://ansistrano.com). Set up of the webserver itself has not been automated. I have a configuration for a different project that could be used for this, however.

#### Data model migration

The current data model is loosely based on DMWG ideas (especially as those ideas have evolved substantially over the course of development). Now that the interpretation model has been solidified, it would be good to revisit the internal data model of this tool and bring it in alignment with the DMWG standards.

#### Refactor controllers and views around data model migration

Ideally there should be a partial for every object in the data model and views should be configured by composing these. The goal is to reduce code duplication, errors and effort, and maintain a consistent look and feel for users of the site.

#### Search based on ontological relationships

The backend of this was hosted on a graph database with the explicit goal of allowing users to search based on ontologic relationships (i.e., retrieve all assertions based on subclasses of a given term; navagate a tree of relationships in the system). Right now we are only minimally taking advantage of this; it would be good to take this farther.

#### Represent and search for variants/regions

We are looking to import variants from ClinVar and make them searchable on this pwebsite. This will ideally come with a data representation that works for both structural and sequence variants; as well as a visual representation that is coherent between the two.

#### Display rich data from the curation interfaces (as it becomes available via Data Exchange)

Right now we're building a representation of data from the curation interfaces based on a minimal modification of what we're already accepting. It would be good to expand this display to encompass all the data being curated (as it is presented to us in a data model compatible format).

## ClinVar curation tool

### Description

Tool for curating variants present in ClinVar and updating the status of the submission on the ClinVar website.

### Implementation

None to speak of yet.

### Todos

#### Everything

# Tooling

## Apache Kafka

Software for passing messages in the Data Exchange. The Data Exchange is implemented on top of messaging software. Kafka was selected principally due to it's ability to persist data relative to competing software, allowing for a solution to pass data between curation interfaces and several different potential clients, including other curation interfaces and the ClinGen website. Additional required features provided by Kafka are authentication (performed using TLS certs), and ability to operate over insecure networks (accomplished with TLS; we have an internal CA established for this purpose). While the Data Exchange is unlikely to operate at the scale that Kafka is designed for, it is possible to expand to an extremely large scale if necessary. Installation and configuration are performed by Ansible.

## Ansible

Software for configuration management of all server-hosted software (expediter, garde-manger, serveur, clingen-knowledge). Used to deploy updates for ClinGen hosted software as well as server configuration and package dependencies.

## Clojure

Programming language used for managing data flows into and out of the Data Exchange (garde-manger and serveur, so far). Selected for speed of execution and development, maturity, functional programming facilities, and Java interoperability (necessary for OWLApi, useful for Kafka).

## Ruby (on Rails)

Programming language and web framework used in ClinGen Knowledgebase. Selected for maturity, suitability for task (clingen-kb is a read-mostly web application with little need for a single page application style program), ease of development, and developer familiarity.

## Neo4j

Graph database software. Used as the database behind the ClinGen Knowledge Base. Fills the role of a traditional relational database in a web application. Neo4j was selected for its maturity, ease of use (particularly the query language, Cypher), ability to perform graph-based queries (terms for query in the kb are based on graph-structured ontologies).

## JSON-LD

Message serialization standard. Selected for ease of translation into RDF (and therefore connectivity to an onological structure) and usability for people not requiring RDF-structured data (those interested in consuming traditional JSON.

## OWL

Used to describe semantics (and presence) of preloaded types in KB. External OWL ontologies are used for linking assertions to terms. Since it's used already for types defined external to ClinGen, using it internally is a natural extension.
