# StaticDaicon

![static_daicon.png](../assets/images/nodes/static_daicon.png)

**StaticDaicon** — is the node for immovable environment objects: walls, boulders, trees, pillars, and fences.

Under the hood, it creates a **`StaticBody3D`** core. It does not move when hit by physics forces, serving as a solid obstacle for players and rigid bodies.

---

## Features & Use Cases

* **Conveyor Belt Velocities:** Using `constant_linear_velocity_3d` and `constant_angular_velocity_3d`, you can simulate conveyor belts — the object stays stationary while pushing anything standing on top of it.
* **Automatic Z-Index:** Unlike kinematic entities, `StaticDaicon` automatically syncs its position and sorting layer inside `_process()`, so you don't even need to attach a script for basic static props.

---

## Setup

1. Add a **StaticDaicon** to your scene.
2. Assign a `CollisionShape3D` to the **Shape Node** slot.
3. Assign a matching shape to the **Whisker Shape Node** slot so characters sort properly when walking behind it.