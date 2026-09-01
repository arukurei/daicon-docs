# DaiconMapLayer

![daicon_map_layer.png](../assets/images/nodes/daicon_map_layer.png)

**DaiconMapLayer** — 独立した3D `GridMap` コアを内蔵する単一の `TileMapLayer` ノードです。

複数レイヤーをまとめる `DaiconMap` とは異なり、**1つのタイルレイヤー** を専任で管理します。取り外し可能な屋根、橋、破壊可能な足場、個別の透過シェーダーを適用したい壁など、特殊な制御が必要なオブジェクトに最適です。

---

## 特徴

* **独立した3Dコア:** 他のレイヤーから独立した専用の `GridMap` インスタンスを保持します。
* **独立 Z-Index:** 自身の `z_index` の値に基づいて3Dの高さが直接計算されます：
   $$\text{3D座標} = (\text{タイルX}, \ \text{z\_index} - 1, \ \text{タイルY} + \text{z\_index})$$
* **透過シェーダーとの相性:** 単一ノードであるため、ルートノード `Daicon` の `shader_target_nodes` に直接登録して、屋根や壁の裏側にいるキャラクターを透過表示させる処理が極めて容易です。