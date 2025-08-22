# KinematicDaicon

![kinematic_daicon.png](../assets/images/nodes/kinematic_daicon.png)

**KinematicDaicon** - нода представляющая кинематический обьект. В ней собрано всё необходимое чтобы создать двигающийся и взаимодействующий обьект.

---
## **Параметры**:

### - *d3*
<p style="color:#ffb0e0;">CharacterBody3D</p>
Ядро KinematicDaicon.

---
### - *whisker*
<p style="color:#ffb0e0;">Area3D</p>
Отслеживает столкновения и работает в комплексе с **y** и **z-сортировками** для правильной отрисовки обьектов. Он определяет находиться ли обьект за заграждением или нет.

---
### - *shader_cast*
<p style="color:#ffb0e0;">RayCast3D</p>
**ShaderCast** - нода особого назначения. Её цель - определять столкновения с обьектами перед игроком и на основе этого рисовать шейдер.

---
### - *tile_size*
<p style="color:#ffb0e0;">int</p>
Размер плитки определяет, сколько пикселей соответствует 1 метру в 3D.
(по сути, это размер плитки на размер ячейки в 3D)

---
### - *y_3d*
<p style="color:#ffb0e0;">int</p>
Позиция персонажа на оси Z.

---
### - *z_step*
<p style="color:#ffb0e0;">int</p>
Z-шаг в системе сортировки между уровнями высоты.

Например **z_step** = 10, тогда:

Уровень -1 = -10
Уровень 0 = 0
Уровень 1 = 10
Уровень 2 = 20

---
### - *mesh*
<p style="color:#ffb0e0;">MeshInstance3D</p>
Ячейка для меш-ноды которая встраивается в ядро (после установки вы можете увидеть её в разделе 3D).
Имеет собственный словарь в разделе "Core": **mesh_properties**.

---
### - *shape*
<p style="color:#ffb0e0;">Node3D</p>
Ячейка для шейп-ноды которая встраивается в ядро (нужна для столкновений).
Пропускает только **CollisionShape3D** или **CollisionPolygon3D**.
Имеет собственный словарь в разделе "Core": **shape_properties**.

---
### *Mesh & Shape-раздел*

Раздел "Mesh & Shape" содержит параметры для Mesh и Shape.

---
### *Slots-раздел*
<p style="color:#ffb0e0;">Node3D</p>
Слоты - ячейки для нод разработчика если потребуется внедрить их в ядро (связь только через код).

---
### *Core-раздел*
#### - *child_count*
<p style="color:#ffb0e0;">int</p>
Ведет постоянный счёт количества дочерних нод ядра.

---
#### - *properties*
<p style="color:#ffb0e0;">Dictionary</p>
Словари параметров нод, занесенных в ядро через ячейки. Хранят все параметры необходимые для динамического развертывания дочерних нод ядра.

---
### *KinematicBody3D-раздел*

Раздел параметров для корневой ноды ядра. 

`(Смотрите документацию Godot : CharacterBody3D / KinematicBody3D)`

---
### *CollisionObject3D-раздел*

Раздел параметров для корневой ноды ядра. 

`(Смотрите документацию Godot : CollisionObject3D)`

!!!info
	Также содержит **axis_lock**.

---
### *RayCast-раздел*

Раздел параметров для нод **Whisker** и **ShaderCast**. 

`(Смотрите документацию Godot : Area3D - RayCast3D)`

---
## **Методы**:
## - *_ready*

При каждом запуске развертывает ядро. Проводит базовую настройку ноды.

---
### - *_process*

> Работает только в редакторе.

Синхронизирует перемещение ноды в 2D и её ядра в 3D. 

---
### - *update_pos*

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

Функция обновляет позицию обьекта в 2D пространстве передавая ей трехмерные координаты ядра.

coef определяет высоту обьекта.

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

Функция записывает все параметры ноды, её имя, а также класс в словарь и возвращает его.

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

Функция занимается развертыванием ядра.