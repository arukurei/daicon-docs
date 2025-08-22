# DaiconShadow

![kinematic_daicon.png](../assets/images/nodes/daicon_shadow.png)


**DaiconShadow** - ノードは選択したオブジェクトの下に影を生成します。

---
## **パラメーター**:

### - *d3*
<p style="color:#ffb0e0;">CharacterBody3D</p>
DaiconShadowのコア。

---
### *daicon_parent*

<p style="color:#ffb0e0;">Node</p>
影が接続されているノード。

---
### - *tile_size*
<p style="color:#ffb0e0;">int</p>
タイルサイズとは、3D空間において1メートルに相当するピクセル数を決定するものです。
（要するに、3D空間におけるセルサイズあたりのタイルサイズを指します）

*自動的に**daicon_parent***と同期されます。

---
### - *z_step*
<p style="color:#ffb0e0;">int</p>
Zステップは、高さレベル間のソートシステムにおけるステップです。

例えば、**z_step** = 10の場合、次のように設定されます：

レベル -1 = -10
レベル 0 = 0
レベル 1 = 10
レベル 2 = 20

*自動的に**daicon_parent***と同期されます。

---
### - *min_distance*
<p style="color:#ffb0e0;">int</p>
テクスチャのモジュレーションの最小距離（メートル単位）。

- 着色の場合、最適な値は1以上です；
- 脱色の場合、最適な値は1未満です。

---
### - *max_distance*
<p style="color:#ffb0e0;">int</p>
親と自分との最大距離（メートル単位）。

---
### *shadow_mode*
<p style="color:#ffb0e0;">int</p>
影のモジュレーションモード: 「色あせ」、「着色」

---
### *stream_mode*
<p style="color:#ffb0e0;">int</p>

物理ボディのコアの動作モード: 「Logic」, 「Direct」

**Logic** ストリームは、ボディが表面上にあるかどうか（is_on_floor）を確認します。**Direct** では確認しません。

---
### - *shape*
<p style="color:#ffb0e0;">Node3D</p>
コアに組み込まれるシェイプノード用のセル（衝突処理用）。
**CollisionShape3D**または**CollisionPolygon3D**のみを通過させます。
「Shape」セクションに独自の辞書**shape_properties**を持っています。

---
### *Shape-раздел*

Shape のパラメーターを含みます。

---
### *KinematicBody3D-раздел*

カーネルのルート・ノードのパラメータ・セクション。

`(ドキュメントを見る Godot : CharacterBody3D / KinematicBody3D)`

---
### *CollisionObject3D-раздел*

カーネルのルート・ノードのパラメータ・セクション。

`(ドキュメントを見る Godot : CollisionObject3D)`

!!!info
	**axis_lock** も含まれる。

---
## **方法**:
## - *_ready*

各起動時にカーネルをデプロイする。ノードの基本設定を行います。

---
### - *_process*

Синхронизирует перемещение ноды в 2D и её ядра в 3D. 

Добавляет в исключение коллизии с **daicon_parent**, обновляет модуляцию тени (срабатывает единожды при запуске).

---
### - *physics_process*

コアの位置を更新し、z_indexを更新し、影を描画します。

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

この機能は、影の透明度を更新します。

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

この機能は、オブジェクトの2次元空間内の位置を更新し、ノードのz_indexを決定します。
（論理的な流れのため）

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

この機能は、オブジェクトの2次元空間内の位置を更新し、ノードのz_indexを決定します。
（直流用）

---
### - *_expand*

```java
func _expand() -> void:
	_expand_d3()
	if shape_properties:
		_expand_shape()
```

この関数は、カーネルのデプロイメントを扱う。