# Instrucciones GitHub Copilot - Darkfrost Roguelike

## Contexto del Proyecto
- **Proyecto**: Darkfrost - Roguelike de acción 2D top-down con mecánicas de evolución de personaje
- **Motor**: Godot 4.3+
- **Lenguaje**: GDScript (tipado)
- **Característica Principal**: Gameplay en tiempo real con pausa táctica; sistema de evolución lineal (Forma A→B→C)

## Estándares GDScript y Godot

### Tipado Estático (OBLIGATORIO)
Siempre usar tipado estático en todo el código:
```gdscript
# ✅ CORRECTO
var health: int = 100
var position: Vector2 = Vector2(0, 0)
var abilities: Array[AbilityData] = []

func take_damage(amount: int) -> void:
    health -= amount

func calculate_damage() -> int:
    return base_damage + bonus

# ❌ INCORRECTO
var health = 100
func take_damage(amount):
    health -= amount
```

### Convenciones de Nombres
- **Archivos & funciones & variables**: `snake_case` (ej: `player_character.gd`, `calculate_damage()`, `max_health`)
- **Clases & nombres de nodos**: `PascalCase` (ej: `class_name PlayerCharacter`, nodo llamado `Player`)
- **Constantes**: `SCREAMING_SNAKE_CASE` (ej: `const MAX_VELOCITY: float = 500.0`)
- **Señales**: Verbos en pasado (ej: `signal health_changed`, `signal ability_used`)
- **Enums**: `PascalCase` con valores `SCREAMING_SNAKE_CASE`
  ```gdscript
  enum FormState { IDLE, ATTACKING, EVADING }
  ```

### Estructura de Código
1. **Declaración de clase** (si aplica)
2. **Señales**
3. **Variables exportadas** (@export)
4. **Variables miembro privadas**
5. **Referencias Onready** (@onready)
6. **Métodos de ciclo de vida** (_ready, _process, _physics_process)
7. **Métodos públicos**
8. **Métodos privados** (prefijo _)
9. **Manejadores de señales** (prefijo _on_)

Ejemplo:
```gdscript
class_name PlayerCharacter
extends CharacterBody2D

signal health_changed(new_health: int)
signal evolution_triggered(new_form: EvolutionData)

@export var speed: float = 150.0
@export var max_health: int = 100

var current_health: int
var current_form: EvolutionData

@onready var sprite: Sprite2D = $Sprite2D
@onready var abilities: AbilityComponent = $AbilityComponent

func _ready() -> void:
    current_health = max_health
    abilities.ability_used.connect(_on_ability_used)

func take_damage(amount: int) -> void:
    current_health -= amount
    health_changed.emit(current_health)
    if current_health <= 0:
        die()

func _on_ability_used(ability: AbilityData) -> void:
    pass
```

## Patrones Específicos de Godot

### Señales & Conexiones
- Usar palabra clave `signal` para declaraciones
- Conectar en método `_ready()`
- Usar decorador `@onready` para cachear referencias de nodos
- Desconectar señales cuando no se necesitan (especialmente en _exit_tree)

```gdscript
# Conexión
abilities.ability_used.connect(_on_ability_used)

# Desconexión (¡importante!)
func _exit_tree() -> void:
    abilities.ability_used.disconnect(_on_ability_used)
```

### Diseño Basado en Componentes
Usar componentes pequeños y enfocados para lógica reutilizable:
```gdscript
# HealthComponent es un nodo separado
class_name HealthComponent
extends Node

signal health_changed(new_health: int)
signal died

var current_health: int
var max_health: int = 100

func take_damage(amount: int) -> void:
    current_health = max(0, current_health - amount)
    health_changed.emit(current_health)
    if current_health == 0:
        died.emit()
```

Luego adjuntar como nodo hijo:
```gdscript
# En player.gd
@onready var health: HealthComponent = $HealthComponent

func _ready() -> void:
    health.died.connect(_on_died)
```

### Recursos para Datos
Usar Godot Resources para datos del juego (no Scripts):
```gdscript
# ✅ Crear recursos en Inspector de Godot o usar código:
@export var item_data: ItemData

# En ItemData.gd (si necesitas métodos personalizados):
class_name ItemData
extends Resource

@export var name: String
@export var icon: Texture2D
@export var effect_type: String
```

### Sistema de Pausa con get_tree().paused
```gdscript
# Habilitar pausa
get_tree().paused = true
pause_menu.show()

# Deshabilitar pausa
get_tree().paused = false
pause_menu.hide()

# Nota: Los nodos de UI con CanvasLayer aún se procesan mientras está pausado
```

## Evolution System Architecture

### Evolutions are Linear
Each form has an optional `next_evolution` pointing to the next form:
```gdscript
class_name EvolutionData
extends Resource

@export var form_name: String
@export var sprite: Texture2D
@export var base_stats: Dictionary  # { "health": 120, "speed": 160, ... }
@export var abilities: Array[AbilityData]
@export var next_evolution: EvolutionData  # null if final form
```

### Evolution on Level-Up
When player levels up, show evolution menu (pause game):
1. Display available evolution options (just 1 in linear progression)
2. Player selects evolution
3. Update player sprite, stats, abilities
4. Emit `evolution_chosen` signal via EventBus
5. Unpause game

```gdscript
# In evolution_manager.gd
func apply_evolution(evolution: EvolutionData) -> void:
    player.current_form = evolution
    player.update_from_evolution(evolution)
    player.sprite_2d.texture = evolution.sprite
    
    EventBus.evolution_chosen.emit(evolution)
```

## Key Autoloads to Register

Register these in Project Settings > Autoload:

1. `EventBus` (res://autoloads/event_bus.gd) - Global signal hub
2. `GameManager` (res://autoloads/game_manager.gd) - Game state
3. `EvolutionManager` (res://autoloads/evolution_manager.gd) - Evolution tracking
4. `PauseManager` (res://autoloads/pause_manager.gd) - Pause control

## Common Mistakes to Avoid

- ❌ Usar variables sin tipo → ✅ Siempre usar `: Type`
- ❌ Llamadas directas entre escenas → ✅ Usar señales & EventBus
- ❌ No liberar nodos → ✅ Usar `queue_free()` o dejar que el padre lo maneje
- ❌ Actualizar UI directamente desde lógica → ✅ Emitir señales, UI escucha
- ❌ Almacenar datos del juego en escenas → ✅ Usar Resources
- ❌ Ignorar desconexiones en `_exit_tree()` → ✅ Siempre limpiar conexiones
- ❌ Procesar código en `_process()` durante pausa → ✅ Verificar `get_tree().paused`

## Consejos de Rendimiento

- Usar `@onready` para cachear nodos frecuentemente accedidos
- Emitir señales solo cuando el estado realmente cambia
- Usar `if not visible:` para saltar lógica de renderizado para objetos fuera de pantalla
- Perfilar regularmente con el Perfilador de Godot
- Usar `yield()` raramente; preferir señales

## Recordatorio de Estructura de Archivos

```
res://
├── autoloads/              # Managers globales
├── resources/              # Datos del juego (.tres files)
│   ├── evolutions/
│   ├── abilities/
│   ├── items/
│   └── enemies/
├── scenes/                 # Archivos de escena organizados por feature
│   ├── characters/player/
│   ├── characters/enemies/
│   ├── ui/
│   └── main.tscn
├── scripts/                # Scripts compartidos
│   ├── systems/
│   ├── components/
│   └── utils/
└── assets/                 # Sprites, audio, fuentes
```

## En Caso de Duda
- Consulta [Documentación Godot 4](https://docs.godotengine.org/)
- Mantén el código simple y legible
- Usa nombres descriptivos de variables
- Añade comentarios para lógica no obvia
- Prueba frecuentemente las transiciones de evolución durante el desarrollo
