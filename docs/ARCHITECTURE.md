# Architecture - Elemental Zombie Mod

Este documento describe la arquitectura técnica del mod **Elemental Zombie** para **Minecraft Java 1.21.1**, diseñada bajo los principios de **Clean Architecture** y **SOLID** para asegurar un código mantenible, testeable y fácilmente extensible.

## 1. Patrones de Diseño Utilizados

* **Strategy Pattern**: Las habilidades elementales (`ElementalPower`) se implementan como estrategias intercambiables, incluyendo fases pasivas, básicas y definitivas.
* **State Pattern**: El zombie mantiene un estado interno basado en su poder actual y su salud (Mecánica de División al estar "malherido").
* **Adapter Pattern**: Desacopla la lógica del dominio de las clases de Minecraft (detección de items como cubetas/esponjas en la infraestructura).

## 2. Estructura de Capas

El proyecto se organiza en cuatro capas principales para separar la lógica de negocio de los detalles de implementación del juego:

### 🟢 Domain Layer (`src/main/java/com/tuusuario/elementalzombie/domain`)

Contiene la lógica pura y las definiciones de negocio. No debe tener dependencias de Minecraft.

* **entity/**: Modelos de datos del zombie independientes del motor.
* **power/**: Interfaz `ElementalPower` y sus implementaciones concretas (`WaterPower`, `IcePower`).
* **biome/**: Definiciones de biomas abstractos (`BiomeType`).

### 🟡 Application Layer (`src/main/java/com/tuusuario/elementalzombie/application`)

Actúa como orquestador entre el dominio y la infraestructura.

* **resolver/**: Define cómo se asigna un poder a un bioma.
* **service/**: Lógica para actualizar los poderes de las entidades.

### 🔵 Infrastructure Layer (`src/main/java/com/tuusuario/elementalzombie/infrastructure`)

Contiene las implementaciones específicas de Minecraft (Forge o Fabric).

* **entity/**: La clase de la entidad real que hereda de `Zombie`.
* **ai/**: Objetivos de IA personalizados (Goals).
* **biome/**: Implementación de adaptadores para leer datos del mundo de Minecraft.

### 🔴 Bootstrap Layer (`src/main/java/com/tuusuario/elementalzombie/bootstrap`)

Punto de entrada del mod. Se encarga de registrar entidades, items y configurar los servicios.

---

## 3. Flujo de Control

1. **Detección**: `ElementalZombieEntity` (Infrastructure) detecta un cambio de posición.
2. **Traducción**: El `MinecraftBiomeAdapter` convierte el bioma de MC a `BiomeType` (Domain).
3. **Resolución**: El `ElementalPowerService` (Application) consulta al `BiomePowerResolver` qué poder corresponde.
4. **Actualización**: Si el poder es distinto al actual, se invoca `setPower()` en el modelo, disparando `onBiomeEnter`.
5. **Ejecución**: En cada tick, la entidad de infraestructura llama al `onTick()` del poder actual.

## 4. Extensibilidad

Para añadir un nuevo elemento (ej. Fuego):

1. Añadir `FIRE` a `BiomeType`.
2. Crear `FirePower` implementando `ElementalPower`.
3. Registrar la relación bioma -> `FirePower` en el resolver.
4. No se requiere tocar la clase `ElementalZombieEntity`.
