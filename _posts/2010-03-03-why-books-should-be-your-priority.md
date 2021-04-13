---
date: 2018-11-22 12:26:40
layout: post
title: Schema Management and Serialization with Apache Avro
subtitle: Lorem ipsum dolor sit amet, consectetur adipisicing elit.
description: The selection of a data serialization and schema management framework was an important freedom of choice[1] for our organization. During the process, we found ourselves in analysis paralysis [2] due to the number of options, our historical baggage, and competing opinions in the community.
image: https://res.cloudinary.com/dm7h7e8xj/image/upload/v1559822138/theme9_v273a9.jpg
optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559822138/theme9_v273a9.jpg
category: life
tags:
  - books
  - read
author: thiagorossener
paginate: true
---

The selection of a data serialization and schema management framework was an important freedom of choice[1] for our organization. During the process, we found ourselves in analysis paralysis [2] due to the number of options, our historical baggage, and competing opinions in the community. Ultimately, we believe the most important choice we made was to deliberately invest in solving this problem, regardless of the solution. It is our hope that this reflection on our journey is useful to others faced with a similar choice.

<!--page-->

Unlike the choice of message broker, which we have found to be more easily replaced, these frameworks tend to have high inertia. They intertwine with the data model of the system and define contracts between services. They can be expensive to replace. We need to be confident in a solution that addresses our challenges without adding to them or shifting them someplace worse. How can an entire engineering organization effectively share data across dozens of services and safely make changes to the schema as data needs continue to evolve? Will the event system be compatible with the languages of the organization? How do migrations work? What horrors lurk in the night?

After careful consideration, Zefr elected to introduce Apache Avro [[3](#references)] for schema management, and Apache Kafka as our message broker. We succeeded in creating a process where schemas are managed in a language-agnostic format, in source control, for everyone to contribute to and to be used for inter-application communication, data warehousing, testing, and data validation. Schema migrations in production are safe, self-propagating, and fully automated.

## Selecting a Message Broker and Serialization Format

Zefr is a contextual advertising company ingesting hundreds of millions of events a day and streaming them through numerous stages that determine their worthiness to advertisers based on criteria like content, brand safety, channel, length, etc. By 2017 the company had a sizeable number of products and services in market, and had informally adopted the 12 factor app [[4](#references)] and microservice architectural patterns [[5](#references)]. We began the process of formalizing an event sourcing architecture pattern [[6](#references)] to bettern unify a jumble of disjointed services communicating via an array of messaging systems and message formats. Service interface contracts were challenging, as we needed transformation layers for these disparate services to communicate: both the data format, and the message broker. Changes to these contracts introduced risk.

We settled on Apache Kafka as the message broker technology early in the process. This was due to a combination of in-house expertise, community support, technical maturity, and performance. There is much more to say about Apache Kafka and our experiences with the technology, which we will defer to a later time. Importantly for this selection process, the choice of broker helped to inform and constrain our options with respect to serialization format.

![Image](/engineering-blog/assets/img/avro_evolution.jpg)

More stressful for us than choosing the message broker was committing to a message serialization format. We performed an initial discovery process to get a list of our options. JSON and Google Protocol Buffers (protobuf) were both already in use at the company, and the choice was up to the project technical lead. JSON lacked a complete story around schema evolution and we had encountered numerous issues with data integrity. Our choice quickly narrowed to Protobuf or Avro.

### Optional/Nullable Fields

Team members with experience in messaging systems at other companies had been burned by protobuf’s ambiguities in optional field semantics, and this was also our experience using protobuf version 3. The semantics of optional is such that it is impossible to know, without out-of-band signaling, whether a scalar field was set deliberately by a producer, or if the producer didn't set it and it is therefore parsed as a default value [[7](#references)]. This is a reasonable design but was a mismatch in how we thought about optional data semantics.

### Schema Registry Support

Kafka Schema Registry manages schemas and verifies compatibility as they evolve by inferring the schema from a record of a given Kafka topic. At the time, Schema Registry only supported Avro. Such a system would have been prohibitively difficult to develop ourselves, so this was a benefit of using Avro.

### Dynamic Language Support

As prolific Python users, support for dynamic languages was of interest. Avro's Generic Record [[8](#references)] allows for inferring a schema from a record, without needing a defined class. Meanwhile, protobuf requires a preprocessing code generation step in order for the record to be compatible with Python code.

### Choosing Avro

Avro came out on top in these comparisions and had great space and parsing efficiency in the binary format, which we considered to be acceptably human-readable. We have not regretted this choice though, in retrospect, we did not evaluate these different message formats on community support; there seems to be far more discourse on using protobuf with Kafka and that would have been beneficial to us in setting up our systems.

## Schema Management Across an Organization

The next issue would be managing schemas to share across the entire org. Kafka Schema Registry allows for producers to publish new schemas with a unique schema id for clients to be configured to look for but we also wanted more protection from accidental schema updates. We additionally required that dynamic language clients be able to fetch the schema from the repository at run time, static language clients link to released versions of their classes at compile time, and producers either contain a particular schema in their build or fetch by a schema name and version from the schema registry.

![Image](/engineering-blog/assets/full_diagram3.png)

There are two high-level approaches to generating Avro schemas: by hand or generated from code. We hand-write avdl files, which don’t look too far removed from Python dictionaries. A JSON format (AVSC) exists for Avro as well but it is difficult for a human to read and write due to its verbosity. Zefr is a heavy user of the Python serialization library Marshmallow for its HTTP APIs, and alternatively we could have written schemas in Python and converted them to Avro. However we liked having a format that was equally accessible to developers of any language. The IDL is simple enough that non-technical teammates at Zefr have reviewed these schemas. Here is an example:

{% highlight java %}
@namespace ( "com.zefr.avro.message" )
protocol MetadataProtocol {
/**
Event format for video
\*/
record Metadata {
/**
Globally unique identifier for the event across all
topics and messages
_/
union { null, string } uuid = null;
/\*\*
ISO-8601 date when event was created
_/
union { null, string } timestamp = null;
/**
Identifier for the entity that is referred to in the
event
\*/
union { null, string } entity_id = null;
/**
Name of the source app
\*/
union { null, string } source = null;
}
}
{% endhighlight %}

All schemas are stored and managed in version control. Storing these files in version control enforces the discipline of code review and approval, which makes it safe enough for anyone in the organization to propose changes. Once a change is proposed as a GitHub pull request, a series of tests and validation steps is initiated by our build server. Our build server generates artifacts and validates them against production services, including all production Kafka topics. The validation is performed by checking the forwards and backwards compatibility [[9](#references)] of the proposed schema with the latest version in the production Schema Registry. If validation succeeds, and another engineer approves, the change is merged and the updated schema is propagated to the Schema Registry. We also generate artifacts for multiple languages.

## Language Support for Python, Scala, and Kotlin

Almost all of Zefr’s technology is developed in Python, though we have a handful of Kotlin services as well. Each language required its own configuration to import and utilize the Avro schemas for use in inter-service communication. For Python, we developed a meta programming technique that leverages marshmallow to take an AVSC file and dynamically create a module to be pushed to JFrog Artifactory that can be imported and used just like a Python class schema. The Python package fastavro [[10](#references)] substantially improves on serialization/de-serialization speed. It is missing a few features from Avro, (e.g., complex union types) but otherwise delivers a very similar developer experience.

The Python Avro libraries expose Avro records as dictionaries, which makes it intuitive and Pythonic. Here is a usage example that copies the input record, updates the timestamp, and returns a new record.

{% highlight python %}
from copy import deepcopy

import datetime
from uuid import UUID

def update_metadata(record: dict, t: datetime.datetime, uuid: UUID):
output = deepcopy(record)
output['timestamp'] = output.get('timestamp') or t
output['uuid'] = uuid.uuid1()
return output
{% endhighlight %}

For Scala integration, we evaluated several Avro libraries, to include using the generated SpecificRecord [[11](#references)] classes from the reference Java tooling. We decided on Avrohugger [[12](#references)] as it supports native case class generation, as well as support for mapping Avro types to native Scala types.

Zefr began using Kotlin in 2018 for more native Apache Kafka library support via Kafka Streams [[13](#references)]. Integration with Apache Avro was more straightforward than Scala due to Kotlin having a more seamless integration with Java classes. We use a plugin that generates SpecificRecord classes with some nice Kotlin additions such as optional getters and native Kotlin type support. Since we made every field and record optional in our schemas, we found it helpful to use the Kotlin null safety features [[14](#references)]. Below is an example of a Kotlin type extension method [[15](#references)] that operates on an optional field "timestamp" in the record:

{% highlight kotlin %}
fun Metadata.updateMetadata(t: OffsetDateTime,
uuid: UUID): Metadata {
val output = SpecificData().deepCopy(this.schema, this)
output.timestamp = this?.timestamp ?: t.format(
DateTimeFormatter.ISO_INSTANT
)
output.uuid = uuid.toString()
return output
}
{% endhighlight %}

## Schemas and the Data Lake

All topics feed to a Snowflake data lake, where data that was once primarily the exclusive domain of engineers and business intelligence analysts is easily accessed by the rest of the company. An airflow job queries the Kafka cluster on a set interval and attempts to ingest each topic and then create a view in Snowflake. The contents of any topic can be accessed using standardized naming conventions, and it is highly useful in debugging to have the ability to look back in history at our warehoused interprocess communication versus consuming the data in real time. A lot of data science workloads utilize Snowflake after struggles using S3; data scientists have found that writing SQL is easier than writing Spark jobs so almost all discovery, model training, etc. occurs using Snowflake. In addition, our product and executive teams are fairly technical and thrive on this access to data for reporting, business development, and other purposes.

## Lessons Learned the Hard Way

### Nested Records

We have suffered some for our choice to allow the nesting of one record within another. If we had it to do over again, we would have more carefully considered simply including the same fields in the different records because various tools and systems didn’t support nesting. For example, at the time we were using Amazon Redshift as our data warehouse, which, at that time, did not have a concept of a nested type. This meant we needed custom in-line transforms to unnest the records in order for these records to be written to a table. Certain Kafka Connect connectors were also unable to read nested Avro records [[16](#references)], although the S3 sink connector did support writing nested Avro records. KSQL didn’t support nested records at that time, either. In order to support nesting, it required engineering time to support our various use cases, including tracking down known issues in various issue trackers like the Apache Kafka JIRA board [[17](#references)].

### Everything is Optional

One design decision of note was to make all fields optional. As previously mentioned, we were wary of the risks of not being able to deprecate fields and were seeking the ultimate backwards and forwards compatibility. We have considered using annotations to enforce null as a valid option. Unfortunately when you allow null types, you sacrifice type safety.

### Community Support

One downside of using Python with Kafka includes, once again, the relative lack of community discourse on the pairing, though notably fintech company Robinhood uses Kafka with Python on the backend [[18](#references)].

Hardships aside, in an organization where we have already developed such expertise with an affinity for Python, not to mention shared tooling for things like logging and the ease of onboarding new team members, it has been worth the effort to find a solution we are pleased with for our preferred language rather than have to adopt a new one.

## References

1. Freedom of choice: [https://en.wikipedia.org/wiki/Freedom_of_choice](https://en.wikipedia.org/wiki/Freedom_of_choice)
2. Analysis paralysis: [https://en.wikipedia.org/wiki/Analysis_paralysis](https://en.wikipedia.org/wiki/Analysis_paralysis)
3. Avro Spec: [https://avro.apache.org/docs/current/spec.html](https://avro.apache.org/docs/current/spec.html)
4. Twelve-Factor App: [https://12factor.net/](https://12factor.net/)
5. Microservices Architecture: [https://microservices.io/](https://microservices.io/)
6. Event Sourcing Architecture Pattern: [https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
7. Protobuf 3 removal of required: [https://cloud.google.com/apis/design/proto3](https://cloud.google.com/apis/design/proto3)
8. Avro GenericRecord: [https://avro.apache.org/docs/current/api/java/index.html?org/apache/avro/generic/GenericRecord.html](https://avro.apache.org/docs/current/api/java/index.html?org/apache/avro/generic/GenericRecord.html)
9. Avro full Compatibility: [https://docs.confluent.io/current/schema-registry/avro.html](https://docs.confluent.io/current/schema-registry/avro.html)
10. Fastavro: [https://fastavro.readthedocs.io/en/latest/](https://fastavro.readthedocs.io/en/latest/)
11. Kotlin Avro plugin to generate SpecificRecord: [https://github.com/davidmc24/gradle-avro-plugin](https://github.com/davidmc24/gradle-avro-plugin)
12. Scala Avro plugin to generate case classes: [https://github.com/julianpeeters/avrohugger](https://github.com/julianpeeters/avrohugger)
13. Apache Kafka Streams: [https://github.com/apache/kafka/tree/trunk/streams](https://github.com/apache/kafka/tree/trunk/streams)
14. Kotlin type safety: [https://kotlinlang.org/docs/reference/null-safety.html](https://kotlinlang.org/docs/reference/null-safety.html)
15. Kotlin type extension: [https://kotlinlang.org/docs/reference/extensions.html](https://kotlinlang.org/docs/reference/extensions.html)
16. Kafka Connect nested records issue: [https://github.com/confluentinc/kafka-connect-jdbc/issues/611](https://github.com/confluentinc/kafka-connect-jdbc/issues/611)
17. Apache Kafka JIRA board: [https://issues.apache.org/jira/projects/KAFKA/issues](https://issues.apache.org/jira/projects/KAFKA/issues)
18. Robinhood's newer Python Kafka library: [https://github.com/robinhood/faust](https://github.com/robinhood/faust)
