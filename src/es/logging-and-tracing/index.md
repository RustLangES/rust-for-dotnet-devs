# Logging y Tracing

.NET admite varias API de logging. Para la mayoría de los casos, `ILogger` es 
una buena opción predeterminada, ya que funciona con una variedad de proveedores 
de registro integrados y de terceros. En C#, un ejemplo mínimo para registro 
estructurado podría lucir así:

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(builder => builder.AddConsole());
var logger = loggerFactory.CreateLogger<Program>();
logger.LogInformation("Hola {Day}.", "Jueves"); // Hola Jueves.
```

En Rust, se proporciona una fachada de logging ligera a través de [log][log.rs]. 
Tiene menos características que `ILogger`, por ejemplo, aún no ofrece (de manera 
estable) registro estructurado o ámbitos de registro.

Para algo con una paridad de características más cercana a .NET, Tokio ofrece 
[`tracing`][tracing.rs]. `tracing` es un framework para instrumentar 
aplicaciones Rust para recopilar información de diagnóstico estructurada y 
basada en eventos. [`tracing_subscriber`][tracing-subscriber.rs] se puede 
utilizar para implementar y componer suscriptores de `tracing`. El mismo ejemplo 
de registro estructurado anterior con `tracing` y `tracing_subscriber` se vería 
así:

```rust
fn main() {
    // Instalar el recolector global de mensajes de ("consola").
    tracing_subscriber::fmt().init();
    tracing::info!("Hola {Day}.", Day = "Jueves"); // Hola Jueves.
}
```

[OpenTelemetry][opentelemetry.rs] ofrece una colección de herramientas, APIs y 
SDKs utilizados para instrumentar, generar, recopilar y exportar datos de 
telemetría basados en la especificación de OpenTelemetry. En el momento de 
escribir esto, la [API de registro de OpenTelemetry][opentelemetry-logging] aún 
no es estable y la implementación de Rust 
[todavía no soporta el registro][opentelemetry-status.rs], pero sí soporta la 
API de rastreo.

[opentelemetry.rs]: https://crates.io/crates/opentelemetry
[tracing-subscriber.rs]: https://docs.rs/tracing-subscriber/latest/tracing_subscriber/
[opentelemetry-logging]: https://opentelemetry.io/docs/reference/specification/status/#logging
[opentelemetry-status.rs]: https://opentelemetry.io/docs/instrumentation/rust/#status-and-releases
[tracing.rs]: https://crates.io/crates/tracing
[log.rs]: https://crates.io/crates/log
