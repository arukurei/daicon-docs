# DaiconMap

![daicon_map.png](../assets/images/nodes/daicon_map.png)

**DaiconMap** — タイルマップシステムを活用して多層構造の2.5D環境を構築するためのメインノードです。

Godotエディタ上で普段通りに2Dタイルを描くだけで、ノード内部の **`GridMap`** を介して本物の3Dボクセルワールドとコリジョンが自動生成されます。

---

## 仕組み

1. **2Dタイルと3Dメッシュの紐付け:** `TileSet` 内で、各タイルに数値パラメータ `Item`（`MeshLibrary` 内のメッシュID）を設定します。
2. **斜め2.5D投影:** 各レイヤーは固有の `z_index` を持ちます。タイルを配置すると、以下の傾斜投影式に基づいて3D座標が自動計算されます：
   $$\text{3D座標} = (\text{タイルX}, \ \text{レイヤーZ} - 1, \ \text{タイルY} + \text{レイヤーZ})$$
3. **エディタ上のリアルタイム同期:** エディタ編集中にタイルの変更を常時監視し、3Dジオメトリとコリジョンを瞬時に再構築します。

```mermaid
graph LR
    A(["🎨 2D レイヤータイル<br><small>異なる Z-Index を持つ TileMap</small>"]) -->|"Item パラメータ (Mesh ID)"| B(["⚙️ DaiconMap コンバータ<br><small>2.5D 傾斜投影計算</small>"])
    B -->|"自動生成"| C(["🧱 3D GridMap<br><small>実際の3Dブロックとコリジョン</small>"])

    classDef purple fill:#f3e8ff,stroke:#9333ea,stroke-width:1.5px,color:#581c87;
    classDef blue fill:#e0f2fe,stroke:#0284c7,stroke-width:1.5px,color:#0369a1;
    classDef emerald fill:#d1fae5,stroke:#059669,stroke-width:1.5px,color:#065f46;

    class A purple;
    class B blue;
    class C emerald;
```

---

## 主な設定項目

* **Mesh Library (`mesh_library`):** マップ構築に使用する3Dブロックモデルのライブラリ（詳細は [メッシュライブラリガイド](../manual/mesh.md) を参照）。
* **Cell Size (`size: Vector3`):** 1つの3Dボクセルのサイズ（メートル単位、デフォルトは `1x1x1`）。
* **Z Step (`z_step: int`):** 高さレベル間の `z_index` のステップ幅（デフォルトは `10`）。例: レベル0 = `z_index 0`、レベル1 = `z_index 10`、レベル2 = `z_index 20`。
* **Collision Layer & Mask:** キャラクターや物理ボディとの衝突を判定する3Dコリジョンレイヤー。
* **Bake Navigation:** 敵の経路探索用 3D `NavigationMesh` のベイク設定。

> [!TIP] レイヤーの個別ノード化
> TileMap内部のレイヤーだけでなく、`DaiconMap` の子ノードとして個別の `TileMapLayer` を追加して描画した場合でも、同一の 3D GridMap へ自動的に統合されます。