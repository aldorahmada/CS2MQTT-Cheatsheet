# MQTT on C# using MQTTnet

### Used Library & Extension
- MQTTnet
- MQTTnet.Extensions.ManagedClient
- Serilog
- Serilog.Sinks.TextWritter


### 1. Client - Broker Relationship
```

```

### 2. Making Broker
```
public StringWriter _messages;
public void main()
        {
            _messages = new StringWriter();
            Log.Logger = new LoggerConfiguration()
                .WriteTo.TextWriter(_messages)
                .CreateLogger();

            MqttServerOptionsBuilder options = new MqttServerOptionsBuilder()
                .WithDefaultEndpoint()
                .WithDefaultEndpointPort(707)
                .WithConnectionValidator(OnNewConnection)
                .WithApplicationMessageInterceptor(OnNewMessage);

            IMqttServer mqttServer = new MqttFactory().CreateMqttServer();

            mqttServer.StartAsync(options.Build()).GetAwaiter().GetResult();
        }
```
### 2.1 Broker New Client Validation Event
```
public static void OnNewConnection(MqttConnectionValidatorContext context)
{
    Log.Logger.Information(
            "New connection: ClientId = {clientId}, Endpoint = {endpoint}",
            context.ClientId,
            context.Endpoint);
}
```
### 2.2 Broker New Message Event 
```
public static void OnNewMessage(MqttApplicationMessageInterceptorContext context)
{
    var payload = context.ApplicationMessage?.Payload == null ? null : Encoding.UTF8.GetString(context.ApplicationMessage?.Payload);

    MessageCounter++;

    Log.Logger.Information(
        "MessageId: {MessageCounter} - TimeStamp: {TimeStamp} -- Message: ClientId = {clientId}, Topic = {topic}, Payload = {payload}, QoS = {qos}, Retain-Flag =     {retainFlag}",
        MessageCounter,
        DateTime.Now,
        context.ClientId,
        context.ApplicationMessage?.Topic,
        payload,
        context.ApplicationMessage?.QualityOfServiceLevel,
        context.ApplicationMessage?.Retain);
}
```

### 3. Making Client
```
public void main()
{
    //Serilog Datalog Initialization
    _messages = new StringWriter();
    Log.Logger = new LoggerConfiguration()
        .WriteTo.TextWriter(_messages)
        .CreateLogger();

    MqttClientOptionsBuilder builder = new MqttClientOptionsBuilder()
                                .WithClientId("Client1")//Define Clientid
                                .WithTcpServer("localhost", 707);//Define IP and port

    ManagedMqttClientOptions options = new ManagedMqttClientOptionsBuilder()
                            .WithAutoReconnectDelay(TimeSpan.FromSeconds(60))
                            .WithClientOptions(builder.Build())
                            .Build();

    _mqttClient.ConnectedHandler = new MqttClientConnectedHandlerDelegate(OnConnected);
    _mqttClient.DisconnectedHandler = new MqttClientDisconnectedHandlerDelegate(OnDisconnected);
    _mqttClient.ConnectingFailedHandler = new ConnectingFailedHandlerDelegate(OnConnectingFailed);

    _mqttClient.ApplicationMessageReceivedHandler = new MqttApplicationMessageReceivedHandlerDelegate(a => {
        Log.Logger.Information("Message recieved: {payload}", a.ApplicationMessage);
    });
    _mqttClient.StartAsync(options).GetAwaiter().GetResult();
    _mqttClient.SubscribeAsync("dev.to/topic/json2"); //Subcribe Other Client
    _mqttClient.UseApplicationMessageReceivedHandler(a =>
    {
        Log.Logger.Information("Message recieved: {payload}", Encoding.UTF8.GetString(a.ApplicationMessage.Payload));//Incoming Message Format
    });
}
```
### 4. Subscribe on Broker
```
```

### 5. Subscribe on Client
```
```

### 6. Publish on Client
```
```
