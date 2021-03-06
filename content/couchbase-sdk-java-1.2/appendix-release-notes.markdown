# Appendix: Release Notes

The following sections provide release notes for individual release versions of
Couchbase Client Library Java. To browse or submit new issues, see the [Couchbase 
Java Issues Tracker](http://www.couchbase.com/issues/browse/JCBC).

<a id="couchbase-sdk-java-rn_1-2-1a"></a>

## Release Notes for Couchbase Client Library Java 1.2.1 GA (11 October 2013)

The 1.2.1 release is the first bug fix release for the 1.2 series. It fixes some
issues for newly added features in 1.2.0. Couchbase recommends that all 1.2.0 users  upgrade to the 1.2.1 release. 

**New Features and Behavior Changes in 1.2.1**

 * [SPY-135](http://www.couchbase.com/issues/browse/SPY-135): In addition to the 
   exposed synchronous CAS with expiration methods, the asynchronous
   overloaded method is also exposed.

   	```java
   	OperationFuture<CASResponse> casFuture = client.asyncCAS(key, future.getCas(), 2, value);
   ```

**Fixes in 1.2.1**

 * [SPY-137](http://www.couchbase.com/issues/browse/SPY-137): If the `ExecutorService` was not
   overridden through the factory, it is now properly shut down on `client.shutdown()`. This is
   not the case in 1.2.0, which results in some threads still running and preventing the app
   from halting completely.
 * [SPY-138](http://www.couchbase.com/issues/browse/SPY-138): The default `ExecutorService`
   can now be overridden properly through the `setListenerExecutorService(ExecutorService executorService)`
   method.

   If you override the default `ExecutorService`, you are also responsible for shutting it down properly 
   afterward. Because it can be shared across many application scopes, the CouchbaseClient cannot shut it down on your behalf.

   ```java
   CouchbaseConnectionFactoryBuilder builder = new CouchbaseConnectionFactoryBuilder();
   ExecutorService service = Executors.newFixedThreadPool(1);
   CouchbaseConnectionFactory cf = builder.buildCouchbaseConnection(/*...*/);
   ```
 * [JCBC-366](http://www.couchbase.com/issues/browse/JCBC-366): Enabling metrics is now easier and works as originally designed. Now you can just enable it through the builder and do not need to set the property also.

   ```java
   CouchbaseConnectionFactoryBuilder builder = new CouchbaseConnectionFactoryBuilder();
   builder.setEnableMetrics(MetricType.DEBUG);
   CouchbaseConnectionFactory cf = builder.buildCouchbaseConnection();
   ```

<a id="couchbase-sdk-java-rn_1-2-0a"></a>

## Release Notes for Couchbase Client Library Java 1.2.0 GA (13 September 2013)

The 1.2.0 release is the first stable release of the 1.2 series and contains new features that are backward compatible. The underlying spymemcached library, which builds the foundation for many of those features, has been upgraded to 2.10.0.


**New Features and Behavior Changes in 1.2.0**

 * The JARs are now served from the Maven Central Repository, which required a change of the `groupId` element from `couchbase` to `com.couchbase.client`. You can find a list of packages
   at <http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22couchbase-client%22>.
 * Spymemcached has been upgraded to 2.10.0, which implements most of the
   required code for the new features.
 * [JCBC-346](http://www.couchbase.com/issues/browse/JCBC-346): Couchbase Server 2.2 supports
   a new authentication mechanism (SASL CRAM-MD5), which is now supported and automatically
   detected/used by the client as well. You can still force the old one (`PLAIN`) if needed, 
   but this should not be the case in general.
 * [JCBC-347](http://www.couchbase.com/issues/browse/JCBC-347): The default `observe` poll interval
   has been decreased from 100 ms to 10 ms, which should give better performance in most cases. Also, this aligns with replication performance improvements in the Couchbase Server 2.2 release. 
 * [JCBC-138](http://www.couchbase.com/issues/browse/JCBC-138): The SDK now properly replaces the
   node list passed in on bootstrap, randomizes it, and stores it. This means that when the streaming
   connection goes down, the full last known node list can be used to reestablish the streaming
   connection. Randomizing the node list better distributes the streaming connection throughout
   the cluster, so not always the first and same node will be used.
 * [JCBC-343](http://www.couchbase.com/issues/browse/JCBC-343): In addition to blocking on the future
   or polling its status, you can now add a listener to be notified after it is completed. The
   callback is executed in a thread pool autonomously. Every future provides a `addListener()`
   method where a anonymous class that acts as a callback can be passed in. Here is an example:

   ```java
   OperationFuture<Boolean> setOp = client.set("setWithCallback", 0, "content");

   setOp.addListener(new OperationCompletionListener() {
     @Override
     public void onComplete(OperationFuture<?> f) throws Exception {
       System.out.println("I was completed!");
     }
   };
   ```
 * [JCBC-330](http://www.couchbase.com/issues/browse/JCBC-330): To make it easier to supply a new
   time-out when using the CAS operations, there are now overloaded methods provided to add a custom
   time-out. See the API documentation for details.
 * [JCBC-280](http://www.couchbase.com/issues/browse/JCBC-280): Buckets can now not only be created and deleted, but also updated directly through the SDK.
 * [JCBC-344](http://www.couchbase.com/issues/browse/JCBC-344): For easier administration and
   configuration purposes, you can now create a `CouchbaseConnectionFactory` based on system properties.
   Here is an example (the properties can be set through a file as well):

   ```java
   System.setProperty("cbclient.nodes", "http://192.168.1.1:8091/pools;192.168.1.2");
   System.setProperty("cbclient.bucket", "default");
   System.setProperty("cbclient.password", "");

   CouchbaseConnectionFactory factory = new CouchbaseConnectionFactory();
   CouchbaseClient client = new CouchbaseClient(factory);
   ```

   If you need to customize options, the `CouchbaseConnectionFactoryBuilder`  class provides a new method to
   construct a factory like this. The instantiation fails if any of the three properties are missing.

**Fixes in 1.2.0**
  	
 * [JCBC-333](http://www.couchbase.com/issues/browse/JCBC-333): When view nodes are queried and they are in a state in which they don't contain active vBuckets
   (partitions), an error might be returned. This fix ensures that the node is avoided while it is in this state (during rebalance in/out conditions).
 * [JCBC-349](http://www.couchbase.com/issues/browse/JCBC-349): Because of a regression, the
   `flush` command did not work on memcached-buckets since 1.1.9. This is now fixed.
 * [JCBC-350](http://www.couchbase.com/issues/browse/JCBC-350): A race condition when loading 
   documents from a replica has been fixed so that NPEs are no longer thrown.
 * [JCBC-351](http://www.couchbase.com/issues/browse/JCBC-351): Replica read has been hardened
   in a way that it also loads the document from the active node and silently discards replicas
   that are not in an active state. It only returns an exception if not a single replica read
   operation could be dispatched.
