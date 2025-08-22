# DaiconMap

![daicon-map.png](../assets/images/nodes/daicon_map.png)

これは **TileMapLayers** のセットであり、順番にあなたの環境のレイヤーである。

このような各レイヤーは一意の **z-index** を含み、これは空間におけるそのレイヤーの高さの尺度です。言い換えると、**z-ソート** は、そのインデックスに基づいて、変更可能な **Z** 軸上にオブジェクトを配置します。

---
## **パラメーター**:

### - *grid_map*
<p style="color:#ffb0e0;">GridMap</p>
DaiconMapコア。

---
### - *cells_count*
<p style="color:#ffb0e0;">int</p>
立体的なタイルの数。

---
### - *mesh_library*
<p style="color:#ffb0e0;">MeshLibrary</p>
3D環境を構築するメッシュのライブラリ。

---
### - *physics_material*
<p style="color:#ffb0e0;">PhysicsMaterial</p>
個々のタイルの摩擦や弾性などの物理的特性を測定するために使用される。

---
### - *z_step*
<p style="color:#ffb0e0;">int</p>
Zステップは、高さレベル間のソートシステムにおけるステップです。

例えば、**z_step** = 10の場合、次のように設定されます：

レベル -1 = -10
レベル 0 = 0
レベル 1 = 10
レベル 2 = 20

---
### - *size*
<p style="color:#ffb0e0;">Vector3</p>
メートル単位の立体タイルの大きさ。

---
### - *layer*
<p style="color:#ffb0e0;">int</p>
**grid_map** のコリジョンレイヤー。

---
### - *mask*
<p style="color:#ffb0e0;">int</p>
**grid_map** のコリジョンレイヤー。

---
### - *bake_navigation*
<p style="color:#ffb0e0;">bool</p>
3D用のナビゲーショングリッドを焼く。

---
## **方法**:
### - *_ready*

各起動時にカーネルをデプロイする。ノードの基本設定を行います。

---
### - *_process*

> エディターでのみ動作します。

3Dタイルの数が2Dタイルの数と等しくない場合に **grid_map** を更新する（**update_grid_map**を呼び出す）。

2Dのノードの動きと3Dのコアの動きを同期させる。

---
### - *get_cells*

3D環境で使用されている3Dタイルの数を返します。

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

**grid_map** を更新。

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