# Ядро

Ядро - это начинка, упакованная в ноды-дайкон для предоставления им трехмерных свойств.

Ядро не видно в редакторе - оно существует в памяти абстрактно. При каждом запуске вашего проекта, оно формируется с нуля, учитывая выставленные для него параметры на отведенной для этого панели. На этой же панели  находятся стандартные параметры ноды на которой основана нода-дайкон.

---
## Ячейки

Ячейки в первую очередь представляют собой ссылки на дочерние ноды ядра.  Они тесно связаны со своими словарями в разделе "Core" на панели параметров. 

!!!Tip
	По-сути, раздел "Core" отвечает сугубо за техническую часть, но вы вполне можете использовать его для отслеживания параметров дочерних нод помещенных в ядро, без вызова непосредственно ядра через код.

Когда вы помещаете свою ноду в ядро (назначаете её ячейке), ячейка сначала меняет родителя помещенной ноды, а затем записывает и отправляет все её параметры в словарь. 

В дальнейшем этот словарь будет использоваться для динамического развертывания ядра, так как помещенные в ядро ноды, точно также как и оно само, становятся абстрактными.

Ячейка:
```python
@export var node_1 : Node:
	set(node):
		if not d3: return
		if node:
			if node_1:
				node_1_properties = {}
			var node_properties = get_node_properties(node)
			node.reparent(d3)
			d3.move_child(node, 0)
			node.position = Vector3(0, 0.5, 0)
			node_1_properties = node_properties
		else:
			node_1_properties = {}
	get():
		if not node_1_properties: return
		return d3.get_node(str(node_1_properties.Name))
```

Её словарь:
```python
@export var node_1_properties : Dictionary:
	set(dict):
		if not dict and node_1:
			node_1.position = Vector3(0, 0, 0)
			node_1.reparent(self)
			get_child(-1).owner = get_tree().edited_scene_root
		node_1_properties = dict
	get():
		return node_1_properties
```

Сегмент кода развертывания дочерних нод ядра для одной ячейки:
```python
if node_1_properties and ClassDB.class_exists(node_1_properties.Class):
	var node = ClassDB.instantiate(node_1_properties.Class)
	node.set_name(node_1_properties.Name)
	d3.add_child(node)
		
	for prop in node_1_properties.Properties:
		node.set(prop, node_1_properties.Properties[prop])
```

---
## Обращение через код

У каждой ноды-дайкон есть переменная, которая представляет собой ядро. К примеру это может быть **d3** или **grid_map**, в случае нод окружения. Вызвав эту переменную, вы получите неограниченный доступ к ядру.

Таким образом вы получаете возможность отслеживать и изменять параметры которые изначально отсутствуют на панели параметров, обращаться к дочерним нодам корневого узла ядра и в целом взаимодействовать с ним на свое усмотрение.

!!!danger
	Будьте осторожны в своих админ-возможностях. Вы должны прекрасно понимать что делаете чтобы не повредить ноду и её ядро.

---
### Child Count

Параметр **child_count** автоматически ведет постоянный счёт дочерних нод ядра. Изменить его вы не можете.

```python
@export var child_count : int:
	set(count):
		pass
	get():
		if not d3: return 0
		return d3.get_child_count()
```

