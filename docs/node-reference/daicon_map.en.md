# DaiconMap

![daicon-map.png](../assets/images/nodes/daicon_map.png)

This is a set of **TileMapLayers**, which in turn are layers of your environment.

Each such layer contains a unique **z-index**, which is a measure of the height of that layer in space. In other words, **z-sorting** places objects on a modulo **Z** axis based on their index.

---
## **Parameters**:

### - *grid_map*
<p style="color:#ffb0e0;">GridMap</p>
DaiconMap Core.

---
### - *cells_count*
<p style="color:#ffb0e0;">int</p>
Number of three-dimensional tiles.

---
### - *mesh_library*
<p style="color:#ffb0e0;">MeshLibrary</p>
Library of meshes from which the 3D environment is built.

---
### - *physics_material*
<p style="color:#ffb0e0;">PhysicsMaterial</p>
Used to determine the physical properties, such as friction and elasticity, of individual tiles.

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
## **Methods**:
### - *_ready*

Deploys the kernel at each startup. Performs basic configuration of the node.

---
### - *_process*

> Works only in the editor.

Updates **grid_map** when the number of 3D tiles is not equal to the number of 2D tiles (calls **update_grid_map**).

Synchronizes moving of node in 2D and its core in 3D. 

---
### - *get_cells*

Returns the number of 3D tiles used in the 3D environment.

```python
func get_cells() -> int:
	_cells_count = 0
	for layer_index in range(0, get_layers_count()):
		_cells_count += len(get_used_cells(layer_index))
	for layer in get_children():
		if layer is TileMapLayer:
			_cells_count += len(layer.get_used_cells())
	return _cells_count
```

---
### - *update_grid_map*

Updates **grid_map**.

```python
func update_grid_map():
	grid_map.clear()
	for layer_index in range(0, get_layers_count()):
		var z = get_layer_z_index(layer_index) / z_step
		for tile in get_used_cells(layer_index):
			var tile_data = get_cell_tile_data(layer_index, Vector2(tile.x, tile.y))
			grid_map.set_cell_item(Vector3(tile.x, z-1, tile.y+z), tile_data.get_custom_data("Item"))
	for layer in get_children():
		if layer is TileMapLayer:
			var z = layer.z_index / z_step
			for tile in layer.get_used_cells():
				var tile_data = layer.get_cell_tile_data(Vector2(tile.x, tile.y))
				grid_map.set_cell_item(Vector3(tile.x, z-1, tile.y+z), tile_data.get_custom_data("Item"))
```