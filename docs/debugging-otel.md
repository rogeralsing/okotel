
# Example Otel setup

```csharp
var otlpEndpoint = configuration.GetValue<string>("Otlp:Endpoint");
Console.WriteLine($"Using OpenTelemetry Endpoint:{otlpEndpoint}");

// allows fallback to http for exporter <-- this is needed pre .net 6
// otherwise ignore this line:
AppContext.SetSwitch("System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);

services.AddOpenTelemetryTracing((builder) =>
{
    builder
        .SetResourceBuilder(ResourceBuilder.CreateDefault()
        .AddService("MySystem"))
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddGrpcClientInstrumentation()
        .AddSqlClientInstrumentation(options =>
        {
            options.SetDbStatementForText = true;
            options.SetDbStatementForStoredProcedure = true;
        });

        var otlpOptionsEndpoint = new Uri(otlpEndpoint);

        builder.AddOtlpExporter(otlpOptions =>
        {
            otlpOptions.Endpoint = otlpOptionsEndpoint;
        });
});
```        

## Custom Tracing payloads:

### Tags

Otel Tags are annotations associated with a span. They are key value pairs. They can be indexed, this depends on what tracing tool is being used and how it is configured.
e.g. Grafana Tempo and DataDog can both ingest and index tags.

```csharp
//set a tag
Activity.Current?.AddTag("ocpp.payload", fromChargePoint.Payload);
```

### Events

Otel Events are similar to logs, but associated with a span and ingested by the tracing tool. They are not indexed.
If you are using logging that is associated with the traces, e.g. via TraceId and SpanId, logging is likely a better choise than Events.

Some 3rd party libs, e.g. Redis, uses events to log/annotate spans whenever a call to Redis is being made.


```csharp
//add an event
//events are just names + timestamp
Activity.Current?.AddEvent(new ActivityEvent("eventname", DateTimeOffset.UtcNow));
```

### Baggage

Otel Baggate is an interesting concept similar to HTTP headers. It is a key value pair that is associated with a span and is carried between spans.
Meaning you can carry data from one service over to multiple other services.

```csharp
//add baggage
//baggage is like http headers and are carried between spans. and can be read via GetBaggageItem
Activity.Current?.AddBaggage("ocpp.payload", fromChargePoint.Payload);
```


## Debugging OTEL.

If you have set all the above up. but still are not seeing any traces produced. you can debug Otel to see what is wrong.

Add: `OTEL_DIAGNOSTICS.json` "copy always" to the solution.
This enables logging inside otel libs
```
{
  "LogDirectory": ".",
  "FileSize": 1024,
  "LogLevel": "Error"
}
```

This will produce a plain text log in the app directory that can be used to debug otel.

