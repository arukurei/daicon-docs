# DaiconMapLayer

![daicon_map_layer.png](../assets/images/nodes/daicon_map_layer.png)

**DaiconMapLayer** — is a standalone `TileMapLayer` node with its own embedded 3D `GridMap` core.

Unlike the multi-tiered `DaiconMap`, it manages exactly **one tile layer**. It is the ideal tool for modular level segments: removable rooftops, elevated bridges, destructible platforms, or standalone walls that need dedicated shader materials.

---

## Features

* **Dedicated 3D Core:** Operates an isolated `GridMap` instance independent of other map layers.
* **Self-Contained Z-Index:** Height projection is determined directly from the node's own `z_index`:
   $$\text{3D Position} = (\text{Tile X}, \ \text{z\_index} - 1, \ \text{Tile Y} + \text{z\_index})$$
* **Great for Silhouette Shaders:** Being a single discrete node, it can be plugged directly into the `shader_target_nodes` list on the root `Daicon` node to reveal characters walking underneath roofs or overhangs.