# Platform Adapters

## 1. Web Platform

### 1.1 Architecture

The Web Adapter (`@omnicraft/web`) bridges the WASM runtime with the browser DOM.

```typescript
export class OmniCraftRenderer {
    constructor(canvas: HTMLCanvasElement) {
        // Initialize WebGL/Canvas context
        // Setup event listeners
    }
    
    async mount(ComponentClass: any) {
        // Initialize WASM
        await init();
        // Start render loop
    }
}
```

### 1.2 Event Handling

Maps browser events to OmniCraft internal events:
- Mouse/Touch -> Internal pointer events
- Keyboard -> Internal key events

### 1.3 DevTools Integration

Integrating with `OmniCraftDevTools` for time-travel debugging and state inspection.

## 2. Node.js Platform

### 2.1 Server-Side Rendering

The Node.js Adapter (`@omnicraft/node`) enables headless rendering for SSR and testing.

```typescript
import { createCanvas } from 'canvas';

export class OmniCraftNodeRenderer {
    async exportPNG(outputPath: string) {
        this.render();
        // Save to file
    }
}
```

### 2.2 Video Export

Capable of rendering frame-by-frame for high-quality video export using `ffmpeg`.

```typescript
await renderer.renderVideo(
    10, // duration seconds
    60, // fps
    'output.mp4'
);
```

## 3. Framework Integrations

### 3.1 React Integration

The `@omnicraft/react` package provides a wrapper component and hooks.

```tsx
import { OmniCraftComponent } from '@omnicraft/react';
import { MyGame } from './game.omni';

function App() {
  return (
    <OmniCraftComponent 
      component={MyGame}
      width={800} 
      height={600}
    />
  );
}
```

### 3.2 Vue Integration

The `@omnicraft/vue` package offers a first-class Vue component.

```vue
<template>
  <OmniCraftComponent 
    :component="MyGame"
    :width="800"
    :height="600"
  />
</template>
```

### 3.3 Svelte Integration

The `@omnicraft/svelte` package provides a Svelte component wrapper.

```svelte
<script>
  import { OmniCraft } from '@omnicraft/svelte';
  import MyGame from './game.omni';
</script>

<OmniCraft component={MyGame} width={800} height={600} />
```

