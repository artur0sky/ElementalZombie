# Minecraft Elemental Zombie Mod (Java)

Este documento define la **estructura inicial**, **plan de implementación** y **guía para un agente de desarrollo (Antigravity Agent)** para un mod de Minecraft Java **(Versión 1.21.1)** que introduce **zombies elementales** con habilidades dependientes del bioma, siguiendo **SOLID, DRY y Clean Architecture**, con foco en extensibilidad futura.

---

## 1. Visión del Mod

Crear una entidad base `ElementalZombie` capaz de:

* Detectar el bioma donde existe.
* Asignar dinámicamente un **poder elemental** según el bioma.
* Cambiar de poder cuando el bioma cambia.
* Mantener una arquitectura abierta a nuevos biomas y habilidades sin modificar código existente.

Ejemplos iniciales:

* **Zombie de Agua**: nadar, respirar bajo el agua, golpe torpedo y debilidad a esponjas.
* **Zombie de Nieve**: lanza proyectiles congelantes, genera bolas gigantes y debilidad al fuego.
* **Mecánica Global**: al recibir daño crítico, se dividen en mini-zombies elementales.

---

## 2. Principios de Diseño

### SOLID

* **S**: Cada clase tiene una sola responsabilidad (entidad, habilidad, detección de bioma).
* **O**: Nuevos poderes se agregan sin modificar la entidad base.
* **L**: Todas las habilidades son intercambiables vía interfaces.
* **I**: Interfaces pequeñas (`ElementalPower`, `BiomeResolver`).
* **D**: La entidad depende de abstracciones, no implementaciones.

### Clean Architecture (adaptada a mods)

```
mod/
 ├─ domain/        <- lógica pura (sin Minecraft)
 ├─ application/   <- casos de uso
 ├─ infrastructure <- Minecraft Forge/Fabric
 └─ bootstrap/     <- registro del mod
```

---

## 3. Estructura de Carpetas

```
src/main/java/com/tuusuario/elementalzombie/
│
├─ bootstrap/
│   └─ ElementalZombieMod.java
│
├─ domain/
│   ├─ entity/
│   │   └─ ElementalZombieModel.java
│   ├─ power/
│   │   ├─ ElementalPower.java
│   │   ├─ WaterPower.java
│   │   └─ IcePower.java
│   └─ biome/
│       └─ BiomeType.java
│
├─ application/
│   ├─ resolver/
│   │   └─ BiomePowerResolver.java
│   └─ service/
│       └─ ElementalPowerService.java
│
├─ infrastructure/
│   ├─ entity/
│   │   └─ ElementalZombieEntity.java
│   ├─ ai/
│   │   └─ goals/
│   ├─ projectile/
│   │   └─ IceBallEntity.java
│   └─ biome/
│       └─ MinecraftBiomeAdapter.java
│
└─ util/
    └─ CooldownTracker.java
```

---

## 4. Dominio (Domain Layer)

### ElementalPower.java

```java
public interface ElementalPower {
    void onTick(ElementalZombieModel zombie);
    void onAttack(ElementalZombieModel zombie);
    void onHit(ElementalZombieModel zombie, Object target);
    void onBiomeEnter(ElementalZombieModel zombie);
    String getId();
}
```

### BiomeType.java

```java
public enum BiomeType {
    OCEAN,
    RIVER,
    SNOW,
    FROZEN,
    DEFAULT
}
```

### ElementalZombieModel.java

```java
public class ElementalZombieModel {
    private ElementalPower currentPower;

    public void setPower(ElementalPower power) {
        this.currentPower = power;
        power.onBiomeEnter(this);
    }

    public ElementalPower getPower() {
        return currentPower;
    }
}
```

---

## 5. Poderes Iniciales

### WaterPower (Zombie de Agua)

**Habilidades**:

*   **Pasiva**: Nadar, respirar y caminar sobre/bajo el agua.
*   **Básica**: Quita oxígeno al jugador con cada golpe.
*   **Definitiva**: Golpe torpedo que lanza al jugador por los aires.

**Debilidades**:

*   **Cubetas y Esponjas**: Interactuar con estos items cerca del zombie le causa daño masivo o lo aturde.

```java
public class WaterPower implements ElementalPower {
    public void onTick(ElementalZombieModel zombie) { }
    public void onAttack(ElementalZombieModel zombie) { }
    public void onHit(ElementalZombieModel zombie, Object target) { }
    public void onBiomeEnter(ElementalZombieModel zombie) { }
    public String getId() { return "water"; }
}
```

### SnowPower (Zombie de Nieve)

**Habilidades**:

*   **Pasiva**: Congela al jugador después de varios ataques consecutivos.
*   **Básica**: Lanza bolas de nieve con daño aumentado.
*   **Definitiva**: Genera una bola de nieve gigante mientras camina; al lanzarla, congela al objetivo inmediatamente.

**Debilidades**:

*   **Fuego**: Estar cerca del fuego o ser impactado por ataques de fuego le causa debilidad y daño extra.

```java
public class SnowPower implements ElementalPower {
    public void onTick(ElementalZombieModel zombie) { }
    public void onAttack(ElementalZombieModel zombie) { }
    public void onHit(ElementalZombieModel zombie, Object target) { }
    public void onBiomeEnter(ElementalZombieModel zombie) { }
    public String getId() { return "snow"; }
}
```

---

## 5.1 Mecánica de División (Split Mechanic)

Al ser malheridos (baja salud), los zombies elementales se transforman o invocan versiones pequeñas de sí mismos (`MiniElementalZombie`).

*   **Activación**: Salud < 20%.
*   **Efecto**: El zombie grande desaparece y aparecen 1-2 mini-zombies con el mismo poder elemental.

---

## 6. Application Layer

### BiomePowerResolver.java

```java
public interface BiomePowerResolver {
    ElementalPower resolve(BiomeType biome);
}
```

### ElementalPowerService.java

```java
public class ElementalPowerService {
    private final BiomePowerResolver resolver;

    public void updatePower(ElementalZombieModel zombie, BiomeType biome) {
        ElementalPower newPower = resolver.resolve(biome);
        if (!newPower.getId().equals(zombie.getPower().getId())) {
            zombie.setPower(newPower);
        }
    }
}
```

---

## 7. Infrastructure (Minecraft)

### ElementalZombieEntity.java

Responsabilidades:

* Adaptar ticks de Minecraft → `onTick`
* Traducir eventos de daño/ataque
* Obtener bioma actual

Nunca debe contener lógica de poder.

---

## 8. Plan de Implementación

### Fase 1 – Base

* Mod vacío (Forge o Fabric).
* Registrar entidad zombie custom.
* Reemplazar IA básica.

### Fase 2 – Arquitectura

* Implementar dominio puro (sin imports de MC).
* Resolver bioma → poder.

### Fase 3 – Poderes

* Implementar `WaterPower`.
* Implementar `IcePower`.
* Proyectiles custom.

### Fase 4 – Cambio Dinámico

* Escuchar cambio de bioma.
* Swap de poder sin respawn.

### Fase 5 – Extensibilidad

* Registro automático de poderes.
* Configuración por JSON.

---

## 9. Instrucciones para Antigravity Agent

**Rol**: Arquitecto + Implementador

### Reglas Obligatorias

* Prohibido lógica de poderes dentro de entidades MC.
* Cada nuevo poder debe implementar `ElementalPower`.
* Ningún `switch` por bioma fuera del resolver.
* No usar herencia para poderes (usar composición).

### Patrón Base

* Strategy (poder elemental).
* State (poder actual del zombie).
* Adapter (Minecraft → Dominio).

### Extensión Futura

Para agregar un nuevo bioma:

1. Crear clase `XPower implements ElementalPower`.
2. Registrar en `BiomePowerResolver`.
3. (Opcional) agregar assets.

Cero cambios en la entidad.

---

## 10. Próximo Paso Recomendado

Implementar primero **WaterPower**, porque:

* Introduce movimiento avanzado.
* Obliga a desacoplar física del dominio.
* Sirve como base para lava, viento y rayo.

Cuando esté estable, IcePower será trivial.

---

Fin del documento.
