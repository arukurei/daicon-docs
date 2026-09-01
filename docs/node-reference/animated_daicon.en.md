# AnimatedDaicon

![animated_daicon.png](../assets/images/nodes/animated_daicon.png)

**AnimatedDaicon** — is the node for moving obstacles, elevators, doors, traps, and moving platforms.

Under the hood, it operates an **`AnimatableBody3D`** core. It moves along a defined path (via `AnimationPlayer`, `Tween`, or script) and can push or carry characters standing on top of it without physics jitter.

---

## Features

* **Sync To Physics (`sync_to_physics_3d`):** Automatically syncs animation motion with physics engine ticks. Characters riding the platform stay firmly attached and move smoothly.
* **Constant Velocity:** Supports `constant_linear_velocity_3d` for moving walkways and escalators.

---

## Code / Tween Example

When scripting platform movement, extend the node with the `AnimatedDaicon` template:

```gdscript
@tool
extends AnimatedDaicon

func _ready() -> void:
    super._ready()
    if not Engine.is_editor_hint():
        # Example: Simple back-and-forth platform movement
        var body := core as AnimatableBody3D
        var tween := create_tween().set_loops()
        tween.tween_property(body, "position:x", body.position.x + 3.0, 2.0)
        tween.tween_property(body, "position:x", body.position.x, 2.0)
```