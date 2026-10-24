# Test Case Specification: Роботизированный мегалодон «RoboMeg»

Базовый мегалодон: `RoboMeg("RoboMeg", mass_kg=4000, jump_length_km=4.0, bite_force_kn=60.0, max_speed_kmh=50.0, energy=100.0)`.

Соотношение: 26 модульных / 3 сценарных (~89% / 11%).

## Модульные тесты

| ID | Тип | Источник | Предусловие | Вход | Ожидание | Приоритет |
|---|---|---|---|---|---|---|
| TC-M-001 | module | SPEC 3.1, 3.2, 3.3 | — | создание `RoboMeg` с валидными параметрами | атрибуты сохранены; `position_km == 0.0`, `is_airborne == False` | high |
| TC-M-002 | module | SPEC 3.2, 6.2 | — | `name=""` / `name="   "` | `ValueError` | high |
| TC-M-003 | module | SPEC 3.2, 6.2 | — | `mass_kg=0` / `-100`, `jump_length_km=0`, `bite_force_kn=-5`, `max_speed_kmh=0` | `ValueError` (для каждого неположительного параметра) | high |
| TC-M-004 | module | SPEC 3.2, 6.2 | — | `energy=-0.1` / `100.1`; `energy=0` / `100`; без `energy` | `-0.1` и `100.1` → `ValueError`; `0` и `100` принимаются; по умолчанию → `100.0` | high |
| TC-M-005 | module | SPEC 3.3 | — | попытка перезаписи `meg.energy = 50` или `meg.is_airborne = True` | `AttributeError` (атрибуты только для чтения) | high |
| TC-M-006 | module | SPEC 3.4.1, 4.1 сц. 1 | — | `move(25, "water")` | `0.5`; `position_km == 25.0`, `energy == 75.0`, `is_airborne == False` | high |
| TC-M-007 | module | SPEC 3.4.1, 5 | — | `move(4, "air")` при `jump_length_km == 4` | `0.04`; `position_km == 4.0`, `energy == 92.0`, `is_airborne == True` | high |
| TC-M-008 | module | SPEC 3.4.1, 6.2 | — | `move(4.1, "air")` при `jump_length_km == 4` | `JumpTooLong`; состояние не меняется | high |
| TC-M-009 | module | SPEC 3.4.1, 6.2 | — | `move(1, "ground")` | `RoboMegCannotMoveOnGround`; состояние не меняется | high |
| TC-M-010 | module | SPEC 3.4.1, 5, 6.2 | `energy=10` / `energy=10` | `move(15, "water")` / `move(10, "water")` | `EnergyEmpty` (нехватка); ровно 10 → успешно, `energy == 0.0` | high |
| TC-M-011 | module | SPEC 3.4.2, 4.1 сц. 2 | — | `pass_island(3)` при `jump_length_km == 4` | `0.03`; `position_km == 3.0`, `energy == 94.0`, `is_airborne == True` | high |
| TC-M-012 | module | SPEC 3.4.2, 4.1 сц. 3, 5 | — | `pass_island(4)` / `pass_island(6)` при `jump_length_km == 4` | обплыв по полукругу (`1.5 * D`): для D=6 → `dist = 9.0`, `time = 0.18`, `energy == 91.0`, `is_airborne == False` | high |
| TC-M-013 | module | SPEC 3.4.3, 4.1 сц. 4 | `energy=40`, `position_km=5`, прошёл 5 км по воде | `hunt_fish()` | `energy == 60.0`, `position_km == 5.0`, `is_airborne == False` | high |
| TC-M-014 | module | SPEC 3.4.3, 5, 6.2 | `energy=50.0` / `energy=50.1` | `hunt_fish()` | `RoboMegNotHungry`; состояние не меняется | high |
| TC-M-015 | module | SPEC 3.4.3, 6.2 | на старте / после `move(1, "water")` / в воздухе | `hunt_fish()` | `UnsafeHuntingZone` (так как путь по воде после прыжка/старта < 2.0 км); состояние не меняется | high |
| TC-M-016 | module | SPEC 3.4.3, 6.2 | вызов `hunt_fish()` успешно | повторный `hunt_fish()` в той же точке `position_km` | `NoFishHere`; состояние не меняется | high |
| TC-M-017 | module | SPEC 3.4.4 | в воде / в воздухе | `can_overcome("coral_reef")` | `False` / `True` | medium |
| TC-M-018 | module | SPEC 3.4.4 | `bite_force_kn=50` / `energy=30` / `bite_force_kn=60` и `energy=31` | `can_overcome("iceberg")` | `False` / `False` / `True` | medium |
| TC-M-019 | module | SPEC 3.4.4 | `bite_force_kn=25` и в воде / `bite_force_kn=26` в воде / в воздухе | `can_overcome("steel_net")` | `False` / `True` / `True` | medium |
| TC-M-020 | module | SPEC 3.4.4 | `max_speed_kmh=59` / `energy=40` / в воздухе / `max_speed_kmh=60` и `energy=41` в воде | `can_overcome("whirlpool")` | `False` / `False` / `False` / `True` | medium |
| TC-M-021 | module | SPEC 3.4.4, 6.2 | — | `can_overcome("unknown_obstacle")` | `ValueError` | medium |
| TC-M-022 | module | SPEC 3.4.5 | `energy=0` / `energy=100` | `calculate_energy(10, "water")` / `calculate_energy(5, "air")` / `calculate_energy(1, "ground")` | `10.0` / `10.0` / `RoboMegCannotMoveOnGround`; состояние аппарата не меняется | high |
| TC-M-023 | module | SPEC 3.4.6, 5, 6.2 | `energy=50` | `refuel(30)` / `refuel(1000)` / `refuel(0)` / `refuel(-5)` | `None`, `energy == 80.0`; `energy == 100.0`; `ValueError`; `ValueError` | high |
| TC-M-024 | module | SPEC 3.4.7 | — | `get_status()` | словарь с ключами `name`, `energy`, `position_km`, `is_airborne`, `jump_length_km`, `bite_force_kn` и актуальными значениями | medium |
| TC-M-025 | module | SPEC 3.2, 6.2 | — | `name=123`, `mass_kg="4000"`, `energy=True`, `distance_km=None` | `TypeError` | medium |
| TC-M-026 | module | SPEC 6.1 | — | проверка наследования классов исключений | `RoboMegError` наследует `CrazyVehicleError`, а конкретная ошибка (например, `JumpTooLong`) наследует `RoboMegError` | medium |

## Сценарные тесты

| ID | Тип | Источник | Вход | Ожидание | Приоритет |
|---|---|---|---|---|---|
| TC-S-001 | scenario | SPEC 4.2 «Архипелаг островов и охота после прыжка» | Все шаги сценария из раздела 4.2 | `position_km == 21.0`, `energy == 54.0`, `is_airborne == False` | high |
| TC-S-002 | scenario | SPEC 4.2 «Запрет повторной охоты в одной точке при критическом заряде» | Все шаги сценария из раздела 4.2 | `position_km == 5.0`, `energy == 60.0`, `is_airborne == False` | high |
| TC-S-003 | scenario | SPEC 4.2 «Штурм ледяного барьера, рифа и водоворота» | Все шаги сценария из раздела 4.2 | `position_km == 39.0`, `energy == 47.0`, `is_airborne == False`, успешное прохождение всех проверок | high |