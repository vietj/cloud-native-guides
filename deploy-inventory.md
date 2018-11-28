## Deploying the Inventory Service

As explained in the introduction we need an Inventory service as part of the Cool Store Portal architecture to provide the quantity (hence the availability too) of a given product by ID.

We're going to save time by deploying the 'inventory' service instead of coding it. But before we actually deploy the service we're going to explain briefly some concepts.

![CoolStore Architecture]({% image_path coolstore-arch-inventory.png %}){:width="400px"}

#### Technology

Our **'Inventory'** service has been implemented as a Java EE Microprofile application mainly using JAX-RS (REST) and JPA (persistency). Red Hat Openshift provides several runtimes to build services using Node.js, Java, C#, etc. Given that the service is implemented using Java EE Microprofile technology we're going to use  WildFly Swarm runtime. 

#### What is WildFly Swarm?

Java EE applications are traditionally created as an `ear` or `war` archive including all dependencies and deployed in an application server. Multiple Java EE applications can and were typically deployed in the same application server. This model is well understood in the development teams and has been used over the past several years.

WildFly Swarm offers an innovative approach to packaging and running Java EE applications by packaging them with just enough of the Java EE server runtime to be able to run them directly on the JVM using `java -jar`. For more details on various approaches to packaging Java 
applications, read [this blog post](https://developers.redhat.com/blog/2017/08/24/the-skinny-on-fat-thin-hollow-and-uber).

WildFly Swarm is based on WildFly and it's compatible with MicroProfile, which is a community effort to standardize the subset of Java EE standards such as JAX-RS, CDI and JSON-P that are useful for building microservices applications.

Since WildFly Swarm is based on Java EE standards, it significantly simplifies refactoring existing Java EE applications to microservices and allows much of the existing code-base to be reused in the new services.

#### WildFly Swarm Maven Project 

The `inventory-wildfly-swarm` project has the following structure which shows the components of the WildFly Swarm project laid out in different subdirectories according to Maven best practices:

![Inventory Project]({% image_path wfswarm-inventory-project.png %}){:width="340px"}

This is a minimal Java EE project with support for JAX-RS for building RESTful services and JPA for connecting to a database. [JAX-RS](https://docs.oracle.com/javaee/7/tutorial/jaxrs.htm) is one of Java EE standards that uses Java annotations to simplify the development of RESTful web services. [Java Persistence API (JPA)](https://docs.oracle.com/javaee/7/tutorial/partpersist.htm) is another Java EE standard that provides Java developers with an object/relational mapping facility for managing relational data in Java applications.

#### Deploy Inventory on OpenShift

Now it’s time to build and deploy our service on OpenShift. 

OpenShift [Source-to-Image (S2I)]({{OPENSHIFT_DOCS_BASE}}/architecture/core_concepts/builds_and_image_streams.html#source-build) 
feature can be used to build a container image from a git repository. OpenShift S2I uses the [supported OpenJDK container image](https://access.redhat.com/documentation/en-us/red_hat_jboss_middleware_for_openshift/3/html/red_hat_java_s2i_for_openshift) to build the final container image of the 
Inventory service by building the WildFly Swam uber-jar from source code (build strategy **'Source'**), using Maven, to the OpenShift platform.

To build and deploy the Inventory service on OpenShift using the `fabric8` maven plugin, 
which is already configured in Eclipse Che, from the commands palette, click on **DEPLOY > fabric8:deploy**

![Fabric8 Deploy]({% image_path eclipse-che-commands-deploy.png %}){:width="340px"}


During the deployment, you might see that Fabric8 Maven Plugin throws an `java.util.concurrent.RejectedExecutionException` 
exception. This is due to [a bug](https://github.com/fabric8io/kubernetes-client/issues/1035) in one of Fabric8 Maven Plugin 
dependencies which is being worked on right now and will be fixed soon. You can ignore this exception for now. The deployment 
nevertheless succeeds.

![Inventory Deployed]({% image_path wfswarm-inventory-che-deployed.png %}){:width="800px"}

`fabric8:deploy` will cause the following to happen:

* The Inventory uber-jar is built using WildFly Swarm
* A container image is built on OpenShift containing the Inventory uber-jar and JDK
* All necessary objects are created within the OpenShift project to deploy the Inventory service

Once this completes, your project should be up and running. You can see the exposed DNS url for the Inventory service in the OpenShift Web Console or using OpenShift CLI:

~~~shell
$ oc get routes inventory

NAME		HOST/PORT						PATH	SERVICES	PORT	TERMINATION
catalog		catalog-{{COOLSTORE_PROJECT}}-{{OPENSHIFT_USER}}.roadshow.openshiftapps.com 		catalog		8080	None
inventory	inventory-{{COOLSTORE_PROJECT}}-{{OPENSHIFT_USER}}.roadshow.openshiftapps.com   		inventory	8080	None
~~~

Copy the route url for the Inventory service and verify the API Gateway service works using `curl`:

> The route urls in your project would be different from the ones in this lab guide! Use the one from your project.

~~~shell
$ curl http://{{INVENTORY_ROUTE_HOST}}/api/inventory/329299

{"itemId":"329299","quantity":35}
~~~

Well done! You are ready to move on to the next lab.