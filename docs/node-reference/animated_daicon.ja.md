# AnimatedDaicon

![animated_daicon.png](../assets/images/animated_daicon.png)

AnimatedDaicon - 移動するオブジェクトを表すノードで、自身のボディに影響がないときに他のボディに影響を与えることができる。

---
## **Параметры**:

### - *d3*
<p style="color:#ffb0e0;">CharacterBody3D</p>
StaticDaicon のコア。

---
### - *right_whisker*
<p style="color:#ffb0e0;">RayCast3D</p>
これは衝突を追跡し、オブジェクトを正しくレンダリングするために **y** および **z-sorting** 連動します。オブジェクトがバリアの後ろにあるかどうかを判定します。

---
### - *left_whisker*
<p style="color:#ffb0e0;">RayCast3D</p>
衝突を追跡し、オブジェクトを正しくレンダリングするために **y** および **z-sorting** と連動します。オブジェクトがバリアの後ろにあるかどうかを判定します。

---
### - *center_whisker*
<p style="color:#ffb0e0;">RayCast3D</p>
衝突を追跡し、オブジェクトを正しくレンダリングするために **y** および **z-sorting** と連動します。オブジェクトがバリアの後ろにあるかどうかを判定します。

---
### - *shader_cast*
<p style="color:#ffb0e0;">RayCast3D</p>
**ShaderCast** は特別な目的のノードです。その目的は、プレイヤーの前にあるオブジェクトとの衝突を検出し、それに基づいてシェーダーを描画することです。

---
### - *tile_size*
<p style="color:#ffb0e0;">int</p>
1タイルの大きさ（この値は、文字が1メートルに等しく通過するピクセル数を定義する）。

---
### - *y_3d*
<p style="color:#ffb0e0;">int</p>
キャラクターのZ軸上の位置。

---
### - *mesh*
<p style="color:#ffb0e0;">MeshInstance3D</p>
カーネルに埋め込まれたメッシュ・ノード・セル（インストール後、3Dセクションで見ることができる）。
セクション「**Core : mesh_properties**」に独自の辞書があります。

---
### - *shape*
<p style="color:#ffb0e0;">Node3D</p>
コアに埋め込まれたシェイプノードのセル（コリジョンに必要）。
**CollisionShape3D** または **CollisionPolygon3D** のみをスキップします。
これは「**Core : shape_properties**」/"の下に独自の辞書を持っています。

---
### *Mesh & Shape-セクション*

Mesh & Shape "セクションには、MeshとShapeの**Transform**パラメータが含まれています。

---
### *Slots-セクション*
<p style="color:#ffb0e0;">Node3D</p>
スロット - 開発者ノードをコアに実装する必要がある場合のセル（コード経由の通信のみ）。

---
### *Core-セクション*
#### - *child_count*
<p style="color:#ffb0e0;">int</p>
カーネルの子ノードの数を常にカウントする。

---
#### - *properties*
<p style="color:#ffb0e0;">Dictionary</p>
セルを介してカーネルに格納されるノードパラメータの辞書。カーネルの子ノードの動的展開に必要なすべてのパラメータを格納する。

---
### *AnimatedBody-セクション*

カーネルのルート・ノードのパラメータ・セクション。

`(ドキュメントを見る Godot : CharacterBody3D / AnimatedBody3D)`

---
### *CollisionObject3D-セクション*

カーネルのルート・ノードのパラメータ・セクション。

`(ドキュメントを見る Godot : CollisionObject3D)`

!!!info
	**axis_lock** も含まれる。

---
### *RayCast-セクション*

**Whiskers** と **ShaderCast** ノードのパラメータセクション。

`(ドキュメントを見る Godot : RayCast3D)`

---
## **方法**:
## - *_ready*

各起動時にカーネルをデプロイする。ノードの基本設定を行います。

---
### - *_process*

> エディターでのみ動作します。

2Dでのノードの動きと3Dでのコアの動きを同期させる。

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


この関数は，コアの3次元座標を渡して，2次元空間におけるオブジェクトの位置を更新する．

**first_coef** は、オブジェクトがフェンスの後ろにある場合の z-index に対する係数を決定します。

**second_coef** は、**Whiskers** が何も遭遇していないときの z-index に対する係数を決定します。

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

この関数は、すべてのノードのパラメータ、名前、クラスを辞書に書き込み、それを返します。

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

この関数は、カーネルのデプロイメントを扱う。
