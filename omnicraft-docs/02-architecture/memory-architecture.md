# Memory Architecture

> **Sources:**
> - v6.0 Section 9: Manajemen Memori Bersama
> - v6.0 Section 10: Strategi Alokasi

## 9. Manajemen Memori Bersama

### 9.1 Arsitektur Memory Arena

```
┌────────────────────────────────────────────────────────┐
│           SHARED MEMORY ARCHITECTURE                   │
└────────────────────────────────────────────────────────┘

              SharedArrayBuffer (256 MB)
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌────▼────┐    ┌────▼────┐    ┌────▼────┐
   │  Wasm   │    │  WebGPU │    │  WebNN  │
   │  View   │    │  View   │    │  View   │
   └────┬────┘    └────┬────┘    └────┬────┘
        │               │               │
        │    No Copy    │    No Copy    │
        └───────────────┴───────────────┘
                        │
                  Same Memory
```

### 9.2 Allocator Design

```rust
// Bump Allocator for Shared Memory

pub struct BumpAllocator {
    offset: AtomicUsize,
    size: usize,
    
    // Stack of freed regions for reuse
    free_list: Mutex<Vec<FreeBlock>>,
}

struct FreeBlock {
    offset: usize,
    size: usize,
}

impl BumpAllocator {
    pub fn new(size: usize) -> Self {
        Self {
            offset: AtomicUsize::new(0),
            size,
            free_list: Mutex::new(Vec::new()),
        }
    }
    
    pub fn allocate(&self, size: usize, alignment: usize) -> Option<usize> {
        // Align size to requested alignment
        let aligned_size = (size + alignment - 1) & !(alignment - 1);
        
        // Try to reuse freed block first
        {
            let mut free_list = self.free_list.lock().unwrap();
            
            // Find suitable block (first-fit)
            if let Some(pos) = free_list.iter().position(|block| block.size >= aligned_size) {
                let block = free_list.remove(pos);
                
                // Split block if too large
                if block.size > aligned_size + 64 {
                    free_list.push(FreeBlock {
                        offset: block.offset + aligned_size,
                        size: block.size - aligned_size,
                    });
                }
                
                return Some(block.offset);
            }
        }
        
        // Bump allocation
        let current = self.offset.load(Ordering::Relaxed);
        let aligned_offset = (current + alignment - 1) & !(alignment - 1);
        let new_offset = aligned_offset + aligned_size;
        
        if new_offset > self.size {
            return None; // Out of memory
        }
        
        // Try to update offset atomically
        match self.offset.compare_exchange(
            current,
            new_offset,
            Ordering::Release,
            Ordering::Relaxed,
        ) {
            Ok(_) => Some(aligned_offset),
            Err(_) => {
                // Retry if another thread allocated
                self.allocate(size, alignment)
            }
        }
    }
    
    pub fn deallocate(&self, offset: usize, size: usize) {
        let mut free_list = self.free_list.lock().unwrap();
        
        // Add to free list
        free_list.push(FreeBlock { offset, size });
        
        // Sort by offset for easier coalescing
        free_list.sort_by_key(|block| block.offset);
        
        // Coalesce adjacent blocks
        let mut i = 0;
        while i < free_list.len() - 1 {
            if free_list[i].offset + free_list[i].size == free_list[i + 1].offset {
                free_list[i].size += free_list[i + 1].size;
                free_list.remove(i + 1);
            } else {
                i += 1;
            }
        }
    }
    
    pub fn reset(&self) {
        self.offset.store(0, Ordering::Release);
        self.free_list.lock().unwrap().clear();
    }
}
```

### 9.3 Memory Pool untuk GPU Buffers

```rust
// GPU Buffer Pool

pub struct GPUBufferPool {
    device: GPUDevice,
    pools: HashMap<BufferSpec, Vec<PooledBuffer>>,
    max_pool_size: usize,
}

#[derive(Hash, Eq, PartialEq, Clone)]
struct BufferSpec {
    size: usize,
    usage: GPUBufferUsageFlags,
}

struct PooledBuffer {
    buffer: GPUBuffer,
    in_use: bool,
    last_used: Instant,
}

impl GPUBufferPool {
    pub fn new(device: GPUDevice, max_pool_size: usize) -> Self {
        Self {
            device,
            pools: HashMap::new(),
            max_pool_size,
        }
    }
    
    pub fn acquire(&mut self, size: usize, usage: GPUBufferUsageFlags) -> GPUBuffer {
        let spec = BufferSpec { size, usage };
        
        // Get or create pool for this spec
        let pool = self.pools.entry(spec.clone()).or_insert_with(Vec::new);
        
        // Find available buffer
        if let Some(pooled) = pool.iter_mut().find(|b| !b.in_use) {
            pooled.in_use = true;
            pooled.last_used = Instant::now();
            return pooled.buffer.clone();
        }
        
        // Create new buffer if pool not full
        if pool.len() < self.max_pool_size {
            let buffer = self.device.create_buffer(&GPUBufferDescriptor {
                size: size as f64,
                usage,
                mapped_at_creation: false,
            });
            
            pool.push(PooledBuffer {
                buffer: buffer.clone(),
                in_use: true,
                last_used: Instant::now(),
            });
            
            return buffer;
        }
        
        // Pool is full, just create temporary buffer
        self.device.create_buffer(&GPUBufferDescriptor {
            size: size as f64,
            usage,
            mapped_at_creation: false,
        })
    }
    
    pub fn release(&mut self, buffer: &GPUBuffer) {
        // Mark buffer as available
        for pool in self.pools.values_mut() {
            if let Some(pooled) = pool.iter_mut().find(|b| &b.buffer == buffer) {
                pooled.in_use = false;
                pooled.last_used = Instant::now();
                break;
            }
        }
    }
    
    pub fn cleanup_old_buffers(&mut self, max_age: Duration) {
        let now = Instant::now();
        
        for pool in self.pools.values_mut() {
            pool.retain(|b| {
                b.in_use || now.duration_since(b.last_used) < max_age
            });
        }
    }
}
```

## 10. Strategi Alokasi

### 10.1 Memory Budget System

```rust
// Memory Budget Management

pub struct MemoryBudget {
    total_budget: usize,
    allocations: HashMap<MemoryCategory, usize>,
    limits: HashMap<MemoryCategory, usize>,
}

#[derive(Hash, Eq, PartialEq, Clone, Copy)]
pub enum MemoryCategory {
    ECS,
    GPUBuffers,
    GPUTextures,
    NPUModels,
    NPUTensors,
    Temporary,
}

impl MemoryBudget {
    pub fn new(total_budget: usize) -> Self {
        let mut limits = HashMap::new();
        
        // Default budget allocation
        limits.insert(MemoryCategory::ECS, total_budget * 25 / 100);  // 25%
        limits.insert(MemoryCategory::GPUBuffers, total_budget * 30 / 100);  // 30%
        limits.insert(MemoryCategory::GPUTextures, total_budget * 20 / 100);  // 20%
        limits.insert(MemoryCategory::NPUModels, total_budget * 10 / 100);  // 10%
        limits.insert(MemoryCategory::NPUTensors, total_budget * 10 / 100);  // 10%
        limits.insert(MemoryCategory::Temporary, total_budget * 5 / 100);  // 5%
        
        Self {
            total_budget,
            allocations: HashMap::new(),
            limits,
        }
    }
    
    pub fn can_allocate(&self, category: MemoryCategory, size: usize) -> bool {
        let current = self.allocations.get(&category).copied().unwrap_or(0);
        let limit = self.limits.get(&category).copied().unwrap_or(0);
        
        current + size <= limit
    }
    
    pub fn allocate(&mut self, category: MemoryCategory, size: usize) -> Result<(), String> {
        if !self.can_allocate(category, size) {
            return Err(format!(
                "Memory budget exceeded for {:?}. Current: {}, Requested: {}, Limit: {}",
                category,
                self.allocations.get(&category).unwrap_or(&0),
                size,
                self.limits.get(&category).unwrap_or(&0)
            ));
        }
        
        *self.allocations.entry(category).or_insert(0) += size;
        Ok(())
    }
    
    pub fn deallocate(&mut self, category: MemoryCategory, size: usize) {
        if let Some(allocated) = self.allocations.get_mut(&category) {
            *allocated = allocated.saturating_sub(size);
        }
    }
    
    pub fn get_usage(&self, category: MemoryCategory) -> f32 {
        let allocated = self.allocations.get(&category).copied().unwrap_or(0);
        let limit = self.limits.get(&category).copied().unwrap_or(1);
        
        (allocated as f32 / limit as f32) * 100.0
    }
    
    pub fn print_report(&self) {
        println!("\n┌────────────────────────────────────────────┐");
        println!("│         Memory Budget Report               │");
        println!("├────────────────────────────────────────────┤");
        
        for category in [
            MemoryCategory::ECS,
            MemoryCategory::GPUBuffers,
            MemoryCategory::GPUTextures,
            MemoryCategory::NPUModels,
            MemoryCategory::NPUTensors,
            MemoryCategory::Temporary,
        ] {
            let usage = self.get_usage(category);
            let allocated = self.allocations.get(&category).unwrap_or(&0);
            let limit = self.limits.get(&category).unwrap_or(&0);
            
            println!(
                "│ {:?}: {:.1}% ({} / {} MB)",
                category,
                usage,
                allocated / (1024 * 1024),
                limit / (1024 * 1024)
            );
        }
        
        println!("└────────────────────────────────────────────┘\n");
    }
}
```
