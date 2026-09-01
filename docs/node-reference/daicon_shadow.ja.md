# DaiconShadow

![daicon_shadow.png](../assets/images/nodes/daicon_shadow.png)

**DaiconShadow** — 3Dサーフェススキャナーを内蔵した2Dスプライトノードです。3D空間の高低差をリアルタイムに計算し、空中にいる際はスムーズに透明度を変化させながら、足元へリアルな影を投影します。

物理ボディを使用していた旧仕様とは異なり、新バージョンでは軽量な円柱状キャスト（`ShapeCast3D`）を採用。物理エンジンへの負荷を最小限に抑えつつ、自動的に足元の床を検出します。

---

## 仕組み

手動でノードの参照を接続する必要はありません：

1. 任意のDaiconエンティティ（`KinematicDaicon`、`RigidDaicon` など）の子ノードとして **DaiconShadow** を追加します。
2. 影ノードが `find_parent_entity()` で自動的に親を検出し、その3Dコア内部へ円柱状の `ShapeCast3D` センサーを自動注入します。
3. センサーが常時真下をスキャンして足元の最も高い床面を検知し、正しい `z_index` と共に2D影スプライトを画面上の適切な高さへ投影します。

```mermaid
graph LR
    A(["🚀 3Dコア core"]) -->|"自動注入"| B(["🔍 ShapeCast3D"])
    B -->|"床面をスキャン"| C[("🧱 地面・障害物")]
    C -->|"高さ・距離データ"| D(["👥 2D DaiconShadow<br><small>Y位置 · Z-Index · 透明度</small>"])

    classDef purple fill:#f3e8ff,stroke:#9333ea,stroke-width:1.5px,color:#581c87;
    classDef blue fill:#e0f2fe,stroke:#0284c7,stroke-width:1.5px,color:#0369a1;
    classDef amber fill:#fef3c7,stroke:#d97706,stroke-width:1.5px,color:#92400e;
    classDef emerald fill:#d1fae5,stroke:#059669,stroke-width:1.5px,color:#065f46;

    class A purple;
    class B blue;
    class C amber;
    class D emerald;
```

---

> [!TIP] スプライト位置の簡単調整
> インスペクタで **Debug Ray** を有効にすると、2Dビューポート上にセンサーの実際の半径と床までの高さラインが表示されます。キャラクターの足元に合わせてテクスチャサイズやオフセットを素早く調整できます。

---

## ゲーム内の挙動とソート

1. **接地時 (`is_on_floor`):** 遅延や補間のズレなく、影が瞬時に足元へ固定されます。
2. **空中時:** 高さに応じて影がスムーズにフェードアウトします（`fade_start_distance` から `max_distance` の間）。
3. **スマート Z-Index:** 検出した床面の高さを自動計算し、常にキャラクターの足元の下層に正しく描画されます（`z_index = min(floor_z, parent.z_index - 1)`）。

> [!WARNING] 階層構造のルール
> `DaiconShadow` は必ず `DaiconEntity` の子孫ノード（階層の深さは任意）として配置してください。エンティティの外に配置すると、エディタのシーンツリーに設定警告が表示されます。