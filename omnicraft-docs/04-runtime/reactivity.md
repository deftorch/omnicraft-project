# Reactive System Design

## 1. Overview

### 1.1 Reactivity Goals

1. **Fine-grained updates** - Only update what changed
2. **Automatic tracking** - No manual subscriptions
3. **Efficient batching** - Group updates to avoid thrashing
4. **Memory safe** - No memory leaks from subscriptions

## 2. Core Primitives

### 2.1 Signal Design

```rust
// runtime/src/reactive/signal.rs

use std::cell::RefCell;
use std::rc::Rc;

pub struct Signal<T> {
    value: Rc<RefCell<T>>,
    subscribers: Rc<RefCell<Vec<EffectId>>>,
}

impl<T: Clone + 'static> Signal<T> {
    pub fn new(initial: T) -> Self {
        Self {
            value: Rc::new(RefCell::new(initial)),
            subscribers: Rc::new(RefCell::new(Vec::new())),
        }
    }
    
    pub fn get(&self) -> T {
        // Track this read in current effect
        if let Some(effect_id) = get_current_effect() {
            self.subscribers.borrow_mut().push(effect_id);
        }
        self.value.borrow().clone()
    }
    
    pub fn set(&self, new_value: T) {
        *self.value.borrow_mut() = new_value;
        self.notify();
    }
    
    fn notify(&self) {
        // Batch updates to avoid duplicate work
        BATCH_QUEUE.with(|queue| {
            let mut queue = queue.borrow_mut();
            
            for effect in self.subscribers.borrow().iter() {
                queue.insert(*effect);
            }
        });
        
        // Flush batch if not already batching
        if !BATCHING.with(|b| *b.borrow()) {
            flush_batch();
        }
    }
}
```

### 2.2 Computed Values

```rust
// runtime/src/reactive/computed.rs

pub struct Computed<T> {
    compute: Rc<dyn Fn() -> T>,
    cached: Rc<RefCell<Option<T>>>,
    dependencies: Rc<RefCell<Vec<SignalId>>>,
}

impl<T: Clone + 'static> Computed<T> {
    pub fn new(compute: impl Fn() -> T + 'static) -> Self {
        Self {
            compute: Rc::new(compute),
            cached: Rc::new(RefCell::new(None)),
            dependencies: Rc::new(RefCell::new(Vec::new())),
        }
    }
    
    pub fn get(&self) -> T {
        // Check cache
        if let Some(cached) = self.cached.borrow().as_ref() {
            return cached.clone();
        }
        
        // Compute value
        let value = (self.compute)();
        
        // Cache value
        *self.cached.borrow_mut() = Some(value.clone());
        
        value
    }
    
    pub fn invalidate(&self) {
        *self.cached.borrow_mut() = None;
    }
}
```

### 2.3 Effects & Batching

```rust
pub fn create_effect(f: impl Fn() + 'static) -> EffectId {
    let effect_id = allocate_effect_id();
    
    // Run effect in tracking context
    with_effect_context(effect_id, || {
        f();
    });
    
    effect_id
}

pub fn batch(f: impl FnOnce()) {
    BATCHING.with(|b| {
        let was_batching = *b.borrow();
        *b.borrow_mut() = true;
        
        f();
        
        if !was_batching {
            flush_batch();
            *b.borrow_mut() = false;
        }
    });
}
```
