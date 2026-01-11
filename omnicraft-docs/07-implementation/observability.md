# Observability

OmniCraft integrates comprehensive observability features, including structured logging and metrics collection, to monitor compiler performance and runtime behavior.

## Structured Logging

The project uses the `tracing` crate for structured, async-aware logging. This allows for detailed diagnostics and compilation traces.

### Configuration

Logging can be configured via environment variables or CLI flags (e.g., `RUST_LOG`). Logs can be emitted in various formats, including compact text for terminals and JSON for ingestion by log functionality.

```rust
use tracing::{info, warn, error, instrument};

#[instrument(skip(source))]
pub fn compile_module(name: &str, source: &str) -> Result<Module, Error> {
    info!(target: "compiler", module = %name, "Starting compilation");
    
    // ...
    
    if let Err(e) = result {
        error!(target: "compiler", error = ?e, "Compilation failed");
        return Err(e);
    }
    
    info!(target: "compiler", "Compilation finished");
    Ok(module)
}
```

## Metrics Collection

Metrics are collected using the `prometheus` crate, exposing insights into compiler performance and resource usage.

### Key Metrics

1.  **Counter**: Tracking total compilation requests, errors, and warnings.
2.  **Gauge**: Monitoring current memory usage, thread count, and active compilation jobs.
3.  **Histogram**: Measuring distribution of compilation times, optimization pass durations, and I/O latency.

```rust
lazy_static! {
    pub static ref COMPILATION_DURATION: Histogram = register_histogram!(
        "omnicraft_compilation_duration_seconds",
        "Time taken to compile a module"
    ).unwrap();
    
    pub static ref ERROR_COUNT: CounterVec = register_counter_vec!(
        "omnicraft_errors_total",
        "Total number of compilation errors",
        &["type"]
    ).unwrap();
}
```

### Telemetry Export

Metrics can be exported to a Prometheus-compatible endpoint. The CLI command `omnicraft doctor` or `omnicraft analyze` can also utilize this data to provide local insights.
