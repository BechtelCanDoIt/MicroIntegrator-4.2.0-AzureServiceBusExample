# MicroIntegrator-4.2.0-AzureServiceBus-Example
Example configurations to publish to an Azure Service Bus (ASB) Queue or Topic and then consume it using an InboundEndpoint

## Micro Integrator Nuance
It seems that between EI and MI there have been some changes that need to be updated in the current ASB examples in the WSO2 documentation.

A primary aspect of this change is the migration of updating the `jndi.properties` file directly to the `deployment.toml` auto generation in the `jndi.properties` file. This has the impact of needing to adjust some of the naming.

### Notes
#### Sample Subscription for a Topic
For a queue you use the queue name, but for topics you need to subscribe to a [subscription](https://learn.microsoft.com/en-us/azure/templates/microsoft.servicebus/namespaces/topics/subscriptions?pivots=deployment-language-bicep) of the topic. That subscription is setup in the ASB console.

Format: topic name/**subscriptions**/subscription name

Example: `<parameter name="transport.jms.Destination">topic.test/**subscriptions**/subscription.topic.test</parameter>`

#### Protocol/Naming Cross Referencing
![ASB Sample Protocol Names](images/ASB%20Sample%20Protocol%20Names.png)

In these suggested configuration the API connection utilizes the name in the transport.jms.sender section of the deployment.toml. Those settings are placced in either the `<MI_HOME>/conf/axis2/axis2.xml` or `<MI_HOME>/conf/axis2/axis2_blocking_client.xml` depending on which configuration you set up.

In these examples the InboundEndpoints are utilizing the `<MI_HOME>/conf/jndi.properties` file connection factory settings instead.

WARNING: Now that JNDI Connection Factories values are defined you can no longer utilize the older connectionfactory.SBCF / SBTCF naming patterns some have suggested for the EI configurations.

#### ASB Certificate Import
Reminder to import the ASB certificate into your client-trustore.jks file

#### Deployment.toml is now KING!
Do not modify `jndi.properties` nor `axis2.xml` files manually. In MI those will be overwritten when the server is started up.

### Sample Deployment.toml
```
[transport.jms]
sender_enable = true
listener_enable = true
 
[transport.jndi.connection_factories]
'connectionfactory.QueueConnectionFactory' = "amqps://sample-namespace.servicebus.windows.net?amqp.idleTimeout=-1&jms.prefetchPolicy.all=0&jms.username=test&jms.password=***"
'connectionfactory.TopicConnectionFactory' = "amqps://sample-namespace.windows.net?amqp.idleTimeout=-1&jms.prefetchPolicy.all=0&jms.username=test&jms.password=***"

[[transport.jms.sender]]
name = "azureQueueProducerConnectionFactory"
parameter.cache_level = "connection"
parameter.connection_factory_name = "QueueConnectionFactory"
parameter.connection_factory_type = "queue"
parameter.initial_naming_factory = "org.apache.qpid.jms.jndi.JmsInitialContextFactory"
parameter.provider_url = "conf/jndi.properties"
parameter.'transport.jms.MessagePropertyHyphens' = "replace"

[[transport.jms.sender]]
name = "azureTopicProducerConnectionFactory"
parameter.cache_level = "connection"
parameter.connection_factory_name = "TopicConnectionFactory"
parameter.connection_factory_type = "topic"
parameter.initial_naming_factory = "org.apache.qpid.jms.jndi.JmsInitialContextFactory"
parameter.provider_url = "conf/jndi.properties"
parameter.'transport.jms.MessagePropertyHyphens' = "replace"

[[transport.blocking.jms.sender]]
name = "azureQueueProducerConnectionFactory"
parameter.cache_level = "connection"
parameter.connection_factory_name = "QueueConnectionFactory"
parameter.connection_factory_type = "queue"
parameter.initial_naming_factory = "org.apache.qpid.jms.jndi.JmsInitialContextFactory"
parameter.provider_url = "conf/jndi.properties"
parameter.'transport.jms.MessagePropertyHyphens' = "replace"

[[transport.blocking.jms.sender]]
name = "azureTopicProducerConnectionFactory"
parameter.cache_level = "connection"
parameter.connection_factory_name = "TopicConnectionFactory"
parameter.connection_factory_type = "topic"
parameter.initial_naming_factory = "org.apache.qpid.jms.jndi.JmsInitialContextFactory"
parameter.provider_url = "conf/jndi.properties"
parameter.'transport.jms.MessagePropertyHyphens' = "replace"
```

### Example Synapse Code
See `src/main/wso2mi/artifacts/` folder


### Example Log Output
```
[2025-02-25 17:58:45,517]  INFO {LogMediator} - {api:AzureServiceBusQueuePublisher} To: /publishToQueue/sendTopic
MessageID: urn:uuid:d23970ae-771a-4928-8547-93851592c0e2
correlation_id: d23970ae-771a-4928-8547-93851592c0e2
Direction: request

DEBUG = ++++ API sending to TOPIC
Envelope: <?xml version='1.0' encoding='utf-8'?><soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"><soapenv:Body><test>hi</test></soapenv:Body></soapenv:Envelope>
…
DEBUG = ++++ API sent msg to TOPIC
…
[2025-02-25 17:58:46,770]  INFO {LogMediator} - {inboundendpoint:InboundASBTopicConsumer} To:
MessageID: ID:fd68a06e-6f73-4a53-b826-d6443614b187:1:1:1-1
Direction: request
DEBUG = ---- Extracted message from ASB Topic
Envelope: <?xml version='1.0' encoding='utf-8'?><soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"><soapenv:Body><test>hi</test></soapenv:Body></soapenv:Envelope>
```

# Documentation

## WSO2 Docs
- [Connecting to Azure Service Bus](https://apim.docs.wso2.com/en/4.2.0/install-and-setup/setup/mi-setup/brokers/configure-with-azureservicebus/)
- [JMS Transports](https://apim.docs.wso2.com/en/4.2.0/install-and-setup/setup/mi-setup/transport_configurations/configuring-transports/#configuring-the-jms-transport)
- [JMS Transport Listner](https://apim.docs.wso2.com/en/4.2.0/reference/config-catalog-mi/#jms-transport-listener-non-blocking-mode) (see: non-blocking / blocking versions)
- [JMS Transport Sender](https://apim.docs.wso2.com/en/4.2.0/reference/config-catalog-mi/#jms-transport-sender-non-blocking-mode) (see: non-blocking / blocking versions)
- [JNDI Connection Factories](https://apim.docs.wso2.com/en/4.2.0/reference/config-catalog-mi/#jndi-connection-factories)

## Helpful Blogs

- [Integrating Azure Service Bus with WSO2 Micro Integrator](https://medium.com/@pasindugunarathna34/integrating-azure-service-bus-with-wso2-micro-integrator-6c2eb2a69a57)
- [Azure Service Bus Queue Integration with WSO2 EI/MI -Azure Service Bus Producer](https://medium.com/@cvr4314/azure-service-bus-queue-integration-with-wso2-ei-mi-azure-service-bus-producer-f25b21e16a59)
- [Integrating Microsoft Azure Service Bus with WSO2 Enterprise Integrator](https://medium.com/@shefandarren.173/integrating-microsoft-azure-service-bus-with-wso2-enterprise-integrator-b504895ccfe6)


