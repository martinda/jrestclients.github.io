---
layout: page
title: Using folders
parent: Jenkins REST
---

Jenkins jobs can be created inside folders when the [Folders](https://plugins.jenkins.io/cloudbees-folder) plugin is installed.
The folders plugin is installed as a default plugin on all recent versions of Jenkins.

The jenkins-rest library supports folders in all its `JobsApi` methods.

Working with folders is a bit awkward.
In Jenkins, folders are considered to be a type of job, and require their own XML configuration file.
This configuration must be provided to Jenkins, and hence to the jenkins-rest library, in order to create a folder.
The easiest way to obtain an XML configuration for a folder, is to ask Jenkins for it.
First manually create a folder in Jenkins, then ask for the `config.xml` file using curl.

For example:

```bash
curl "http://jenkins/job/folderName/config.xml" -o folder.xml
```
The rest of this section illustrates at a high level how to create folders using the jenkins-rest library.

First the `folder.xml` file is read into a string:
```groovy
String folderConfig = new File("folder.xml").text
```

Then a folder is created by calling the JobsApi create method:

```groovy
RequestStatus status = client.api().jobsApi().create(null, "folder-1", folderConfig)
```

The first argument to the `create` method is the name of the folder in which to create an item.
To create an item in the root folder, simply pass null as the first argument.
The second argument is the name of the item to create, and the last element is the XML configration for the item.
In this case, the item is a folder, so the XML configuration is thus that of a folder.

Folders can be nested:

```groovy
RequestedStatus status = client.api().jobsApi().create("folder-1", "folder-2", folderConfig)
```

Jobs can be created inside nested folders:

```groovy
RequestedStatus status = client.api().JobsApi().create("folder-1/folder-2", "myJob", jobConfig)
```

Where the `jobConfig` is the XML configuration string of the job.

The above will result in the following URL to `myJob`: `http://host/job/folder-1/job/folder-2/job/myJob`.
Note that the jenkins-rest library takes care of adding the `job/` prefixes to the items, and users do not need to pass them
when calling the APIs.

Here is a complete example:

```groovy
@Grab(group='io.github.jrestclients', module='jenkins-rest', version='0.0.30')

import com.cdancy.jenkins.rest.JenkinsClient
import com.cdancy.jenkins.rest.domain.common.Error
import com.cdancy.jenkins.rest.domain.common.RequestStatus

void dealWithErrors(String msg, List<Error> errors) {
    if (errors.size() > 0) {
        for (Error error : errors) {
            System.err.println("Exception: " + error.exceptionName())
        }
        throw new RuntimeException(msg)
    }
}

// Read the configuration files (previously obtained with curl or other)
String folderConfig = new File("folder.xml").text
String jobConfig = new File("job.xml").text

// Create a client instance
JenkinsClient client = JenkinsClient.builder().build()

// Create a folder
RequestStatus status = client.api().jobsApi().create(null, "folder-1", folderConfig)
dealWithErrors("Unable to create folder", status.errors())
println("Folder created successfully")

// Create a nested folder
status = client.api().jobsApi().create("folder-1", "folder-2", folderConfig)
dealWithErrors("Unable to create folder", status.errors())
println("Folder created successfully")

// Create a job inside the nested folders
status = client.api().jobsApi().create("folder-1/folder-2", "myJob", jobConfig)
dealWithErrors("Unable to create myJob", status.errors())
println("Job created successfully")

// Clean up
status = client.api().jobsApi().delete("folder-1/folder-2", "myJob")
dealWithErrors("Unable to delete myJob", status.errors())
status = client.api().jobsApi().delete("folder-1", "folder-2")
dealWithErrors("Unable to delete folder-2", status.errors())
status = client.api().jobsApi().delete(null, "folder-1")
dealWithErrors("Unable to delete folder-1", status.errors())

```

With a live instance of Jenkins running, the example can be executed like so:

```bash
export JENKINS_REST_CREDENTIALS=user:password
export JENKINS_REST_ENDPOINT=http://localhost:8080/
groovy FolderExample.groovy
```

