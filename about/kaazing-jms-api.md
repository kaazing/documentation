---
Title: Supported APIs in Kaazing Gateway JMS Client Libraries
Product: Gateway
Section: enterprise-shield
DocType: Regular
Enterprise: True
---

This document lists how the KAAZING Gateway JMS client libraries support the JMS 1.1 classes that have superseded JMS 1.0 API classes, and the JMS APIs that are not supported by the KAAZING Gateway JMS client libraries.

**Note:** KAAZING Gateway support for APIs listed in this document depends upon whether the back-end message broker system supports the APIs.

For information about the JMS API, see the [JMS API specification](http://www.oracle.com/technetwork/java/docs-136352.html).

For documentation on the available JMS API methods in the KAAZING Gateway JMS Client Libraries, see [Client API Documentation](../index.md) on the main Documentation page.

This document also contains the following two sections:

-   [JMS APIs Not Supported by the KAAZING Gateway JMS Client Libraries](#jms-apis-not-supported-by-the-kaazing-gateway-jms-client-libraries)
-   [JMS 1.1 Classes Supported by the KAAZING Gateway JMS Client Libraries](#jms-11-classes-supported-by-the-kaazing-gateway-jms-client-libraries)

Refer to the appropriate KAAZING Gateway client topics in [For Developers](../index.md) to learn how to use the KAAZING Gateway JMS Client Libraries.

#### JMS APIs Not Supported by the KAAZING Gateway JMS Client Libraries

The following table lists the specific APIs that are *not* supported by the KAAZING Gateway JMS Client Libraries.

**Note**: JNDI is only supported in the Java JMS client.

| JMS API Interface/Class | Support Level in KAAZING Gateway - Enterprise Edition                                                                                                                                                                                                                                                                                                                                                         | Client Technology |
|:------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------|
| BytesMessage            | The following methods are not supported in JavaScript:-,`readLong()`, `readFloat()`, `readDouble()`, `writeLong()`, `writeFloat()`, `writeDouble(), `writeObject()`                                                                                                                                                                                                                                           | JavaScript        |
| Connection              | The following methods are not supported: `createConnectionConsumer()- ,`createDurableConnectionConsumer()`                                                                                                                                                                                                                                                                                                    | All               |
| Connection              | The following methods are not supported: `setClientId()`, `run()`                                                                                                                                                                                                                                                                                                                                             | All               |
| ConnectionConsumer      | This class is not supported.                                                                                                                                                                                                                                                                                                                                                                                  | All               |
| KMBytesMessage          | The following methods are not supported in Objective-C: `readFloat()`, `readDouble()`, `writeFloat()`, `writeDouble()`, `writeObject()` **Note:**: To send float and double values, you can use message properties and `MapMessage`. See the [Objective-C JMS Client API documentation](http://developer.kaazing.com/documentation/jms/4.0/apidoc/client/ios/jms/KMStompJMS/index.html) for more information. | Objective-C       |
| Message                 | The following method is not supported: `correlationIdAsBytes`. **Note:** Not all types and conversions are supported. See the appropriate client API [](http://developer.kaazing.com/documentation/jms/4.0/apidoc/client/flash/jms/index.html)documentation to see which ones are supported.                                                                                                                  | All               |
| MessageConsumer         | The following methods are not supported in Flash: `receive()`, `receiveNoWait()`                                                                                                                                                                                                                                                                                                                              | Flash             |
| MessageConsumer         | The following method is not supported in JavaScript: `receive()`                                                                                                                                                                                                                                                                                                                                              | JavaScript        |
| MessageProducer         | The following methods are not supported: `disableMessageId()`, `disableMessageTimestamp()`                                                                                                                                                                                                                                                                                                                    | All               |
| MessageProducer         | The following methods are not supported in JavaScript: `getDisableMessageId()`, `getDisableMessageTimestamp()`, `setDisableMessageId()`, `setDisableMessageTimestamp()`                                                                                                                                                                                                                                       | JavaScript        |
| ObjectMessage           | This class is not supported.                                                                                                                                                                                                                                                                                                                                                                                  | All               |
| QueueBrowser            | This class is not supported.                                                                                                                                                                                                                                                                                                                                                                                  | All               |
| QueueRequestor          | This class is not supported.                                                                                                                                                                                                                                                                                                                                                                                  | All               |
| ServerSession           | This class is not supported.                                                                                                                                                                                                                                                                                                                                                                                  | All               |
| ServerSessionPool       | This class is not supported.                                                                                                                                                                                                                                                                                                                                                                                  | All               |
| Session                 | The following methods are not supported: `createBrowser()`, `createConsumer()` on a transacted session, `createObjectMessage()`, `createStreamMessage()`, ``run()`                                                                                                                                                                                                                                            | All               |
| StreamMessage           | This class is not supported.                                                                                                                                                                                                                                                                                                                                                                                  | All               |

**Note:** Constants defined by JMS in the Session class are not supported by ActionScript. Use `SessionConstants` instead.

##### Transactional Interfaces Not Supported

The following transactional interfaces are not supported:

-   XAConnection
-   XAConnectionFactory
-   XAQueueConnection
-   XAQueueConnectionFactory
-   XAQueueSession
-   XASession
-   XATopicConnection
-   XATopicConnectionFactory
-   XATopicSession

#### JMS 1.1 Classes Supported by the KAAZING Gateway JMS Client Libraries

The following classes are part of the JMS 1.0 API and have been superseded by JMS 1.1 classes. Use the following JMS 1.1 classes supported by KAAZING Gateway.

**Note:** JNDI is only supported in the Java JMS client.

| For the following JMS 1.0 API Interface/Class | Use this KAAZING Gateway JMS API 1.1 Interface/Class or Functionality | Client Technology             |
|:----------------------------------------------|:----------------------------------------------------------------------|:------------------------------|
| BytesMessage                                  | BytesMessage                                                          | JavaScript Java Android Flash |
| MapMessage                                    | MapMessage                                                            | JavaScript Java Android Flash |
| QueueConnection                               | Connection                                                            | JavaScript Java Android Flash |
| QueueConnectionFactory                        | ConnectionFactory                                                     | JavaScript Java Android Flash |
| QueueReceiver                                 | MessageConsumer                                                       | JavaScript Java Android Flash |
| QueueRequestor                                | TemporaryQueue                                                        | JavaScript Flash              |
| QueueSender                                   | MessageProducer                                                       | JavaScript Java Android Flash |
| QueueSession                                  | Session                                                               | JavaScript Java Android Flash |
| TextMessage                                   | TextMessage                                                           | JavaScript Java Android Flash |
| TopicConnection                               | Connection                                                            | All                           |
| TopicConnectionFactory                        | ConnectionFactory                                                     | All                           |
| TopicPublisher                                | MessageProducer                                                       | JavaScript Java Android Flash |
| TopicRequestor                                | TemporaryTopic                                                        | Flash Java Android JavaScript |
| TopicSession                                  | Session                                                               | JavaScript Java Android Flash |
| BytesMessage                                  | IBytesMessage                                                         | .NET Silverlight              |
| MapMessage                                    | IMapMessage                                                           | .NET Silverlight              |
| QueueConnection                               | IConnection                                                           | .NET Silverlight              |
| QueueConnectionFactory                        | IConnectionFactory                                                    | .NET Silverlight              |
| QueueRequestor                                | ITemporaryQueue                                                       | .NET Silverlight              |
| QueueReceiver                                 | IMessageConsumer                                                      | .NET Silverlight              |
| QueueSender                                   | IMessageProducer                                                      | .NET Silverlight              |
| QueueSession                                  | ISession                                                              | .NET Silverlight              |
| TextMessage                                   | ITextMessage                                                          | .NET Silverlight              |
| TopicConnectionFactory                        | IConnectionFactory                                                    | .NETSilverlight               |
| TopicPublisher                                | IMessageProducer                                                      | .NET Silverlight              |
| TopicRequestor                                | ITemporaryTopic                                                       | NET Silverlight               |
| TopicSession                                  | ISession                                                              | NET Silverlight               |
| BytesMessage                                  | KMBytesMessage                                                        | Objective-C                   |
| MapMessage                                    | KMMapMessage                                                          | Objective-C                   |
| QueueConnection                               | KMConnection                                                          | Objective-C                   |
| QueueConnectionFactory                        | KMConnectionFactory                                                   | Objective-C                   |
| QueueReceiver                                 | KMMessageConsumer                                                     | Objective-C                   |
| QueueRequestor                                | KMTemporaryQueue                                                      | Objective-C                   |
| QueueSender                                   | KMMessageProducer                                                     | Objective-C                   |
| QueueSession                                  | KMSession                                                             | Objective-C                   |
| TextMessage                                   | KMTextMessage                                                         | Objective-C                   |
| TopicConnection                               | KMConnection                                                          | Objective-C                   |
| TopicConnectionFactory                        | KMConnectionFactory                                                   | Objective-C                   |
| TopicPublisher                                | KMMessageProducer                                                     | Objective-C                   |
| TopicRequestor                                | KMTemporaryTopic                                                      | Objective-C                   |
| TopicSession                                  | KMSession                                                             | Objective-C                   |
