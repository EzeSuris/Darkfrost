# Darkfrost 🧊

Un roguelike 2D top-down en tiempo real donde tu personaje evoluciona y se transforma a medida que progresas.

## 🎮 Concepto Central

**Darkfrost** trata sobre evolución estratégica. Cada enemigo derrotado otorga experiencia. Cada subida de nivel presenta una elección: evoluciona a tu siguiente forma, desbloqueando nuevas habilidades y stats mejorados. Cuanto más profundo te aventures en los dungeons congelados, más poderoso—y diferente—te volverás.

### Características Principales
- 🎯 **Acción en Tiempo Real**: Combate rápido con pausa táctica (ESC)
- 🔄 **Evolución de Personaje**: Progresión lineal con cambios significativos de habilidades en cada forma
- ❄️ **Exploración Top-Down**: Dungeons generados proceduralmente para navegar
- 💪 **Poder Progresivo**: Hazte más fuerte y desbloquea nuevas estrategias a medida que evolucionas
- 💀 **Persistencia Roguelike**: Cada run es permanente, pero las mejoras se transfieren

## 📋 Estructura del Proyecto

```
Darkfrost/
├── .github/
│   └── copilot-instructions.md    # Directrices para agentes AI de codificación
├── autoloads/                     # Sistemas singleton globales
│   ├── game_manager.gd
│   ├── event_bus.gd
│   ├── evolution_manager.gd
│   └── pause_manager.gd
├── resources/                     # Datos del juego (Resources)
│   ├── abilities/                 # Definiciones de habilidades
│   ├── evolutions/                # Definiciones de formas de evolución
│   ├── enemies/                   # Datos de enemigos
│   ├── items/                     # Drops de items
│   └── player/                    # Stats base del jugador
├── scenes/                        # Escenas del juego
│   ├── characters/
│   │   ├── player/
│   │   │   ├── player.tscn
│   │   │   ├── player.gd
│   │   │   └── forms/             # Variantes visuales por evolución
│   │   └── enemies/
│   ├── levels/                    # Niveles del dungeon
│   ├── abilities/                 # Escenas de efectos de habilidades
│   ├── items/                     # Escenas de recolección de items
│   ├── ui/
│   │   ├── hud/                   # HUD del juego
│   │   ├── evolution_menu/        # Selección de evolución en subida de nivel
│   │   ├── pause_menu/            # Menú de pausa
│   │   └── ability_bar/           # Visualización de habilidades
│   └── main.tscn                  # Punto de entrada
├── scripts/
│   ├── systems/                   # Sistemas principales del juego
│   │   ├── ability_system.gd
│   │   ├── evolution_system.gd
│   │   ├── combat_system.gd
│   │   └── level_generator.gd
│   ├── components/                # Componentes de nodos reutilizables
│   │   ├── health_component.gd
│   │   ├── hitbox_component.gd
│   │   └── ability_component.gd
│   └── utils/                     # Utilidades auxiliares
├── assets/                        # Arte, audio, fuentes
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   ├── fonts/
│   ├── sprites/
│   │   ├── characters/forms/      # Sprites de formas del jugador
│   │   ├── enemies/
│   │   ├── items/
│   │   ├── tiles/
│   │   ├── abilities/             # VFX de habilidades
│   │   └── ui/
│   └── shaders/
├── docs/
│   ├── ROADMAP_ROGUELIKE.md       # Hoja de ruta de desarrollo
│   └── .gdignore                  # Excluir docs de la exportación
├── AGENTS.md                      # Instrucciones para agentes AI
└── project.godot
```

## 🚀 Inicio Rápido

### Requisitos
- **Godot 4.3** o posterior
- Windows, macOS, o Linux

### Instalación

1. Clona el repositorio:
   ```bash
   git clone https://github.com/EzeSuris/Roguelike.git
   cd Roguelike/nuevo-proyecto-de-juego
   ```

2. Abre en Godot:
   - Lanza Godot 4.3+
   - Haz clic en "Abrir Proyecto"
   - Navega a `project.godot`

3. Ejecuta el juego:
   - Presiona **F5** para jugar
   - O haz clic en el botón Play

### Controles

| Entrada | Acción |
|---------|--------|
| **Teclas de Flecha / WASD** | Mover personaje |
| **LMB / Barra espaciadora** | Usar habilidad primaria |
| **RMB** | Usar habilidad secundaria |
| **E** | Interactuar / Recoger items |
| **ESC** | Pausar juego |
| **Tab** | Alternar inventario |

## 🧬 Sistema de Evolución

Tu personaje evoluciona linealmente a través de formas, cada una desbloqueando nuevas habilidades y estilos de juego:

- **Forma 1 - Criomante**: Magia de hielo básica, proyectiles lentos
- **Forma 2 - Nacido del Hielo**: Velocidad mejorada, ataques AoE de congelación
- **Forma 3 - Ventisca Eterna**: Forma final, habilidades de combinación devastadoras

En cada subida de nivel, eliges evolucionar (o saltarte para enfocarte en otros stats). Solo puedes progr esar hacia adelante en la cadena de evolución.

## 🎨 Prioridades de Desarrollo

El proyecto se construye en fases:

### Fase 1: Fundación *(Actual)*
- Movimiento del jugador & cámara
- Detección de colisiones básica
- Estructura de escenas

### Fase 2: Combate
- Stats del jugador & enemigos
- Ataques melee/ranged
- Sistema de daño

### Fase 3: Evolución
- Sistema de subida de nivel
- UI del menú de evolución
- Intercambio de habilidades

### Fase 4: Generación Procedural
- Algoritmos de generación de dungeons
- Generación de habitaciones/pasillos
- Spawn de enemigos

### Fase 5: Contenido
- Variedad de enemigos
- Tipos de items & efectos
- Encuentros con jefes

### Fase 6: Pulido
- Efectos visuales & animaciones
- Diseño de sonido
- Balance & ajustes

Ver [docs/ROADMAP_ROGUELIKE.md](docs/ROADMAP_ROGUELIKE.md) para hitos detallados.

## 🛠️ Descripción General de la Arquitectura

### Autoloads (Singletons Globales)

- **EventBus**: Centro de señales para todos los sistemas
- **GameManager**: Estado del juego & configuración
- **EvolutionManager**: Rastrea XP, nivel y evolución actual del jugador
- **PauseManager**: Maneja pausa/reanudación con gestión de UI

### Sistemas Principales

- **AbilitySystem**: Ejecuta habilidades, gestiona cooldowns
- **CombatSystem**: Calcula daño, aplica efectos
- **EvolutionSystem**: Maneja transiciones de forma y actualizaciones de habilidades
- **LevelGenerator**: Generación procedural de dungeons

### Patrón de Componentes

- **HealthComponent**: Salud, daño, muerte
- **AbilityComponent**: Habilidades activas y cooldowns
- **HitboxComponent**: Detección de colisiones y daño

## 📖 Estándares de Codificación

Este proyecto usa **GDScript con tipado estático**. Ver [AGENTS.md](AGENTS.md) y [.github/copilot-instructions.md](.github/copilot-instructions.md) para directrices detalladas.

### Referencia Rápida
- **Archivos**: `snake_case` (ej: `player_character.gd`)
- **Clases**: `PascalCase` (ej: `class_name PlayerCharacter`)
- **Funciones**: `snake_case` con type hints (ej: `func take_damage(amount: int) -> void`)
- **Constantes**: `SCREAMING_SNAKE_CASE` (ej: `const MAX_SPEED: float = 200.0`)
- **Señales**: verbos en pasado (ej: `signal health_changed`)

## 🐛 Depuración

### Ejecutar con Salida de Depuración
Presiona F5 para ejecutar. La salida es visible en el panel "Salida" en la parte inferior del editor.

### Breakpoints
- Haz clic en un número de línea en el script para establecer un breakpoint
- La ejecución se pausa cuando se alcanza esa línea
- Inspecciona variables en el panel "Variables"

### Problemas Comunes
- **Autoloads no encontrados**: Verifica Configuración del Proyecto > Autoload para registrarlos
- **Señal no conectada**: Asegúrate de que `_ready()` se está llamando
- **Nodos desapareciendo**: Verifica que los nodos no se estén liberando prematuramente

## 🤝 Contribuyendo

Este es un proyecto de aprendizaje en solitario, ¡pero siéntete libre de hacer fork y crear tu propia versión!

### Directrices
- Sigue los estándares de codificación en AGENTS.md
- Mantén commits atómicos y bien mensajeados
- Prueba transiciones de evolución frecuentemente
- Documenta cualquier sistema nuevo

## 📝 Licencia

Licencia MIT - Ver archivo LICENSE para detalles

## 🎯 Ideas Futuras

- [ ] Meta-progresión persistente (los desbloqueos se transfieren entre runs)
- [ ] Más árboles de evolución (decisiones ramificadas)
- [ ] Combinaciones de hechizos (mezclar habilidades para efectos únicos)
- [ ] Tablas de clasificación / estadísticas de run
- [ ] Soporte de mods para evoluciones personalizadas
- [ ] Multijugador cooperativo

## 🔗 Recursos

- [Documentación Godot 4](https://docs.godotengine.org/)
- [Conceptos Básicos de GDScript](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/index.html)
- [Desarrollo de Roguelike](https://www.roguebasin.com/)
- [Patrones de Diseño de Juegos](https://gameprogrammingpatterns.com/)

---

**Estado**: Desarrollo Temprano 🚧

Última Actualización: 7 de enero de 2026
