# Runtime Performance Monitoring

OmniCraft provides built-in tools for monitoring the performance of applications at runtime. This includes tracking frame rates, memory usage, and entity counts.

## Performance Metrics

The runtime exposes a `PerformanceMonitor` system that tracks key metrics:

-   **FPS (Frames Per Second)**: Instantaneous and averaged frame rates.
-   **Frame Time**: The time taken to process and render a single frame (in milliseconds).
-   **Memory Usage**: Heap usage (if available via browser APIs or runtime hooks).
-   **Entity Count**: The total number of active ECS entities.

### FPS Component

A dedicated `FPSComponent` can be added to the scene to display an on-screen overlay of performance stats.

```rust
// Rust usage example
app.add_system(fps_monitor_system);
app.add_plugin(DiagnosticsPlugin::default());
```

```typescript
// JavaScript/TypeScript usage
import { PerformanceMonitor } from '@omnicraft/runtime';

const monitor = new PerformanceMonitor();
monitor.attach(document.body);
```

### System Info

The monitoring tools also capture system information to help with debugging performance issues on different hardware:

-   **User Agent**
-   **Logical Cores**
-   **GPU Renderer** (via WebGL/WebGPU context)

## Integration with DevTools

These runtime metrics are synchronized with the OmniCraft DevTools extension, allowing developers to view real-time performance graphs alongside the component tree.
