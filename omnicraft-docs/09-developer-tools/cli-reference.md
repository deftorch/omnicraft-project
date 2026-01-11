# CLI Reference

## 1. Advanced Commands

### 1.1 Watch Mode

Watches for file changes and incrementally recompiles.

```bash
omnicraft watch [options]
```

**Options:**
- `-p, --pattern`: File pattern to watch (default: "src/**/*.omni")
- `--poll`: Use polling (legacy support)
- `--debounce`: Delay in ms (default: 300)

### 1.2 Benchmark

Runs performance benchmarks on your project.

```bash
omnicraft benchmark [options]
```

**Metrics:**
- Compilation Time (min/max/avg)
- Bundle Size
- Memory Usage
- Throughput (files/sec)

### 1.3 Analyze

Analyzes component dependencies and generates code metrics.

```bash
omnicraft analyze <file> [options]
```

**Outputs:**
- Dependency Graph
- Code Metrics (LOC, Complexity)
- AST Dump (optional)

### 1.4 Doctor

Diagnoses installation and configuration issues.

```bash
omnicraft doctor
```

**Checks:**
- Node.js & Rust toolchains
- WASM targets
- Configuration validity
- Network connectivity

### 1.5 Init

Interactive project scaffolding tool.

```bash
omnicraft init
```

Prompts for:
- Project Name
- Template (Blank, Game, specialized)
- Features (TypeScript, ESLint, Testing)
- Git & Package Manager
