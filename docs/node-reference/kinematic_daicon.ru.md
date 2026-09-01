# KinematicDaicon

![kinematic_daicon.png](../assets/images/nodes/kinematic_daicon.png)

**KinematicDaicon** — это главная нода для создания управляемых персонажей, врагов, NPC и любых объектов с кинематическим движением.

Внутри неё скрыто полноценное 3D-ядро **`CharacterBody3D`**. Вы управляете телом через привычные методы Godot (`velocity`, `move_and_slide()`, `is_on_floor()`), а нода сама проецирует все прыжки, скольжения по стенам и подъемы по склонам на 2D-экран.

---

## Код

Создайте ноду в сцене, нажмите правой кнопкой мыши → **Расширить скрипт** и выберите шаблон `KinematicDaicon`.

Вот готовый базовый пример управления персонажем:

```gdscript
@tool
extends KinematicDaicon

const SPEED := 5.0
const JUMP_VELOCITY := 4.5
const GRAVITY := 9.8

func _physics_process(delta: float) -> void:
    # В режиме редактора логику не запускаем
    if Engine.is_editor_hint(): return
    
    # 1. Получаем доступ к 3D-ядру
    var body := core as CharacterBody3D
    if not body: return

    # 2. Считываем 2D-ввод и переводим в 3D (X и Z)
    var input_dir := Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
    var direction := Vector3(input_dir.x, 0.0, input_dir.y).normalized()

    # 3. Горизонтальное перемещение
    if direction != Vector3.ZERO:
        body.velocity.x = direction.x * SPEED
        body.velocity.z = direction.z * SPEED
    else:
        body.velocity.x = move_toward(body.velocity.x, 0.0, SPEED)
        body.velocity.z = move_toward(body.velocity.z, 0.0, SPEED)

    # 4. Гравитация и прыжок (ось Y)
    if not body.is_on_floor():
        body.velocity.y -= GRAVITY * delta
    elif Input.is_action_just_pressed("ui_accept"):
        body.velocity.y = JUMP_VELOCITY

    # 5. Выполняем перемещение и синхронизируем 2D-спрайт
    body.move_and_slide()
    update_pos()
```

> [!TIP] Ось Z в 3D — это вертикаль экрана
> Обратите внимание: ввод `input_dir.y` (вверх/вниз на клавиатуре) передается в `direction.z` 3D-мира. Так персонаж ходит вглубь сцены, а за прыжки и падения отвечает ось `Y`.

---

## Настройка коллизии

1. Добавьте в сцену узел `CollisionShape3D` (например, с формой `CapsuleShape3D` или `BoxShape3D`).
2. В инспекторе `KinematicDaicon` выберите созданную ноду в поле **Shape Node**.
3. Узел исчезнет из 2D-дерева и внедрится внутрь ядра.
4. Назначьте форму в **Whisker Shape Node**, чтобы персонаж корректно скрывался за стенами.