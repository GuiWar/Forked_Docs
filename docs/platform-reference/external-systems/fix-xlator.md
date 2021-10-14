---
title: FIX Xlator
sidebar_label: FIX-Xlator
sidebar_position: 3
id: fix-xlator
---

The FIX Xlator is a plugin for the streamer and streamer client, which enables type-safe handling of FIX messages. It also gives access to a set of vital integration features, such as FIX_IN, EXECUTION_REPORT and CUSTOM_FIX.


### Enabling the FIX Xlator

To enable the plugin:
1. Add an Xlator plugin dependency to the module pom for the streamer or streamer-client.

```xml
<dependency> 
    <groupId>global.genesis</groupId>
    <artifactId>fix-xlator</artifactId>
    <version>${fix.version}</version>
</dependency>
```

2. Add the following code block at the beginning of the configuration file for the streamer or streamer-client:
  
```kotlin
plugins {
    plugin(FixXlatorPlugin)
}

fixConfiguration {
    version = fix50ref
}
```
 The `plugins` tag enables the plugin, and the `fixConfiguration` tag specifies the version to use.

3. Add a dependency on the class path for the module called {applicationName}-fix-messages. This file is generated using the [fix-codegen-plugin](/platform-reference/external-systems/fix-xlator/#fix-code-generation-plugin).

### Streamer

Enabling the plugin in a streamer definition enables the `fixStream` definition.

**Fix Streams**:
Fix Streams are enhanced stream definitions that come with a few useful defaults, enhanced fixed handling and automatic conversion to GenesisSet.

#### Types of fixStream
There are three separate types of fixStream configuration:

```kotlin
fixStream("FIX_IN") 

fixStream<ExecutionReport>("EXECUTION_REPORT")

fixStream("CUSTOM", CUSTOM_FIX.FIX_INDEX, CUSTOM_FIX.DATA, ExecutionReport::class)
```


| Name | Source Table | Stream Index | Fix Column | Stream Type |
| --- | --- | --- | --- | --- |
| FIX_IN | FIX_IN | BY_RX_SEQUENCE | FIX_DATA | Message |
| EXECUTION_REPORT | FIX_IN | BY_RX_SEQUENCE | FIX_DATA | ExecutionReport |
| CUSTOM | CUSTOM_FIX | FIX_INDEX | DATA | ExecutionReport |

When using the FIX_IN table, the appropriate index and column are selected automatically.

When specifying a message type, this will become a filter on type, so the "EXECUTION_REPORT" stream will only stream execution reports. Also, the fields can now be accessed in a type-safe manner:

```kotlin
fixStream<ExecutionReport>("EXECUTION_REPORT_VODL") {
    where { report ->
        report.lastMkt() == "VODL"
    }
}
```

### Streamer Client

The FIX-Xlator plugin enables a number of extension functions for the streamer client. These are explained below.

#### Message extension functions

`toGenesisSet`

This converts a fix message to a GenesisSet. Optional parameters, list of fields:

```kotlin
val set = message.toGenesisSet()
```

`set`

This is an operator function that allows you to set message fields straight from a GenesisSet or a value.

```kotlin
executionReport.set(executionReport.yield, set)
// or
executionReport[executionReport.yield] = set
// or
executionReport[executionReport.yield] = 1.2
```

Please note that this function will only accept joda DateTime values for any of the quickfix date types. The value will be converted appropriately internally.

`get`

This function will get any field from a quick fix message:

```kotlin
val yield = executionReport[executionReport.yield]
```

The return values are always nullable. Any quick fix date type will automatically be converted to a joda DateTime value before being returned.

#### GenesisSet extension functions

`set`

This function sets the field value in the GenesisSet. Optionally, you can specify the field name. Otherwise, the field name will be automatically converted

```kotlin
genesisSet.set(executionReport.yield)

genesisSet.set("REPORTED_YIELD", executionReport.yield)
```

`setWithDefault`

This function is similar to `set`, but enables you to specify a default value:

```kotlin
genesisSet.setWithDefault(executionReport.yield, 1.0)

genesisSet.setWithDefault("REPORTED_YIELD", executionReport.yield, 1.0)

genesisSet.setWithDefault("REPORTED_YIELD", executionReport.yield, executionReport.otherYield)
```

#### Field extension functions

`invoke`

This operator function simplifies the syntax for getting and setting field values. It will return null if the field is not set.

```kotlin
val yield = executionReport.yield()
executionReport.yield(1.0)
```

### FIX code generation plugin

Fix code generation plugin is used to generate Java sources from a QuickFIX XML dictionary.

Create a new maven module called {applicationName}-fix-messages. Add the following plugin dependency to the module pom file. 

If you need to create multiple modules, the name of each module must be:

{applicationName}-fix-{type}-messages. 

For example:

test-fix-abc-messages
test-fix-xyz-messages

```xml
<plugins>
    <plugin>
        <groupId>org.quickfixj</groupId>
        <artifactId>quickfixj-codegenerator</artifactId>
        <version>${quickfix.version}</version>

        <executions>
            <execution>
                <phase>generate-sources</phase>
                <goals>
                    <goal>generate</goal>
                </goals>
                <configuration>
                    <dictFile>${project.basedir}/src/main/resources/specs/${dictionary-file}</dictFile>
                    <decimal>true</decimal>
                    <orderedFields>true</orderedFields>
                    <packaging>global.genesis.quickfix.${fix-version}</packaging>
                    <fieldPackage>global.genesis.quickfix.${fix-version}.field</fieldPackage>
                </configuration>
            </execution>
        </executions>
    </plugin>
    <plugin>
        <groupId>${project.parent.groupId}</groupId>
        <artifactId>fix-codegen-plugin</artifactId>
        <version>${fix.version}</version>

        <executions>
            <execution>
                <phase>generate-sources</phase>
                <goals>
                    <goal>generateFix</goal>
                </goals>
                <configuration>
                    <dictionaryFile>${project.basedir}/src/main/resources/specs/${dictionary-file}</dictionaryFile>
                    <fixVersion>${fix-version}</fixVersion>
                    <packageName>global.genesis.quickfix.${fix-version}</packageName>
                </configuration>
            </execution>
        </executions>
    </plugin>
</plugins>
```