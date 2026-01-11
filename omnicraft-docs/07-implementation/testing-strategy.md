# Testing Strategy

## 1. Testing Framework

### 1.1 Architecture

OmniCraft includes a built-in test runner designed for component testing.

```rust
pub struct TestContext {
    component: Box<dyn Component>,
    renderer: TestRenderer,
    signals: HashMap<String, Box<dyn Any>>,
    snapshots: Vec<Snapshot>,
    events: Vec<TestEvent>,
}
```

### 1.2 Visual Regression Testing

The framework supports visual snapshots to detect unintended UI changes.

```rust
ctx.snapshot();
ctx.assert_snapshot_matches("counter-incremented").unwrap();
```

### 1.3 Interaction Testing

Simulate user interactions programmatically.

```rust
// Click at coordinates
ctx.click(500.0, 425.0);

// Wait for conditions
ctx.wait_for(
    |c| c.get_signal::<f32>("rotation").unwrap_or(0.0) >= 360.0,
    5000
).unwrap();
```

## 2. Example Test

```rust
#[omnicraft_test]
async fn test_counter_increment() {
    let mut ctx = TestContext::new(800, 600);
    ctx.mount(CounterComponent::new());
    
    // Check initial state
    assert_eq!(ctx.get_signal::<i32>("count"), Some(0));
    
    // Interact
    ctx.click(500.0, 425.0);
    
    // Verify state
    assert_eq!(ctx.get_signal::<i32>("count"), Some(1));
}
```
