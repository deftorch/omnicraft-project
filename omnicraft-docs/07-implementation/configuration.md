# Configuration System

## 1. Schema

OmniCraft uses Zod for robust configuration validation. The configuration file (`omnicraft.config.js`) supports typed options for compiler, dev server, and build settings.

```typescript
export const ConfigSchema = z.object({
  compiler: z.object({
    target: z.enum(['wasm32', 'wasm64']).default('wasm32'),
    optimizationLevel: z.number().min(0).max(3).default(2),
    optimizations: z.object({
      loopUnrolling: z.boolean().default(true),
      pgo: z.boolean().default(false),
    }),
  }),
  dev: z.object({
    port: z.number().default(3000),
    hmr: z.boolean().default(true),
  }),
});
```

## 2. Config Loader

The loader supports multiple file formats (`.js`, `.json`, `.yaml`) and automatically merges user config with defaults.

```typescript
// omnicraft.config.js default
export default {
  compiler: {
    optimizationLevel: 2,
    incremental: true,
  },
  dev: {
    port: 3000,
    hmr: true,
  },
};
```

## 3. Migration

The system includes a `ConfigMigrator` to automatically upgrade configuration files between versions (e.g., `0.x` to `1.0`).

```typescript
public migrate(oldConfig: any, fromVersion: string): OmniCraftConfig {
    if (fromVersion === '0.x') {
        return this.migrateFrom0x(oldConfig);
    }
    // ...
}
```
