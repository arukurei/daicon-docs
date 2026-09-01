# RigidDaicon

![rigid_daicon.png](../assets/images/nodes/rigid_daicon.png)

**RigidDaicon** — это нода для объектов с полноценной симуляцией 3D-физики: ящиков, толкаемых бочек, катящихся валунов и падающих обломков.

Внутри неё работает ядро **`RigidBody3D`**. Его перемещением управляет физический движок Godot с учетом массы, гравитации, трения и импульсов от ударов, а нода непрерывно проецирует положение и вращение тела на 2D-экран.

---

## Возможности и физика

* **Масса и гравитация:** Настраивайте вес объекта (`mass`), упругость материалов (`physics_material_override`) и индивидуальную шкалу гравитации (`gravity_scale`).
* **Импульсы и силы:** Можно толкать тело через стандартные методы 3D-физики (`apply_central_impulse()`, `apply_torque_impulse()`).
* **Заморозка (Freeze):** Поддерживает режим `freeze` (Static или Kinematic) для создания объектов, которые «просыпаются» и начинают падать только при наступлении определенного события.
* **Автоматическая синхронизация:** Нода самостоятельно вызывает `update_pos()` каждый физический тик (`_physics_process`), поэтому 2D-спрайт всегда следует за физическим телом.

---

## Пример взаимодействия через код

Чтобы толкнуть объект (например, от взрыва или удара игрока):

```gdscript
@tool
extends RigidDaicon

func _physics_process(delta: float) -> void:
    super._physics_process(delta)
    if Engine.is_editor_hint(): return

    # Пример: приложить импульс при взаимодействии
    var body := core as RigidBody3D
    if body and Input.is_action_just_pressed("ui_select"):
        body.apply_central_impulse(Vector3(0, 5.0, -2.0))
```

---

## Настройка

1. Добавьте **RigidDaicon** в сцену.
2. В слот **Shape Node** назначьте форму коллизии (`BoxShape3D`, `SphereShape3D` или `CylinderShape3D`).
3. В инспекторе настройте массу (`mass`) и параметры затухания скорости (`Linear / Angular Damp`).
4. Назначьте форму в **Whisker Shape Node** для корректной сортировки `z_index`.