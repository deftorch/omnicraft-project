# Language Server Protocol (LSP) Implementation

## 1. Overview

The OmniCraft LSP server provides IDE intelligence including auto-completion, hover information, and real-time diagnostics.

### 1.1 Architecture

```rust
pub struct OmniCraftLanguageServer {
    client: Client,
    documents: Arc<RwLock<HashMap<Url, Document>>>,
    compiler: Arc<RwLock<Compiler>>,
}

struct Document {
    uri: Url,
    text: String,
    version: i32,
    ast: Option<Component>,
    diagnostics: Vec<Diagnostic>,
    type_info: Option<TypeInfo>,
}
```

## 2. Features

### 2.1 Auto-Completion

The server offers context-aware completion for:
- **Tags**: `<circle>`, `<rect>`, etc.
- **Attributes**: `x`, `y`, `fill` (typed).
- **Signals**: Reactive signal names.
- **Functions**: Component methods.

### 2.2 Hover Information

Hovering over symbols provides rich documentation:
- **Signals**: Shows type and reactive nature.
- **Components**: Shows available props and docs.

```typescript
// Example Hover Output
Symbol::Signal { name, type_ } => {
    format!(
        "```omnicraft\nconst {}: {}\n```\n\n📡 **Reactive Signal**\n\nAutomatically updates components when value changes.",
        name,
        type_.to_string()
    )
}
```

### 2.3 Diagnostics

Real-time error checking with `publishDiagnostics` capability, mapping compiler errors to editor squiggles.

## 3. Implementation Details

The server uses `tower-lsp` and shares the core compiler crate for parsing and analysis, ensuring perfect parity between the build process and IDE feedback.
