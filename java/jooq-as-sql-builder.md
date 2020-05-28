# Use Jooq as SQL Builder

## Maven dependencies

```
<dependency>
	<groupId>org.jooq</groupId>
	<artifactId>jooq</artifactId>
	<version>${jooq.version}</version>
</dependency>
<dependency>
	<groupId>org.jooq</groupId>
	<artifactId>jooq-meta</artifactId>
	<version>${jooq.version}</version>
</dependency>
<dependency>
	<groupId>org.jooq</groupId>
	<artifactId>jooq-codegen</artifactId>
	<version>${jooq.version}</version>
 </dependency>
```

## Jooq Maven codegen plugin

The plugin configuration in this example only keeps the table objects.

```
<build>
  <plugins>
    <plugin>
      <groupId>org.jooq</groupId>
      <artifactId>jooq-codegen-maven</artifactId>
      <version>3.13.2</version>

      <executions>
        <execution>
          <id>jooq-codegen</id>
          <phase>generate-sources</phase>
          <goals>
            <goal>generate</goal>
          </goals>
          <configuration>
              <!--<logging>WARN</logging>-->
              <!-- JDBC connection parameters -->
              <jdbc>
                <driver>org.postgresql.Driver</driver>
                <url>jdbc:postgresql://localhost/mydb</url>
                <user>user</user>
                <password>password</password>
              </jdbc>

              <!-- Generator parameters -->
              <generator>
                <database>
                  <name>org.jooq.meta.postgres.PostgresDatabase</name>
                  <includes>.*</includes>
                  <excludes>databasechange.* | pg_.* | flyway_schema_history | _st.* | st.*</excludes>
                  <inputSchema>public</inputSchema>
                </database>
                <generate>
                  <!-- A flag indicating whether Java 8's java.time types should be used by the source code generator, rather than JDBC's java.sql types. -->
                  <javaTimeTypes>true</javaTimeTypes>
                  <!-- Generate index information -->
                  <indexes>true</indexes>
                  <!-- Primary key / foreign key relations should be generated and used. This is a prerequisite for various advanced features -->
                  <relations>true</relations>
                  <!-- Generate deprecated code for backwards compatibility -->
                  <deprecated>false</deprecated>
                  <!-- Generate the {@link javax.annotation.Generated} annotation to indicate jOOQ version used for source code -->
                  <generatedAnnotation>false</generatedAnnotation>
                  <!-- Generate Sequence classes -->
                  <sequences>true</sequences>
                  <!-- Generate Key classes -->
                  <keys>true</keys>
                  <!-- Generate Table classes -->
                  <tables>true</tables>
                  <!-- Generate TableRecord classes -->
                  <records>true</records>
                  <!-- Generate POJOs -->
                  <pojos>false</pojos>
                  <!-- Generate basic equals() and hashCode() methods in POJOs -->
                  <pojosEqualsAndHashCode>true</pojosEqualsAndHashCode>
                  <!-- Generate basic toString() methods in POJOs -->
                  <pojosToString>false</pojosToString>
                  <!-- Generate immutable POJOs. -->
                  <!--<immutablePojos>true</immutablePojos>-->
                  <!-- Generate serializable POJOs -->
                  <serializablePojos>false</serializablePojos>
                  <!-- Generated interfaces to be implemented by records and/or POJOs -->
                  <interfaces>false</interfaces>
                  <!-- Turn off generation of all global object references -->
                  <globalObjectReferences>false</globalObjectReferences>
                  <!-- Turn off generation of global catalog references -->
                  <globalCatalogReferences>false</globalCatalogReferences>
                  <!-- Turn off generation of global schema references -->
                  <globalSchemaReferences>false</globalSchemaReferences>
                  <!-- Turn off generation of global table references -->
                  <globalTableReferences>false</globalTableReferences>
                  <!-- Turn off generation of global sequence references -->
                  <globalSequenceReferences>false</globalSequenceReferences>
                  <!-- Turn off generation of global UDT references -->
                  <globalUDTReferences>false</globalUDTReferences>
                  <!-- Turn off generation of global routine references -->
                  <globalRoutineReferences>false</globalRoutineReferences>
                  <!-- Turn off generation of global queue references -->
                  <globalQueueReferences>false</globalQueueReferences>
                  <!-- Turn off generation of global database link references -->
                  <globalLinkReferences>false</globalLinkReferences>
                  <!-- Turn off generation of global key references -->
                  <globalKeyReferences>false</globalKeyReferences>
                  <!-- Generate fluent setters in records, POJOs, interfaces -->
                  <fluentSetters>false</fluentSetters>
                  <routines>false</routines>
                  <udts>false</udts>
                  <queues>false</queues>
                  <records>false</records>
                  <embeddables>false</embeddables>

                </generate>
                
                <target>
                  <packageName>org.hisp.dhis.jdbc.jooq</packageName>
                  <directory>target/generated-sources/jooq</directory>
                </target>
              </generator>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Gradle plugin: <https://github.com/etiennestuder/gradle-jooq-plugin>

## Code example


```
final String sql = DSL.select(
    Users.USERS.USERID, Users.USERS.UID, Users.USERS.CODE )
    .from( Users.USERS )
    .where( Users.USERS.UID.in( ":ids" ) )
    .getSQL();
```




