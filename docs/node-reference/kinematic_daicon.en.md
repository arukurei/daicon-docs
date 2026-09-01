# KinematicDaicon

![kinematic_daicon.png](../assets/images/nodes/kinematic_daicon.png)

**KinematicDaicon** — is the primary node for creating player characters, enemies, NPCs, and any objects with kinematic movement.

Under the hood, it embeds a full-fledged 3D core: **`CharacterBody3D`**. You control the body using standard Godot methods (`velocity`, `move_and_slide()`, `is_on_floor()`), while the node automatically projects all jumps, wall slides, and slope traversal directly onto the 2D screen.

---

## Code

Add the node to your scene, right-click it → **Extend Script**, and select the `KinematicDaicon` template.

Here is a ready-to-use basic character controller example:

```gdscript
@tool
extends KinematicDaicon

const SPEED := 5.0
const JUMP_VELOCITY := 4.5
const GRAVITY := 9.8

func _physics_process(delta: float) -> void:
    # Do not run game logic inside the editor
    if Engine.is_editor_hint(): return
    
    # 1. Access the 3D core
    var body := core as CharacterBody3D
    if not body: return

    # 2. Read 2D input and translate to 3D (X and Z)
    var input_dir := Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
    var direction := Vector3(input_dir.x, 0.0, input_dir.y).normalized()

    # 3. Horizontal movement
    if direction != Vector3.ZERO:
        body.velocity.x = direction.x * SPEED
        body.velocity.z = direction.z * SPEED
    else:
        body.velocity.x = move_toward(body.velocity.x, 0.0, SPEED)
        body.velocity.z = move_toward(body.velocity.z, 0.0, SPEED)

    # 4. Gravity and jumping (Y axis)
    if not body.is_on_floor():
        body.velocity.y -= GRAVITY * delta
    elif Input.is_action_just_pressed("ui_accept"):
        body.velocity.y = JUMP_VELOCITY

    # 5. Move and synchronize the 2D sprite
    body.move_and_slide()
    update_pos()
```

> [!TIP] 3D Z-Axis translates to Screen Verticality
> Notice how `input_dir.y` (up/down on keyboard) maps to `direction.z` in the 3D world. This moves the character deeper into the scene, while the `Y` axis handles jumping and falling.

---

## Setting Up Collisions

1. Add a `CollisionShape3D` node to your scene (e.g. with a `CapsuleShape3D` or `BoxShape3D`).
2. In the `KinematicDaicon` inspector, select the created node in the **Shape Node** slot.
3. The node will disappear from the 2D tree and inject directly into the core.
4. Assign a collision shape into **Whisker Shape Node** to ensure the character hides behind walls with accurate sorting.