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

* **Zombie Acuático**: control de corrientes, embestida tipo torpedo, inmune al agua.
* **Zombie de Hielo**: lanza proyectiles de nieve que ralentizan y eventualmente congelan.

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

### WaterPower.md

**Comportamiento**:

* Inmune a daño por ahogamiento.
* Detecta jugador en agua → embestida tipo torpedo.
* Manipula velocidad del flujo (empuje).

```java
public class WaterPower implements ElementalPower {
    public void onTick(ElementalZombieModel zombie) { }
    public void onAttack(ElementalZombieModel zombie) { }
    public void onHit(ElementalZombieModel zombie, Object target) { }
    public void onBiomeEnter(ElementalZombieModel zombie) { }
    public String getId() { return "water"; }
}
```

### IcePower.md

**Comportamiento**:

* Ataque a distancia (bola de nieve).
* Cada impacto aplica lentitud.
* Al alcanzar N impactos → congelación temporal.

```java
public class IcePower implements ElementalPower {
    public void onTick(ElementalZombieModel zombie) { }
    public void onAttack(ElementalZombieModel zombie) { }
    public void onHit(ElementalZombieModel zombie, Object target) { }
    public void onBiomeEnter(ElementalZombieModel zombie) { }
    public String getId() { return "ice"; }
}
```

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
