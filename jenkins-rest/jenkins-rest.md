---
layout: page
title: Jenkins REST
has_children: true
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

[![Stack Overflow](https://img.shields.io/badge/stack%21overflow-jenkins&#8211;rest-4183C4.svg)](https://stackoverflow.com/questions/tagged/jenkins+rest)
[![Javadoc](https://img.shields.io/badge/javadoc-javadoc.io-green)](https://javadoc.io/doc/io.github.jrestclients/jenkins-rest/latest/index.html)

# jenkins-rest

Java client is built on the top of jclouds for working with Jenkins REST API.

## Setup

Client's can be built like so:
```groovy
JenkinsClient client = JenkinsClient.builder()
.endPoint("http://127.0.0.1:8080") // Optional. Defaults to http://127.0.0.1:8080
.credentials("admin:password") // Optional.
.build();

SystemInfo systemInfo = client.api().systemApi().systemInfo();
assertTrue(systemInfo.jenkinsVersion().equals("1.642.4"));
```
      
## Latest release

Can be found in maven central like so:
```
<dependency>
  <groupId>io.github.jrestclients</groupId>
  <artifactId>jenkins-rest</artifactId>
  <version>X.Y.Z</version>
  <classifier>sources|tests|javadoc|all</classifier> (Optional)
</dependency>
```

## Documentation

* javadocs can be found via [javadoc.io](https://javadoc.io/doc/io.github.jrestclients/jenkins-rest)

## Property based setup

Client instances do NOT need to supply the endPoint or credentials as a part of instantiating the JenkinsClient object. 
Instead one can supply them through system properties, environment variables, or a combination 
of the two. System properties will be searched first and if not found, will attempt to 
query the environment.

Setting the `endpoint` can be done with any of the following (searched in order):

- `jenkins.rest.endpoint`
- `jenkinsRestEndpoint`
- `JENKINS_REST_ENDPOINT`

Setting the `credentials` can be done with any of the following (searched in order):

- `jenkins.rest.credentials`
- `jenkinsRestCredentials`
- `JENKINS_REST_CREDENTIALS`

## Credentials

jenkins-rest credentials can take 1 of 2 forms:

- Colon delimited username and password: __admin:password__ 
- Base64 encoded username and password: __YWRtaW46cGFzc3dvcmQ=__ 

## Examples

The [mock](https://github.com/jrestclients/jenkins-rest/tree/main/src/test/java/com/cdancy/jenkins/rest/features) and [live](https://github.com/jrestclients/jenkins-rest/tree/main/src/test/java/com/cdancy/jenkins/rest/features) tests provide many examples
that you can use in your own code.

## Components

- jclouds \- used as the backend for communicating with Jenkins REST API
- AutoValue \- used to create immutable value types both to and from the jenkins program
    
## Local Testing

Running mock tests can be done like so:

	./gradlew clean build mockTest
	
Running integration tests can be done like so (requires existing jenkins instance):

	./gradlew clean build integTest 

### Integration tests settings

#### Jenkins instance requirements

- a running instance accessible on http://127.0.0.1:8080 (can be changed in the gradle.properties file)
- no pre-existing jobs
- Jenkins security
  - an `admin` user (credentials used by the tests can be changed in the gradle.properties file) with `ADMIN` role (required as the tests install plugins)
  - [CSRF protection enabled](https://wiki.jenkins.io/display/JENKINS/CSRF+Protection). Not mandatory but [recommended by the Jenkins documentation](https://jenkins.io/doc/book/system-administration/security/#protect-users-of-jenkins-from-other-threats). The lib supports Jenkins instances with our without this protection (see #14)
- Plugins
  - [CloudBees Credentials](https://plugins.jenkins.io/cloudbees-credentials): otherwise an http 500 error occurs when accessing
to http://127.0.0.1:8080/job/test-folder/job/test-folder-1/ `java.lang.NoClassDefFoundError: com/cloudbees/hudson/plugins/folder/properties/FolderCredentialsProvider`
  - [OWASP Markup Formatter](https://plugins.jenkins.io/antisamy-markup-formatter) configured to use `Safe HTML`
  - [Configuration As Code](https://plugins.jenkins.io/configuration-as-code) plugin installed
  - [Cloudbees Folder](https://plugins.jenkins.io/cloudbees-folder) plugin installed

This project provides instructions to setup a [pre-configured Docker container](https://github.com/jrestclients/jenkins-rest/blob/main/src/main/docker/README.md)

#### Integration tests configuration

- jenkins url and authentication method used by the tests are defined in the `gradle.properties` file
- by default, tests use the `credentials` authentication but this can be changed to use the `token` authentication


#### Running integration tests from within your IDE

- the `integTest` gradle tasks set various System Properties
- if you don't want to use gradle as tests runner in your IDE, configure the tests with the same kind of System Properties

## Submitting a pull-request

Pull-requests can be presented to the project.

To run a pull-request, type `/test` as the only comment once you are ready to test the pull-request.

A project owner will respond as soon as time permits.

# Additional Resources

* [Jenkins REST API](https://www.jenkins.io/doc/book/using/remote-access-api/#RemoteaccessAPI-JavaAPIwrappers)
* [Apache jclouds](https://jclouds.apache.org/start/)

