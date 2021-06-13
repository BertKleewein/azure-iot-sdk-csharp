## Plug and Play convention compatible APIs

#### Common

```diff
public class ClientOptions {
+    public ClientOptions(PayloadConvention payloadConvention = null);
+    public PayloadConvention PayloadConvention { get; }
}

public class DeviceClient : IDisposable {
+    public PayloadConvention PayloadConvention { get; }
}

public class ModuleClient : IDisposable {
+    public PayloadConvention PayloadConvention { get; }
}
```

```csharp

public abstract class PayloadConvention {
    protected PayloadConvention();
    public abstract PayloadEncoder PayloadEncoder { get; }
    public abstract PayloadSerializer PayloadSerializer { get; }
    public virtual byte[] GetObjectBytes(object objectToSendWithConvention);
}

public abstract class PayloadEncoder {
    protected PayloadEncoder();
    public abstract Encoding ContentEncoding { get; }
    public abstract byte[] EncodeStringToByteArray(string contentPayload);
}

public abstract class PayloadSerializer {
    protected PayloadSerializer();
    public abstract string ContentType { get; }
    public abstract T ConvertFromObject<T>(object objectToConvert);
    public abstract IWritablePropertyResponse CreateWritablePropertyResponse(object value, int statusCode, long version, string description = null);
    public abstract T DeserializeToType<T>(string stringToDeserialize);
    public abstract string SerializeToString(object objectToSerialize);
    public abstract bool TryGetNestedObjectValue<T>(object nestedObject, string propertyName, out T outValue);
}

public sealed class DefaultPayloadConvention : PayloadConvention {
    public static readonly DefaultPayloadConvention Instance;
    public DefaultPayloadConvention();
    public override PayloadEncoder PayloadEncoder { get; }
    public override PayloadSerializer PayloadSerializer { get; }
}

public class Utf8PayloadEncoder : PayloadEncoder {
    public static readonly Utf8PayloadEncoder Instance;
    public Utf8PayloadEncoder();
    public override Encoding ContentEncoding { get; }
    public override byte[] EncodeStringToByteArray(string contentPayload);
}

public class NewtonsoftJsonPayloadSerializer : PayloadSerializer {
    public static readonly NewtonsoftJsonPayloadSerializer Instance;
    public NewtonsoftJsonPayloadSerializer();
    public override string ContentType { get; }
    public override T ConvertFromObject<T>(object objectToConvert);
    public override IWritablePropertyResponse CreateWritablePropertyResponse(object value, int statusCode, long version, string description = null);
    public override T DeserializeToType<T>(string stringToDeserialize);
    public override string SerializeToString(object objectToSerialize);
    public override bool TryGetNestedObjectValue<T>(object nestedObject, string propertyName, out T outValue);
}

public abstract class PayloadCollection : IEnumerable, IEnumerable<KeyValuePair<string, object>> {
    protected PayloadCollection();
    public IDictionary<string, object> Collection { get; private set; }
    public PayloadConvention Convention { get; internal set; }
    public virtual object this[string key] { get; set; }
    public virtual void Add(string key, object value);
    public virtual void AddOrUpdate(string key, object value);
    public void ClearCollection();
    public bool Contains(string key);
    public IEnumerator<KeyValuePair<string, object>> GetEnumerator();
    public virtual byte[] GetPayloadObjectBytes();
    public virtual string GetSerializedString();
    protected void SetCollection(PayloadCollection payloadCollection);
    IEnumerator System.Collections.IEnumerable.GetEnumerator();
    public bool TryGetValue<T>(string key, out T value);
}

public static class ConventionBasedConstants {
    public const char ComponentLevelCommandSeparator = '*';
    public const string AckCodePropertyName = "ac";
    public const string AckDescriptionPropertyName = "ad";
    public const string AckVersionPropertyName = "av";
    public const string ComponentIdentifierKey = "__t";
    public const string ComponentIdentifierValue = "c";
    public const string ValuePropertyName = "value";
}

public class CommonClientResponseCodes {
    public const int Accepted = 202;
    public const int BadRequest = 400;
    public const int NotFound = 404;
    public const int OK = 200;
    public CommonClientResponseCodes();
}
```

### Properties

```csharp
/// <summary>
/// Retrieve the device properties.
/// </summary>
/// <param name="cancellationToken">A cancellation token to cancel the operation.</param>
/// <returns>The device properties.</returns>
public Task<ClientProperties> GetClientPropertiesAsync(CancellationToken cancellationToken = default);

/// <summary>
/// Update properties.
/// </summary>
/// <param name="propertyCollection">Reported properties to push.</param>
/// <param name="cancellationToken">A cancellation token to cancel the operation.</param>
/// <returns>The response containing the operation request Id and updated version no.</returns>
public Task<ClientPropertiesUpdateResponse> UpdateClientPropertiesAsync(ClientProperties properties, CancellationToken cancellationToken = default);

/// <summary>
/// Sets the global listener for Writable properties
/// </summary>
/// <param name="callback">The global call back to handle all writable property updates.</param>
/// <param name="userContext">Generic parameter to be interpreted by the client code.</param>
/// <param name="cancellationToken">A cancellation token to cancel the operation.</param>
public Task SubscribeToWritablePropertiesEventAsync(Func<ClientProperties, object, Task> callback, object userContext, CancellationToken cancellationToken = default);
```

#### All related types

```csharp
public class ClientProperties : ClientPropertyCollection {
    public ClientProperties();
    public ClientComponentCollection Components { get; private set; }
}

public class ClientPropertyCollection : IEnumerable, IEnumerable<KeyValuePair<string, object>> {
    public ClientPropertyCollection();
    public long Version { get; protected set; }
    public IDictionary<string, object> Collection { get; private set; }
    public void AddProperty(string propertyName, object value);
    public void AddProperty(IDictionary<string, object> properties);
    public void AddOrUpdateProperty(IDictionary<string, object> properties);
    public bool ContainsProperty(string propertyName);
    public virtual bool TryGetPropertyValue<T>(string propertyName, out T propertyValue);
}

public class ClientComponentCollection : IEnumerable, IEnumerable<KeyValuePair<string, ClientComponent>> {
    public ClientCompoentCollection()
    public IDictionary<string, ClientComponent> Collection { get; private set; }
    public void AddComponent(string componentName, ClientComponent value);
    public void AddComponent(IDictionary<string, ClientComponent> components);
    public void AddOrUpdateComponent(IDictionary<string, ClientComponent> components);
    public bool ContainsComponent(string componentName);
    public virtual bool TryGetComponentValue(string componentName, out ClientComponent component);
}

public class ClientComponent : ClientPropertyCollection {
    public ClientComponent();
}

public class ClientWritableProperty {
    public ClientWritableProperty();
    public object Value;
    public IWritablePropertyResponse Response;
}

public interface IWritablePropertyResponse {
    int AckCode { get; set; }
    string AckDescription { get; set; }
    long AckVersion { get; set; }
    object Value { get; set; }
}

public sealed class NewtonsoftJsonWritablePropertyResponse : IWritablePropertyResponse {
    public NewtonsoftJsonWritablePropertyResponse(object propertyValue, int ackCode, long ackVersion, string ackDescription = null);
    public int AckCode { get; set; }
    public string AckDescription { get; set; }
    public long AckVersion { get; set; }
    public object Value { get; set; }
}

public class ClientPropertiesUpdateResponse {
    public ClientPropertiesUpdateResponse();
    public string RequestId { get; internal set; }
    public long Version { get; internal set; }
}
```

### Telemetry

```csharp
/// <summary>
/// Send telemetry using the specified message.
/// </summary>
/// <remarks>
/// Use the <see cref="TelemetryMessage(string, TelemetryCollection)"/> constructor to pass in the optional
/// <see cref="TelemetryCollection"/> that specifies your payload and serialization and encoding rules.
/// </remarks>
/// <param name="telemetryMessage">The telemetry message.</param>
/// <param name="cancellationToken">A cancellation token to cancel the operation.</param>
public Task SendTelemetryAsync(TelemetryMessage telemetryMessage, CancellationToken cancellationToken = default);
```
#### All related types

```csharp
public class TelemetryCollection : PayloadCollection {
    public TelemetryCollection();
    public override void Add(string telemetryName, object telemetryValue);
    public override void AddOrUpdate(string telemetryName, object telemetryValue);
}

public sealed class TelemetryMessage : MessageBase {
    public TelemetryMessage(string componentName = null);
    public TelemetryCollection Telemetry { get; set; }
    public override Stream GetBodyStream();
}
```

### Commands

```csharp
/// <summary>
/// Set the global command callback handler.
/// </summary>
/// <param name="callback">A method implementation that will handle the incoming command.</param>
/// <param name="userContext">Generic parameter to be interpreted by the client code.</param>
/// <param name="cancellationToken">A cancellation token to cancel the operation.</param>
public Task SubscribeToCommandsAsync(Func<CommandRequest, object, Task<CommandResponse>> callback, object userContext, CancellationToken cancellationToken = default);
```
#### All related types

```csharp
public sealed class CommandRequest {
    public string CommandName { get; private set; }
    public string ComponentName { get; private set; }
    public string DataAsJson { get; }
    public T GetData<T>();
    public byte[] GetDataAsBytes();
}

public sealed class CommandResponse {
    public CommandResponse(int status);
    public CommandResponse(object result, int status);
    public string ResultAsJson { get; }
    public int Status { get; private set; }
}
```