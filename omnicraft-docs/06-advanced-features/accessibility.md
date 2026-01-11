# Accessibility Integration

OmniCraft prioritizes accessibility (a11y) out of the box, ensuring that applications built with the framework are usable by everyone. The compiler and runtime automatically handle many ARIA attributes and focus management tasks.

## ARIA Integration

The compiler analyzes the component structure and automatically injects appropriate ARIA roles and attributes where inference is possible.

### Automatic Role Assignment

For standard semantic elements, explicit roles are often redundant, but for custom interactive components, OmniCraft helps manage accessibility state.

```rust
// Example structure for ARIA attribute management
pub struct AriaProperties {
    pub role: Option<String>,
    pub label: Option<String>,
    pub description: Option<String>,
    pub hidden: bool,
    pub live: Option<AriaLive>,
}

pub enum AriaLive {
    Off,
    Polite,
    Assertive,
}
```

### Focus Management

The runtime includes a focus management system to handle focus trapping (for modals) and restoring focus when components unmount.

-   **FocusTrap**: A primitive that keeps keyboard navigation within a specific container.
-   **SkipLinks**: Automatic generation of skip-to-content links for main navigation areas.

## Linter Integration

The OmniCraft linter checks for common accessibility issues at compile time:
-   Missing `alt` text on images.
-   Interactive elements without labels.
-   Invalid ARIA attribute combinations.
