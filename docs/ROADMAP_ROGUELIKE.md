# 🗺️ Hoja de Ruta de Desarrollo de Darkfrost

Guía completa paso a paso para construir un roguelike desde cero, con enfoque en **mecánicas de evolución de Darkfrost**.

## Tabla de Contenidos
1. [Fase 1: Fundación](#fase-1-fundación)
2. [Fase 2: Combate](#fase-2-combate)
3. [Fase 3: Sistema de Evolución](#fase-3-sistema-de-evolución)
4. [Fase 4: Generación Procedural](#fase-4-generación-procedural)
5. [Fase 5: Contenido y Variedad](#fase-5-contenido-y-variedad)
6. [Fase 6: Pulido y Balance](#fase-6-pulido-y-balance)

---

## Fase 1: Fundación

**Objetivo**: Conseguir un personaje jugable que se mueva alrededor de la pantalla con seguimiento básico de cámara.

### 1.1 Configuración del Proyecto y Autoloads

- [ ] **Registra los Autoloads** en Configuración del Proyecto > Autoload:
  - `EventBus` (res://autoloads/event_bus.gd)
  - `GameManager` (res://autoloads/game_manager.gd)
  - `PauseManager` (res://autoloads/pause_manager.gd)
  - `EvolutionManager` (res://autoloads/evolution_manager.gd)

- [ ] **Crea EventBus** (`autoloads/event_bus.gd`):
  ```gdscript
  extends Node
  
  # Eventos del jugador
  signal player_moved(position: Vector2)
  signal player_leveled_up(new_level: int)
  signal player_died
  
  # Eventos de combate
  signal damage_dealt(amount: int, target: Node)
  signal health_changed(entity: Node, new_health: int)
  
  # Eventos de evolución
  signal evolution_available(evolutions: Array[EvolutionData])
  signal evolution_chosen(evolution: EvolutionData)
  
  # Flujo del juego
  signal enemy_died(enemy: Node)
  signal level_completed
  signal pause_toggled(is_paused: bool)
  ```

- [ ] **Crea GameManager** (`autoloads/game_manager.gd`):
  ```gdscript
  extends Node
  
  var current_level: int = 1
  var player_stats: Dictionary = {}
  var rng: RandomNumberGenerator
  
  func _ready() -> void:
      rng = RandomNumberGenerator.new()
      rng.randomize()
  ```

- [ ] **Crea PauseManager** (`autoloads/pause_manager.gd`):
  ```gdscript
  extends Node
  
  var is_paused: bool = false
  
  func _ready() -> void:
      set_process_input(true)
  
  func _input(event: InputEvent) -> void:
      if event.is_action_pressed("ui_cancel"):  # Tecla ESC
          toggle_pause()
          get_tree().root.set_input_as_handled()
  
  func toggle_pause() -> void:
      is_paused = !is_paused
      get_tree().paused = is_paused
      EventBus.pause_toggled.emit(is_paused)
  ```

- [ ] **Crea EvolutionManager** (`autoloads/evolution_manager.gd`):
  ```gdscript
  extends Node
  
  var player_xp: int = 0
  var player_level: int = 1
  var current_evolution: EvolutionData
  
  func add_xp(amount: int) -> void:
      player_xp += amount
      # TODO: Verifica si subió de nivel (ej: 100 XP por nivel)
  ```

### 1.2 Crea la Escena Principal

- [ ] **Crea main.tscn** en `scenes/`:
  - Raíz: `Main` (Node2D)
  - Hijos:
    - `World` (Node2D) - contiene toda la jugabilidad
    - `UI` (CanvasLayer) - contiene toda la UI
    - `SoundManager` (Node) - gestión de audio

- [ ] **main.gd básico**:
  ```gdscript
  extends Node2D
  
  func _ready() -> void:
      print("Juego iniciado en nivel ", GameManager.current_level)
  ```

### 1.3 Crea Movimiento del Jugador

- [ ] **Crea la escena del jugador** (`scenes/characters/player/player.tscn`):
  - Raíz: `Player` (CharacterBody2D)
  - Hijos:
    - `Sprite2D` (Sprite2D) - muestra el sprite del personaje
    - `CollisionShape2D` (CollisionShape2D) - cuerpo de física
    - `Camera2D` (Camera2D) - sigue al jugador
    - `HealthComponent` (Node) - gestión de salud (TBD)

- [ ] **Crea player.gd**:
  ```gdscript
  class_name PlayerCharacter
  extends CharacterBody2D
  
  @export var speed: float = 150.0
  
  @onready var sprite: Sprite2D = $Sprite2D
  @onready var camera: Camera2D = $Camera2D
  
  func _ready() -> void:
      camera.make_current()
  
  func _physics_process(delta: float) -> void:
      var input_dir := Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
      velocity = input_dir * speed
      move_and_slide()
      
      if velocity.length() > 0:
          EventBus.player_moved.emit(global_position)
  ```

- [ ] **Añade un sprite placeholder**: 
  - Crea un PNG de 32x32 simple o usa un rectángulo coloreado como placeholder
  - Asigna al nodo `Sprite2D`

### 1.4 Crea Mundo y Tilemap

- [ ] **Crea un tilemap** (`scenes/levels/level_1.tscn`):
  - Raíz: `Level` (Node2D)
  - Hijo: `TileMap` - para el piso/paredes del dungeon
  - Usa el tileset integrado de Godot o crea cuadrados de colores simples

- [ ] **Instancia el jugador en el nivel**:
  - Añade la escena del Jugador como hijo del Level
  - Posiciona en la ubicación de inicio

- [ ] **Prueba Movimiento Básico**:
  - Presiona F5 para ejecutar
  - El jugador debería moverse con teclas de flecha/WASD
  - La cámara debería seguir
  - Aún sin colisiones (está bien por ahora)

### 1.5 Añade UI Placeholder

- [ ] **Crea HUD básico** (`scenes/ui/hud/hud.tscn`):
  - Raíz: `HUD` (Control)
  - Hijos:
    - `LevelLabel` (Label) - muestra el nivel actual
    - `HealthBar` (ProgressBar) - muestra salud del jugador
    - `XPBar` (ProgressBar) - muestra progreso al siguiente nivel
    - `AbilityBar` (HBoxContainer) - mostrará habilidades más tarde

- [ ] **Crea hud.gd**:
  ```gdscript
  extends Control
  
  @onready var level_label: Label = $LevelLabel
  @onready var health_bar: ProgressBar = $HealthBar
  @onready var xp_bar: ProgressBar = $XPBar
  
  func _ready() -> void:
      EventBus.player_leveled_up.connect(_on_player_leveled_up)
      EventBus.health_changed.connect(_on_health_changed)
      
      level_label.text = "Nivel: %d" % EvolutionManager.player_level
  
  func _on_player_leveled_up(new_level: int) -> void:
      level_label.text = "Nivel: %d" % new_level
  
  func _on_health_changed(entity: Node, new_health: int) -> void:
      if entity.is_in_group("player"):
          health_bar.value = new_health
  ```

### Checkpoint 1
✅ **Al final de la Fase 1, deberías tener:**
- Personaje jugable que se mueve
- La cámara sigue al jugador
- HUD básico con nivel y visualización de salud
- Sistema de pausa (ESC para alternar pausa)
- Todos los autoloads registrados

---

## Fase 2: Combate

**Objetivo**: Implementar daño, ataques básicos e IA de enemigos.

### 2.1 Create HealthComponent

- [ ] **Create HealthComponent** (`scripts/components/health_component.gd`):
  ```gdscript
  class_name HealthComponent
  extends Node
  
  signal health_changed(new_health: int)
  signal died
  
  @export var max_health: int = 100
  var current_health: int
  
  func _ready() -> void:
      current_health = max_health
  
  func take_damage(amount: int) -> void:
      current_health = max(0, current_health - amount)
      health_changed.emit(current_health)
      if current_health == 0:
          died.emit()
  
  func heal(amount: int) -> void:
      current_health = min(max_health, current_health + amount)
      health_changed.emit(current_health)
  ```

- [ ] **Update player.tscn**:
  - Add `HealthComponent` as child of Player node
  - Set `max_health` to 100 in Inspector

- [ ] **Update player.gd**:
  ```gdscript
  @onready var health: HealthComponent = $HealthComponent
  
  func _ready() -> void:
      camera.make_current()
      health.died.connect(_on_died)
  
  func _on_died() -> void:
      print("Player died!")
      EventBus.player_died.emit()
      queue_free()
  ```

### 2.2 Create HitboxComponent

- [ ] **Create HitboxComponent** (`scripts/components/hitbox_component.gd`):
  ```gdscript
  class_name HitboxComponent
  extends Area2D
  
  signal hit_detected(other_area: Area2D)
  
  @export var damage: int = 10
  
  func _ready() -> void:
      area_entered.connect(_on_area_entered)
  
  func _on_area_entered(other_area: Area2D) -> void:
      if other_area.is_in_group("hurtbox"):
          hit_detected.emit(other_area)
  ```

- [ ] **Create HurtboxComponent** (`scripts/components/hurtbox_component.gd`):
  ```gdscript
  class_name HurtboxComponent
  extends Area2D
  
  signal damaged(amount: int)
  
  @onready var parent: Node = get_parent()
  
  func take_damage(amount: int) -> void:
      if parent.has_node("HealthComponent"):
          parent.get_node("HealthComponent").take_damage(amount)
      damaged.emit(amount)
  
  func _ready() -> void:
      add_to_group("hurtbox")
  ```

### 2.3 Create Basic Enemy

- [ ] **Create Enemy scene** (`scenes/characters/enemies/basic_enemy.tscn`):
  - Root: `Enemy` (CharacterBody2D)
  - Children:
    - `Sprite2D` (Sprite2D)
    - `CollisionShape2D` (CollisionShape2D)
    - `HealthComponent` (Node) - set max_health to 30
    - `HurtboxComponent` (Area2D)
    - `AttackHitbox` (Area2D) - for enemy attacks

- [ ] **Create enemy.gd**:
  ```gdscript
  class_name BasicEnemy
  extends CharacterPosition2D
  
  @export var speed: float = 80.0
  @export var xp_reward: int = 10
  
  @onready var sprite: Sprite2D = $Sprite2D
  @onready var health: HealthComponent = $HealthComponent
  var player: PlayerCharacter
  
  func _ready() -> void:
      player = get_tree().get_first_child_in_group("player")
      health.died.connect(_on_died)
  
  func _physics_process(delta: float) -> void:
      if not player:
          return
      
      # Chase player
      var direction := (player.global_position - global_position).normalized()
      velocity = direction * speed
      move_and_slide()
  
  func _on_died() -> void:
      EvolutionManager.add_xp(xp_reward)
      EventBus.enemy_died.emit(self)
      queue_free()
  ```

### 2.4 Create Basic Attack

- [ ] **Create AbilityComponent** (`scripts/components/ability_component.gd`):
  ```gdscript
  class_name AbilityComponent
  extends Node
  
  signal ability_used(ability: AbilityData)
  
  @export var abilities: Array[AbilityData] = []
  var ability_cooldowns: Dictionary = {}
  
  func _ready() -> void:
      for ability in abilities:
          ability_cooldowns[ability] = 0.0
  
  func _process(delta: float) -> void:
      for ability in abilities:
          ability_cooldowns[ability] = max(0.0, ability_cooldowns[ability] - delta)
  
  func use_ability(ability_index: int) -> bool:
      if ability_index >= abilities.size():
          return false
      
      var ability := abilities[ability_index]
      if ability_cooldowns[ability] > 0:
          return false
      
      ability_cooldowns[ability] = ability.cooldown
      ability_used.emit(ability)
      return true
  ```

- [ ] **Create AbilityData Resource** (`resources/abilities/ability_data.gd`):
  ```gdscript
  class_name AbilityData
  extends Resource
  
  @export var ability_name: String = "Ability"
  @export var damage: int = 15
  @export var cooldown: float = 1.0
  @export var icon: Texture2D
  ```

- [ ] **Update player.gd** to include attacks:
  ```gdscript
  @onready var abilities: AbilityComponent = $AbilityComponent
  
  func _process(_delta: float) -> void:
      if Input.is_action_just_pressed("ui_accept"):  # Spacebar
          abilities.use_ability(0)
  ```

### 2.5 Test Combat

- [ ] **Create test level** with a few enemies
- [ ] **Run the game**: F5
- [ ] Player should take damage when touched by enemy
- [ ] Player should deal damage when pressing spacebar (or implement proper attack)
- [ ] Enemies should die when health reaches 0

### Checkpoint 2
✅ **Al final de la Fase 2, deberías tener:**
- Sistema de salud/daño funcionando
- Enemigos básicos que persiguen y dañan al jugador
- El jugador puede causar daño
- Recompensas de XP al matar enemigos
- HUD se actualiza con cambios de salud

---

## Fase 3: Sistema de Evolución

**Objetivo**: Implementar la mecánica central—subidas de nivel con selección de evolución.

### 3.1 Create Evolution Data Structure

- [ ] **Create EvolutionData Resource** (`resources/evolutions/evolution_data.gd`):
  ```gdscript
  class_name EvolutionData
  extends Resource
  
  @export var form_name: String = "Basic Form"
  @export var sprite: Texture2D
  @export var description: String = ""
  
  # Stats
  @export var base_max_health: int = 100
  @export var base_speed: float = 150.0
  @export var base_damage: int = 10
  
  # Abilities that this form has
  @export var abilities: Array[AbilityData] = []
  
  # Next evolution in the chain (null if final form)
  @export var next_evolution: EvolutionData
  ```

- [ ] **Create evolution resources**:
  - `resources/evolutions/form_1_cryomancer.tres`
  - `resources/evolutions/form_2_frostborn.tres`
  - `resources/evolutions/form_3_eternal_blizzard.tres`

### 3.2 Update EvolutionManager

- [ ] **Enhance EvolutionManager** (`autoloads/evolution_manager.gd`):
  ```gdscript
  class_name EvolutionManager
  extends Node
  
  const XP_PER_LEVEL: int = 100
  
  var player_xp: int = 0
  var player_level: int = 1
  var current_evolution: EvolutionData
  
  var available_evolution: EvolutionData
  
  func _ready() -> void:
      # Load starting evolution
      current_evolution = load("res://resources/evolutions/form_1_cryomancer.tres")
  
  func add_xp(amount: int) -> void:
      player_xp += amount
      
      # Check for level-up
      while player_xp >= XP_PER_LEVEL:
          player_xp -= XP_PER_LEVEL
          level_up()
  
  func level_up() -> void:
      player_level += 1
      EventBus.player_leveled_up.emit(player_level)
      
      # Check if current form has a next evolution
      if current_evolution.next_evolution:
          available_evolution = current_evolution.next_evolution
          EventBus.evolution_available.emit([available_evolution])
  
  func apply_evolution(evolution: EvolutionData) -> void:
      current_evolution = evolution
      EventBus.evolution_chosen.emit(evolution)
  ```

### 3.3 Create Evolution Menu UI

- [ ] **Create EvolutionMenu** (`scenes/ui/evolution_menu/evolution_menu.tscn`):
  - Root: `EvolutionMenu` (Control)
  - Children:
    - `Background` (ColorRect) - semi-transparent overlay
    - `Panel` (PanelContainer) - center panel
      - `VBoxContainer`:
        - `TitleLabel` (Label) - "Choose Your Evolution"
        - `EvolutionButton` (Button) - for the available evolution
        - `SkipButton` (Button) - option to skip

- [ ] **Create evolution_menu.gd**:
  ```gdscript
  extends Control
  
  @onready var evolution_button: Button = $Panel/VBoxContainer/EvolutionButton
  @onready var skip_button: Button = $Panel/VBoxContainer/SkipButton
  
  var available_evolutions: Array[EvolutionData] = []
  
  func _ready() -> void:
      visible = false
      EventBus.evolution_available.connect(_on_evolution_available)
      
      evolution_button.pressed.connect(_on_evolution_chosen)
      skip_button.pressed.connect(_on_skip)
  
  func _on_evolution_available(evolutions: Array[EvolutionData]) -> void:
      available_evolutions = evolutions
      
      if evolutions.size() > 0:
          evolution_button.text = "Evolve to %s" % evolutions[0].form_name
      
      visible = true
      PauseManager.toggle_pause()  # Pause the game
  
  func _on_evolution_chosen() -> void:
      if available_evolutions.size() > 0:
          EvolutionManager.apply_evolution(available_evolutions[0])
          _close_menu()
  
  func _on_skip() -> void:
      _close_menu()
  
  func _close_menu() -> void:
      visible = false
      available_evolutions.clear()
      PauseManager.toggle_pause()  # Unpause
  ```

### 3.4 Update Player to Respond to Evolution

- [ ] **Update player.gd**:
  ```gdscript
  func _ready() -> void:
      camera.make_current()
      health.died.connect(_on_died)
      EventBus.evolution_chosen.connect(_on_evolution_chosen)
      add_to_group("player")
  
  func _on_evolution_chosen(evolution: EvolutionData) -> void:
      # Update sprite
      sprite.texture = evolution.sprite
      
      # Update stats
      health.max_health = evolution.base_max_health
      health.heal(evolution.base_max_health)
      speed = evolution.base_speed
      
      # Update abilities
      abilities.abilities = evolution.abilities
      abilities._ready()  # Reset cooldowns
      
      print("Evolved to %s!" % evolution.form_name)
  ```

### 3.5 Test Evolution System

- [ ] **Run the game** and kill 10 enemies to level up
- [ ] Evolution menu should appear
- [ ] Player sprite should change on evolution
- [ ] Stats should update

### Checkpoint 3
✅ **Al final de la Fase 3, deberías tener:**
- Sistema de subida de nivel funcionando
- Menú de selección de evolución
- Sprite y stats del jugador cambian con la evolución
- La barra de XP muestra el progreso
- ¡La mecánica central es funcional!

---

## Fase 4: Generación Procedural

**Objetivo**: Generar dungeons aleatorios en lugar de niveles diseñados a mano.

### 4.1 Create Level Generator

- [ ] **Create LevelGenerator** (`scripts/systems/level_generator.gd`):
  ```gdscript
  class_name LevelGenerator
  extends Node
  
  @export var dungeon_width: int = 100
  @export var dungeon_height: int = 100
  @export var room_max_size: int = 15
  @export var room_min_size: int = 6
  @export var max_rooms: int = 15
  
  var rooms: Array[Rect2i] = []
  var tilemap: TileMap
  
  func _ready() -> void:
      tilemap = get_parent().get_node("TileMap")
  
  func generate_dungeon() -> void:
      rooms.clear()
      create_rooms()
      create_corridors()
      apply_to_tilemap()
  
  func create_rooms() -> void:
      for _i in range(max_rooms):
          var room_width: int = GameManager.rng.randi_range(room_min_size, room_max_size)
          var room_height: int = GameManager.rng.randi_range(room_min_size, room_max_size)
          var x: int = GameManager.rng.randi_range(0, dungeon_width - room_width)
          var y: int = GameManager.rng.randi_range(0, dungeon_height - room_height)
          
          var new_room := Rect2i(x, y, room_width, room_height)
          
          var overlaps: bool = false
          for room in rooms:
              if room.intersects(new_room):
                  overlaps = true
                  break
          
          if not overlaps:
              rooms.append(new_room)
  
  func create_corridors() -> void:
      # TODO: Create corridors between rooms
      pass
  
  func apply_to_tilemap() -> void:
      # TODO: Draw rooms and corridors on tilemap
      pass
  ```

### 4.2 Create Procedural Level Scene

- [ ] **Create procedural level** (`scenes/levels/procedural_level.tscn`):
  - Root: `Level` (Node2D)
  - Children:
    - `TileMap` (TileMap)
    - `LevelGenerator` (Node) - attach script
    - `EnemySpawner` (Node)
    - `Player` (PlayerCharacter instance)

### 4.3 Create Enemy Spawner

- [ ] **Create EnemySpawner** (`scripts/systems/enemy_spawner.gd`):
  ```gdscript
  extends Node
  
  @export var enemy_scene: PackedScene
  @export var initial_enemy_count: int = 5
  @export var max_enemies: int = 20
  
  func _ready() -> void:
      for _i in range(initial_enemy_count):
          spawn_random_enemy()
  
  func spawn_random_enemy() -> void:
      if get_child_count() >= max_enemies:
          return
      
      var enemy: Node = enemy_scene.instantiate()
      var random_pos := Vector2(
          GameManager.rng.randi_range(0, 800),
          GameManager.rng.randi_range(0, 600)
      )
      enemy.global_position = random_pos
      add_child(enemy)
  ```

### Checkpoint 4
✅ **Al final de la Fase 4, deberías tener:**
- Algoritmo de generación de dungeons funcionando
- Habitaciones aleatorias creadas en cada nivel
- Enemigos spawn en ubicaciones aleatorias
- Puedes progresar a través de múltiples niveles

---

## Fase 5: Contenido y Variedad

**Objetivo**: Añadir diferentes tipos de enemigos, items y más habilidades.

### 5.1 Create More Enemy Types

- [ ] Create `FastEnemy` - moves quickly, low health
- [ ] Create `StrongEnemy` - slower, high health and damage
- [ ] Create `RangedEnemy` - stays at distance, shoots projectiles

### 5.2 Create Item System

- [ ] **Create ItemData Resource** (`resources/items/item_data.gd`)
- [ ] **Create item pickup scene** (`scenes/items/item_pickup.tscn`)
- [ ] **Create item effects**:
  - Health potion (restore 30 HP)
  - Damage boost (increase next attack damage)
  - Speed boost (temporarily increase movement speed)

### 5.3 Create More Abilities

- [ ] Create `AbilityData` for different ability types:
  - Ice Bolt (projectile)
  - Freeze Aura (AoE)
  - Dash (movement ability)

### 5.4 Create Boss Encounters

- [ ] Create boss enemy with unique behavior
- [ ] Boss drops special reward

### Checkpoint 5
✅ **Al final de la Fase 5, deberías tener:**
- 3+ tipos de enemigos
- Recolección de items y caídas
- 5+ habilidades únicas
- Encuentro con jefe en el piso final

---

## Fase 6: Pulido y Balance

**Objetivo**: Hacer que el juego se sienta bien y esté balanceado.

### 6.1 Visual Polish

- [ ] Add particle effects for:
  - Attacks
  - Evolution transformation
  - Enemy death
  - Item collection

- [ ] Add animations:
  - Player walking
  - Enemy death
  - Evolution transition

### 6.2 Audio

- [ ] Add background music
- [ ] Add sound effects for:
  - Attacks
  - Level-up
  - Evolution
  - Enemy death
  - Item pickup

### 6.3 Balancing

- [ ] Tune enemy difficulty progression
- [ ] Balance ability cooldowns and damage
- [ ] Adjust XP requirements per level
- [ ] Test and playtest extensively

### 6.4 User Experience

- [ ] Add tutorial/how-to-play screen
- [ ] Add game over screen with stats
- [ ] Add settings menu (volume, difficulty)
- [ ] Optimize performance

### 6.5 Final Polish

- [ ] Create main menu
- [ ] Add pause menu options
- [ ] Add quality-of-life features
- [ ] Final balancing pass

### Checkpoint 6
✅ **¡Proyecto Completado!**
- Roguelike completamente jugable con mecánicas de evolución
- Pulido y sensación profesional
- Listo para compartir

---

## Consejos Adicionales

### Probando tu Sistema de Evolución
1. Añade comando de consola para saltar al siguiente nivel: `EvolutionManager.level_up()`
2. Crea datos de prueba con tiempos rápidos de evolución
3. Prueba todas las tres formas de evolución a fondo

### Manteniéndote Organizado
- Haz commits frecuentes (después de cada fase)
- Usa mensajes de commit significativos
- Mantén la documentación actualizada
- Prueba después de cada característica

### Optimización de Rendimiento
- Perfila regularmente con el Perfilador de Godot
- Usa `@onready` para cachear referencias de nodos frecuentemente accedidos
- Evita `queue_free()` en loops
- Pool de proyectiles/efectos

### Principios de Diseño de Roguelike
- **Decisiones Significativas**: La selección de evolución debe sentirse impactante
- **Riesgo/Recompensa**: Dificultad más alta debería dar mejores recompensas
- **Progresión**: Los jugadores deberían sentirse más fuertes con el tiempo
- **Rejugabilidad**: La aleatorización mantiene cada run fresca

---

## Recursos

- [Documentación Godot 4](https://docs.godotengine.org/)
- [Desarrollo de Roguelike](https://www.roguebasin.com/)
- [Patrones de Diseño de Juegos](https://gameprogrammingpatterns.com/)
- [Generación de Dungeons](https://www.gamedeveloper.com/design/rooms-and-mazes-the-dungeon-generators-of-yore)

---

**¡Feliz desarrollo! 🎮❄️**

Última Actualización: 7 de enero de 2026
