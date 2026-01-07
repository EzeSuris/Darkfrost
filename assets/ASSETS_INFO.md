# Assets de Darkfrost

Documentación de los assets utilizados en el proyecto.

## 📦 Assets Incluidos

### 1. Tiles de Dungeon
**Ubicación**: `assets/sprites/tiles/`

**Fuente**: Free 2D Top-Down Pixel Dungeon Asset Pack (Craftpix)

**Archivos incluidos**:
- `walls_floor.png` - Tiles de paredes y pisos del dungeon
- `decorative_cracks_floor.png` - Grietas decorativas para el piso
- `decorative_cracks_walls.png` - Grietas decorativas para las paredes
- `decorative_cracks_coasts_animation.png` - Animaciones de grietas en costas
- `doors_lever_chest_animation.png` - Puertas, palancas y cofres animados
- `Objects.png` - Objetos varios del dungeon
- `fire_animation.png` - Animación de fuego
- `fire_animation2.png` - Animación de fuego alternativa
- `trap_animation.png` - Animación de trampas
- `Water_coasts_animation.png` - Animación de agua y costas
- `water_detilazation_v2.png` - Detalles de agua

**Uso recomendado**:
- Crear TileMap para los niveles del dungeon
- Los archivos `_animation` se pueden usar con AnimatedSprite2D
- `walls_floor.png` es el principal para construir habitaciones

---

### 2. Sprites del Jugador (Soldado)
**Ubicación**: `assets/sprites/characters/forms/`

**Fuente**: Tiny RPG Character Asset Pack v1.03 - Soldier (100x100)

**Archivos incluidos**:
- `Soldier.png` - Spritesheet completo
- `Soldier-Idle.png` - Animación idle
- `Soldier-Walk.png` - Animación de caminar
- `Soldier-Attack01.png` - Animación de ataque 1
- `Soldier-Attack02.png` - Animación de ataque 2
- `Soldier-Attack03.png` - Animación de ataque 3
- `Soldier-Hurt.png` - Animación de daño recibido
- `Soldier-Death.png` - Animación de muerte
- `Shadow sprites/` - Sprites con sombras incluidas

**Uso recomendado**:
- Usar como **Forma 1** del sistema de evolución (Criomante básico)
- Los spritesheets tienen múltiples frames que puedes separar en AnimatedSprite2D
- Tamaño: 100x100 píxeles por frame

**Ejemplo de uso en AnimatedSprite2D**:
```gdscript
# En el nodo AnimatedSprite2D del jugador
# Crear animaciones:
# - "idle" usando Soldier-Idle.png
# - "walk" usando Soldier-Walk.png
# - "attack" usando Soldier-Attack01.png, etc.
```

---

### 3. Sprites de Enemigos (Orco)
**Ubicación**: `assets/sprites/enemies/`

**Fuente**: Tiny RPG Character Asset Pack v1.03 - Orc (100x100)

**Archivos incluidos**:
- `Orc.png` - Spritesheet completo
- `Orc-Idle.png` - Animación idle
- `Orc-Walk.png` - Animación de caminar
- `Orc-Attack01.png` - Animación de ataque 1
- `Orc-Attack02.png` - Animación de ataque 2
- `Orc-Hurt.png` - Animación de daño recibido
- `Orc-Death.png` - Animación de muerte
- `Shadow sprites/` - Sprites con sombras incluidas

**Uso recomendado**:
- Usar como **BasicEnemy** en la Fase 2 del roadmap
- Puedes crear variantes (Orco rápido, Orco fuerte) usando el mismo spritesheet con diferentes colores
- Tamaño: 100x100 píxeles por frame

---

### 4. Sprites de Proyectiles
**Ubicación**: `assets/sprites/abilities/`

**Fuente**: Tiny RPG Character Asset Pack v1.03 - Arrow Projectile

**Uso recomendado**:
- Para habilidades de proyectiles
- Puede ser usado como base para efectos de hielo (cambiar color a azul/cyan)
- Se puede rotar según la dirección del disparo

---

## 🎨 Notas de Arte

### Paleta de Colores Sugerida para Darkfrost
Ya que el tema es "hielo y nieve", considera añadir shaders o efectos de color:
- **Azul hielo**: #5BC0EB
- **Cyan oscuro**: #3A506B
- **Blanco nieve**: #F0F3FF
- **Azul profundo**: #1C2541

### Sprites Faltantes (para crear o buscar)
- [ ] Formas 2 y 3 del personaje (Frostborn, Eternal Blizzard)
- [ ] Efectos de habilidades de hielo (congelación, ráfaga, etc.)
- [ ] Más tipos de enemigos (rápidos, tanques, a distancia)
- [ ] Items (pociones, power-ups, etc.)
- [ ] Elementos de UI (iconos de habilidades, marcos, etc.)
- [ ] Efectos de partículas (nieve, niebla, chispas de hielo)

---

## 📝 Licencias

**Dungeon Pack**: Craftpix (revisar `docs/assets/free-2d-top-down-pixel-dungeon-asset-pack/license.txt`)

**Character Pack**: Tiny RPG Character Asset Pack (Free - revisar términos de uso)

**Importante**: Ambos packs son para uso en proyectos. Verifica las licencias antes de distribuir comercialmente.

---

## 🔧 Integración con Godot

### Importando Sprites
1. Godot detectará automáticamente los PNG en estas carpetas
2. Ajusta configuración de importación:
   - **Filter**: Desactivar (para mantener pixel art nítido)
   - **Mipmaps**: Desactivar
   - **Repeat**: Desactivar (a menos que sea para tiles repetidos)

### Creando Animaciones
Para los spritesheets con múltiples frames:
1. Usa **SpriteFrames** resource
2. Carga cada PNG como frame individual
3. Ajusta FPS según velocidad deseada (típicamente 8-12 para pixel art)

### TileMap Setup
Para `walls_floor.png`:
1. Crea un **TileSet** resource
2. Importa el PNG
3. Define las regiones de tiles (típicamente 16x16 o 32x32)
4. Configura colisiones para paredes

---

**Última actualización**: 7 de enero de 2026
