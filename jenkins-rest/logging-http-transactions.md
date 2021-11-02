---
layout: page
title: Logging HTTP Transactions
parent: Jenkins REST
---

When writing and debugging an application that uses the `jenkins-rest` library,
it can be useful to activate logging and trace HTTP transactions.
For these cases, pass a logging framework to the `JenkinsClient` builder modules list.
For example using SLF4J, this can be done like so:

```groovy
@Grab(group='com.cdancy', module='jenkins-rest', version='0.0.11')
@Grab(group='org.apache.jclouds.driver', module='jclouds-slf4j', version='2.1.0')
@Grab(group='org.slf4j', module='slf4j-log4j12', version='1.7.25')

import com.cdancy.jenkins.rest.JenkinsClient
import org.jclouds.logging.slf4j.config.SLF4JLoggingModule

JenkinsClient client = JenkinsClient.builder()
    .modules(new SLF4JLoggingModule())
    .build()

println(client.api().systemApi().systemInfo())
```

To control the logging, provide an SLF4J12 configuration file at runtime to the application. This configuration file could contain:

```properties
log4j.rootLogger=TRACE, A1
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n
```

Pass this configuration file to the application using the `JAVA_OPTS` environment variable. For example:

```bash
$ JAVA_OPTS=-Dlog4j.configuration=file:log4j.properties path/to/app
```

If you wish to build logging into your application, put a `log4j.properties` file in its `src/main/resources` folder.
If you use the gradle build system, it will automatically include the content of `src/main/resources` to the application jar when it is published.

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

Determine the version to use for the `jclouds-slf4j` and `slf4j-log4j12` modules by looking at the `jenkins-rest` `testCompile` dependencies.

For example:

```bash
$ git clone https://github.com/cdancy/jenkins-rest
...
$ git checkout v0.0.11
$ ./gradlew dependencies --configuration testCompile | grep "jclouds-slf4j\|slf4j-api"
+--- org.apache.jclouds.driver:jclouds-slf4j:2.1.0
|    +--- org.slf4j:slf4j-api:1.7.2 -> 1.7.25
     \--- org.slf4j:slf4j-api:1.7.25

```

The versions to use for `jclouds-slf4j` is `2.1.0`, and the version to use for `slf4j-log4j12` must match the one used for `slf4j-api`, i.e. `1.7.25`.


