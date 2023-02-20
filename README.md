# Getting started

## Configuration

Contextmap generates documentation by scanning your code.
The configuration depends on your project's programming language and framework.

In your configurations you need to include the scan-key. This is needed to uniquely identify your organization's account.
When you login you will find this key under "Administration > Scan".

### Java with Spring

Most of the scanning happens at compile-time for instance the REST API, entities, markdown, etc. these can be
extracted from the code. However, some things are only known at runtime, for instance Spring beans, config-server settings, etc.
Therefor to run a complete scan of the code you should configure both the compile-time plugin and runtime dependency.

#### Compile-time scan

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
      <version>1.21.0</version>
      <configuration>
        <key>PLACE_KEY_HERE</key>
      </configuration>
    </plugin>
  </plugins>
</build>

```

> ✔️ We highly recommend to modify your CI/CD pipeline to include the contextmap scan.
> This way your documentation will be automatically kept up to date.

For multi-module maven projects, the plugin needs to be added only to the root's pom.xml file. That way all
child-modules will also be scanned. In this case an additional configuration property is needed to ensure all modules
are linked to the same component.  
The configuration will look like this:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>io.contextmap</groupId>
      <artifactId>java-spring-compiletime</artifactId>
      <version>1.21.0</version>
      <configuration>
        <key>PLACE_KEY_HERE</key>
        <multiModuleComponentName>COMPONENT_NAME</multiModuleComponentName>
      </configuration>
    </plugin>
  </plugins>
</build>
```

#### Runtime scan

To configure the runtime scanning of your project, add the following dependency to your pom.xml file.
The runtime scan will only happen once at startup of your project.

```xml
<dependencies>
  <dependency>
    <groupId>io.contextmap</groupId>
    <artifactId>java-spring-runtime</artifactId>
    <version>1.21.0</version>
  </dependency>
</dependencies>
```

By default the scan at startup is disabled. To enable it and to define the necessary key, add the following
to your configuration file (for instance the application.properties file)

```ini
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

```ini
contextmap.key=PLACE_KEY_HERE
contextmap.enabled=true
contextmap.scan.multi-module-component-name=COMPONENT_NAME
```

#### Custom annotations

Your code already contains lots of knowledge and information, which contextmap scans as-is.
But sometimes you might want to give a little nudge to your documentation, to emphasize or rephrase something.
We have foreseen a library with custom annotations that can be used to achieve this.
To do so, add the following dependency to your pom.xml file.

```xml
<dependencies>
  <dependency>
    <groupId>io.contextmap</groupId>
    <artifactId>java-annotations</artifactId>
    <version>1.21.0</version>
  </dependency>
</dependencies>
```

You can read more below on how to use these custom annotations.
We do recommend to limit the use of custom annotations to only those cases where it really helps to improve knowledge sharing.

#### Deployments

If you have multiple environments, then you can document which version is deployed to which environment.
This allows you to see deployments in the releases overview of your components.

To achieve this, add the following command to your deployment workflow "`mvn contextmap:deployment -Dversion=VERSION -Denvironment=ENVIRONMENT`".
The version and environment parameters are to be filled in by your workflow.


### Typescript

Any framework which uses typescript can be scanned, for instance Angular, React, etc.
Also note that with the help of @babel/plugin-proposal-decorators you can also configure a non-typescript pure javascript project.

#### Compile-time scan
To configure compile-time scanning of your project, add the following dev-dependency to your package.json file.
Afterwards you can run the compile-time scan either manually, and/or configure your CI/CD pipeline to run the scan.
The command to run the compile-time scan is "`npm run contextmap:scan`". 

```json
"scripts": {
  "contextmap:scan": "node node_modules/@contextmap/typescript-compiletime/cli.js"
},
"devDependencies": {
  "@contextmap/typescript-compiletime": "^1.2.0",
},
"contextmap": {
  "key": "PLACE_KEY_HERE"
}
```

> ✔️ We highly recommend to modify your CI/CD pipeline to include the contextmap scan.
> This way your documentation will be automatically kept up to date.
 
Instead of configuring the key as a property, you can also add it as argument to the script via `--key=PLACE_KEY_HERE`. This
way, you can for instance refer to an environment variable if needed.

#### Custom decorators
Your code already contains lots of knowledge and information, which contextmap scans as-is.
But sometimes you might want to give a little nudge to your documentation, to emphasize or rephrase something.
We have foreseen a package with custom decorators that can be used to achieve this.
To do so, add the following dependency to your package.json file.

```json
"dependencies": {
  "@contextmap/typescript-decorators": "^1.0.0"
}
```

#### Deployments

If you have multiple environments, then you can document which version is deployed to which environment.
This allows you to see deployments in the releases overview of your components.

To achieve this, add the following command to your deployment workflow "`npm run contextmap:deployment -- --version=VERSION --environment=ENVIRONMENT`".
The version and environment parameters are to be filled in by your workflow.

```json
"scripts": {
  "contextmap:deployment": "node node_modules/@contextmap/typescript-compiletime/deployment.js"
}
```

## What is documented

### Java with Spring

#### Properties

The properties are scanned at compile-time.
The overview of a component contains the following details:

- **System name** is based on the property `contextmap.scan.system-name` in your .properties file or .yml file,
  if not available then it falls back to the groupId in the pom.xml
- **Component name** is based on the property `spring.application.name` in your .properties file or .yml file,
  if not available then it falls back to the name in the pom.xml
- **Domain vision statement** is based on the description in the pom.xml
- **Technology** is based on the dependencies in the pom.xml
- **Team** is based on the name of the first developer in the pom.xml
- **Team's organization** is based on the organization of the first developer in the pom.xml
- **Team's email** is based on the email of the first developer in the pom.xml
- **Bytes of code** is determined by scanning the files in your source folder, counting the filesizes
- **Languages** are determined by scanning the files in your source folder, and looking at the filenames
- **Version** is based on the version in the pom.xml
- **Url issue management** is based on the url from issueManagement's url in the pom.xml
- **Url source code** is based on the url from scm's url in the pom.xml
- **Url for external documentation** is based on the url in the pom.xml
- **Url buid pipeline** is based on the url from ciManagement's url in the pom.xml
- **Component type** is based on the property `contextmap.scan.component-type` in your .properties file or .yml file,
  its value can be `MICROSERVICE`, `MICROFRONTEND`, `GATEWAY` or `LIBRARY`. 
  If not available, then it falls back to the default value `MICROSERVICE`.

If you want to extend the overview and include custom information, then add a property
`contextmap.scan.overview-decision-record` in your .properties file or .yml file with the filename of the markdown file
which needs to be included.

#### Domain Entities

Domain entities are scanned at compile-time based on any of the following annotations:

- @Entity (javax.persistence.Entity)
- @MappedSuperclass (javax.persistence.MappedSuperclass)
- @Document (org.springframework.data.mongodb.core.mapping.Document,
  org.springframework.data.elasticsearch.annotations.Document,
  org.springframework.data.couchbase.core.mapping.Document,
  com.microsoft.azure.spring.data.cosmosdb.core.mapping.Document)
- @SolrDocument (org.springframework.data.solr.core.mapping.SolrDocument)
- @RedisHash (org.springframework.data.redis.core.RedisHash)
- @Table (org.springframework.data.cassandra.core.mapping.Table)
- @Node (org.springframework.data.neo4j.core.schema.Node)
- @Container (com.azure.spring.data.cosmos.core.mapping.Container)
- @ContextAggregateRoot (io.contextmap.annotations.ContextAggregateRoot)
- @ContextEntity (io.contextmap.annotations.ContextEntity)

For example:

```java
// For example when using JPA 
@Entity
public class Order {

}

// For example, when a class is considered part of the domain
// even when it is not necessarily persisted
@ContextEntity
public class Order {

}
```

The following annotations will identify an entity as aggregate root:

- @ContextAggregateRoot (io.contextmap.annotations.ContextAggregateRoot)
- @Document (org.springframework.data.mongodb.core.mapping.Document,
  org.springframework.data.elasticsearch.annotations.Document,
  org.springframework.data.couchbase.core.mapping.Document,
  com.microsoft.azure.spring.data.cosmosdb.core.mapping.Document)
- @SolrDocument (org.springframework.data.solr.core.mapping.SolrDocument)
- @Container (com.azure.spring.data.cosmos.core.mapping.Container)

For example:

```java
@Entity
@ContextAggregateRoot
public class Order {

}
```

The custom annotation `@ContextSoftLink` can be used to identify a soft-link between entities.
This can be used to link entities from the same component, but also between different components.
Use this annotation to document otherwise hidden dependencies between entities.

For example to link to an entity in the same component:

```java
@Entity
public class ProductReadModel {
    
  @ContextSoftLink(entity = Product.class)
  private UUID id;
  
}
```

For example to link to an entity in another component:

```java
@Entity
public class OrderItem {
    
  @ContextSoftLink(component = "inventory-service", entityName = "Product")
  private UUID productId;
  
}
```

#### Published REST API

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

REST API's are grouped per class in which they are defined. By default the class' name will be used as title for the group.
You could annotate the class with any of the following annotations. These provide attributes to customize the name and description
which are displayed for the group of REST API's.

- Swagger's @Tag (io.swagger.v3.oas.annotations.tags.Tag)
- Swagger's @Api (io.swagger.annotations.Api)
- Custom annotation @ContextRestApi (io.contextmap.annotations.rest.ContextRestApi)

Methods can be documented by any of the following annotations. This way a description can be added to the endpoint methods.

- Swagger's @Operation (io.swagger.v3.oas.annotations.Operation)
- Swagger's @ApiOperation (io.swagger.annotations.ApiOperation)
- Custom annotation @ContextRestEndpoint (io.contextmap.annotations.rest.ContextRestEndpoint)


Properties of a requestbody or responsebody can be documented by any of the following annotations. 

- Swagger's @Schema (io.swagger.v3.oas.annotations.media.Schema)
- Swagger's @ApiModelProperty (io.swagger.annotations.ApiModelProperty)
- Custom annotation @ContextApiProperty (io.contextmap.annotations.ContextApiProperty)

For example:

```java
public class OrderDto {
    
  @ContextApiProperty(description = "This datetime is in ISO-8601", example = "2021-12-31")
  private LocalDateTime createdOn;

}
```

> By default the name of a property's class is used as its datatype.
> Also by default, all possible values of an enum property are included as example.

If no endpoints are found while scanning for annotations, then a scan for **Spring REST Docs** snippets occurs.
During the build of projects which use Spring REST Docs to document the api, you will need to make sure that 
the snippets are generated before scanning for contextmap, i.e. the tests need to have run.

#### Subscribed REST API

The subscribed REST API is scanned at compile-time.
The synchronous links between components in contextmap are based on the subscribed REST APIs.
The following annotations will identify a class as a subscribed REST API, and as such
create a link between the components. The name attribute of these annotations will be used, to
identify the component.

- @FeignClient (org.springframework.cloud.openfeign.FeignClient)
- @FeignClient (org.springframework.cloud.netflix.feign.FeignClient)
- @LoadBalancerClient (org.springframework.cloud.loadbalancer.annotation.LoadBalancerClient)
- @LoadBalancerClients (org.springframework.cloud.loadbalancer.annotation.LoadBalancerClients)
- @ContextClient (io.contextmap.annotations.client.ContextClient)

> The custom annotation @ContextClient can be used to model any dependency, not just REST.

When using Spring Cloud Gateway, the configuration files are scanned to find routes.
Any configured route which has a load balanced uri-property (i.e. starting with "lb://") will be identified
as a link to another component.

#### Events

Events are scanned at runtime.
The asynchronous links between components in contextmap are based on the events.
Contextmap currently supports scanning events for the following message brokers

- RabbitMQ
- ActiveMQ (JMS)
- Kafka
- Azure EventHub


##### RabbitMQ

Exchanges on which the scanned component publishes messages are scanned by finding Spring beans of type

- Exchange (org.springframework.amqp.core.Exchange)
- RabbitTemplate (org.springframework.amqp.rabbit.core.RabbitTemplate)

Also logback AMQP appenders are scanned and identified as exchanges. 

Queues on which a component subscribes are scanned by finding Spring beans of type
Binding (org.springframework.amqp.core.Binding)

##### ActiveMQ (JMS)

Queues/Topics on which the scanned component publishes messages are scanned by finding Spring beans of type

- Queue (javax.jms.Queue)
- Topic (javax.jms.Topic)

Queues/Topics on which the scanned component subscribes are scanned by finding beans with methods
annotated with @JmsListener (org.springframework.jms.annotation.JmsListener)

##### Kafka

Topics on which the scanned component publishes messages are scanned by finding Spring beans of type

- NewTopic (org.apache.kafka.clients.admin.NewTopic)
- KafkaTemplate (org.springframework.kafka.core.KafkaTemplate)

Topics on which a component subscribes are scanned by finding Spring beans annotated by
KafkaListener (org.springframework.kafka.annotation.KafkaListener)

##### Azure EventHub

EventHubs on which the scanned component publishes messages are scanned by finding Spring beans of type

- EventHubProducerClient (com.azure.messaging.eventhubs.EventHubProducerClient)
- EventHubProducerAsyncClient (com.azure.messaging.eventhubs.EventHubProducerAsyncClient)

EventHubs on which a component subscribes are scanned by finding Spring beans of type

- EventHubConsumerClient (com.azure.messaging.eventhubs.EventHubConsumerClient)
- EventHubConsumerAsyncClient (com.azure.messaging.eventhubs.EventHubConsumerAsyncClient)

##### Event Payload

Use the custom annotation `@ContextEvent` to allow contextmap to identify the payload
(or potentially multiple payloads) of an event which is published.
For example:

```java
// For RabbitMQ you can refer to the name of an Exchange or RabbitTemplate registered as Spring Bean
@ContextEvent(publishedBy = "orderCreatedExchange")
public class OrderCreated {

}

// For ActiveMQ you can refer to the name of a Queue or Topic registered as Spring Bean
@ContextEvent(publishedBy = "orderCreatedTopic")
public class OrderCreated {

}

// For Kafka you can refer to the name of a Topic or KafkaTemplate registered as Spring Bean
@ContextEvent(publishedBy = "orderCreatedTopic")
public class OrderCreated {

}

// Or you could refer to a configuration property for the target name on which is published
@ContextEvent(publishedBy = "${order-created.exchange}")
public class OrderCreated {

}
```

Any annotation used to document a REST requestbody or responsebody, can also be used to document the payload of an event.

#### Storages

Storages are scanned at runtime.
Contextmap currently supports scanning the following types of storages:

- **JDBC databases** are scanned by finding Spring beans of type DataSource
  (javax.sql.DataSource).  
  Tables and views for the current schema/catalog of the database are included in the scan.
- **MongoDB** is scanned by finding Spring beans of type MongoTemplate 
  (org.springframework.data.mongodb.core.MongoTemplate) and ReactiveMongoTemplate 
  (org.springframework.data.mongodb.core.ReactiveMongoTemplate)
- **CosmosDB** is scanned by finding Spring beans of type CosmosTemplate 
  (com.azure.spring.data.cosmos.core.CosmosTemplate) and ReactiveCosmosTemplate 
  (com.azure.spring.data.cosmos.core.ReactiveCosmosTemplate)
- **Solr** is scanned by finding Spring beans of type SolrTemplate
  (org.springframework.data.solr.core.SolrTemplate)
- **ElasticSearch** is scanned by finding Spring beans of type RestHighLevelClient
  (org.elasticsearch.client.RestHighLevelClient), if none are detected then bean of type RestClient
  (org.elasticsearch.client.RestClient) are scanned
- **Caches** are scanned by getting the caches from Spring's CacheManager
  (org.springframework.cache.CacheManager)

Furthermore, a custom annotation @ContextStorage (io.contextmap.annotations.storage.ContextStorage)
can be used to symbolize any custom types of storage

```java
// Example of a class responsible for accessing certain files
// ContextMap can not discover this storage because there is no well-defined SpringBean, hence the need for the custom annotation
@ContextStorage(name = "training-sets", type = StorageType.FILE_SYSTEM)
public class TrainingSetData {

}
```

#### Decision records

Decision records and other markdown files are scanned at compile-time.
This is done by looking at the source folder and checking the file-extension.
Each file with extension `.md`, `.ad` or `.adr` will be included.

Unmodified files will be ignored. If you modify a file which was previously scanned, then
the next time it is scanned it will be updated.

Note that [Mermaid](https://mermaid-js.github.io/mermaid/#/) diagrams can be included. 

#### Diagrams

PlantUML diagrams included in your project are scanned at compile-time.
This is done by looking at the source folder and checking the file-extension.
Each file with extension `.puml`, `.iuml` or `.plantuml` will be included.

Unmodified files will be ignored. If you modify a file which was previously scanned, then
the next time it is scanned it will be updated.

The Smetana rendering engine of PlantUML is used. Depending on the diagram, this means you may need to add
an extra configuration line to ensure the diagram is rendered correctly.
```plantuml
@startuml
!pragma layout smetana    <-- extra configuration line

@enduml
```

#### Features

Features are scanned at compile-time.
This is done by looking at the source folder and checking the file-extension.
Each file with extension `.feature`, or `.story` will be included.

Unmodified files will be ignored. If you modify a file which was previously scanned, then
the next time it is scanned it will be updated.

#### Releases

Releases are scanned at compile-time.
All local tags in Git will be included. The commits associated with each tag are also included.
Only the date and the message of a commit is tracked.
Other information (such as the person who made the commit) is not tracked.

#### Recent Commits

Recent commits are scanned at compile-time.
All commits in Git from the last 90 days will be included.
Only the date and the message of a commit is tracked.
Other information (such as the person who made the commit) is not tracked.

#### Glossary

The glossary terms are scanned at compile-time.
Use the custom annotation @ContextGlossary to scan for terms to include in your glossary.
For example:

```java
@ContextGlossary("A request to make, supply, or deliver food or goods")
public class Order {

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

}
```

#### Actors

The actors are scanned at compile-time.
Use the custom annotation @ContextActor to indicate that a certain actor uses the component.

For example:

```java
@ContextActor("Helpdesk")
public class OrderApplication {

}
```

Or use @ContextActors, to link multiple actors at once.

For example:

```java
@ContextActors({
  @ContextActor("Data Scientist"),
  @ContextActor("Data Engineer")
})
public class DataAnalysisApplication {

}
```


### Typescript

#### Properties
The properties are scanned at compile-time.
The overview of a component contains the following details:

- **System name** is based on the property contextmap.scan.systemName from the package.json file
- **Component name** is based on the property name from the package.json file
- **Domain vision statement** is based on the description from the package.json file
- **Technology** is based on the dependencies from the package.json file
- **Team** is based on the property author.name from the package.json file
- **Team's organization** is based on the property contextmap.scan.organization from the package.json file
- **Team's email** is based on the property author.email from the package.json file
- **Bytes of code** is determined by scanning the source files, counting the filesizes
- **Languages** are determined by scanning the source files, and looking at the filenames
- **Version** is based on the version from the package.json file
- **Url issue management** COMING SOON
- **Url source code** COMING SOON
- **Url for external documentation** COMING SOON
- **Url build pipeline** COMING SOON
- **Component type** is based on the property contextmap.scan.componentType from the package.json file,
  its value can be `MICROSERVICE`, `MICROFRONTEND`, `GATEWAY` or `LIBRARY`
  if not available then it falls back to the default value `MICROFRONTEND`

#### Subscribed REST API
The subscribed REST API is scanned at compile-time.
The synchronous links between components in contextmap are based on the subscribed REST APIs.
Use the custom decorator @ContextClient to identify a dependency to another component.

For example:

```typescript
@ContextClient('webshop-site-gateway')
export class GatewayHttpService {

}
```


#### Decision records

Decision records and other markdown files are scanned at compile-time.
This is done by looking at the source folder and checking the file-extension.
Each file with extension `.md`, `.ad` or `.adr` will be included.

Unmodified files will be ignored. If you modify a file which was previously scanned, then
the next time it is scanned it will be updated.

Note that [Mermaid](https://mermaid-js.github.io/mermaid/#/) diagrams can be included.


#### Features

Features are scanned at compile-time.
This is done by looking at the source folder and checking the file-extension.
Each file with extension `.feature`, or `.story` will be included.

Unmodified files will be ignored. If you modify a file which was previously scanned, then
the next time it is scanned it will be updated.

#### Releases

Releases are scanned at compile-time.
All local tags in Git will be included. The commits associated with each tag are also included.
Only the date and the message of a commit is tracked.
Other information (such as the person who made the commit) is not tracked.

#### Recent Commits

Recent commits are scanned at compile-time.
All commits in Git from the last 90 days will be included.
Only the date and the message of a commit is tracked.
Other information (such as the person who made the commit) is not tracked.

#### Glossary

The glossary terms are scanned at compile-time.
Use the custom decorator @ContextGlossary to scan for terms to include in your glossary.
For example:

```typescript
@ContextGlossary({ definition: 'A request to make, supply, or deliver food or goods' })
export class Order {

}
```

You can specify the name attribute in case the name of the class is not the term you want to use in the glossary.
You can also document any aliases which could be used for the same term.

```typescript
@ContextGlossary({
  name: 'Invoice',
  definition: 'A list of goods sent or services provided, with a statement of the sum due for these',
  aliases: ['Bill']
})
export class InvoiceEntity {

}
```

#### Actors
The actors are scanned at compile-time.
Use the custom decorator @ContextActor/@ContextActors to indicate that a certain actor uses the component.

For example:

```typescript
@ContextActor('Customer')
export class AppComponent {

}
```

Or in case there are multiple actors:

```typescript
@ContextActors(['Customer', 'Support'])
export class AppComponent {

}
```
