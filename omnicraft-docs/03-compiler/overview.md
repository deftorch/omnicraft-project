# Compiler Overview

The compiler is the heart of OmniCraft, transforming high-level `.omni` code into highly efficient runtime instructions.

## 1. Compiler Design

The compiler follows a traditional pipeline architecture, enhanced with caching for performance.

### 1.1 Compiler Pipeline

```rust
pub struct Compiler {
    lexer: Lexer,
    parser: Parser,
    type_checker: TypeChecker,
    optimizer: Optimizer,
    codegen: CodeGenerator,
    cache: CompilationCache,
    diagnostics: DiagnosticEngine,
}

impl Compiler {
    pub fn compile(&mut self, source: &str, file_path: &Path) -> Result<Output> {
        // Check cache first
        let hash = calculate_hash(source);
        if let Some(cached) = self.cache.get(file_path, &hash) {
            return Ok(cached);
        }
        
        // Compilation pipeline
        let tokens = self.lexer.tokenize(source)?;
        let ast = self.parser.parse(tokens)?;
        let typed_ast = self.type_checker.check(ast)?;
        
        // Collect diagnostics
        if self.diagnostics.has_errors() {
            return Err(anyhow!("Compilation failed"));
        }
        
        let optimized = self.optimizer.optimize(typed_ast)?;
        let code = self.codegen.generate(optimized)?;
        
        // Cache result
        self.cache.insert(file_path, hash, code.clone());
        
        Ok(code)
    }
}
```

### 1.2 Lexical Analysis

The lexer converts source text into tokens.

```rust
pub enum Token {
    Script, Canvas, Const, Let, Function, // Keywords
    Number(f64), String(String), Boolean(bool), // Literals
    Identifier(String),
    Plus, Minus, Star, Slash, Equal, // Operators
    LeftBrace, RightBrace, LeftAngle, RightAngle, // Delimiters
}
```

### 1.3 Parser Design

The parser builds an Abstract Syntax Tree (AST).

```rust
pub struct Component {
    pub name: String,
    pub script: Option<ScriptBlock>,
    pub template: TemplateBlock,
    pub style: Option<StyleBlock>,
    pub metadata: ComponentMetadata,
}
```

### 1.4 Optimizer Design

Optimization passes improve the generated code.

```rust
pub trait OptimizationPass {
    fn optimize(&self, ast: &mut TypedAST) -> Result<()>;
}

// Example passes: ConstantFolding, DeadCodeElimination, LoopUnrolling
```

## 2. Type System

The type system ensures safety and correctness before runtime.

### 2.1 Type Representation

```rust
pub enum Type {
    Number, String, Boolean,
    Array(Box<Type>),
    Function { params: Vec<Type>, return_type: Box<Type> },
    Signal(Box<Type>),
    Union(Vec<Type>),
    Intersection(Vec<Type>),
    Generic { name: String, constraints: Vec<Type> },
    Unknown,
}
```

### 2.2 Type Inference

The type system uses a constraint-based inference algorithm to determine types where not explicitly annotated, reducing developer burden while maintaining safety.

## 3. Incremental Compilation

Incremental compilation is key to a fast developer experience, rebuilding only what changed.

### 3.1 Architecture

The incremental compiler tracks file dependencies and hashes to avoid unnecessary work.

```rust
pub struct IncrementalCompiler {
    cache: CompilationCache,
    dependency_graph: DependencyGraph,
    file_watcher: FileWatcher,
}

pub struct CacheEntry {
    pub file_hash: String,
    pub ast: Component,
    pub compiled_output: CompiledOutput,
    pub dependencies: Vec<PathBuf>,
    pub timestamp: SystemTime,
}
```

### 3.2 Dependency Tracking

A dependency graph maintains relationships between files (imports, exports) to determine which files need recompilation when a source file changes.

## 4. Code Generation

Code generation produces the final artifacts for execution.

### 4.1 Rust Generation

The compiler generates optimized Rust code for the WebAssembly runtime.

```rust
// Generated structure example
#[wasm_bindgen]
pub struct MyComponent {
    world: OmniWorld,
    count: Signal<f64>,
}
```

### 4.2 TypeScript Definitions

TypeScript definitions (`.d.ts`) are generated automatically to provide IDE support and type safety for consumers.

## 5. Error Diagnostics

Clear and actionable error messages are a priority.

### 5.1 Diagnostic System

The compiler features a rich diagnostic system with colored output, code snippets, and helpful suggestions.

```rust
pub struct Diagnostic {
    pub severity: Severity,
    pub message: String,
    pub location: SourceLocation,
    pub context: String,
    pub suggestions: Vec<Suggestion>,
}
```
