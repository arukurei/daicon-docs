# Daicon

![daicon.png](../assets/images/daicon.png)

Daicon - корневая нода сцены, отвечающая за шейдеры.

---
## **Параметры**:

### - *shader_trigger_nodes*
<p style="color:#ffb0e0;">Array[Node]</p>
Список узлов-триггеров использующих **ShaderCast** или другие механизмы проектирования шейдеров на узлах-целях.

!!!note
	Нода-триггер обязательно должна содержать ShaderCast или другой механизм проектирования шейдера. Иначе будет ошибка.

---
### - *shader_target_nodes*
<p style="color:#ffb0e0;">Array[Node]</p>
Список узлов-целей. Отрисовывают шейдер, используя переданную информацию позиции от узлов-триггеров.

---
### - *PositionArray*
<p style="color:#ffb0e0;">Array[Vector2]</p>
Динамически обновляемый список позиций всех шейдеров на экране.

---
## **Методы**:

### - *_physics_process*

```java
func _physics_process(delta: float) -> void:
(1) shader_target_nodes.sort_custom(func(a, b): return a.z_index < b.z_index)
	for shader_target in shader_target_nodes:
	(2) PositionArray.clear()
	(3) shader_trigger_nodes.sort_custom(func(a, b): return a.z_index < b.z_index)
		for shader_trigger in shader_trigger_nodes:
		(4) if shader_trigger.shader_cast.is_colliding() and shader_target.z_index >= shader_trigger.z_index:
			(5) PositionArray.append(get_viewport().get_final_transform() * shader_trigger.get_global_transform_with_canvas() * Vector2(0,0))
	(6) shader_target.material.set_shader_parameter("CircleCentres", PositionArray)
		shader_target.material.set_shader_parameter("NumCircleCentres", PositionArray.size())
```

1. Сортировка списка узлов-целей за z-индексом.
2. Очистка списка позиций.
3. Сортировка списка триггеров за z-индексом
4. Проверка: активен ли ShaderCast и если z-индекс цели выше за z-индекс триггера
5. Заносим координаты триггера в список позиций
6. Вносим координаты в параметр шейдера

---
### - *_ready*

```java
func _ready() -> void:
	self.set_y_sort_enabled(true)
```

Параметр **set_y_sort_enabled** всегда равен **true**.