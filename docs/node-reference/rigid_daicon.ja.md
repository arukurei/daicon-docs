# RigidDaicon

![rigid_daicon.png](../assets/images/nodes/rigid_daicon.png)

**RigidDaicon** — 木箱、押せる樽、転がる岩、瓦礫など、リアルな3D物理シミュレーションによって動くオブジェクト用のノードです。

内部で **`RigidBody3D`** コアを動作させます。質量、重力、摩擦、衝突による衝撃を考慮したGodotの物理エンジンによって移動が計算され、ノードがその位置と回転をリアルタイムに2D画面へ投影します。

---

## 物理挙動と特徴

* **質量と重力:** オブジェクトの重量（`mass`）、表面の反発・摩擦（`physics_material_override`）、個別の重力倍率（`gravity_scale`）を柔軟に調整可能です。
* **インパルスとフォース:** 通常の3D物理メソッド（`apply_central_impulse()`、`apply_torque_impulse()` 等）を使用してオブジェクトを吹き飛ばしたり転がしたりできます。
* **フリーズ機能 (Freeze):** イベントや接触があるまで物理演算を停止させておく `freeze`（Static / Kinematicモード）に対応しています。
* **自動同期:** ゲームプレイ中、`_physics_process()` 内で自動的に `update_pos()` が実行されるため、2Dスプライトが物理ボディからズレる心配がありません。

---

## スクリプトからの操作例

爆発やプレイヤーのキックなどで衝撃を与える例：

```gdscript
@tool
extends RigidDaicon

func _physics_process(delta: float) -> void:
    super._physics_process(delta)
    if Engine.is_editor_hint(): return

    # 例: インタラクション時に衝撃を加える
    var body := core as RigidBody3D
    if body and Input.is_action_just_pressed("ui_select"):
        body.apply_central_impulse(Vector3(0, 5.0, -2.0))
```

---

## 設定手順

1. シーンに **RigidDaicon** を追加します。
2. **Shape Node** スロットに衝突形状（`BoxShape3D`、`SphereShape3D`、`CylinderShape3D` など）を設定します。
3. インスペクタで質量（`mass`）や減衰値（`Linear / Angular Damp`）を調整します。
4. 正しい深度ソートのために **Whisker Shape Node** にも形状を割り当てます。