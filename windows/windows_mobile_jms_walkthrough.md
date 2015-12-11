---
Title: Build Microsoft Windows Mobile JMS Clients
Product: Gateway
Section: windows
DocType: Regular
Enterprise: True
---

This topic provides the information needed to create a Microsoft Windows Mobile app using the KAAZING Gateway .NET and Silverlight JMS API.

For information on using the KAAZING Gateway .NET and Silverlight JMS API to create applications for Windows 8 Desktop and Surface Pro, Surface RT and Silverlight, and other Microsoft .NET Framework 4.5 or 4.0 environments, see [How to Build Microsoft .NET and Silverlight JMS Clients Using KAAZING Gateway](o_dev_dotnet.md).

**Note:** The KAAZING Gateway .NET and Silverlight JMS API supports Microsoft Windows Phone 8.1 and higher, Microsoft Windows Mobile 10, and Microsoft Windows Surface RT. It does not support Windows CE.


| # | Step                                                                                          | Section or Reference                                                                                             |
|:--|:----------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| 1 | Learn how to use the JMS API provided by the KAAZING .NET and Silverlight JMS client library. | [Use the Microsoft Windows Mobile WebSocket Client API](#use-the-microsoft-windows-phone-websocket-client-api)   |
| 2 | Learn how to authenticate your client with the Gateway.                                       | [Secure Your Microsoft Windows Mobile JMS Client](#secure-your-microsoft-windows-phone-client)                   |
| 3 | Set up logging for your client.                                                               | [Display Logs for the Microsoft Windows Mobile JMS Client](#display-logs-for-the-microsoft-windows-phone-client) |


### Introduction

In this how-to, you will learn how to use the KAAZING Gateway Windows Mobile JMS client library to enable your Windows Mobile client to communicate with a JMS broker via KAAZING Gateway over WebSocket.

This document is for a Windows Mobile developer who wants their Windows Mobile client to communicate over WebSocket with KAAZING Gateway, which then connects to a JMS broker.

This topic covers the following information:

- [Use the Microsoft Windows Mobile JMS Client API](#use-the-microsoft-windows-phone-jms-client-api)
    - [Components and Tools](#components-and-tools)
    - [Taking a Look at the Microsoft Windows Mobile JMS Client Demo](#taking-a-look-at-the-microsoft-windows-phone-jms-client-demo)
    - [Primary Microsoft Windows Mobile JMS API Features](#primary-microsoft-windows-phone-jms-api-features)
    - [Supported Data Types](#supported-data-types)
    - [Build the Microsoft Windows Mobile JMS Demo](#build-the-microsoft-windows-mobile-jms-demo)
    - [Demo Files](#demo-files)
- [Secure Your Microsoft Windows Mobile JMS Client](#secure-your-microsoft-windows-phone-jms-client)
    - [Creating a Basic Challenge Handler](#creating-a-basic-challenge-handler)
- [Display Logs for the Microsoft Windows Mobile JMS Client](#display-logs-for-the-microsoft-windows-phone-jms-client)

## Use the Microsoft Windows Mobile JMS Client API

In this procedure, you will learn how to create a Windows Mobile app using the KAAZING Gateway .NET and Silverlight JMS API. You will learn how to create a Visual Studio 2013 project and add the necessary references in order to use the .NET and Silverlight JMS API, and implement the .NET and Silverlight JMS API methods to enable your Windows Mobile app to send and receive messages with the `jms` service running on a KAAZING Gateway.

### Components and Tools

Before you get started, review the components and tools used to build the Microsoft Windows Mobile JMS client in this procedure.

| Component or Tool                         | Description                                                                                                                                                                              | Location                                                                                                                                                                                                                                                              |
|:------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| KAAZING Gateway - Enterprise Edition      | KAAZING Gateway provides a highly scalable, near zero-latency, full-duplex solution for JMS applications.                                                                                | The KAAZING Gateway is available at [kaazing.com](http://kaazing.com/products/editions/kaazing-websocket-gateway-jms/).                                                                                                                                               |
| Target Framework or Environment           | Microsoft Windows Mobile devices                                                                                                                                                         | [Windows Mobile website](https://www.windowsphone.com/en-us)                                                                                                                                                                                                          |
| Library for target deployment environment | The DLLs needed to build a Microsoft Windows Mobile JMS app.                                                                                                                             | There are two DLLs, Kaazing.Gateway.dll and Kaazing.JMS.dll. They are available in the KAAZING Gateway distribution (`GATEWAY_HOME/lib/client/dotnet`).                                                                                                               |
| JMS-compliant message broker              | KAAZING Gateway will work with any JMS message broker.                                                                                                                                   | The open source Apache ActiveMQ broker is included in the KAAZING Gateway distribution.                                                                                                                                                                               |
| Development Tool                          | Visual Studio                                                                                                                                                                            | [www.visualstudio.com](http://www.visualstudio.com)                                                                                                                                                                                                                   |
| Package Installer (optional)              | You can use the NuGet Package Manager to install the Microsoft Windows Mobile JMS client library as a Nuget Package.                                                                     | Starting with Visual Studio 2012, NuGet is included in every edition (except Team Foundation Server) by default. Updates to NuGet can be found through the Extension Manager. For Visual Studio 2010, NuGet is available through the Visual Studio Extension Manager. |
| NuGet Command-Line (optional)             | NuGet Command-Line Utility                                                                                                                                                               | [https://docs.nuget.org/consume/installing-nuget](https://docs.nuget.org/consume/installing-nuget)                                                                                                                                                                    |
| Secure Networking of TLS/SSL              | A Microsoft Windows Mobile JMS client runs on Microsoft Windows Mobile devices. Windows Mobile devices manage TLS/SSL connections, requesting TLS/SSL certificates from KAAZING Gateway. | See the article, [Installing digital certificates](https://msdn.microsoft.com/en-us/library/dn643705.aspx), from Microsoft.                                                                                                                                           |

### Taking a Look at the Microsoft Windows Mobile JMS Client Demo

Take a look at the Microsoft Windows Mobile Client JMS Demo that was built with the Microsoft .NET and Silverlight JMS client library. To see this demo in action, perform the following steps:

1. Go to [kaazing.org](http://www.kaazing.org) to fork or download the KAAZING Gateway Microsoft Windows Mobile JMS demo. The Windows Mobile Silverlight demo repo is located at [WindowsPhone](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/tree/develop/jms/demo/WindowsPhone/JmsDemo).

2. Build the demo by following the steps described in the repo.

### Primary Microsoft Windows Mobile JMS API Features

The Microsoft Windows Mobile JMS client library uses the KAAZING Microsoft .NET and Silverlight JMS API. The primary features of the Microsoft .NET and Silverlight JMS API involve common event handlers for opening and closing WebSocket connections, and for sending and receiving messages.

#### Establishing Connections to KAAZING Gateway

The following example demonstrates how to open and close a connection. The code includes authentication and updates the client UI with message describing the connection state.

The [`ConnButton_Click`](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml.cs#L76-L87) event handler is used to capture the event of a user clicking the Connect button on the app. You can see this method call [`JMS_ConnectAsync`](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml.cs#L215-L265) to actually create the connection object (using the `StompConnectionFactory` class), connect to KAAZING Gateway (using the `CreateConnection()` method), and create consumers (using `IMessageConsumer`) and a session (using `CreateSession()`).

``` c#
...
using Kaazing.JMS;
using Kaazing.JMS.Stomp;
using Kaazing.Security;

namespace JmsDemo
{
    public partial class MainPage : PhoneApplicationPage
    {
        private IConnection connection = null;
        private ISession session = null;
        private IMessageConsumer consumer = null;
        private IDictionary<String, List<IMessageConsumer>> consumers = null;
        private int lockUIControls = 0;
    ...
        private void ConnButton_Click(object sender, RoutedEventArgs e)
        {
            if (Interlocked.CompareExchange(ref lockUIControls, 1, 0) == 0)
            {
                string url = UrlBox.Text;

                Task.Run(() =>
                {
                    var t = JMS_ConnectAsync(url);
                }, CancellationToken.None);
            }
        }
        ...
        private void ClosedHandler()
        {
            var t = this.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                Log("CLOSED");
                ConnectButton.IsEnabled = true;
                DisconnectButton.IsEnabled = false;

                // disable other buttons
                SendButton.IsEnabled = false;
                SubscribeButton.IsEnabled = false;
                UnsubscribeButton.IsEnabled = false;
            });
        }
        ...
        private async Task JMS_ConnectAsync(string url)
        {
            try
            {
                HandleLog("CONNECT:" + url);

                await Task.Run(() =>
                {
                    try
                    {
                        StompConnectionFactory connectionFactory = new StompConnectionFactory(new Uri(url));
                        connectionFactory.WebSocketFactory.ChallengeHandler = basicHandler;
                        connection = connectionFactory.CreateConnection("", "");
                        connection.ExceptionListener = new ExceptionHandler(this);
                        consumers = new Dictionary<String, List<IMessageConsumer>>();
                        session = connection.CreateSession(false, SessionConstants.AUTO_ACKNOWLEDGE);
                        connection.Start();
                        var t = this.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
                        {
                            Log("CONNECTED");
                            // Enable User Interface for Connected application
                            SubscribeButton.IsEnabled = true;
                            SendButton.IsEnabled = true;
                            DisconnectButton.IsEnabled = true;
                            ConnectButton.IsEnabled = false;
                        });
                    }
                    catch (Exception exc)
                    {
                        if (connection != null)
                        {
                            connection.Close();
                        }

                        var t = this.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
                        {
                            Log("CONNECTION FAILED: " + exc.Message);
                            ConnectButton.IsEnabled = true;
                        });
                    }
                });
            }
            catch (Exception ex)
            {
                HandleLog(ex.Message, Colors.Red);
            }
            finally
            {
                Interlocked.CompareExchange(ref lockUIControls, 0, 1);
            }
        }
```

Closing the connection is displayed in the following method, [`JMS_CloseAsync()`](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml.cs#L267-L303). The method uses the `Close()` method of `StompConnection` class.

``` c#
        private async Task JMS_CloseAsync()
        {
            try
            {
                HandleLog("CLOSING");

                if (connection != null)
                {
                    await Task.Run(() =>
                    {

                        // Close in a new thread to prevent blocking UI
                        try
                        {
                            connection.Close();
                        }
                        catch (Exception exc)
                        {
                            throw exc;
                        }
                        finally
                        {
                            connection = null;
                            ClosedHandler();
                        }
                    });
                }
            }
            catch (Exception ex)
            {
                HandleLog(ex.Message, Colors.Red);
            }
            finally
            {
                Interlocked.CompareExchange(ref lockUIControls, 0, 1);
            }
        }
```

#### Subscribing to a Topic or Queue

The following example is taken from a Windows Mobile JMS Client demo and includes the [`JMS_SubscribeAsync`](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml.cs#L305-L364) method. This method uses the `session` object created when the `StompConnectionFactory` object was created at the start of the program:

``` c#
        private ISession session = null;
        ...
        session = connection.CreateSession(false, SessionConstants.AUTO_ACKNOWLEDGE);
```
The `JMS_SubscribeAsync` method demonstrates how the JMS client uses the destination specified by the user, the string `/topic/`, to create a topic (`session.CreateTopic(dest)`), and how the client creates a queue (`session.CreateQueue(dest)`) if the destination is missing the string `/topic/`.

`JMS_SubscribeAsync` also creates a consumer of the topic or queue (`session.CreateConsumer(destination)`) and a `MessageListener` to receive asynchronously delivered messages.


``` c#

        private async Task JMS_SubscribeAsync(string dest)
        {
            try
            {
                HandleLog("SUBSCRIBE:" + dest);
                if (consumers.ContainsKey(dest))
                {
                    HandleLog("Destion already Consumed:" + dest);
                }
                else
                {
                    await Task.Run(() =>
                    {
                        IDestination destination;
                        if (dest.StartsWith("/topic/"))
                        {
                            destination = session.CreateTopic(dest);
                        }
                        else
                        {
                            destination = session.CreateQueue(dest);
                        }

                        consumer = session.CreateConsumer(destination);
                        consumer.MessageListener = new MessageHandler(this);

                        List<IMessageConsumer> consumerList = new List<IMessageConsumer>();
                        consumerList.Add(consumer);

                        try
                        {
                            consumers.Add(dest, consumerList);
                            var t = this.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
                            {
                                SubscribeButton.IsEnabled = false;
                                UnsubscribeButton.IsEnabled = true;
                            });

                        }
                        catch (ArgumentException)
                        {

                            // we catch the ArgumentException here because in Java the the VALUE
                            // is replaced if a KEY already exists:
                            List<IMessageConsumer> oldValue = consumers[dest];
                            consumers.Remove(dest);
                            consumers.Add(dest, consumerList);
                        }
                    });
                }
            }
            catch (Exception ex)
            {
                HandleLog(ex.Message, Colors.Red);
            }
            finally
            {
                Interlocked.CompareExchange(ref lockUIControls, 0, 1);
            }
        }
```

#### Sending and Receiving Messages

The following example is taken from a Windows Mobile JMS Client demo and includes the [`JMS_SendAsync`](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml.cs#L413-L501) method.

`JMS_SendAsync` creates a topic or queue from the destination supplied by the user the same way as `JMS_SubscribeAsync`. Next, it creates the message to send using `iMessage` and `session.CreateTextMessage(msg)`. The `iMessage` interface is the root interface of all JMS messages. It defines the message header and the acknowledge method used for all messages. Once the message is created, `JMS_SendAsync` creates the producer (using 'IMessageProducer' and the destination), sends the message (using `producer.Send(message)`), and closes the producer.

To send the text message input as binary, `IBytesMessage` is used. `IBytesMessage` inherits from the `IMessage` interface and adds a bytes message body. The receiver of the message supplies the interpretation of the bytes. The `WriteUTF` method of `IBytesMessage` writes the string to the bytes message stream using UTF-8 encoding in a machine-independent manner.

``` c#
        private async Task JMS_SendAsync(string dest, string msg, bool binaryMessage, bool showJMSHeaders)
        {
            try
            {
                // Create a destination for the producer
                IDestination destination;
                if (dest.StartsWith("/topic/"))
                {
                    destination = session.CreateTopic(dest);
                }
                else if (dest.StartsWith("/queue/"))
                {
                    destination = session.CreateQueue(dest);
                }
                else
                {
                    throw new ArgumentException("Destination must start with /topic/ or /queue/");
                }

                // Create the message to send
                IMessage message;
                StringBuilder sb = new StringBuilder();
                if (binaryMessage)
                {
                    sb.AppendLine("SEND BYTE: " + BitConverter.ToString(Encoding.UTF8.GetBytes(msg)) + ": " + dest);
                    message = session.CreateBytesMessage();
                    ((IBytesMessage)message).WriteUTF(msg);
                }
                else
                {
                    sb.AppendLine("SEND TEXT: " + msg + ": " + dest);
                    message = session.CreateTextMessage(msg);
                }

                await Task.Run(() =>
                {
                    // Create the producer, send, and close
                    IMessageProducer producer = session.CreateProducer(destination);
                    try
                    {
                        producer.Send(message);
                        if (showJMSHeaders)
                        {
                            // show JMS message headers
                            if (message.JMSDeliveryMode == Kaazing.JMS.DeliveryModeConstants.NON_PERSISTENT)
                            {
                                sb.AppendLine("DeliveryMode: NON_PERSISTENT");
                            }
                            else if (message.JMSDeliveryMode == Kaazing.JMS.DeliveryModeConstants.PERSISTENT)
                            {
                                sb.AppendLine("DeliveryMode: PERSISTENT");
                            }
                            else
                            {
                                sb.AppendLine("DeliveryMode: UNKNOWN");
                            }
                            sb.AppendLine("JMSPriority: " + message.JMSPriority);
                            sb.AppendLine("JMSMessageID: " + message.JMSMessageID);
                            sb.AppendLine("JMSTimestamp: " + message.JMSTimestamp);
                            sb.AppendLine("JMSCorrelationID: " + message.JMSCorrelationID);
                            sb.AppendLine("JMSType: " + message.JMSType);
                            sb.AppendLine("JMSReplyTo: " + message.JMSReplyTo);
                        }
                        HandleLog(sb.ToString().TrimEnd('\n', '\r'), Colors.Green);
                    }
                    catch (Exception exception)
                    {
                        var t = this.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
                        {
                            Log("EXCEPTION: " + exception.Message);
                        });
                    }
                    finally
                    {
                        producer.Close();
                    }
                });
            }
            catch (Exception ex)
            {
                HandleLog(ex.Message, Colors.Red);
            }
            finally
            {
                Interlocked.CompareExchange(ref lockUIControls, 0, 1);
            }
        }
```

You will notice that `JMS_SubscribeAsync` also contains the `consumer.MessageListener = new MessageHandler(this);` for receiving messages once the client is subscribed. The `MessageListener` object is for asynchronous delivery, where messages arrive at the message consumer by calling the `onMessage` method of `MessageListener`.

You can see `onMessage` in [`MessageHandler`](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml.cs#L546-L622) class. `MessageHandler` manages incoming messages, sorting text and binary messages, and writing them to the Log.  

`MessageHandler` also manages message of the [`MapMessage`](http://docs.oracle.com/cd/E17802_01/products/products/jms/javadoc-102a/javax/jms/MapMessage.html) type using `IMapMessage`. A `IMapMessage` object is used to send a set of name-value pairs. The names are String objects, and the values are primitive data types.


### Supported Data Types

You can send a JMS message using one of the following data types:

- String (`ITextMessage`) — A text message (UTF-8).
- Binary (`IBytesMessage`) — A compact byte array representation for sending, receiving and processing binary data.

### Build the Microsoft Windows Mobile JMS Demo

The following procedure summarizes the Visual Studio steps and primary code elements used in the Microsoft Windows Mobile JMS Demo available in the KAAZING Gateway distribution (GATEWAY_HOME/demo/dotnet). At the end of this procedure, you will have a working Windows Mobile JMS client that can send and receive messages with a local KAAZING Gateway hosted on your computer.

The completed demo looks file the following:

![Microsoft Windows Mobile JMS Demo](images/windows-phone-jms-demo.png)

1. Create a new Visual Studio project.
    1. To create a new Visual Studio Windows Mobile JMS project, click **File**, click **New**, and then click **Project**.
    2. Name the project **JMSDemo**. This name is used in order that you can follow the code examples later in this procedure.
    3. In **Templates**, expand **Visual C#**, then expand **Windows Mobile Apps**, and double-click **Blank App (Windows Phone)**.
2. Locate the Microsoft Windows .NET and Silverlight library in the KAAZING Gateway distribution. In a browser, navigate to the KAAZING Gateway distribution folder containing the KAAZING Gateway Microsoft Windows .NET and Silverlight libraries (`GATEWAY_HOME/lib/dotnet`). For information on the location, see [Components and Tools](#components-and-tools).
3. Add the KAAZING Gateway Microsoft Windows .NET and Silverlight library DLL to your Visual Studio Windows Mobile project.      1. In Solution Explorer, right-click **References**, and then click **Add Reference**. For more information, see [How to: Add or Remove References By Using the Add Reference Dialog Box](https://msdn.microsoft.com/en-us/library/wkze6zky.aspx).
    2. In **Reference Manager**, click **Browse**.
    3. Locate the **Kaazing.WebSocket.dll** file in `GATEWAY_HOME/lib/dotnet/websocket/wp81` (TODO), and then click **Add**.
    4. Locate the **Kaazing.JMS.dll** file in `GATEWAY_HOME/lib/dotnet/jms/wp81` (TODO), click **Add**, and then click **OK** to close the reference dialog.
    5. To see the library package, in **Solution Explorer**, expand the **References** section. **Kaazing.WebSocket** and **Kaazing.JMS** are listed. You can double-click a package to see it in the **Object Browser**.
4. Create the main files for the program. The following steps demonstrate the primary programming steps involved in creating the demo or any client using the KAAZING Gateway Windows Mobile JMS client library.
    6. In your Visual Studio Windows Mobile project, click **MainPage.xaml**.
    7. Replace the contents of **MainPage.xaml** with the code in [MainPage.xaml](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml) in the [Windows Mobile JMS Demo repo](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/tree/develop/jms/demo/WindowsPhone/JmsDemo).
    8. Expand **MainPage.xaml** to reveal **MainPage.xaml.cs**.
    9. Replace the contents of **MainPage.xaml.cs** with the code in [MainPage.xaml.cs](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/MainPage.xaml.cs).
    10. Right-click **MainPage.xaml.cs** and then click **View Designer**. The designer displays the Windows Mobile WebSocket Demo GUI.
    11. Right-click **MainPage.xaml.cs** and then click **View Code**. In the code you will notice an error on for `DemoLoginHandler()`. `DemoLoginHandler()` is used for managing client authentication.
    12. To add the authentication GUI to your app, right-click the project title, **JMSDemo**, click **Add**, and then click **New Item**.
    13.  In **Add New Item**, expand **Visual C#**, click **Windows Mobile**, and then click **Windows Mobile User Control**.     14. In **Name**, enter **LogonControl.xaml** and click **Add**.
    15. In Solution Explorer, right-click **LogonControl.xaml** and then click **View Designer**.
    16. Replace the contents of the user control code with the code in [LogonControl.xaml](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/LogonControl.xaml). This will give your app the GUI controls for authentication.
    17. In Solution Explorer, expand **LogonControl.xaml** and then click **LogonControl.xaml.cs**.
    18. Replace the contents of **LogonControl.xaml.cs** with the code in [LogonControl.xaml.cs](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/LogonControl.xaml.cs).
    19. To add the authentication code to your app, right-click the project title, **JmsDemo**, click **Add**, and then click **New Item**.
    20. In **Add New Item**, expand **Visual C#**, click **Code**, and then click **Class**.
    21. In **Name**, enter **DemoLoginHandler.cs** and click **Add**.
    22. In Solution Explorer, expand **DemoLoginHandler.cs** and click **DemoLoginHandler**.
    23. Replace the contents of **DemoLoginHandler** with the code in [DemoLoginHandler.cs](https://github.com/chao-sun-kaazing/enterprise.dotnet.client/blob/develop/jms/demo/WindowsPhone/JmsDemo/DemoLoginHandler.cs). If you return to **MainPage.xaml.cs**, you will see that the error for DemoLoginHandler() is gone. You have added authentication to your app.

5. Start KAAZING Gateway and the Apache ActiveMQ broker as described in [Setting Up the Gateway](https://github.com/kaazing/gateway/blob/develop/doc/about/setup-guide.md).  

    Next you will run the Windows Mobile JMS Demo on the [Windows Mobile Emulator](https://msdn.microsoft.com/en-us/library/windows/apps/ff402563%28v=vs.105%29.aspx).

7. On the **Standard** toolbar, select **Emulator 8.1 WVGA 4 inch 512MB**.
    ![Emulator 8.1 WVGA 4 inch 512MB](images/Emulator512B.png)  

    To deploy and run your app without debugging, click the **DEBUG** menu, click **Start without Debugging**, or press **Ctrl+F5**.

8. In the emulator, in location, enter **ws://localhost:8001/jms** and click **Connect**. The Log displays `CONNECTED`.

9. Click **Subscribe**. The app is subscribed to the destination.
    ``` bash
    TODO
    ```
10. Click **Send**. The message is sent to the destination.

10. Click the **Binary** checkbox and click **Send** again. The message is sent to the destination and the binary message is received:

    ``` bash
    TODO
    ```

### Demo Files
The following demo files are used for the KAAZING Gaterway Microsoft Windows Mobile WebSocket demo ([Windows Mobile Silverlight app demo](https://github.com/kaazing/enterprise.dotnet.client/tree/develop/ws/demo/WindowsPhoneSilverlight)).

- [MainPage.xaml]()
- [MainPage.xaml.cs]()
- [LoginControl.xaml]()
- [LoginControl.xaml.cs]()
- [LoginHandlerDemo.cs]()

## Secure Your Microsoft Windows Mobile Client

This topic provides information on how to add user authentication functionality to KAAZING Gateway Windows Mobile clients. The KAAZING Gateway Windows Mobile Clients use the KAAZING Gateway Microsoft .NET and Silverlight API authentication classes and methods. For information on these authentication classes and methods, see [Secure Your Microsoft .NET and Silverlight Client](https://github.com/kaazing/enterprise.dotnet.client/blob/develop/ws/ws/doc/p_dev_dotnet_secure.md).

### Creating a Basic Challenge Handler

Authenticating your client involves implementing a challenge handler to respond to authentication challenges from the KAAZING Gateway. If your challenge handler is responsible for obtaining user credentials, then you will also need to implement a login handler.

Here is an example of a challenge handler and login handler for a Windows Mobile app. The challenge handler manages the authentication challenge from the KAAZING Gateway, and the login handler is used to obtain the user credentials and return them to the challenge handler as part of a callback.

``` c#
using Kaazing.JMS;
using Kaazing.JMS.Stomp;
using Kaazing.Security;
using Windows.UI.Popups;

namespace JmsDemo
{
    /// <summary>
    /// An empty page that can be used on its own or navigated to within a Frame.
    /// </summary>
    public sealed partial class MainPage : Page
    {
        ...
        // username and password from Login Popup Page
        public PasswordAuthentication Credentials;
        BasicChallengeHandler basicHandler;
        public AutoResetEvent userInputCompleted = new AutoResetEvent(false);

        // Constructor
        public MainPage()
        {
            this.InitializeComponent();

            this.NavigationCacheMode = NavigationCacheMode.Required;

            // Create handler for authentication
            basicHandler = BasicChallengeHandler.Create();
            basicHandler.LoginHandler = new DemoLoginHandler(this);
        }

        ...
        private async Task JMS_ConnectAsync(string url)
        {
            try
            {
                HandleLog("CONNECT:" + url);

                await Task.Run(() =>
                {
                    try
                    {
                        StompConnectionFactory connectionFactory = new StompConnectionFactory(new Uri(url));
                        connectionFactory.WebSocketFactory.ChallengeHandler = basicHandler;
                        ...
        ...

                public void PopupLoginPage()
        {
            var t = this.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                if (popup.Child == null)
                {
                    Border border = new Border();

                    StackPanel panel1 = new StackPanel();
                    panel1.Height = 600;
                    panel1.Width = 370;
                    panel1.Background = new SolidColorBrush(Colors.Black);
                    panel1.Opacity = 0.8;
                    StackPanel panel2 = new StackPanel();
                    panel2.Background = new SolidColorBrush(Colors.White);
                    panel2.Opacity = 100;
                    panel2.Margin = new Thickness(0, 50, 0, 0);
                    LogonControl login = new LogonControl();
                    login.parent = this;
                    panel2.Children.Add(login);
                    panel1.Children.Add(panel2);
                    border.Child = panel1;

                    popup.Child = border;
                }
                popup.IsOpen = true;
            });
        }
...
```

This code calls a popup defined in the file [LoginControl.xaml](TODO). The code for the popup collects the user credentials in [LoginControl.xaml.cs](TODO). Lastly, the code in [DemoLoginHandler]() passes the credentials back to the code of the app in [MainPage.xaml.cs](TODO).

Learn more about the authentication options available to your Windows Mobile client in [Secure Your Microsoft .NET and Silverlight Client](https://github.com/kaazing/enterprise.dotnet.client/blob/develop/ws/ws/doc/p_dev_dotnet_secure.md).

## Display Logs for the Microsoft Windows Mobile Client

The KAAZING Gateway Windows Mobile Clients use the KAAZING Gateway Microsoft .NET and Silverlight API logging and debugging features. For information on these authentication classes and methods, see [Display Logs for Microsoft .NET and Silverlight Clients](https://github.com/kaazing/enterprise.dotnet.client/blob/develop/ws/ws/doc/p_clientlogging_dotnet.md).

Visual Studio includes debugging features for Windows Mobile 8. For more information, see [Phone Debugging in Visual Studio 2013 Update 2](http://blogs.msdn.com/b/visualstudioalm/archive/2014/04/04/phone-debugging-in-visual-studio-2013-update-2.aspx) from Microsoft.
