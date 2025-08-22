# DaiconShadow

![kinematic_daicon.png](../assets/images/nodes/daicon_shadow.png)

**DaiconShadow** - the node creates a shadow under the selected object.

---
## **Parameters**:

### - *d3*
<p style="color:#ffb0e0;">CharacterBody3D</p>
DaiconShadow core.

---
### *daicon_parent*

<p style="color:#ffb0e0;">Node</p>
The node to which the shadow is attached.

---
### - *tile_size*
<p style="color:#ffb0e0;">int</p>
Tile Size determines how many pixels equal 1 meter in 3D.
(basically it is the tile size per cell size in 3D)

*Automatically synchronized with **daicon_parent***

---
### - *z_step*
<p style="color:#ffb0e0;">int</p>
Z-step in the sorting system between height levels.

For example **z_step** = 10, then:

Level -1 = -10
Level 0 = 0
Level 1 = 10
Level 2 = 20

*Automatically synchronized with **daicon_parent***

---
### - *min_distance*
<p style="color:#ffb0e0;">int</p>
Minimum distance for texture modulation (in meters).

- For coloration, the optimal value is 1 or more;
- For discoloration, the optimal value is less than 1.

---
### - *max_distance*
<p style="color:#ffb0e0;">int</p>
Maximum distance between parent and self (in meters).

---
### *shadow_mode*

<p style="color:#ffb0e0;">int</p>
Shadow modulation mode: “Discoloration”, “Coloration”

---
### *stream_mode*

<p style="color:#ffb0e0;">int</p>

Physical body core behavior mode: “Logic”, “Direct”

**Logic** stream checks the body's state relative to the surface (is_on_floor). **Direct** does not.

---
### - *shape*
<p style="color:#ffb0e0;">Node3D</p>
A cell for a shape-node that is embedded in the core (needed for collisions).
Skips only **CollisionShape3D** or **CollisionPolygon3D**.
Has its own dictionary in the Shape section: **shape_properties**.

---
### *Shape-section*

Contains parameters for Shape.

---
### *KinematicBody3D-section*

Parameter section for the root node of the kernel. 

`(See Godot documentation : CharacterBody3D / KinematicBody3D)`.

---
### *CollisionObject3D-section*

Parameter section for the root node of the kernel. 

`(See Godot documentation : CollisionObject3D)`.

!!!info
	Also contains **axis_lock**.

---
## **Methods**:
## - *_ready*

Deploys the kernel at each startup. Performs basic configuration of the node.

---
### - *_process*

Synchronizes the movement of the node in 2D and its core in 3D.

Adds collision exclusion with **daicon_parent**, updates shadow modulation (triggers once at startup).

---
### - *physics_process*

Updates the position of the core, updates z_index, draws a shadow.

```python
func _physics_process(delta: float) -> void:
	if not Engine.is_editor_hint():
		if daicon_parent and daicon_parent.d3:
			var distance := d3.position.distance_to(daicon_parent.d3.global_position - daicon_parent.offset_3d)
			
			if stream_mode:
				#direct
				_update_direct_position(distance)
			elif not stream_mode:
				#logic
				_update_position(distance, delta)
```

---
### *_update_modulation*

```python
func _update_modulation(distance):
	if shadow_mode:
		#coloration
		if distance > min_distance:
			self.modulate.a = _initial_alpha
		else:
			self.modulate.a = lerp(0.0, _initial_alpha, distance / min_distance)
	elif not shadow_mode:
		#discoloration
		if distance > min_distance:
			self.modulate.a = lerp(0.0, _initial_alpha, min_distance / distance)
		else:
			self.modulate.a = _initial_alpha
```

This function updates the transparency of the shadow.

---
### - *update_pos*

```python
func _update_position(distance, delta):
	if daicon_parent.d3.is_on_floor():
		self.visible = true
		_update_modulation(distance)
		
		self.position.y = start_y
		d3.position = daicon_parent.d3.position - daicon_parent.offset_3d
		self.z_index = (round(d3.position.y + 0.3) * z_step) + 1
	else:
		if distance < max_distance:
			d3.velocity.y -= GRAVITY * delta
		else:
			self.visible = false
			d3.position = daicon_parent.d3.position - daicon_parent.offset_3d
		
		if d3.is_on_floor():
			self.visible = true
			_update_modulation(distance)
			self.position.y = start_y + (distance * tile_size)
			self.z_index = (round(d3.position.y + 0.3) * z_step) + 1
			d3.position = daicon_parent.d3.position - daicon_parent.offset_3d
	d3.move_and_slide()
```

The function updates the position of an object in 2D space and determines the z_index of the node.
(for logic stream)

---
### - *_update_direct_position*

```python
func _update_direct_position(distance):
	if distance < max_distance:
		d3.velocity.y = -GRAVITY
	else:
		self.visible = false
		d3.position = daicon_parent.d3.position - daicon_parent.offset_3d
	
	if d3.is_on_floor():
		self.visible = true
		_update_modulation(distance)
		self.position.y = start_y + (distance * tile_size)
		self.z_index = (round(d3.position.y + 0.3) * z_step) + 1
		d3.position = daicon_parent.d3.position - daicon_parent.offset_3d
	d3.move_and_slide()
```

The function updates the position of an object in 2D space and determines the z_index of the node.
(for direct stream)

---
### - *_expand*

```java
func _expand() -> void:
	_expand_d3()
	if shape_properties:
		_expand_shape()
```

The function is responsible for deploying the core.