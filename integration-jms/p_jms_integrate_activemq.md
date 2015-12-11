---
Title: Integrate Apache ActiveMQ
Product: Gateway
Section: integration-jms
DocType: Regular
Enterprise: True
---

In this procedure, you will learn how to integrate KAAZING Gateway and Apache ActiveMQ, a powerful open source messaging server.

Apache ActiveMQ provides a factory class in their JAR files. The Apache ActiveMQ factory class that ships in the KAAZING Gateway bundle is `org.apache.activemq.jndi.ActiveMQInitialContextFactory` in the JAR file `activemq-client-5.10.0.jar`. To use the `jms` service with a different version or instance of Apache ActiveMQ, you copy the JAR files to `GATEWAY_HOME/lib` and specify the name of the factory class in the `jms` service configuration element.

The following table lists the Java Archive (JAR) files needed by the Gateway for different versions of Apache ActiveMQ.

| Apache ActiveMQ version                                                                                     | JAR files to copy into `GATEWAY_HOME/lib`                                                              |
|-------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| [5.8.0](http://activemq.apache.org/activemq-580-release.html "Apache ActiveMQ ™ -- ActiveMQ 5.8.0 Release") | activemq-broker-5.8.0.jar, activemq-client-5.8.0.jar, hawtbuf-1.9.jar                                  |
| [5.9.1](http://activemq.apache.org/activemq-591-release.html "Apache ActiveMQ ™ -- ActiveMQ 5.9.1 Release") | activemq-broker-5.9.1.jar, activemq-client-5.9.1.jar, hawtbuf-1.9.jar                                  |
| [5.10.0](http://activemq.apache.org/activemq-5100-release.html)                                             | activemq-broker-5.10.0.jar, activemq-client-5.10.0.jar, hawtbuf-1.10.jar (shipped with the Gateway) |

To Integrate Apache ActiveMQ
----------------------------

1.  Download and install Apache ActiveMQ following the instructions in the Apache ActiveMQ [documentation](http://activemq.apache.org/version-5-getting-started.html).
2.  Download and install the Gateway as described in [Setting Up the Gateway](../about/setup-guide.md).
3.  To use a version of Apache ActiveMQ other than 5.10.0, remove the existing Apache ActiveMQ 5.10.0 files from the `GATEWAY_HOME/lib` folder: `activemq-broker-5.10.0.jar`, `activemq-client-5.10.0.jar`, `hawtbuf-1.10.jar`. Otherwise, to use a different instance of ActiveMQ 5.10.0, skip to Step 5.
4.  Copy the appropriate JAR files from `ACTIVEMQ_HOME/lib` to `GATEWAY_HOME/lib`. See the above table for the exact JAR names for your version of ActiveMQ.
    -   `activemq-broker-5.x.y.jar`
    -   `activemq-client-5.x.y.jar`

5.  Open the file `GATEWAY_HOME/conf/gateway-config.xml` in a text editor and locate the `jms` service element (search for "JMS Service").
6.  Update the `properties` element of the [jms](../admin-reference/r_conf_jms.md#jms) service to connect to the hostname for ActiveMQ (in this example, we use `activemq.example.com` to connect to ActiveMQ on a remote machine):

    ``` xml
    <service>
      <name>JMS Service</name>
      <description>JMS Service</description>
      <accept>ws://${gateway.hostname}:${gateway.extras.port}/jms</accept>

      <type>jms</type>

      <properties>
        <connection.factory.name>ConnectionFactory</connection.factory.name>
        
        <context.lookup.topic.format>dynamicTopics/%s</context.lookup.topic.format>
        <context.lookup.queue.format>dynamicQueues/%s</context.lookup.queue.format>
        <env.java.naming.factory.initial>
          org.apache.activemq.jndi.ActiveMQInitialContextFactory
        </env.java.naming.factory.initial>
        <env.java.naming.provider.url>
          tcp://activemq.example.com:61616
        </env.java.naming.provider.url>
      </properties>

      <realm-name>demo</realm-name>

      <!--
      <authorization-constraint>
        <require-role>AUTHORIZED</require-role>
      </authorization-constraint>
      -->

      <cross-site-constraint>
        <allow-origin>http://${gateway.hostname}:${gateway.extras.port}</allow-origin>
      </cross-site-constraint>
    </service>
    ```

    Notes
    -----

    -   The `context.lookup` properties in the above example make use of an ActiveMQ feature that allows special JNDI namespaces to be reserved for dynamic topics and queues (dynamic means they are not predefined in the broker but are instead created by the application as needed). If you wish to use dynamic destinations but your broker does not have this JNDI feature, then you can set the [`destination.strategy`](../admin-reference/r_conf_jms.md#destinationstrategy) property to `session` to tell the Gateway to use the JMS Session methods `createTopic()` or `createQueue()` to create topics and queues as needed.
    -   The `${gateway.hostname}` [service-default](../admin-reference/r_configure_gateway_service_defaults.md) in `tcp://${gateway.hostname}:61616` is used because in this example Apache ActiveMQ is run on the same host as the Gateway. In a production deployment, a different address will be used, such as `tcp://broker.example.com:61616`.

JMS Integration Verification
----------------------------

To verify that the JMS integration is working, perform the following steps:

1.  Start your instance of Apache ActiveMQ.
2.  Start the demo services as described in [Setting Up the Gateway](../about/setup-guide.md) (navigate to the `GATEWAY_HOME/bin` directory and run the **demo-services.start** (Linux/Mac OSX) or the **demo-services.start.bat** (Windows) script).
3.  Start the Gateway as described in [Setting Up the Gateway](../about/setup-guide.md) (in the `GATEWAY_HOME/bin` directory, run the **gateway.start** (Linux/Mac OSX) or the **gateway.start.bat** (Windows) script).
4.  In a browser, navigate to the out of the box demos at `http://localhost:8001/demo/`.
5.  Click the **JavaScript Stock Demo**.
6.  Click **Connect**. The stock feed will display a simulated live feed.

Subscription and Destination Support
--------------------------------------------------------------

KAAZING Gateway integration with Apache ActiveMQ has the following subscription and destination support:

-   Durable subscriptions are not supported.
-   Dynamic destinations are supported.
-   Static destinations are supported.

Notes
-----

-   To learn how to secure your JMS configuration, see [Secure Your JMS Configuration](../security/o_jms_secure.md).
-   The `context.lookup` properties in the above example make use of an ActiveMQ feature that allows special JNDI namespaces to be reserved for dynamic topics and queues (dynamic means they are not predefined in the broker but are instead created by the application as needed). If you wish to use dynamic destinations but your broker does not have this JNDI feature, then you can set the `destination.strategy property` of your `jms` service to `session` to tell the Gateway to use the JMS Session methods `createTopic()` or `createQueue()` to create topics and queues as needed.

    Client applications (using the KAAZING Gateway JMS client library) can then lookup/create their `Topic` and `Queue` objects using either the `session.createTopic()` method (for example, `Topic myTopic = session.createTopic("myTopic");`) or by performing a JNDI lookup using the `initialContext.lookup()` method (for example, `Topic myTopic = initialContext.lookup("/queue/myQueue");`). The `initialContext.lookup()` method is available to use after creating the JNDI `InitialContext` object when creating the JMS connection.

See Also
--------

-   For information on troubleshooting JMS integration, see [Troubleshoot JMS Integration](../integration-jms/p_jms_integrate_tshoot.md).
-   For information on troubleshooting JMS clients, see [Troubleshoot Your Clients](https://github.com/kaazing/gateway/blob/develop/doc/troubleshooting/p_dev_troubleshoot.md).
-   For general troubleshooting information, see [Troubleshoot KAAZING Gateway](https://github.com/kaazing/gateway/blob/develop/doc/troubleshooting/o_troubleshoot.md).


