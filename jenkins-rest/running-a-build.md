---
layout: page
title: Running a build and waiting for the result
parent: Jenkins REST
---

Perhaps the most powerful use of the jenkins-rest is when the REST API
calls are combined to submit a job, track its progress and report on
the build status.

At a high level, the steps to submit a job and track it to its completion are:

1. Call the `JobsApi` `build` method. This returns an `IntegerResponse` instance with the queue id number of the submitted job
1. Poll the `QueueApi` `queueItem` method using the queue id number from the `Integer Response`. Polling returns a `QueueItem` instance which contains the state of the build in the queue. The state could be a:
    1. build cancellation before the build even starts (abort the polling, the build will never run)
    1. build pending (continue to wait or time out)
    1. build executing with a build number (stop polling the queue)
1. Poll the `JobsApi` `buildInfo` method using the `QueueItem` build number. This returns a `BuildInfo` instance. When the `result` method no longer returns null, the build is done and the `result` method returns a string representing the build status (passed, failed, etc).

Before we look at the example, we need a simple job definition:

```xml
<project>
  <actions/>
  <description>HelloWorld</description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <canRoam>true</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers/>
  <concurrentBuild>false</concurrentBuild>
  <builders/>
  <publishers/>
  <buildWrappers/>
</project>
```

Here is the example:

```groovy
@Grab(group='io.github.jrestclients', module='jenkins-rest', version='0.0.30')

import com.cdancy.jenkins.rest.JenkinsClient
import com.cdancy.jenkins.rest.domain.common.Error
import com.cdancy.jenkins.rest.domain.common.IntegerResponse
import com.cdancy.jenkins.rest.domain.common.RequestStatus
import com.cdancy.jenkins.rest.domain.job.BuildInfo
import com.cdancy.jenkins.rest.domain.queue.QueueItem

void dealWithErrors(String msg, List<Error> errors) {
    if (errors.size() > 0) {
        for (Error error : errors) {
            System.err.println("Exception: " + error.exceptionName())
        }
        throw new RuntimeException(msg)
    }
}

String config = new File("job.xml").text

// Create a client instance
JenkinsClient client = JenkinsClient.builder().build()

// Create a job
RequestStatus status = client.api().jobsApi().create(null, "myJob", config)
dealWithErrors("Unable to create job", status.errors())
println("Job successfuly created")

// Submit a build
IntegerResponse queueId = client.api().jobsApi().build(null, "myJob")
dealWithErrors("Unable to submit build", queueId.errors())
println("Build successfuly submitted with queue id: " + queueId.value())

// Poll the Queue, check for the queue item status
QueueItem queueItem = client.api().queueApi().queueItem(queueId.value())
while (true) {
    if (queueItem.cancelled()) {
        throw new RuntimeException("Queue item cancelled")
    }

    if (queueItem.executable()) {
        println("Build is executing with build number: " + queueItem.executable().number())
        break
    }

    Thread.sleep(10000)
    queueItem = client.api().queueApi().queueItem(queueId.value())
}

// Get the build info of the queue item being built and poll until it is done
BuildInfo buildInfo = client.api().jobsApi().buildInfo(null, "myJob", queueItem.executable().number())
while (buildInfo.result() == null) {
    Thread.sleep(10000)
    buildInfo = client.api().jobsApi().buildInfo(null, "myJob", queueItem.executable().number())
}
println("Build status: " + buildInfo.result())

// Clean up
status = client.api().jobsApi().delete(null, "myJob")

```

With a live instance of Jenkins running, the example can be executed like so:

```bash
export JENKINS_REST_CREDENTIALS=user:password
export JENKINS_REST_ENDPOINT=http://localhost:8080/
groovy RunningABuild.groovy
```

