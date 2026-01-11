# Optimization Pipeline

## 1. Overview

The optimization pipeline transforms the AST to improve runtime performance without changing behavior.

```rust
pub trait OptimizationPass {
    fn optimize(&self, ast: &mut TypedAST) -> Result<()>;
}
```

## 2. Standard Passes

### 2.1 Loop Unrolling

Unrolls small loops with static bounds to reduce branch overhead.

```rust
// Before
for i in 0..4 {
    process(i);
}

// After
process(0);
process(1);
process(2);
process(3);
```

**Criteria:**
- Loop bounds must be known at compile time.
- Loop body cost * iterations must be below threshold.
- No complex control flow (break/continue) inside.

### 2.2 Strength Reduction

Replaces expensive operations with cheaper equivalents.

** Examples:**
- `x * 2` → `x << 1` (Multiplication to Shift)
- `x / 4` → `x >> 2` (Division to Shift)
- `x % 2` → `x & 1` (Modulo to AND)
- `Math.pow(x, 2)` → `x * x`

### 2.3 Constant Folding

Evaluates constant expressions at compile time.

- `2 + 3` → `5`
- `true && false` → `false`
