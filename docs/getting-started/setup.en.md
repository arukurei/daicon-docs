# Setup

## Daicon-Nodes 

Each Daicon node is a 2D node with a 3D filling. This keeps your game development workflow firmly grounded in familiar 2D space, while real 3D physics runs behind the scenes.

The internal 3D filling is called the **core**. It is generated automatically and exists abstractly in memory. To configure the core, developers are provided with an intuitive parameter panel in the inspector.

> [!NOTE] Direct Access 
> If you ever need low-level control over physics, you can access the core directly in code via the `core` variable and get complete access to all properties of the underlying 3D body.

![Pasted image 20250222092623.png](../assets/images/pasted-images/Pasted%20image%2020250222092623.png)

---

## 1. First Steps

> ![daicon.png](../assets/images/nodes/daicon.png)
>  
> First, add a root node to your scene. Usually in Godot this would be a Node2D, but the plugin provides a dedicated root node: **Daicon**.

> [!TIP] 
> You can still use standard `Node2D` nodes as your scene roots if you don't need the global sorting dispatcher and silhouette x-ray shaders (detailed in the [Daicon Reference](../node-reference/daicon.md)).

### Script Override

The addon nodes already have engine base scripts attached. This file defines the baseline functionality for **ALL** nodes of this type, so it should never be edited directly.

![Pasted image 20250222173306.png](../assets/images/pasted-images/Pasted%20image%2020250222173306.png)

To write custom gameplay logic for a specific object, you must extend the script:

1. Right-click the node in the scene tree.
2. In the context menu, select **«Extend Script»**.
3. In the dialog window that appears, check the **«Template»** box.
4. Select the preset template provided by the plugin.
5. Choose a save path inside your project and click **«Create»**.

Now your `.gd` file extends the base class, and you can write custom code freely.

![Pasted image 20250222173449.png](../assets/images/pasted-images/Pasted%20image%2020250222173449.png)

> [!INFO]
> Daicon ships with ready-to-use script templates in `addons/script_templates/`. To enable them across your project, copy the contents of that folder into your project's root `script_templates/` directory (by default `res://script_templates/`).

---

## 2. Environment

To build your game world, the plugin offers two nodes with similar principles but distinct roles:

### DaiconMap <small>(main world node)</small>
> ![daicon-map.png](../assets/images/nodes/daicon_map.png)
> 
> Represents a multi-layered tilemap. Each layer carries a unique **z-index**, which serves as its spatial elevation metric.
> 
> In other words, **z-sorting** arranges objects along the projected depth axis based on their index (**Y** axis in 3D).

### DaiconMapLayer <small>(standalone layer)</small>
> ![daicon_map_layer.png](../assets/images/nodes/daicon_map_layer.png)
> 
> Represents a single standalone `TileMapLayer` node with its own embedded `GridMap` core.
> 
> Ideal for removable rooftops, bridges, destructible platforms, or standalone props requiring custom shaders.

---

### Environment Setup

![Pasted image 20250819122909.png](../assets/images/pasted-images/Pasted%20image%2020250819122909.png)

* **Mesh Library:** The 3D mesh library used to construct the world geometry.
* **Physics Material:** Physical surface properties for tiles (friction and bounce).
* **Z Step:** The Z-index step between spatial elevation levels.
* **Visible 3D:** Toggles the visibility of the `GridMap` 3D mesh in the viewport.
* **Cell Size:** The size of a single 3D tile in meters (default `1x1x1`).
* **Collision Layer and Mask:** 3D physics collision layers.
* **Bake Navigation:** Bakes a 3D navigation mesh for pathfinding.
* **Transform Rotation3D and Scale3D:** Rotation and scale of the 3D core.

> [!EXAMPLE] Coordinate Auto-Sync
> The **Transform Position3D** section is synchronized with **Position2D**. Whenever you move a Daicon node in 2D space, its 3D core automatically updates its 3D position accordingly.

![Pasted image 20250222102256.png](../assets/images/pasted-images/Pasted%20image%2020250222102256.png)
![Pasted image 20250222102350.png](../assets/images/pasted-images/Pasted%20image%2020250222102350.png)
/// caption
<small>Position2D = Vector2(0, 0); Position3D = Vector3(0, 0, -0.5)</small>
///

![Pasted image 20250222102459.png](../assets/images/pasted-images/Pasted%20image%2020250222102459.png)
![Pasted image 20250222102539.png](../assets/images/pasted-images/Pasted%20image%2020250222102539.png)
/// caption
<small>Position2D = Vector2(-163, -157); Position3D = Vector3(-10.1875, 0, -10.3125)</small>
///

---

#### The Mesh Library

**DaiconMap** and **DaiconMapLayer** require a mesh library (`MeshLibrary`) — the 3D modular building blocks for your tiles (see [Mesh Library Manual](../manual/mesh.md)).

> [!TIP] Starter Mesh
> You can use the basic starter mesh library bundled with Daicon.

Once the library field is filled, your `TileSet` receives a custom data layer named **Item**. It binds each 2D tile to its corresponding mesh ID in the library. As soon as you paint a tile on the map, the corresponding 3D block is generated in space.

![Pasted image 20250222153734.png](../assets/images/pasted-images/Pasted%20image%2020250222153734.png)

> [!WARNING] Crucial Rule for Side Walls
> For vertical side wall tiles (the red tiles in the example), set their **local `z_index = -1`**. This eliminates visual sorting glitches when a character stands tightly enclosed between two walls.
> 
> ![Pasted image 20250821114142.png](../assets/images/pasted-images/Pasted%20image%2020250821114142.png)

---

#### Creating DaiconMap Layers

1. Create several layers in the `TileMap` inspector.
2. Assign them appropriate Z-indices stepped by `Z Step` (e.g. 0, 10, 20).
3. Open the bottom «TileMap» panel.
4. Click the tools menu in the top-right corner and select **«Extract TileMap layers as separate TileMapLayer nodes»**.

Now your environment layers are represented as full-fledged `TileMapLayer` nodes.

> [!TIP]
> Make sure to carefully monitor the `y-sort-enabled = true` property and the `z_index & z_step` of each child layer under DaiconMap.

With 2.5D projection taken into account, each layer above 0 shifts forward by one `z_step`, and lower layers shift backward accordingly, producing a tilted Top-Down perspective:

![Pasted image 20250222111927.png](../assets/images/pasted-images/Pasted%20image%2020250222111927.png)
> If this weren't the case, the perspective would be a pure flat top-down view (Top).

> [!WARNING] Layer Stacking Order
> Pay close attention to the tile stacking order when painting elevated terrain:
> 
> ![Pasted image 20250302203525.png](../assets/images/pasted-images/Pasted%20image%2020250302203525.png)
> /// caption
> - The lower the layer — the darker the color. 
> - Red — the side face of blocks.
> - The number indicates height elevation. To find the `z_index`, multiply the elevation number by `z_step`.
> ///

---

## 3. Player

> ![kinematic_daicon.png](../assets/images/nodes/kinematic_daicon.png)
> 
> The **KinematicDaicon** node is used for kinematic objects. It bundles everything needed for dynamic movement, collisions, and automatic 2.5D sorting.

### Creating the Player:

1. Add a **KinematicDaicon** node to your scene.
2. Extend its script via the context menu (**«Extend Script»** → choose the `KinematicDaicon` template).
3. Add visual nodes to the character (e.g. `Sprite2D`, `AnimationPlayer`, or `AnimationTree`).

---

### Player Parameters

![Pasted image 20250819124635.png](../assets/images/pasted-images/Pasted%20image%2020250819124635.png)

* **Tile Size:** World scale. Determines how many screen pixels equal 1 meter in 3D (default is 16).
* **Y 3D:** Character position along the 3D world's vertical Y axis.
* **Z Step:** The Z-index step between height tiers (must match `DaiconMap`).
* **Z Sort Coef:** Height coefficient of the character to ensure proper overlapping with other props.
* **Transform 3D:** Fine-tuning group for the 3D core: foot origin offset (`offset_3d`), rotation (`rotation_3d`), and scale (`scale_3d`).

---

### The Slot System (Slots)

In Daicon, all 3D physical components are assigned via **slots**. This allows you to construct a 3D body from standard nodes directly in the editor:

1. **Shape Node:** Slot for the main body collision shape (`CollisionShape3D` or `CollisionPolygon3D`).
2. **Mesh Node:** Slot for a 3D model (`MeshInstance3D`), used for visuals or debugging.
3. **Whisker Node & Whisker Shape Node:** Slots for the `Area3D` occlusion sensor and its shape.
4. **Shader Cast Node:** Slot for the `RayCast3D` beam that detects obstacles in front of the character.

> [!TIP] How to Assign a Node to a Slot
> Create the required 3D node (e.g. `CollisionShape3D`) in the scene tree, select `KinematicDaicon`, and assign that node into the corresponding slot field in the inspector. The node will be serialized, removed from the 2D tree, and injected inside the hidden 3D core.

---

### Sensors: Whisker & ShaderCast

![Pasted image 20250819125046.png](../assets/images/pasted-images/Pasted%20image%2020250819125046.png)

Two key spatial interaction tools are embedded into the character's core:

* **Whisker (Occlusion Sensor):** An `Area3D` node. It detects when the character steps behind elevated walls or terrain and dynamically adjusts the character's `z_index` so they hide behind the obstacle properly.
* **ShaderCast (Silhouette Shader Beam):** A `RayCast3D` ray. It shoots forward/towards the camera to detect if the character is obscured by a wall, sending a signal to the root `Daicon` node to reveal the character silhouette.

![Pasted image 20250819134809.png](../assets/images/pasted-images/Pasted%20image%2020250819134809.png)
/// caption
- **Whisker** sensor zone (red)
- **ShaderCast** (blue ray)
///

> [!INFO] Whisker Sizing and Offset
> * For optimal results, the collision shape for **Whisker Shape** should be slightly smaller than the character bounds (e.g., for a 1x1x1 m cube, make the whisker shape `0.9x0.9x0.9 m`). This prevents false triggers on tile seams.
> * By default, Whisker is shifted slightly forward (by 1.1 m) to detect obstacles directly in front of the character.