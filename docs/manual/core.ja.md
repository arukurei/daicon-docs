# コアとスロットシステム

**コア (Core)** — 2Dオブジェクトに現実の3Dプロパティ（物理挙動、3D空間座標など）を与える、Daiconエンティティ（`DaiconEntity`）内部の3Dの「中身」です。

エディタ編集中、コアは通常のノードとしてシーンツリーには表示されず、自動的に生成・管理されます（メモリ上に抽象的に存在します）。ゲーム起動時やエディタの `@tool` モード時、インスペクタの設定値と保存されたスロットデータに基づいてゼロから展開されます。

---

## アーキテクチャ: 2Dシェル & 3Dコア

すべての `DaiconEntity`（および派生ノード: `KinematicDaicon`, `StaticDaicon`, `RigidDaicon`, `AnimatedDaicon`）は、完全な同期メカニズムで動作します：

1. **2Dノード (シェル / 外観):** 2Dシーンツリーに存在し、スプライト描画、アニメーション、2Dトランスフォーム、画面上の `z_index` を担当します。
2. **3Dノード (コア / `core`):** 内部の子ノード（`CharacterBody3D`, `RigidBody3D`, `StaticBody3D`, `AnimatableBody3D`）として自動生成されます。3D空間での実際の物理演算、コリジョン、移動を制御します。

2Dと3D間の座標・トランスフォームは、`tile_size` と `offset_3d` を考慮した投影計算により自動的に同期されます。

---

## スロットシステム (DaiconSlots)

3Dコアにメッシュ、衝突形状、センサー、カスタムノードを組み込むために、Daiconは **スロットシステム** を採用しています。

スロットを使用すると、エディタ上で任意の3Dノードを選択し、`Dictionary` にシリアライズ（スクリプト、プロパティ、メタデータを保持）した上で、一時ノードをシーンから削除し、非表示の3Dコア内部へと自動注入できます。

> [!INFO] 技術的な仕組み（要約）
> ノードの全情報は専用の `*_properties` 辞書に保存されます。この辞書データをもとに、コア内でスロットノードが動的に復元・展開されます。
> 
> ```mermaid
> graph TD
>     A(["✨ カスタム3Dノード"]) -->|インスペクタで割り当て| B(["⚙️ DaiconSlots.serialize_node()"])
>     B -->|データ化| C(["📦 辞書 *_properties"])
>     C -->|復元| D(["🧩 DaiconSlots.expand_slot()"])
>     D -->|シーンへ注入| E(["🚀 3Dコア core"])
> 
>     classDef purple fill:#f3e8ff,stroke:#9333ea,stroke-width:1.5px,color:#581c87;
>     classDef blue fill:#e0f2fe,stroke:#0284c7,stroke-width:1.5px,color:#0369a1;
>     classDef amber fill:#fef3c7,stroke:#d97706,stroke-width:1.5px,color:#92400e;
>     classDef emerald fill:#d1fae5,stroke:#059669,stroke-width:1.5px,color:#065f46;
> 
>     class A purple;
>     class B,D blue;
>     class C amber;
>     class E emerald;
> ```

### エンティティの標準スロット

各 `DaiconEntity` のインスペクタには **Slots** グループが用意されています：

| インスペクタのスロット | 保存先辞書 | 配置先親ノード | 用途 |
| :--- | :--- | :--- | :--- |
| **Mesh Node** | `mesh_properties` | `core` | デバッグおよびビジュアル用の3Dモデル表示（`MeshInstance3D`）。 |
| **Shape Node** | `shape_properties` | `core` | メインの衝突判定形状（`CollisionShape3D` または `CollisionPolygon3D`）。 |
| **Whisker Node** | `whisker_properties` | `core` | 遮蔽物の検知と動的 `z_index` ソートを行う `Area3D` センサー。 |
| **Whisker Shape Node** | `whisker_shape_properties` | `whisker` | Whiskerセンサー用の衝突形状。自動的に `whisker_node` の子としてアタッチされます。 |
| **Shader Cast Node** | `shader_cast_properties` | `core` | 障害物の背後に入った際のシェーダー透過エフェクトをトリガーする `RayCast3D`。 |

---

## 実際のスロットの使い方

手動で保存ロジックを書く必要はありません。`DaiconSlots` がすべて自動処理します：

* **スロットへのノード登録:** 任意の3Dノード（例: `CollisionShape3D`）をインスペクタのスロット欄にドラッグ＆ドロップします。ノードはデータとして保存され、**2Dシーンツリーから削除されて `core` の内部へと注入されます**。
* **ノードの取り出し (Eject):** リセットアイコンを押す（または `null` を渡す）と、保存されていた設定を維持したまま、ノードが通常のシーンツリーへ即座に復元されます。
* **破損防止機能:** ディスク上でスクリプトの名前を変更したり削除したりしても、シーンが壊れたりデータが消えたりしません。Daiconは安全にエンジン基本クラスへフォールバックし、コンソールで警告します。
* **複雑なノード階層:** ノードに子ノードが含まれている場合、階層全体がスクリプトやリソース、メタデータごとまとめてシリアライズ・復元されます。

> [!TIP] リソースの個別化 (Uniquification)
> エンティティを複製した際、スロット内のリソースは自動的に複製（`make_unique`）されるため、1つのインスタンスを変更しても他のオブジェクトに影響しません。

---

## カスタムスロット (`custom_slots`)

標準の5つのスロットに加えて、`DaiconEntity` にはコードから拡張可能な配列が用意されています：

```gdscript
@export_storage var custom_slots: Array[Dictionary] = []
```

任意の3Dノードや複雑なノードツリーをコード経由でコアへ注入できます：

```gdscript
# 例: カスタム3Dノードをエンティティのコアに組み込む
var my_light = OmniLight3D.new()
var light_data = DaiconSlots.serialize_node(my_light, true)
entity.custom_slots.append(light_data)

# コア内でスロットを展開
DaiconSlots.expand_slot(light_data, entity.core)
```

---

## コードからのコア直接操作

スクリプトからコアとやりとりするには、**`core`** 変数を使用します：

```gdscript
# 例: KinematicDaicon 内部の CharacterBody3D にアクセス
var body := core as CharacterBody3D
if body:
    body.velocity = Vector3(1, 0, 0)
    body.move_and_slide()
    update_pos()
```

> [!DANGER] コア直接操作時の注意点
> `core` ノードを手動で `queue_free()` 等で削除しないでください。速度や座標を変更した場合は、2Dシェルが3Dコアの座標と正しく同期するように、必ず `update_pos()` を呼び出してください。
> 
> 管理者権限を持つ強力な機能ですので、ノードやコアの整合性を損なわないよう、挙動をしっかり理解した上でご活用ください。