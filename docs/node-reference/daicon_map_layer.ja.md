# DaiconMapLayer

![daicon_map_layer.png](../assets/images/nodes/daicon_map_layer.png)

**TileMapLayer** は、**DaiconMap** のコアを埋め込んだ、単一のローカルな独立したノード**TileMapLayer** を表します。

したがって、**DaiconMapLayer** はサポートノード、または特別な注意を必要とする環境の個々の要素を表すノードです。

---
## **パラメーター**:

### - *grid_map*
<p style="color:#ffb0e0;">Array[Node]</p>
DaiconMapコア。

---
### - *mesh_library*
<p style="color:#ffb0e0;">MeshLibrary</p>
3D環境を構築するメッシュのライブラリ。

---
### - *physics_material*
<p style="color:#ffb0e0;">PhysicsMaterial</p>
個々のタイルの摩擦や弾性などの物理的特性を測定するために使用される。

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
### - *update_grid_map*

**grid_map** を更新。

```python
func update_grid_map():
	grid_map.clear()
	for tile in get_used_cells():
		var tile_data = get_cell_tile_data(Vector2(tile.x, tile.y))
		grid_map.set_cell_item(Vector3(tile.x, z_index-1, tile.y+z_index), tile_data.get_custom_data("Item"))
```