# DaiconShadow

![daicon_shadow.png](../assets/images/nodes/daicon_shadow.png)

**DaiconShadow** — is a 2D sprite with a built-in 3D surface scanner. The node projects a realistic shadow beneath your object, calculating 3D height variations and smoothly fading alpha in mid-air.

Unlike older approaches based on rigid bodies or character bodies, the new version uses a lightweight cylinder cast (`ShapeCast3D`) that puts virtually zero stress on the physics engine and finds the ground under your feet automatically.

---

## How It Works

You don't need to hook up node references manually:

1. Add **DaiconShadow** as a child node to any Daicon entity (`KinematicDaicon`, `RigidDaicon`, etc.).
2. The shadow automatically locates its parent via `find_parent_entity()` and injects a hidden `ShapeCast3D` cylinder sensor directly into its 3D core.
3. The sensor continuously casts downward, finds the highest point of the ground beneath the object, and projects the 2D shadow sprite to the exact screen position with the correct `z_index`.

```mermaid
graph LR
    A(["🚀 3D Core core"]) -->|"auto-inject"| B(["🔍 ShapeCast3D"])
    B -->|"scans floor"| C[("🧱 Surface")]
    C -->|"height & distance"| D(["👥 2D DaiconShadow<br><small>Y-Position · Z-Index · Opacity</small>"])

    classDef purple fill:#f3e8ff,stroke:#9333ea,stroke-width:1.5px,color:#581c87;
    classDef blue fill:#e0f2fe,stroke:#0284c7,stroke-width:1.5px,color:#0369a1;
    classDef amber fill:#fef3c7,stroke:#d97706,stroke-width:1.5px,color:#92400e;
    classDef emerald fill:#d1fae5,stroke:#059669,stroke-width:1.5px,color:#065f46;

    class A purple;
    class B blue;
    class C amber;
    class D emerald;
```

---

> [!TIP] Quick Sprite Alignment
> Enable **Debug Ray** in the inspector to see the actual sensor radius and height line directly inside the 2D viewport. This makes aligning your shadow texture with character feet effortless.

---

## In-Game Behavior & Sorting

1. **On the ground (`is_on_floor`):** The shadow snaps to the soles instantly with zero lag or floaty interpolation.
2. **In the air:** The shadow fades out smoothly as height increases (between `fade_start_distance` and `max_distance`).
3. **Smart Z-Index:** The shadow always determines the height level of the detected floor point and is guaranteed to render beneath the character's feet (`z_index = min(floor_z, parent.z_index - 1)`).

> [!WARNING] Hierarchy Rule
> `DaiconShadow` must always be a descendant (at any nesting level) of a `DaiconEntity`. Moving it outside of an entity will cause the editor to display a configuration warning in the scene tree.