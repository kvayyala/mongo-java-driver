+++
date = "2015-03-19T12:53:26-04:00"
title = "SSL"
[menu.main]
  parent = "Async Connection Settings"
  identifier = "Async SSL"
  weight = 10
  pre = "<i class='fa'></i>"
+++

## TLS/SSL

The asynchronous Java driver supports TLS/SSL connections to MongoDB servers using the underlying support 
for TLS/SSL provided by the JDK.

In its default configuration, the asynchronous driver is supported on Java 8+ when TLS/SSL is enabled.  
Otherwise, applications must configure the asynchronous driver to use [Netty](http://netty.io/).


## Specify TLS/SSL and Netty Configuration

{{% note %}}
If your application requires Netty, it must explicitly add a dependency to
Netty artifacts.  The driver is currently tested against Netty 4.1.
{{% /note %}}

### Via Connection String

To configure the driver to use TLS/SSL, include the `ssl=true` option in the connection string, as in:

```java
MongoClient client = MongoClients.create("mongodb://localhost/?ssl=true");
```

To configure the driver to use use TLS/SSL with Netty, include the `ssl=true` and `streamType=netty` options in the connection string, as
in:

```java
MongoClient client = MongoClients.create("mongodb://localhost/?streamType=netty&ssl=true");
```

{{% note %}}
The streamType connection string query parameter is deprecated as of the 3.10 release and will be removed in the next major release.
{{% /note %}}

### Via `MongoClientSettings`

To specify TLS/SSL with [`MongoClientSettings`]({{< apiref "com/mongodb/MongoClientSettings.Builder.html#streamFactoryFactory-com.mongodb.connection.StreamFactoryFactory-">}}),
set the ``sslEnabled`` property to ``true`, as in

```java
MongoClient client = MongoClients.create(MongoClientSettings.builder()
        .applyToClusterSettings(builder -> builder.hosts(Arrays.asList(new ServerAddress())))
        .applyToSslSettings(builder -> builder.enabled(true))
        .build());
```

To specify TLS/SSL using Netty, set the ``sslEnabled`` property to ``true``, and the stream factory to 
[`NettyStreamFactoryFactory`]({{< apiref "com/mongodb/connection/netty/NettyStreamFactoryFactory" >}}), as in

```java
EventLoopGroup eventLoopGroup = new NioEventLoopGroup();  // make sure application shuts this down

MongoClient client = MongoClients.create(MongoClientSettings.builder()
        .applyToClusterSettings(builder -> builder.hosts(Arrays.asList(new ServerAddress())))
        .applyToSslSettings(builder -> builder.enabled(true))
        .streamFactoryFactory(NettyStreamFactoryFactory.builder()
                .eventLoopGroup(eventLoopGroup).build())
        .build());
```

By default, the Netty-based streams will use the [NioEventLoopGroup](http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html)
and Netty's [default `ByteBufAllocator`](http://netty.io/4.0/api/io/netty/buffer/ByteBufAllocator.html#DEFAULT), but these are
configurable via the [`NettyStreamFactoryFactory`]({{< apiref "com/mongodb/connection/netty/NettyStreamFactoryFactory" >}}) constructor.   

To override the default [`javax.net.ssl.SSLContext`](https://docs.oracle.com/javase/8/docs/api/javax/net/ssl/SSLContext.html) used for SSL
connections, set the `sslContext` property on the `SslSettings`, as in:

```java
MongoClient client = MongoClients.create(MongoClientSettings.builder()
        .applyToClusterSettings(builder -> builder.hosts(Arrays.asList(new ServerAddress())))
        .applyToSslSettings(builder ->
                builder.enabled(true)
                       .context(sslContext))
        .streamFactoryFactory(NettyStreamFactoryFactory.builder()
                .eventLoopGroup(eventLoopGroup).build())
        .build());
```


## Disable Hostname Verification


By default, the driver ensures that the hostname included in the
server's SSL certificate(s) matches the hostname(s) provided when
creating a `MongoClient`. However, the hostname verification
requires a Java 7 JVM, as it relies on additions introduced in Java 7
to the `javax.net.SSLParameters` class.

If your application must run on Java 6, or for some other reason you need
to disable host name verification, you must explicitly indicate this using the `invalidHostNameAllowed` property:

```java
MongoClient client = MongoClients.create(MongoClientSettings.builder()
        .applyToClusterSettings(builder -> builder.hosts(Arrays.asList(new ServerAddress())))
        .applyToSslSettings(builder -> 
                builder.enabled(true)
                       .invalidHostNameAllowed(true))
        .build());
```

Or via the connection string:

```java
MongoClient client = MongoClients.create("mongodb://localhost/?ssl=true&sslInvalidHostNameAllowed=true");
```

{{% note %}}
The streamType connection string query parameter is deprecated as of the 3.10 release and will be removed in the next major release.
{{% /note %}}

## JVM System Properties for TLS/SSL

A typical application will need to set several JVM system properties to
ensure that the client is able to validate the TLS/SSL certificate
presented by the server:

-  `javax.net.ssl.trustStore`:
      The path to a trust store containing the certificate of the
      signing authority

-  `javax.net.ssl.trustStorePassword`:
      The password to access this trust store

The trust store is typically created with the
[`keytool`](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)
command line program provided as part of the JDK. For example:

```bash
    keytool -importcert -trustcacerts -file <path to certificate authority file>
        -keystore <path to trust store> -storepass <password>
```

A typical application will also need to set several JVM system
properties to ensure that the client presents an TLS/SSL certificate to the
MongoDB server:

- `javax.net.ssl.keyStore`
      The path to a key store containing the client's TLS/SSL certificates

- `javax.net.ssl.keyStorePassword`
      The password to access this key store

The key store is typically created with the
[`keytool`](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)
or the [`openssl`](https://www.openssl.org/docs/apps/openssl.html)
command line program.

For more information on configuring a Java application for TLS/SSL, please
refer to the [`JSSE Reference Guide`](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSS
ERefGuide.html).

## JVM Support for TLS v1.1 and newer

Industry best practices recommend, and some regulations require, the use of TLS 1.1 or newer. Though no application changes are required
for the driver to make use of the newest TLS protocols, Java runtime environments prior to Java 8 started to enable TLS 1.1 only in later
updates:

* Java 7
  - Starting with
    [Update 131](http://www.oracle.com/technetwork/java/javaseproducts/documentation/javase7supportreleasenotes-1601161.html#R170_131),
    released October 8, 2016, TSL 1.1 and TLS 1.2 are enabled by default.
  - Starting with
    [Update 95](http://www.oracle.com/technetwork/java/javaseproducts/documentation/javase7supportreleasenotes-1601161.html#R170_95),
    released January 19, 2016, TLS 1.1 and TLS 1.2 can be enabled by applications via the `jdk.tls.client.protocols` system property.

* Java 6
  - Starting with
    [Update 141](http://www.oracle.com/technetwork/java/javase/documentation/overview-156328.html#R160_141), released on January 17, 2017,
    TSL 1.1 and TLS 1.2 are enabled by default.
  - Starting with
    [Update 115 b32](http://www.oracle.com/technetwork/java/javase/documentation/overview-156328.html#6u115-b32), released July 19, 2016,
    TLS 1.1 and TLS 1.2 can be enabled by applications via the `jdk.tls.client.protocols` system property.

Note that these updates are only available from Oracle via its Java SE commercial support program.  Java 7 Update 131
is available via [OpenJDK](http://openjdk.java.net/install/).
