# DaiconMapLayer

![daicon_map_layer.png](../assets/images/nodes/daicon_map_layer.png)

Представляет собой единую локальную независимую ноду **TileMapLayer** со встроенным ядром **DaiconMap**.

Таким образом **DaiconMapLayer** - это нода поддержки или нода, представляющая отдельный элемент вашего окружения, который требует особого внимания.

---
## **Параметры**:

### - *grid_map*
<p style="color:#ffb0e0;">Array[Node]</p>
Ядро DaiconMap.

---
### - *mesh_library*
<p style="color:#ffb0e0;">MeshLibrary</p>
Библиотека меша из которого строиться 3D окружение.

---
### - *physics_material*
<p style="color:#ffb0e0;">PhysicsMaterial</p>
Используется для определения физических свойств, таких как трение и упругость, отдельных тайлов.

---
### - *size*
<p style="color:#ffb0e0;">Vector3</p>
Размер одного трехмерного тайла в метрах.

---
### - *layer*
<p style="color:#ffb0e0;">int</p>
Слои столкновений для **grid_map**.

---
### - *mask*
<p style="color:#ffb0e0;">int</p>
Слои столкновений для **grid_map**.

---
### - *bake_navigation*
<p style="color:#ffb0e0;">bool</p>
Запечь навигационную сетку для 3D.

---
## **Методы**:
### - *_ready*

При каждом запуске развертывает ядро. Проводит базовую настройку ноды.

---
### - *_process*

> Работает только в редакторе.

Обновляет **grid_map** когда количество трехмерных тайлов не равно количеству двумерных (вызывает **update_grid_map**).

Синхронизирует перемещение ноды в 2D и её ядра в 3D. 

---
### - *update_grid_map*

Обновляет **grid_map**.

```python
func update_grid_map():
	grid_map.clear()
	for tile in get_used_cells():
		var tile_data = get_cell_tile_data(Vector2(tile.x, tile.y))
		grid_map.set_cell_item(Vector3(tile.x, z_index-1, tile.y+z_index), tile_data.get_custom_data("Item"))
```