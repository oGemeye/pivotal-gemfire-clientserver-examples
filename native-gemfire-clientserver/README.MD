## Native (Pivotal) GemFire Client/Server Example Application

The following example demonstrates how to configure both a *GemFire* client and server cache application using *GemFire's*
[public Java API](http://data-docs-samples.cfapps.io/docs-gemfire/latest/javadocs/japi/index.html).

This example tests connectivity between the *GemFire* client and server, then runs a test case that tests a custom 
*GemFire* [CacheLoader](http://data-docs-samples.cfapps.io/docs-gemfire/latest/javadocs/japi/com/gemstone/gemfire/cache/CacheLoader.html) 
([CubesCacheLoader](https://github.com/jxblum/pivotal-gemfire-clientserver-examples/blob/master/native-gemfire-clientserver/src/main/java/io/pivotal/gemfire/main/GemFireServerApplication.java#L101-L111)) 
associated with the **Cubes** *Region* computing the cube of a number (specified as the cache "key") when requested, 
such as...

```
Region.get(4) == 64
```

Thus, the **Cubes** *Region* will return "64" when the key "4" is requested.  The *CubesCacheLoader* is only invoked
if the given key (i.e. number) is not present in the cache already.

### Prerequisites

To run this example you must install *JDK* 8 along with *Apache Maven* 3.3.x in your development environment 
and build the project.

See the [build](https://github.com/jxblum/pivotal-gemfire-clientserver-examples#build) section for more details.

### Running
 
The example *GemFire Server* ([GemFireServerApplication](https://github.com/jxblum/pivotal-gemfire-clientserver-examples/blob/master/native-gemfire-clientserver/src/main/java/io/pivotal/gemfire/main/GemFireServerApplication.java)) 
is a native *GemFire* application.  By default, the [module Maven POM file](https://github.com/jxblum/pivotal-gemfire-clientserver-examples/blob/master/native-gemfire-clientserver/pom.xml#L67-L107) 
is configured with the *Maven Exec* plugin, which can be used to run the *GemFire Server*.

From the project root directory, you can run the *GemFire Server* on the command-line with the following `mvn` command:

```
$ mvn -pl native-gemfire-clientserver exec:java
```

OOTB, this uses the module's default configuration settings (e.g. ports) to configure the *GemFire Server*.

In addition to starting a *GemFire Server*, the *GemFire* configuration also configures the server to start an embedded 
*GemFire Locator* on port `11235` along with an embedded *GemFire Manager* on port `1199` as well as a *Cache Server* 
listening for cache clients on port `12480`. 

For convenience, this project module provides a [**connect.gfsh** shell script](https://github.com/jxblum/pivotal-gemfire-clientserver-examples/blob/master/native-gemfire-clientserver/bin/connect.gfsh) 
to connect to the running *GemFire Server* in order to inspect the configuration.  **connect.gfsh** connects to
the running *GemFire Server's* embedded *Locator*, which in turn opens a connection to the embedded *Manager*, then
proceeds to `list members`, `describe` our member, `list regions`, `describe` our *Region* and then perform a `get`,
which will invoke the `CubesCacheLoader` on our initially empty cache, resulting in the cube calculation 
being performed.
                                                                                 
When `describe member` is executed, you should see that the **Cubes** *Region* exists on our server along with 
the network service endpoint of the *GemFire* `CacheServer` listening on port `12480`.  When `describe region` is 
executed, you should see the registered `CubesCacheLoader`.

The *Gfsh* shell script can be ran interactively (inside the *GemFire Shell*) or non-interactively 
(e.g. scripted in `Bash`).

Of course, to use *Gfsh*, you must have a [distribution](https://network.pivotal.io/products/pivotal-gemfire) 
of *Pivotal GemFire* installed on your system.  You can also use [Apache Geode](http://geode.incubator.apache.org/) 
in place of *Pivotal GemFire*.  Be sure to follow the [installation instructions in the User Guide](http://gemfire.docs.pivotal.io/docs-gemfire/getting_started/installation/install_intro.html).

Invoke the *Gfsh* shell script with the `run` command inside the *GemFire Shell*:

```
gfsh>run --file=/absolute/path/to/pivotal-gemfire-clientserver-examples/native-gemfire-clientserver/bin/connect.gfsh
```

Or non-interactively, in a (Bash) shell script like so:

```
!/bin/bash
...

gfsh -e "run --file=/absolute/path/to/pivotal-gemfire-clientserver-examples/native-gemfire-clientserver/bin/connect.gfsh"
```

If you are running the *GemFire Server* remotely, you will need to tweak the *connect.gfsh* shell script appropriately.

> NOTE: make sure to adjust the file system paths and file references based on your installation location.

### Test

Once the *GemFire Server* is configured and running, you can run the test with the following:

```
$ mvn -pl native-gemfire-clientserver test
```

*(optional)* Once the test runs and completes successfully, you can return to *Gfsh* and execute the `describe region` 
command to see that the contents of the cache were updated successfully by the test during execution.

```
gfsh>describe region --name=Cubes
```

*(optional)* You can also just make additional requests right from *Gfsh* to the *GemFire Server* cache **Cubes** *Region* 
in order to perform additional computations.

### Running Remotely

If the *GemFire* client and server are separated by a network, running on different systems, then the example application 
and tests allow you to configure IP addresses, ports and so on in order to establish a connection.

The following is a list of project-specific configuration System properties you can use when invoking either 
the client (test) or the server:

System Property | Client/Server | Default | Description
--------------- | ------------- | ------- | -----------
gemfire.log.level | both | config | Configures the GemFire log level.
gemfire.locator.host-port | server | localhost[11235] | Configures an embedded GemFire Locator that will run on startup.
gemfire.manager.port | server | 1199 | Configures the port on which the embedded GemFire Manager listens for JMX clients (e.g. Gfsh).
gemfire.cache.server.bind-address | server | localhost | Configures the IP address of the NIC on which the GemFire Cache Server will bind the ServerSocket.
gemfire.cache.server.hostname-for-clients | server | localhost | Configures the DNS hostname that client will use to resolve the GemFire Cache Server.
gemfire.cache.server.port | both | 12480 | Configures the port on which the embedded GemFire Cache Server listens for Gemfire cache clients. Also the port the client uses to connect to the GemFire Cache Server.
gemfire.cache.server.host | client | localhost | Configures the host that client uses to connect to the GemFire Cache Server. 

To run the *GemFire Server* listening on a different port configured with a log level of `warning`, use the following:

```
$ mvn -pl native-gemfire-clientserver exec:java -Dgemfire.cache.server.port=43210 -Dspring.gemfire.log.level=warning
```

Likewise, you need to make sure to change the port that the *GemFire* cache client tries to connect to the server on 
when running the test.  Additionally, in the remote case, you will need to specify the server's IP address:

```
$ mvn -pl native-gemfire-clientserver test -Dgemfire.cache.server.port=43210 -Dspring.gemfire.cache.server.host=10.101.21.1
```
