---
Title: Configure F5 BIG-IP Load Balancing
Product: Gateway
Section: high-availability
DocType: Regular
---

For enterprises that want to use the [F5 BIG-IP load balancer](https://f5.com/products/big-ip), this topic provides instructions on configuring BIG-IP to support a Kaazing WebSocket Gateway server pool.

Kaazing WebSocket Gateway includes its own load balancing and clustering functionality, documented in [Configure the Gateway for High Availability](o_high_availability.md). The advantages of using Kaazing WebSocket Gateway for load balancing and clustering are:

- Designed for the load balancing of long-lived WebSocket connections.
- Does not introduce a permanent, extra network hop for the lifetime of the connection.
- You can free up your load balancer for other HTTP or Web use cases.
- You can close your inbound firewall ports using the Kaazing WebSocket Gateway Enterprise Shield feature.


Example F5 BIG-IP and Kaazing WebSocket Gateway Topology
--------------------------------------------------------

The following illustration provides a network topology example for F5 BIG-IP and Kaazing WebSocket Gateway integration. The IP addresses used by the hosts in the diagram will have been obtained via name resolution.

![](../images/f5-big-ip-topology.png)

By following the arrows you can see how a Kaazing WebSocket Gateway client connects to the Gateway and the back-end server:

1. The client initiates the WebSocket connection to F5 BIG-IP using the BIG-IP virtual server’s external, public IP address.
2. BIG-IP routes the connection to a Gateway in its server pool using its internal IP address and a load balancing configuration such as round robin.
3. All of the Gateways are configured to accept connections from the BIG-IP internal IP address. The Gateway selected by BIG-IP accepts the incoming connection. The WebSocket connection to the Gateway is now established.
4. The Gateway establishes the connection with the back-end server over TCP. Traffic may now travel back and forth from the client to the back-end server.

Components and Tools
--------------------

The following table describes the components involved in integrating F5 BIG-IP and Kaazing WebSocket Gateway.

| Components                        | Description                                                                                                                                                                                                                                                                                                                                                                                                                     |
|:----------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Kaazing WebSocket Gateway         | You can install the Gateway on servers in your internal, trusted network and configure the F5 BIG-IP load balancer to treat them as a server pool. Once you have defined the pool, you can configure how you want the Gateway traffic balanced. For example, using Round Robin.                                                                                                                                              |
| Kaazing WebSocket Gateway clients | Client applications built using Kaazing Websocket Gateway client libraries will connect to the F5 BIG-IP load balancer using the external IP address configured for the BIG-IP Virtual Server.                                                                                                                                                                                                                                  |
| F5 BIG-IP load traffic management | This guide applies to the set of local traffic management products that are part of the BIG-IP system family of products. For example, BIG-IP Local Traffic Manager and BIG-IP DNS.                                                                                                                                                                                                                                             |
| Virtual server                    | A virtual server is a traffic-management object on the BIG-IP system that is represented by an IP address and a service.                                                                                                                                                                                                                                                                                                        |
| Load balancing pool               | A BIG-IP virtual server (also known as a load balancing virtual server) directs client traffic to a load balancing pool. A load balancing pool is a logical set of devices, such as Gateways, that you group together to receive and process traffic. When you first create the virtual server, you assign an existing default pool to it. From then on, the virtual server automatically directs traffic to that default pool. |
| Load balancing method             | The default load balancing method for the BIG-IP system is Round Robin, which simply passes each new connection request to the next server in line. All other load balancing methods take server capacity and/or status into consideration.                                                                                                                                                                                     |
| External self IP address          | The external self IP address is an IP address on the BIG-IP system that is used by the Kaazing WebSocket Gateway clients when establishing WebSocket connections.                                                                                                                                                                                                                                                               |
| Internal self IP address          | The internal self IP address is an IP address on the BIG-IP system that you associate with the load balancing pool.                                                                                                                                                                                                                                                                                                             |
| Nodes                             | A node is a logical object on the BIG-IP system that identifies the IP address of a physical resource on the network, such as a Gateway. You can explicitly create a node, or you can instruct the BIG-IP system to automatically create one when you add a pool member to a load balancing pool. You will create a node for each of the servers running Kaazing WebSocket Gateway.                                             |
| Custom iRule                      | An iRule is used to manage your network traffic. You will use a custom iRule to ensure that WebSocket traffic passes through the BIG-IP system and onto the Gateways in the load balancing pool. An iRule uses syntax based on the industry-standard Tools Command Language (Tcl). It allows you to select pools based on header data and direct traffic by searching on any type of content data that you define.              |


To Deploy F5 BIG-IP with Kaazing WebSocket Gateway
--------------------------------------------------

1. Install Kaazing WebSocket Gateway on servers in your internal network. These servers will be used by BIG-IP as a server pool for maintaining client connections to Kaazing WebSocket Gateway.
2. Configure each Gateway service to listen for connections on its IP address and connect to the back-end servers in your internal network hosting the needed services. The easiest way to do this is to configure the Property Defaults on each Gateway with the IP address as the value for the host name of the Gateway (see `gateway.hostname` below) and the IP address of the back-end server the Gateway is connecting to (see `backend.hostname` below):

    ```xml
    <properties>
    <property>
    <name>gateway.hostname</name>
    <value>20.20.20.14</value>
    </property>
    <property>
    <name>backend.hostname</name>
    <value>192.0.2.11</value>
    </property>
    <property>
    <name>gateway.extras.port</name>
    <value>8001</value>
    </property>
    </properties>
    ```
    For more information, see [properties](../admin-reference/r_conf_service.html#propertiesele) and [Configure Kaazing WebSocket Gateway](../admin-reference/p_conf_files.html).

    Note the IP addresses of these servers as you will need the IP addresses when you set up Nodes in BIG-IP.

3. Configure the services on the Gateway to accept connections from BIG-IP and connect to the back-end servers in your internal network hosting the needed services. For example, here are services for a JMS connection and an AMQP connection:

    ```xml
    <!-- JMS service -->
    <service>
      <name>JMS Service</name>
      <description>JMS Service</description>
      <accept>ws://${gateway.hostname}:${gateway.extras.port}/jms</accept>

      <type>jms</type>

      <properties>
        <connection.factory.name>ConnectionFactory</connection.factory.name>
        <context.lookup.topic.format>dynamicTopics/%s</context.lookup.topic.format>
        <context.lookup.queue.format>dynamicQueues/%s</context.lookup.queue.format>
        <env.java.naming.factory.initial>org.apache.activemq.jndi.ActiveMQInitialContextFactory</env.java.naming.factory.initial>
        <env.java.naming.provider.url>tcp://${backend.hostname}:61616</env.java.naming.provider.url>
      </properties>

      <cross-site-constraint>
        <allow-origin>http://${gateway.hostname}:${gateway.extras.port}</allow-origin>
      </cross-site-constraint>
    </service>

    <!-- AMQP service -->
    <service>
      <name>AMQP Service</name>
      <description>AMQP Service</description>
      <accept>ws://${gateway.hostname}:${gateway.extras.port}/amqp</accept>
      <connect>tcp://${backend.hostname}:5672</connect>

      <type>amqp.proxy</type>

    </service>
    ```
    For more information, see [service](../admin-reference/r_conf_service.html#service) and [Configure Kaazing WebSocket Gateway](../admin-reference/p_conf_files.html).

4. Open the BIG-IP browser-based GUI. When you installed BIG-IP, a Management IP address was configured (for example, 192.168.4.245). This address is used for performing system management via a web browser. Enter that IP address into your browser to open the management console (for example, http://192.168.4.245).
5. Set up the internal network address that BIG-IP will use as its address on the internal enterprise network.
    1. In the **Main** tab, click **Network**.
    2. Click **Self IPs**.
    3. Click **Create**.
    4. In **Name**, enter the IP address (for example, 20.20.20.1).
    5. In **VLAN/Tunnel**, select **internal**.
    6. Click **Save**.
6. Enable the ports that BIG-IP will use for the internal network.
    1. In the **Main** tab, click **Network**.
    2. Click **Interfaces**. The **Interfaces** list is displayed.
    3. Click the checkbox for each interface and click **Enable**.
7. Enable the nodes connected to the internal network.
    1. In the **Main** tab, click **Network**.
    2. Click **Nodes**, and then **Nodes** List. You will create a node for each of the servers running Kaazing WebSocket Gateway.
    3. For each node, click **Create**.
    4. In **Name**, enter a name for the server running the Gateway (for example, **Gateway-1**).
    5. In **Address**, enter the IP address of one of the servers running the Gateway.
    6. Repeat these steps for the remaining servers running the Gateway.
8. Set up the external network address that BIG-IP will use as its address when connecting to the Internet and that Kaazing WebSocket Gateway clients will use when creating a WebSocket connection.
    1. In the **Main** tab, click **Network**.
    2. Click **Self IPs**.
    3. Click **Create**.
    4. In **Name**, enter the IP address that faces the external network.
    5. In **VLAN/Tunnel**, select **external**.
    6. Click **Finished**.
9. Set up the [F5 Load Balancing Pool](https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm_configuration_guide_10_0_0/ltm_pools.html).
    1. In the **Main** tab, click **Local Traffic**.
    2. Click **Pools**, and then click **Pool List**.
    3. Click **Create**.
    4. In **Name**, enter a name for the pool.
    5. In **Load Balancing Method**, select **Round Robin**. For information about other methods, see [Selecting a load balancing method in the BIG-IP](https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm_configuration_guide_10_0_0/ltm_pools.html#1215305) documentation.
    6. In **New Members**, in **Address**, select each node you added and click **Add**. All three nodes should be added in **New Members**.
    7. Click **Finished**.
10. Create a custom iRule for WebSocket traffic. You need a custom iRule for WebSocket traffic because BIG-IP might discard packets containing HTTP Upgrade headers. For more information, see [SOL14814: The BIG-IP system may drop WebSocket traffic](https://support.f5.com/kb/en-us/solutions/public/14000/800/sol14814.html).
    1. In the **Main** tab, click **Local Traffic**.
    2. Click **iRules**, and then click **New**.
    3. In **Name**, enter a name for the iRule, such as `allow-ws`.
    4. Enter the following iRule:
      ```
      when HTTP_REQUEST {
        log local0. "processing irule for http request"
        if { "websocket" eq [string tolower [HTTP::header value "Upgrade"]] } {
          HTTP::disable
          log local0. "enabling websockets and disabling http"
        }
      }
      ```
    5. Click **Finished**.
11. Set up the Virtual Server with the iRule and Load Balancing Pool.
    1. In the **Main** tab, click **Local Traffic**.
    2. Click **Virtual Servers**, and then click **Virtual Servers List**.
    3. Click **Create**.
    4. In **Name**, enter a name for the Virtual Server.
    5. In **Source**, enter `0.0.0.0/0`. This value instructs the Virtual Server to accept any address.
    6. In **Destination**, enter the IP address that clients will use to connect to the Gateway via the BIG-IP Load Balancer.
    7. In **Service Port**, select **All Ports**.
    8. In **Protocol**, select **TCP**.
    9. In **HTTP Profile**, select **http**.
    10. Under **Resources**, in **iRules**, in **Available**, click the iRule you created (for example, **allow-ws**), and click **<<** to move the iRule into the **Enabled** textbox.
    In Default Pool, click the name of the Load Balancing Pool you created.
    Click **Finished**.
12. Save the system configuration.
    1. In the **Main** tab, click **System**.
    2. Click **Archives**, and then click **Create**.
    3. In **Name**, enter a name for the system configuration.
    4. Click **Finished**.
13. Start each Gateway in the internal network.
14. Set up Kaazing WebSocket Gateway clients. Ensure that the WebSocket clients direct their WebSocket connections to the IP Address you configured as the Destination for the Virtual Server.
15. Test a public client’s WebSocket connection to a Gateway in the load balancing pool.
    1. You can use the Echo test available at [http://www.websocket.org/echo.html](http://www.websocket.org/echo.html) to connect to a Gateway in the load balancing pool.
    2. In the Echo test, in **Location**, enter the external IP address of the BIG-IP virtual server, for example, **ws://203.0.113.10:80/**.
    3. Click **Connect**. In **Log**, you should see the message **CONNECTED**. If you do not see the message, try pinging the IP address to ensure that the client can connect to that address. If the ping is successful, ensure that your BIG-IP configuration is correct, and that your Gateways are running.
    4. Click **Send**. The message is sent to a Gateway in the load balancing pool and the Echo response is returned to the client in the Log.
    5. Next, use your own Kaazing WebSocket Gateway client to connect to the back-end server hosting a service via BIG-IP and the Gateways in the load balancing pool.

See Also
--------

- For more information about using BIG-IP features, see the [BIG-IP LTM manual](https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm_configuration_guide_10_0_0.html).
- For information about WebSocket issues with BIG-IP, see [SOL14814: The BIG-IP system may drop WebSocket traffic](https://support.f5.com/kb/en-us/solutions/public/14000/800/sol14814.html).
