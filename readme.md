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

* 