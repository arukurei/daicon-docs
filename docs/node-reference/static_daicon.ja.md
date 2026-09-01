# StaticDaicon

![static_daicon.png](../assets/images/nodes/static_daicon.png)

**StaticDaicon** — 壁、岩、木、柱、柵など、静止した環境オブジェクトを配置するためのノードです。

内部で **`StaticBody3D`** コアを生成します。外部の物理的な力によって動くことはなく、プレイヤーや物理オブジェクトに対する堅牢な障害物として機能します。

---

## 特徴と用途

* **コンベア速度の設定:** `constant_linear_velocity_3d` や `constant_angular_velocity_3d` を設定することで、ベルトコンベアのような挙動を簡単に再現できます（オブジェクト自体は静止したまま、上に乗ったキャラクターを押し出します）。
* **自動 Z-Index 同期:** キネマティック体とは異なり、`StaticDaicon` は `_process()` 内で自動的に位置とソートレイヤーを同期するため、基本的な静的オブジェクトであればスクリプトを追加する必要はありません。

---

## 設定手順

1. シーンに **StaticDaicon** を追加します。
2. **Shape Node** スロットに `CollisionShape3D` を設定します。
3. キャラクターが背後に回った際に正しくソートされるよう、**Whisker Shape Node** スロットにも形状を割り当てます。