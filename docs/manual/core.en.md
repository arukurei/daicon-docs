# Core

The core - is the stuffing packed into dicon nodes to give them three-dimensional properties.

The core is not visible in the editor - it exists in memory abstractly. Each time you start your project, it is formed from scratch, taking into account the parameters set for it on the panel allocated for this purpose. The same panel contains the standard parameters of the node on which the daikon node is based.

---
## Slots

Slots are primarily references to child nodes of the core.  They are closely linked to their dictionaries in the “Core” section of the options panel. 

!!!Tip
	Essentially, the “Core” section is purely technical, but you can use it to keep track of the parameters of child nodes placed in the core, without calling the core directly through code.

When you place your node in the core (assign it to a slot), the slot first changes the parent of the placed node, and then records and sends all its parameters to a dictionary. 

This dictionary will then be used to dynamically deploy the kernel, as the nodes placed in the kernel, just like the kernel itself, become abstract.

Slot:
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

Its Dictionary:
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

A segment of the kernel child node deployment code for a single slot:
```python
if node_1_properties and ClassDB.class_exists(node_1_properties.Class):
	var node = ClassDB.instantiate(node_1_properties.Class)
	node.set_name(node_1_properties.Name)
	d3.add_child(node)
		
	for prop in node_1_properties.Properties:
		node.set(prop, node_1_properties.Properties[prop])
```

!!!Warning
	Take into account the fact that when a node is dipped into the kernel, its position, regardless of its previous position, moves to the center of the kernel and drops by 0.5:
	
	`node.position = Vector3(0, 0.5, 0)`.

---
## Addressing through code

Each dicon node has a variable that represents the core. For example, this could be **d3** or **grid_map**, in the case of environment nodes. By calling this variable, you get unrestricted access to the core.

Thus you get the ability to track and change parameters that are not initially available in the parameter panel, access child nodes of the root node of the kernel and generally interact with it at your discretion.

!!!danger
	Be careful with your admin capabilities. You should know exactly what you are doing so as not to damage the node and its kernel.
	
---
### Child Count

The **child_count** parameter automatically keeps a constant count of the kernel's child nodes. You cannot change it.

```python
@export var child_count : int:
	set(count):
		pass
	get():
		if not d3: return 0
		return d3.get_child_count()
```

