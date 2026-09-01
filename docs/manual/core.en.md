# Core & Slot System

**Core** — is the internal three-dimensional stuffing of Daicon entities (`DaiconEntity`), which gives 2D objects real 3D properties (physics, 3D space coordinates, etc.).

The core is not visible in the scene tree as a regular node while editing — it is generated and managed automatically (exists in memory abstractly). When launching the game or working in `@tool` mode in the editor, the core unfolds from scratch based on the inspector properties and saved slot data.

---

## Architecture: 2D Shell & 3D Core

Any `DaiconEntity` (and its derivatives: `KinematicDaicon`, `StaticDaicon`, `RigidDaicon`, `AnimatedDaicon`) works on the principle of continuous synchronization:

1. **2D Node (Shell):** Lives in the 2D scene tree, handles rendering sprites, animations, 2D transforms, and screen-space `z_index`.
2. **3D Node (Core / `core`):** Automatically spawned as an internal child node (`CharacterBody3D`, `RigidBody3D`, `StaticBody3D`, or `AnimatableBody3D`). It drives real physics, collisions, and movement across the 3D world.

Positions and transforms between 2D and 3D sync automatically via projection formulas with `tile_size` and `offset_3d` taken into account.

---

## The Slot System (DaiconSlots)

To attach meshes, collision shapes, sensors, or custom nodes into the 3D core, Daicon uses the **Slot System**.

A slot lets you pick any 3D node in the editor, serialize it into a `Dictionary` (preserving its script, properties, and metadata), remove the temporary node from the scene, and automatically inject it inside the hidden 3D core.

> [!INFO] Technical implementation in a nutshell
> All node information is stored inside a dedicated `*_properties` dictionary. This dictionary is later used to dynamically deploy slot nodes inside the core.
> 
> ```mermaid
> graph TD
>     A(["✨ Custom 3D Node"]) -->|Assigned in Inspector| B(["⚙️ DaiconSlots.serialize_node()"])
>     B -->|Serialize to Data| C(["📦 Dictionary *_properties"])
>     C -->|Restore| D(["🧩 DaiconSlots.expand_slot()"])
>     D -->|Inject into Scene| E(["🚀 3D Core core"])
> 
>     classDef purple fill:#f3e8ff,stroke:#9333ea,stroke-width:1.5px,color:#581c87;
>     classDef blue fill:#e0f2fe,stroke:#0284c7,stroke-width:1.5px,color:#0369a1;
>     classDef amber fill:#fef3c7,stroke:#d97706,stroke-width:1.5px,color:#92400e;
>     classDef emerald fill:#d1fae5,stroke:#059669,stroke-width:1.5px,color:#065f46;
> 
>     class A purple;
>     class B,D blue;
>     class C amber;
>     class E emerald;
> ```

### Standard Entity Slots

The inspector of each `DaiconEntity` provides a dedicated **Slots** group:

| Inspector Slot | Storage Dictionary | Target Parent | Purpose |
| :--- | :--- | :--- | :--- |
| **Mesh Node** | `mesh_properties` | `core` | 3D visual/debug model display (`MeshInstance3D`). |
| **Shape Node** | `shape_properties` | `core` | Primary body collision shape (`CollisionShape3D` or `CollisionPolygon3D`). |
| **Whisker Node** | `whisker_properties` | `core` | `Area3D` sensor for calculating occlusions and dynamic `z_index` sorting. |
| **Whisker Shape Node** | `whisker_shape_properties` | `whisker` | Collision shape for the Whisker sensor. Automatically attached to `whisker_node`. |
| **Shader Cast Node** | `shader_cast_properties` | `core` | `RayCast3D` ray for triggering shader reveal effects when behind obstacles. |

---

## How Slots Work in Practice

You don't need to manually configure serialization — `DaiconSlots` handles everything behind the scenes:

* **Assigning a node to a slot:** Drag and drop any 3D node (e.g. `CollisionShape3D`) into the inspector slot field. The node will be packed into data, **removed from the 2D scene tree, and injected inside `core`**.
* **Ejecting a node:** Click the reset icon (or pass `null`) — the node will instantly unpack back into your scene tree with all stored settings intact.
* **Crash prevention:** If you rename or delete a node's script on disk, the scene won't break or wipe your settings — Daicon will safely fall back to the base engine class and notify you in the output console.
* **Complex hierarchies:** If a node contains child nodes, the entire branch will be packed and unpacked together (including scripts, resources, and metadata).

> [!TIP] Resource Uniquification
> When duplicating entities, slot resources are automatically cloned (`make_unique`), ensuring edits on one instance don't accidentally leak to others.

---

## Custom Slots (`custom_slots`)

In addition to the five default slots, `DaiconEntity` provides an array for code-driven customization:

```gdscript
@export_storage var custom_slots: Array[Dictionary] = []
```

It is designed to inject arbitrary 3D nodes and complex node trees into the core via code:

```gdscript
# Example: Packing a custom 3D node into the entity core
var my_light = OmniLight3D.new()
var light_data = DaiconSlots.serialize_node(my_light, true)
entity.custom_slots.append(light_data)

# Deploying the slot inside the core
DaiconSlots.expand_slot(light_data, entity.core)
```

---

## Direct Core Access via Code

To interact with the core in code, use the **`core`** variable:

```gdscript
# Example: Accessing CharacterBody3D inside KinematicDaicon
var body := core as CharacterBody3D
if body:
    body.velocity = Vector3(1, 0, 0)
    body.move_and_slide()
    update_pos()
```

> [!DANGER] Caution When Modifying the Core Directly
> Do not free the `core` node manually. Whenever you change velocities or positions, always call `update_pos()` so the 2D shell updates its on-screen position relative to the 3D core coordinates.
> 
> Be careful with your admin superpowers: with unrestricted access to the foundation, you should know exactly what you are doing so you don't break the node and its core.