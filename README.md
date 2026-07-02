# pavlau OpenRewrite recipes

For now there is only one recipe. 

This recipe is aimed at log4 and is a rewrite of org.openrewrite.java.logging.slf4j.WrapExpensiveLogStatementsInConditionals for log4j.

**Beware**, OpenRewrite framework only works on Maven and Gradle projets, it rewrites source code.

# Optimize Log4j log statements

Created from **org.openrewrite.java.logging.slf4j.WrapExpensiveLogStatementsInConditionals**

_When trace, debug and info log statements use methods for constructing log messages, those methods are called regardless of whether the log level is enabled. This recipe optimizes these statements by either wrapping them in if-statements (LOG4J 1.x+) or using JAVA8 Lambda to wrap expansive statements (only LOG4J 2.0+) to ensure expensive methods are only called when necessary._

This recipe is available under the [Moderne Source Available License](https://docs.moderne.io/licensing/moderne-source-available-license).

Run on a maven project : `mvn -U org.openrewrite.maven:rewrite-maven-plugin:run -Drewrite.recipeArtifactCoordinates=eu.pavlau.openrewrite:openrewrite-recipe-pavlau:LATEST -Drewrite.activeRecipes=log4j.eu.pavlau.openrewrite.WrapExpensiveLogStatementsInConditionals -Drewrite.options=wrapSimpleElements=true`

## Examples
##### Example 1 Log4j2 Wrap
`WrapExpensiveLogStatementsInConditionalsLog4j2Test#wrapInIsInfoEnabled`


<Tabs groupId="beforeAfter">
<TabItem value="java" label="java">


###### Before
```java
import org.apache.logging.log4j.Logger;

class A {
    void method(Logger logger) {
        logger.info("Result: {}", calculateResult());
        logger.info("This was {} and {}", doExpensiveOperation(), "bar");
    }

    String calculateResult() {
        return "result";
    }

    String doExpensiveOperation() {
        return "foo";
    }
}
```

###### After
```java
import org.apache.logging.log4j.Logger;

class A {
    void method(Logger logger) {
        if (logger.isInfoEnabled()) {
            logger.info("Result: {}", calculateResult());
            logger.info("This was {} and {}", doExpensiveOperation(), "bar");
        }
    }

    String calculateResult() {
        return "result";
    }

    String doExpensiveOperation() {
        return "foo";
    }
}
```

</TabItem>
</Tabs>

---

##### Example 2 Log4j2 Lambda
`WrapExpensiveLogStatementsInConditionalsLog4j2Test#useLambdaForSimpleMethodCalls`


<Tabs groupId="beforeAfter">
<TabItem value="java" label="java">


###### Before
```java
import org.apache.logging.log4j.Logger;

class A {
    void method(Logger logger) {
        logger.info("Value: " + computeValue());
        logger.debug("Complex: " + computeValue() + " suffix");
    }

    String computeValue() {
        return "value";
    }
}
```

###### After
```java
import org.apache.logging.log4j.Logger;

class A {
    void method(Logger logger) {
        logger.info(() -> "Value: " + computeValue());
        logger.debug(() -> "Complex: " + computeValue() + " suffix");
    }

    String computeValue() {
        return "value";
    }
}
```

</TabItem>
</Tabs>


##### Example 3 Log4j1 Wrap
`WrapExpensiveLogStatementsInConditionalsTest#logStatementsInOuterIf`


<Tabs groupId="beforeAfter">
<TabItem value="java" label="java">


###### Before
```java
import org.apache.log4j.Logger;

class A {
    void method(Logger LOG) {
        if (expensiveOp().equals("test")) {
            String s = "message";
            LOG.info("SomeString " + "some param");
            LOG.info("SomeString " + expensiveOp());
            LOG.debug("SomeString " + "some param");
            LOG.debug(expensiveOp());
            LOG.info(expensiveOp());
        }
    }

    String expensiveOp() {
        return "expensive";
    }
}
```

###### After
```java
import org.apache.log4j.Logger;

class A {
    void method(Logger LOG) {
        if (expensiveOp().equals("test")) {
            String s = "message";
            if (LOG.isInfoEnabled()) {
                LOG.info("SomeString " + "some param");
                LOG.info("SomeString " + expensiveOp());
            }
            if (LOG.isDebugEnabled()) {
                LOG.debug("SomeString " + "some param");
                LOG.debug(expensiveOp());
            }
            if (LOG.isInfoEnabled()) {
                LOG.info(expensiveOp());
            }
        }
    }

    String expensiveOp() {
        return "expensive";
    }
}
```

</TabItem>
</Tabs>

## Usage
This recipe has an optional configuration options. 
- wrapSimpleElements (false) : Enable Wrapping simple elements in lambdas instead of adding if*Enabled statements
It can be activated by adding a dependency on `org.openrewrite.recipe:rewrite-logging-frameworks` in your build file or by running a shell command (in which case no build changes are needed):

mvn -U org.openrewrite.maven:rewrite-maven-plugin:run -Drewrite.recipeArtifactCoordinates=eu.pavlau.openrewrite:openrewrite-recipe-pavlau:LATEST -Drewrite.activeRecipes=log4j.eu.pavlau.openrewrite.WrapExpensiveLogStatementsInConditionals -Drewrite.options=wrapSimpleElements=true

You can also include the launch in you maven  pom.xml file

```xml 

<project>
    ...
    <build>
        ...
        <plugins>
            ...
            <plugin>
                <groupId>org.openrewrite.maven</groupId>
                <artifactId>rewrite-maven-plugin</artifactId>
                <version>6.19.0</version>
                <configuration>
                    <exportDatatables>true</exportDatatables>
                    <activeRecipes>
                        <recipe>log4j.eu.pavlau.openrewrite.WrapExpensiveLogStatementsInConditionalseu.pavlau.openrewrite.log4j.WrapExpensiveLogStatementsInConditionals</recipe>
                    </activeRecipes>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>eu.pavlau.openrewrite</groupId>
                        <artifactId>eu.pavlau.openrewrite</artifactId>
                        <version>1.0</version>
                    </dependency>
                </dependencies>
            </plugin>
            ...
        </plugins>
        ...
    </build>
    ...
</project>
```

And run it with `mvn rewrite:run`
[Doc Maven](https://docs.openrewrite.org/reference/rewrite-maven-plugin)


## Data Tables

<Tabs groupId="data-tables">
<TabItem value="org.openrewrite.table.SourcesFileResults" label="SourcesFileResults">

### Source files that had results
**org.openrewrite.table.SourcesFileResults**

_Source files that were modified by the recipe run._

| Column Name | Description |
| ----------- | ----------- |
| Source path before the run | The source path of the file before the run. `null` when a source file was created during the run. |
| Source path after the run | A recipe may modify the source path. This is the path after the run. `null` when a source file was deleted during the run. |
| Parent of the recipe that made changes | In a hierarchical recipe, the parent of the recipe that made a change. Empty if this is the root of a hierarchy or if the recipe is not hierarchical at all. |
| Recipe that made changes | The specific recipe that made a change. |
| Estimated time saving | An estimated effort that a developer to fix manually instead of using this recipe, in unit of seconds. |
| Cycle | The recipe cycle in which the change was made. |

</TabItem>

<TabItem value="org.openrewrite.table.SearchResults" label="SearchResults">

### Source files that had search results
**org.openrewrite.table.SearchResults**

_Search results that were found during the recipe run._

| Column Name | Description |
| ----------- | ----------- |
| Source path of search result before the run | The source path of the file with the search result markers present. |
| Source path of search result after run the run | A recipe may modify the source path. This is the path after the run. `null` when a source file was deleted during the run. |
| Result | The trimmed printed tree of the LST element that the marker is attached to. |
| Description | The content of the description of the marker. |
| Recipe that added the search marker | The specific recipe that added the Search marker. |

</TabItem>

<TabItem value="org.openrewrite.table.SourcesFileErrors" label="SourcesFileErrors">

### Source files that errored on a recipe
**org.openrewrite.table.SourcesFileErrors**

_The details of all errors produced by a recipe run._

| Column Name | Description |
| ----------- | ----------- |
| Source path | The file that failed to parse. |
| Recipe that made changes | The specific recipe that made a change. |
| Stack trace | The stack trace of the failure. |

</TabItem>

<TabItem value="org.openrewrite.table.RecipeRunStats" label="RecipeRunStats">

### Recipe performance
**org.openrewrite.table.RecipeRunStats**

_Statistics used in analyzing the performance of recipes._

| Column Name | Description |
| ----------- | ----------- |
| The recipe | The recipe whose stats are being measured both individually and cumulatively. |
| Source file count | The number of source files the recipe ran over. |
| Source file changed count | The number of source files which were changed in the recipe run. Includes files created, deleted, and edited. |
| Cumulative scanning time (ns) | The total time spent across the scanning phase of this recipe. |
| Max scanning time (ns) | The max time scanning any one source file. |
| Cumulative edit time (ns) | The total time spent across the editing phase of this recipe. |
| Max edit time (ns) | The max time editing any one source file. |

</TabItem>

</Tabs>