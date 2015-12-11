---
Title: Build Flash WebSocket Clients
Product: Gateway
Section: flash
DocType: Regular
Enterprise: True
---

The following checklist provides the steps necessary to build clients to communicate with KAAZING Gateway:

| \# | Step                                                                                               | Topic or Reference                                                          |
|:---|:---------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------|
| 1  | Set up the Gateway, review the out of the box Flash and Flex demos, and build a Flex project.      | [Set Up Your Development Environment](#set-up-your-development-environment) |
| 2  | Learn how to use the WebSocket API provided by the Kaazing Flash client library in ActionScript.   | [Use the Flash WebSocket API](#use-the-flash-websocket-api)                 |
| 3  | Learn how to use the EventSource API provided by the Kaazing Flash client library in ActionScript. | [Use the Flash EventSource API](#use-the-flash-eventsource-api)             |
| 4  | Learn how to authenticate your Flash client with the Gateway.                                      | [Secure Your Flash Client](#secure-your-flash-client)                       |
| 5  | Set up logging for your client.                                                                    | [Display Logs for the Flash Client](#display-logs-for-the-flash-client)     |
| 6  | Troubleshoot the most common issue that occurs when using Kaazing Flash clients.                   | [Troubleshoot Your Flash Client](#troubleshoot-your-flash-client)           |

Introduction
------------

KAAZING Gateway Flash client library enables you to use HTML5 Communication protocols (for example, WebSocket and Server-Sent Events) in new or existing web applications. For example, a new application could receive streaming financial data from a back-end server using WebSocket or ByteSocket, or a new Flash client could receive streaming news data through Server-Sent Events. The following figure shows a high-level overview of the architecture:

![Flash client architecture overview](images/f-html5-flash-client2-web.jpg)

**Figure: Flash client architecture overview**

This document provides information for developing a Flash application developer with WebSocket.

### About the KAAZING Gateway Flash Client Library

The KAAZING Gateway client libraries include a Flash client library enabling the use of ActionScript and HTML5 Communication protocols in your Flash applications. As a Flash application developer, you can use this API to use WebSocket to communicate with your back-end server.

The Flash client library is useful to developers familiar with Flash. For a description of the methods currently supported by the KAAZING Gateway Flash client library, see the [ActionScript Client API](http://developer.kaazing.com/documentation/5.0/apidoc/client/flash/gateway/index.html) documentation.

### About Adobe Flash and Flex

The Adobe Flex SDK is used for the development and deployment of Flash applications (applications relying on the Flash Player or Flash plugin for display in the browser). Typically, Flash applications are used to create rich presentation layers. The Adobe Flex SDK includes a set of user interface components for building a rich Internet application (RIA), such as buttons, data grids, text controls, and trees. The Adobe Flex platform consists of the following components:

-   The MXML declarative user interface language
-   The ActionScript scripting language
-   The Adobe Flex compiler
-   Runtime services

Flash is a compiled language. The Adobe Flex compiler is used to generate Shockwave Flash (SWF) files and Shockwave Component (SWC) files from source ActionScript and MXML files.

**Note**: You can use Flex SDK 3.5 or higher with Kaazing's Flash client libraries.

Set Up Your Development Environment
===================================

In this procedure, you will learn how to set up your Flash development environment to develop Kaazing Flash clients.

To Set Up Your Development Environment
--------------------------------------

1.  Download and install KAAZING Gateway as described in [Setting Up KAAZING Gateway](https://github.com/kaazing/gateway/blob/develop/doc/about/setup-guide.md).
2.  In this how-to, you will use Adobe Flash Builder (formerly known as Adobe Flex Builder) to build a Flash client. You can download Adobe Flash Builder from <http://opensource.adobe.com/wiki/display/flexsdk/Flex+SDK>.
3.  Take a look at the out of the box demo built with the Flash client API, which is part of the KAAZING Gateway bundle.
4.  Start the Gateway as described in [Setting Up KAAZING Gateway](https://github.com/kaazing/gateway/blob/develop/doc/about/setup-guide.md).
5.  In a browser, navigate to the out of the box demos at `http://localhost:8001/demo/`.
6.  Click **Flash & Flex Demos**. To run the out of the box Flash & Flex Client demos, you must have Flash Player 10 or higher installed in your browser.
7.  Build a simple Flex project to use with the Kaazing libraries.
    1.  In Flash Builder, create a Flash project by choosing **File \> New \> Flex Project**.
    2.  In the Project Name field, type a name for your project.
    3.  Choose a location for the project.
    4.  Under Application Type, choose **Web**.
    5.  Under Flex SDK version, choose the Flex SDK version you wish to use. Kaazing's Flash client libraries support Flex SDK 3.5 and higher.
    6.  Click **Next**, then click **Next**.
    7.  On the Build Paths page, ensure the Library path is active. Here, you will add the SWC library from *KAAZING Gateway*.
    8.  Next to Component set, ensure **MX only** is selected.
    9.  Click **Add SWC...**, then click **Browse**.
    10. Navigate to` GATEWAY_HOME/lib/client/flash`, then choose **org.kaazing.gateway.client.swc**.
    11. Click **Open**, then click **OK**.
    12. Click **Finish**. The project files display in the Package Explorer. Now your project is created and you can start building Flash clients.

Using the KAAZING Gateway Flash Client Library
----------------------------------------------

A Flash client is an application or process that creates and sends and/or receives messages. For your Flash application to use WebSocket when communicating with your back-end server through the Gateway, you'll need to use the Flash client API to build a Flash client that you can then add to your Flash application.

Use the Flash WebSocket API
===========================

This procedure describes how you can use the `WebSocket` API, provided by the Kaazing Flash client library, in ActionScript. This API allows you to take advantage of the WebSocket standard as described in the [HTML5 specification](http://dev.w3.org/html5/spec/Overview.html#network). You can create a Flash application that uses the Kaazing Flash client library to interact directly with a back-end server. The support for WebSocket is provided by the `WebSocket` class and its supporting classes.

The following steps show you how to use the `WebSocket` API in an existing Flash application. This example highlights some of the most commonly used `WebSocket` methods and is not meant to be an end-to-end tutorial. Refer to the [ActionScript WebSocket API](http://developer.kaazing.com/documentation/5.0/apidoc/client/flash/gateway/com/kaazing/gateway/client/html5/WebSocket.html) documentation for a complete description of all the available methods.

To Use the WebSocket API
------------------------

1.  Add the necessary import statements:

    ``` as
    import org.kaazing.gateway.client.md5.MessageEvent
    import org.kaazing.gateway.client.md5.WebSocket
    ```

2.  Create a new WebSocket object:

    ``` as
    var webSocket:WebSocket = new WebSocket(url)
    ```

3.  Add event-handlers to the `WebSocket` object to listen for `WebSocket` events, as shown in the following example. The `WebSocket` object has three methods: the `onopen` method is called when a WebSocket connection is established, the `onmessage` method is called when messages are received, and the `onclose` method is called when the WebSocket connection is closed.

    ``` as
    webSocket.onopen = openHandler
    webSocket.onmessage = readHandler
    webSocket.onclose = closeHandler   
    ```

4.  Define the three functions on the WebSocket object:

    ``` as
    private function openHandler(ev:Event):void {    
        trace("CONNECTED\n");
    }

    private function readHandler(ev:Event):void {
        var msg:String = ev.data
        trace("MESSAGE: "+msg+"\n");
    }

    private function closeHandler(ev:Event):void {
        trace("CLOSED\n");
    }

    ```

    After the eventHandlers are set, the WebSocket constructor causes the WebSocket to connect to the back-end server.

5.  While the WebSocket connection is open (that is, *after* the `onopen` event-handler is called and *before* the `onclose` event-handler is called), you can use the `send` method to send text-only messages, as shown in the following example.

    `webSocket.send("Hello World!")`

Notes
-----

-   To view sample source code using the WebSocket API, see the `ws.mxml` file located in `GATEWAY_HOME/demo/flash/src/gateway`.

Use the Flash EventSource API
=============================

This procedure describes how you can use the `EventSource` API provided by the Kaazing Flash client library. This API allows you to take advantage of the Server-Sent Events standard as described in the [HTML5 specification](http://www.w3.org/html/wg/html5/#server-sent-events). For example, you can create a Flash client that uses the Kaazing Flash client library to receive streaming data from a news feed for streaming financial data. The support for Server-Sent Events is provided by the `EventSource` class and its supporting classes.

The following steps show you how to use the `EventSource` API. This example highlights some of the most commonly used `EventSource` methods and is not meant to be an end-to-end tutorial. Refer to the [ActionScript EventSource API](http://developer.kaazing.com/documentation/5.0/apidoc/client/flash/gateway/com/kaazing/gateway/client/html5/EventSource.html) documentation for a complete description of all the available methods.

To Use the EventSource API
--------------------------

1.  Add the necessary import statements:

    ``` as
    import org.kaazing.gateway.client.md5.MessageEvent
    import org.kaazing.gateway.client.md5.EventSource
    ```

2.  Create a new EventSource object:

    ``` as
    var es:EventSource = new EventSource(location.text)
    ```

3.  Add event handlers to the `EventSource` object to listen for `EventSource` events, as shown in the following example. The `EventSource` object provides mechanisms to attach event-handlers that are invoked when the corresponding event is raised. The `EventSource` object provides support for registering event handlers for specific events. The `onopen` method is called when a Server-Sent Events connection is established, the `onmessage` method is called when messages are received, and the `onerror` method is called when errors occur.

    ``` as
    eventSource.onopen = openHandler
    eventSource.onmessage = readHandler
    eventSource.onError = errorHandler   
    ```

4.  Define the three functions that serve as event handlers:

    ``` as
    private function openHandler(ev:Event):void {    
        trace("CONNECTED\n");
    }

    private function readHandler(ev:Event):void {
        var msg:String = ev.data
        trace("MESSAGE: "+msg+"\n");
    }

    private function errorHandler(ev:Event):void {
        trace("ERROR\n");
    }
    ```

Notes
-----

-   To view sample source code using the EventSource API, see the `sse.mxml` file located in `GATEWAY_HOME/demo/flash/src/gateway/`.

Secure Your Flash Client
========================

Before you add security to your clients, follow the steps in [Configure Authentication and Authorization](https://github.com/kaazing/gateway/blob/develop/doc/security/o_auth_configure.md) to set up security on KAAZING Gateway for your client. The authentication and authorization methods configured on the Gateway influence your client's security implementation. For information on secure network connections between clients and the Gateway, see [Secure Network Traffic with the Gateway](../security/o_tls.md).


To Secure Your Flash Client
---------------------------

This section contains the following topics:

-   [Creating a Basic Challenge Handler](#creating-a-basic-challenge-handler)
-   [Using Wildcards to Match Sub-domains and Paths](#using-wildcards-to-match-sub-domains-and-paths)
-   [Creating a Basic Login Handler](#creating-a-basic-login-handler)
-   [Creating a More Complex Login Handler](#creating-a-more-complex-login-handler)
-   [Managing Log In Attempts](#managing-log-in-attempts)
-   [Creating Kerberos Challenge Handlers](#creating-kerberos-challenge-handlers)
-   [Directly Establish a Global Kerberos Challenge Handler](#directly-establish-a-global-kerberos-challenge-handler)
-   [Directly Establish a Global Kerberos Challenge Handler for a Specific Location](#directly-establish-a-global-kerberos-challenge-handler-for-a-specific-location)
-   [Implementing a Global Negotiate HTTP Authentication Strategy](#creating-a-global-negotiate-http-authentication-strategy)
-   [Implementing a Negotiate HTTP Authentication Strategy for a Specific Location](#creating-a-negotiate-http-authentication-strategy-for-a-specific-location)

Authenticating your client involves implementing a challenge handler to respond to authentication challenges from the Gateway. If your challenge handler is responsible for obtaining user credentials, then you will also need to implement a login handler.

**Note**: To use the KAAZING Gateway Flash client security library, ensure you add the following argument to the Flash compiler:

`-include-libraries GATEWAY_HOME/lib/client/flash/com.kaazing.gateway.client.swc`

If you built your application using Flash Builder, you can do this by choosing **Project** \> **Properties**. In the dialog, choose **Flex Compiler**, then add the argument to the **Additional compiler arguments** text box. Otherwise, add the argument to the Flash compiler manually.</span>

### Creating a Basic Challenge Handler

A challenge handler is a constructor used in an application to respond to authentication challenges from the Gateway when the application attempts to access a protected resource. Each of the resources protected by the Gateway is configured with a different authentication scheme (for example, Basic, Application Basic, or Application Token), and your application requires a challenge handler for each of the schemes that it will encounter or a single challenge handler that will respond to all challenges. Also, you can add a dispatch challenge handler to route challenges to specific challenge handlers according to the URI of the requested resource.

For information about each authentication scheme type, see [Configure the HTTP Challenge Scheme](https://github.com/kaazing/gateway/blob/develop/doc/security/p_authentication_config_http_challenge_scheme.md).

Clients with a single challenge handling strategy for all 401 challenges can simply set a specific challenge handler as the default using `ChallengeHandlers.setDefault()`. The following is an example of how to implement the authentication challenge in your client taken from the out of the box Flash & Flex demo at `http://localhost:8001/demo/`:

```

 // Configure a Basic Challenge Handler
 var basicHandler:BasicChallengeHandler = new DefaultBasicChallengeHandler();
 basicHandler.setLoginHandler(new DemoLoginHandler(this));
 ChallengeHandlers.setDefault(basicHandler);
```

For a complete example of a Flash client using security, see the out of the box demos located in `GATEWAY_HOME/demo/flash`.

**Note:** When using a `BasicChallengeHandler`, you must load the `DefaultBasicChallengeHandler`. To do so, either explicitly set this in your code (as shown in the previous code example), or add the necessary compiler argument to Flash Builder by going to Project Properties \> Build path \> Flex compiler and typing following into the Additional Compiler Arguments text box:

 `-includes com.kaazing.gateway.client.security.impl.DefaultBasicChallengeHandler`

Here is an example of how to set a global challenge handler declaratively:

```
var loginHandler:LoginHandler = new SampleLoginHandler1();

ChallengeHandlers.setDefault(
 (ChallengeHandlers.load(BasicChallengeHandler) as BasicChallengeHandler)
 .setLoginHandler(loginHandler)
```

The following example demonstrates how to register a location-specific challenge handler:

```
var dispatchChallengeHandler:DispatchChallengeHandler =
 ChallengeHandlers.load(DispatchChallengeHandler);

var basicChallengeHandler:BasicChallengeHandler = ChallengeHandlers.load(BasicChallengeHandler);

basicChallengeHandler.loginHandler =  new SampleLoginHandler1();

dispatchChallengeHandler.register("ws://my.server.com", basicChallengeHandler);

ChallengeHandlers.setDefault(dispatchChallengeHandler);
```

Here is how to register a location-specific challenge handler declaratively:

```
ChallengeHandlers.setDefault(ChallengeHandlers.load(DispatchChallengeHandler)
 .register("ws://my.server.com",
  (ChallengeHandlers.load(BasicChallengeHandler) as BasicChallengeHandler)
  .setLoginHandler(new SampleLoginHandler1()))
 );
```

To following example demonstrates how to register multiple location-specific challenge handlers declaratively:

```
var myServerLoginHandler:LoginHandler = new SampleLoginHandler1();

var anotherServerLoginHandler:LoginHandler = new SampleLoginHandler2();

ChallengeHandlers.setDefaultImplementation(ChallengeHandler, SampleChallengeHandler);

ChallengeHandlers.setDefault(ChallengeHandlers.load(DispatchChallengeHandler)
    .register("ws://my.server.com",
        (ChallengeHandlers.load(BasicChallengeHandler) as BasicChallengeHandler)
            .setLoginHandler(myServerLoginHandler))
    .register("ws://another.server.com",
        (ChallengeHandlers.load(ChallengeHandler) as SampleChallengeHandler)
            .setLoginHandler(anotherServerLoginHandler))
);
```

### Using Wildcards to Match Sub-domains and Paths

You can use wildcards (“\*”) when registering locations using `DispatchChallengeHandler`. Some examples of `locationDescription` values with wildcards are:

-   `*/` matches all requests to any host on port 80 (default port), with no user information or path specified.
-   `*.hostname.com:8000` matches all requests to port 8000 on any sub-domain of hostname.com, but not hostname.com itself.
-   `server.hostname.com:*/*` matches all requests to a particular server on any port on any path but not the empty path.

### <a name="basicloginhandler"></a>Creating a Basic Login Handler

The following example demonstrates how to implement a login handler in your client:

```

// Implement a login handler class
public class SampleLoginHandler extends LoginHandler {
    public function SampleLoginHandler() {
        super(this);
    }
    override public function getCredentials(continuation:Function):void {
        continuation("joe", "welcome");
    }
}
```

### Creating a More Complex Login Handler

The following example implements a more complex login handler to restrict login attempts to 3:

```
package org.kaazing.gateway.client.security.demo {
    import org.kaazing.gateway.client.security.LoginHandler;
    import flash.display.DisplayObject;
    import flash.utils.ByteArray;
    import mx.managers.PopUpManager;

    public class LimitedRetriesDemoLoginHandler extends LoginHandler {
        private var displayObject: DisplayObject;
         private const LIMIT: int = 3;
         private var attempts: int = 0;
         public function LimitedRetriesDemoLoginHandler(displayObject: DisplayObject) {
             super(this);
             this.displayObject = displayObject;
         }

        override public function getCredentials(continuation:Function):void {
             attempts++;

             if ( attempts > LIMIT ) {
                 // send an event perhaps to notify the wider application of attempt violation
                 attempts = 0;
                 continuation(null, null);
             } else {
                 var loginWindow:LoginForm=LoginForm(PopUpManager.createPopUp(displayObject,
                        LoginForm, false));
                 PopUpManager.centerPopUp(loginWindow);
                 loginWindow.continuation = continuation;
             }
        }
    }
}
```

#### Managing Log In Attempts

As displayed above, when it is not possible for the KAAZING Gateway client to create a challenge response, the client must return `null` to the Gateway to stop the Gateway from continuing to issue authentication challenges.

The following sample taken from [Creating a More Complex Login Handler](#creating-a-more-complex-login-handler) demonstrates how to stop the Gateway from issuing further challenges.

```
if ( attempts > LIMIT ) {
    // send an event perhaps to notify the wider application of attempt violation
    attempts = 0;
    continuation(null, null);
} else {
    var loginWindow:LoginForm=LoginForm(PopUpManager.createPopUp(displayObject,
           LoginForm, false));
    PopUpManager.centerPopUp(loginWindow);
    loginWindow.continuation = continuation;
}
```

### Creating Kerberos Challenge Handlers

The following examples demonstrate different implementations of Kerberos challenge handlers. When registered with the `DispatchChallengeHandler`, a `KerberosChallengeHandler` directly responds to Negotiate challenges where Kerberos-generated authentication credentials are required. In addition, a `KerberosChallengeHandler` can be used indirectly in conjunction with a `NegotiateChallengeHandler` to assist in the construction of a challenge response using object identifiers. For more information, see the [*ActionScript Client API*](http://developer.kaazing.com/documentation/5.0/apidoc/client/flash/gateway/index.html).

### Directly Establish a Global Kerberos Challenge Handler

The following example uses `KerberosChallengeHandler` to create a global Kerberos challenge handler:

```
var loginHandler:LoginHandler = new SampleLoginHandler1();
ChallengeHandlers.setDefault(
    (ChallengeHandlers.load(KerberosChallengeHandler) as KerberosChallengeHandler)
        .setLoginHandler(loginHandler));
```

### Directly Establish a Global Kerberos Challenge Handler for a Specific Location

The following example uses the `register()` method to establish a challenge handler for a specific location:

```
var loginHandler:LoginHandler = new SampleLoginHandler1();

ChallengeHandlers.setDefault(
    (ChallengeHandlers.load(DispatchChallengeHandler) as DispatchChallengeHandler)
        .register("ws://some.server.com:8010/kerberos5",
            (ChallengeHandlers.load(KerberosChallengeHandler) as KerberosChallengeHandler)
                .setDefaultLocation("ws://kb.hostname.com/kerberos5")
                .setRealmLocation("ATHENA.MIT.EDU", "ws://athena.hostname.com/kerberos5")
                .setServiceName("HTTP/servergw.hostname.com")
                .setLoginHandler(loginHandler))
);
```

### Creating a Global Negotiate HTTP Authentication Strategy

The following example establishes a single HTTP authentication strategy for all challenges.

```
var loginHandler:LoginHandler = new SampleLoginHandler1();

ChallengeHandlers.setDefault(
    (ChallengeHandlers.load(NegotiateChallengeHandler) as NegotiateChallengeHandler)
        .register((ChallengeHandlers.load(KerberosChallengeHandler) as KerberosChallengeHandler)
            .setLoginHandler(loginHandler))
        // register additional alternatives to negotiate here.
);
```

### Creating a Negotiate HTTP Authentication Strategy for a Specific Location

The following example uses the `register()` method to establish a location-specific negotiate HTTP authentication strategy:

```
var loginHandler:LoginHandler = new SampleLoginHandler1();
ChallengeHandlers.setDefault(
    (ChallengeHandlers.load(DispatchChallengeHandler) as DispatchChallengeHandler)
        .register("ws://some.server.com",
            (ChallengeHandlers.load(NegotiateChallengeHandler) as NegotiateChallengeHandler)
                .register((ChallengeHandlers.load(KerberosChallengeHandler) as
                    KerberosChallengeHandler)
                        .setDefaultLocation("ws://kb.hostname.com/kerberos5")
                        .setRealmLocation("ATHENA.MIT.EDU", "ws://athena.hostname.com/kerberos5")
                        .setServiceName("HTTP/servergw.hostname.com")
                        .setLoginHandler(loginHandler))
        // register additional alternatives to negotiate here.
        )
);
```

Display Logs for the Flash Client
=================================

The Kaazing ActionScript and Flex/Flash code uses [Flex Logging](http://livedocs.adobe.com/flex/3/html/help.md?content=logging_09.html#178687) to log messages.

To Enable Flash Client Log Messages
-----------------------------------

1.  Build your Kaazing Flash client, as described in [Build Flash WebSocket Clients](o_dev_flash.md).
2.  Download and install the [debug version of the Flash Player plug-in](http://www.adobe.com/support/flashplayer/downloads.html).

    For more information on using this, see Adobe's [online help](http://livedocs.adobe.com/flex/3/html/logging_03.md#178599http://livedocs.adobe.com/flex/3/html/logging_03.html#178599) on using the debugger version of Flash Player.
3.  Create the `mm.cfg` file for the Flash Player. For information where to save this file for your operating system, see Adobe's [online help](http://helpx.adobe.com/flash-player/kb/configure-debugger-version-flash-player.html) on creating the `mm.cfg` file.
4.  In the `mm.cfg` file, add the following lines to enable tracing:

    ``` as
    TraceOutPutFileName=/Users/<username>/Library/Preferences/Macromedia/Flash Player/Logs/flashlog.txt
    ErrorReportingEnable=1
    MaxWarnings=0
    TraceOutputFileEnable=1
    ```

    The `TraceOutPutFileName` parameter specifies where the log file will be created automatically (this example uses a Linux file directory). The *username* value represents the current user's home folder.

5.  Configure the log level for the various Kaazing classes in the client code. For example, the out of the box Flash & Flex WebSocket Echo demo, (which you can view by following the steps in [Setting Up KAAZING Gateway](https://github.com/kaazing/gateway/blob/develop/doc/about/setup-guide.md)) includes an example of this configuration, as shown here:

    ``` as
     private function initLogging():void {
     // Create a target.
       var logTarget:TraceTarget = new TraceTarget();
     // Log only messages for the classes in Kaazing packages
       logTarget.filters=["org.kaazing.gateway.client.*"];
     // Log all log levels.
       logTarget.level = LogEventLevel.ALL;
     // Add date, time, category, and log level to the output.
       logTarget.includeDate = true;
       logTarget.includeTime = true;
       logTarget.includeCategory = true;
       logTarget.includeLevel = true;
     // Begin logging.
       Log.addTarget(logTarget);
     }
    ```

6.  Save your changes then start (or restart) your browser.
7.  Visit the page containing the Flash applet. Log messages should display in the log file, for example in `/Users/username/Library/Preferences/Macromedia/Flash Player/Logs/flashlog.txt`. For information on locating your local log file, visit Adobe's [online help](http://helpx.adobe.com/flash-player/kb/configure-debugger-version-flash-player.html) on the log file location.

 **Notes:**

  - If you run the Flash client in Google Chrome using its integrated Flash plugin, no logging will occur, even if you have installed the debug version of the Adobe Flash player. To fix this configuration in Chrome, open the Chrome plugin settings (chrome://plugins/), disable the Chrome Flash Player, and then enable the Adobe Flash Player.

Troubleshoot Your Flash Client
==============================

This procedure provides troubleshooting information for the most common issue that occurs when using Kaazing Flash clients.

What Problem Are You Having?
----------------------------

-   [Error: org.kaazing.gateway.client.security.impl.*class\_name* cannot be found](#error-orgkaazinggatewayclientsecurityimplclass_name-cannot-be-found)
-   [Kerberos challenge handler not working](#kerberos-challenge-handler-not-working)

Error: org.kaazing.gateway.client.security.impl.*class\_name* cannot be found
------------------------------------------------------------------------------------------------------------------

When compiling your Flash client code, you see something similar to the following, where *class\_name* is something like `DefaultBasicChallengeHandler`:

`"ERROR: A class with the name 'org.kaazing.gateway.client.security.impl.*class\_name*' could not be found..."`

This warning occurs when you have not included the Kaazing Flash client security library in your Flash compiler.

To resolve this issue, add the following argument to the Flash compiler:

```
-include-libraries
`GATEWAY_HOME`/lib/client/actionscript/org.kaazing.gateway.client.swc
```

If you built your application using Flash Builder, you can do this by choosing **Project \> Properties**. In the dialog, choose **Flex Compiler**, then add the argument to the **Additional compiler arguments** text box. Otherwise, add the argument to the Flash compiler manually.

Kerberos challenge handler not working
-------------------------------------------------------------

**Cause:** [Kerberos challenge handlers](o_dev_flash.md#creating-kerberos-challenge-handlers) might not work for one or more of the following reasons:

-   The client cannot connect to the Kerberos Domain Controller (KDC).

    **Solution:** Ping the KDC from the computer running the client and the server hosting the Gateway. Also, ensure that you can Telnet to Kerberos port number 88 from both computers (`telnet> open KDC-server-name 88`).

-   The client cannot obtain a Kerberos ticket.

    **Solution:** Test ticket acquisition by executing the following commands to ensure that the KDC is accessible and able to issue service tickets:

    **For Linux:**

    `$ kinit -t /etc/keytab-name.keytab -S service-instance-name username@KDC-server-name`

    **For Windows:**

    `$ kinit username@KDC-server-name`

    The output will be:

    `Please enter the password for username@KDC-server-name:`

    Enter the password, and then enter:

    `$ klist`

    The ticket cache is displayed along with each ticket's expiration date.

-   Service name is in the incorrect format in the Kerberos challenge handler code.

    **Solution:** The service name should be in the format: `HTTP/servergw.hostname.com`. See [Creating Kerberos Challenge Handlers](../p_dev_flash_secure.md#creating-kerberos-challenge-handlers) for examples.

-   The pop-up dialog in the client used to obtain user credentials does not ensure that the username format is correct.

    **Solution:** Ensure that the result of the pop-up dialog used to obtain user credentials is formatted as
    `username@KDC-server-name`.

Next Step
---------

You have completed the Flash client howtos. For more information on client API development, see the [ActionScript (Flex) Client API](http://developer.kaazing.com/documentation/5.0/apidoc/client/flash/gateway/index.html).
