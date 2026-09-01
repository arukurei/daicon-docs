# AnimatedDaicon

![animated_daicon.png](../assets/images/nodes/animated_daicon.png)

**AnimatedDaicon** — это нода для движущихся препятствий, лифтов, дверей, ловушек и летающих платформ.

Внутри неё работает ядро **`AnimatableBody3D`**. Оно двигается по заданной траектории (через `AnimationPlayer`, `Tween` или код) и способно толкать и перевозить стоящих на нем персонажей без дрожания физики.

---

## Особенности

* **Sync To Physics (`sync_to_physics_3d`):** Автоматически синхронизирует движение анимации с тиками физического движка. Персонажи, стоящие на платформе, двигаются вместе с ней идеально плавно.
* **Постоянная скорость:** Поддерживает `constant_linear_velocity_3d` для создания движущихся эскалаторов и лент.

---

## Пример управления через код / Tween

Если вы управляете платформой через код, используйте скрипт с шаблоном `AnimatedDaicon`:

```gdscript
@tool
extends AnimatedDaicon

func _ready() -> void:
    super._ready()
    if not Engine.is_editor_hint():
        # Пример простого движения платформы туда-обратно
        var body := core as AnimatableBody3D
        var tween := create_tween().set_loops()
        tween.tween_property(body, "position:x", body.position.x + 3.0, 2.0)
        tween.tween_property(body, "position:x", body.position.x, 2.0)
```
