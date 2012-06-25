
# Pinger FMC example

This example contains a couple simple Camel routes that send messages over an ActiveMQ
queue using the fabric discovery method.  The example itself is contained in a git repo
so that it's possible to easily switch from version 1.0 to 2.0 to highlight migrating
containers from an old version to a new version.  Both routes are defined in blueprint
XML files that are packaged up into OSGi bundles.

## Modules:

* _ping-features_ - builds the features.xml that defines features for the two example camel
routes.  Note that the features.xml contains all dependent repositories, so when
deploying to fabric you don't necessarily need any parent profile other than "default"

* _ping-sender_ - the sender camel route.  The route is just a timer that sets the message
body to something and then sends to a queue.  The sender's configuration PID is:

    org.fusesource.ping.sender

which translates to:

    org.fusesource.ping.sender.properties

in FMC.  Possible settings are:

    rate - How long to wait between message sends
    message - The body of the message to send
    outQueue - What queue to send messages to


* _ping-receiver_ - the receiver camel route.  The route simply listens on an ActiveMQ
destination and logs the message body.  The receiver's configuration PID is:

    org.fusesource.ping.receiver

which translates to:

    org.fusesource.ping.receiver.properties

in FMC.  Possible settings are:

    inQueue - What queue to listen on for messages


## Building and Running:

The root pom.xml is set to deploy the build artifacts to a local fabric server instance,
i.e. to run the example create a fabric locally with

    fabric:create

And then build and deploy the artifacts by running:

    mvn clean install deploy

Changing the deploy target is a matter of editing the example's root pom.xml and using a
different hostname in the target URL:

    <distributionManagement>
      <repository>
        <id>fabric-maven-proxy</id>
        <name>FMC Maven Proxy</name>
        <!-- Change to your target host -->
        <url>http://admin:admin@localhost:8107/maven/upload/</url>
      </repository>
    </distributionManagement>

Note if you change this and plan on trying out container migration you should commit
this change so that it can be cherry-picked/merged later on when you switch to the
Version2.0 branch.

Once the artifacts are deployed you need to configure some profiles and containers using
FMC.  Here's what I usually do:

0.  Ensure you're on the "master" branch:

    git checkout master

1.  Create a container using the "mq" profile, this deploys a broker
2.  Create a profile called "pinger-base" that is a child of the "default" profile and 
configure the example's feature repository:

    mvn:org.fusesource.examples.ping/ping-features/1.0/xml/features

3.  Create two profiles called "ping-sender" and "ping-receiver" respectively and make 
sure they're children of the "pinger-base" profile.
4.  In the "ping-sender" profile, add the feature:

    ping-sender

5.  In the "ping-receiver" profile, add the feature:

    ping-receiver

6.  At this stage you can then deploy containers using either the ping-sender profile,
ping-receiver profile or both.  If you look at the broker details of the contianer running
the "mq" profile you should see messages flowing.

7.  To see migration working, run the following command in the example source:

    git checkout Version2.0

8.  You will likely need to edit/merge/cherry-pick your URL update from "master"
9.  Build everything again with a

    mvn clean install deploy

10.  Create a new version in FMC.
11.  Edit the pinger-base profile in the new version you created, and change the
URL of the features repository to:

    mvn:org.fusesource.examples.ping/ping-features/2.0/xml/features

12.  Use the "Migrate Container" button to move ping-sender and ping-receiver containers
to the new version.  The routes should start using a different queue.

