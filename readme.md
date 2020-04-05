# Udemy Course: Apache Kafka Series - Confluent Schema Registry & REST Proxy

* [Course Link](https://www.udemy.com/course/confluent-schema-registry)
* [Course Repo](https://github.com/simplesteph/kafka-avro-course)
* [Lecturers Website](https://courses.datacumulus.com/)

## Section 1: Course Introduction

### Lecture 1. The need of schemas in Kafka

* kafka takes bytes as an input and publishes them (that why we need serializer/deserializer)
* no data verifications
* Τhe Need  for a schema registry
    * what if the producer sends bad data
    * what if a field gets renamed?
    * what if the data format changes from one day to another?
    * Consumers Break!!! (UBER had that problem)
* We need data to be self describable
* We need to be able to evolve data without breaking downstream consumers
* What if Kafka Brokers verified the messages they receive?
    * that would break kafka efficiency
    * kafka is just pass through
* Kafka Schema Registry: 
    * has to be a separate component
    * Producers and Consumers need to be able to talk to it
    * it must be able to reject data
* a common data format must be agreed upon
    * it needs to support schemas
    * it needs to support evolution
    * it needs to be lightweight
* Confluent Schema Registry!! used Apache Avro as data format

### Lecture 2. Course Structure & Objectives

* Part 1: Avro Schemas + AVro in Java
* Part 2: Kafka Schema Registry, Writing Avro Producers and Consumers in Java
* Part 3: Kafka REST Proxy - Using Insomnia REST Client
* Learn about Avro:
    * Understand and write schemas, primitive and complex types
    * Create Avro objects in Java and serialize/deserialize them
    * Learn about schema evolution and developer workflow
* Learn about Schema Registry
    * Architecture and purpose of the Schema Registry
    * Usage for kafka Producer,Consumer,Kafka Sttreams Kafka Connect
    * Understand backward,forward and full compatibility
* Learn about the Rest Proxy
    * Achitecture, APIs

### Lecture 4. Architecture for Kafka with the Schema Registry and REST Proxy

* Pipeline without Schema Registry: Source => Java Producer => Kafka =>java Consumer => Target System
* Confulet Schema Registry Purpose
    * Store and retrieve schemas for Producers/Consumers
    * Enforce Backward/Forward compatibility on topics
    * Decrease the size of payload
* Confluent Schema Registry architecture
    * Producer sends Avro Content to Kafka
    * Producer sends Schema to Schema Registry
    * Consumer reads Avro Content from Kafka
    * Consumer gets Schema from Schema Registry
* Confluent REST Proxy
    * Kafka Cluster
    * Kafka Schema Registry <= Register/Retreve => Kafka Rest Proxy
    * Kafka Rest Proxy <>=Write/Consume Data to/from Kafka => Kafka Cluster 
    * Non Java Producers = HTTP POST=> Kafka Rest Proxy =HHTP GET=> Non-Java Condumers
* A good client for testing REST APIs is [Insomnia REST client](https://insomnia.rest/)

## Section 3: Avro Schemas

### Lecture 6. What is Avro?


* Evolution of dat:
* CSV:
    * easy to parse + 
    * easy to read + 
    * easy to make sense of + 
    * data types of elements have to be inferent. no guarantees -
    * parsing is tricky when data contain delimiters -
    * we might or not have colun names -
* Relational Table Definitions
    * data fully typed + 
    * data fits a table +
    * data has to be flat (no nesting) -
    * data stored in DB, data def different for each DB
* JSON
    * data can taky any form (arrays, nested elements) +
    * JSON is widely accepted format on web +
    * JSON can be red by any language
    * JSON has no schema enforcing -
    JSON objects can be big because of repetition in keys
* AVRO
    * AVRO is defined by a schema (schema is written in JSON)
    * to get started we can view AVRO as JSON with a schema attached to it (Schema + Payload)
    * Data is Fully typed + 
    * Data is compressed automatically (less CPU usage) + 
    * Schema comes with the data + 
    * Documetation is embedded in the schema + 
    * Data can be read across any language + 
    * schema can evolve over time in a safe manner (schema evolution)
    * AVRO support for some languaes might be lacking -
    * Cant print or see the data without using the AVRO tools (its compressed and serialized)
* AVRO vs Protobuf vs Thrift vs Parquet vs ORC
    * overall all of these data formats achieve more or less the same goal
    * for Kafka we care about one message being self explicit and fully described as we're dealing with streaming (no ORC Parquet etc)
    * Avro has good support from Hadoop based tech like Hive
    * Avro is selected as only supported data format from Confluent Schema Registry so well go with that
    * there is no need to compare perf. we need proof that avro is the bottleneck  in our app. (we need insane volumes of 1mil messages per sec to see that)

### Lecture 7. Avro Primitive Types

* Primitive types are the support base types
    * null
    * boolean
    * int
    * long
    * float
    * double
    * bytes: 8bit unsigned array (char[])
    * string
* '.avsc' files are for avro schema files (JSON format)
* it can be as simple as `{"type": "string"} ` for primitive types. instead of string we could have any of the above

### Lecture 8. Avro Record Schema Definition

* Avro Record Schemas are defined using JSON
* It has some common fields"
* Name: name of the schema
* Namespace: (equivalent of package in Java)
* Doc: Documentation to explain the schema
* Aliases: optional other names for the schema
* Fields:
    * Name: name of the field
    * Doc: documentation for the field
    * Type: datatype for that field (can be a primitive type)
* Well practice defining an Avro Schema for Customer (fname,lname,age,height,wight,autoemail(bool) "default true")
```
{
    "type":"record",
    "namespace": "com.example",
    "name": "Customer",
    "doc": "Avro Schema for Customer",
    "fields": [
        {"name": "first_name", "type": "string", "doc": "First name of customer"},
        {"name": "last_name", "type": "string", "doc": "last name of customer"},
        {"name": "age", "type": "int", "doc": "age of customer"},
        {"name": "height", "type": "float", "doc": "height in cms"},
        {"name": "weight", "type": "float", "doc": "weight in kgs"},
        {"name": "automated_email", "type": "boolean", "default": true, "doc": "true if customer wants marketing emails"}
    ]
}
```

### Lecture 9. Avro Complex Types

* In Avro we have complex types like:
    * Enums
    * Arrays
    * Maps
    * Maps
    * Unions
    * Calling other Schemas as types
* Enums are for fields we know that their vals can be enumerated
    * Example: Customer status (Bronze,Silver,Gold) `{"type": "enum", "name": "CustomerStatus","symbols":["BRONZE","SILVER","GOLD"]}`
    * once an enum is set, changing the enum values is fobidden if we want to maintain compatibility
* Arrays a re a way for us to represent a list of undefined size of items that all share the same schema
    * example customer emails: `{"type": "array", "items": "string"}`
* Maps are a way to define a list of keys and values where the keys are strings
    * example: secrets questions `{"type":"map","values": "string"}`
    * do not store secrets in Avro
* Unions can allow a field value to take different types
    * if defaults are defined the default must be the type of the first item in the union
    * the most common use case for unions is to define an option value `{"name":"middle_name","type":["null","string"],"default":null}`

### Lecture 10. Practice Exercise: Customer & CustomerAddress

* we ll define a more complex avro schema
* Customer
    * first name
    * middle name (optional)
    * last name 
    * age
    * automated email (boolean, default true)
    * customer emails (array)
    * customer address (schema record)
* CustomerAddress
    * address
    * city
    * postcode (numer or string)
    * type (enum: PO BOX, RESIDENTIAL, ENTERPRISE)
* first we define the CustomerAddress record
```
[
  {
      "type": "record",
      "namespace": "com.example",
      "name": "CustomerAddress",
      "fields": [
        { "name": "address", "type": "string" },        
        { "name": "city", "type": "string" },       
        { "name": "postcode", "type": ["int","string"] },        
        { "name": "type", "type": "enum", "symbols": ["PO BOX", "RESIDENTIAL", "ENTERPRISE"] }        
      ]
  },
  {
     "type": "record",
     "namespace": "com.example",
     "name": "Customer",
     "fields": [
       { "name": "first_name", "type": "string" },
       { "name": "middle_name", "type": ["null","string"],"default": null },
       { "name": "last_name", "type": "string" },
       { "name": "age", "type": "int" },
       { "name": "height", "type": "float" },
       { "name": "weight", "type": "float" },
       { "name": "automated_email", "type": "boolean", "default": true },
       { "name": "customer_emails","type": "array","items":"string","default": []},
       { "name": "customer_address","type": "com.exampleCustomerAddress" }
     ]
}]
```

* when we pass another schema as type (nested schema) we pass its name complte with the namespace in front

### Lecture 11. Avro Logical Types

* Avro has a concept of logical types used to give more meaning to already existing primitive types
* Most commonly used are
    * decimals (bytes )
    * date (int) - number of days since UNIX epoch
    * time-millis (long) - num of millisecconds after midnight 
    * timestamp-millis (long) - number of milliseconds since epoch
* to use logical type we add "logicalType":"time-millis" to the field name and it will help avro aschema processors to inter a specific type
* Example customer signup timestamp `{"name":"singup_ts","type":"long","logicalType":"timestamp-millis"}`
* logical tyles are new (1.7.7) not fully supported by all languages and dont play nicly with unions. be careful when using them!!!!

### Lecture 12. The complex case of Decimals

* Floats and doubles are floating binary point types. They represent a number like 101101.01011
* Decimal is a floating decimal point type. they represent a number like this 12345.5677 
* some decimals cannot be represented accurately as floats or doubles
* People use floats and doubles for sientific computations (imprecise computations) because whese types are fast
* People use decimals for money. thats why it got createdin the first place. Use them when you need extra accurate results
* Avro uses a decimal logical type. but the primitive type used is "bytes". That means that if you print an avro as json we wont see the dec imal value. but some raw data
* transforming these bytes into a decimal is very error prone if the lang library we use didn't implement the feature corrclty
* we should check the maturity state in 2020 before using them.
* aletrnatives:
    * use "string". looks good easy to parse and understand
    * create our own decimal type: integer part (long), decimal part (long)
    * create our own fraction type: numerator (long) denominator (long)

### Lecture 13. Example of Avro Schemas on the Web

* [Avro Documentation](http://avro.apache.org/docs/current/spec.html)
* [Oracle Avro Getting Started](https://docs.oracle.com/cd/E57769_01/html/GettingStartedGuide/avroschemas.html)
* [Avro Schemas used by Rabo Bank](https://github.com/Axual/rabo-alerts-blog-post/tree/master/src/main/avro)
* [Avro examples by Gwen Shapira](https://github.com/gwenshap/kafka-examples)

## Section 4: Avro in Java

### Lecture 16. Generic Record in Avro - Hands On

* A GenericRecord is used to create an avro object from a schema, the schema being referenced as:
    * a file
    * a string
* It's not the most recomended way of creating Avro objects because things can fail at runtime, but it is the most simple way
* We ll learn to
    * create a generic record
    * write it to a file (avro .avsc file)
    * read it from a file
    * read the generic record
* In IntelliJ new => file => project => maven (1.8 java)
    * groupId: com.github.achliopa.avro
    * artifact-id: avro-examples
    * enable auto imports
* in pom.xml we cp the following from the courseRepo project
* first the avro version
```
    <properties>
        <avro.version>1.8.2</avro.version>
    </properties>
```
* the dependency lib for avro
```
        <!--Only dependency needed for the avro part-->
        <!-- https://mvnrepository.com/artifact/org.apache.avro/avro -->
        <dependency>
            <groupId>org.apache.avro</groupId>
            <artifactId>avro</artifactId>
            <version>${avro.version}</version>
        </dependency>
```
* last in plguins we force java version
* and we add a plugin to generate java from records
```
            <!--for specific record-->
            <plugin>
                <groupId>org.apache.avro</groupId>
                <artifactId>avro-maven-plugin</artifactId>
                <version>${avro.version}</version>
                <executions>
                    <execution>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>schema</goal>
                            <goal>protocol</goal>
                            <goal>idl-protocol</goal>
                        </goals>
                        <configuration>
                            <sourceDirectory>${project.basedir}/src/main/resources/avro</sourceDirectory>
                            <stringType>String</stringType>
                            <createSetters>false</createSetters>
                            <enableDecimalLogicalType>true</enableDecimalLogicalType>
                            <fieldVisibility>private</fieldVisibility>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```
* the last plugin import is to tell where to pick generated source from
```
<!--force discovery of generated classes-->
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>build-helper-maven-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <execution>
                        <id>add-source</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>add-source</goal>
                        </goals>
                        <configuration>
                            <sources>
                                <source>target/generated-sources/avro</source>
                            </sources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```
* we add a package in src>main>java 'com.github.achliopa.avro' and in it a 'generic' package
* in it we add class 'GenericRecordExample
* add a main method and in it
0. define schema (we pass it as a string)
```
        Schema.Parser parser = new Schema.Parser();
        Schema schema = parser.parse("{\n" +
                "     \"type\": \"record\",\n" +
                "     \"namespace\": \"com.example\",\n" +
                "     \"name\": \"Customer\",\n" +
                "     \"doc\": \"Avro Schema for our Customer\",     \n" +
                "     \"fields\": [\n" +
                "       { \"name\": \"first_name\", \"type\": \"string\", \"doc\": \"First Name of Customer\" },\n" +
                "       { \"name\": \"last_name\", \"type\": \"string\", \"doc\": \"Last Name of Customer\" },\n" +
                "       { \"name\": \"age\", \"type\": \"int\", \"doc\": \"Age at the time of registration\" },\n" +
                "       { \"name\": \"height\", \"type\": \"float\", \"doc\": \"Height at the time of registration in cm\" },\n" +
                "       { \"name\": \"weight\", \"type\": \"float\", \"doc\": \"Weight at the time of registration in kg\" },\n" +
                "       { \"name\": \"automated_email\", \"type\": \"boolean\", \"default\": true, \"doc\": \"Field indicating if the user is enrolled in marketing emails\" }\n" +
                "     ]\n" +
                "}");
```
1. create a generic record
```
        GenericRecordBuilder customerBuilder = new GenericRecordBuilder(schema);
        customerBuilder.set("first_name","John");
        customerBuilder.set("last_name","Dow");
        customerBuilder.set("age",25);
        customerBuilder.set("height", 170f);
        customerBuilder.set("weight", 80.5f);
        customerBuilder.set("automated_email",false);
        GenericData.Record customer = customerBuilder.build();
```
* if we omit fields from the record we get error so schema must be followed
* non complyance produces runtime error AvroRuntimeException so we need to try/catch them and test code
2. write the generic record in file
```
        final DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(schema);
        try (DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<>(datumWriter)) {
            dataFileWriter.create(myCustomer.getSchema(), new File("customer-generic.avro"));
            dataFileWriter.append(myCustomer);
            System.out.println("Written customer-generic.avro");
        } catch (IOException e) {
            System.out.println("Couldn't write file");
            e.printStackTrace();
        }
```
* to write to a file we use a DatumWriter passing the schema and then we wrote to the file
* we see that in the file we pass the schema and then the record
* we run the code and peek in the generated 'customer-generic.avro'. its compacted and non readable in general
3. read generic record from file
```
final File file = new File("customer-generic.avro");
        final DatumReader<GenericRecord> datumReader = new GenericDatumReader<>();
```
4. interpret as a generic record
```
GenericRecord customerRead;
        try (DataFileReader<GenericRecord> dataFileReader = new DataFileReader<>(file, datumReader)){
            customerRead = dataFileReader.next();
            System.out.println("Successfully read avro file");
            System.out.println(customerRead.toString());

            // get the data from the generic record
            System.out.println("First name: " + customerRead.get("first_name"));

            // read a non existent field
            System.out.println("Non existent field: " + customerRead.get("not_here"));
        }
        catch(IOException e) {
            e.printStackTrace();
        }
```
* we use next to pull records out of the file`customerRead = dataFileReader.next();`
* we use .hasNext() to query if there are more records to be pulled
* getters give us the fields

### Lecture 17. Specific Record in Avro - Hands On

* SpecificRecord is also an Avro Object but it is obtained using code generation from an Avro schema
* There are different plugins for different build tools (gradle,maven,sbt) but in our example we ll use the official code generation tool shipping with Avro: Maven
* We will create a file containing the AvroSchema
* We will apply the Maven Plugin 
* We will get generated code
* We will perform the exact same tasks as the previous lecture, but all using a SpecificRecord now
* we create a new directory in resources folder called 'avro' and in it a text file 'customer.avsc'
* we cp in it the customer schema
* specific record code generation from the avro schema in .avsc file is possible with the 2 plugins we added in the pom.xml file
    * the code genration plugin uses the avro version, gets the path to the file and a set of rules to follow
    * the second plugin gets told where to find the generated sources
* we run either `mvn clean package` or do it from UI to rebuild the project and trigger code generation from schema
* in target>generated sources>avro we find the  generated class following the namespace and name convention rom schema
* we create a new package in main src>main>java named 'com.github.achliopa.avro.specific' and in it a class 'SpecificRecordExamples' and add a main
* steps we will implement
1. create specific record
```
        Customer.Builder customerBuilder = Customer.newBuilder();
        customerBuilder.setAge(25);
        customerBuilder.setFirstName("John");
        customerBuilder.setLastName("Doe");
        customerBuilder.setHeight(176f);
        customerBuilder.setWeight(80.5f);
        customerBuilder.setAutomatedEmail(false);
        Customer customer = customerBuilder.build();
```
* building a record is so much easier and safer now
* we get compile errors for not following the field type rules
* next steps are same as previous examples
2. write to file
3. read from file
4. interpret

### Lecture 18. Check-in on now vs later in Kafka

* we used java to create an avro object 
* then we wrote the avro bytes to an avro file
* then we red the avro bytes from file to an avro object with java
* with kafka instead of avro file we have the kafka+schema registry

### Lecture 19. Avro Tools - Hands On

* it is possible to read avro files using the avro tools commands
* these are very handy when we want to display (print) data to our command line for a quick analysis of a content of an avro file
* in our courserepo /avro-java folder we have an avro-tools.sh fil with the necessary commands
* in project folder we `wget https://repo1.maven.org/maven2/org/apache/avro/avro-tools/1.8.2/avro-tools-1.8.2.jar`
* from terminal in the project dir we run avro tools with `java -jaravro-tools-1.8.2.jar avro-tools-1.8.2.jar`
* we will use tojson option to convert an avro object from file to json for printing `java -jar avro-tools-1.8.2.jar tojson --pretty customer-generic.avro`
* we see the record not the schema.

### Lecture 20. Reflection in Avro - Hands On

* we can use Reflection in order to build Avro schemas fo our class
* This is less common scenario but still a valid one. usefull when we want to add classes to our avro bvjects
* Workflow: Existing Java Class => Reflection => Avro Schema and Object
* an example can be found in the courseRepo in avro-examples reflection package
* there we use a simple POJO 'ReflectedCustomer' as blueprint also it has a Nullable filed (union between null and string in our case)
* in the 'ReflectionExamples.java' class the magic happens
* we use ReflectData to build an object+schema from Class and write/read to file

### Lecture 21. Schema Evolution - Theory

* schema evolution in a nutshell is the reason we use avro in kafka
* avro enables us to evolve our schema over time, to adapt with the changes from the business
* e.g in v1 we ask for First and last name of customer in v2. we want to ask for their phone number
* we want to evolve without breaking our programs reading existing streams
* There are 4 kinds of schema evolution: 
* Backward compatible: 
    * when a new schema can be used to read old data. 
    * in avro this can be done by adding fields with default value. if filed does not exist. avro with use the default. 
    * we want backward compatibility when we want to perform qqueries (eg in hive) over old and new data using the new schema
* Forward Compatible: 
    * when an old schema can be used to read new data. 
    * in avro we just add a field. we can read new data with old schema. avro will just ignore the new fields. * Deleting fileds without defaults is NOT forward compatible. 
    * we want this compatibility when we want to make  a data stream evolve without changing downstream consumers
* Fully compatible: 
    * forward+backward
    * in avro only adding fields with defaults in the new schema ensures full compatibility
    * in avro only removing fields with defaults in the new schema ensures full compatibility
    * We should follow these rules to ensure full compatibility and peace of mind
* Breaking: 
    no compatibility at all
    * In AVRO: adding/removing elements from ENUMS, changing type of field, renaming a field without defaults
* Rules to follow in AVRO for Schema Evolutiuon
    * make primary key required
    * give default values to all fileds that might be removed in future
    * be careful with enums as they cant evolve
    * dont rename fields. use aliases
    * when evolving the schema ALWAYS give default values
    * when evolving schema NEVER delete a required field

### Lecture 22. Schema Evolution - Hands On

* we ll do a code example to demo schema evolution
* we ll write with old schema (v1) and read with a new schema (v2) (backward compatible change)
* we ll write with new schema (v2) and read with an old schema (v1) (forward compatible change)
* in resources>avro we add 2 schema files from repo 
    * customer-v1.avsc
    * customer-v2.avsc
* v2 has 2 new fields with defaults and a field w/ defaults is deleted
* we add an evolution package
* we run `mvn clean package` to generate the classes
* in our package we add a class 'SchemaEvolutionExamples'
* we use SpecificRecord building the record from the v1 customer Class and write it to file
* in read we use the v2 class (new schema)
* then we reverse
* no issues. AWESOME!!

## Section 5: Setup and Launch Kafka

* In this course we will install kafka w/ Schema Registry + REST Proxy
* We will use a ready made docker image for this

### Lecture 27. Docker on Linux (ubuntu as an example)

* we have both docker engine and docker compose on linux
* we will start a kafka cluster using docker compose
* we will also show how to get the confluent binaries for the commands we run
* in 2-start-kafka/ the commands are in 'confluent-tools.sh' and a docker-compose.yml file
* the compose file uses the landoop fast data dev image
* the rest proxy is the same we used for kafka connect
* we put the compose file in a folder and fire up terminal going in the dir `sudo docker-compose up`
* we wont install the confluent platform locally due to size we will opt for portability with docker `sudo docker run -it --rm --net=host confluentinc/cp-schema-registry:3.3.1 bash`
* Then you can do `kafka-avro-console-consumer` or  `kafka-avro-console-producer`

## Section 6: Confluent Schema Registry and Kafka

### Lecture 33. Confluent Schema Registry

* we will now see how to store our created Avro Schemas in a central place to use in Kafka
* thats the confluent schema registry, an open source project from Confluent
* Confluent Schema Registry
    * store and retrieve schemas for Producers/Consumers
    * enforce Backward/Forward/Full compatibility on topics
    * decrease the size of the payload of data sent to Kafka
* Operations we can do against it using REST API
    * add schemas
    * retrieve schemas
    * update schemas
    * delete schema
* Our landoop image has a UI to view schemas. go to localhost:3030 => Schemas
* we can set the desired compatibility level. we set it to FULL
* then we click NEW
* we get a sample topic and schema to start with
* the convention to follow is
    * if our topic is named 'my-topic'
    * the topic for the key AVRO schema will be 'my-topic-key'
    * the topic for the value AVRO schema will be 'my-topic-value'
* in our example we set name 'customer-test-value'
* we edit the schema
```
{
  "type": "record",
  "name": "CustomerTest",
  "doc": "This is a test of the Schema Registry",
  "namespace": "com.example",
  "fields": [
    {
      "name": "first_name",
      "type": "string"
    },
    {
      "name": "age",
      "type": "int"
    },
    {
      "name": "height",
      "type": "float"
    }
  ]
}
```

* we click VALIDATE and then CREATE NEW SCHEMA
* we can then view the schema and its properties
* we select EDIT and add a field with default val to push a full compatible change on schema.
* we validate for full compatibility and the apply the change as v2 (EVOLVE SCHEMA)
* we can now see a history of schema

### Lecture 34. Kafka Avro Console Producer & Consumer

* the Avro Console Producer allows us to quickly send data to Kafka manually by specifying the schema as a n argument
* the binaries come with the Confluent Distribution of Kafka (accessible with docker or Confluent Binaries) 
* we ll see how to use the Kafka Avro Console Producer and Consumer
* all the commands for testing are in `3-schema-registry/1-kafka-avro-console-producer-consumer.sh`
* if we are not already inside the confluent container tool we run `sudo docker run --net=host -it confluentinc/cp-schema-registry:3.3.1 bash`
* we start an avro producer
    * set the kafka broker
    * topic
    * as property we pass the schema registry location (REST proxy)
    * we also pass the value.schema
```
kafka-avro-console-producer \
    --broker-list 127.0.0.1:9092 --topic test-avro \
    --property schema.registry.url=http://127.0.0.1:8081 \
    --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'
```
* in localhost:3030 we see that the above command created a topic and added a schema to the cluster. also datatype is set to avro
* we start now cping values in the console of the producer following the value.schema `{"f1": "value1"}` and see them published. we push illegal values to trigger an error `{"f2": "value4"}` and `{"f1": 1}` this breaks the producer
* kafka-avro-console-cosumer consumes a topic and a schema from the registry to decode avro
    * its a standard command by we pass as property the schema registry REST api endpoint
```
kafka-avro-console-consumer --topic test-avro \
    --bootstrap-server 127.0.0.1:9092 \
    --property schema.registry.url=http://127.0.0.1:8081 \
    --from-beginning
```
* we see the records being consumed as streings after decoding
* we will break the schema producing to the already present topic with another schema
```
kafka-avro-console-producer \
    --broker-list localhost:9092 --topic test-avro \
    --property schema.registry.url=http://127.0.0.1:8081 \
    --property value.schema='{"type":"int"}'
```
* if we pass a compatible value say '1' it breaks ant the reason is schema incompatibility
* these doesnt mean we cannot eveolve the schema. it has to be compatible with the evolution rules
```
kafka-avro-console-producer \
    --broker-list localhost:9092 --topic test-avro \
    --property schema.registry.url=http://127.0.0.1:8081 \
    --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"},{"name": "f2", "type": "int", "default": 0}]}'
```
* we can now publish using the new schema `{"f1": "evolution", "f2": 1 }`
* consumer consumes it,,,

### Lecture 35. Writing a Kafka Avro Producer in Java

* now that we have a feeling for the console producer and consumer using avro schema we can dive in Java
* we will see how to write consumers and producers for our production app
* we create anew project 'kafka-avro-v1'
* we cp properies dependencies and plugins from course repo project to the project pom.xml file. what we added:
    * enforced confluent avro and kafka version for the libs
    * we add the confluent repository
    * we add lib dependecies: apache avro, kafka client, kafka-avro serializer from confluent
    * in plugins we enforce the java version 1.8, how to dicover the sources etc
* in src we add package 'com.github.achliopa.kafka' and add a class KafkaAvroProducerV1
* in main we will create a producer startin wit the valilla kafka producer config. our key will be string. the value is avro we also se the schema regisstry endpoint
```
properties.setProperty("value.serializer", KafkaAvroSerializer.class.getName());
properties.setProperty("schema.registry.url", "http://127.0.0.1:8081");
```

* building the producer is like how we know but we need the Customer class to pass in builder. we will generate it from schema using specific record
* our schema will be customer-v1.avsc in resources. run `mvn clean package`
* we now have a class for
* we build and run our producer. for every run the metadata returned show offset increase. we confirm creation in localhost:3030
* we start a 
```
kafka-avro-console-consumer --topic customer-avro \
    --bootstrap-server localhost:9092 \
    --from-beginning \
    --property schema.registry.url=http://127.0.0.1:8081
```

### Lecture 36. Writing a Kafka Avro Consumer in Java

* now that we have written the Java AVro Producer we will write a Java Avro Consumer. we create an KafkaAvroConsumerV1 class
* we pass in kafka and avro consumer props
* we use KafkaAvroDeserializer
* we build the consumer
* we subscribe to topic and poll for messages

### Lecture 38. Writing a V2 Kafka Producer

* we create a new project v2
* we copy everything from v1 project just change class names to v2
* we mod the schema evolving it to v2 for full compatibility
* we fire up producer v1 and consumer v2. it works
* we fire up producer v2 and consumer v1. it works
* both use same topic

### Lecture 40. Summary on Compatibility Changes

* Write a forward compatible change (common)
    * update the producer to V2, this wont break the consumers
    * take the time and update consumers to V2
* Write a backward compatible change (less common)
    * update all consumers to V2, we can read V1 produced data
    * when all update update producer to V2

### Lecture 41. Kafka Schema Registry Deep Dive

* What actually happens in the Schema Registry
* the avro bytes contain schema + content (payload)
* with kafka avro serializer
    * Avro Schema => register schema if not registered already (Get ID) =>Avro Schema (Schema Registry _ schema ID (4bytes))
    * Avro Content => 2. prepend magic byte (version), 3. prepend schema ID => Avro Content (KAfka)
* deserializer does the reverse
* Schema registry externa,izes the schema 
* redices message size a lot as schema is never sent
* Now schema registry becomes a critical part of infrastructure (think redundancy)

### Lecture 42. Managing Schemas Efficiently & Section Summary

* create repo that holds the schema and geenrate the SpecificRecord classes.
* Publish that schema using CICD once deemed valid and compatible with the current schema
* in projects reference the published classes for our schema using maven e.g
* write our normal producer/consumer
* strive for FULL compatibility

## Section 7: Confluent REST Proxy

### Lecture 43. Kafka REST Proxy Introduction and Purpose

* Kafka is great for java based consumers/producers. 
* clients sometimes are lacking for other langs
* additionaly sometimes AVRO support for some langs isnt great when JSON/HTTP reqs are excellent
* Confluent developed the REST proxy to facilitate clientin other langs
* Its opensource from Confluent
* Rest Proxy  exchnages AVRO schema with Kafka Schema Registry offering JSON and HTTP API to Non-Java Producers/Consumers while it also taks to the KAfka Cluster for data
* ITs integrated with schema registry so consumers .oriducers cab read/write to avro topics
* there is a performace hit using the REST proxy instead of Kafka Native protocol and its estimated that the throuput decreases by 3-4x
* its responsibility of prod app to batch events
* Confluent REST Proxy is bundled on Docker Kafka Cluster
* In this Section
    * REST proxy calls and versions
    * Topic ops
    * Prod/cons binary
    * Prod/cons JSON
    * Prod/cons Avro
    * Deploy and scale the REST proxy

### Lecture 44. V1 vs V2 APIs

* Apache Kafka has an old consumer and old producer API that were valid up to 0.8
* After v0.8 Kafka released a new producer and consumemer API
* REST proxy v2 has support for new API 
* Making a reques to the REST proxy `application/vnd.kafka[.embedded_format].[api_version]+[serialization_format]`
    * embedded_format: json,bin,avro
    * api_verson: v2
    * serilization_format: json

### Lecture 45. Insomnia Setup (REST Client)

* we ll install Insomnia REST client just to test it (if its better than POSTMAN)
```
# Add to sources
echo "deb https://dl.bintray.com/getinsomnia/Insomnia /" \
    | sudo tee -a /etc/apt/sources.list.d/insomnia.list

# Add public key used to verify code signature
wget --quiet -O - https://insomnia.rest/keys/debian-public.key.asc \
    | sudo apt-key add -

# Refresh repository sources and install Insomnia
sudo apt-get update
sudo apt-get install insomnia
```

* all the requests are in 'rest-proxy-insomnia.json' in 4-rest-proxy
* in this folder we have the commands to create the topics in the kafka-cluster
* in the tool: insomnia=>import/export => import=>from file 
* we import the json and have all the commands available

### lecture 46. Topic Operations

* geting a list of topics (GET /topics)
* getting a specific topic (GET /topics/topic_name)
* we cannot create topics using the REST proxy we use the cli from host
```
kafka-topics.sh --create --zookeeper localhost:2181 --topic rest-binary --replication-factor 1 --partitions 1
kafka-topics.sh --create --zookeeper localhost:2181 --topic rest-json --replication-factor 1 --partitions 1
kafka-topics.sh --create --zookeeper localhost:2181 --topic rest-avro --replication-factor 1 --partitions 1
```
* we get topics with `GET http://localhost:8082/topics`
* in header we set to accept as type 'application/vnd.kafka.v2+json'
* to learn on a specific topic `http://{{ hostname  }}:8082/topics/__consumer_offsets`

### Lecture 47. Producing in Binary with the Kafka REST Proxy

* we have 3 choices with REST proxy to produce data
    * binary (raw bytes encoded in base64)
    * json (plain json)
    * avro (json encoded)
* before sending binary to REST Proxy we ened to base64 encode them
* we can test the encoding [here](http://www.utilities-online.info/base64/)
* in our example data is already encoded
* to produce binary on the REST proxy `POST http://{{ hostname  }}:8082/topics/rest-binary`
    * conent-type: application/vnd.kafka.binary.v2+json
    * accept: application/vnd.kafka.v2+json, application/vnd.kafka+json, application/json
    * json body:
```
{
  "records": [
    {
      "key": "a2V5",
      "value": "aGVsbG8gd29ybGQhISE="
    },
    {
      "value": "XCJyYW5kb206JSQh",
      "partition": 0
    },
    {
      "value": "bm8gcGFydGl0aW9ucw=="
    }
  ]
}
```

* reply with partition status
```
{
  "offsets": [
    {
      "partition": 0,
      "offset": 0,
      "error_code": null,
      "error": null
    },
    {
      "partition": 0,
      "offset": 1,
      "error_code": null,
      "error": null
    },
    {
      "partition": 0,
      "offset": 2,
      "error_code": null,
      "error": null
    }
  ],
  "key_schema_id": null,
  "value_schema_id": null
}
```

### Lecture 48. Consuming in Binary with the Kafka REST Proxy

* to consume with the REST Proxy, we first need to create a consumer in a specific consumer group
* once we open a consumer, the REST Proxy return a URL to directly hit in order to keep on consuming from the same REST Proxy instance
* if the REST Proxy shuts down, it will try to graceful close the consumers
* if the REST Proxy shuts down, it will try to gracefully close the consumers
* we can set
    * `auto.offset.reset` latest or earliest
    * `auto.commit.enable` true or false
* Steps are (for any data format)
    * create consumer
    * subscribe to a topic (or topic list)
    * get records
    * process records (in our app)
    * commit offsets (once in a whilw)
* all these are different REST API actions

### Lecture 49. Producing in JSON with the Kafka REST Proxy

* JSON data does not need to be transofrmed before being sent to the REST Proxy, as our POST request takes JSON as an input
* any kind of valid JSON can be sent there are no restrictions
* each lang has support for JSON each easy to send semi structutred data
* to produce json `POST http://{{ hostname  }}:8082/topics/rest-json`
    * content-type: application/vnd.kafka.json.v2+json
    * accept: application/vnd.kafka.v2+json, application/vnd.kafka+json, application/json
    * json: 
```
{
  "records": [
    {
      "key": "somekey",
      "value": {"foo": "bar"}
    },
    {
      "value": [ "foo", "bar" ],
      "partition": 0
    },
    {
      "value": 53.5
    }
  ]
}
```

* in this example we send key and values

### Lecture 50. Consuming in JSON with the Kafka REST Proxy

* Steps are (for any data format)
    * create consumer
    * subscribe to a topic (or topic list)
    * get records
    * process records (in our app)
    * commit offsets (once in a whilw)
* all these are different REST API actions
* JSON data are read as is
* smae like before only header changes

### Lecture 51. Producing in Avro with the Kafka REST Proxy

* the REST proxy has primary support for Avro as its directly connected to the Schema Registry
* We send the schema in JSON (stringified) and we send the Avro payload encoded in JSON
* after the first produce call, we can get a schema id to reuse in the next requests without sending the schema again to make requests smaller
* header will change as well in that case
* request `POST http://{{ hostname  }}:8082/topics/rest-avro`
    * content-type: application/vnd.kafka.avro.v2+json
    * accept: application/vnd.kafka.v2+json, application/vnd.kafka+json, application/json
    * json (other)
```
{
  "value_schema": "{\"type\": \"record\", \"name\": \"User\", \"fields\": [{\"name\": \"name\", \"type\": \"string\"}, {\"name\" :\"age\",  \"type\": [\"null\",\"int\"]}]}",
  "records": [
    {
      "value": {"name": "testUser", "age": null }
    },
    {
      "value": {"name": "testUser", "age": {"int": 25} },
      "partition": 0
    }
  ]
}
```
* as we see we send schema and record. schema is stringifies
* we get schema id and use it in our next POST
```
{
  "value_schema_id": 7,
  "records": [
    {
      "value": {"name": "newUser", "age": null }
    },
    {
      "value": {"name": "newUser", "age": {"int": 30} },
      "partition": 0
    }
  ]
}
```

### Lecture 52. Consuming in Avro with the Kafka REST Proxy

* steps are the same for consumer (diff REST API endpoints)
* avro data will get JSON encoded (like with avro-tools)

## Section 8: Annexes

### Lecture 54. Full Avro End to End: Kafka Producer + Kafka Connect + Kafka Streams

* [Medium Blogpost End-to-End RT Pipeline](https://medium.com/@stephane.maarek/how-to-use-apache-kafka-to-transform-a-batch-pipeline-into-a-real-time-one-831b48a6ad85)
* [Github Repo](https://github.com/simplesteph/medium-blog-kafka-udemy)

* Full End-to-End App
* Its a full example leveraging Avro
    * An Avro Producer for Reviews
    * 2 kafka Stream apps using Avro
    * One Kafka Connect Sink Using Avro
* code is production ready end demos the full workflow
* Udemy reviews are digested every 24h (spam dtection algo)
* UDemy Batch Architecture
    * Rest Endpoint `POST /reviews/{course-id}` -> New Review
    * NewReview -> DB [All New Reviews] -> Pull All -> Fraud detection job {runs every 24 hours} -> append -> DB [Fraudulent Reviews], DB[All Valid Reviews]
    * DB [Course Stats Alltime/Recent] -> pul -> REST Endpoint GET /course/{course-id}
    * DB[all valid revies] -> pull all -> Statistics job -> update -> DB [Course Stats]
* Envisioned Kafka Architecture
    * REST Endpoint `POST /reviews/{course-id}` -> Kafka Producer (Udemy Reviews) -> KAFKA CLUSTER [Review Topic] -> Kafka Streams Fraud Detection <->Machine Learning Model -> KAFKA CLUSTER[Fraud Topic] , KAFKA CLUSTER [Valid Reviews Topic]
    * KAFKA CLUSTER [Valid Reviews Topic] -> kafka Streams Stts App -> KAFKA CLUSTER [All time stats], KAFKA CLUSTER [Recent Stats] -> Kafka Connector JDBC Sin -> upser  -> PostgreDB -> Rest API pull stats 

### Lecture 55. Kafka REST Proxy Installation and Scaling - Overview

* Check [Kafka REST Proxy Config](https://docs.confluent.io/current/kafka-rest/config.html) for istructions
* See Kafka REST proxy section -> installation
* then check config options
* `id` we need for adding more instances 
* `bootstrap-server` to connect to kafka cluster 
* `listeners` is for the REST Proxy port
* `schema-registry.url` point to Confluent installation
* latest REST proxy is 4.0
* fire multiple instances behind a load balancer (NGINX) to scale
## Section 9: Next Steps

### Lecture 56. What's next?

* [Avro Docs](http://avro.apache.org/docs/current/spec.html)
* [Schema Registry Docs](https://docs.confluent.io/current/schema-registry/index.html)
* [rEST Proxy Docs](https://docs.confluent.io/current/kafka-rest/index.html)

* CLEAN UP `sudo docker system prune`