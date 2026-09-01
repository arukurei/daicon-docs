# RigidDaicon

![rigid_daicon.png](../assets/images/nodes/rigid_daicon.png)

**RigidDaicon** — is the node for objects driven by realistic 3D physics simulation: crates, pushable barrels, rolling boulders, and falling debris.

Under the hood, it operates a **`RigidBody3D`** core. Its movement is entirely handled by Godot's physics engine taking into account mass, gravity, friction, and collision impulses, while the node continuously projects the 3D body's motion onto the 2D screen.

---

## Physics & Capabilities

* **Mass & Gravity:** Configure body weight (`mass`), surface friction/bounciness (`physics_material_override`), and custom gravity scale (`gravity_scale`).
* **Forces & Impulses:** Push and launch objects using standard 3D physics methods (`apply_central_impulse()`, `apply_torque_impulse()`).
* **Freeze Mode:** Supports `freeze` (Static or Kinematic mode) to create props that stay dormant until triggered by an event or impact.
* **Auto-Synchronization:** The node automatically executes `update_pos()` inside `_physics_process()` during gameplay, ensuring the 2D sprite never drifts from the physical body.

---

## Code Interaction Example

To apply forces (e.g. from an explosion or a player kick):

```gdscript
@tool
extends RigidDaicon

func _physics_process(delta: float) -> void:
    super._physics_process(delta)
    if Engine.is_editor_hint(): return

    # Example: Apply an impulse on interaction
    var body := core as RigidBody3D
    if body and Input.is_action_just_pressed("ui_select"):
        body.apply_central_impulse(Vector3(0, 5.0, -2.0))
```

---

## Setup

1. Add a **RigidDaicon** to your scene.
2. Assign a collision shape (`BoxShape3D`, `SphereShape3D`, `CylinderShape3D`) to the **Shape Node** slot.
3. Configure `mass` and damping parameters (`Linear / Angular Damp`) in the inspector.
4. Assign a matching shape to **Whisker Shape Node** for accurate depth sorting.