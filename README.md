# ContextMap

Knowledge sharing and collaboration in an event-driven architecture, with polyglot microservices and self-organizing teams doing devops... it's a challenge.
Contextmap scans your code, generates documentation and visualizes it in a centralized developer portal: accurate, zero-effort and always up to date documentation


## Getting started

### Configuration

Contextmap generates documentation by scanning your code.
The configuration depends on your project's programming language and framework.

#### Java with Spring

Most of the scanning happens at compile-time for instance the REST API, entities, markdown, etc. these can be
extracted from the code. However, some things are only known at runtime, for instance Spring beans, config-server settings, etc.
Therefor to run a complete scan of the code you should configure both the compile-time plugin and runtime dependency.

##### Compile-time scan

To configure compile-time scanning of your project, add the following plugin
to your pom.xml file. Afterwards you can run the compile-time scan either manually, and/or configure
your CI/CD pipeline to run the scan.
The command to run the compile-time scan is "`mvn contextmap:scan`".
Make sure to run this command with the root directory of your project as current directory.

```xml
<build>
  <plugins>
    <plugin>
      <groupId>io.contextmap</groupId>
      <artifactId>java-spring-compiletime</artifactId>
      <version>1.5.0</version>
      <configuration>
        <key>PLACE_KEY_HERE</key>
      </configuration>
    </plugin>
  </plugins>
</build>
```

> ✔️ We highly recommend to modify your CI/CD pipeline to include the contextmap scan.
> This way your documentation will be automatically kept up to date.

For multi-module maven projects, the plugin needs to be added to the root's pom.xml file. That way all
child-modules will be scanned. In this case an additional configuration property is needed to ensure all modules
are linked to the same component.  
The configuration will look like this:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>io.contextmap</groupId>
      <artifactId>java-spring-compiletime</artifactId>
      <version>1.5.0</version>
      <configuration>
        <key>PLACE_KEY_HERE</key>
        <multiModuleComponentName>COMPONENT_NAME</multiModuleComponentName>
      </configuration>
    </plugin>
  </plugins>
</build>
```

##### Runtime scan

To configure the runtime scanning of your project, add the following dependency to your pom.xml file.
The runtime scan will only happen once at startup of your project.

```xml
<dependencies>
  <dependency>
    <groupId>io.contextmap</groupId>
    <artifactId>java-spring-runtime</artifactId>
    <version>1.5.0</version>
  </dependency>
</dependencies>
```

By default the scan at startup is disabled. To enable it and to define the necessary key, add the following
to your configuration file (for instance the application.properties file)

```properties
contextmap.key=PLACE_KEY_HERE
contextmap.enabled=true
```

> ✔️ If you have multiple environments, such as development, test, production, etc. then you want to make
> sure to only configure the runtime scan on one environment. This way you will have a consistent view
> of a single environment.

For multi-module maven projects, the dependency needs to be added in the module which is used to run the project.
(i.e. which contains the executed main method).
An additional property is required to indicate that the runtime scan needs to be added to a multi-module project.  
The configuration will look like this:

```properties
contextmap.key=PLACE_KEY_HERE
contextmap.enabled=true
contextmap.scan.multi-module-component-name=COMPONENT_NAME
```

##### Custom annotations

Your code already contains lots of knowledge and information, which contextmap scans as-is.
But sometimes you might want to give a little nudge to your documentation, to emphasize or rephrase something.
We have foreseen a library with custom annotations that can be used to achieve this.
To do so, add the following dependency to your pom.xml file.

```xml
<dependencies>
  <dependency>
    <groupId>io.contextmap</groupId>
    <artifactId>java-annotations</artifactId>
    <version>1.5.0</version>
  </dependency>
</dependencies>
```

You can read more below on how to use these custom annotations.
We do recommend to limit the use of custom annotations to only those cases where it really helps to improve knowledge sharing.

### What is documented

#### Java

##### Properties

The properties are scanned at compile-time.
The overview of a component contains the following details:

- **System name** is based on the property contextmap.scan.system-name in your .properties file or .yml file,
  if not available then it falls back to the &lt;groupId /&gt; in the pom.xml
- **Component name** is based on the property spring.application.name in your .properties file or .yml file,
  if not available then it falls back to the &lt;name /&gt; in the pom.xml
- **Domain vision statement** is based on the &lt;description /&gt; in the pom.xml
- **Technology** is based on the &lt;dependencies /&gt; in the pom.xml
- **Team** is based on the first &lt;developer /&gt; in the pom.xml
- **Bytes of code** is determined by scanning the files in your source folder, counting the filesizes
- **Languages** are determined by scanning the files in your source folder, and looking at the filenames
- **Version** is based on the &lt;version /&gt; in the pom.xml
- **Url issue management** is based on the url from &lt;issueManagement&gt;&lt;url /&gt;&lt;/issueManagement&gt; in the pom.xml
- **Url source code** is based on the url from &lt;scm&gt;&lt;url /&gt;&lt;/scm&gt; in the pom.xml
- **Url for external documentation** is based on the &lt;url /&gt; in the pom.xml
- **Url buid pipeline** is based on the url from &lt;ciManagement&gt;&lt;url /&gt;&lt;/ciManagement&gt; in the pom.xml
- **Component type** is based on the property contextmap.scan.component-type in your .properties file or .yml file,
  its value can be `MICROSERVICE`, `MICROFRONTEND` or `GATEWAY`,
  if not available then it falls back to the default value `MICROSERVICE`

##### Domain Entities

Domain entities are scanned at compile-time based on any of the following annotations:

- @Entity (javax.persistence.Entity)
- @MappedSuperclass (javax.persistence.MappedSuperclass)
- @Document (org.springframework.data.mongodb.core.mapping.Document,
  org.springframework.data.elasticsearch.annotations.Document,
  org.springframework.data.couchbase.core.mapping.Document)
- @SolrDocument (org.springframework.data.solr.core.mapping.SolrDocument)
- @RedisHash (org.springframework.data.redis.core.RedisHash)
- @Table (org.springframework.data.cassandra.core.mapping.Table)
- @Node (org.springframework.data.neo4j.core.schema.Node)
- @ContextAggregateRoot (io.contextmap.annotations.ContextAggregateRoot)
- @ContextEntity (io.contextmap.annotations.ContextEntity)

For example:

```java
@Entity
public class Order {
  ...
}
```

The following annotations will identify an entity as aggregate root:

- @ContextAggregateRoot (io.contextmap.annotations.ContextAggregateRoot)
- @Document (org.springframework.data.mongodb.core.mapping.Document,
  org.springframework.data.elasticsearch.annotations.Document,
  org.springframework.data.couchbase.core.mapping.Document)
- @SolrDocument (org.springframework.data.solr.core.mapping.SolrDocument)

For example:

```java
@Entity
@ContextAggregateRoot
public class Order {
  ...
}
```

##### Published REST API

The published REST API is scanned at compile-time.
The following annotations will identify a class as a published REST API:

- @RestController (org.springframework.web.bind.annotation.RestController)
- @Controller (org.springframework.stereotype.Controller)

Any method included in such a class with one of the following annotations, is seen as a published endpoint

- @GetMapping (org.springframework.web.bind.annotation.GetMapping)
- @PostMappping (org.springframework.web.bind.annotation.PostMapping)
- @PutMapping (org.springframework.web.bind.annotation.PutMapping)
- @DeleteMapping (org.springframework.web.bind.annotation.DeleteMapping)
- @PatchMapping (org.springframework.web.bind.annotation.PatchMapping)
- @RequestMapping (org.springframework.web.bind.annotation.RequestMapping)

The custom annotation `@ContextApiProperty` can be used to customize the documentation of a property.

For example:

```java
public class OrderDto {
  ...
  @ContextApiProperty(description = "This datetime is in ISO-8601", example = "2021-12-31")
  private LocalDateTime createdOn;
  ...
}
```

> By default the name of a property's class is used as its datatype.
> Also by default, all possible values of an enum property are included as example.

##### Subscribed REST API

The subscribed REST API is scanned at compile-time.
The synchronous links between components in contextmap are based on the subscribed REST APIs.
The following annotations will identify a class as a subscribed REST API, and as such
create a link between the components:

- @FeignClient (org.springframework.cloud.openfeign.FeignClient)
- @FeignClient (org.springframework.cloud.netflix.feign.FeignClient)

##### Events

Events are scanned at runtime.
The asynchronous links between components in contextmap are based on the events.
Contextmap currently supports scanning the following eventing solutions

- RabbitMQ
- ActiveMQ (JMS)

###### RabbitMQ

Exchanges on which the scanned component publishes messages are scanned by finding Spring beans of type

- Exchange (org.springframework.amqp.core.Exchange)
- RabbitTemplate (org.springframework.amqp.rabbit.core.RabbitTemplate)

Queues on which the scanned component subscribes are scanned by finding Spring beans of type
Binding (org.springframework.amqp.core.Binding)

###### ActiveMQ (JMS)

Queues/Topics on which the scanned component publishes messages are scanned by finding Spring beans of type

- Queue (javax.jms.Queue)
- Topic (javax.jms.Topic)

Queues/Topics on which the scanned component subscribes are scanned by finding beans with methods
annotated with @JmsListener (org.springframework.jms.annotation.JmsListener)

###### Event Payload

Use the custom annotation `@ContextEvent` to allow contextmap to identify the payload
(or potentially multiple payloads) of an event which is published.
For example:

```java
// For RabbitMQ you can refer to the name of an Exchange or RabbitTemplate registered as Spring Bean
@ContextEvent(publishedBy = "orderCreatedExchange")
public class OrderCreated {
  ...
}

// For ActiveMQ you can refer to the name of a Queue or Topic registered as Spring Bean
@ContextEvent(publishedBy = "orderCreatedTopic")
public class OrderCreated {
  ...
}

// Or you could refer to a configuration property for the target name on which is published
@ContextEvent(publishedBy = "${order-created.exchange}")
public class OrderCreated {
  ...
}
```

##### Storages

Storages are scanned at runtime.
Contextmap currently supports scanning the following types of storages:

- **JDBC databases** are scanned by finding Spring beans of type DataSource
  (javax.sql.DataSource).   
  Tables and views for the current schema/catalog of the database are included in the scan.
- **MongoDB** are scanned by finding Spring beans of type MongoTemplate
  (org.springframework.data.mongodb.core.MongoTemplate)
- **Solr** is scanned by finding Spring beans of type SolrTemplate
  (org.springframework.data.solr.core.SolrTemplate)
- **Caches** are scanned by getting the caches from Spring's CacheManager
  (org.springframework.cache.CacheManager)
 

##### Decision records

Decision records and other markdown files are scanned at compile-time.
This is done by looking at the source folder and checking the file-extension.
Each file with extension `.md`, `.ad` or `.adr` will be included.

Unmodified files will be ignored. If you modify a file which was previously scanned, then
the next time it is scanned it will be updated.

##### Features

Features are scanned at compile-time.
This is done by looking at the source folder and checking the file-extension.
Each file with extension `.feature`, or `.story` will be included.

Unmodified files will be ignored. If you modify a file which was previously scanned, then
the next time it is scanned it will be updated.

##### Releases

Releases are scanned at compile-time.
All local tags in Git will be included. The commits associated with each tag are also included.
Only the date and the message of a commit is tracked.
Other information (such as the person who made the commit) is not tracked.

##### Recent Commits

Recent commits are scanned at compile-time.
All commits in Git from the last 90 days will be included.
Only the date and the message of a commit is tracked.
Other information (such as the person who made the commit) is not tracked.

##### Glossary

The glossary terms are scanned at compile-time.
Use the custom annotation @ContextGlossary to scan for terms to include in your glossary.
For example:

```java
@ContextGlossary("A request to make, supply, or deliver food or goods")
public class Order {
  ...
}
```

You can specify the name attribute in case the name of the class is not the term you want to use in the glossary.
You can also document any aliases which could be used for the same term.

```java
@ContextGlossary(
  value = "A list of goods sent or services provided, with a statement of the sum due for these",
  name = "Invoice",
  aliases = {"Bill"}
)
public class InvoiceEntity {
  ...
}
```
