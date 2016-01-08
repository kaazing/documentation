---
Title: Identity Logging
Product: Gateway
Section: Management
DocType: Regular
---

Kaazing WebSocket Gateway identity logging identifies clients that have signed in using a Login Module as part of a Single Sign On (SSO) interaction. Once a client is signed in successfully, the Gateway logs all the network activity for that client across TCP, HTTP(S), and WebSocket (WS and WSS) connections. For each client connection, the logs display the client IP and client identity, enabling you to determine which protocol connection for each client succeeded or failed.

Configure the Log4j File
------------------------

A WebSocket connection with the Gateway will have multiple transports associated with it, depending on the connection scenario. For example, a secure connection to the Gateway will include the TLS transport. To display specific transports in the log, ensure that the Log4j file for the Gateway, located in `GATEWAY_HOME/conf/log4j-config.xml`, has the loggers included, for example:

```xml
<logger name="transport.tcp">
    <level value="info"/>
    <appender-ref ref="AccessFile"/>
</logger>

<logger name="transport.http">
    <level value="info"/>
    <appender-ref ref="AccessFile"/>
</logger>

<logger name="transport.wsn">
    <level value="info"/>
    <appender-ref ref="AccessFile"/>
</logger>

<logger name="transport.wseb">
    <level value="info"/>
    <appender-ref ref="AccessFile"/>
</logger>
```

You can add loggers for other transports as needed.

Log File Syntax for Identity Logging
------------------------------------

When looking at the log file, you can identify all of the log information for each user by locating the client IP address and then locating each log item that includes that address. Here is an example of a log that includes identity logging.

The first line is an opened TCP connection from client 192.0.2.1. It identifies the IP address of the client and the IP address of the server hosting the Gateway, 198.51.100.1:

`INFO [tcp#1 192.0.2.1:52882] OPENED: (#00000001: kaazing niosocketchannel, server, /192.0.2.1:52882 => /198.51.100.1:8080)`

The next line is an open HTTP connection from the same client and it includes the authorized name of the client, johnSmith and the URI used to create the HTTP connection with the Gateway:

`INFO [http#2 johnSmith 192.0.2.1:52882] OPENED: (#00000002: kzg http, server, http://example.com:8080/jms?.kl=Y => http://example.com:8080/jms)`

The next line is the open, native WebSocket connection to the JMS broker from the same client and it includes the authorized name of the client, johnSmith and the URI used to create the WebSocket connection with the Gateway:

`INFO  [wsn#3 johnSmith 192.0.2.1:52882] OPENED: (#00000004: kzg wsn, server, ws://example.com:8001/jms => wsn://example.com:8001/jms)`

The next line is the second WebSocket connection for the user. The second connection is used when the Kaazing WebSocket Gateway extension handshake is used:

`INFO  [wsn#4 johnSmith 192.0.2.1:52882] OPENED: (#00000004: kzg wsn, server, ws://example.com:8001/jms => wsn://example.com:8001/jms)`

The next line is a closed HTTP connection. The HTTP connection with the Gateway is now closed because the WebSocket handshake has taken place and the client and Gateway are connected over WebSocket:

`INFO [http#2 johnSmith 192.0.2.1:52882] CLOSED: (#00000002: kzg http, server, http://example.com:8001/jms?.kl=Y => http://example.com:8001/jms)`

The next line shows the Kaazing WebSocket Gateway extension handshake used for proprietary extensions:

`INFO  [ws://192.0.2.1/jms x-kaazing-handshake] [http://192.0.2.1/jms?.kl=Y ws/rfc6455]`

Transport and Service Client Identifiers
----------------------------------------

The Gateway logs the identity associated with each transport level and service. For example, when a user is authorized by the Gateway via a login module, a record is written to the log for the HTTP transport connection and includes the username associated with the security token presented to the Gateway during the sign in process.

Higher layer transports and services inherit client information from the layer beneath. For example, HTTP and TLS inherit the client identity from TCP, which is a remote IP address and port. WebSocket and protocol transports like JMS inherit client identity from HTTP, which is both the IP address or, if available, the authenticated client identifier, such as the username associated with the security token.

If a transport layer does not specify its own identity resolver, then it inherits the identity resolver from the layer below. For example, if the identity resolver for TCP returns a string like `74.14.20.137:58392`, then each layer above that will also show that string as the authorized entity in the log lines, unless overridden by a new identity resolver.

A description of each transport and service log record is listed in the following table.

### Transport and Service Identifiers Table

| Transport or Service | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|:---------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TCP and UDP          | The Gateway logs the remote address and [ephemeral port](https://en.wikipedia.org/wiki/Ephemeral_port) of the client, for example, `192.0.2.1:52882`. Example:  `INFO [tcp#1 192.0.2.1:52882] OPENED: (#00000001: kaazing niosocketchannel, server, /192.0.2.1:52882 => /198.51.100.1:8080)`                                                                                                                                                                                                                                                                                                                                                |
| TLS                  | The Gateway configuration file uses the `ssl.verify-client` child element of `accept-options` or `connect-options` to specify whether or not to use client certificates. The value can be set to required, optional, or none (the default).  Currently, TLS identity logging is not implemented for TLS. The client identity for this layer is inherited from below.                                                                                                                                                                                                                                                                        |
| SOCKS                | Authorization is not implemented at the SOCKS layer by the Gateway. The client identity for this layer is inherited from below.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| HTTP                 | The client identity at the HTTP transport layer is a user identifier, such as a username. This assists administrators in identifying their users in the log.  Example:  `INFO [http#2 johnSmith 192.0.2.1:52882] OPENED: (#00000002: kzg http, server, http://example.com:8080/jms?.kl=Y => http://example.com:8080/jms)`.  For the HTTP layer, an administrator may override the default client identity by specifying a Principal class that contains an identity string. For more information, see [Overriding Client Identity using a Principal Class](#overriding-client-identity-using-a-principal-class).                            |
| WebSocket            | Authorization is not implemented at the WebSocket layer as the handshake is done in HTTP. The WebSocket layer inherits client identity from the HTTP layer.  Example:  `INFO  [wsn#3 johnSmith 192.0.2.1:52882] OPENED: (#00000004: kzg wsn, server, ws://example.com:8001/jms => wsn://example.com:8001/jms)`                                                                                                                                                                                                                                                                                                                              |
| JMS                  | If a [JmsUserIdentityResolver](http://developer.kaazing.com/documentation/jms/4.0/apidoc/server/jms/server/spi/index.html?com/kaazing/gateway/jms/server/spi/security/JmsUserIdentityResolver.html) is specified, then it is used as the identity string when logging. The JmsUserIdentityResolver is responsible for resolving the JMS user identity from the authenticated Subject.  If no JmsUserIdentityResolver is specified, then logging inherits from the layer below.                                                                                                                                                              |
| AMQP                 | If an [AmqpPrincipal](http://developer.kaazing.com/documentation/amqp/4.0/apidoc/server/amqp/server/spi/com/kaazing/gateway/amqp/server/spi/AmqpPrincipal.html) class is specified,  then it is used for the client identity string when logging. AmqpPrincipal is used to inject credentials into AMQP 0.9.1 messages. To implement single-sign-on, developers can set up AmqpPrincipal in the Subject their existing LoginModule implementation.  For more information, see [Promote User Identity into the AMQP Protocol](../security/p_aaa_userid_promo.html).  If no AmqpPrincipal is set, then logging inherits from the layer below. |


Overriding Client Identity in HTTP using a Principal Class
----------------------------------------------------------

The HTTP layer allows you to override the default client identifier by specifying a Principal class that contains the identity string. To specify the Principal class, use the existing `<user-principal-class>` setting which is part of the `<realm>` element.

The `<user-principal-class>` setting contains the name of the class that represents a user principal. When a principal of this class is authorized, a notification of such is sent through the management service. If no management service is configured, then this element is ignored. The class must implement `java.security.Principal` (see [http://docs.oracle.com/javase/7/docs/api/java/security/Principal.html](http://docs.oracle.com/javase/7/docs/api/java/security/Principal.html) for more information).

The `<user-principal-class>` setting informs JMX that these classes exist, and the class will appear in JConsole. You may populate the value in each login module. Any Principal class you add to your login module is available for this element, and then available to JMX.

Here is an example of the setting in the Gateway configuration:

```xml
<realm>
  <user-principal-class>
	 com.kaazing.demo.auth.MyUsernamePrincipal
  </user-principal-class>

  <name>demo</name>
  <authentication>
    <http-challenge-scheme>Application Token<http-challenge-scheme>

    <login-modules>
    ...
    </login-modules>
  </authentication>
</realm>
```

**Notes:**
  - In the case of multiple instances of `<user-principal-class>`, the first `<user-principal-class>` is used for identity logging.
  - If the value of `<user-principal-class>` cannot be converted to a valid class, an exception with the following message is thrown:

`Class <class name> could not be loaded. Please check the gateway configuration xml and confirm that user-principal-class value(s) are spelled correctly for realm <realm name>.`

You can use the login module to create the Principal, populate it, and add it to the Subject. Typically the Principal will contain the username or some other user identifier, but the administrator can choose to put any information they want into it.

Here is sample Java code from a login module that creates and populates the Principal and then adds it to the Subject.

File 1:

```java
/**
 * Store the username or user id that uniquely identifies a user.
 */

package com.kaazing.demo.auth;

public class MyUsernamePrincipal implements Principal {

    String username;

    public MyUsernamePrincipal(String username) {
        this.username = username;
    }

    @Override
    public String getName() {
        return username;
    }

}
```

File 2:

```java
package com.kaazing.demo.auth;

/**
 * LoginModule which extracts the username from the authorization token and
 * adds it as a Principal to the Subject.
 */
public class SimpleLoginModule implements LoginModule
{
  private MyUsernamePrincipal usernamePrincipal;
  ...
  public boolean login()
  throws LoginException
  {
    ...
    // Get the username from the token and validate it
    String username = ...
    if (isValid(username)) {
      this.usernamePrincipal = new MyUsernamePrincipal(username);
    }
    ...
  }

  public boolean commit()
  throws LoginException
  {
    ...
    if (this.usernamePrincipal != null)
    {
      this.subject.getPrincipals().add(this.usernamePrincipal);
    }
    ...
  }
  ...
}
```

An interesting feature of the HTTP layer is that it has access to the Forwarded header. Layers below HTTP accessing the remote IP address might be getting the IP address of an intermediary in the DMZ, such as another Gateway or a third party load balancer. In that case, all identity logging done at the lower layers might have the same IP address, but not the actual IP address of the client.

If an intermediary propagates the Forwarded header, then the header can be accessed from the LoginModule via a [callback](../security/p_geolocation_security.md#filter-remote-ip-addresses-using-a-custom-login-module). You can then choose to add the true client IP address to the Principal, perhaps concatenating it with the username.

Using Identity Logging for Troubleshooting
------------------------------------------

When a user is experiencing problems connecting to resources via the Gateway, you can use the identity logging in the log file to troubleshoot the problem for that user. Let’s take a look at a couple of examples.

###Is the User Connected?

**Problem:** A user says that he is connected, but is not receiving messages from the back-end server via the Gateway.   

**Troubleshoot:**

  0. Open the log file to identify the user. The log file is located in `GATEWAY_HOME/log/error.log`.
  1. Verify that the user is open on all transports (TCP, etc). Look for OPENED in the log. For example:

    INFO [http#2 johnSmith 192.0.2.1:52882] OPENED: (#00000002: kzg http, server, http://example.com:8080/jms?.kl=Y => http://example.com:8080/jms)

  2. If TCP and HTTP connections opened successfully, but the WebSocket connection (wsn) opened and then closed, then authorization likely failed. For example:

  ```
  INFO  [http#5 192.0.2.1:63901] OPENED: (#00000005: kzg http, server, http://example.com:8001/echo?.kl=Y => http://example.com:8001/echo)
  INFO  [wsn#6 192.0.2.1:63901] OPENED: (#00000006: kzg wsn, server, ws://example.com:8001/echo => wsn://example.com:8001/echo)
  INFO  [http#5 192.0.2.1:63901] CLOSED: (#00000005: kzg http, server, http://example.com:8001/echo?.kl=Y => http://example.com:8001/echo)
  INFO  [wsn#6 192.0.2.1:63901] CLOSED: (#00000006: kzg wsn, server, ws://example.com:8001/echo => wsn://example.com:8001/echo)
  ```

###Why Did the Connection Close?

**Problem:** A client connected, but the connection closed on them.

**Troubleshoot**: The Gateway will include a error message in the error.log, such as:

`INFO [wsn#6 192.0.2.1:63901] EXCEPTION: java.io.IOException: Network connectivity has been lost or transport was closed at other end`

Notes
-----

  - The syntax for native WebSocket connections is `wsn`.
  - The syntax for emulated WebSocket connections is `wse` and `wseb`. You might also see the suffix u or d to indicate upstream and downstream connections, such as `wseb#4u` and `wseb#5d`.  
  - The syntax for Kaazing WebSocket Gateway extensions is `httpxe`. These connections are not important for administrators to monitor.
