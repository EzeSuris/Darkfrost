# Darkfrost - Instrucciones para Agentes AI de Codificación

## Descripción General del Proyecto

**Darkfrost** es un roguelike de acción en tiempo real 2D top-down construido con Godot 4.x usando GDScript. La mecánica central es la **evolución del personaje**: a medida que el jugador progresa y derrota enemigos, gana experiencia y sube de nivel. En cada nivel, puede elegir cómo evolucionar, obteniendo nuevas habilidades y aumentos de stats. El árbol de evolución es lineal (Forma A → Forma B → Forma C), permitiendo estilos de juego únicos y toma de decisiones estratégica.

- **Género**: Roguelike de acción en tiempo real con pausa táctica
- **Vista**: 2D top-down
- **Sistema de Tiempo**: Tiempo real con pausa (ESC para pausar y tomar decisiones tácticas)
- **Mecánica Central**: Sistema de evolución de personaje con árboles de habilidades ramificadas
- **Plataforma**: PC (Windows, Mac, Linux)
- **Motor**: Godot 4.3+

## Arquitectura del Juego

### Autoloads (Singletons)
Estos son sistemas globales que deben registrarse en Configuración del Proyecto > Autoload:

- **GameManager** (`autoloads/game_manager.gd`)
  - Seguimiento del nivel/piso actual
  - Gestión del estado del juego (en ejecución, pausado, fin del juego)
  - Configuración global y ajustes

- **EventBus** (`autoloads/event_bus.gd`)
  - Centro de señales para comunicación entre sistemas
  - Emite: player_leveled_up, evolution_available, player_died, enemy_died, etc.

- **EvolutionManager** (`autoloads/evolution_manager.gd`)
  - Rastrea XP, nivel y forma actual del jugador
  - Gestiona evoluciones disponibles en cada nivel
  - Maneja transiciones de forma y actualizaciones de habilidades
  - Emite eventos de evolución a través de EventBus

- **PauseManager** (`autoloads/pause_manager.gd`)
  - Maneja lógica de pausa/reanudación
  - Controla la escala de tiempo
  - Gestiona visibilidad del menú de pausa

### Sistemas Principales
- **AbilitySystem**: Maneja ejecución de habilidades, cooldowns y efectos
- **CombatSystem**: Cálculo de daño, knockback, detección de golpes
- **EvolutionSystem**: Gestiona transiciones de forma y cambio de habilidades
- **LevelGenerator**: Generación procedural de mazmorras

### Recursos Clave
- **EvolutionData** (`resources/evolutions/*.tres`): Define cada forma de evolución (sprite, habilidades, stats)
- **AbilityData** (`resources/abilities/*.tres`): Define habilidades individuales (daño, cooldown, efecto)
- **ItemData** (`resources/items/*.tres`): Drops y power-ups
- **EnemyData** (`resources/enemies/*.tres`): Stats y comportamiento de enemigos

### Arquitectura de Nodos
```
Main (Node2D)
├── World (Node2D)
│   ├── TileMap
│   ├── Entities (Node)
│   │   ├── Player
│   │   └── Enemies
│   ├── Items (Node)
│   └── FOVRenderer
├── UI (CanvasLayer)
│   ├── HUD
│   ├── EvolutionMenu
│   ├── PauseMenu
│   └── AbilityBar
└── SoundManager (AudioStreamPlayer)
```

## Convenciones de Codificación

### Estándares GDScript
- **Siempre usar tipado estático**: `var health: int = 100` no `var health = 100`
- **Tipear parámetros de funciones y valores de retorno**: 
  ```gdscript
  func take_damage(amount: int) -> void:
  func calculate_damage() -> int:
  ```
- **Usar retornos tempranos** para reducir anidamiento
- **PascalCase** para nombres de clases y nodos
- **snake_case** para archivos, variables, funciones y nombres de señales
- **SCREAMING_SNAKE_CASE** para constantes

### Directrices de Señales
- Usar **verbos en pasado** para señales: `health_changed`, `ability_used`, `evolution_unlocked`
- Emitir señales después de cambios de estado, no antes
- Conectar señales en `_ready()` usando `@onready` cuando sea posible
- Usar EventBus para comunicación entre sistemas

### Organización de Archivos
- Mantener escenas y sus scripts en la misma carpeta (ej: `scenes/player/player.tscn` + `scenes/player/player.gd`)
- Usar Resources para datos, Nodes para comportamiento
- Separar responsabilidades: los componentes manejan responsabilidades individuales
- Usar composición sobre herencia

## Patrones Clave

### Patrón del Sistema de Evolución
Cada forma (evolución) se define mediante un recurso `EvolutionData`:
```gdscript
# En un recurso de evolución (ej: form_1_basic.tres)
class_name EvolutionData
extends Resource

@export var form_name: String = "Forma Básica"
@export var sprite: Texture2D
@export var max_health: int = 100
@export var speed: float = 150.0
@export var abilities: Array[AbilityData]
@export var next_evolution: EvolutionData  # Para progresión lineal
```

Cuando el jugador sube de nivel y elige una evolución:
1. EvolutionManager emite señal `evolution_chosen`
2. El nodo del jugador actualiza sprite, stats y habilidades
3. Las habilidades antiguas se reemplazan con las nuevas
4. El HUD se actualiza para reflejar los nuevos iconos de habilidades

### Arquitectura Basada en Componentes
- **HealthComponent**: Gestiona HP, daño recibido, muerte
- **AbilityComponent**: Gestiona habilidades activas y cooldowns
- **HitboxComponent**: Maneja colisiones y detección de daño

Ejemplo:
```gdscript
# En player.gd
@onready var health: HealthComponent = $HealthComponent
@onready var abilities: AbilityComponent = $AbilityComponent

func _ready() -> void:
    health.health_changed.connect(_on_health_changed)
    abilities.ability_used.connect(_on_ability_used)
```

### Sistema de Pausa
El juego en tiempo real puede pausarse presionando ESC. El sistema de pausa:
- Establece `get_tree().paused = true` para congelar física y timers
- Muestra menú de pausa sobre el juego
- Permite interacción de UI mientras está pausado
- También puede usarse para la pantalla de selección de evolución

```gdscript
# En pause_manager.gd
func toggle_pause() -> void:
    is_paused = !is_paused
    get_tree().paused = is_paused
    pause_menu.visible = is_paused
    EventBus.pause_toggled.emit(is_paused)
```

## Ubicaciones Importantes

- **Escena Principal**: `res://scenes/main.tscn`
- **Escena del Jugador**: `res://scenes/characters/player/player.tscn`
- **Recursos de Evolución**: `res://resources/evolutions/`
- **Recursos de Habilidades**: `res://resources/abilities/`
- **Scripts de Sistemas**: `res://scripts/systems/`
- **Escenas de UI**: `res://scenes/ui/`

## Prioridades de Desarrollo

1. **Fundación** (Fase 1): Movimiento del jugador, colisión básica, cámara siguiendo
2. **Combate** (Fase 2): Ataque melee simple, IA de enemigos, sistema de daño
3. **Evolución** (Fase 3): Sistema de subida de nivel, menú de evolución, intercambio de habilidades
4. **Generación** (Fase 4): Generación procedural de mazmorras
5. **Contenido** (Fase 5): Variedad de enemigos, items, pulido visual
6. **Refinamiento** (Fase 6): Balance, efectos, diseño sonoro

## Compilar y Ejecutar

1. Abre `project.godot` en Godot 4.3+
2. Presiona F5 o haz clic en Reproducir para ejecutar `res://scenes/main.tscn`
3. ESC para pausar
4. Cierra con Alt+F4 o el botón de cerrar

## Comandos Comunes

- **Ejecutar escena principal**: F5
- **Ejecutar escena actual**: Shift+F5
- **Pausar ejecución**: F7
- **Abrir Sistema de Archivos**: Ctrl+Alt+F
- **Abrir Editor de Scripts**: Ctrl+Alt+E

## Pruebas y Depuración

- Usa `print()` para salida de depuración (visible en el panel de Salida)
- Usa `assert()` para validación en tiempo de ejecución
- Revisa el panel del Depurador para monitoreo de rendimiento
- Usa breakpoints en el Editor de Scripts (haz clic en el número de línea)

---

## Notas Finales

- **Calidad de Código**: Apunta a claridad sobre complejidad. Tu yo futuro apreciará código legible.
- **Control de Versión**: Haz commits frecuentes con mensajes claros.
- **Documentación**: Mantén este archivo actualizado a medida que la arquitectura evoluciona.
- **Pruebas**: Prueba manualmente las transiciones de evolución y ejecución de habilidades frecuentemente.

Para preguntas sobre patrones de Godot, consulta los docs oficiales en https://docs.godotengine.org/
