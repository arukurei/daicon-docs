# RigidDaicon

![daicon.png](../assets/images/nodes/rigid_daicon.png)

**RigidDaicon** - a node representing a kinematic object with complex physics. The core is a three-dimensional physical body moved using physical simulation.

---
## **Parameters**:

### - *d3*
<p style="color:#ffb0e0;">RigidBody3D</p>
RigidDaicon core.

---
### - *whisker*
<p style="color:#ffb0e0;">Area3D</p>
It tracks collisions and works in conjunction with **y** and **z-sorting** to correctly render objects. It determines whether an object is behind a barrier or not.

---
### - *shader_cast*
<p style="color:#ffb0e0;">RayCast3D</p>
**ShaderCast** is a special purpose node. Its purpose is to detect collisions with objects in front of the player and draw a shader based on that.

---
### - *tile_size*
<p style="color:#ffb0e0;">int</p>
Tile Size determines how many pixels equal 1 meter in 3D.
(basically it is the tile size per cell size in 3D)

---
### - *y_3d*
<p style="color:#ffb0e0;">int</p>
The character's position on the Z-axis.

---
### - *z_step*
<p style="color:#ffb0e0;">int</p>
Z-step in the sorting system between height levels.

For example **z_step** = 10, then:

Level -1 = -10
Level 0 = 0
Level 1 = 10
Level 2 = 20

---
### - *mesh*
<p style="color:#ffb0e0;">MeshInstance3D</p>
A mesh node cell that is embedded in the core (after installation you can see it in the 3D section).
It has its own dictionary in the “Core” section: **mesh_properties**.

---
### - *shape*
<p style="color:#ffb0e0;">Node3D</p>
A cell for a shape-node that is embedded in the core (needed for collisions).
Skips only **CollisionShape3D** or **CollisionPolygon3D**.
Has its own dictionary in the Core section: **shape_properties**.

---
### *Mesh & Shape-section*

The “Mesh & Shape” section contains the parameters for Mesh and Shape.

---
### *Slots-section*
<p style="color:#ffb0e0;">Node3D</p>
Slots - cells for developer nodes if it is necessary to implement them in the core (communication through code only).

---
### *Core-section*
#### - *child_count*
<p style="color:#ffb0e0;">int</p>
Keeps a constant count of the number of child nodes of the kernel.

---
#### - *properties*
<p style="color:#ffb0e0;">Dictionary</p>
Dictionaries of node parameters stored in the kernel via cells. Stores all parameters necessary for dynamic deployment of kernel child nodes.

---
### *RigidBody3D-section*

Parameter section for the root node of the kernel. 

`(See Godot documentation : RigidBody3D)`

---
### *CollisionObject3D-section*

Parameter section for the root node of the kernel. 

`(See Godot documentation : CollisionObject3D)`.

!!!info
	Also contains **axis_lock**.

---
### *RayCast-section*

Parameters section for **Whisker** and **ShaderCast** nodes.

`(See Godot documentation: Area3D - RayCast3D)`

---
## **Methods**:
## - *_ready*

Deploys the kernel at each startup. Performs basic configuration of the node.

---
### - *_process*

> Works only in the editor.

Synchronizes movement of a node in 2D and its core in 3D. 

---
### - *_physics_process*

Update object position. 

---
### - *_update_pos*

```python
func update_pos(coef = 1):
	self.position.x = (d3.position.x - offset_3d.x) * tile_size
	self.position.y = ((d3.position.z - offset_3d.z) - (d3.position.y - offset_3d.y)) * tile_size
	
	if whisker.get_overlapping_bodies():
		if whisker.get_overlapping_bodies()[0].has_meta("z_index"):
			self.z_index = whisker.get_overlapping_bodies()[0].get_meta("z_index") - 1
		else:
			self.z_index = (int(d3.position.y + (offset_3d.y * 1.1))) * z_step - 1
	else:
		self.z_index = ((d3.position.y - offset_3d.y) + coef) * z_step + 2
	
	d3.set_meta("z_index", self.z_index)
```

The function updates the position of the object in 2D space by passing it the 3D coordinates of the core.

coef determines the height of the object.

---
### - *get_node_properties*

```java
func get_node_properties(node: Node) -> Dictionary:
	var properties : Dictionary = {
		"Name" : node.name,
		"Class" : node.get_class(),
		"Properties" : {}
	}
	for prop in node.get_property_list():
		if prop.usage & PROPERTY_USAGE_STORAGE:
			properties.Properties[prop.name] = node.get(prop.name)
	return properties
```

The function writes all node parameters, its name, and class into a dictionary and returns it.

---
### - *_expand*

```java
func _expand()  -> void:
	_expand_d3()
	_expand_ray_cast()
	if mesh_properties:
		_expand_mesh()
	if shape_properties:
		_expand_shape()
	_expand_slots()
```

The function deals with the deployment of the kernel.