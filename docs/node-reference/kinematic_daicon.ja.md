# KinematicDaicon

![kinematic_daicon.png](../assets/images/nodes/kinematic_daicon.png)

**KinematicDaicon** — プレイヤーキャラクター、敵、NPC、その他キネマティックな移動を行うあらゆるオブジェクトを作成するための主要ノードです。

内部には完全な3Dコア **`CharacterBody3D`** が組み込まれています。Godotの使い慣れたメソッド（`velocity`、`move_and_slide()`、`is_on_floor()`）でボディを操作でき、ジャンプ、壁のすり抜け、坂道の昇降などすべての挙動が自動的に2D画面へ投影されます。

---

## コード

シーンにノードを追加し、右クリック → **「スクリプトを拡張」** から `KinematicDaicon` テンプレートを選択します。

以下は、すぐに使える基本的なキャラクター移動のコード例です：

```gdscript
@tool
extends KinematicDaicon

const SPEED := 5.0
const JUMP_VELOCITY := 4.5
const GRAVITY := 9.8

func _physics_process(delta: float) -> void:
    # エディタ上ではゲームロジックを実行しない
    if Engine.is_editor_hint(): return
    
    # 1. 3Dコアを取得
    var body := core as CharacterBody3D
    if not body: return

    # 2. 2D入力を読み取り、3D（X軸およびZ軸）へ変換
    var input_dir := Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
    var direction := Vector3(input_dir.x, 0.0, input_dir.y).normalized()

    # 3. 水平方向の移動
    if direction != Vector3.ZERO:
        body.velocity.x = direction.x * SPEED
        body.velocity.z = direction.z * SPEED
    else:
        body.velocity.x = move_toward(body.velocity.x, 0.0, SPEED)
        body.velocity.z = move_toward(body.velocity.z, 0.0, SPEED)

    # 4. 重力とジャンプ（Y軸）
    if not body.is_on_floor():
        body.velocity.y -= GRAVITY * delta
    elif Input.is_action_just_pressed("ui_accept"):
        body.velocity.y = JUMP_VELOCITY

    # 5. 移動を実行し、2Dスプライトと同期
    body.move_and_slide()
    update_pos()
```

> [!TIP] 3DのZ軸は画面の縦方向（奥行き）に対応
> 入力値 `input_dir.y`（キーボードの上下）が3D空間の `direction.z` に割り当てられている点に注目してください。これによりキャラクターがシーンの奥行き方向へ移動し、`Y` 軸はジャンプや落下を担当します。

---

## コリジョンの設定手順

1. シーンに `CollisionShape3D` ノード（`CapsuleShape3D` や `BoxShape3D` など）を追加します。
2. `KinematicDaicon` のインスペクタで、**Shape Node** スロットに作成したノードを割り当てます。
3. ノードが2Dシーンツリーから消え、コアの内部へと注入されます。
4. キャラクターが壁の後ろに隠れた際に正しくソートされるよう、**Whisker Shape Node** にも形状を設定します。