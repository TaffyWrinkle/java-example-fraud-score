java-example-fraud-score
========================

Example use of DeployR as a real-time, R analytics scoring engine.

## About

This example demonstrates the use of
[DeployR](http://deployr.revolutionanalytics.com) as a real-time, R
analytics scoring engine.

The example scenario mimics a real world application where employees
at a fictitious bank can request _fraud scores_ for one or more bank
account records to help detect fraudulent account activity.

This example is built using the _DeployR_ [RBroker
Framework](http://deployr.revolutionanalytics.com/dev), the simplest way
to integrate R analytics inside any Java, JavaScript or .NET
application.

This example consists of three distinct parts:

1. Example R Analytics
2. Example Server Application
3. Example Client Application

The final section of this document provides additional details regarding the
_DeployR integration_ implemented for this example.

## Example R Analytics

```
Source: analytics/*
```

This example uses an R model built to score fictitious bank account data
to help uncover fraudulent account activity. The model used is found
here:

```
analytics/fraudModel.rData
```

The model uses three variables associated with individual bank accounts:

1. The number of transactions on the account
2. The account balance
3. The credit line on the account

This example makes use of a scoring function that uses the model to help
determine the likelihood of _fraud_ on a given account based on these
data inputs. The scoring function is found here:

```
analytics/ccFraudScore.R
```

The R scripts and data models used by this example application are
bundled by default within the DeployR 7.3 repository, inside the
example-fraud-score directory owned by testuser.

However, if for any reason your DeployR repository does not contain
these files you can add them using the DeployR Repository Manager as
follows:

1. Log in as testuser into the Repository Manager
2. Create a new repository directory called example-fraud-score
3. Upload analytics/fraudModel.rData to the example-fraud-score
   directory
4. Upload analytics/ccFraudScore.R to the example-fraud-score directory


## Example Server Application


```
Source: src/main/java/*
```

The example server application represents a real world application,
capable of processing client requests for _fraud scores_ on one or more bank account records.

The example server application is implemented as a simple [Spring Boot](http://projects.spring.io/spring-boot) Java Web application.

The example server application exposes a simple REST API through which
client applications can submit _fraud scoring_ requests:

```
/fraud/score/{num_of_records}
```

The example server application handles client requests internally using the
[RBroker Framework](http://deployr.revolutionanalytics.com/dev) to run
the required R analytics that _scores_ each requested bank account record.

The _score_, generated by the R analytics and returned to the server via
the _RBroker Framework_, is then pushed by the server application, using
[STOMP-over-WebSocket](http://docs.spring.io/autorepo/docs/spring-framework/current/spring-framework-reference/html/websocket.html), to client applications that
have subscribed as listeners on:

```
/fraudengine/topic/fraud
```

## Example Client Application


```
Source: src/main/webapp/*
```

The example client application simulates a computer system made
available to employees at a fictitious bank.

The example client application is implemented as a simple, single page
[AngularJS](https://angularjs.org) application. The single page is divided into
two distinct sections:

1. RBroker Runtime Window

    This window gives the user a live view of all activity that
is taking place through the _RBroker Framework_ on behalf of the client
application. The runtime data provided indicates among other things the
live throughput performance being delivered by _DeployR_ behaving as a
real-time R analytics scoring engine.

    Note: The details presented in the RBroker Runtime Window
would not typically appear in a real-world client application. These data are presented
here simply to aid developers understand by observation how an _RBroker Framework_
integration works.

2. Bank Account Fraud Score Result Window

    This window displays the _fraud score_ results generated by the
real-time R analytics scoring engine powered by _DeployR_. The end user
can also use the _Execute_ button in this window to launch one or more
_fraud scoring_ requests.

The data for example bank account records is randomly generated by the
application. Requests are submitted by the client application through
the server application REST API.

The _score_ generated per request is returned to the client application
within a STOMP message delivered over a STOMP-WebSocket channel.

## Running the Example

A Gradle build script is provided to run the example:

```
build.gradle
```

By default, the build configuration assumes an instance of the DeployR server
is running on `localhost`. If your instance of DeployR is running at some
other IP address then please udpate the `endpoint` property in the
configuration file as appropriate.

You do not need to install Gradle before running these commands. To run
this example application on a Unix based OS, run the following shell
script:

```
gradlew run
```


To run this example application on a Windows based OS, run the following
batch file:

```
gradlew.bat run
```

Observe the console output in your terminal window to determine if the
server application has started successfully. Once started, open
the client application in your Web browser:

```
http://localhost:9080
```

## Multiple Users Running the Example

By default, the example build configuration defaults to using *testuser*
account credentials when authenticating with the DeployR server. If two
or more users intend running this example application at the same time
against the same DeployR server instance then:

1. Each user must update the `username` and `password` properties in their
build.gradle configuration file, each indicating the *username* and 
*password* for their own user account.

2. The _Access Control_ setting for each of the example R analytics file
depencencies in the DeployR repository must be changed from _Private_ access
to _Shared_ access.


## DeployR Integration Details

#### R Analytics Dependencies

DeployR-powered applications typically depend on repository-managed R analtyics
scripts, models and/or data files. See the _DeployR_ [Repository Manager](http://deployr.revolutionanalytics.com/documents/help/repo-man/) for details on how best to manage your own R analytics dependencies.

This example depends on two repository-managed files:

1. /testuser/example-fraud-score/fraudModel.rData
2. /testuser/example-fraud-score/ccFraudScore.R

Both files, an R model and scoring function respectively, are owned by _testuser_ and can be found in the _example-fraud-score_ repository-managed directory owned by _testuser_.

These example file dependencies ship, pre-deployed in the _DeployR_ repository so there is no further action for you to take in order for this example to use them.

#### RBroker Framework - Pooled Task Runtime

This examples uses the [RBroker Framework](http://deployr.revolutionanalytics.com/documents/dev/rbroker/) to integrate _DeployR_ real-time scoring capabilities inside the example server application.

Specifically, this example uses the [Pooled Task Runtime](http://deployr.revolutionanalytics.com/documents/dev/rbroker/#pooled) provided by the _RBroker Framework_.

#### RBroker Framework - Throughput

The _Resize_ button in the _RBroker Runtime Window_ in the example client application lets the end user experiment with the size of the pool of R sessions associated with the _Pooled Task Runtime_.

We recommend experimenting with the size of the pool and observing the effect this has on throughput. See the [simulations](http://deployr.revolutionanalytics.com/documents/dev/rbroker/#simulation), [profiling](http://deployr.revolutionanalytics.com/documents/dev/rbroker/#profiling) and [resource management](http://deployr.revolutionanalytics.com/documents/dev/rbroker/#gridprimer) sections of the _RBroker Framework_ tutorial for related details.
