# StaticDaicon

![static_daicon.png](../assets/images/static_daicon.png)

StaticDaicon - node representing a static object.

---
## **Parameters**:

### - *d3*
<p style="color:#ffb0e0;">CharacterBody3D</p>
StaticDaicon core.

---
### - *right_whisker*
<p style="color:#ffb0e0;">RayCast3D</p>
It tracks collisions and works in conjunction with **y** and **z-sorting** to correctly render objects. It determines whether an object is behind a barrier or not.

---
### - *left_whisker*
<p style="color:#ffb0e0;">RayCast3D</p>
It tracks collisions and works in conjunction with **y** and **z-sorting** to correctly render objects. It determines whether an object is behind a barrier or not.

---
### - *center_whisker*
<p style="color:#ffb0e0;">RayCast3D</p>
It tracks collisions and works in conjunction with **y** and **z-sorting** to correctly render objects. It determines whether an object is behind a barrier or not.

---
### - *shader_cast*
<p style="color:#ffb0e0;">RayCast3D</p>
**ShaderCast** is a special purpose node. Its purpose is to detect collisions with objects in front of the player and draw a shader based on that.

---
### - *tile_size*
<p style="color:#ffb0e0;">int</p>
The size of one tile (the value determines the number of pixels that the character passes equal to 1 meter).

---
### - *y_3d*
<p style="color:#ffb0e0;">int</p>
The character's position on the Z-axis.

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

The “Mesh & Shape” section contains **Transform** parameters for Mesh and Shape.

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
### *StaticBody-section*

Parameter section for the kernel root node.

`(See Godot documentation : StaticBody)`.

---
### *CollisionObject3D-section*

Parameter section for the root node of the kernel. 

`(See Godot documentation : CollisionObject3D)`.

!!!info
	Also contains **axis_lock**.

---
### *RayCast-section*

Parameter section for **Whiskers** and **ShaderCast** nodes.

`(See Godot documentation : RayCast3D)`.

---
## **Методы**:
## - *_ready*

Deploys the kernel at each startup. Performs basic configuration of the node.

---
### - *_process*

> Works only in the editor.

Synchronizes movement of a node in 2D and its core in 3D. 

---

---
### - *update_pos*

```python
func update_pos(first_coef = 1.1, second_coef = 2.1):
	self.position.x = d3.position.x * tile_size
	self.position.y = d3.position.z * tile_size - d3.position.y * tile_size
	if right_whisker.is_colliding() or left_whisker.is_colliding() or center_whisker.is_colliding():
		self.z_index = d3.position.y + first_coef
	else:
		self.z_index = d3.position.y + second_coef
```

The function updates the position of the object in 2D space by passing it the 3D coordinates of the core.

**first_coef** determines the coefficient to the z-index when the object is behind the fence.

**second_coef** determines the coefficient to the z-index when **Whiskers** do not encounter anything.

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
### *_expand*

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
