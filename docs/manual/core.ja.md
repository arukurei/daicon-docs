# カーネル

芯は、大根の節に詰められ、立体的な性質を与える詰め物である。

カーネルはエディタには表示されず、抽象的にメモリ上に存在します。プロジェクトを開始するたびに、この目的のために割り当てられたパネルで設定されたパラメータを考慮して、カーネルはゼロから形成されます。同じパネルには daicon-node のベースとなるノードの標準パラメータが含まれています。

---
## セル

セルは主にコアの子ノードへの参照である。 それらは、オプション・パネルの 「Core 」セクションにある辞書と密接にリンクしている。

!!!Tip
	実際、「Core」セクションは純粋に技術的なものだが、コードを通して直接カーネルを呼び出すことなく、カーネル内に配置された子ノードのパラメーターを追跡するために使用できる。

ノードをコアに配置する（セルに割り当てる）と、セルはまず配置されたノードの親を変更し、それからすべてのパラメータを辞書に書き込んで送信する。

このボキャブラリーは、カーネル内に配置されたノードがカーネルそのものと同じように抽象化されると、カーネルを動的にデプロイするために使われる。

セル:
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

彼女の語彙:
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

単一セルのカーネル子ノード展開コードのセグメント:
```python
if node_1_properties and ClassDB.class_exists(node_1_properties.Class):
	var node = ClassDB.instantiate(node_1_properties.Class)
	node.set_name(node_1_properties.Name)
	d3.add_child(node)
		
	for prop in node_1_properties.Properties:
		node.set(prop, node_1_properties.Properties[prop])
```

!!!Warning
	ノードがコアに潜り込むと、その位置は以前の位置に関係なくコアの中心に移動し、0.5だけ下がるという事実を考慮に入れてください：
	
	`node.position = Vector3(0, 0.5, 0)`

---
## コードによるアドレス指定

それぞれのダイコンノードにはコアを表す変数があります。例えば環境ノードであれば **d3** や**grid_map** などです。この変数を呼び出すことでコアに無制限にアクセスすることができます。

そのため、パラメータ・パネルにないパラメータを追跡して変更したり、カーネルのルート・ノードの子ノードにアクセスしたり、カーネルを自由に操作することができる。

!!!danger
	管理者権限には注意してください。ノードとそのコアにダメージを与えないように、自分のしていることを正確に知っておくべきだ。

---
### Child Count

**child_count** パラメーターは、自動的にカーネルの子ノードの数を一定に保ちます。これを変更することはできない。

```python
@export var child_count : int:
	set(count):
		pass
	get():
		if not d3: return 0
		return d3.get_child_count()
```

