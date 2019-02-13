# OpenShift - Jenkins Build Pipeline for Gradle/Grails Apps

## Why Use Gradle?

a co-worker of mine pointed me towards [this article](https://rule1.quora.com/Use-Maven-Not-Gradle) when I asked if they had tried running Gradle builds on OpenShift.  Although it is enlightening, it doesn't really help if you already have a Gradle app you want to build and you don't want to convert it to Maven (well... not right away, anyway).

## Alternative to Jenkins Agent

If you're familiar with the concept of [source-to-image](https://docs.okd.io/latest/architecture/core_concepts/builds_and_image_streams.html#source-build)(s2i), an alternative to setting up a Gradle Jenkins agent would be to use the `fabric8/s2i-java` s2i base image to build your app, as it is capable of building Grails apps.  Using the `oc` cli tool, you can invoke it like this:

```
$ oc new-app fabric8/s2i-java:3.0-java8~https://github.com/grails-samples/music
```

Voila!  You're done!  This is great, but sometimes you want a bit more process in your pipeline, so you choose to go the Jenkins route.

## Jenkins Gradle Agent

Thankfully, the [Red Hat Community of Practice](https://github.com/redhat-cop) Github org has you covered with this [Jenkins Gradle Slave](https://github.com/redhat-cop/containers-quickstarts/tree/master/jenkins-slaves/jenkins-slave-gradle).

To build this slave in your OpenShift environment, simply run the following command in the project where your Jenkins master instance lives:

```
$ oc process -f https://raw.githubusercontent.com/redhat-cop/containers-quickstarts/master/jenkins-slaves/.openshift/templates/jenkins-slave-generic-template.yml \
    -p NAME=jenkins-slave-gradle \
    -p SOURCE_CONTEXT_DIR=jenkins-slaves/jenkins-slave-gradle \
    | oc create -f -
```

Ok!  When that build completes, you will have a new [ImageStream](https://docs.okd.io/latest/architecture/core_concepts/builds_and_image_streams.html#image-streams) that is labeled with `role=jenkins-slave` so that Jenkins master will automatically be able to pick up and use this new agent type.

*NOTE:* If Jenkins is already running, you will have to restart Jenkins for it to become aware of the new agent.

## Jenkins Pipeline

If you've followed the steps above, then you're ready for some Jeknins pipeline code.  In order to build your Gradle app, use a `jenkins-slave-gradle` node, like so:


```
    node("jenkins-slave-gradle") {
        stage("Checkout") {
            git url: "${gitSourceUrl}", branch: "${gitSourceRef}"
        }
        stage("Build JAR") {
            sh "gradle build"
        }
    }
```

The end!