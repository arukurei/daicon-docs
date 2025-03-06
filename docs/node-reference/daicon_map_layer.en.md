# DaiconMapLayer

![daicon_map_layer.png](../assets/images/daicon_map_layer.png)

Represents a single local independent node **TileMapLayer** with an embedded core **DaiconMap**.

A **DaiconMapLayer** is thus a support node or a node representing an individual element of your environment that requires special attention.

---
## **Параметры**:

### - *grid_map*
<p style="color:#ffb0e0;">Array[Node]</p>
DaiconMap Core.

---
### - *mesh_library*
<p style="color:#ffb0e0;">MeshLibrary</p>
Library of meshes from which the 3D environment is built.

---
### - *physics_material*
<p style="color:#ffb0e0;">PhysicsMaterial</p>
Used to determine the physical properties, such as friction and elasticity, of individual tiles.

---
### - *size*
<p style="color:#ffb0e0;">Vector3</p>
The size of one three-dimensional tile in meters.

---
### - *layer*
<p style="color:#ffb0e0;">int</p>
Collision layers for **grid_map**.

---
### - *mask*
<p style="color:#ffb0e0;">int</p>
Collision layers for **grid_map**.

---
### - *bake_navigation*
<p style="color:#ffb0e0;">bool</p>
Bake a navigation grid for 3D.

---
### - *rotation_3d*
<p style="color:#ffb0e0;">Vector3</p>
Core Rotation Customization.

---
### - *scale_3d*
<p style="color:#ffb0e0;">Vector3</p>
Customizing the scale of the core.

---
## **Методы**:
### - *_ready*

Deploys the kernel at each startup. Performs basic configuration of the node.

---
### - *_process*

> Works only in the editor.

Updates **grid_map** when the number of 3D tiles is not equal to the number of 2D tiles (calls **update_grid_map**).

Synchronizes moving of node in 2D and its core in 3D. 

---
### - *update_grid_map*

Updates **grid_map**.

```python
func update_grid_map():
	grid_map.clear()
	for tile in get_used_cells():
		var tile_data = get_cell_tile_data(Vector2(tile.x, tile.y))
		grid_map.set_cell_item(Vector3(tile.x, z_index-1, tile.y+z_index), tile_data.get_custom_data("Item"))
```