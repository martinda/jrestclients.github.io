---
layout: page
title: Logging HTTP Transactions
parent: Common Features
---

When writing and debugging an application that uses one of the rest libraries,
such as the `jenkins-rest` library,
it can be useful to activate logging and trace HTTP transactions.
For these cases, pass a logging framework to the client builder modules list.
For example using SLF4J, this can be done like so:

```groovy
@Grab(group='io.github.jrestclients', module='jenkins-rest', version='0.0.31')
@Grab(group='org.apache.jclouds.driver', module='jclouds-slf4j', version='2.3.0')
@Grab("org.slf4j:slf4j-api:1.7.32")
@Grab("org.slf4j:slf4j-simple:1.7.32")

import com.cdancy.jenkins.rest.JenkinsClient
import org.jclouds.logging.slf4j.config.SLF4JLoggingModule

JenkinsClient client = JenkinsClient.builder()
    .modules(new SLF4JLoggingModule())
    .build()

println(client.api().systemApi().systemInfo())
```

Save the above code in a file called `jenkins-version.groovy`.

To control the logging, set the `org.slf4j.simpleLogger.defaultLevel`
(see the [slf4j-simple documentation](http://www.slf4j.org/api/org/slf4j/impl/SimpleLogger.html) for the details).

```bash
$ JAVA_OPTS=-Dorg.slf4j.simpleLogger.defaultLogLevel=debug groovy jenkins-version.groovy
```

Alternatively, you can create a file called `simplelogger.properties` residing beside the `jenkins-version.groovy`:

```properties
org.sl4fj.simpleLogger.defaultLevel=debug
```

The `simplelogger.properties` file will be loaded automatically when you run:

```bash
# Loads simplelogger.properties from the current directory
groovy jenkins-version.groovy
```

## Conditional logging

The logging module can be passed based on some run time condition as well:

```groovy
JenkinsClient.Builder cb = JenkinsClient.builder()
if (someCondition) {
    cb = cb.modules(new SLF4JLoggingModule())
}
JenkinsClient jc = cb.build()
println(jc.api().systemApi().systemInfo())
```

### Determining logger versions

Determine the version to use for the `jclouds-slf4j` and `slf4j-log4j12` modules by looking at the `jenkins-rest` `testRuntime` dependencies.

For example:

```bash
$ git clone https://github.com/jrestclients/jenkins-rest
...
$ git checkout 0.0.31
$ ./gradlew dependencies --configuration testRuntimeClasspath | grep "slf4j"
+--- org.apache.jclouds.driver:jclouds-slf4j:2.3.0
|    +--- org.slf4j:slf4j-api:1.7.2 -> 1.7.25
     \--- org.slf4j:slf4j-api:1.7.25

```

The versions to use for `jclouds-slf4j` is `2.3.0`,
and the version to use for `slf4j-log4j12` must match the one used for `slf4j-api`, i.e. `1.7.25`.


