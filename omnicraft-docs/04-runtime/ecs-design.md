# Entity-Component-System (ECS) Design

The ECS architecture is the core of OmniCraft's runtime, enabling efficient entity management and system processing.

## 1. Overview

The ECS design prioritizes performance and modularity by separating data (Components) from behavior (Systems).

### 1.1 Why ECS?
- **Better data locality**: Components are stored in contiguous arrays, improving cache performance.
- **Easier to parallelize**: Systems can run in parallel if they operate on disjoint sets of components.
- **Flexibility**: Composition over inheritance allows for more flexible behavior definitions.
- **Scalability**: Capable of handling large numbers of entities efficiently.

## 2. Core Architecture

The architecture revolves around three main concepts: World, Entities, and Components.

### 2.1 World & Entity
The `World` is the container for all entities and components. Entities are lightweight IDs.

```rust
pub struct World {
    entities: Vec<Entity>,
    components: ComponentStorage,
    scene_graph: SceneGraph,
}

pub struct Entity {
    id: EntityId,
    generation: u32,
}
```

### 2.2 Components
Components are data containers without behavior.

```rust
pub struct Transform {
    pub position: Vec2,
    pub rotation: f32,
    pub scale: Vec2,
}

pub enum Shape {
    Circle { radius: f32 },
    Rectangle { width: f32, height: f32 },
    Path { points: Vec<Vec2> },
}
```

## 3. Data Model

The data model defines how entities and components are organized and accessed in memory.

### 3.1 Scene Graph Model

The Scene Graph manages the hierarchy and spatial relationships of entities.

```rust
pub struct SceneGraph {
    // Root node
    root: NodeId,
    // Node storage
    nodes: Vec<Node>,
    // Hierarchy
    parent_map: HashMap<NodeId, NodeId>,
    children_map: HashMap<NodeId, Vec<NodeId>>,
    // Spatial indexing for fast queries
    spatial_index: QuadTree<NodeId>,
    // Render order
    z_index_map: BTreeMap<i32, Vec<NodeId>>,
}

pub struct Node {
    pub id: NodeId,
    pub entity: EntityId,
    pub local_transform: Transform,
    pub world_transform: Transform,
    pub dirty: bool,
}
```

### 3.2 Component Storage (Sparse Set)

We use a **Sparse Set** architecture for efficient component storage and retrieval.

```rust
pub struct ComponentStorage {
    // Type-erased component arrays
    storage: HashMap<TypeId, ComponentArray>,
    // Entity to component index mapping
    entity_to_index: HashMap<EntityId, HashMap<TypeId, usize>>,
}

struct ComponentArray {
    // Sparse set for fast lookup
    sparse: Vec<Option<usize>>,
    dense: Vec<EntityId>,
    // Actual component data (contiguous memory)
    data: Vec<u8>,
    component_size: usize,
}
```

### 3.3 Shared Memory Layout

The Trinity Architecture uses a shared memory arena to enable zero-copy data sharing between Wasm, WebGPU, and WebNN.

```rust
pub struct SharedMemoryArena {
    buffer: SharedArrayBuffer,
    size: usize,
    allocator: BumpAllocator,
    regions: Vec<MemoryRegion>,
}

// Memory Layout (256 MB Total)
// [0 - 64MB]      Wasm Heap (ECS World, Scene Graph)
// [64MB - 192MB]  GPU Buffers (Vertex, Index, Uniforms)
// [192MB - 240MB] NPU Tensors (Weights, Inputs/Outputs)
// [240MB - 256MB] Shared Workspace
```

## 4. Core Algorithms

Core algorithms provide the underlying logic for scheduling and spatial management.

### 4.1 Workload Scheduling

The scheduler dynamically routes tasks to the most appropriate backend (CPU, GPU, or NPU).

```rust
pub struct WorkloadScheduler {
    capabilities: Capabilities,
    performance_history: PerformanceHistory,
    strategy: SchedulingStrategy, // FastestFirst, PowerEfficient, etc.
}

impl WorkloadScheduler {
    pub fn schedule_task(&self, task: &Task) -> ExecutionPlan {
        match task.task_type {
            TaskType::Rendering => ExecutionPlan::WebGPU, // Always GPU
            TaskType::AI => {
                if self.capabilities.has_webnn_npu {
                    ExecutionPlan::WebNN(DeviceType::NPU)
                } else if self.capabilities.has_webnn_gpu {
                    ExecutionPlan::WebNN(DeviceType::GPU)
                } else {
                    ExecutionPlan::Wasm // Fallback
                }
            },
            TaskType::Compute => {
                // Heuristic based on complexity
                if task.complexity > threshold {
                    ExecutionPlan::WebGPUCompute
                } else {
                    ExecutionPlan::Wasm
                }
            },
            TaskType::Logic => ExecutionPlan::Wasm,
        }
    }
}
```

### 4.2 Spatial Partitioning (QuadTree)

Used for efficient collision detection and spatial queries.

```rust
pub struct QuadTree<T> {
    root: QuadTreeNode<T>,
    bounds: Rect,
    max_depth: u32,
    max_items_per_node: usize,
}

// Recursively subdivides space into 4 quadrants when item count exceeds threshold
```

### 4.3 Path Simplification

We use the **Douglas-Peucker Algorithm** to simplify vector paths for rendering efficiency.

```rust
pub fn simplify_path(points: &[Vec2], epsilon: f32) -> Vec<Vec2> {
    // Recursive algorithm to remove points that are within 'epsilon' distance
    // of the line segment connecting the start and end points.
}
```
